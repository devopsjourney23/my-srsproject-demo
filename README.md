# my-srsproject-demo

## 01. Basic Architecture

Referring to srsran documentation, we'll try using ZeroMQ method for this setup.
https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup

Youtube Video link to my hands-on demo --> https://www.youtube.com/watch?v=SrawJvOwUk0

![image](https://github.com/devopsjourney23/my-srsproject-demo/assets/142556153/4ef2824a-71c4-4769-9122-418ac79b9efc)



  <ins>Components:</ins>
  
  **srsUE:** [srsRAN 4G](https://github.com/srsran/srsRAN_4G) does include a prototype 5G UE (srsUE) which can be used for testing.
  
  **gNB:** srsRAN Project is a 5G CU/DU solution and does not include a UE application.
  
  **5GC:** we can either use Open5GS or Free5GC components for 5GC setup.
  
  Setup can use either USRP based hardware or ZMQ based virtual radios to simulate e2e environment.
  We'll be demonstrating ZMQ based virtual radio setup which can also be used along with CICD pipeline to automate process.



## 02. VM Machine Readiness

<ins>**Machine Requirements:**</ins> Ubuntu 22.04.1 LTS, 50GB disk, 8GB RAM

<ins>**Installing Docker:**</ins>

```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update


sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo docker run hello-world
```

&nbsp;


## 03. Starting Open5GS
&nbsp;

<ins>**Starting Open5GS:**</ins>  Alternatively, we could use Free5GC project as well. 

```
git clone https://github.com/srsran/srsRAN_Project.git
cd srsRAN_Project/docker/
docker compose up --build 5gc
```

&nbsp;

<ins>**Sample Output**</ins>
```
root@ubuntu22:~/srsRAN_Project/docker# docker compose up --build 5gc
WARN[0000] /root/srsRAN_Project/docker/docker-compose.yml: `version` is obsolete
[+] Building 2.4s (16/16) FINISHED                                                                                                                                                               docker:default
 => [5gc internal] load build definition from Dockerfile                                                                                                                                                   0.0s
 => => transferring dockerfile: 3.11kB                                                                                                                                                                     0.0s
 => [5gc internal] load metadata for docker.io/library/ubuntu:22.04                                                                                                                                        2.2s
 => [5gc internal] load .dockerignore                                                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                                                            0.0s
 => [5gc base 1/8] FROM docker.io/library/ubuntu:22.04@sha256:77906da86b60585ce12215807090eb327e7386c8fafb5402369e421f44eff17e                                                                             0.0s
 => [5gc internal] load build context                                                                                                                                                                      0.0s
 => => transferring context: 142B                                                                                                                                                                          0.0s
 => CACHED [5gc base 2/8] RUN DEBIAN_FRONTEND=noninteractive apt-get update     && apt install -y software-properties-common     && rm -rf /var/lib/apt/lists/*                                            0.0s
 => CACHED [5gc base 3/8] RUN DEBIAN_FRONTEND=noninteractive apt-get update     && apt-get install -y     python3-pip     python3-setuptools     python3-wheel     ninja-build     build-essential     fl  0.0s
 => CACHED [5gc base 4/8] RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y --no-install-recommends wget gnupg     && wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc   0.0s
 => CACHED [5gc base 5/8] RUN echo v2.7.0 > ./open5gsversion                                                                                                                                               0.0s
 => CACHED [5gc base 6/8] RUN git clone --depth 1 --branch $(cat ./open5gsversion) https://github.com/open5gs/open5gs open5gs    && cd open5gs     && meson build --prefix=`pwd`/install     && ninja -C   0.0s
 => CACHED [5gc base 7/8] RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash -     && apt-get install -y nodejs     && cd open5gs/webui     && npm ci --no-optional                               0.0s
 => CACHED [5gc base 8/8] RUN python3 -m pip install pymongo click pyroute2 ipaddress python-iptables                                                                                                      0.0s
 => CACHED [5gc open5gs 1/3] WORKDIR /open5gs                                                                                                                                                              0.0s
 => CACHED [5gc open5gs 2/3] COPY open5gs-5gc.yml open5gs-5gc.yml.in                                                                                                                                       0.0s
 => CACHED [5gc open5gs 3/3] COPY open5gs_entrypoint.sh add_users.py setup_tun.py subscriber_db.cs[v] ./                                                                                                   0.0s
 => [5gc] exporting to image                                                                                                                                                                               0.0s
 => => exporting layers                                                                                                                                                                                    0.0s
 => => writing image sha256:ec5fb3fd04e3105c9dd6df08474b10ab5e3210cbfe433b01124cb1ae5c4a3e86                                                                                                               0.0s
 => => naming to docker.io/library/docker-5gc                                                                                                                                                              0.0s
[+] Running 1/0
 ✔ Container open5gs_5gc  Created                                                                                                                                                                          0.0s
Attaching to open5gs_5gc

_truncated_
.
.
.
.
.
_truncated_

open5gs_5gc  | 04/03 11:21:58.980: [amf] INFO: gNB-N2 accepted[10.53.1.1]:50364 in ng-path module (../src/amf/ngap-sctp.c:113)
open5gs_5gc  | 04/03 11:21:58.980: [amf] INFO: gNB-N2 accepted[10.53.1.1] in master_sm module (../src/amf/amf-sm.c:741)
open5gs_5gc  | 04/03 11:21:58.999: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1231)
open5gs_5gc  | 04/03 11:21:58.999: [amf] INFO: gNB-N2[10.53.1.1] max_num_of_ostreams : 30 (../src/amf/amf-sm.c:780)
```


## 04. Starting gNB

<ins>**Building/Compiling gNB:**</ins>

```
apt-get install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev libzmq3-dev git curl jq -y
git clone https://github.com/srsran/srsRAN_Project.git
cd srsRAN_Project/
mkdir build
cd build/
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j`nproc`
cp /root/srsRAN_Project/build/apps/gnb /usr/bin/gnb
```

&nbsp;

**<ins>Starting gNB application:</ins>**

Logs are at /tmp location.

```
-- Use ue configuration file modified for ZeroMQ.
gnb -c /root/srsRAN_config/gnb_zmq.yaml

--Sample Output
root@srsProject:~# gnb -c /root/srsRAN_config/gnb_zmq.yaml

The PRACH detector will not meet the performance requirements with the configuration {Format 0, ZCZ 0, SCS 1.25kHz, Rx ports 1}.
Lower PHY in executor blocking mode.

-- srsRAN gNB (commit 1483bda30) ==--

Connecting to AMF on 10.53.1.2:38412
Available radio types: zmq.
Cell pci=1, bw=20 MHz, 1T1R, dl_arfcn=368500 (n3), dl_freq=1842.5 MHz, dl_ssb_arfcn=368410, ul_freq=1747.5 MHz

==== gNodeB started ===
Type <t> to view trace
t

          |--------------------DL---------------------|-------------------------UL------------------------------
 pci rnti | cqi  ri  mcs  brate   ok  nok  (%)  dl_bs | pusch  rsrp  mcs  brate   ok  nok  (%)    bsr    ta  phr
   1 4601 |  15   1   20   9.5k   14    5  26%      0 |  65.5   ovl   26    36k   15    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
   1 4601 |  15   1    0      0    0    0   0%      0 |   n/a   n/a    0      0    0    0   0%      0   0us  n/a
```

&nbsp;

<ins>**Packet Capture:**</ins>

```
#Enabled Packet Capture in gNB config files.


# Packet capture is supported at the MAC NAS layer. MAC-layer packets are captured to file a the compact format decoded by the Wireshark. For decoding, use the UDP dissector and the UDP heuristic dissection. Edit the preferences (Edit > Preferences > Protocols > DLT_USER) for DLT_USER to add an entry for DLT=149 with Protocol=udp. Further, enable the heuristic dissection in UDP under: Analyze > Enabled Protocols > MAC-LTE > mac_lte_udp and MAC-NR > mac_nr_udp
# NAS-layer packets are dissected with DLT=148, and Protocol = nas-eps.
```


## 05. Starting UE

<ins>**Building/Compiling srsUE:**</ins>

```
apt-get install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev git curl jq -y
git clone https://github.com/srsRAN/srsRAN_4G.git
cd srsRAN_4G/
mkdir build
cd build/
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j`nproc`
cp /root/srsRAN_4G/build/srsue/src/srsue /usr/bin/srsue

ip netns add ue1  
ip netns list
```

&nbsp;

<ins>**Starting srsUE application:**</ins>

Logs are at /tmp location.

```
--Use ue configuration file modified for ZeroMQ.
srsue /root/srsRAN_config/ue_zmq.conf

--Sample Output
root@srsProject:~# srsue /root/srsRAN_config/ue_zmq.conf
Active RF plugins: libsrsran_rf_zmq.so
Inactive RF plugins:
Reading configuration file /root/srsRAN_config/ue_zmq.conf...

Built in Release mode using commit ec29b0c1f on branch master.

Opening 1 channels in RF device=zmq with args=tx_port=tcp://127.0.0.1:2001,rx_port=tcp://127.0.0.1:2000,base_srate=23.04e6
Supported RF device list: zmq file
CHx base_srate=23.04e6
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
CH0 rx_port=tcp://127.0.0.1:2000
CH0 tx_port=tcp://127.0.0.1:2001
Current sample rate is 23.04 MHz with a base rate of 23.04 MHz (x1 decimation)
Current sample rate is 23.04 MHz with a base rate of 23.04 MHz (x1 decimation)
Waiting PHY to initialize ... done!
Attaching UE...
Random Access Transmission: prach_occasion=0, preamble_index=0, ra-rnti=0x39, tti=334
Random Access Complete.     c-rnti=0x4601, ta=0
RRC Connected
PDU Session Establishment successful. IP: 10.45.1.2
RRC NR reconfiguration successful.
```

&nbsp;

**Packet Capture:**

```
#Enabled Packet Capture in ue config files.

# Packet capture is supported at the MAC, MAC_NR, and NAS layer. MAC-layer packets are captured to file a the compact format decoded by the Wireshark. For decoding, use the UDP dissector and the UDP heuristic dissection. Edit the preferences (Edit > Preferences > Protocols > DLT_USER) for DLT_USER to add an entry for DLT=149 with Protocol=udp. Further, enable the heuristic dissection in UDP under: Analyze > Enabled Protocols > MAC-LTE > mac_lte_udp and MAC-NR > mac_nr_udp
```


## 06. UE Traffic Testing
Refer to section "Testing the network" of https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html

**<ins>Routing:  
</ins>**

```
ip ro add 10.45.0.0/16 via 10.53.1.2
ip r
ip netns exec ue1 ip route add default via 10.45.1.1 dev tun_srsue
ip netns exec ue1 ip r
```

**<ins>Ping:  
</ins>**

```
Uplink:
ip netns exec ue1 ping 10.45.1.1

Downlink:
ping 10.45.1.2
```

**<ins>iPerf3:  
</ins>**

```
apt install iperf3 -y

Start Server on network side:
iperf3 -s -i 1

Start UDP/TCP Stream on UE side:
# TCP
sudo ip netns exec ue1 iperf3 -c 10.53.1.1 -i 1 -t 60
# or UDP
sudo ip netns exec ue1 iperf3 -c 10.53.1.1 -i 1 -t 60 -u -b 10M
```



## 07. Multi-UE Setup

Refer Multi-UE Emulation section of https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html  
<br/>Multi-UE setup requires changes in below components along with installation of GNU-Radio:  
**Open5GS:** Subscriber database needs to be updated with more UEs. Use given "subscriber_db.csv" file.

```
cp /home/lax/subscriber_db.csv /root/srsRAN_Project/docker/open5gs/

#edit /root/srsRAN_Project/docker/open5gs/open5gs.env file as below:
 MONGODB_IP=127.0.0.1
 OPEN5GS_IP=10.53.1.2
 UE_IP_BASE=10.45.0
 DEBUG=false
 SUBSCRIBER_DB="subscriber_db.csv"
 #SUBSCRIBER_DB=001010123456780,00112233445566778899aabbccddeeff,opc,63bfa50ee6523365ff14c1f45f88737d,8000,9,10.45.1.2
 
#Start Open5GS Core
cd /root/srsRAN_Project/docker
docker compose up --build 5gc
```

**gNB:** Use modified gnb config to operate with a channel bandwidth of 10 MHz and use the ZMQ-based RF driver.

```
cp /home/lax/gnb_zmq.yaml /root/srsRAN_config/gnb_zmq.yaml

#Start gNB
gnb -c /root/srsRAN_config/gnb_zmq.yaml
```

**srsUE:** srsUE config files were modified to operate with a channel bandwidth of 10 MHz and use the ZMQ-based RF driver. In addition, the config files were modified to allow the execution of multiple srsUE processes on the same host PC. Specifically, each config file has different: ports for ZMQ connections, pcap/logs filename, imsi, network namespace.

```
cp /home/lax/*.conf /root/srsRAN_config/

#Start three srsUE in three different namespaces and terminals
ip netns add ue1
ip netns add ue2
ip netns add ue3

srsue /root/srsRAN_config/ue1_zmq.conf
srsue /root/srsRAN_config/ue2_zmq.conf
srsue /root/srsRAN_config/ue3_zmq.conf

#If UE needs to connect to internet, IP Forwarding needs to be enabled on Host Machine of Open5GS Core.
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -o <IFNAME> -j MASQUERADE
ip netns exec ue1 ping -i 1 8.8.8.8
```

&nbsp;

**GNU-Radio Companion:**  Run the GRC Flowgraph associated with the broker. Once started, clock "Execute the flow graph".

```
apt install xterm gnuradio -y
cp /home/lax/.Xauthority /root/
gnuradio-companion /root/srsRAN_config/multi_ue_scenario.grc
```
