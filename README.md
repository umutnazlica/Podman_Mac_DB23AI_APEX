# 🧪 Oracle Database 23ai + APEX Container Environment with Podman on MAC 

> 💡 Steps to create containerized Oracle Database 23ai + APEX environment, running on Podman at MAC OS.


## 🌟 Introduction

This guide includes information related how to run the latest Oracle 23ai (23.7.0.25.01) ARM container image on a MacBook with Podman. And how to run ORDS image and because of latest ORDS image had some problems we will not use the latest one, and because of this ORDS image does not contain APEX installation we will manually install and configure APEX. 

This guide includes information related how to:

- ✅ Download the latest Oracle Database 23ai Free image, Start Oracle 23ai free database using Podman.
- ✅ Check Oracle Alert log file, connect to database using several options.
- ✅ Download the Oracle ORDS image, Start ORDS using Podman.
- 📦 Manually install and configure APEX ver 24.2.
- ✅ Connect to APEX instance.

> Ideal for DBAs, developers, whom are exploring Oracle Database 23ai, APEX, and running Oracle Database in containerized environment on MAC.

---

## 🧰 Prerequisites

- A Running Podman on MacOS. (If not already installed , BREW can be used to install Podman)
   Brew Install (https://brew.sh/)
    brew install podman
   Podman install instructions can be found : https://podman.io/docs/installation
- Existing account to Login container-registry.oracle.com
- SQLCL
   brew install sqlcl
   sqlcl requires Java 11+. You can install the latest version with:
     brew install --cask temurin
     export PATH=/opt/homebrew/Caskroom/sqlcl/25.1.1.113.2054/sqlcl/bin:"$PATH"
 - Skopeo - tool for managing images and can be installed using brew.
      brew install skopeo
      brew install jq
      
   

---

### ⚙️ Setup Instructions

--- Check podman version
> podman -v
**podman version 5.5.2** 

--- Confirm current hardware platform
> arch
**arm64** 

--- Check MacOS version
> sw_vers -ProductVersion
**14.7.6** 

--- Initialize VM 
> podman machine init --cpus 4 --memory 8192 --disk-size 50 
> podman machine start
> podman machine ls

**NAME                     VM TYPE     CREATED        LAST UP            CPUS        MEMORY      DISK SIZE**
**podman-machine-default*  applehv     2 minutes ago  Currently running  4           8GiB        50GiB**

--- Login to Oracle Container Registry
> podman login container-registry.oracle.com

**Username: <YourRegisteredEMailAddress>**
**Password: <YourPassword>**
**Login Succeeded!**
**You need to have registerd user for container-registry.oracle.com to be able to download images,and standard terms and restrictions should be accepted if you login for the first time**

--- Confirm available images using skopeo
> skopeo inspect docker://container-registry.oracle.com/database/free:23.7.0.0-arm64 --tls-verify=false | jq '[.RepoTags]'

--- Pull latest Oracle Database 23ai Free image
> podman pull container-registry.oracle.com/database/free:latest --tls-verify=false
> podman image list

**REPOSITORY                                   TAG         IMAGE ID      CREATED      SIZE**
**container-registry.oracle.com/database/free  latest      a06de2558e27  7 weeks ago  9.28 GB**

--- Create persistent volume called oradata, verify the volume properties
> podman volume create --label version=23ai oradata
> podman volume inspect oradata

--- Setup Podman secret to manage the database password
> pwd **make sure you are in the right folder to store the text file, if not cd to that folder**
> echo -n <TypeYourDBPasswordHere> > ./oradbsecret.txt
> cat oradbsecret.txt **verify your password is created**
> podman secret create orasecret ./oradbsecret.txt
> podman secret ls
**ID                         NAME        DRIVER      CREATED             UPDATED**
**96312af6ae84f4f0919d92c9f  orasecret   file        About a minute ago  About a minute ago**

--- Create a network in environment so that containers in this network can communicate with each other using a hostname, view the properties
> podman network create oranetwork
> podman network inspect oranetwork

--- Start the container
> podman run --detach --volume oradata:/opt/oracle/oradata \
--secret orasecret,type=env,target=ORACLE_PWD \
--publish 1521:1521 \
--name oracle23ai \
--hostname oracle23ai \
--network oranetwork \
container-registry.oracle.com/database/free:latest

> podman ps 
**wait antil container reports healthy**
**CONTAINER ID  IMAGE                                               COMMAND               CREATED             STATUS                       PORTS                   NAMES**
**d3d00b45f8f2  container-registry.oracle.com/database/free:latest  /bin/bash -c $ORA...  About a minute ago  Up About a minute (healthy)  0.0.0.0:1521->1521/tcp  oracle23ai**

--- View database log
> podman logs oracle23ai

--- Connect to database from your mac terminal
> sql sys/<TypeYourDBPasswordHere>@//localhost:1521/FREE as sysdba

Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free**
Version 23.8.0.25.04**

SQL> select INSTANCE_NAME, HOST_NAME, VERSION_FULL, EDITION from v$instance;

INSTANCE_NAME    HOST_NAME     VERSION_FULL    EDITION    
________________ _____________ _______________ __________ 
FREE             oracle23ai    23.8.0.25.04    FREE  

--- Connect to database from inside the container
> podman exec -it oracle23ai sqlplus / as sysdba

Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free
Version 23.8.0.25.04

SQL> select INSTANCE_NAME, HOST_NAME, VERSION_FULL, EDITION from v$instance;

INSTANCE_NAME    HOST_NAME     VERSION_FULL    EDITION    
________________ _____________ _______________ __________ 
FREE             oracle23ai    23.8.0.25.04    FREE  

--- Connect to container shell
> podman exec -ti oracle23ai /bin/sh




---

### 🔧 Configuration

1. **Clone this repository**  
   ```bash
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
   cd YOUR_REPO
