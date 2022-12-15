
# 5G Stand Alone Enviroment using Open Air Interface

This tutorial describes how to setup a complete 5G Stand Alone testbed using the Open Air Interface (OAI) projects. Section 1 has a description of the considered scenarios, and the hardware and software specifications used to run the components. Section 2 presents the necessary steps to download and confiure the OAI core network (OAI-5GCN). The section 3 is optional and shows how to install the Wireshark software. Section 4 presents the download, configuration and build steps to OAI gNB (OAI-gNB). Section 5 presents the OAI user equipment (OAI-UE) configuration file. Section 6, 7 and 8 present the step to configure and run scenarios 1, 2 and 3, respectively. And section 9 presents some possible tools that can be used to test UE/CN communication.

## 1. Scenario

The setup described in this tutorial was done in a single PC, running the OAI-5GCN, OAI-gNB and OAI-UE. Three scenarios are considered:

1. Monolithic gNB
    
<img src="figures/scenario_1.png" width="400">
    
2. CU/DU split in the same machine
    
<img src="figures/scenario_2.png" width="500">
    
3. CU/DU split with CU running in a virtual machine
    
<img src="figures/scenario_3.png" width="500">

- PC configurations
    - OS: Ubuntu 20.04
    - Processor: Intel i7 12XX (n cores X GHz)
    - Memory: 16 GB DDR4 XX MHz
    - Storage: 512 GB NVMe 4
- OAI-5GCN
    - Branch: v1.4.0
    - Basic run
- OAI-gNB
    - Branch: develop
    - Tag: 2022.w42
    - Band: n78
    - Bandwidth: 40 MHz (106 PRBs)
    - USRP: National Instruments USRP-2901 (Ettus B210)
    - Antenna: Wideband (600 MHz to 6 GHz) SMA antenna
- OAI-UE
    - Branch: develop
    - Tag: 2022.w42
    - Band: n78
    - Bandwidth: 40 MHz (106 PRBs)
    - USRP: National Instruments USRP-2901 (Ettus B210)
    - Antenna: Wideband (600 MHz to 6 GHz) SMA antenna
    - IMSI: 208990000007487
    - Key: fec86ba6eb707ed08905757b1bb44b8f
    - OpcKey: C42449363BBAD02B66D16BC975D77CC1
- Common configuration
    - MCC (Mobile Country Code): 208
    - MNC (Mobile Network Code): 99
    - TAC (Tracking Area Code): 0x01
    - SST (Slice/Service Type): 0x01
    - SD (Slice Diferentiator): 0x01

## 2. OAI-5GCN

### 2.1. Installing the pre-requisites

Verify python version. It must be at least 3.6

```console
python3 --version
```
Install git, net-tools, putty and the proper version of docker.

