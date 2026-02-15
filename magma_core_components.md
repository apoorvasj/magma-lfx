<h1> Description of key Components in Magma's Architecture </h1>
Before diving into the architecture, there are some terms to understand.
<h3> Reverse Proxy </h3>
A reverse proxy acts as a load-balancer for user traffic that comes to a server. It also performs other key functions such as authentication, rate limits and routing requests to the correct endpoint. Nginx is one such  reverse proxy.

<h2> NMS </h2>
NMS or Network Management System provides a centralised UI to add and remove equipment, networks, subscribers and view related metrics. 
<h3> Tech Stack of NMS </h3>
React components (Material UI), hooks 

<h2> Orchestrator </h2>
<img src="images\orc8r_diagram.png">
<h3> Security & Identity </h3> 
<ul>
<li><h4>accessd </h4>
Checks operator identity objects and if admin has read/write permissions for a specific part of the network.
<li><h4>bootstrapper </h4>
Handles the "handshake" when a new hardware gateway joins the network for the first time, giving it the necessary security certificates.
<li><h4>certifier </h4>
Keeps track of all active certificates and ensures they are valid.
<li><h4>obsidian </h4>
Verifies API request access and reverse proxies the request to orc8r services.
</ul>
<h3> Configuration and Metadata </h3> 
<ul>
<li><h4>configurator </h4>
maintains configurations and metadata for networks
<li><h4>device </h4>
maintains configs and metadata for devices in the network (gateways)
<li><h4>orchestrator </h4>
provides mconfigs for configurations of core gateway services (such as magmad), metrics and crud API 
<li><h4>tenants </h4>
crud interface to manage nms tenants
</ul>
<h3>Subscriber and State Management </h3> 
<ul>
<li><h4>directoryd </h4>
stores sucbscriber identity (IMSI, IP, MAC Address) as well as location (gateway hardware ID)
<li><h4>state </h4>
maintains reported state for devices in the network
<li><h4>streamer </h4>
fetches updates for various data streams (mconfig, subscibers) from the appropriate orchestrator service.</ul>
<h3> Communication and Discovery </h3> 
<ul>
<li><h4>dispatcher</h4>
maintains the constant live connection (SyncRPC) between cloud and gateways so they can talk in both directions.
<li><h4>service_registry</h4>
Helps services find each other in the orchestrator since in a Kubernetes cluster, IPs change.
</ul>
<h3> Observability and Data </h3> 
<ul>
<li><h4>metricsd</h4>
collects runtime metrics from gateways and orchestrator services.
<li><h4>ctraced</h4>
handles gateway call traces, basically call tracing.
<li><h4>orc8r_worker</h4>
a singleton backgroun helper
<li><h4>analytics</h4>
periodically fetches and aggregates metrics, exporting them to Prometheus
</ul>

