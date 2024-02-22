## Introduction
### Diagram
![image](https://github.com/momchilosavic/keycloak/assets/48445874/5cdb3482-7c01-48cd-82a7-b8d9f493a619)
### How to deploy?
Cluster: EKS - Kubernetes v1.26 <br />
Tools needed: helm <br />
Command: helm upgrade --install ./chart keycloak --namespace <namespace> --create-namespace --debug --set <key in chart/values.yaml>=<value> <br />
All values are predefined in values.yaml but they can be overwriten with appropriate values when running helm upgrade <br />
For sake of simplicity, credentials and sensitive data are defined in values.yaml </br >
Also, for sake of simplicty, hostPath is used for persistent storage. </br >
#### ${\color{red}IMPORTANT}$
To make this work with hostPath you have to allow root group to write to the directory on host node </br >
`chmod 775 /tmp/postgres` <br />
`chmod 775 /tmp/infinispan` <br />
`chmod 775 /tmp/keycloak` <br />
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
Connect to the postgres pod and list rows in realm table before and after realm creation <br />
![image](https://github.com/momchilosavic/keycloak/assets/48445874/a974b710-817c-459b-b795-7d14fc8c0133)
### Create at least one distributed Keycloak cache on the remote infinispan and configure Keycloak to connect to it. Other Keycloak caches can be distributed on the embedded Infinispan.
#### Documentation
[Connect Keycloak with an external Infinispan - Keycloak, Configuring distributed caches - Keycloak, Guide to Infinispan Server](https://www.keycloak.org/high-availability/connect-keycloak-to-external-infinispan)<br />
[Configuring distributed caches - Keycloak](https://www.keycloak.org/server/caching)<br />
[Guide to Infinispan Server](https://infinispan.org/docs/stable/titles/server/server.html)
#### Explanation
Infinispan cache configuration can be found in ConfigMap named infinispan-cache. Sessions cache is created.<br />
Keycloak cache configuration can be found in ConfigMap named keycloak-cache. Distributed cache configuration is added for sessions cache.
#### Test
Connect to the infinispan pod and list keys in sessions cache before and after login to the Keycloak portal<br />
![image](https://github.com/momchilosavic/keycloak/assets/48445874/eaaba79d-6a8b-4f02-9e19-e563c6cd9677)
### Configure authentication for the remote Infinispan caches
#### Documentation
[Using the Infinispan Command Line Interface](https://infinispan.org/docs/stable/titles/cli/cli.html#creating-users_getting-started)
#### Explanation
In the PostStart lifecycle hook of the Infinispan container, execute command to create user "keycloak" in group "application" when Infinispan is ready. Shell script for this is defined in ConfigMap create-user. <br />
In the cache configuration for Keycloak set keycloak user for distributed cache authentication.
#### Test
Connect to Infinispan container and check if user is created:<br />
![image](https://github.com/momchilosavic/keycloak/assets/48445874/8cfe757d-a96f-43f6-a1f7-f90ae82664e3)
### Configure custom Infinispan logging (enable debug mode for some components and/or all)
#### Documentation
[Guide to Infinispan Server](https://infinispan.org/docs/stable/titles/server/server.html#configuring-server-logging)
#### Explanation
Based on log4j2.xml file, log level is updated to DEBUG. File is provided to container using ConfigMap "infinispan-logs".
#### Test
List logs from Infinispan container<br />
![image](https://github.com/momchilosavic/keycloak/assets/48445874/3f1925a0-be28-462d-8909-e8b192775ab3)

### Add a custom provider https://github.com/aerogear/keycloak-metrics-spi and deploy it to the Keycloak server
#### Documentation
[aerogear/keycloak-metrics-spi: Adds a Metrics Endpoint to Keycloak (github.com)](https://github.com/aerogear/keycloak-metrics-spi?tab=readme-ov-file#on-keycloak-quarkus-distribution)<br />
[Configuring provider - Keycloak](https://www.keycloak.org/server/configuration-provider)
#### Explanation
Jar file is provided to Keycloak container using persistent storage and Init container that is downloading jar file via curl
#### Test
![image](https://github.com/momchilosavic/keycloak/assets/48445874/cc937fdd-ac4a-44be-9c44-0ec479e46558)
### Expose Keycloak for access outside of your cluster
#### Explanation
Kubernetes cluster used for deployment has nginx ingress controller running, so I only created new ingress object that maps “/” route to the Keycloak service. DNS name of AWS NLB that is mapped with ingress nginx controller Load Balancer service is used to access Keycloak portal.<br />
I won’t provide ingress controller manifest as it is cloud provider dependent.<br />
As alternative, service with NodePort could be used. <br />
To switch between Ingress for ingress controller and NodePort you can set ingress.useIngressController to false. </br >
#### Test


