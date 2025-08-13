# NPS
# SnortML
## how to install snortML
There are a couple easy ways to accomplish this task. Option 1 is to install the Snort 2.9 executable or snort3 tarball from `https://www.snort.org/downloads#snort3-downloads` and then compile with CMAKE + C++ compiler. 

Option 2 is to download the container for snort 3 on docker hub `https://hub.docker.com/r/ciscotalos/snort3` 

Now that we have snort installed, SnortML is going to need libML from `https://github.com/snort3/libml` 
## training snortML
The following video is your new bible: `https://www.youtube.com/watch?v=q3_2nU5vXz0&t=1276s` 

When pulling the git `https://github.com/snort3/libml`, configure.sh should come as well as train.py 

train.py needs all of your labeled and formated datapoints as such:
```python3
data = [
    { 'str':'foo=1', 'attack':0 },
    { 'str':'foo=1%27%20or%201=1%2D%2D', 'attack':1 }
]
```
attack:0 means benign, 1 means malicious

Within the container, run:
```bash
python3 venv venv
#### break for python3 terminal to load #####
source venv /bin/activate
pip install tensor
## need tensorflow dependency for neural network API
./train.py
snort -q --talos --lua 'trace = { modules = { snort_ml = { all = 1 } } }; snort_ml_engine = { http_param_model = "snort.model" }; snort_ml = {};' -r <desired pcap for analysis> 
```
## current state
SnortML is designed for http and ssh, hence the lua text above having http_param_model. I do not believe snort will be the future of this project, although there is a lot to learn from what talos created already.
# Connect to Hamming
FQDN: hamming.uc.nps.edu

`ssh william.k.harris@hamming-sub1.uc.nps.edu`

using the s commands: `sinfo` `srun` `salloc` and their man pages, gain a beards or genai (Omniverse) partition for compute

# splunk MLTK
# zeek
