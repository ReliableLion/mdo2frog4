# Virtualizer

The Virtualizer is an intermediate module sit between the [https://github.com/netgroup-polito/frog4-orchestrator/]FROG4 
global orchestrator and the upper layers of the Unify architecture. It operates as follows:
  * receives commands from the upper layers of the Unify architecture based on the virtualizer 
    library defined by WP3;
  * converts those commands in the formalism natively supported by the FROG4 orchestrator
    (described in [https://github.com/netgroup-polito/nffg-library](https://github.com/netgroup-polito/nffg-library));
  * sends the command to the FROG4 orchestrator through the API.

This module is an adaptation of a old virtualizer module used to integrate the un-orchestrator 
with the upper layes of the Unify architecture.
In this case the virtualizer interact through its northbound with the MultiDomain Orchestrator
(that is basically Escape) and through its southbound with the FROG4 global orchestrator.

## Required libraries

In the following we list the steps required on an **Ubuntu 14.04**.

	; Install required libraries
	; - build-essential: it includes GCC, basic libraries, etc
	; - cmake: to create cross-platform makefiles
	; - cmake-curses-gui: nice 'gui' to edit cmake files
	; - libboost-all-dev: nice c++ library with tons of useful functions
	; - libmicrohttpd-dev: embedded micro http server
	; - libxml2-dev: nice library to parse and create xml
	; - ethtool: utilities to set some parameters on the NICs (e.g., disable TCP offloading)
	
	$ sudo apt-get install build-essential cmake cmake-curses-gui libboost-all-dev libmicrohttpd-dev libxml2-dev ethtool

	; Install other required libraries 
	$ sudo apt-get install python-pip
	$ sudo pip install gunicorn falcon cython requests

## How to configure the Virtualizer

The Virtualizer reads its configuration from the file [./config/configuration.ini](config/configuration.ini), 
which must be properly edited before starting the Virtualizer itself.
The file template.xml must be setted with all the vm templates that the virtualizer will deploy.
The file .info.xml must be setted with all the port setted in the [./config/universal-node-example.xml].

## How to run the Virtualizer

	$ gunicorn -b ip:port virtualizer:api

where 'ip' and 'port' must be set to the desired values.


## Configuration file examples

Examples of configurations that can be sent to the Virtualizer are available in [./config/nffg_examples](nffg_examples).
In particular:
  * [./config/nffg_examples/simple_passthrough_nffg.xml](./config/nffg_examples/simple_passthrough_nffg.xml): 
    simple configuration that implements a simple passthrough function, i.e., traffic is 
    received from a first physical port and sent out from a second physical port, 
    after having been handled to the vSwitch;
  * [./config/nffg_examples/passthrough_with_vnf_nffg.xml](./config/nffg_examples/passthrough_with_vnf_nffg.xml): 
    configuration that includes a VNF. Traffic is received from a first physical 
    port, provided to a network function, and then sent out from a second physical 
    port;
  * [./config/nffg_examples/passthrough_with_vnf_nffg_and_match_and_action.xml](./config/passthrough_with_vnf_nffg_and_match_and_action.xml): 
    this configuration includes flows matching some protocol fields, and having 
    actions that manipulate protocol fields;
  * [./config/nffg_examples/nffg_delete_flow_vnf.xml](./config/nffg_examples/nffg_delete_flow_vnf.xml): 
    configuration that deletes some flows and a VNF instantiated on the Universal 
    Node.

## Rest API

The Virtualizer accept commands through its REST interface. The main REST commands 
to be used to interact with it are detailed in the following.

Get information about the Virtualizer

    GET /virt/ HTTP/1.1
    
Test the Virtualizer aliveness

    GET /virt/ping HTTP/1.1

Retrieve the current configuration of the universal node

    POST /virt/get-config HTTP/1.1

This command returns the current configuration in the format defined by the virtualzier library

Deploy a new configuration 

    POST /virt/edit-config HTTP/1.1

The body of the message must contain the new configuration for the universal node 
espressed in the format defined by the Virtualizer library.

### Send commands to the Virtualzier
    
In order to interact with the Virtualizer throug its REST API, you can use your favorite REST tool (e.g., some nice 
plugins for Mozilla Firefox). Just in also use the CURL command line tool, such as in the following example 
(where the NF-FG to be instantiated is stored in the file 'myGraph.xml'):

$ curl -i -d "@myGraph.json" -X POST  http://virtualizer-address:port/virt/edit-config
