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

### ⚙️ Setup Instructions for Oracle Database 23ai Free Image

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

--- Create persistent volume to host APEX files
```bash
podman volume create --label version=24.2  apexdbbin
```
--- Create persistent volume to host APEX files on ORDS container
```bash
podman volume create --label version=24.2  ordsapex
```
--- Create persistent volume to host ORDS config on ORDS container
```bash
podman volume create --label version=25.1.1  ordsconfig
```
--- Setup Podman secret to manage the database password <br>
```bash
pwd
```
make sure you are in the right folder to store the password text file, if not cd to that folder <br>
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
--volume apexdbbin:/opt/oracle/apex \
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
--- grep ip address <br> 
```bash
podman inspect oracle23ai | grep IPAddress
```
--- Connect to database from your mac terminal <br> 
```bash
sql sys/"TypeYourDBPasswordHere"@//localhost:1521/FREE as sysdba
```

```bash
select INSTANCE_NAME, HOST_NAME, VERSION_FULL, EDITION from v$instance;
```

![Image](https://github.com/user-attachments/assets/641aa5a1-44cf-4e0a-a5b7-9769e7d64443)

--- Connect to database from inside the container <br>
```bash
podman exec -it oracle23ai sqlplus / as sysdba
```
```bash
select INSTANCE_NAME, HOST_NAME, VERSION_FULL, EDITION from v$instance;
```
![Image](https://github.com/user-attachments/assets/641aa5a1-44cf-4e0a-a5b7-9769e7d64443)

--- Connect to container shell <br>
 ```bash
podman exec -it oracle23ai /bin/sh 
 ```
### ⚙️ Setup Instructions for ORDS + APEX

--- Pull ORDS 25.1.1 Image from Oracle Container Registry <br>
> We are using the 25.1.1 version instead of latest, because latest version has some problems, will be fixed <br>

```bash
podman pull container-registry.oracle.com/database/ords:25.1.1 --tls-verify=false
```
```bash
podman image list
```
> 1) Download the latest APEX version "Oracle APEX 24.2 - English language only" <br>
> from https://www.oracle.com/tools/downloads/apex-downloads/ to Downloads folder <br>
> 2) Unzip the apex_24.2_en.zip to Downloads folder  <br>

--- Verify the contents of APEX unzipped folder<br>
```bash
cd  ~/downloads/apex_24.2_en/apex
ls
```
--- Copy all files to container apexbin folder
```bash
podman cp ~/downloads/apex_24.2_en/apex/. oracle23ai:/opt/oracle/apex
```
--- Connect to container and verify that files exists
```bash
podman exec -it oracle23ai /bin/sh
```
```bash
cd /opt/oracle/apex
ls
```
```bash
sqlplus / as sysdba
```
```bash
show pdbs
```
--- Create new pdb for APEX Schema
```bash
CREATE PLUGGABLE DATABASE APEXPDB 
  ADMIN USER APEXADM IDENTIFIED BY "TypeYourPasswordHere"
  STORAGE (MAXSIZE 10G)
  DEFAULT TABLESPACE apex 
    DATAFILE '/opt/oracle/oradata/FREE/APEXPDB/apexpdb01.dbf' SIZE 250M 
    AUTOEXTEND ON
    PATH_PREFIX = '/opt/oracle/oradata/FREE/APEXPDB/'
    FILE_NAME_CONVERT = ('/opt/oracle/oradata/FREE/pdbseed/', 
                         '/opt/oracle/oradata/FREE/APEXPDB/');
```
```bash
alter pluggable database APEXPDB open read write;
```
> Save state of pdb in order to open pdb in read write mode during next db startup, otherwise it will stay in mount mode and container will be in unhealhy state at next startup! <br>

```bash
alter pluggable database APEXPDB save state;
```
> Verify the state! <br>

```bash
select con_name, instance_name, state FROM dba_pdb_saved_states;
```

--- Verify new pdb created and in read write mode
```bash
show pdbs
```
--- Alter session to connect to apex pdb, we need to run the install scripts in apex pdb
```bash
ALTER SESSION SET CONTAINER = APEXPDB;
```
```bash
@apexins.sql SYSAUX SYSAUX TEMP /i/
```
--- Make sure you are still in apex pdb
```bash
show con_name;
```
```bash
@apxchpwd.sql
```
> admin user : ADMIN<br>
> admin email : "type your email here"<br>
> admin password : "type your password here"  must be complex password, case sensitive with special character <br>

--- Make sure you are still in apex pdb
```bash
show con_name;
```
```bash
alter user apex_public_user identified by "TypePasswordHere" account unlock;
```
--- Make sure you are still in apex pdb
```bash
show con_name;
```
```bash
@apex_rest_config.sql
```
> Enter a password for the APEX_LISTENER user <br>
> Enter a password for the APEX_REST_PUBLIC_USER user <br>

--- DB Apex Installation complete, you can now exit
```bash
exit
```
--- Test database connection to APEX pdb
```bash
sql sys/"TypeYourSysPasswordHere"@//localhost:1521/APEXPDB as sysdba
```
--- Start ORDS Container
```bash
podman run --rm --name ords \
--network=oranetwork \
-p 8080:8080 \
-p 27017:27017 \
-e DBHOST=oracle23ai \
-e DBPORT=1521 \
-e DBSERVICENAME=APEXPDB \
-e ORACLE_PWD="TypeYourSysPasswordHere" \
--volume ordsconfig:/etc/ords/config \
--volume ordsapex:/opt/oracle/apex \
container-registry.oracle.com/database/ords:25.1.1
```
--- We need to copy apex files to ORDS container
```bash
podman cp ~/downloads/apex_24.2_en/apex/. ords:/opt/oracle/apex
```
--- We need to stop and rerun the container after we copied the apex files
```bash
podman stop ords
```
```bash
podman run --rm --name ords \
--network=oranetwork \
-p 8080:8080 \
-p 27017:27017 \
-e DBHOST=oracle23ai \
-e DBPORT=1521 \
-e DBSERVICENAME=APEXPDB \
-e ORACLE_PWD="TypeYourSysPasswordHere" \
--volume ordsconfig:/etc/ords/config \
--volume ordsapex:/opt/oracle/apex \
container-registry.oracle.com/database/ords:25.1.1
```
--- We can now test APEX installation
> From your browser goto : http://localhost:8080/ords , click GO under Oracle APEX <br>

![Image](https://github.com/user-attachments/assets/05c8a0cf-c7fe-4349-8c10-d8bf9ead9615)

> Type: INTERNAL, User:ADMIN, Password: "TypeAdminPasswordYouCreate" <br>

![Image](https://github.com/user-attachments/assets/f2138fd9-fd93-4d7d-a884-b8bb2fb181c1)

--- You can Create Workspace and start using APEX on your Mac

![Image](https://github.com/user-attachments/assets/126dcae7-25a5-4271-a875-2c7c67a0266c)

---

### 🔧 Configuration

1. **Clone this repository**  
   ```bash
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
   cd YOUR_REPO

  ```
