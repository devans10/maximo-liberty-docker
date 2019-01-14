# Building and deploying an IBM Maximo Asset Management V7.6.1 with Liberty image to Docker
------------------------------------------------------------------------------------

Maximo with Liberty on Docker enables to run Maximo Asset Management with WebSphere Liberty on Docker. The images are deployed fine-grained services instead of single instance. The following instructions describe how to set up IBM Maximo Asset Management V7.6 Docker images. This images consist of several components e.g. WebSphere Liberty, Db2, and Maximo installation program.

![Componets of Docker Images](https://raw.githubusercontent.com/nishi2go/maximo-liberty-docker/master/maximo-docker.png)

## Required packages
--------------------

* IBM Installation Manager binaries from [Installation Manager 1.8 download documents](http://www-01.ibm.com/support/docview.wss?uid=swg24037640)

  IBM Enterprise Deployment (formerly known as IBM Installation Manager) binaries:
  * IED_V1.8.8_Wins_Linux_86.zip

* IBM Maximo Asset Management V7.6.1 binaries from [Passport Advantage](http://www-01.ibm.com/software/passportadvantage/pao_customer.html)

  IBM Maximo Asset Management V7.6.1 binaries:
  * MAM_7.6.1_LINUX64.tar.gz

  IBM WebSphere Liberty base license V9 binaries:
  * wlp-base-license.jar

  IBM HTTP Server and WebSphere Plugin V9 binaries:
  * was.repo.9000.ihs.zip
  * was.repo.9000.plugins.zip

  IBM Java SDK V8 binaries:
  * sdk.repo.8030.java8.linux.zip

  IBM Db2 Advanced Workgroup Edition V11.1 binaries:
  * DB2_AWSE_REST_Svr_11.1_Lnx_86-64.tar.gz

* Feature Pack/Fix Pack binaries from [Fix Central](http://www-945.ibm.com/support/fixcentral/)

  IBM HTTP Server Fixpack V9.0.0.7 binaries:
  * 9.0.0-WS-IHSPLG-FP007.zip

  IBM Java SDK Fixpack V8.0.5.16 Installation Manager Repository binaries:
  * ibm-java-sdk-8.0-5.16-linux-x64-installmgr.zip

  IBM Db2 Server V11.1 Fix Pack 3
  * v11.1.3fp3_linuxx64_server_t.tar.gz

## Building IBM Maximo Asset Management V7.6.1 with Liberty image by using build tool
------------------------------------------------------

Prerequisites: all binaries must be accessible via a web server during building phase.

You can use a tool for building docker images by using the build tool.

Usage:
```
Usage: build.sh [DIR] [OPTION]...

-c | --check            Check required packages
-C | --deepcheck        Check and compare checksum of required packages
-d | --check-dir [DIR]  The directory for validating packages (Docker for Windows only)
-h | --help             Show this help text
```

Procedures:
1. Place the downloaded Maximo, IBM Db2, IBM Installation Manager and IBM WebSphere Application Server traditional binaries on a directory
2. Clone this repository
    ```bash
    git clone https://github.com/nishi2go/maximo-docker.git
    ```
3. Move to the directory
    ```bash
    cd maximo-docker
    ```
4. Run build tool
   ```bash
   bash build.sh [Image directory] [-c] [-C]
   ```

   Example:
   ```bash
   bash build.sh /images -c
   ```

   Example for Docker for Windows:
   ```bash
   bash build.sh "C:/images" -c -d /images
   ```
   Note 1: This script works on Windows Subsystem on Linux.<br>
   Note 2: md5sum is required. For Mac, install it manually - https://raamdev.com/2008/howto-install-md5sum-sha1sum-on-mac-os-x/
5. Run containers by using the Docker Compose file to create and deploy instances:
    ```bash
    docker-compose up -d
    ```
    Note: It will take 3-4 hours (depend on your machine) to complete the installation.
6. Make sure to be accessible to Maximo login page: http://hostname/maximo

## Building IBM Maximo Asset Management V7.6.1 with Liberty image by manually
------------------------------------------------------

Prerequisites: all binaries must be accessible via a web server during building phase.

Procedures:
1. Place the downloaded Maximo, IBM Db2, IBM Installation Manager and IBM WebSphere Application Server traditional binaries on a directory
2. Create docker network for build with:
    ```bash
    docker network create build
    ```
3. Run nginx docker image to be able to download binaries from HTTP
    ```bash
    docker run --name images -h images --network build \
    -v [Image directory]:/usr/share/nginx/html:ro -d nginx
    ```
4. Clone this repository
    ```bash
    git clone https://github.com/nishi2go/maximo-docker.git
    ```
5. Move to the directory
    ```bash
    cd maximo-docker
    ```
6. Build Docker images:
    Build Db2 image:
    ```bash
    docker build -t maximo/db2:11.1.3 -t maximo/db2:latest --network build maxdb
    ```
    Build IBM Enterprise Deployment (IBM Installation Manager) image:
    ```bash
    docker build -t maximo/ibmim:1.8.8 -t maximo/ibmim:latest --network build ibmim
    ```
    Build WebSphere Liberty image:
    ```bash
    docker build -t maximo/maxliberty:18.0.0.2 -t maximo/maxliberty:latest maxliberty --network build maxweb
    ```
    Build IBM HTTP Server image:
    ```bash
    docker build -t maximo/maxweb:9.0.0.7 -t maximo/maxweb:latest --network build maxweb
    ```
    Build Maximo Asset Management Installation image:
    ```bash
    docker build -t maximo/maximo:7.6.1 -t maximo/maximo:latest --network build maximo
    ```
    Note: If the build has failed during Maximo Feature Pack installation, run the docker build again.
7. Run containers by using the Docker Compose file to create and deploy instances:
    ```bash
    docker-compose up -d
    ```
    Note: It will take 3-4 hours (depend on your machine) to complete the installation.
8. Make sure to be accessible to Maximo login page: http://hostname/maximo
