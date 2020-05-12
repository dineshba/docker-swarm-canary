This example is using Envoy-Pilot xDS Server (https://github.com/tak2siva/Envoy-Pilot)

This docker compose setup contains:

    - Envoy (port: 10000)
    - App1 (port: 8123)
    - App2 (port: 8126)

To start 
```
docker-compose up
```

## Config Version 1.0

Open consul on localhost:8500 and check the key `xDS/app-cluster/adstest-cluster/CDS/config`
By default the cluster `some_service` will be pointing to `app1:8123` with version `1.0`

`xDS/app-cluster/adstest-cluster/CDS/config`

```
[
		{
			"name": "some_service",
			"connect_timeout": "0.25s",
			"type": "STRICT_DNS",
			"lb_policy": "ROUND_ROBIN",
			"hosts" : [ { "socket_address" : { "address" : "app1", "port_value" : 8123 }}]
		}
]
```

`xDS/app-cluster/adstest-cluster/CDS/version`

```
1.0
```


Verify that by 

```
curl http://localhost:10000
this is app --ONE--
```

## Config Version 2.0 (Dynamic Updates)

Now change the `some_service` hosts to point `app2:8126` with version `2.0` (i.e)

`xDS/app-cluster/adstest-cluster/CDS/config`

```
[
		{
			"name": "some_service",
			"connect_timeout": "0.25s",
			"type": "STRICT_DNS",
			"lb_policy": "ROUND_ROBIN",
			"hosts" : [ { "socket_address" : { "address" : "app2", "port_value" : 8126 }}]
		}
]
```

`xDS/app-cluster/adstest-cluster/CDS/version`

```
2.0
```


Verify that by 

```
curl http://localhost:10000
this is app --TWO--
```