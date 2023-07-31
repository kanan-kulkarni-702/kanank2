# Overview

This document describes the steps while implementing ScalarDL in the File Manager Application.
File Manager application is an application which manages files and folders along with Byzantine Fault Detection. The important files are managed by using ScalarDL as a middleware between MySQL database and File Manager application.

- ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) `#f03c15`
- ![#c5f015](https://placehold.co/15x15/c5f015/c5f015.png) `#c5f015`
- ![#1589F0](https://placehold.co/15x15/1589F0/1589F0.png) `#1589F0`    

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

For database

     scalar.db.storage=multi-storage
     scalar.db.multi_storage.storages=mysql,mysql1

     scalar.db.multi_storage.storages.mysql.storage=jdbc
     scalar.db.multi_storage.storages.mysql.contact_points=jdbc:mysql://mysql_container_forLedger:3306/
     scalar.db.multi_storage.storages.mysql.username=root
     scalar.db.multi_storage.storages.mysql.password=root


     scalar.db.multi_storage.storages.mysql1.storage=jdbc
     scalar.db.multi_storage.storages.mysql1.contact_points=jdbc:mysql://{SCALAR-DB-IP}:3306/
     scalar.db.multi_storage.storages.mysql1.username={Your-UserName}
     scalar.db.multi_storage.storages.mysql1.password={Your-Password}
     scalar.db.multi_storage.namespace_mapping=coordinator:mysql1,scalardb:mysql1,scalar_file_management:mysql1,scalar:mysql
     scalar.db.multi_storage.default_storage=mysql1


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


 ## C. Registering Contracts/Functions To Scalar DL

We are using contracts and functions of ScalarDL. The functions are used in case scalarDB operation is successful and DL operation fails. We are keeping audit status as 3. But if DL operation fails, then control is lost. Initially, we are keeping audit status 3, and then after both the operations are successful, we change the status to 1 via the UpdateFunction.

### Prerequisite:
Please go through the links below before performing this step.

https://github.com/scalar-labs/scalardl/blob/master/docs/how-to-write-contract.md

https://github.com/scalar-labs/scalardl/blob/master/docs/how-to-write-function.md

https://github.com/scalar-labs/scalardl/blob/master/docs/getting-started-auditor.md

### Modify client.properties
This file is being used by the application. We need to mention the client certificate path and key in this document. This is necessary for interacting with ScalarDL. The application is authenticated by DL, and the app can perform DL operations.

The first thing you need to do is configure the Client SDK. The following sample properties are required properties for the Client SDK to interact with ScalarDL Ledger.

## Refer: 
   https://github.com/scalar-labs/scalardl/blob/master/docs/getting-started.md#configure-properties

> Optional. A hostname or an IP address of the server. Use localhost by default if not
>  specified.
>  It assumes that there is a single endpoint that is given by DNS or a load balancer.

    scalar.dl.client.server.host=localhost
>  Optional. A port number of the server. Use 50051 by default if not specified.

    scalar.dl.client.server.port=50051

>  Optional. A port number of the server for privileged services. Use 50052 by default if not specified.
>  scalar.dl.client.server.privileged_port=50052

>  Required. The holder ID of a certificate.
>  It must be configured for each private key and unique in the system.

    scalar.dl.client.cert_holder_id=client

>  Optional. The version of the certificate. Use 1 by default if not specified.
>  Use another bigger integer if you need to change your private key.
> scalar.dl.client.cert_version=1

>  Required. The path of the certificate file.

    scalar.dl.client.cert_path=/path/to/client.pem

>  Required. The path of a corresponding private key file to the certificate.
>  Exceptionally it can be empty in some requests to privileged services
>  such as registerCertificate and registerFunction since they don't need a signature.

    scalar.dl.client.private_key_path=/path/to/client-key.pem

>  Optional. A flag to enable TLS communication. False by default.

    scalar.dl.client.tls.enabled=false

>  Optional. A custom CA root certificate for TLS communication.
>  If the issuing certificate authority is known to the client, it can be empty.
> scalar.dl.client.tls.ca_root_cert_path=/path/to/ca-root-cert

>  Optional. An authorization credential. (e.g. authorization: Bearer token)
>  If this is given, clients will add "authorization: <credential>" http/2 header.
> scalar.dl.client.authorization.credential=credential

>  Experimental. Proxy server
> scalar.dl.client.proxy.server=localhost:10051

    scalar.dl.client.auditor.enabled=true
    scalar.dl.client.auditor.host=localhost

    scalar.dl.client.auditor.linearizable_validation.contract_id=validate-ledger-1

 
### As indicated above, the following properties are mentioned.

    scalar.dl.client.server.host, 
    scalar.dl.client.cert_holder_id,
    scalar.dl.client.cert_path,
    scalar.dl.client.private_key_path,
    scalar.dl.client.auditor.enabled=true
    scalar.dl.client.auditor.host=localhost
    scalar.dl.client.auditor.linearizable_validation.contract_id= 


## 3. Writing Contracts and Functions as Required by the Application
We are using contracts and functions in this application. The Contracts are used to manage data in ScalarDL, and functions are used when we are accessing ScalarDB from ScalarDL.

### A. Contracts Used in Application
The contracts created and used in the application are:

1) Get-all-age

  -  Get a list of ages for a given asset ID.
2) Get-asset

  -  Get JSON data for a given asset ID and age.
3) Add-item

  - To add items in ScalarDL, only files which are added in the BFD folder.
  - This contract is used when a file asset entry is to be added to ScalarDL.
4) Update-item

  - Rename, update file, delete operations call this contract.
  - The file data in the JSON is modified based on the operation.
