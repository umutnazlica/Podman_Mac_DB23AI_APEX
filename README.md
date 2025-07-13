# 🧪 Oracle Database 23ai + APEX Container Environment with Podman on MAC 

> 💡 Steps to create containerized Oracle Database 23ai + APEX environment, running on Podman at MAC OS.


## 🌟 Introduction

This guide includes information related how to run the latest Oracle 23ai (23.7.0.25.01) ARM container image on a MacBook with Podman. And how to run ORDS image and because of latest ORDS image had some problems we will not use the latest one, and because of this ORDS image does not contain APEX installation we will manually install and configure APEX. 

This guide includes information related how to:

- ✅ Download the latest Oracle Database 23ai Free image, start Oracle 23ai free database using Podman.
- ✅ Check Oracle Alert log file, connect to database using several options.
- ✅ Download the Oracle ORDS image, Start ORDS using Podman.
- 📦 Manually install and configure APEX ver 24.2.
- ✅ Connect to APEX instance.

> Ideal for DBAs, developers, whom are exploring Oracle Database 23ai, APEX, and running Oracle Database in containerized environment on MAC.

---

## 🧰 Prerequisites

- A Running Podman on MacOS. (If not already installed , BREW can be used to install Podman).  
 - Brew Install (https://brew.sh/)  
   ```bash
   brew install podman
   ```  
   Podman install instructions can be found : https://podman.io/docs/installation
- Existing account for login to container-registry.oracle.com to download the container images
- SQLCL<br>
  ```bash
  brew install sqlcl
  ```
  sqlcl requires Java 11+. You can install the latest version with:<br>
  ```bash
  brew install --cask temurin
  ```
  ```bash
  export PATH=/opt/homebrew/Caskroom/sqlcl/25.1.1.113.2054/sqlcl/bin:"$PATH"
  ```
- Skopeo recommended. (tool for managing images and can be installed using brew)<br>
  ```bash
  brew install skopeo
  ```
  ```bash
  brew install jq
  ```
---

### ⚙️ Setup Instructions

--- Check podman version <br>
```bash
podman -v
```
> podman version 5.5.2 

--- Confirm current hardware platform <br>
```bash
arch
```
> arm64 

--- Check MacOS version <br>
```bash
sw_vers -ProductVersion
```
>14.7.6 

--- Initialize VM <br>
```bash
podman machine init --cpus 4 --memory 8192 --disk-size 50
```
```bash
podman machine start
```
```bash
podman machine ls
```

![Image](https://github.com/user-attachments/assets/cd7fba3d-30c0-4276-94ee-0cfe8b63288e)

--- Login to Oracle Container Registry <br>
```bash
podman login container-registry.oracle.com
```

> Username: "YourRegisteredEMailAddress" <br>
  Password: "YourPassword"<br> 
  Login Succeeded! <br>
  
> You need to have registered user for container-registry.oracle.com to be able to download container images,and standard terms and restrictions should be accepted if you login for the first time.

--- Confirm available images using skopeo <br>
```bash
skopeo inspect docker://container-registry.oracle.com/database/free:23.7.0.0-arm64 --tls-verify=false | jq '[.RepoTags]'
```
![Image](https://github.com/user-attachments/assets/c431beb7-2267-4cc8-86ed-a69f2889d763)

--- Pull latest Oracle Database 23ai Free image <br>
```bash
podman pull container-registry.oracle.com/database/free:latest --tls-verify=false
```
```bash
podman image list
```

![Image](https://github.com/user-attachments/assets/570638b5-86a9-47db-babe-883a63c48646)

--- Create persistent volume called oradata, verify the volume properties <br>
```bash
podman volume create --label version=23ai oradata
```
```bash
podman volume inspect oradata
```

![Image](https://github.com/user-attachments/assets/0d4a4271-7f08-434e-ad8c-2e3943744951)

--- Setup Podman secret to manage the database password <br>
```bash
pwd
```
make sure you are in the right folder to store the text file, if not cd to that folder <br>
```bash
echo -n <TypeYourDBPasswordHere> > ./oradbsecret.txt
```
```bash
cat oradbsecret.txt
```
```bash
podman secret create orasecret ./oradbsecret.txt
```
```bash
podman secret ls
```

![Image](https://github.com/user-attachments/assets/4d596d1e-b47e-4813-973b-765537abbb88)

--- Create a network in environment so that containers in this network can communicate with each other using a hostname, view the properties. <br>
```bash
podman network create oranetwork
```
```bash
podman network inspect oranetwork
```

![Image](https://github.com/user-attachments/assets/eda8d846-7b94-4be7-8d49-39e03a6381e8)

--- Start the container <br>
```bash
podman run --detach --volume oradata:/opt/oracle/oradata \
--secret orasecret,type=env,target=ORACLE_PWD \
--publish 1521:1521 \
--name oracle23ai \
--hostname oracle23ai \
--network oranetwork \
container-registry.oracle.com/database/free:latest
```

```bash
podman ps 
```

![Image](https://github.com/user-attachments/assets/a793bbd4-75c0-4112-ac74-992746b6d9fb)

--- View database log <br> 
```bash
podman logs oracle23ai
```
--- Connect to database from your mac terminal <br> 
```bash
sql sys/"TypeYourDBPasswordHere"@//localhost:1521/FREE as sysdba
```

Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free**
Version 23.8.0.25.04**

```bash
select INSTANCE_NAME, HOST_NAME, VERSION_FULL, EDITION from v$instance;
```

![Image](https://github.com/user-attachments/assets/641aa5a1-44cf-4e0a-a5b7-9769e7d64443)

--- Connect to database from inside the container <br>
```bash
podman exec -it oracle23ai sqlplus / as sysdba
```

Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free
Version 23.8.0.25.04

SQL> select INSTANCE_NAME, HOST_NAME, VERSION_FULL, EDITION from v$instance;

![Image](https://github.com/user-attachments/assets/641aa5a1-44cf-4e0a-a5b7-9769e7d64443)

--- Connect to container shell <br>
 ```bash
podman exec -it oracle23ai /bin/sh 
 ```



---

### 🔧 Configuration

1. **Clone this repository**  
   ```bash
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
   cd YOUR_REPO
