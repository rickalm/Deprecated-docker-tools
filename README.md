# Docker Functions

These bash functions are intended to help determine information about the container your application is running within. Most of the functions center around determining the NAT'ted port mappings outside of the container so that the application can properly advertise how to reach it.

Most if not all of the functions require access to the docker socket and its default location is /var/run/docker.sock but can be placed anywhere as long as the environment variable DOCKER_SOCKET points to the alternative location

## Environment Variables

* DEBUG - if set, diverts stderr to ${log_dir}/${appname}.stderr and activates bash debugging (set -x)
* DOCKER_SOCKET - default /var/run/docker.sock
* NODE_NAME - alternate value for name of docker host
* NODE_IP - alternate value for docker host IP address

## "Internal" Functions

These functions are mostly for use by the library itself for accessing the docker socket or other internal resources. These functions form the basis for the Public Functions

These Functions DO NOT rely on access to the docker socket to determine their answer

* get_docker_mode() - Determines which networking mode the application is running in and return the word BRIDGE or NET
* get_docker_container_id - returns the long numeric identifier for the container

* get_node_name() - returns the short hostname as known by the /bin/hostname command, but can be overridden by the NODE_NAME environment variable
* get_node_ip() - returns the first IP address as known by the /bin/hostname command, but can be overridden by the NODE_IP environment variable


The remainder of the functions require access to the docker socket to function

* get_docker_container_name - returns the logical name assigned to the container by the user at launch
* extract_from_json() - Passed a json document via STDIN, this function passes its arguments to JQ and then strips any leading and trailing " (double quote) from the response
* info_docker() - passes the argument to the socket as the path to get (GET /$1 HTTP/1.1) and trims the result to just the JSON reply from the socket
* info_docker_host() - returns the json document based on /info which describes the host dockerd is running on
* info_docker_container() - returns the json document describing the current container

## "Public" Functions

These functions are the most useful to application developers for obtaining information about the networking conditions the container is operating with

* is_docker_mode_bridge() - alias for [ $(get_docker_mode)" == "BRIDGE" ]
* is_docker_mode_host() - alias for [ $(get_docker_mode)" == "HOST" ]

These functions will determine the mapped values when running within a bridged container, otherwise will return the values from the host environment

* get_docker_host_ip() - Attempts to resolve the IP address of the Docker Host, otherwise returns the same value as get_node_ip
* get_docker_container_nat_port(&lt;port to un-nat&gt;) - Will determine the external port that is used to NAT to the internal container port otherwise returns the value passed to it
* get_docker_container_nat_ip(&lt;port to un-nat&gt;) - If port is natted and if the Nat specifies an alternative interface to bind to it returns that IP address (NAT Bound IP -> Docker Host IP -> Container IP)

While not technically a "Docker Function" this is used to perform an UNPRIVLEDGED nmap TCP port scan to look for neighbor processes. This was initialy conceived for bringing up a cluster where the cluster leads had no prior knowledge of their peers at boot (AWS autoscale groups are a good example of this)
 
* port_scan_network(&lt;port to scan&gt; [ ,&lt;subnet mask 16-32&gt; [ ,&lt;IP address to use as base address to scan&gt; ] ] )

- If in docker_host mode, will determine the subnet mask based on the interface used for default route
- If subnet mask is the string "subnet" will replace it with the discovered subnet mask (done for the case where you dont want to specify a mask and passing an empty parameter causes issues)
- can scan a specific host by calling port_scan_network &lt;port&gt; 32 &lt;host_ip&gt;
- values returned from this function is a space seperated list of IP:PORT for each responder discovered
	
