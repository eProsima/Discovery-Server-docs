Example application
###################

**Table of Contents**

* :ref:`UDP transport attribute settings`
* :ref:`TCP transport attribute settings`
* :ref:`HelloWorldExample command line syntax`

The fast-RTPS **HelloWorldExample** has been updated to illustrate the client-server functionality. Its installation
details are explained in `installation section <installation.html>`_. Basically, the publisher and subscriber
participants are now *clients* and can only discover each other when a *server* participant is created. 

As usual, we launch publishers and subscribers by running HelloWorldExampleDS.exe with the corresponding **publisher**
or **subscriber** argument. Each publisher and subscriber is launched within its own participant, but now the
:code:`HelloWorldPublisher::init()` and :code:`HelloWorldSubscriber::init()` methods are modified to create clients
and hard code the server's reference.

UDP transport attribute settings
--------------------------------

In order to use UDP we can rely in the default transport where the locators are actual ports and IP addresses.

UDP transport code setup for a client
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: c++

    RemoteServerAttributes ratt;
    ratt.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::CLIENT;
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_pub");
	
    Locator_t server_address(LOCATOR_KIND_UDPv4, 65215);
    IPLocator::setIPv4(server_address, 127, 0, 0, 1);

    ratt.metatrafficUnicastLocatorList.push_back(server_address);
    PParam.rtps.builtin.discovery_config.m_DiscoveryServers.push_back(ratt);
	
    mp_participant = Domain::createParticipant(PParam);
	
Note that according with `former attributes explanation <command_line#rtps-attributes-dealing-with-discovery-services>`_
we must populate the **DiscoverySettings discovery_config** specifying we want to create a
**DiscoveryProtocol_t::CLIENT** and adding a new *RemoteServerAttributes* object to the *m_DiscoveryServers* list.
In this case the UDP port 65215 is hardcoded as is the server prefix.

UDP transport code setup for a server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: c++

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::SERVER;
    PParam.rtps.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_server");
	
    Locator_t server_address(LOCATOR_KIND_UDPv4, 65215);
    IPLocator::setIPv4(server_address, 127, 0, 0, 1);

    PParam.rtps.builtin.metatrafficUnicastLocatorList.push_back(server_address);
	
    mp_participant = Domain::createParticipant(PParam);
	
Note that according with `former attributes explanation <command_line#rtps-attributes-dealing-with-discovery-services>`_
we must populate the **DiscoverySettings discovery_config** specifying we want to create a
**DiscoveryProtocol_t::SERVER** and adding a new listening locator to any **BuiltinAttributes** metatraffic lists
(this locator or locators must be known by the clients). In this case the UDP port 65215 is hardcoded as is the
server prefix.

TCP transport attribute settings
--------------------------------

For TCP transport is mandatory to disable the default transport setting the
**RTPSParticipantAttributes::useBuiltinTransports** as false and creating a new *transport descriptor* thus
fast RTPS framework might create a suitable transport object.

TCP transport code setup for a client
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: c++

    RemoteServerAttributes ratt;
    ratt.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::CLIENT;
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_pub");
	
    Locator_t server_address; 
    server_address.kind = LOCATOR_KIND_TCPv4;
    IPLocator::setLogicalPort(server_address, 65215);
    IPLocator::setPhysicalPort(server_address, 9843); 
    IPLocator::setIPv4(server_address, 127, 0, 0, 1);

    ratt.metatrafficUnicastLocatorList.push_back(server_address);
    PParam.rtps.builtin.discovery_config.m_DiscoveryServers.push_back(ratt);

    PParam.rtps.useBuiltinTransports = false;
    std::shared_ptr<TCPv4TransportDescriptor> descriptor = std::make_shared<TCPv4TransportDescriptor>();

    // Generate a listening port for the client
    std::default_random_engine gen(System::GetPID());
    std::uniform_int_distribution<int> rdn(49152, 65535);
    descriptor->add_listener_port(rdn(gen)); // IANA ephemeral port number

    descriptor->wait_for_tcp_negotiation = false;
    PParam.rtps.userTransports.push_back(descriptor);

    mp_participant = Domain::createParticipant(PParam);
	