5) Validate-ledger

  - Modified existing Validate contract.

Validate-ledger Contract

The application mainly aims at detecting a fault in case of a BFD file.

The application user can select a BFD file and click on the validate button to check if the file is tampered. If validation fails i.e. ledger and auditor data not in sync, the file is shown with a different icon.

ScalarDL provides a contract to validate the data. In the File Manager application, the file records are updated in Ledger and Auditor and the File Operation history is recorded in Ledger and Auditor. Every update, delete, rename operation performed on a file is recorded in Ledger as well as Auditor and the ‘age’ of that Asset is updated.

In order to be able to detect a fault at the ‘age’ level, we have modified the existing ‘Vaildate’ contract and written a few other contracts.

Reference:
https://github.com/scalar-labs/scalardl-java-client-sdk/blob/master/src/main/java/com/scalar/dl/client/contract/ValidateLedger.java


 
### Validate-ledger contract in the application

    public class Validator extends JacksonBasedContract {

    static final String ASSET_ID_KEY = "asset_id";
    static final String AGE_KEY = "age";
    static final String START_AGE_KEY = "start_age";
    static final String END_AGE_KEY = "end_age";
    static final String DATA_KEY = "data";

    @Nullable
    @Override
    public JsonNode invoke(Ledger<JsonNode> ledger, JsonNode argument, @Nullable JsonNode properties) {

        if (!argument.has(ASSET_ID_KEY)) {
            throw new ContractContextException("please set asset_id in the argument");
        }

        String assetId = argument.get(ASSET_ID_KEY).asText();

        int startAge = 0;
        int endAge = Integer.MAX_VALUE;
        if (argument.has(AGE_KEY)) {
            int age = argument.get(AGE_KEY).asInt();
            startAge = age;
            endAge = age;
        } else {
            if (argument.has(START_AGE_KEY)) {
                startAge = argument.get(START_AGE_KEY).asInt();
            }
            if (argument.has(END_AGE_KEY)) {
                endAge = argument.get(END_AGE_KEY).asInt();
            }
        }
        AssetFilter filter =
                new AssetFilter(assetId)
                        .withStartAge(startAge, true)
                        .withEndAge(endAge, true)
                        .withAgeOrder(AssetFilter.AgeOrder.ASC);
        List<Asset<JsonNode>> assets = ledger.scan(filter);
        return null;
    }

    }


### Process to setup the contracts

1.	Create a folder such as ‘config’ in the project and place the client.pem, client-key.pem

2.	Then create contracts.toml to mention all contract-id, contract-binary-name, contract-class-file

### Ex
    [[contracts]]
    contract-id = "get-all-age"
    contract-binary-name = "com.perceptproject.ScalarDBFileManagement.contracts.GetAllAge"
    contract-class-file = "build/classes/java/main/com/perceptproject/ScalarDBFileManagement/contracts/GetAllAge.class"

Explanation: 

contract-id : 			name of contract, 

contract-binary-name :	mention binary name,

contract-class-file:  		mention classpath

3.	If there are functions in your applications,  create functions.toml

         [[functions]]
         function-id ="update-function"
         function-binary-name = "com.perceptproject.ScalarDBFileManagement.functions.Update"
         function-class-file ="build/classes/java/main/com/perceptproject/ScalarDBFileManagement/functions/Update.clas"

4.	Create a ‘register’ file as below.

        #!/bin/bash
        ${SCALAR_SDK_HOME}/client/bin/register-cert --properties 
        ./client.properties
        ${SCALAR_SDK_HOME}/client/bin/register-contracts --properties ./client.properties --contracts-file ./path/to/contracts.toml
        ${SCALAR_SDK_HOME}/client/bin/register-functions --properties ./client.properties --functions-file ./path/to/functions.toml

The contracts, functions and certificates to be registered are mentioned in the above file

Note:  SCALAR_SDK_HOME is the path of Project, which has client sdk, client.properties.

The parameters to be assigned in the above file are:

### /register-cert : 	
It will register the certificate of client (i.e application)
### /register-contracts: 	
provide a contract/list of contracts in a file such as contracts.toml and the client certificate to get register the contracts.
### /register-functions:	
provide a function/list of contracts in a file such as functions.toml and the client properties file to get register functions.

5.	Execute the File using the below statement.

        $ SCALAR_SDK_HOME=/mnt/d/GitWorkSpacePercept/ScalarDB-FileManagementGit/percept/ScalarDB-FileManagement ./register


Refer:
Writing contracts: ‘https://github.com/scalar-labs/scalardl/blob/master/docs/how-to-write-contract.md’

 
## B. Functions used in application

### 1)	update-function

This function is written to access ScalarDB by ScalarDL during a ScalarDL transaction.

The fault detectable data record must be created in ScalarDL. Sometimes, it might be possible that a data transaction is successful in ScalarDB and has failed in ScalarDL. Thus the fault detectable data record is not present in ScalarDL. 
In such a case, a different status is assigned to the file so that the missed record could be rewritten in ScalarDL again. This status is managed in ScalarDB and is to be written during the ScalarDL transaction. Hence, a function is written to handle this problem.

Reference: 
https://github.com/scalar-labs/scalardl/blob/master/docs/how-to-write-function.md


### 4. Service used by the application to interact with ScalarDL

Application used the below service to interact with scalatDL - connect bin

To Execute the Client we are using ClientService

    @Bean
    public ClientService getService() {
        ClientServiceFactory factory = new ClientServiceFactory();

        ClientService service = null;
        try {
            log.info("Creating bean ..");
            service = factory.create(new ClientConfig(new   File(Constants.CLIENT_PROPERTIES)));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return service;
    }
    //


