```console
sudo apt install -y git net-tools putty

sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs)  stable"
sudo apt update
sudo apt install -y docker docker-ce

sudo usermod -a -G docker $(whoami)
sudo reboot

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 2.2. Setup

Download OAI 5G core network and checkout to latest tag (v1.4.0)

```console
git clone https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git ~/oai-cn5g-fed
cd ~/oai-cn5g-fed
git checkout v1.4.0
```
Pull CN services images

```console
#!/bin/bash
docker pull oaisoftwarealliance/oai-amf:v1.4.0
docker pull oaisoftwarealliance/oai-nrf:v1.4.0
docker pull oaisoftwarealliance/oai-spgwu-tiny:v1.4.0
docker pull oaisoftwarealliance/oai-smf:v1.4.0
docker pull oaisoftwarealliance/oai-udr:v1.4.0
docker pull oaisoftwarealliance/oai-udm:v1.4.0
docker pull oaisoftwarealliance/oai-ausf:v1.4.0
docker pull oaisoftwarealliance/oai-upf-vpp:v1.4.0
docker pull oaisoftwarealliance/oai-nssf:v1.4.0
# Utility image to generate traffic
docker pull oaisoftwarealliance/trf-gen-cn5g:latest
```
Re-tag images

```console
#!/bin/bash
docker image tag oaisoftwarealliance/oai-amf:v1.4.0 oai-amf:v1.4.0
docker image tag oaisoftwarealliance/oai-nrf:v1.4.0 oai-nrf:v1.4.0
docker image tag oaisoftwarealliance/oai-smf:v1.4.0 oai-smf:v1.4.0
docker image tag oaisoftwarealliance/oai-spgwu-tiny:v1.4.0 oai-spgwu-tiny:v1.4.0
docker image tag oaisoftwarealliance/oai-udr:v1.4.0 oai-udr:v1.4.0
docker image tag oaisoftwarealliance/oai-udm:v1.4.0 oai-udm:v1.4.0
docker image tag oaisoftwarealliance/oai-ausf:v1.4.0 oai-ausf:v1.4.0
docker image tag oaisoftwarealliance/oai-upf-vpp:v1.4.0 oai-upf-vpp:v1.4.0
docker image tag oaisoftwarealliance/oai-nssf:v1.4.0 oai-nssf:v1.4.0
docker image tag oaisoftwarealliance/trf-gen-cn5g:latest trf-gen-cn5g:latest
```

### 2.3. Configuration

For this tutorial, it is used CN basic configuration. The only thing that must be updated is the .yaml file to match common configuration of network and add the UE IMSI data to the CN database.

- Change [docker-compose-basic-nrf.yaml](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/tree/v1.4.0/docker-compose/docker-compose-basic-nrf.yaml) configuration file
    - On oai-amf:
        ```
        SERVED_GUAMI_MCC_0=208
        SERVED_GUAMI_MNC_0=99
        PLMN_SUPPORT_MCC=208
        PLMN_SUPPORT_MNC=99
        PLMN_SUPPORT_TAC=0x1
        SST_0=1
        SD_0=1
        ```
    - On oai-smf:
        ```
        NSSAI_SST0=1
        NSSAI_SD0=1
        ```
    - On oai-spgwu:
        ```
        MCC=208
        MNC=99
        MNC03=099
        TAC=1
        NSSAI_SST_0=1
        NSSAI_SD_0=1
        ```
- Add UE IMSI to the database file ([oai_db2.sql](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/tree/v1.4.0/docker-compose/database/oai_db2.sql))
    - Search for 
        ```
        INSERT INTO `AuthenticationSubscription` (`ueid`, `authenticationMethod`, `encPermanentKey`, `protectionParameterId`, `sequenceNumber`, `authenticationManagementField`, `algorithmId`, `encOpcKey`, `encTopcKey`, `vectorGenerationInHss`, `n5gcAuthMethod`, `rgAuthenticationInd`, `supi`) VALUES
        ```
    - Add UE data to this query
        ```
        ('208990000007487', '5G_AKA', 'fec86ba6eb707ed08905757b1bb44b8f', 'fec86ba6eb707ed08905757b1bb44b8f', '{\"sqn\": \"000000000020\", \"sqnScheme\": \"NON_TIME_BASED\", \"lastIndexes\": {\"ausf\": 0}}', '8000', 'milenage', 'C42449363BBAD02B66D16BC975D77CC1', NULL, NULL, NULL, NULL, '208990000007487');
        ```
    - Search for
        ```
        INSERT INTO `SessionManagementSubscriptionData` (`ueid`, `servingPlmnid`, `singleNssai`, `dnnConfigurations`) VALUES
        ```
    - Add UE data to this query
        ```
        ('208990000007487', '20895', '{\"sst\": 222, \"sd\": \"123\"}','{\"default\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 1,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"NOT_PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"100Mbps\", \"downlink\":\"100Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"12.1.1.4\"}]}}');
        ```

The following services will be set up:

- mysql : 192.168.70.131
- udr: 192.168.70.136
- udm: 192.168.70.137
- ausf: 192.168.70.138
- nrf: 192.168.70.130
- amf: 192.168.70.132
- smf: 192.168.70.133
- spgwu/upf: 192.168.70.134
- ext-dn: 192.168.70.135

## 3. Wireshark (Optional)
### 3.1. Installing
Wireshark is a well know software to analyse network traffic. One of the communication protocols between gNB and CN is NGAP, which is available only on development version. To install it, run the following commands:

```console
sudo add-apt-repository ppa:wireshark-dev/stable
sudo apt update
sudo apt install wireshark
```
After installation, you can check Wireshark version with `wireshark --version` command. It must be higher than 2.9.x.

```console
Wireshark 3.6.7 (Git v3.6.7 packaged as 3.6.7-1~ubuntu20.04.0+wiresharkdevstable)
```

### 3.2. Running
To run Wireshark properly, it must run with admin privileges. Open a new terminal and run the software with `sudo wireshark` command.
     
## 4. OAI-gNB

The installation, buildiung and configuration are the same for all scenarios. On scenario 3, these instructions must be done on all PCs.

### 4.1. Installing the pre-requisites

Install the tools:
- libboost
- libusb
- doxygen
- python3-docutils
- python3-mako
- python3-numpy
- python3-requests
- python3-ruamel.yaml
- python3-setuptools
- cmake
- build-essential

```console
sudo apt install -y libboost-all-dev libusb-1.0-0-dev doxygen python3-docutils python3-mako python3-numpy python3-requests python3-ruamel.yaml python3-setuptools cmake build-essential
```

Build UHD from source. This is needed to do the USRP communication.

```console
git clone https://github.com/EttusResearch/uhd.git ~/uhd
cd ~/uhd
git checkout v4.0.0.0
cd host
mkdir build
cd build
cmake ../
make -j 4
make test # This step is optional
sudo make install
sudo ldconfig
sudo uhd_images_downloader
```

Clone this repository to get gNB configuration files.

```console
git clone https://github.com/felipe-arnhold/oai-testbed.git ~/oai-testbed
```

### 4.2. gNB setup and building

Download RAN code and checkout to 2022.w42 tag

```console
# Get openairinterface5g source code
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git ~/openairinterface5g
cd ~/openairinterface5g
git checkout 2022.w42
```

Install dependencies

```console
# Install dependencies in Ubuntu 20.04
cd ~/openairinterface5g
source oaienv
cd cmake_targets
./build_oai -I
```

Build the gNB and nrUE:
- -w USRP: build the software to use USRP as radio interface
- --ninja: Tell the compiler to use Ninja build system
- --nrUE: build new radio User Equipment
- --gNB: build new radio gNB
- --build-lib all: build all additional libraries
- -c: clear previous builds

```console
cd ~/openairinterface5g
source oaienv
cd cmake_targets
./build_oai -w USRP --ninja --nrUE --gNB --build-lib all -c
```

*For more build options, run ./build_oai --help

### 4.3. Configuration file

This tutorial uses the default [configuration file for band n78 with 106 PRBs](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/2022.w42/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf) as base.

The common configuration for all scenarios is the frequency configuration and supported PLMN.

The figure bellow shows the frequency allocation configured for band n78. There are 106 physical resources blocks (PRBs) and a sub-carrier spacing of 30 kHz, which combined represent a bandwidth of 40 MHz. The absolute point A frequency is 640008, which corresponds to 3600.120 MHz. The SS Block (yellow) is positioned in the middle of the bandwidth, at 641280 (3619.200 MHz). The initial bandwidth part (blue) corresponds to the total bandwidth. The value of controlResourceSetZero is 12, which correspond to a CORESET (red) 48 PRBs long starting at PRB 16 (3GPP 38.213, 13-4 Table).

<img src="figures/Frequency_allocation.svg" width="300">

To support the PLMN configured on UE and CN, replace `tracking_area_code` and `plmn_list` to the code bellow:

```
tracking_area_code  =  1;
    plmn_list = ({
                  mcc = 208;
                  mnc = 99;
                  mnc_length = 2;
                  snssaiList = (
                    {
                      sst = 1;
                      sd  = 0x1; // 0 false, else true
                    }
                  );
                  });
