# Realizing Burst-tolerant Datacenter Networks with Valinor

Microsecond-scale congestion events, known as microbursts, are a main cause of packet loss and poor application performance in today's datacenters.
Given the low network utilization in datacenters, one would expect packet deflection, in-situ re-routing of packets that arrive at a full buffer to a different port, to effectively prevent packet loss. However, if deployed naively, deflection leads to excessive packet re-ordering, exacerbated congestion, and head-of-the-line blocking in switch buffers. Valinor resolves these challenges by selectively deflecting the packets that cause persistent congestion in the network. To enable this, we augment the end-host network stacks with a transport-independent extension that tracks and marks flows with their remaining bytes. Our in-network deflection component uses the flow size information to re-route packets from flows with more data to send. Finally, an extension to the receive-side of end-host stacks retrieves the correct ordering of packets before passing them to transport and higher-level protocols. We evaluate Valinor under various datacenter workloads and show that it is effective in managing microbursts under light and heavy loads. For a network facing 75\% link utilization and bursty traffic, Valinor is able to reduce the mean incast query completion times by 57%, 54%, 76% over ECMP, DRILL, and DIBS when using DCTCP for congestion control and 97%, 97%, 16% when using Swift, respectively.

## Re-producing the simulation results

This repository provides the instructins and files required to run the simulations, extract the results, and plot the figures.We use [Omnet++ simulator](https://omnetpp.org/) and INET framework to run the simulations. [Omnet++ manual](https://doc.omnetpp.org/omnetpp/manual/) and its exmaples are good references for introduction to the simulator. We ran our simulations on Ubuntu machines so all the commands are for Ubuntu.To run the simulations, you should follow the following steps:

* Step 1: Install Omnet++
* Step 2: Install dependencies
* Step 3: Clone the repository
* Step 4: Build the project
* Step 5: Run the simulations and extract the results

### Step 1: Installing Omnet++

The complete instructions for installing Omnet++ can be found in [Omnet++ installation guide](https://doc.omnetpp.org/omnetpp/InstallGuide.pdf). **In case you already have the simulator installed, you can skip this step.** To install Omnet++ on an Ubuntu OS, you should run the following commands:

```
wget https://github.com/omnetpp/omnetpp/releases/download/omnetpp-5.6.2/omnetpp-5.6.2-src-linux.tgz
tar xvfz omnetpp-5.6.2-src-linux.tgz
cd omnetpp-5.6.2/
. setenv
```

For the next line we are assuming that you have extracted omnet in $HOME. If this is not true, replace $HOME with the path you have extracted the omnet files to.

```
echo "export PATH=$HOME/omnetpp-5.6.2/bin:\$PATH" >> ~/.bashrc
```

```
source ~/.bashrc
./configure WITH_QTENV=no WITH_OSG=no WITH_OSGEARTH=no
```

In case, the configure command printed out that "omnetpp-5.6.2/bin" is not added to your path, close and re-open your terminal and run the configure command again. If you are not using GUI, reboot your system using ```sudo reboot```. Make sure you get the **"Your PATH contains /opt/omnetpp-5.6.2/bin. Good!"** message in the configuration process before moving forward.

```
make
```

### Step 2: Installing dependencies

We use sqlite3 in our codes. To install it run the following commands:

```
sudo apt update
sudo apt install sqlite3
sudo apt-get install libsqlite3-dev
```

### Step 3: Cloning the repository

To clone the repository, run the following script:

```
git clone https://github.com/hopnets/valinor_simulations.git
```

### Step 4: Building the project

We have provided a ```build.sh``` shell script to simplify this process. To build the project modules, run the following scripts:

```
cd valinor_simulations/Omnet_Sims/
./build.sh
```

### Step 5: Running the simulations and extracting the results

Considering that running the simulations included in the paper takes a long time (about 2-3 weeks) we provide you with two sets of configurations: 
* A configuration file for running small scale simulations using 1Gbps links instead of 10Gbps and 40Gbps links. In this setup we create 25% background load and change the load generated by incast flows from 10% to 70%.
* The configuration files required for running heavy simulations that are included in the paper.

We have provided bash scripts for running both these sets and extracting their results. We have also included a sample python file which is used to process the extracted results of the small scale simulations and compute the Query Completion Times (QCTs). To run the small scale simulations, extract their results, and measure the QCTs run the following commands:

```
cd dc_simulations/simulations/Valinor_Sims
bash download_dist_files_1g.sh
bash run_simple_1g.sh 
```

The commands above, download the distribution files for the small scale simulations and simulate the following scenarios:
* DCTCP + ECMP
* DCTCP + DRILL
* DCTCP + DIBS
* DCTCP + Valinor

Every scenario is expected to take less than 3 hours. After the simulations are over, our bash script automatically runs the python code for the case with 95% load and prints the results. Accordingly, the produced results should be similar to the following:

```
4,spines,8,aggs,40,servers,1,burstyapps,1,mice,40,reqPerBurst,19.75,bgintermult,0.071,burstyintermult,250,ttl,2,rndfwfactor,2,rndbouncefactor,20000,inc
astfsize,0.00120,mrktimer,0.00120,ordtimer,0,rep,dctcp,ecmp.csv
type,requested, started, finished, %Flow Completion, mean, p10, p20, p50, p90, p99, p99.9
all,700629, 644206, 500509, 71.43709438233358, 0.9035198559196383, 0.0027676920115998985, 0.23800162310879955, 0.7700354720689999, 2.0077613670308008, 
3.4086337941725975, 4.282782992595809
 mice of size, 3272, 3189, 3180, 97.1882640586797, 0.0779965320272783, 5.651716879997438e-05, 0.00016592004219999622, 0.0006479080000001414, 0.00134734
32311002839, 1.0011150451317201, 2.627085622420876
cat of size, 3407, 3330, 1886, 55.35661872615204, 0.6408765407683458, 0.001762701393499988, 0.008222333916000024, 0.209886923125, 2.1534168505725, 3.68
0864207729554, 4.371477864332964
elephant of size, 25, 23, 4, 16.0, 0.14661606853300013, 0.10581463790990003, 0.1121618918198001, 0.12065566903300029, 0.20818581875610012, 0.2419355787
3500995, 0.24531055473290103
 bursty,693925, 637664, 495439, 71.39662067226286, 0.9098244404951409, 0.0029989001767999567, 0.2646591219934001, 0.7704694512530001, 2.014488231308199
8, 3.40962390337562, 4.286127312187507
background,6704, 6542, 5070, 75.62649164677804, 0.2874375921124215, 9.315383320003258e-05, 0.00039761385199992287, 0.0009834576329996736, 1.00056002410
12, 3.2408673568643787, 4.039009506724958
Queries, 0, 17500, 1186, 6.777142857142857, 3.443625160091087, 2.6232832229145, 2.89414442277, 3.4466851690614995, 4.2760554903334995, 4.7126774274068,
 4.9133858131497306

4,spines,8,aggs,40,servers,1,burstyapps,1,mice,40,reqPerBurst,19.75,bgintermult,0.071,burstyintermult,250,ttl,2,rndfwfactor,2,rndbouncefactor,20000,inc
astfsize,0.00120,mrktimer,0.00120,ordtimer,0,rep,dctcp,drill.csv
type,requested, started, finished, %Flow Completion, mean, p10, p20, p50, p90, p99, p99.9
all,703832, 641659, 477687, 67.86946316734675, 1.0298296357981376, 0.0036188928988001314, 0.3491261024022001, 0.876681708508, 2.161909185377001, 3.5131
20593832041, 4.335185638332323
 mice of size, 3286, 3208, 3201, 97.41326841144249, 0.05495433622796219, 5.853442999992353e-05, 0.00016172419599991272, 0.0006790667139999762, 0.001320
0741100005686, 1.0010064390370002, 2.5613873491084593
cat of size, 3434, 3359, 1779, 51.80547466511357, 0.5805412308059658, 0.001609507964200141, 0.0039674866104000735, 0.16146413850600005, 1.8509151254294
003, 3.6145161040695, 4.115616411179996
elephant of size, 26, 26, 3, 11.538461538461538, 0.10031560799166661, 0.09431294124460002, 0.09553894644520003, 0.09921696204700003, 0.1067577331165998
6, 0.10845440660725983, 0.10862407395632583
 bursty,697086, 635066, 472704, 67.81143216188534, 1.0381279545102466, 0.15582109748029982, 0.3939309235071998, 0.87731663874, 2.1664716674509004, 3.51
572875198854, 4.338031374559926
background,6746, 6593, 4983, 73.8659946635043, 0.24262344505187536, 9.783677840004003e-05, 0.0003963509455999659, 0.0009710518150001235, 0.877555242361
4003, 3.0386230910765604, 3.9416558355236884
Queries, 0, 17500, 1172, 6.6971428571428575, 3.492059415592647, 2.5950040902034996, 2.901149741557, 3.548579388255, 4.3844759362044, 4.815061341139679,
 4.932485087877952

4,spines,8,aggs,40,servers,1,burstyapps,1,mice,40,reqPerBurst,19.75,bgintermult,0.071,burstyintermult,250,ttl,2,rndfwfactor,2,rndbouncefactor,20000,incastfsize,0.00120,mrktimer,0.00120,ordtimer,0,rep,dctcp,dibs.csv
type,requested, started, finished, %Flow Completion, mean, p10, p20, p50, p90, p99, p99.9
all,706137, 624386, 411080, 58.2153321522594, 1.0646161792927584, 0.001512368817000032, 0.17786951161260012, 0.8170995053414996, 2.4407175085992, 3.803333011258371, 4.529922768871086
 mice of size, 3291, 3240, 3226, 98.02491643877241, 0.05101097443430658, 5.901520450002362e-05, 0.0008176479999999764, 0.0013882537564999797, 0.0023305600000000926, 1.00149849237175, 2.6297263568915756
cat of size, 3439, 3220, 1121, 32.59668508287293, 0.6864049231728181, 0.001441562464000068, 0.0027685607480001373, 0.03286976585500012, 2.370972504787, 3.938498857517, 4.764982883554406
elephant of size, 26, 23, 3, 11.538461538461538, 0.1281168424240001, 0.09983762160320006, 0.10082622967340012, 0.10379205388400026, 0.16612597866080003, 0.18015111173557996, 0.181553625043058
 bursty,699381, 617903, 406730, 58.155711979593384, 1.073704947534576, 0.0016717374977000489, 0.2029768177814001, 0.8639440373825, 2.4460125040351013, 3.8057035188610926, 4.529962355688849
background,6756, 6483, 4350, 64.38721136767317, 0.21480590182277562, 9.787215050001699e-05, 0.000991770092799904, 0.0015462912245001181, 0.8771840705405998, 3.226393756749811, 4.4459843776404275
Queries, 0, 17500, 249, 1.4228571428571428, 3.5943158852147263, 2.5490543232758, 2.9075970346794, 3.736700805329, 4.6175067445566, 4.8243148405168, 4.926462310074928

4,spines,8,aggs,40,servers,1,burstyapps,1,mice,40,reqPerBurst,19.75,bgintermult,0.071,burstyintermult,250,ttl,2,rndfwfactor,2,rndbouncefactor,20000,incastfsize,0.00120,mrktimer,0.00120,ordtimer,0,rep,dctcp,valinor.csv
type,requested, started, finished, %Flow Completion, mean, p10, p20, p50, p90, p99, p99.9
all,706760, 594809, 526221, 74.45540211670156, 0.8788331791513427, 0.674342980587, 0.676776130346, 0.7723981464309999, 1.465001677253, 2.6333061684715995, 3.3946427234425602
 mice of size, 3294, 3293, 3293, 99.96964177292045, 0.00035123916248010423, 5.622321540017519e-05, 8.739019179988538e-05, 0.0001052160000001301, 0.00016321909359986455, 0.000545883972520001, 0.001543906542167682
cat of size, 3440, 1247, 841, 24.447674418604652, 0.3158612196790214, 0.0008680073169999858, 0.0015985511440002043, 0.018333252799, 0.900304798678, 3.4314982568872017, 4.277152951497738
elephant of size, 26, 6, 3, 11.538461538461538, 0.10142027584833337, 0.096515492409, 0.09732527268999999, 0.09975461353299997, 0.10699132421380012, 0.10861958411698014, 0.10878241010729815
 bursty,700000, 590263, 522084, 74.58342857142857, 0.885285460173189, 0.6744910055508998, 0.6769474992097999, 0.772482567058, 1.4650339578776002, 2.6333134055875806, 3.3946122967514745
background,6760, 4546, 4137, 61.19822485207101, 0.06456373631608628, 7.340362440007853e-05, 9.136001039991016e-05, 0.00011375337600005864, 0.019338946428200204, 2.0580358969355044, 4.008590732094909
Queries, 0, 17500, 8005, 45.74285714285714, 1.6087287174550076, 0.7711473605348, 0.7731677268893999, 0.8809048311120002, 2.6367102924524, 3.3964733290294, 3.5918592925148682
```

The results above illustrate that, under 95% load, ECMP, DRILL, and DIBS complete 6.77%, 6.69%, and 1.42% of the querries, respectively, while Valinor completes 45.74%.

The config files for large scale simulations can be used for evaluating Valinor, DIBS, ECMP, and DRILL while using TCP, DCTCP, and Swift as the transport protocol. Every scenario with these configurations takes 2 to 3 weeks to complete. To run the large scale simulations, first make sure that you are in the right directory ("valinor_simulations/Omnet_Sims/dc_simulations/simulations/Valinor_Sims") and then run the following command to download the distribution files:

```
bash download_dist_files.sh
```

After the distribution files are downloaded, you can use the provided bash scripts to run the large scale simulations for different incast arrival rates (dqps), flow sizes, and scales. Additionally, you can run the simulations for various degrees of burstiness and fat-tree topology. The list of the provided bash scripts is as below:
* **Different arrival rates with 15%, 25%, 50%, and 75% background load**
  * run_15_bg_dqps.sh
  * run_25_bg_dqps.sh
  * run_50_bg_dqps.sh
  * run_75_bg_dqps.sh
* **Different scales with 50% background load**
  * run_50_bg_dscale.sh
* **Different flow size with 50% background load**
  * run_50_bg_dfsize.sh
* **Different degrees of burstiness with 80% constant load**
  * run_80_constant_dburstiness.sh
* **Fat-tree topology under different arrival rates and 50% background load**
  * run_fattree.sh
