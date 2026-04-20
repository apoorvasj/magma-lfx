<h1> About Magma </h1>
Magma is an open-source software platform that gives network operators a better way to manage connections, subscribers and services. Cellular networks can be difficult to setup and expensive to maintain but Magma aims to change this by helping companies deploy cellular networks. It is a converged core supporting lte, 5g and wifi. It has 3 main components
<ol>
<li> Orchestrator
<li> NMS or Network Management System
<li> Access Gateway (AGW)
</ol> 
You can learn more about the components <a href="https://www.youtube.com/watch?v=59U5mL6saXs"> here </a>. <br>
Keep in mind, orchestrator and nms can be setup on the same machine whereas the AGW has to be setup on a separate machine/VM. First, we will walk through setting up NMS + orchestrator :)
Before getting started, these were some resources I referred to- a mix of official docs and past contributor's guides. (as of Feb 2026, the official docs could be outdated and the docker based deployment is recommended to be the easiest one).
<ol>
<li> <a href= "https://magma.github.io/magma/docs/lte/deploy_install_docker"> Official docs </a>
<li> <a href= "https://magma-installation-docs-laniw.surge.sh/">Lani's guide </a>
<li> Kidus' guide
</ol>
This guide uses <b> DOCKER-BASED </b> deployment.<br><br>
<h1> Prerequisites for the setup </h1>
Some important points to note-
<ol>
<li> A windows OS is currently not supported as the developing enviroment due to some linux only tools during setup. However, there are workarounds such as WSL, dual booting your system or a VM. Personally I would prefer a VM (as my AGW system did not have much RAM) and there are tools such as <a href = "https://www.virtualbox.org/">VirtualBox</a> that help you setup a VM quite easily! Your choice should be based on the compute and storage power of your system.
<li> It is recommended to keep nms+orchestrator on one system and agw on a separate one. This can even be done with 2 separate Linux VMs.
<li> For my setup, my orchestrator and nms used macOS whereas I had AGW running on a Linux VM.
<li> Go ahead and install <a href = "docker.com">docker</a> and docker compose.
<li> Ensure you have git.
</ol>

<h1> Getting started with orchestrator + NMS </h1>
<ol>
<li> Clone the Magma repository. 
<code> git clone https://github.com/magma/magma.git <br>
cd magma</code>
<li> Build the orchestrator docker containers. Navigate to the orc8r directory on your machine. <code> cd orc8r/cloud/docker </code>. Once you're in that path, run <code> ./build.py --all </code> This builds all docker images for the orchestrator.
<li> Once the orchestrator build is completed, we can start the orchestrator cloud using <code> ./run.py </code><br><br>
<blockquote> At this point I encountered an error, so let us dive into how I debugged it!
Error in running orchestrator step i.e ./run.py
 failed to solve: process "/bin/sh -c gem install     elasticsearch:7.13.0     fluent-plugin-elasticsearch:5.2.1     fluent-plugin-multi-format-parser:1.0.0     --no-document" did not complete successfully: exit code: 1


--------------------

  13 |     USER root

  14 | >>> RUN gem install \

  15 | >>>     elasticsearch:7.13.0 \

  16 | >>>     fluent-plugin-elasticsearch:5.2.1 \

  17 | >>>     fluent-plugin-multi-format-parser:1.0.0 \

  18 | >>>     --no-document

  19 |     USER fluent

--------------------

target fluentd: failed to solve: process "/bin/sh -c gem install     elasticsearch:7.13.0     fluent-plugin-elasticsearch:5.2.1     fluent-plugin-multi-format-parser:1.0.0     --no-document" did not complete successfully: exit code: 1<br><br>
In my case, there were two incompatible gems, excon and multi-json (incompatible with ruby 2.7). So I had to pin both their versions. A quick search in vscode revealed two files <code> orc8r/cloud/docker/fluentd_forward/dockerfile </code> and <code> orc8r/cloud/docker/fluentd/dockerfile </code> contained these lines: 
<img src = "images/fluentd_old.png">
Changing this to: <br>
<img src = "images/fluentd_new.png"> resolved the error.
 </blockquote>
Once the orchestrator is successfully built and you run ./run.py, you should see something like this:

<img src = "images/orc8r_built.png">
<img src = "images/orc8r_run.png">
</ol>
Yay! the orchestrator has been setup.<br><br>
<h1> Setting up NMS </h1>

1. Set the Magma root directory 

```bash id="u7v8w9"
export MAGMA_ROOT=${PWD}
```

2. Move to the NMS directory and build NMS services:

```bash id="a4b5c6"
cd ${MAGMA_ROOT}/nms
docker-compose build
``` 
<br>
<img src = "images/nms containers.png">

3. After this, you will be able to access the UI by visiting https://magma-test.localhost, and using the email admin@magma.test and password password1234

<br> 

<img src = "images/nms ui.png">

4. In case you experience 502 bad gateway error, wait for the container to start. However if it persists check the health of your container using docker ps.

<img src = "images/502 Bad Gateway.png">
<img src = "images/unhealthy container.png">

5. In this case, kill the container using <code> docker kill *container id* </code> and start it again using <code> docker start *container id* </code>

6. Open the networks menu and register your first network.

<img src = "images/nms networks page.png">


<br>
<h1> Setting up AGW </h1>
I installed the access gateway through an Ubuntu 20.04 VM. The software I used for this was VirtualBox. One of the most important things is while creating VM, I set the <b> networking configuration to bridge mode with 2 network interfaces. </b> (Make sure to do this as I encountered errors due to initially only setting up 1 network interface!) <br><br> 

