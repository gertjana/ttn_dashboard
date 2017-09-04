## Setup to consume sensor data from the The Things Network

consisting of

* Telegraf - Metrics gathering
* InfluxDB - time series db
* Chronograf - admin ui for influx
* Kapacitor - Alerting framework
* Grafana - Dashboards

in a bunch of docker containers

### requirements:

your data that comes from TTN should be in a json format for instance:
```
{
  "temperature": 24.3,
  "pressure": 1021,
  "timestamp": "2017-08-21T13:54:09.838022211Z"
}
```

### Configuration
we will connect to the MQTT broker of TTN to get our sensor data

First get a telegraf config:

from the root of this application run:
```
docker run --rm telegraf -sample-config > ./telegraf/telegraf.conf
```
in the telegraf.conf look for the `outputs.influxdb` section and change the url to 
```
http://influxdb:8086
```
and then look for the `inputs.mqtt_consumer` section and change it to:

filling in the `<ttn-region>`, `<ttn-application-eui>` and `<ttn-acces-key>` from the TTN application you want to connect to 

```
[[inputs.mqtt_consumer]]
  servers = ["<ttn-region>.thethings.network:1883"]
  qos = 0

  topics = [
    "+/devices/+/up"
  ]
  ## Optional SSL Config
  # ssl_ca = "/etc/telegraf/ca.pem"
  # ssl_cert = "/etc/telegraf/cert.pem"
  # ssl_key = "/etc/telegraf/key.pem"
  ## Use SSL but skip chain & host verification
  # insecure_skip_verify = false

  persistent_session = false
  client_id = ""

  username = "<ttn-application-eui>"
  password = "<ttn-access-key>"

  data_format = "json"
```
If you want ssl, enable that section and add the certificates directory to the docker-compose and set the paths accordingly

then you go run
```
DOMAIN=your.domain docker-compose up -d
```
and access chronograf at `chronograf.your.domain:8888`, 
connect to influxdb:8086

In the data explorer, you should find your data under:
```
telegraf.autogen > mqtt_consumer > topics - 1
```

or in grafana at `grafana.your.domain:3000` 
also add an influxdb datasource at `influxdb:8086`
