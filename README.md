<h1 align="center">Team_STAMHS_5G</h1>
<p align="center">
    <strong>Description</strong>
    <br>
    Comment
    <br>
    Members of the group incl. matriculation number
</p>
<br/>


# 5G Core Network with Dynamic Slicing and Orchestration

## Introduction:

The objective of this project is to use Open5GS with Network Slicing, several UPFs, and monitoring tools in Docker for traffic analysis in order to create a standalone 5G Core Network.

This system consists of a 5G Standalone (SA) Core Network, multiple UPFs allocated per slice (S-NSSAI), Packetrusher-generated User Equipment (UE) traffic, a simulated 5G RAN with multiple gNBs, and Prometheus & Grafana for real-time monitoring.

Through the automated deployment of new UPFs when current ones surpass their thresholds, the project facilitates dynamic UPF scaling, guaranteeing effective traffic control. Furthermore, it uses Docker and Docker Compose for smooth deployment and offers thorough traffic analysis using iperf3.

## System Architecture

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/System%20Architecture.jpg)

## Requirements:

- Operating System: Ubuntu 20.04.06 LTS (Focal Fossa) (Linux recommended)
- gtp5g - 5G compatible GTP kernel module
- Docker & Docker Compose
- Open5GS (5G Core Implementation)
- Packetrusher (5G RAN Simulator)
- Prometheus (Monitoring)
- Grafana (Visualization)
- Traffic Generators (iperf3)

## Installation Process:

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
### Step 2: Install Dependencies
```bash
sudo apt update 
```
```bash
sudo apt upgrade
```
### Step 3: Configure It 

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

### Step 4: Build The Docker Images

> Tip: This is the recommended way to build the project, you can build the images all at once with a single command taking advantage of docker buildx parallelism.

> Note: This method uses the `docker-bake.hcl` file and requires `docker-buildx-plugin` which is integrated with the Operating System itself.

From the top level directory of the repository run:
```bash
docker buildx bake
```
### Step 5: Run The Containers

Select the appropriate deployment and from the top level directory of the repository run:

Run the network-slicing deployment
```bash
docker compose -f compose-files/network-slicing/docker-compose.yaml --env-file=.env up -d
```
Tear down the network-slicing deployment
```bash
docker compose -f compose-files/network-slicing/docker-compose.yaml --env-file=.env down
```


### Changes in configuration files of Open5GS 5GC U-Plane2

- `docker-open5gs/configs/etc/network-scicling/packetrusher-1.yaml`
```diff
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-25 20:18:50.813311682 +0900
@@ -10,16 +10,17 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.115
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.115
   session:
-    - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+    - subnet: 10.46.0.1/16
+      dnn: internet
+      dev: ogstun
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_ueransim"></a>


## Dockerized Deployment Overview

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Dockerized%20Deployment%20Overview.jpg)

## UPF & Network Slicing

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UPF%20%26%20Network%20Slicing-1.jpg)
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UPF%20%26%20Network%20Slicing-2.png)
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UPF%20%26%20Network%20Slicing-3.png)

## Traffic Simulation - UE Sending Traffic via PacketRusher

#### Packetrusher-1 (UE1 Traffic) 
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Simulation%20-%20UE%20Sending%20Traffic%20via%20PacketRusher-P-1.jpg)

#### Packetrusher-2 (UE2 Traffic)
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Simulation%20-%20UE%20Sending%20Traffic%20via%20PacketRusher-P-2.jpg)

#### Packetrusher-3 (UE3 Traffic)
![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Simulation%20-%20UE%20Sending%20Traffic%20via%20PacketRusher-P-3.jpg)

## Traffic Received at UPF

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Received%20at%20UPF.jpg)

## UPF Internet Connectivity Test

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/UPF%20Internet%20Connectivity%20Test.jpg)

## Traffic Throughput for Each UE

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/Traffic%20Throughput%20for%20Each%20UE.jpg)
