```

**Due some restrictions, it is needed to add the following line to the gNB configuration (after `nr_cellid = 12345678L;`)**
```
min_rxtxtime = 6;
```

## 5. UE configuration file

The UE configuration file consists on IMSI number and the keys

```
uicc0 = {
imsi = "208990000007487";
key = "fec86ba6eb707ed08905757b1bb44b8f";
opc= "C42449363BBAD02B66D16BC975D77CC1";
dnn= "oai";
nssai_sst=1;
nssai_sd=1;
}
```

This file is located at [oai-testbed/oai-cfg-files/ue.conf](https://github.com/felipe-arnhold/oai-testbed/blob/main/oai-cfg-files/ue.conf)

## 6. Scenario 1

The scenario 1 is the monolithic version.

### 6.1 Run the Core Network

```console
cd ~/oai-cn5g-fed/docker-compose/
python3 core-network.py --type start-basic --scenario 1
```

### 6.2 Run the gNB

```console
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-softmodem -O ~/oai-testbed/oai-cfg-files/gnb.sa.band78.fr1.106PRB.usrpb210.conf --sa -E --continuous-tx
```

**Note: There should be only one USRP connected to the USB 3.0 before starting gNB. After started, the second USRP (for UE) can be connected.**

### 6.3 Run the UE

To run the OAI-UE, some information must be passed through command line.

- -r (Bandwidth in number of PRBs): 106
- --numerology (Subcarriers spacing): 1 (30 kHz)
- --band: 78
- -C (Frequency of point A): 3619,2 MHz
- --ue-fo-compensation: Better RF
- --sa: Stand alone
- -E: three-quarter of sampling frequency.

```console
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ue-fo-compensation --sa -E -O ~/oai-testbed/oai-cfg-files/ue.conf
```

## 7. Scenario 2

In scenario 2, the gNB is split in CU and DU. The CU has the PDCP and RRC layers and DU has the RLC, MAC and PHY layers. This split is done through configuration file.

### 7.1 Prepare the CU condiguration file

For CU, it is necessary to add the interface configuration to the gNB configuration (after `nr_cellid = 12345678L;`). The used interface is the F1 through localhost (lo) network interface.
Local address is the CU IP and remote address is the DU IP through lo interface. Two ports must be defined (c and d)

```
tr_s_preference = "f1";
local_s_if_name = "lo";
local_s_address = "127.0.0.4";
remote_s_address = "127.0.0.3";
local_s_portc   = 501;
local_s_portd   = 2153;
remote_s_portc  = 500;
remote_s_portd  = 2153;
```

Since the CU does not have RLC, MAC and PHY layers, the `MACRLCs`, `L1s` and `RUs` parametes must be removed.

The CU configuration file is located at [oai-testbed/oai-cfg-files/gnb.cu.sa.band78.fr1.106PRB.usrpb210.conf](https://github.com/felipe-arnhold/oai-testbed/blob/main/oai-cfg-files/gnb.cu.sa.band78.fr1.106PRB.usrpb210.conf)

### 7.2 Prepare the DU configuration file

For DU, the F1 interface must be configured inside MACRLC configuration. This information must match the information defined in the CU configuration file. Now, the local is the DU IP and remote the CU IP.

```
MACRLCs = (
  {
    num_cc           = 1;
    tr_s_preference  = "local_L1";
    tr_n_preference  = "f1";
    local_n_if_name = "lo";
    local_n_address = "127.0.0.3";
    remote_n_address = "127.0.0.4";
    local_n_portc   = 500;
    local_n_portd   = 2153;
    remote_n_portc  = 501;
    remote_n_portd  = 2153;

  }
);
```

Since the DU does not communicate directly to the core, the `amf_ip_address` and `NETWORK_INTERFACES` parameters are removed.

The DU configuration file is located at [oai-testbed/oai-cfg-files/gnb.du.sa.band78.fr1.106PRB.usrpb210.conf](https://github.com/felipe-arnhold/oai-testbed/blob/main/oai-cfg-files/gnb.du.sa.band78.fr1.106PRB.usrpb210.conf)

### 7.3 Running scenario 2

####  7.3.1 Run the Core Network

```console
cd ~/oai-cn5g-fed/docker-compose/
python3 core-network.py --type start-basic --scenario 1
```

####  7.3.2 Run the gNB-CU

```console
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-softmodem -O ~/oai-testbed/oai-cfg-files/gnb.cu.sa.band78.fr1.106PRB.usrpb210.conf --sa -E --continuous-tx
```

####  7.3.3 Run the gNB-DU

```console
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-softmodem -O ~/oai-testbed/oai-cfg-files/gnb.du.sa.band78.fr1.106PRB.usrpb210.conf --sa -E --continuous-tx
```

####  7.3.4 Run the UE

```console
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ue-fo-compensation --sa -E -O ~/oai-testbed/oai-cfg-files/ue.conf
```

## 8. Scenario 3

In this scenario, the CU runs in a virtual machine, to simulate the CU/DU split in different PCs. The software used to run the CU is the Virtual Box, also with Ubuntu 20.04.5 LTS. The network interface was configured as bridge, so the VM will receive an IP inside the same network of main PC.
Now, the interfaces configured to CU and DU must be updated, since localhost is not used.

### 8.1 Modifications on configuration files

In the CU configuration file, the netwrok interface and IP addresses must be updated. This information can be achieved running `ifconfig` command.
In this case, in the CU side, the network interface is `enp0s3`, the IP of VM is `191.4.204.57` and the IP of the machine were DU is placed is `191.4.204.111`.

```
tr_s_preference = "f1";
local_s_if_name = "enp0s3";
local_s_address = "191.4.204.57";
remote_s_address = "191.4.204.111";
local_s_portc   = 501;
local_s_portd   = 2153;
remote_s_portc  = 500;
remote_s_portd  = 2153;
```

The `NETWORK_INTERFACES` parameter must be updated also with network interface and local IP.

```
NETWORK_INTERFACES :
    {
        GNB_INTERFACE_NAME_FOR_NG_AMF            = "enp0s3";
        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "191.4.204.57";
        GNB_INTERFACE_NAME_FOR_NGU               = "enp0s3";
        GNB_IPV4_ADDRESS_FOR_NGU                 = "191.4.204.57";
        GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
    };
