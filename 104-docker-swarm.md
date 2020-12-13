# Docker Swarm 104

Laboratory:
```
moss         10.5.0.11
gilfoyle     10.5.0.12
```

```
[root@moss ~]# docker swarm  --help

Usage:	docker swarm COMMAND

Manage Swarm

Commands:
  ca          Display and rotate the root CA
  init        Initialize a swarm
  join        Join a swarm as a node and/or manager
  join-token  Manage join tokens
  leave       Leave the swarm
  unlock      Unlock swarm
  unlock-key  Manage the unlock key
  update      Update the swarm

Run 'docker swarm COMMAND --help' for more information on a command.
```


Connect to **moss** and initialize swarm and start listening on a specific IP address (useful in case you have multiple networks)
```bash
[root@moss ~]# docker swarm init --advertise-addr 10.5.0.11
Swarm initialized: current node (ywdmx28ljh63khgkipaoffdrk) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-63tpy1vff295o7eenf0pm3dhh0t2evd8cvhrtj9rm3yi0adegw-e3cqag2k2s5mx212847v6iexp 10.5.0.11:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

```

Now add **gilfoyle** as worker to our cluster
```bash
[root@gilfoyle ~]# docker swarm join --token SWMTKN-1-3n0m18cbka5xqms1qeumvleihz7626jk312meset08igngzkmz-1lb76vp1gea9r9wb20bq5sqni 10.5.0.11:2377
This node joined a swarm as a worker.
```

Now from **moss** we can see our cluster
```bash
[root@moss ~]#  docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
ywdmx28ljh63khgkipaoffdrk *   moss                Ready               Active              Leader              19.03.14
[root@moss ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
ed6ec4236263        bridge              bridge              local
e177687c512a        docker_gwbridge     bridge              local
049ee3b70018        host                host                local
jyqxvpi40dhy        ingress             overlay             swarm
af669e346b38        none                null                local
```

The following two commands will tell you how to add a new worker or manager to the cluster.
```bash
[root@moss ~]# docker swarm  join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3n0m18cbka5xqms1qeumvleihz7626jk312meset08igngzkmz-1lb76vp1gea9r9wb20bq5sqni 10.5.0.11:2377

[root@moss ~]# docker swarm  join-token manager 
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3n0m18cbka5xqms1qeumvleihz7626jk312meset08igngzkmz-am8qhncp3crg999tqy8fq27wt 10.5.0.11:2377
```

