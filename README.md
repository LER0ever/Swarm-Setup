# Server Manifest
This repo contains configurations for my servers and workstation clustering setup.  
(And now also contains detailed explanations and guide:)

## Features
- Basic Load-Balancing using Reverse Proxy Round-Robin
- Dynamic Reverse Proxy from Docker Flow
- Cluster Visualizer using Docker Swarm Visualizer
- Vitualized subnet for both local dorm computers and remote servers
- High Availability, Service auto-restarting

## Services
- My own git hosting (gitea)
- Continuous Integration (drone-ci)
- LAN Ports forwarding (frp)
- Kanban Task Board (kanboard)
- Status pages for each node (netdata)
- Dev branch of my personal website (caddy, gatsby)
- Shadowsocks
- Transmission Daemon for downloading you-know-what
- Portainer Management
- Mastodon (deleted, too memory consuming)
- More to come

## Screenshots
#### Swarm Visualizer
![Swarm Visualizer](https://i.imgur.com/mCOhH4K.png)
#### Code Hosting
![Gitea](https://i.imgur.com/nQVZpzy.png)
#### NPM Registry
![NPM Registry](https://i.imgur.com/sJTxFkt.png)
#### Kanban (one of the boards)
![Kanboard](https://i.imgur.com/gjRTBMS.png)
#### Drone CI
![drone](https://i.imgur.com/XCcCIib.png)
#### Transmission WebUI
![transmission](https://i.imgur.com/693g9yN.png)


## Technical Details

### 1. Private networking
**This is achieved using Weave Net.**

Installation:  
```bash
sudo curl -L git.io/weave -o /usr/local/bin/weave
sudo chmod a+x /usr/local/bin/weave
```

Before starting the swarm, weave is started in docker and created an overlay
network between hosts both in my dorm and in the cloud, so as to create a 
virtual subnet containing all the nodes.

Then, `weave expose` is executed to not only connects docker containers to that
virtual network, but also brings the physical hosts in, which enables me to later
create a swarm without having to manually expose 2377, 4xxx ports on each
computers which are behind the firewall.

After doing these, my addresses for each node would become for example:

- Main Server: 10.40.0.0
- Singapore Server: 10.44.0.0
- LosAngeles Server: 10.48.0.0
- Dorm Desktop: 10.38.0.0
- Dorm Samsung: 10.34.0.0
- Laptop Somewhere: 10.32.0.1

And they can reach each other using the above addresses.

Additionally, one can use Weave scope to visualize the virtual network and
manage containers in the network.

### 2. Swarm Setup
Once the nodes have their own `10.x.x.x` virtual address, docker swarm mode
can be easily setup by running the following:

##### On the master node:
```bash
eval $(weave env)
docker swarm init --advertise-addr 10.40.0.0:2377
# and you'll get a command for workers to join
```

##### On every worker node:
```bash
eval $(weave env)
docker swarm join --token xxxx --advertise-addr 10.3x.x.x:2377
```

### 3. Docker Stack deploy
The monitor stack and the actual services are seperated so that they won't affect each other.

```bash
# Deploy monitor stack
docker stack deploy -c monitor-stack.yml m_prod_180625

# Deploy services
docker stack deploy -c swarm-stack.yml prod_180625
```

My own stack visualizer is live (or not) at [swarm.rongyi.io](https://swarm.rongyi.io)

## License
The code and configuration in this repo is currently UNLICENSED.
Copyright (c) LER0ever. All Rights Reserved.