The **DiscoverySettings discovery_config** is almost the same as in
`UDP client case <#udp-transport-code-setup-of-a-client>`_.  Note that here the *server_address* locator
specifies 65215 as a logical port and 9843 as a physical one. The reason behind this is that TCP transport was
devised in order to allow a single TCP connection tunnel several participants traffic through it.
In order to differenciate each participant sharing the connection a *logical port concept* was introduced.
The transport will understand that must connect to the physical port (using TCP protocol) and rely meta traffic
to the logical port 65215 which is the meta traffic mailbox of the server we are interested in.

A new TCPv4TransportDescriptor must be created and a physical listening port selected. In this case each
HelloWorldExample instance creates a single participant thus the linked proccess ID is a suitable seed to make up
a listening port number (this way each time a new client is created a different port is selected).

TCP transport code setup for a server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: c++

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::SERVER;
    PParam.rtps.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_server");

    Locator_t server_address; 
    server_address.kind = LOCATOR_KIND_TCPv4;
    IPLocator::setLogicalPort(server_address, 65215);
    IPLocator::setIPv4(server_address, 127, 0, 0, 1);

    PParam.rtps.builtin.metatrafficUnicastLocatorList.push_back(server_address);

    std::shared_ptr<TCPv4TransportDescriptor> descriptor = std::make_shared<TCPv4TransportDescriptor>();
    descriptor->wait_for_tcp_negotiation = false;
    descriptor->add_listener_port(9843);

    PParam.rtps.useBuiltinTransports = false;
    PParam.rtps.userTransports.push_back(descriptor);

    mp_participant = Domain::createParticipant(PParam);

The **DiscoverySettings discovery_config** is almost the same as in
`UDP server case <#udp-transport-code-setup-of-a-server>`_. Note that here the *server_address* locator specifies
65215 as a logical port instead of a physical one. 

A new TCPv4TransportDescriptor must be created and a physical listening port selected. Unlike the client code this
listening port (9843 in the example) must be known beforehand for all clients in order to successfully deliver
meta traffic to the server. 

HelloWorldExample command line syntax
=====================================

The enviromental variables must be appropriately setup as explained in the
`installation steps <installation.html#installation-steps>`_ by employing a colcon generated script file.
For colcon builds the relative path to the script from the example directory would be:

Linux::

	$ . ../../../../../local_setup.bash
		
Windows::

	>..\..\..\..\..\local_setup.bat
		
otherwise modify the console PATH or terminal LIB_PATH_DIR environmental variables to allow the example binary to locate
fast shared libraries.

The command line syntax is the usual for the HelloWorldExample although a new flag **-t** or **--tcp** is introduced to
enforce the use of tcp transport:

Linux:

.. code-block:: bash

	$ ./HelloWorldExampleDS publisher|subscriber|server [ -h | -t | -c [<num>] | -i [<num>] ]
	
Windows:

.. code-block:: bat

	> HelloWorldExampleDS publisher|subscriber|server [ -h | -t | -c [<num>] | -i [<num>] ]

+----------+------------------+---------------------------------------------------------+
| SHORTCUT |       FLAG       |                         MEANING                         |
+==========+==================+=========================================================+
| -h       | --help           | Produce help message                                    |
+----------+------------------+---------------------------------------------------------+
| -t       | --tcp            | Use tcp transport instead of the default UDP one        |
+----------+------------------+---------------------------------------------------------+
| -c <num> | --count=<num>    | Number of datagrams to send (0=infinite) defaults to 10 |
+----------+------------------+---------------------------------------------------------+
| -i <num> | --Interval=<num> | Time between samples in milliseconds defaults to 100    |
+----------+------------------+---------------------------------------------------------+

