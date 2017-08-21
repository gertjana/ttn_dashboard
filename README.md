## Setup to consume sensor data from the The Things Network

### requirements:

your data that comes from TTN should be in a json format with all the fields at the 'root' level:
```
{
  "temperature": 24.3,
  "pressure": 1021,
  "timestamp": "2017-08-21T13:54:09.838022211Z"
}
```

### Configuration

consisting of

* InfluxDB - time series db
* Chronograf - admin ui
* Kapacitor - Alerting
* Telegraf - Metrics gathering 

in a bunch of docker containers

First get a telegraf config:
```
docker run --rm telegraf -sample-config > ./telegraf/telegraf.conf
```

in the config look for the `output.influxdb` and change that to 
```
http://influxdb:8086
```
and then look for the `input.mqtt_consumer` section and change it to:
```
[[inputs.mqtt_consumer]]
  servers = ["<region>.thethings.network:1883"]
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

  username = "<ttn-app>"
  password = "<ttn-key>"

  data_format = "json"
```
If you want ssl, enable that section and add the certificates directory to the docker-compose and set the paths accordingly

then you go run
```
docker-compose up -d
```
and access chronograf at `localhost:8888`

