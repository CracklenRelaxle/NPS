File locally hosted at `"$HOME\Downloads\Dataset\unziped\20200428_UOWM_IEC104_Dataset_c_sc_na_1\20200428_UOWM_IEC104_Dataset_c_sc_na_1_iecserver7\"` \
Dataset pulled from: `https://ieee-dataport.org/documents/iec-60870-5-104-intrusion-detection-dataset#files` \

What fields are useful?

These are all of the available fields in the CSV: \
Flow ID,Src IP,Src Port,Dst IP,Dst Port,Protocol,Timestamp,Flow Duration,Tot Fwd Pkts,Tot Bwd Pkts,TotLen Fwd Pkts,TotLen Bwd Pkts,Fwd Pkt Len Max,Fwd Pkt Len Min,Fwd Pkt Len Mean,Fwd Pkt Len Std,Bwd Pkt Len Max,Bwd Pkt Len Min,Bwd Pkt Len Mean,Bwd Pkt Len Std,Flow Byts/s,Flow Pkts/s,Flow IAT Mean,Flow IAT Std,Flow IAT Max,Flow IAT Min,Fwd IAT Tot,Fwd IAT Mean,Fwd IAT Std,Fwd IAT Max,Fwd IAT Min,Bwd IAT Tot,Bwd IAT Mean,Bwd IAT Std,Bwd IAT Max,Bwd IAT Min,Fwd PSH Flags,Bwd PSH Flags,Fwd URG Flags,Bwd URG Flags,Fwd Header Len,Bwd Header Len,Fwd Pkts/s,Bwd Pkts/s,Pkt Len Min,Pkt Len Max,Pkt Len Mean,Pkt Len Std,Pkt Len Var,FIN Flag Cnt,SYN Flag Cnt,RST Flag Cnt,PSH Flag Cnt,ACK Flag Cnt,URG Flag Cnt,CWE Flag Count,ECE Flag Cnt,Down/Up Ratio,Pkt Size Avg,Fwd Seg Size Avg,Bwd Seg Size Avg,Fwd Byts/b Avg,Fwd Pkts/b Avg,Fwd Blk Rate Avg,Bwd Byts/b Avg,Bwd Pkts/b Avg,Bwd Blk Rate Avg,Subflow Fwd Pkts,Subflow Fwd Byts,Subflow Bwd Pkts,Subflow Bwd Byts,Init Fwd Win Byts,Init Bwd Win Byts,Fwd Act Data Pkts,Fwd Seg Size Min,Active Mean,Active Std,Active Max,Active Min,Idle Mean,Idle Std,Idle Max,Idle Min,Label

To extract fields, use 
```powershell
cat .\20200428_UOWM_IEC104_Dataset_c_sc_na_1_iecserver7.pcap_Flow.csv | foreach {$_.split(',')[1]
```
I moved all files to my Ubuntu image, then processed them with Zeek
## Processing files with Zeek
Zeek installed through `https://github.com/zeek/zeek/wiki/Binary-Packages` \
After installation, edit `/etc/environment` file and add `/opt/zeek/bin` to $PATH \
conducted:
```bash
find ~ -type f -name "*pcap" -exec zeek -r {} \;
# got error about check sums, recommended use option -C. rerunning with -C
find ~ -type f -name "*pcap" -exec zeek -C -r {} \;
# format is terrible, trying with LogAscii::use_json=T
find ~ -type f -name "*pcap" -exec zeek -C -r {} LogAscii::use_json=T \;
#realized there are split pcaps with "only" in the name, removing those with
find ../ -type f -name "*only.pcap" -exec rm {} \;
#now seeing if that fixes the checksum problem
find ~ -type f -name "*pcap" -exec zeek -r {} LogAscii::use_json=T \;
#still throws checksum issue, going back to
find ~ -type f -name "*pcap" -exec zeek -C -r {} LogAscii::use_json=T \;
```
seems successful. ran `zeek -C -r LogAscii::use_json=T` on the largest file to ensure that Zeek appends the logs and doesn't replace. It does append. Moving logs to hamming