```

The CU configuration file is located at [oai-testbed/oai-cfg-files/gnb.cu.sa.band78.fr1.106PRB.usrpb210_external.conf](https://github.com/felipe-arnhold/oai-testbed/blob/main/oai-cfg-files/gnb.cu.sa.band78.fr1.106PRB.usrpb210_external.conf)

At DU side, the same thing must be done, but now in the `MACRLC` parameter. Note that the interface is `enp0s4` now.

```
MACRLCs = (
  {
    num_cc           = 1;
    tr_s_preference  = "local_L1";
    tr_n_preference  = "f1";
    local_n_if_name = "enp4s0";
    local_n_address = "191.4.204.111";
    remote_n_address = "191.4.204.57";
    local_n_portc   = 500;
    local_n_portd   = 2153;
    remote_n_portc  = 501;
    remote_n_portd  = 2153;
  }
);
```

The DU configuration file is located at [oai-testbed/oai-cfg-files/gnb.du.sa.band78.fr1.106PRB.usrpb210_external.conf](https://github.com/felipe-arnhold/oai-testbed/blob/main/oai-cfg-files/gnb.du.sa.band78.fr1.106PRB.usrpb210_external.conf)

### 8.2 Run the core network

```console
cd ~/oai-cn5g-fed/docker-compose/
python3 core-network.py --type start-basic --scenario 1
```

### 8.3 Allow IP forwarding

The OAI 5G core network creates a network interface called "demo-oai". For a monolithic version, where CN and gNB are in the same machine, everything works fine. However, placing the gNB in another machine, it will be necessary to allow IP packages forwarding on CN machine and add an IP route to the gNB machine.

On core network machine:
```console
sudo sysctl net.ipv4.conf.all.forwarding=1
sudo iptables -P FORWARD ACCEPT
```

On gNB-CU machine (replace IP_CN to actual IP of CN machine):
```console
sudo ip route add 192.168.70.128/26 via IP_CN
```

To test it, ping any CN service from gNB machine, for example:
```console
ping 192.168.70.134
```

### 8.4 Run the gNB-CU on the virtual machine

```console
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-softmodem -O ~/oai-testbed/oai-cfg-files/gnb.cu.sa.band78.fr1.106PRB.usrpb210_external.conf --sa -E --continuous-tx
```

### 8.5 Run the gNB-DU

```console
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-softmodem -O ~/oai-testbed/oai-cfg-files/gnb.du.sa.band78.fr1.106PRB.usrpb210_external.conf --sa -E --continuous-tx
```

### 8.6 Run the UE

```console
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --nokrnmod --ue-fo-compensation --sa -E -O ~/oai-testbed/oai-cfg-files/ue.conf
```

## 9. Testing

With the setup running, two tools can be used to test the UE/CN communication: `ping` and `iperf`. To perform a these tests, the IP address of UE must be know. This can be achieved running `ifconfig` on the UE machine (same as DU and CN) and search for the `oai-tun1` network interface (created by OAI-UE).

### 9.1. ping

- For Uplink test, open a new terminal on UE machine and ping to CN. The interface `oai-tun1` must be defined.

```console
ping 192.168.70.135 -I oai-tun1
```

- For Downlink test, open a new terminal on CN and ping to UE through docker. Replace UE_IP to actual user equipment IP address.

```console
docker exec -it oai-ext-dn ping UE_IP
```

### 9.2. iperf

The iperf test opens a server and a client and can be used to test throughput.

- For uplink test, open a server on the UE side and a client on the CN side with 10 M of bandwith. Replace UE_IP to actual user equipment IP address.
    - On UE:
    
    ```console
    iperf -s -u -i 1 -B UE_IP
    ```
    
    - On CN:
    
    ```console
    docker exec -it oai-ext-dn iperf -u -t 86400 -i 1 -fk -B 192.168.70.135 -b 10M -c UE_IP
    ```
    
- For downlik, open a server on the CN side and a client on the UE side with 10 M of bandwith. Replace UE_IP to actual user equipment IP address.
    - On UE:
    
    ```console
    iperf -u -t 86400 -i 1 -fk -B UE_IP -b 10M -c 192.168.70.135
    ```
    
    - On CN:
    
    ```console
    docker exec -it oai-ext-dn iperf -s -u -i 1 -B 192.168.70.135
    ```
