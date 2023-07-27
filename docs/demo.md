
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
docker compose -f docker-compose-cfssl.yml up

The docker-compose-cfssl.yml file contains configurations for three Docker images:

Service
Init
OCSP Serve

This will set the environment for execution of commands required to create certificates.