<h2> Setting up AGW VM </h2>
<li>This is the configuration I used.
<img src = "images/agw config.png">
<li> Ensure VM can reach the orchestrator machine. For this you can use <code> ping (IP of orhcestrator machine) </code>.
<li>In a new terminal window run <code>sudo apt update && sudo apt upgrade -y</code>
<li> Run <code>ssudo apt install -y openssh-server curl wget git</code>

<h2> Install docker and docker Compose </h2>
These are the steps I followed from Lani's blog the link to which has been provided above.


1. Update the package list and install Docker:

```bash id="a1b2c3"
sudo apt update
sudo apt install -y docker.io
```

2. Install Docker Compose:

```bash id="d4e5f6"
sudo curl -SL https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

3. Create the Docker group and add the current user:

```bash id="g7h8i9"
sudo groupadd docker
sudo gpasswd -a ${USER} docker
```

4. Restart Docker service and verify installation:

```bash id="j1k2l3"
sudo service docker restart
docker --version
docker-compose --version
```
## Magma AGW Docker Installation

1. Switch to root user and create the certificates directory:

```bash id="m1n2o3"
sudo -i
mkdir -p /var/opt/magma/certs
```

2. Copy the Orchestrator rootCA certificate:

```bash id="p4q5r6"
cat > /var/opt/magma/certs/rootCA.pem << EOF
-----BEGIN CERTIFICATE-----
...certificate contents...
-----END CERTIFICATE-----
EOF
```

**Note:** This is a very important step. The certificate contents must be obtained from the Magma folder on the Orchestrator machine. A quick search in VSCode helped locate the correct `rootCA.pem` certificate file easily within cache directory.

3. Download and run the Docker installation script:

```bash id="s7t8u9"
wget https://github.com/magma/magma/raw/v1.9/lte/gateway/deploy/agw_install_docker.sh
bash agw_install_docker.sh
```

4. Verify the installation:

```bash id="v1w2x3"
cd /var/opt/magma/docker
sudo docker-compose ps
```
<img src = "images/agw containers.png">
Check that the services health shows as healthy. If any containers are unhealthy, try to kill and restart, if that doesn't work you will have to check logs and debug further.

## AGW Configuration and Registration

1. Create the `control_proxy.yml` configuration file:

```bash id="y4z5a6"
cat << EOF | sudo tee /var/opt/magma/configs/control_proxy.yml
cloud_address: controller.magma.test
cloud_port: 7443
bootstrap_address: bootstrapper-controller.magma.test
bootstrap_port: 7444
fluentd_address: fluentd.magma.test
fluentd_port: 24224
rootca_cert: /var/opt/magma/certs/rootCA.pem
EOF
```
I was able to find these port numbers after checking the mapping of containers on my orchestrator machine using <code> docker ps </code>

2. Update the `/etc/hosts` file:

```bash id="b7c8d9"
sudo nano /etc/hosts
```

Add the following entries by replacing `<ORCHESTRATOR_IP>` with the actual IP address of the Orchestrator machine:

```text
<ORCHESTRATOR_IP> controller.magma.test
<ORCHESTRATOR_IP> bootstrapper-controller.magma.test
<ORCHESTRATOR_IP> fluentd.magma.test
```

This is an important step because a common error occurs due to incorrect host file configuration.

3. Start AGW services:

```bash id="e1f2g3"
sudo docker-compose up -d
```

4. Retrieve the Hardware ID and Challenge Key:

```bash id="h4i5j6"
sudo docker-compose exec magmad show_gateway_info.py
```

**Note:** In the NMS UI, you are supposed to register the AGW using the hardware ID and challenge key displayed above.

After the gateway successfully checks in, this is what the equipments page will look like.

<img src = "images/gateway checkin.png">

<h1> Debugging AGW setup </h1>
This is the official debug docs which were quite helpful to me:
<a href = "https://magma.github.io/magma/docs/howtos/troubleshooting/agw_unable_to_checkin"> Debug AGW issues </a>.
I ran into a few erros while trying to complete the AGW setup so let us dive into how they were debugged:

1. The most important factor is to set two network interfaces in your VM settings.
2. AGW unable to checkin to NMS


If the AGW is not able to check in to the Orchestrator, the issue can be diagnosed using the `checkin_cli.py` script.


### Resolution

Run the check-in diagnostic script inside the `magmad` container using the full Docker Compose path:

```bash id="n1o2p3"
cd /var/opt/magma/docker
sudo docker compose exec magmad checkin_cli.py
```

This script helps diagnose AGW and Orchestrator communication issues. If the test fails, it usually suggests the potential root cause and recommended fixes.

A successful output will look similar to this:

```text id="q4r5s6"
1. -- Testing TCP connection to controller.magma.test:443 --
2. -- Testing Certificate --
3. -- Testing SSL --
4. -- Creating direct cloud checkin --
5. -- Creating proxy cloud checkin --
Success!
```
Sometimes, I'd have to run this checkin_cli.py more than once until the first checkin happened.

If the output is not successful, follow the recommended steps shown by the script. If the issue still persists after applying those fixes, proceed with further debugging steps.

3. Check the ports in control_proxy.yml! The port numbers cannot be set arbitrarily and had to match with the ones showing up in my docker ps command in orchestrator machine. When the ports didn't match, my AGW health was bad. Docker displays <code>HOST_PORT : CONTAINER_PORT </code> and the ports you configure in the control_proxy need to match HOST PORT.

4. Wheneve my orchestrator machine went into standby/sleep mode the AGW would stop checking in. So ensure this is not the case.