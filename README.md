# Technical Assignment for Aiven 

## Scenario
A customer has approached us with the following requirements: they would like a system that was able to take 
multiple inputs from different backend systems and serve to various internal and external services. 

They need insight into how the system is operating at all times and some of the data needs to be modified in transit. 

One of the destination systems is a database that serves their mobile and web app. Currently, complex queries can 
take 20 seconds to complete and the requested data is updated once a day. 

All of their systems are hosted in networks that are not visible on the Public Internet and split between 
Google Cloud Platform and Microsoft Azure. Their peak data production is 3MB/s with 2MB/s read. 
They only need to hold data for 7 days.

- What services would you recommend for this customer? 
- How would you meet their security requirements? 
- How would you determine the cost (size) of the plan(s) they would need?
 
## Solution
 - [Slides](slides.pdf)
 - [Infrastructure setup script](#infrastructure-setup-script)

### Infrastructure setup script
Note: due to the time constraints, I have decided to go with Aiven CLI, which is in fact quite usable.

- Install Aiven CLI
```shell script
python3 -m pip install aiven-client
```

- Create services and integrations
```shell script
# Authenticate CLI
avn user login <you@example.com>

# (optional) Check defaults of current/default project
avn project details

# (optional) Update cloud location to your preferences
avn project update --cloud google-europe-north1

# (optional) List service types
avn service types

# (optional) List plans for relevant service types in the selected cloud
avn service plans --cloud google-europe-north1 | grep -E "kafka:|kafka_connect:|influx:|grafana:"

# Create service: Kafka
avn service create kafka1 -t kafka -p business-4 --no-project-vpc

# Create service: Kafka Connect
avn service create kafkaconnect1 -t kafka_connect -p business-4 --no-project-vpc

# Create service: InfluxDB
avn service create influx1 -t influxdb -p startup-4 --no-project-vpc

#Create service: Grafana
avn service create grafana1 -t grafana -p startup-1 --no-project-vpc

# Create integration: Kafka -> Kafka Connect
avn service integration-create -t kafka_connect -s kafka1 -d kafkaconnect1 

# Create integration: Kafka -> InfluxDB
avn service integration-create -t metrics -s kafka1 -d influx1

# Create integration: InfluxDB -> Grafana
avn service integration-create -t dashboard -s grafana1 -d influx1
```
- Access Grafana dashboard for monitoring
  * go to your project in https://console.aiven.io/, find `grafana1` service and on `Overview` tab you'll find URL, user and password
  * `influx1` is already set as a datasource
  * create dashboards as needed
  
- Remember to terminate services not to incur additional costs
```shell script
# Terminate services:
avn service terminate kafka1 && avn service terminate kafkaconnect1 && avn service terminate influx1 && avn service terminate grafana1
```