# Overview

This document describes the steps while implementing ScalarDL in the File Manager Application.
File Manager application is an application which manages files and folders along with Byzantine Fault Detection. The important files are managed by using ScalarDL as a middleware between MySQL database and File Manager application.

The file related details are stored as an asset in ScalarDL Ledger as well as Auditor and thus, in case there is any inconsistency in any one of them, fault can be detected by the system.

## Important Processes:

1. **ScalarDL Installation and Setup**

2. **Setup ScalarDL Ledger and Auditor**

   a. **Creation of Ledger, Auditor, and Client Application Certificates**

   b. **Registration of Certificates between Ledger and Auditor**

   c. **Registering Contracts and Functions to ScalarDL**

3. **Writing Contracts and Functions as required by the application**


1. ScalarDL Installation
Refer: https://github.com/scalar-labs/scalardl/blob/master/docs/installation-with-docker.md

2. Setup ScalarDL Ledger and Auditor
(Refer:  https://github.com/scalar-labs/scalardl/blob/master/docs/ca/caclient-getting-started.md)

Prerequisites:

Installation of golang-cfssl:
Cmd:   sudo apt install golang-cfssl 

Execution of docker-compose-cfssl.yml file:
CMD: docker compose -f docker-compose-cfssl.yml up

The file docker-compose-cfssl.yml is created by us and contains three images : 

- Service
- Init
- OCSP Serve

This will set the environment for execution of commands required to create certificates.

## A. Creation of Certificates for Ledger, Auditor, and Client Application

These certificates are required to establish a communication in a secure way between a client application, Ledger, and Auditor of Scalar DL.
Following commands are executed to create certificates:

```bash
$ openssl ecparam -name prime256v1 -out prime256v1.pem
$ openssl req -new -newkey ec:prime256v1.pem -nodes -keyout client-key.pem.pkcs8 -out client.csr
$ openssl ec -in client-key.pem.pkcs8 -out client-key.pem

$ cfssl sign -remote "localhost:8888" -profile "client" client.csr | cfssljson -bare client -
```

The above process creates client.pem which is a certificate for client application.

After Running this command the files created are listed as below:
```bash
$ ls -l
client-cert.json
client-key.pem
client.csr
client.pem
```
We require three certificates - for ledger, auditor, and client (i.e application)

Hence, the above commands are executed each for Ledger and Auditor. The name of the key is changed from ‘client’ to ‘ledger’ in case of Ledger certificate and is changed from ‘ client’ to auditor’ in case of Auditor certificate.

## B.	Registration of certificates between Ledger and Auditor 

It is necessary to register a Ledger certificate with Auditor and Auditor certificate with Ledger.
The following steps are performed in order to complete registration.

-	Modify the compose file (docker-compose-ledger-auditor.yml) to initialize the Ledger and Auditor DBs.  
Please note that we have already provided a modified version of the compose file which can be directly used if you are using ScalarDL with MySQL.

-	Update scalardb-ledger.properties file and  scalardb-auditor.properties file for loading the ledger and auditor schemas.

-	Run the Compose files. The Ledger and Auditor Schemas are loaded in this process.

Note:

Refer to the Compose File docker-compose-ledger-auditor.yml in the shared folder - ScalarDL withMySQL Reference.
MySQL DB is used for Ledger & Auditor.
It is assumed that you have generated certificates for Ledger and Auditor and kept them in the fixture folder inside 'ScalarDL withMySQL Reference'.












