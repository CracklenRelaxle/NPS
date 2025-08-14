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

## current state
Need to use zeek to generate logs about PCAP, then compare Flow ID of the csv with Zeek ID to label zeek logs as malicious or benign