The main difference with the plain HelloWorldExample is that additionally to the Publisher and Subscriber instance we
must launch a server instance in order to allow publishers and subscribers to discover each other. A simple test 
would be as follows:

Linux:

	Terminal 1::
	
		$ ./HelloWorldExampleDS publisher
		
	Terminal 2::
	
		$ ./HelloWorldExampleDS subscriber	

	Terminal 3::
	
		$ ./HelloWorldExampleDS server	

Windows:
	
	Console 1::
	
		> HelloWorldExampleDS publisher
	
	Console 2::
	
		> HelloWorldExampleDS subscriber
	
	Console 3::
	
		> HelloWorldExampleDS server

The HelloWorldExampleDS server instance can be replace by a discovery-server instance that creates a suitable server.
Thus instead of calling :code:`HelloWorldExample --server` we can call:

Windows:

.. code-block:: bat

	> discovery-server-X.X.X(d).exe config-file.xml
	
Linux:

.. code-block:: bash

	$ ./discovery-server-X.X.X(d) config-file.xml


As an example we show here the configuration files associated with each specific kind of transport:


HelloWorld_UDP_config.xml
-------------------------

.. literalinclude:: ..\examples\HelloWorld_UDP_config.xml
    :linenos:
    :language: xml
    :start-after:	<profiles> 
    :end-before:	</profiles>

The xml basic mimics the `UDP attribute source code <#upd-transport-code-setup-for-a-server>`_ shown above:

 + server prefix is specified.
 + discovery kind set to SERVER.
 + metatrafic locators set to the UDP listening port.
 
Note that leaseDuration was set to INFINITY in order to mimic the HelloWorldExample usual participants but can be 
whatever value without affecting the discovery operation.

HelloWorld_TCP_config.xml
-------------------------

.. literalinclude:: ..\examples\HelloWorld_TCP_config.xml
    :linenos:
    :language: xml
    :start-after:	<profiles> 
    :end-before:	</profiles>

The xml basic mimics the `TCP attribute source code <#tcp-transport-code-setup-for-a-server>`_ shown above:

 + a tcp transport descriptor is created specifying the physical listening port as 9843.
 + the above transport descriptor is add to the participant user transports.
 + builtin transport is disabled to avoid UDP operation. This wouldn't disturb TCP communication in any way and is
   specified merely to prove that the actual discovery traffic is not going through UDP.
 + server prefix is specified
 + discovery kind set to SERVER.
 + metatrafic locators set to the logical listening port. The real TCP locator is provided in the transport this one
   is merely a port number that is linked with this particular server.
 
 Again, note that leaseDuration was set to INFINITY in order to mimic the HelloWorldExample usual participants but 
 can be whatever value without affecting the discovery operation.
 
HelloWorld_UDP_TCP_config.xml
-----------------------------

.. literalinclude:: ..\examples\HelloWorld_UDP_TCP_config.xml
    :linenos:
    :language: xml
    :start-after:	<profiles> 
    :end-before:	</profiles>

The above xml config generates a server able to listen simultaneously on TCP or UDP ports. It mixes concepts from
previos UDP and TCP config files:

 + a tcp transport descriptor is created specifying the physical listening port as 9843.
 + the above transport descriptor is add to the participant user transports.
 + builtin transport is not disabled in order to allow UDP traffic.
 + server prefix is specified
 + discovery kind set to SERVER.
 + metatrafic locators set to the logical TCP listening port and UDP actual IP address and listening port. Note that
   are both set to 65215 but this doesn't arise any problems because TCP is a logical port and has no physical meaning.
 
Using this last config xml file to generate a server allows, not only that participants with the same transport
(either UDP or TCP) discover each other, but that all participants (disregarding selected transport) discover
each other. A publisher in a TCP participant can match a subscriber in a TCP one (they cannot exchange data although
because as HelloWorldExample clients are setup only one transport is selected).
 


