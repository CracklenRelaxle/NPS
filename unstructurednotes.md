Can't get docker on hamming, back on my vm

Different ideas
Maybe make a module?
Downsize data

Run PCAP through Zeek \
Aggregate or correlate logs together \
Singapore university dataset? \
Electric power maybe \
simulated - iec104 \
  -likely iec104 that we need to look at \
backnet for AC \
Sergio \
gaining benign dataset with various items and then attack \
Siam system and splunk? \
Use zeek logs on static data \
Maybe compare a PCAP with dominion energy 

For getting iec104: idaho national labs \
                    ece people \
                    proffessor giovanna erritti
                    

VOMIT OF OTHER NOTES. JUST USE crl+F

```
Minutes from 20250729: 
 
Topics: 
Introductions 
 
SnortML platform 
Research what it does and how it works 
There are a series of comprehensive videos Cisco released explaining how it works that I am going through now 
 
Splunk ML 
Research what it does and how it works 
Get in contact with Zach (a fellow student soon to graduate) who is intimately familiar with the toolkit 
 
Ground Truth and how to get it 
Acquire datasets with labeled data 
There are datasets online, Zach may also have a usable labeled dataset 
There are 3 categories of information:  malicious, benign, and unknown 
 
Final product of the cohort 
I need to create a technical report using the Thesis template 
I need to propose my Thesis and have it approved before working on it 
Bring any issues and roadblocks with me to meetings and to the week at NPS 
 
Meeting and networking with other students 
Do as much as I can while present at NPS 
 
Due outs for me: 
Understand both SnortML and Splunk ML 
(Implied: learn about neural networks) 
Find a dataset to use 
Get my account finalized for NPS 
 
Due outs for mentors:  None at this time. 
 
 
Progress for 20250808 
 
1. Understand both SnortML and Splunk ML 
I spent this week mostly looking into SnortML as it is open source, making it the easier first topic. Detailed notes:  NPS/README.md at main · CracklenRelaxle/NPS 
Wavetops for SnortML: 
Neural network that accepts labeled strings (either 0 for benign or 1 for malicious) in a python3 train.py script for ingest libml/examples/classifier/train.py at master · snort3/libml 
After training, it ingests PCAP data and creates alerts under GID 411 and prepends the message with (snort_ml) SnortML: Machine Learning-based Exploit Detection 
A number is assigned to each string ingested from 0 to 1; the threshold for alerting is hard coded (but easily changeable) at .95, seen at line 76 of libml/src/test/binary_classifier_test.cc at master · snort3/libml 
 
Wavetops for Splunk MLTK: 
Takes the term Tool Kit to heart. MLTK is not a finished AI product, but a “way to create custom machine learning” https://docs.splunk.com/Documentation 
Available models/algorithms: LLM (for crafting regex or summarizing query results); AutoPrediction; Linear regression; Logistic regression, Density function, distribution statistics, probabillistic measures, StateSpaceForecast, State-space method using Kalman filter, K-means, DBSCAN, Spectral Clustering, Birch https://docs.splunk.com/Documentation/MLApp/5.6.0/User/Showcaseexamples 
Logistic regression or distribution statistics (I believe “distribution statistics” is a probability distribution algorithm such as gaussian) are the most probable for the desired end state (using ML to pick anomalous logs) 
 
2. Find a dataset to use 
No progress. All time has been spent getting my virtual environment and NPS account setup.
3. Get my account finalized for NPS 
Account acquired (william.k.harris@nps.edu). something is wrong with password reset for my MFA, working with sysadmins to get it corrected.  
VPN is configured on my laptop. 
UNSTRUCTURED: 
Hamming: remote super computer 
https://nps.edu/web/technology/hpc 
Omniverse – GPUs available 
- make sure VPN capable for remote work 
  Snort3.exe first 
 
How do we represent our data with this model? libml/examples/classifier/train.py at master · snort3/libml · GitHub 
 
LSTM? 
Nvidia Morpheus? 
 
Specific topic for research/technical report? 
 
Write hypothesis and 3 research questions 
- test snort and splunk ml tools/compare performance 
-test and compare own model 
-generate synthetic traffic through llm 
 
Due outs from meeting: 
Gain access to omniverse and Hamming 
Make sure they are remote accessible 
Download snort3.exe 
Look at NVidia Morpheus 
Look at LSTM 
 
 
Hypothesis 1: Machine learning models can help analysts detect anomalous behavior that is not caught with signature-based detection programs. 
Research question 1: How can SnortML be used to detect malicious network trafic that is not already detected through base snort rules? Research question 2: How can Splunk MLTK be used to generate log data for training purposes? Research question 3: When should machine learning be applied to defensive cyberspace analysis and when is it impractical? 






Minutes 20250808 
 
Topics: 
Progress report on preliminary research 
Showed current understanding of Splunk MLTK and SnortML 
Also acquired NPS account 
 
Access to compute/GPUs 
Both Hamming and Omniverse are viable options, pending access through VPN tunnel 
 
Research questions, hypothesis, and topic specificity 
 
Dataset acquisition  
Ensure a dataset is chosen that matches what my unit needs 
Understand how the data has to be formatted/represented in the train.py script for SnortML 
 
Due outs: 
More preliminary research on LSTM, Nvidia Morpheus, and how to write research questions 
Hypothesis/research questions draft 1 
Find a dataset 
Access to compute and Splunk license 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 


Progress report for 20250811 
 
More preliminary research on LSTM, Nvidia Morpheus, and how to write research questions  
I think I have the hang on research questions. Open ended “how” and “why” questions 
LSTM = long short term memory, fixes exploding and vanishing floating points 
NVidia Morpheus is an SDK, option viable for LLM? 
 
Hypothesis/research questions draft 1  
Hypothesis 1: Machine learning models can help analysts detect anomalous behavior that is not caught with signature-based detection programs. 
Research question 1: How can SnortML be used to detect malicious network trafic that is not already detected through base snort rules?  
Things to look at: False positives 
False negatives 
Latency 
Comparison to regular snort/signature repositories 
Get datasets from Ian Watkins 
Research question 2: How can Splunk MLTK be used to find anamolous logs that are not already alerted on via ids?  
Zach Holting? 
Need to figure out how to ingest data since there is no train.py 
Need to figure out where 
Research question 3: When should machine learning be applied to defensive cyberspace analysis and when is it impractical? 
 
Find a dataset  
https://www.kaggle.com/datasets/advaitnmenon/network-traffic-data-malicious-activity-detection 
seems solid. Normal IP traffic 
https://data.niaid.nih.gov/resources?id=mendeley_xw7r4tt54g 
shaky. Seems like aggregated datasets recreated with chatGPT  
https://www.sciencedirect.com/science/article/pii/S1389128624006364 
THIS ONE SEEMS BIS 
https://github.com/antoine-lemay/Modbus_dataset is the download 
STILL HAVE TO DOWNLOAD AND FORMAT DATASETS. INTERNET IS TERRIBLE 
ACTUALLY USING:  https://zenodo.org/records/7108614
Access to compute and Splunk license 
Can't do until the workday starts 
Somewhat supervised: using linear regression number to then round up and say 95% = 100% and use it as training data 
Caldera for fake databases/honeypots? -Jason Lansborough? 
 
Pile of contacts I need: 
Nicolas Lebovitz 
NVidia 
Nlebovitz@nvidia.com 
 
Zach  for AI 
Ian Watkins for dataset 
```
