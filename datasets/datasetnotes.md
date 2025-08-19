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
timestamp = get_timestamp("C:\\Users\\wkenn\\Downloads\\iec104data\\csvfiles\\20200428_UOWM_IEC104_Dataset_c_sc_na_1_attacker2.pcap_Flow.csv")
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
timestamp = get_timestamp("C:\\Users\\wkenn\\Downloads\\iec104data\\csvfiles\\20200428_UOWM_IEC104_Dataset_c_sc_na_1_attacker2.pcap_Flow.csv")
unix_timestamps = convert_to_unix(timestamp)

#function to add timestamps to new csv here
new_csv("C:\\Users\\wkenn\\Downloads\\iec104data\\csvfiles\\20200428_UOWM_IEC104_Dataset_c_sc_na_1_attacker2.pcap_Flow.csv", ".\\testfile.csv", unix_timestamps)
```

## current state
Need to match zeek logs to CSV malicious and benign labels


