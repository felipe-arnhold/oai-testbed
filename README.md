
# 5G Stand Alone Enviroment using Open Air Interface

Uma breve descrição sobre o que esse projeto faz e para quem ele é

**This tutorial is based on 5G_SA... tutorial available on Open Air Interface repository.**

## 1. Scenario

The setup described in this tutorial was done in a single PC, running the OAI-5GCN, OAI-gNB and OAI-UE. Three scenarios are considered
    1. Monolithic gNB
    2. CU/DU split
    3. CU/DU split with CU running in a virtual machine

<img src="figures/scenarios_v2.svg" width="1000">

- PC configurations
    - OS: Ubuntu 20.04
    - Processor: Intel i7 12XX (n cores X GHz)
    - Memory: 16 GB DDR4 XX MHz
    - Storage: 512 GB NVMe 4
- OAI-5GCN
    - Master branch
    - Basic run
- OAI-gNB
    - Branch develop
    - Commit: 0xXX
    - Band: n78
    - Bandwidth: 40 MHz (106 PRBs)
    - USRP: National Instruments USRP-2901 (Ettus B210)
    - Antenna: Wideband (600 MHz to 6 GHz) SMA antenna
- OAI-UE
    - Branch develop
    - Commit: 0xXX
    - Band: n78
    - Bandwidth: 40 MHz (106 PRBs)
    - USRP: National Instruments USRP-2901 (Ettus B210)
    - Antenna: Wideband (600 MHz to 6 GHz) SMA antenna
    - IMSI: 
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
reboot

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

#### Wireshark (Optional)
##### Installing
Wireshark is a well know software to analyse network traffic. One of the communication protocols between gNB and CN is NGAP, which is available only on development version. To install it, run the following commands:

```console
sudo add-apt-repository ppa:wireshark-dev/stable
sudo apt update
sudo apt install wireshark
```
After installation, you can check Wireshark version with `Wireshark --version` command. It must be higher than 2.9.x.

```console
Wireshark 3.4.7 (Git v3.4.7 packaged as 3.4.7-1~ubuntu18.04.0+wiresharkdevstable1)
```

##### Running
To run Wireshark properly, it must run with admin privileges. Open a new terminal and run the software with `sudo wireshark` command.

### 2.2 Setup

```console
git clone https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git ~/oai-cn5g-fed
cd ~/oai-cn5g-fed
git checkout v1.4.0
```

### 2.3. Configuration

For this tutorial, it is used CN basic configuration. The only thing that must be updated is the .yaml file to match common configuration of network and add the UE IMSI data to the CN database.

- Change [docker-compose-basic-nrf.yaml](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/master/docker-compose/docker-compose-basic-nrf.yaml) configuration file
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
- Add UE IMSI to the database file ([oai_db2.sql](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/master/docker-compose/database/oai_db2.sql))
    - Search for 
        ```
        INSERT INTO `AuthenticationSubscription` (`ueid`, `authenticationMethod`, `encPermanentKey`, `protectionParameterId`, `sequenceNumber`, `authenticationManagementField`, `algorithmId`, `encOpcKey`, `encTopcKey`, `vectorGenerationInHss`, `n5gcAuthMethod`, `rgAuthenticationInd`, `supi`) VALUES
        ```
    - Add UE data to this query
        ```
        ('XX', 'XX', 'XX', 'XX', '{\"sqn\": \"000000000020\", \"sqnScheme\": \"NON_TIME_BASED\", \"lastIndexes\": {\"ausf\": 0}}', '8000', 'milenage', 'XX', NULL, NULL, NULL, NULL, 'XX'),
        ```
        
## 3. OAI-gNB

### Installing the pre-requisites

### gNB setup and building

### Configuration file

## 4. Running scenario 1

- Run the Core Network
- Run the gNB
- Run the UE

## 5. Running scenario 2

- Run the Core Network
- Run the gNB-CU
- Run the gNB-DU
- Run the UE

## 6. Running scenario 3

- Modifications on configuration files
- Run the core network

- Note

The OAI 5G core network creates a network interface called "demo-oai". For a monolithic version, where CN and gNB are in the same machine, everything works fine. However, placing the gNB in another machine, it will be necessary to allow IP packages forwarding on CN machine and add an IP route to the gNB machine.

- Run the gNB-CU
- Run the gNB-DU
- Run the UE

## Testing

### Ping
### iperf
