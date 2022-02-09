---
tags: Large Systems
---
:::success
# LS Lab 2 - Container Orchestration & Load-balancing
Name: Ivan Okhotnikov
:::


## Task 1 - Choose Container Engine & Orchestration

:::warning
1. Choose a container engine. Some suggestions:
Deploy a true container cluster farm, across several team machines. It is however recommended to proceed in a virtual machine environments so the worker nodes can have the exact same system patch level (which makes it easier).
Bonus: if you choose Docker, play with alternate storage drivers e.g. BTRFS or ZFS instead of OverlayFS.
2. Choose an orchestration engine then:
3. Analyze and describe the best practices of usage the chosen container engine.
:::


## Implementation:

:::info
> 1. Choose a container engine. Some suggestions:
Deploy a true container cluster farm, across several team machines. It is however recommended to proceed in a virtual machine environments so the worker nodes can have the exact same system patch level (which makes it easier).
> 2. Choose an orchestration engine then:

I will choose Docker + Docker Swarm

<center>

![](https://i.imgur.com/gCCFsJd.png)
Figure 1: My topology
</center>

To begin with, I will specify my hostname for each virtual machine (this will help me in the future to see the difference between nodes in the cluster)
```bash=0
sudo sh -c 'echo "worker1" > /etc/hostname'
```
Then I will need to install docker on each machine
```bash=1
sudo apt update
sudo apt install -y docker.io
```

For my manager node, I will execute an initialization command that will help me connect the workers
```bash=3
docker swarm init
```

<center>

![](https://i.imgur.com/HEWCpvv.png)
Figure 2: Swarm init command
</center>

In the output of the command, I have instructions for connecting my workers

```bash=4
docker swarm join --token SWMTKN-1-4oxppmp2gc6487mux1pzq91oy4axxn5pl9428terj69tqijek1-0i421166m8clpll01a6u31vpz 192.168.122.33:2377
```


After connecting to all nodes, I can view the list of nodes connected to my manager, their status and hostname (figure 3)
</center>

```bash=5
sudo docker node ls
```


<center>

![](https://i.imgur.com/kek01iq.png)
Figure 3: Swarm node list
</center>

> 3. Analyze and describe the best practices of usage the chosen container engine.

1 - Make the number of layers in the image as small as possible, which will reduce the weight of the image (the fewer RUN commands the better)
2 - It is better to store images in an image storage system (for example docker hub), this will allow you to quickly download images on other devices and minimizes the number of images stored on the device
3 - Do not store encryption keys, tls certificates and other valuable data inside the container. It is better to use environment variables for this
4 - Use images that have been signed by the development and tested by the quality department. This way it will be possible to avoid incidents with fake images

For docker swarm:
Since management takes place with the help of a manager node, it is necessary to have several such so that in the event of a failure of one, it can simply be replaced
Docker svorm already has a system for assembling logs for services that are deployed on several nodes, this is very useful and simplifies reading and analyzing logs from all nodes

:::

## Task 2 - Distributing an Application/Microservices
:::warning
Base level (it means that this task will be evaluated very meticulously): deploy at least a simple application (e.g. a simple web page showing the hostname of the host node it is running upon) and validate that its instances are spreading across the farm. It is literally not necessary to create
your own service/application: you can use something from a public repository, for example, from DockerHub. However, no one forbids you to work with self-written projects.
Hint: creating such application is particularly easy to achieve with Swarm when the nodes does not
share storage.

Semi-Bonus 1: deploy a microservices instead of standalone application, e.g. full stack web application with web server, database...
Bonus 2: use Veewee/Vargant to build and distribute your development environment.
:::

## Implementation:

:::warning
References:
https://docs.docker.com/engine/swarm/stack-deploy/
https://blog.gelin.ru/2017/04/docker-swarm-mode.html
:::

:::info


To deploy my service, I used the create service command and set the name and image name for it
```bash=1
sudo docker service create --name test nginxdemos/hello
```

<center>

![](https://i.imgur.com/FVCr9ts.png)
Figure 4: Deploy service
</center>


After that I can open a web page and get the results (figure 5)
</center>

<center>

![](https://i.imgur.com/IJ5tnGK.png)
Figure 5: A web page from a deployed service
</center>

> Semi-Bonus 1: deploy a microservices instead of standalone application, e.g. full stack web application with web server, database...


I will deploy microservices for the most popular task - LAMP (linux, Apache2, Mysql, PHP)

For this task I created [github repository](https://github.com/IvanHunters/swarm-lamp-application), which contains the docker-compose file itself, the nginx configuration file and the working folder for php-fpm

The docker-compose file itself contains not only standard instructions (deploying the network, mounting partitions and specifying images), but also instructions for deploying a particular service.

```bash=1
        deploy:
          replicas: 2
          restart_policy:
            condition: on-failure
          placement:
            constraints:
              - node.role == worker
```

```bash=1
placement:
          constraints:
            - node.labels.db == true
```

How many replicas are needed, on which replicas to run (constraints)
I have an indication of either the label or the role of the worker
To create a label for a worker, simply run the command

```bash=1
docker node update --label-add db=true worker3
```


```bash=1
  restart_policy:
          condition: on-failure
```
What to do if the service crashes (I indicated restart)

For this part of the lab work, I created my own php-fpm image. It is based on the official php-fpm7.2 image, but only with my minor modifications

I also specified the paths for mounting folders inside containers (IT is EXTREMELY IMPORTANT that the specified folders are specified)

The contents of the docker-compose file can be found in the repository [here](https://github.com/IvanHunters/swarm-lamp-application/blob/main/docker-compose.yml)


After all the necessary files and folders have been placed on the working nodes and the label is specified, you can proceed to deploying microservices:

```bash=1
docker stack deploy -c docker-compose.yml test2
```

To show the process more clearly, I used the vizualizer image (figure 6) 
</center>

<center>

![](https://i.imgur.com/39zCdz0.png)
Figure 6: Vizualizer web interface
</center>


It's time to move on to the tests
Since I have specified different scripts for each php-fpm node, I can observe the work of the load balancer (the name of the node on the web page is constantly changing)
<center>

![](https://i.imgur.com/rRMTTgC.png)
Figure 7: Php script output
<center>
    
</center>
    
![](https://i.imgur.com/fQyc2L8.png)
Figure 8: Php script output
</center>


Since in the NGINX configuration I specified to work with the php domain name, all balancing takes place at the dns level between the services that have deployed for PHP

```nginx=1
 location / {
        try_files $uri $uri/ /index.php$is_args$query_string;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
```

When I turn off one of the nodes, one of the nodes will take the entire load and only 1 worker will be shown and the second replica will be deployed for only one node

<center>

![](https://i.imgur.com/E8OJIES.png)
Figure 9: Vizualizer Output
<center>
    
</center>
    
![](https://i.imgur.com/ZKRFEre.png)
Figure 10: Php script output
</center>


<!-- > Bonus 2: use Veewee/Vargant to build and distribute your development environment. -->

<!-- 
> Bonus 3: if you use k8s, prepare a helm chart for your application. -->



:::



## Task 3 - Validation:
:::warning
Validate that when a node goes down a new instance is launched. Show how the redistribution of the instances can happen when the broken node comes back alive. Describe all steps of tests in your report in details.
:::

## Implementation:
:::info

Since by default my `test` service with an nginx replica creates only 1 replica - I will need to update it with an indication of the number of nodes (two)

```bash=1
docker service scale test=2
```

<center>

![](https://i.imgur.com/2nYUofY.png)
Figure 11: Initial service distribution scheme
</center>

Now when I turn off the worker2 node, the service will still work on the master worker, which ensures fault tolerance

After turning off one of the nodes, the second replica automatically deployed to another, accessible node

<center>

![](https://i.imgur.com/NsT80ER.png)
Figure 12: The result of the distribution of the service
</center>

Now when I turn on the worker2 node, one will become a spare and the service will not start on it immediately


I will consider another example (I need 3 replicas)

<center>

![](https://i.imgur.com/lS21cYu.png)
Figure 11: Initial service distribution scheme
</center>

And now I will turn off 1 of the nodes

Since the number of service replicas should remain unchanged, one of the nodes will take 2 replicas

<center>

![](https://i.imgur.com/4vta0wQ.png)
Figure 12: The resulting service distribution
</center>

And even when we turn on the third node, the processes will not return to the original distribution (because for this you need to introduce a resource limit for each node)

If you specify the workload of a node equal to the work of one service, it will not take the second process.

But this can be handled manually without restarting all nodes
1 - specify a smaller scale for the service, and then return to the original state
2 - execute `docker service update {service name} --force`


<center>

![](https://i.imgur.com/bl5V7oj.png)
Figure 13: The result of manually redistributing the service

</center>
:::



## Task 4 - Load-balancing
:::warning
1. Choose a load-balancing facility, which will be installed and tested against your orchestrator.
You can choose either l3 or l7 solutions. The disadvantage of layer-3 is that web sessions may break against statefull applications. There is a trick to deal with that, though, for example with OpenBSD Packet Filter.
Note: for K8S, you can try to do Ingress.

Choose a load-balancer to distribute the load to the worker nodes, for example either layer 3:
* layer-3 BSD pf / npf / ipfilter/ipnat
* layer-3 Linux Netfilter ( iptables )
* layer 3 Linux nftables ( nft )
* layer-3+7 HAProxy
or HTTP reverse-proxy:
* layer 7 NGINX / NGINX Plus (dynamic objects?)
* layer 7 Apache Traffic Server? (static objects?)
* layer 7 OpenBSD Relayd
* K8S Ingress / Ingress-NGINX

2. Swarm or K8S’s network overlay is already spreading the requests among the farm, right? So why would a load-balancer still be needed? Explain and show briefly how a real-life network architecture look like with a small diagram.
:::

## Implementation:
:::warning
References:
https://www.nginx.com/blog/docker-swarm-load-balancing-nginx-plus/#swarm-demo
:::

:::info
> 1. Choose a load-balancing facility, which will be installed and tested against your orchestrator.

I will configure based on my microservice
To do this, I will need to change the nginx configuration file

by adding a distribution network

```nginx=1
upstream backend {
   server 192.168.122.8:9000;
   server 192.168.122.65:9000;
}
```

Then I specified this network when using `fastcgi_pass`

```nginx=1
    location / {
        try_files $uri $uri/ /index.php$is_args$query_string;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass backend;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
```

I also organized a port forwarding for the php `9000:9000` service so that I could connect to it from the outside and organize balancing

After making the changes, I updated the microservice

```bash=1
docker stack deploy -c docker-compose.yml test2
```

As a result, the web page opens differently. Previously, every reboot there was a change in the content, and now every two

<center>

![](https://i.imgur.com/sVQQoFA.png)
Figure 14: Working page
</center>

> 2. Swarm or K8S’s network overlay is already spreading the requests among the farm, right? So why would a load-balancer still be needed? Explain and show briefly how a real-life network architecture look like with a small diagram.

The Swarm load balancer is initially a basic Level 4 load balancer.
Since many applications require additional features such as:

SSL/TLS termination
Content-based routing
Access control and authorization

The best solution for this is to use a third-party load balancer

:::

## Bonus - Autoscaling
:::warning
1. How to scale instances in the Docker Swarm or another facility? Could it be done automatically?
:::

## Implementation
:::info
> 1. How to scale instances in the Docker Swarm or another facility? Could it be done automatically?

It can be done
One solution is to use the grobal flag to describe the number of replicas to run in the docker-compose file
This will allow you to run the service on all available nodes (without specifying the available number)
In combination with specifying roles, labels (as I did before), you can achieve automatic scaling of the application
And if you also use resource limits for each node, then this will help even more to distribute the load between all nodes
:::


## Bonus - Updates & Monitoring (theoretical)
:::warning
1. Suppose your application must be updated. Describe the process of applying image or instance updates with your orchestrators of choice, and do it with a minor change in your sample application. In other words, how to ship the containers depending on your engine and orchestrator?
2. It is always a good practice to do monitoring and logging. How can this be done with your orchestrators of choice? Are there built-in tools or must you use third-party ones?
:::

## Implementation
:::info
> 1. Suppose your application must be updated. Describe the process of applying image or instance updates with your orchestrators of choice, and do it with a minor change in your sample application. In other words, how to ship the containers depending on your engine and orchestrator?

Docker Swarm already has a built-in service update feature
For example, when using the command
```bash=1
docker service update [OPTION] serviceName
```

To display all available ones, you can use the command:

```bash=1
docker service update -h
```

Using the update command, you can make many detailed changes inside the service, including the delay of updates between nodes
This is useful because if errors occur during the upgrade process that prevent the container from starting, the upgrade process will be completed and other nodes with working containers will not be affected. This provides the system with greater fault tolerance.
There is also the possibility of rolling back so that the node returns to its previous state, which was more stable.


> 2. It is always a good practice to do monitoring and logging. How can this be done with your orchestrators of choice? Are there built-in tools or must you use third-party ones?

In docker swarm, you can monitor the status of nodes out of the box. You can also view logs for each running service. But sometimes I want more information

To do this, you can use the following solutions:
- prometheus.io
- dynatrace.com
- grafana.com
- cAdvisor

But most often, in order to achieve greater informativeness, solutions are combined:
For example Prometheus + Grafana
:::

