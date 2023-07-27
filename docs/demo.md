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

   1. Refer to the Compose File `docker-compose-ledger-auditor.yml` in the shared folder - ScalarDL withMySQL Reference.
   2. MySQL DB is used for Ledger & Auditor.
   3. It is assumed that you have generated certificates for Ledger and Auditor and kept them in the fixture folder inside 'ScalarDL withMySQL 
      Reference'.

**Modify the compose file (`docker-compose-ledger-auditor.yml`)**

_It is required to modify the `docker-compose-ledger-auditor.yml` file to initialize the Ledger and Auditor DBs._

_Please note that we have already provided a modified version of the compose file which can be directly used if you are using ScalarDL with MySQL._

**Update `scalardb-ledger.properties` and `scalardb-auditor.properties` file**

Before running the compose files, it is required to add the necessary details for loading ledger and auditor schemas in the respective properties files.

   1)	Update `scalardb-ledger.properties` file for Loading ledger schema in Ledger(DB)
      

>  The JDBC URL

    scalar.db.contact_points=jdbc:mysql://mysql_container_forLedger:3306/

>  The username and password

    scalar.db.username=root
    scalar.db.password=root

>  JDBC storage implementation

    scalar.db.storage=jdbc
    

   2)	Update `scalardb-ledger.properties` file for loading  auditor-schema in Auditor(DB)

> The JDBC URL

    scalar.db.contact_points=jdbc:mysql://mysql_container_forAuditor:3306/
    
> The username and password

    scalar.db.username=root
    scalar.db.password=root
    
> JDBC storage implementation

    scalar.db.storage=jdbc

   3)	Changes required in ledger.properties file for using ScalarDL Functions 

If the application is designed to use both ScalarDB as well as ScalarDL or more specifically, if ScalarDL functions are used in the application then it is required to update ledger.properties file and then override the existing  ledger.properties in the below image:
       
   `ghcr.io/scalar-labs/scalar-ledger:3.6.0`


**ledger.properties file**

Note: The part written in red color in the below file is added for multi storage databases and for using Functions.

> For ledger

> Name of ledger ("Scalar Ledger" by default). It is used to identify a ledger.

    scalar.dl.ledger.name=Scalar Ledger

> Namespace of ledger tables ("scalar" by default).

    scalar.dl.ledger.namespace=scalar

> Server port (50051 by default).

    scalar.dl.ledger.server.port=50051

> Server privileged port (50052 by default).

    scalar.dl.ledger.server.privileged_port=50052

> Server admin port (50053 by default).

    scalar.dl.ledger.server.admin_port=50053

> Prometheus exporter port (8080 by default). Prometheus exporter will not be started if a negative number is given.

    scalar.dl.ledger.server.prometheus_exporter_port=8080

> A flag to enable TLS between clients and servers (false by default).

    scalar.dl.ledger.server.tls.enabled=false

> Certificate chain file used for TLS communication.
> It can be empty if scalar.dl.ledger.server.tls.enabled=false.

    scalar.dl.ledger.server.tls.cert_chain_path=

> Private key file used for TLS communication.
> It can be empty if scalar.dl.ledger.server.tls.enabled=false .

    scalar.dl.ledger.server.tls.private_key_path=

> A flag to enable asset proof that is used to verify assets (false by default).
> This feature must be enabled in both client and server.

    scalar.dl.ledger.proof.enabled=true

> Private key file used for signing a proof entry.

    scalar.dl.ledger.proof.private_key_path=/scalar/ledger-key.pem

> Required if scalar.dl.ledger.proof.enabled is true and scalar.dl.ledger.proof.private_key_path is empty.
> PEM-encoded private key data.

    scalar.dl.ledger.proof.private_key_pem=

> A flag to enable a function for a mutable database (true by default).

    scalar.dl.ledger.function.enabled=true

> A flag to use nonce as a transaction ID (true by default).

    scalar.dl.ledger.nonce_txid.enabled=true

> A flag to use Auditor (disabled by default).

    scalar.dl.ledger.auditor.enabled=true

> Auditor certificate holder ID ("auditor" by default).

    scalar.dl.ledger.auditor.cert_holder_id=auditor

> Auditor certificate version (1 by default).

    scalar.dl.ledger.auditor.cert_version=1

> A flag to use ordering for better tamper-evidence (false by default).
> Ordering feature is deprecated and will be removed in release 4.0.0.

    scalar.dl.ledger.ordering.enabled=false

> Binary names of contracts that can be executed

    scalar.dl.ledger.executable_contracts=

> A flag to access the asset table directly without going through asset_metadata (false by default).
> This should be set to false for some databases such as Cassandra that incur multiple database lookups for scanning a clustering key with limit > 1.
> This should be set to true if an underlying database can utilize index scan to access the latest asset entry efficiently.

    scalar.dl.ledger.direct_asset_access.enabled=

> A flag to manage transaction states by Ledger (false by default).
> This must be enabled when using JdbcTransactionManager as the transaction manager of Scalar DB.

    scalar.dl.ledger.tx_state_management.enabled=

The updates required in ledger.properties file are as below:

As we are using functions to access ScalarDb, the scalarDB instance details need to be configured in ledger.properties file.
Hence , mysql and mysql1
  1)  We have to mention two configurations for MySQL ( one for Default ScalarDL Ledger & another for ScalarDb (which you are using in the 
      application ))
  2)	Change the username, password, and contact_points accordingly 
  3)	Mention the schemaName and storage name respectively in the ‘scalar.db.multi_storage.namespace_mapping’ property.
      Eg. 
     coordinator:mysql1  (This mapping signifies to use coordinator schema from MySQL1 configuration)
  4)	Set the default MySQL storage in the property scalar.db.multi_storage.default_storage  Note that in the above configuration,  ‘MySQL1’ is 
     the default storage.

**Note:**
Check the statement

 ./ledger.properties:/scalar/ledger/ledger.properties
in the docker-compose-ledger-auditor.yml  which is provided along with the documentation. This statement is actually added to override existing ledger.properties.

In case, ScalarDL and ScalarDB are not used together in the application (  in other words, not using Functions in the application) then it is required to remove this statement from the  docker-compose-ledger-auditor.yml file.

## Run Scalar DL Compose File 
Command: **sudo docker compose -f {compose-file.yml} up -d**

**Result:**
1)	Ledger and auditor certificates have been registered with each other. So the ledger and auditor can talk.
2)	Respective Schema has been loaded (as configured in compose file) in Ledger Container, Auditor Container 
3)	Multiple ports gets exposed like 9901,50051,50052(ledger-envoy) ,9902,40051,40052(scalar-envoy) and much more on that instance 


    ##C. Registering Contracts/Functions To Scalar DL

We are using contracts and functions of ScalarDL. The functions are used in case scalarDB operation is successful and DL operation fails. We are keeping audit status as 3. But if DL operation fails, then control is lost. Initially, we are keeping audit status 3, and then after both the operations are successful, we change the status to 1 via the UpdateFunction.

###Prerequisite:
Please go through the links below before performing this step.

https://github.com/scalar-labs/scalardl/blob/master/docs/how-to-write-contract.md
https://github.com/scalar-labs/scalardl/blob/master/docs/how-to-write-function.md
https://github.com/scalar-labs/scalardl/blob/master/docs/getting-started-auditor.md

###Modify client.properties
This file is being used by the application. We need to mention the client certificate path and key in this document. This is necessary for interacting with ScalarDL. The application is authenticated by DL, and the app can perform DL operations.

The first thing you need to do is configure the Client SDK. The following sample properties are required properties for the Client SDK to interact with ScalarDL Ledger.






























