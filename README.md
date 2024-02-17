## Introduction
### Diagram
![image](https://github.com/momchilosavic/keycloak/assets/48445874/5cdb3482-7c01-48cd-82a7-b8d9f493a619)
## Solutions
### Deploy Keycloak and a separate Infinispan to a local Kubernetes cluster 
#### Documentation
[Guide to Infinispan Server, ](https://infinispan.org/docs/stable/titles/server/server.html)
### Connect Keycloak to a database (you are free to choose how you do that)
#### Documentation
[Configuring the database - Keycloak](https://www.keycloak.org/server/db)
#### Explanation
Postgres database is used. Postgres version 15. PV and PVC are used to mount node’s file system to pod, so data is persistent.
#### Test
Connect to the postgres pod and list rows in realm table before and after realm creation 
![image](https://github.com/momchilosavic/keycloak/assets/48445874/a974b710-817c-459b-b795-7d14fc8c0133)
### Create at least one distributed Keycloak cache on the remote infinispan and configure Keycloak to connect to it. Other Keycloak caches can be distributed on the embedded Infinispan.
#### Documentation
[Connect Keycloak with an external Infinispan - Keycloak, Configuring distributed caches - Keycloak, Guide to Infinispan Server](https://www.keycloak.org/high-availability/connect-keycloak-to-external-infinispan)
[Configuring distributed caches - Keycloak](https://www.keycloak.org/server/caching)
[Guide to Infinispan Server](https://infinispan.org/docs/stable/titles/server/server.html)
#### Explanation
Infinispan cache configuration can be found in ConfigMap named infinispan-cache. Sessions cache is created.
Keycloak cache configuration can be found in ConfigMap named keycloak-cache. Distributed cache configuration is added for sessions cache.
#### Test
Connect to the infinispan pod and list keys in sessions cache before and after login to the Keycloak portal
![image](https://github.com/momchilosavic/keycloak/assets/48445874/eaaba79d-6a8b-4f02-9e19-e563c6cd9677)
### Configure authentication for the remote Infinispan caches
#### Documentation
[Using the Infinispan Command Line Interface](https://infinispan.org/docs/stable/titles/cli/cli.html#creating-users_getting-started)
#### Explanation
In the PostStart lifecycle hook of the Infinispan container execute command to create keycloak user in application group when Infinispan is ready. Shell script for this is defined in ConfigMap create-user. 
In the cache configuration for Keycloak set keycloak user for distributed cache authentication.
#### Test
Connect to Infinispan container and check if user is created:
![image](https://github.com/momchilosavic/keycloak/assets/48445874/8cfe757d-a96f-43f6-a1f7-f90ae82664e3)
### Configure custom Infinispan logging (enable debug mode for some components and/or all)
#### Documentation
[Guide to Infinispan Server](https://infinispan.org/docs/stable/titles/server/server.html#configuring-server-logging)
#### Explanation
Based on log4j2.xml file, log level is updated to DEBUG and file is provided to container using ConfigMap infinispan-logs.
#### Test
List logs from Infinispan container

![image](https://github.com/momchilosavic/keycloak/assets/48445874/b1559326-0977-4d97-a2c3-d710706df6d5)
### Add a custom provider https://github.com/aerogear/keycloak-metrics-spi and deploy it to the Keycloak server
#### Documentation
[aerogear/keycloak-metrics-spi: Adds a Metrics Endpoint to Keycloak (github.com)](https://github.com/aerogear/keycloak-metrics-spi?tab=readme-ov-file#on-keycloak-quarkus-distribution)
[Configuring provider - Keycloak](https://www.keycloak.org/server/configuration-provider)
#### Explanation
Jar file is provided to Keycloak container using persistent storage and Init container that is downloading jar file via curl
#### Test
![image](https://github.com/momchilosavic/keycloak/assets/48445874/cc937fdd-ac4a-44be-9c44-0ec479e46558)
### Expose Keycloak for access outside of your cluster
#### Explanation
Kubernetes cluster used for deployment have nginx ingress controller running, so I only created new ingress object that maps “/” route to the Keycloak service. DNS name of AWS NLB that is mapped with ingress nginx controller Load Balancer service is used to access Keycloak portal.
I won’t provide ingress controller manifest as it is cloud provider dependent.
As alternative, service with NodePort could be used. 
#### Test


