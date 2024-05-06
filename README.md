# Deploy elk on docker compose

#### step1: Start Filebeat, Logstash, Elasticsearch, Kibana, FleetServer

docker compose up --build -d

username: elastic

password: changeme

---

#### step2: Configure logstash

##### 2.1 Consume from rabbitmq

**format message**

{NameService:**"NameService"**, Data: **"Data"**, Err: **"Err"**}

![1714788663380](image/README/1714788663380.png)

---

**revise setting from logstash.conf**

input:

queue => ${**_queueName}_**

addfield => { "queue" => **${queueName}** }

![1714788543932](image/README/1714788543932.png)

output:

[queue] == **${queueName}**

index => "${**queueName}\_**%{**NameService**}%{+YYYY.MM.DD}"

![1714788580366](image/README/1714788580366.png)

---

**example on KIbana**

Menu > Stack Management > Index Management > Data > Index Management

![1714784223452](image/README/1714784223452.png)

##### 2.2 Consume from beat

**format message**

{"level":"**logLevel**","timestamp":"**2024-05-06T03:55:43.944Z**","caller":"**logs/logs.go:46**","msg":"{\"NameService\":\"**NameService**\",\"Data\":" **logMessage**"}

**_PS, See stap to set up apm agent step5_**

---

**revise setting from logstash.conf**

uncomment **beat{...}** to enable receive data from filebeat

![1714967556954](image/README/1714967556954.png)

---

You can custom field to show on Kibana

mutate {

    add_field => {"**customField**" => "**FieldtoMap**"}

}

![1714969136040](image/README/1714969136040.png)

---

set if condition to specific containerName before send data to Kibana

output:

    [type] == "**containerName**"

![1714969183802](image/README/1714969183802.png)

#### step3: Configure on Kibana

##### 3.1 Set data view

Menu > Stack Management > Index Management > Kibana > Data Views > Create data view

![1714785749921](image/README/1714785749921.png)

![1714786219962](image/README/1714786219962.png)

Menu > Stack Management > Index Management > Kibana > Data Views > Create data view

![1714786964779](image/README/1714786964779.png)

##### 3.2 Set retention policy (Delete)

**Create delete policy**

Menu > Stack Management > Index Lifecycle Management > Index Lifecycle Policies > Create policy

![1714787696591](image/README/1714787696591.png)

![1714787863130](image/README/1714787863130.png)

![1714787826426](image/README/1714787826426.png)

**Create index template**

Menu > Management > Dev Tool

![1714788241694](image/README/1714788241694.png)

**Add policy into index template**

![1714788262956](image/README/1714788262956.png)

_PS, copy code in set_policy.txt_

**Check linked index with policy**

![1714788007312](image/README/1714788007312.png)

##### 3.3 Set rule alert

###### 3.3.1 Create connectors

Menu > Stack Management > Rules > Alerts and Insigths > Connectors

![1714789895921](image/README/1714789895921.png)

![1714790053690](image/README/1714790053690.png)

###### 3.3.2 Create rule

Menu > Stack Management > Rules > Alerts and Insigths > Rules > Create rule

![1714789586816](image/README/1714789586816.png)

STACK RULES > Index threshold

![1714789696680](image/README/1714789696680.png)

Actions > Webhook

![1714789730564](image/README/1714789730564.png)

![1714790410843](image/README/1714790410843.png)

#### step4: Configure APM

Menu > Management > Fleet > Setting

![1714790750392](image/README/1714790750392.png)

Click Edit to import finger print and Certificate

![1714790815800](image/README/1714790815800.png)

Correct host for elasticsearch

default: http://localhost:9200 >> https://es01:9200
![1714790854825](image/README/1714790854825.png)

**Advanced YAML configuration**

docker cp es-es01-1:/usr/share/elasticsearch/config/certs/ca/ca.crt ./

ssl:
certificate_authorities:

- |
  ${ca.crt}

**Elasticsearch CA trusted fingerprint (optional)**

cmd : openssl x509 -fingerprint -sha256 -noout -in ./ca.crt | awk -F"=" {' print $2 '} | sed s/://g

![1714791149158](image/README/1714791149158.png)

![1714790969002](image/README/1714790969002.png)

Then service which connect to fleet server will appear

Menu > Observability > APM

![1714791382040](image/README/1714791382040.png)

#### step5: Setup APM agent on golang

set these variable

![1714985097876](image/README/1714985097876.png)

create adapter for apm repo
![1714985422711](image/README/1714985422711.png)

Handling http request/response and capture error

![1714985502936](image/README/1714985502936.png)

#### Uninstall

> docker compose down --remove-orphans --volume
