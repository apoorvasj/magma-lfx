<h1> Description of key Components in Magma's Architecture </h1>
Before diving into the architecture, there are some terms to understand.
<h3> Reverse Proxy </h3>
A reverse proxy acts as a load-balancer for user traffic that comes to a server. It also performs other key functions such as authentication, rate limits and routing requests to the correct endpoint. Nginx is one such  reverse proxy.
<h2> Sequence diagram of the three components interaction </h2>
The diagram below describes how the three core components of magma interact with each other i.e the ochestrator, NMS and access gateway.
<img src="images\magma_sequence_diagram_final.png">
<h2> NMS </h2>
NMS or Network Management System provides a centralised UI to add and remove equipment, networks, subscribers and view related metrics. 
NMS or Network Management System provides a centralized UI to manage Magma networks. It allows operators to configure gateways and eNodeBs, monitor network state, view metrics, and manage alerts.

<img src="images\nms ui components.png">

<h3> Tech Stack of NMS </h3>
<ul>
<li>React (Material UI + hooks)</li>
<li>Magmalte (Express backend)</li>
<li>Sequelize ORM</li>
<li>PostgreSQL (NMS DB)</li>
</ul>

<h3> UI Components </h3>
<ul>

<li><h4>Host Page</h4>
Manages organizations, users, and feature flags.</li>

<li><h4>NMS Page</h4>
Displays dashboards, networks, equipment, subscribers, metrics, and alerts.</li>

<li><h4>Admin Page</h4>
Used to add networks and users to organizations.</li>

</ul>

<h3> Backend (Magmalte) </h3>
<ul>

<li><h4>AuthenticationMiddleware</h4>
Handles login using username/password or SAML-based SSO.</li>

<li><h4>SessionMiddleware</h4>
Manages user sessions and cookies stored in NMS DB.</li>

<li><h4>csrfMiddleware</h4>
Protects against CSRF attacks.</li>

<li><h4>appMiddleware</h4>
Handles general app settings like JSON parsing and compression.</li>

<li><h4>userMiddleware</h4>
Handles user login routes.</li>

<li><h4>hostOrgMiddleware</h4>
Restricts access to host organization routes.</li>

</ul>

<h3> API Routing </h3>
<ul>

<li><h4>apiControllerRouter</h4>
Proxies requests to Orc8r APIs and enforces network-level access control.</li>

<li><h4>networkRouter</h4>
Handles network creation and management.</li>

<li><h4>grafanaRouter</h4>
Embeds Grafana dashboards and fetches metrics from Orc8r.</li>

</ul>

<h3> Database </h3>
<ul>

<li><h4>NMS DB</h4>
Stores users, organizations, feature flags, audit logs, and sessions.</li>

</ul>

<h3> External Services </h3>
<ul>

<li><h4>Orc8r</h4>
Provides APIs for network configuration and subscriber management.</li>

<li><h4>Grafana</h4>
Used for metrics visualization and dashboards.</li>

<li><h4>Prometheus & AlertManager</h4>
Handle monitoring and alerting.</li>

</ul>
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

<h2> Access Gateway (AGW) </h2>
Magma’s Access Gateway (AGW) is the edge component of Magma. It provides network services and policy enforcement. In an LTE network, it implements an evolved packet core (EPC), together with AAA and PGW functionality, and it is designed to work with existing commercial radio hardware. :contentReference[oaicite:0]{index=0}

<h3> Architecture </h3>
The AGW is built as a modular set of services. The main service is <b>magmad</b>, which orchestrates the lifecycle of the other AGW services and also acts as the bootstrapping client with Orchestrator. The AGW also contains services for control-plane handling, session and policy management, subscriber state, packet processing, telemetry, and optional monitoring. :contentReference[oaicite:1]{index=1}

<h3> Core Services </h3>

<ul>
<li><b>magmad</b> – parent service that starts the AGW services, reports service metrics, and bootstraps with Orchestrator.</li>
<li><b>control_proxy</b> – manages transport between gateways and the controller, using HTTP/2, TLS, and multiplexed gRPC calls.</li>
<li><b>sctpd</b> – handles SCTP termination for control signaling and preserves RAN connections in stateless mode.</li>
<li><b>mme / accessd</b> – handles LTE control-plane functionality and normalizes control signaling with RAN nodes.</li>
<li><b>enodebd</b> – manages eNodeBs, supports provisioning, and collects performance metrics.</li>
</ul> :contentReference[oaicite:2]{index=2}

<h3> Session, Policy, and Subscriber Management </h3>
<ul>
<li><b>mobilityd</b> – manages subscriber IP address allocation and release.</li>
<li><b>sessiond</b> – manages session lifecycle and policy state for users.</li>
<li><b>policydb</b> – stores static PCRF rules that are streamed from Orchestrator.</li>
<li><b>subscriberdb</b> – stores subscriber identity information and supports S6a-related authentication flow.</li>
<li><b>directoryd</b> – acts as a lookup service for subscriber and session keys.</li>
</ul> :contentReference[oaicite:3]{index=3}

<h3> Data Plane and Observability </h3>
<ul>
<li><b>pipelined</b> – programs Open vSwitch OpenFlow rules for the data plane. It is implemented as a chain of services that can be enabled or disabled through the Orchestrator REST API.</li>
<li><b>dpid</b> – provides deep packet inspection for policy enforcement.</li>
<li><b>health</b> – checks the state of core AGW services and clears corrupt state when needed.</li>
<li><b>eventd</b> – forwards generated events for later storage and query in Orchestrator / Elasticsearch.</li>
<li><b>smsd</b> – syncs SMS information with Orchestrator.</li>
<li><b>ctraced</b> – manages call tracing on the AGW and sends packet captures back to Orchestrator.</li>
<li><b>monitord</b> and <b>td-agent-bit</b> – optional services for monitoring and log aggregation.</li>
</ul> :contentReference[oaicite:4]{index=4}

<h3> Big Picture </h3>
AGW sits between the radio access network and Orchestrator. It receives configuration and policy from the cloud side, programs the local data plane, and reports subscriber state, health, metrics, and events back upstream. :contentReference[oaicite:5]{index=5}