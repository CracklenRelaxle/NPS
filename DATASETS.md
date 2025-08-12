Currently in "C:$USER\Downloads\Dataset\unziped\20200428_UOWM_IEC104_Dataset_c_sc_na_1\20200428_UOWM_IEC104_Dataset_c_sc_na_1_iecserver7\" \
Dataset pulled from: https://zenodo.org/records/7108614 \
Maybe it shouldn't be trusted, but itll have to do for Proof of Concept

What fields are useful?

To extract fields, use 
```bash
cat .\20200428_UOWM_IEC104_Dataset_c_sc_na_1_iecserver7.pcap_Flow.csv | foreach {$_.split(',')[1]
```
