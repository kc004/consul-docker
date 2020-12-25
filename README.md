# consul-docker

### Deployment
First let's pull consul image locally.
```
$ docker pull consul
```

Now, we will run consul server and client.
```
$ docker run -d -p 8500:8500 -p 8600:8600/udp --name=consul-server consul agent -server -ui -node=server-1 -bootstrap-expect=1 -client=0.0.0.0
```
Now we need IP address of consul-server
```
$ docker exec consul-server consul members
```
Note the ip address of consul server and let's join the client into the cluster
```
$ docker run --name=consul-client consul agent -node=client-1 -join=172.17.0.2 {{IP Adress of server}}
```
In a new terminal,
You can see new client has joined by executing below command
```
$  docker exec consul-server consul members
```

Now, we need to register our services with consul
```
docker exec consul-client /bin/sh -c "echo '{\"service\": {\"name\": \"web1\", \"tags\": [\"web1\"], \"port\": 5000}}' >> /consul/config/web1.json"
```
```
docker exec consul-client /bin/sh -c "echo '{\"service\": {\"name\": \"web2\", \"tags\": [\"web2\"], \"port\": 5001}}' >> /consul/config/web2.json"
```
```
docker exec consul-client /bin/sh -c "echo '{\"service\": {\"name\": \"redis\", \"tags\": [\"redis\"], \"port\": 6379}}' >> /consul/config/redis.json"
```

After adding all serivce json file in /consul/config, we need to reload to detect changes
```
$ docker exec consul-client consul reload
```

If we go back to consul-client terminal we can see synced all services
```
[INFO]  agent: Synced service: service=redis
[INFO]  agent: Synced service: service=web1
[INFO]  agent: Synced service: service=web2
```

We can query consul for the location of our serivce
```
$ dig @127.0.0.1 -p 8600 web1.service.consul

$ dig @127.0.0.1 -p 8600 web2.service.consul

$ dig @127.0.0.1 -p 8600 redis.service.consul
```
