<h1 align="center">Team_STAMHS_5G</h1>
<div align="center">
 
| Name                  | Matriculation Number |  
|-----------------------|---------|  
| Md. Ashraf Uddin      | 1398481 |  
| Tanvir Ahmed          | 1386435 |  
| Saleque Ahmed         | 1393172 |  
| Md. Sohel Rana        | 1428128 |  
| Hasibuzzaman          | 1448140 |  
| Md. Mosharraf Hossain | 1386448 |  

</div>


# 5G Core Network with Dynamic Slicing and Orchestration

## Introduction

The goal of this project is to use Open5GS to deploy Network Slicing in order to create a standalone 5G core network, 5G RAN based on Packetrusher, 5G RAN with multiple gNBs each using its own responsible AMF, 5G UE (User Equipment) based on Packetrusher using 

Docker and Docker Compose. And Implementation of a management and orchestration solution for Docker containers and 5G NFs. Also, to implement auto scale up of UPFs when the traffic threshold is  exceeded.

## System Architecture

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/System%20Architecture.jpg)

## Requirements

- Operating System: Ubuntu 20.04.06 LTS (Focal Fossa) (Linux recommended)
- gtp5g - 5G compatible GTP kernel module
- Docker & Docker Compose
- Open5GS (5G Core Implementation)
- Packetrusher (5G RAN Simulator)
- Prometheus (Monitoring)
- Grafana (Visualization)
- Traffic Generators (iperf3)

## Installation Process

### Step 1: Environment Setup
- Download and Install Ubuntu 20.04.6 LTS (Focal Fossa) [From here.](https://releases.ubuntu.com/focal/)

> Note: After Ubuntu Installation OS will provide a pop-up message to update. Do not update the version.

- Download and Install gtp5g - 5G compatible GTP kernel module [From here.](https://github.com/free5gc/gtp5g)

- Clone the Open5GS repository from the source link.

```bash
git clone https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g.git
```
- Enter inside the cloned local repository
```bash
cd mobcom-team_stamhs_5g
```
### Step 2: Configure It 

First, update the `.env` file with the desired values to use.

The `.env` file is used to build the images using Make or Docker Compose, as well as deploying in Docker Compose.

`OPEN5GS_VERSION` is the version of Open5GS to use.
- Accepted values are the tags, branches or commit IDs used in the Open5GS project
- Implemented value: v2.7.2
- Tested values: v2.5.5, v2.5.6, v2.5.8, v2.6.1, v2.6.2, v2.6.3, v2.6.4, v2.6.6, v2.7.0, v2.7.1, v2.7.2

`UBUNTU_VERSION` is the version of the ubuntu Docker image used as base for the containers.
- Accepted values are the tags used by Ubuntu in Docker Hub
- Implemented value: focal
- Tested values: focal, jammy

`MONGODB_VERSION` is the version of the mongo Docker image used as database for Open5GS.
- Accepted values are the tags used by MongoDB in Docker Hub
- Implemented value: The one specified in the `.env` file which is 6.0

`NODE_VERSION` is the version of Node.js being used to build the Open5GS WebUI.
- Accepted values are the tags used by Node in Docker Hub for its bookworm-slim image and the Node.js dependency of Open5GS WebUI
- Default value: 20

`DOCKER_HOST_IP` is the IP address of the host running Docker.

### Step 3: Build The Docker Images

> Tip: This is the recommended way to build the project, you can build the images all at once with a single command taking advantage of docker buildx parallelism.

> Note: This method uses the `docker-bake.hcl` file and requires `docker-buildx-plugin` which is integrated with the Operating System itself.

From the top level directory of the repository run:
```bash
docker buildx bake
```
### Step 4: Run The Containers

Select the appropriate deployment and from the top level directory of the repository run:

Run the network-slicing deployment
```bash
docker compose -f compose-files/network-slicing/docker-compose.yaml --env-file=.env up -d
```
Tear down the network-slicing deployment
```bash
docker compose -f compose-files/network-slicing/docker-compose.yaml --env-file=.env down
```

## Testing

### Route adding at upfs:

```bash
docker exec -it <CONTAINER_NAME> /bin/bash
```

```bash
ip route add <UE1_IP_SUBNET> via <UPF_IP> dev <UPF_Interface>
```

```bash
ip route del <UE2_IP_SUBNET> via <UPF_IP> dev <UPF_Interface>
```

### Listening traffic to receive:

```bash
iperf3 -s -i 1 -p <PORT1> & iperf3 -s -i 1 -p <PORT2> & iperf3 -s -i 1 -p <PORT3> &
```

### Generate traffic at UEs:

```bash
ip vrf exec <VRF_IF> iperf3 -c <IPERF_SERVER> -p <PORT> -t 9000 -b <BW-K/M/G>

# Or, to generate 10Gbps traffic

ip vrf exec <VRF_IF> iperf3 -c <IPERF_SERVER> -p <PORT> -t 9000 -b 10G

# Installing iperf3, (if not found):

apt-get install iperf3 -y
```

#### Dockerized Deployment Overview

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Dockerized%20Deployment%20Overview.jpg)

#### UE Registration

##### UE-1 
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UE1%20registration.jpeg)

##### UE-2 
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UE2%20Registration.jpeg)

##### UE-3 
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UE3%20registration.jpeg)

#### IP configurations of UPFs and Packetrusher
##### UPFs

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UPF%20%26%20Network%20Slicing-1.jpg)

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UPF%20%26%20Network%20Slicing-2.png)

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UPF%20%26%20Network%20Slicing-3.png)

##### Packetrusher-1
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Simulation%20-%20UE%20Sending%20Traffic%20via%20PacketRusher-P-1.jpg)

##### Packetrusher-2
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Simulation%20-%20UE%20Sending%20Traffic%20via%20PacketRusher-P-2.jpg)

##### Packetrusher-3
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Simulation%20-%20UE%20Sending%20Traffic%20via%20PacketRusher-P-3.jpg)

#### Listening to receive traffic
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/ready%20to%20receive%20iperf3%20traffics%20from%20multi-ues(3).jpeg)

#### Traffic Throughput for Each UE

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Throughput%20for%20Each%20UE.jpg)

#### Traffic Received at UPF

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Received%20at%20UPF.jpg)

#### UPF Internet Connectivity Test

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UPF%20Internet%20Connectivity%20Test.jpg)

## Conclusion

In this project, we have created an independent 5G Core Network (5GC) with Open5GS that features support for network slicing, multiple User Plane Functions (UPFs), and a containerized architecture. This architecture efficiently routes traffic among UPFs, User Equipments (UEs), and gNodeBs (gNBs) efficiently. We have created an efficient 5G RAN simulation with Packetrusher support featuring multiple gNBs and UEs to support extensive traffic generation and testing. Besides this, iperf3 testing checks that traffic between different UEs is correctly divided between UPFs in accordance with S-NSSAI to ensure proper network slicing.
