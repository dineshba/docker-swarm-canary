## Canary Deployment in Docker Swarm

### Steps:

#### Deploy the application that needs the canary deployment

```sh
docker stack deploy -c docker-compose-app-only.yml myapplication
```

Run with proper replicas and update_config
```yaml
deploy:
      replicas: 2
      update_config:
        delay: 60s
        parallelism: 1
```

#### Deploy a utils container which can be used for debugging

```sh
docker stack deploy -c docker-compose-utils.yml utils
```

```sh
# Exec into the container
$(docker ps | grep utils | awk '{print "docker exec -it " $1  " bash" }')

# continously ping the application
while true; do curl myapplication:8123; sleep 1; date; echo ""; done;
```

#### Deploy the infra components that helps the canary.

In our case, we are going to use `envoy proxy` for the dynamic traffic shifting. We are going to use `Envoy-Pilot` which listens to the `consul` and updates the envoy proxies on the fly. So without restart, we can achieve the canary deployment. So this will install a consul (for production we need to run consul in cluster mode) and envoy-pilot

```sh
docker stack deploy -c docker-compose-infra.yml infra
```

#### Inject envoy-proxy before our application so that we can start controlling the traffic

- All the traffic that uses `myapplication` for **dns** will flow through the envoy
- If you want to send the traffic from **interlock** to envoy, remove the labels related to interlock from `myapplication` and add it to `envoy`. Labels such as

```yaml
com.docker.lb.hosts: dns-name-to-register-in-interlock
com.docker.lb.port: port
```

> 10/20 sec downtime during this step
```sh
docker stack deploy -c docker-compose-app-with-envoy.yml myapplication
```

#### Start the version 2 of the application in the new docker stack and change the config in consul, so that envoy-proxy can start splitting the traffic between two versions.

```sh
docker stack deploy -c docker-compose-app-version-two.yml myapplication_v2
```

#### If everything goes well, install version of the application

> 5/10 sec downtime during this step
```sh
docker stack deploy -c docker-compose-app-version-two-only.yml myapplication --prune
```


> Note: `--prune` will clean the no longer referenced service in yaml file. In our case it will remove `consul-import` and `myapplication_behind_envoy`

#### Clean the extra components which are used for canary

```sh
docker stack rm infra
docker stack rm myapplication_v2
docker stack rm utils
```