to marry the attack values with logs, I will attempt to use timestamps + flow id. In order to match the timestamps of the logs vs the timestamp of the CSV, I will conver the csv time to unix time. The following is script 1 to do so:
```python
#!/bin/python3
from datetime import datetime
import time

def convert_to_unix(timestamp_str):
    # Define the format of the input timestamp
    format_str = "%d/%m/%Y %I:%M:%S %p"
    
    # Parse the string into a datetime object
    dt = datetime.strptime(timestamp_str, format_str)
    
    # Convert to Unix timestamp
    unix_time = int(time.mktime(dt.timetuple()))
    
    return unix_time

# Example usage
timestamp = "28/04/2020 09:15:31 PM"
unix_timestamp = convert_to_unix(timestamp)
print(f"Unix time: {unix_timestamp}")
```
This script appears to do exactly what is needed, as the output is `1588122931`. Now I need to make the script receive a filename as an argument and create a new csv with the desired unix timestamp. Iteration 2 makes a giant list of unix time with a hard coded file input:
```python3
#!/bin/python3
from datetime import datetime
import time
import pandas as pd

def get_timestamp(filename):
    # input the file to the panda DF with error handle
    try:
        df = pd.read_csv(filename)
    except:
        print("CSV not properly loaded")

    # return just the values of the timestamp as a list
    return df['Timestamp'].values




def convert_to_unix(timestamp_list):
    # Define the format of the input timestamp
    unix_time = []
    format_str = "%d/%m/%Y %I:%M:%S %p"
    
    # Parse each string in the list as datatime object
    for i in timestamp_list:
        dt = datetime.strptime(i, format_str)
    
    # Convert to Unix timestamp and add it to the unix_time list
        unix_time.append(int(time.mktime(dt.timetuple())))
    
    return unix_time

#call the functions
timestamp = get_timestamp("$USER\\Downloads\\iec104data\\csvfiles\\20200428_UOWM_IEC104_Dataset_c_sc_na_1_attacker2.pcap_Flow.csv")
unix_timestamp = convert_to_unix(timestamp)
print(f"Unix time: {unix_timestamp}")
```
This next iteration takes a hard coded input file, hard coded output file, and creates a new CSV with the unix timestamp
```python3
#!/bin/python3
from datetime import datetime
import time
import pandas as pd

def get_timestamp(filename):
    # input the file to the panda DF with error handle
    try:
        df = pd.read_csv(filename)
    except:
        print("CSV not properly loaded")

    # return just the values of the timestamp as a list
    return df['Timestamp'].values




def convert_to_unix(timestamp_list):
    # Define the format of the input timestamp
    unix_time = []
    format_str = "%d/%m/%Y %I:%M:%S %p"
    
    # Parse each string in the list as datatime object
    for i in timestamp_list:
        dt = datetime.strptime(i, format_str)
    
    # Convert to Unix timestamp and add it to the unix_time list
        unix_time.append(int(time.mktime(dt.timetuple())))
    
    return unix_time

def new_csv(infile, outfile, timestamplist):
    #add the unix time to the dataframe as a column
    df = pd.read_csv(infile)
    df['unixtimestamp'] = timestamplist

    #create the file
    df.to_csv(outfile, index=False)

#create list of timestamps
timestamp = get_timestamp("$USER\\Downloads\\iec104data\\csvfiles\\20200428_UOWM_IEC104_Dataset_c_sc_na_1_attacker2.pcap_Flow.csv")
unix_timestamps = convert_to_unix(timestamp)

#function to add timestamps to new csv here
new_csv("$USER\\Downloads\\iec104data\\csvfiles\\20200428_UOWM_IEC104_Dataset_c_sc_na_1_attacker2.pcap_Flow.csv", ".\\testfile.csv", unix_timestamps)
```
Lastly - tada! the script parses all csv in a given folder, and sends them to "csvfiles" directory of chosen location with appended unix time.
```python
#!/bin/python3
from datetime import datetime
import time
import pandas as pd
import os

def get_timestamp(filename):
    # input the file to the panda DF with error handle
    try:
        df = pd.read_csv(filename)
    except:
        print("CSV not properly loaded")

    # return just the values of the timestamp as a list
    return df['Timestamp'].values

def convert_to_unix(timestamp_list):
    # Define the format of the input timestamp
    unix_time = []
    format_str = "%d/%m/%Y %I:%M:%S %p"
    
    # Parse each string in the list as datatime object
    for i in timestamp_list:
        dt = datetime.strptime(i, format_str)
    
    # Convert to Unix timestamp and add it to the unix_time list
        unix_time.append(int(time.mktime(dt.timetuple())))
    
    return unix_time

def new_csv(infile, outfile, timestamplist):
    #add the unix time to the dataframe as a column
    df = pd.read_csv(infile)
    df['unixtimestamp'] = timestamplist

    print(f"new file {outfile} should be created")
    #create the file
    df.to_csv(outfile, index=False)

# get user input in the crudest way possible:
print("Please input the name of the directory with files to parse:")
indir = input()
print("Please input the full path of the desired output directory:")
outdir = input()

#create directory for output:
os.makedirs(f"{outdir}\\csvfiles", exist_ok=True)
print(f"Directory '{outdir}\\csvfiles' created.")

#iterate through the given directory
filename = 1
for i in os.listdir(indir):
    if i.endswith('csv'):
        fullpath = os.path.join(indir, i)
        print(fullpath)

    #create list of timestamps
        timestamp = get_timestamp(fullpath)
        unix_timestamps = convert_to_unix(timestamp)

    #function to add timestamps to new csv here
        new_csv(fullpath, f"{outdir}\\csvfiles\\{filename}.csv", unix_timestamps)
        filename += 1
```
Now with the timestamps, I will match as much data as possible between each log and each row of the csv. I will start with a nested for loop going through each csv file and checking every line of all the log files. I can copy the for loop from the previous code, but i will want all output to go to stdout first, so it will be commented out. All logs have source dest ip and port and timestamp except for dhcp.log, weird.log, and packet_filter.log. This is because packet_filter.log should accept all traffic (it logs the bpf used), dhcp doesn't use destination ips or ports at all, and weird.log has 1 entry for a seemingly malformed packet. I will therefore be `using the following columns in my csv: unixtimestamp,Src IP,Src Port,Dst IP,Dst Port.` The more data I use, the less likely I am to have data collision of any kind.
```python3
#!/bin/python3
import pandas as pd
import os
import re

#function for reading the file and extracting regex string to apply to logs
def make_match_string(infile, index):
    df = pd.read_csv(infile)
  #  print(df.loc[index])

    #start creating my regex bash with each column
    unixtimestamp = f"{df['unixtimestamp'].loc[index]}"
    srcport = f"{df['Src Port'].loc[index]}"
    dstport = f"{df['Dst Port'].loc[index]}"
    srcip = f"{df['Src IP'].loc[index]}"
    dstip = f"{df['Dst IP'].loc[index]}"

    #doctor up IPs to escape "."
    srcip = srcip.replace(".", "\.")
    dstip = dstip.replace(".", "\.")

    #now combine my unholy regex
    regexstring = f"{unixtimestamp}.*{srcip}.*{srcport}.*{dstip}.*{dstport}"

    return regexstring

def main():
    fullpath = "C:\\Users\\wkenn\\Downloads\\iec104data\\csvfilesandunixtime\\1.csv"
    index = 1
    print(make_match_string(fullpath, index))
    
main()
```
This first function creates a regex string of a given row in the csv for use against the log data. I now need to make the function that finds the correct log. Then I need to make everything iterative.

Unfortunately, these unix timestamps don't match. I do not understand why, but the timestamps of the CSV don't match up to the zeek logs or the PCAP. I do not know where these timestamps were pulled from. Working theory is that my unix conversion is not taking timezone into account, and that the timestamps given are not in UTC. Will see if there is a delta between the greatest timestamp of both the CSVs and the logfiles. Initial analysis shows a difference of 7 hours; the same amount of time difference between greece and here. 

## current state
Need to match zeek logs to CSV malicious and benign labels


