Example application
###################

The fast-RTPS **HelloWorldExample** has been updated to illustrate the client-server functionality. Its installation details are explained in section Intallation_. Basically, the publisher and subscriber participants are now *clients* and can only discover each other when a *server* participant is created. 

As usual, we launch publishers and subscribers by running HelloWorldExampleDS.exe with the corresponding **publisher** or **subscriber** argument. Each publisher and subscriber is launched within its own participant, but now the HelloWorldPublisher::init() and HelloWorldSubscriber::init() methods are modified to create clients and hard code the server's reference.

UDP transport attribute settings
--------------------------------

In order to use UDP we can rely in the default transport where the locators are actual ports and IP addresses.

UDP transport code setup for a client
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
	
Note that according with `former attributes explanation <command_line#rtps-attributes-dealing-with-discovery-services>`_ we must populate the **DiscoverySettings discovery_config** specifying we want to create a **DiscoveryProtocol_t::CLIENT** and adding a new *RemoteServerAttributes* object to the *m_DiscoveryServers* list. In this case the UDP port 65215 is hardcoded as is the server prefix.

UDP transport code setup for a server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
	
Note that according with `former attributes explanation <command_line#rtps-attributes-dealing-with-discovery-services>`_ we must populate the **DiscoverySettings discovery_config** specifying we want to create a **DiscoveryProtocol_t::SERVER** and adding a new listening locator to any **BuiltinAttributes** metatraffic lists (this locator or locators must be known by the clients). In this case the UDP port 65215 is hardcoded as is the server prefix.

TCP transport attribute settings
--------------------------------

For TCP transport is mandatory to disable the default transport setting the **RTPSParticipantAttributes::useBuiltinTransports** as false and creating a new *transport descriptor* thus fast RTPS framework might create a suitable transport object.

TCP transport code setup for a client
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
	
The **DiscoverySettings discovery_config** is almost the same as in `UDP client case <#udp-transport-code-setup-of-a-client>`_.  Note that here the *server_address* locator specifies 65215 as a logical port and 9843 as a physical one. The reason behind this is that TCP transport was devised in order to allow a single TCP connection tunnel several participants traffic through it. In order to differenciate each participant sharing the connection a *logical port concept* was introduced. The transport will understand that must connect to the physical port (using TCP protocol) and rely meta traffic to the logical port 65215 which is the meta traffic mailbox of the server we are interested in.

A new TCPv4TransportDescriptor must be created and a physical listening port selected. In this case each HelloWorldExample instance creates a single participant thus the linked proccess ID is a suitable seed to make up a listening port number (this way each time a new client is created a different port is selected).

TCP transport code setup for a server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
	
The **DiscoverySettings discovery_config** is almost the same as in `UDP client case <#udp-transport-code-setup-of-a-server>`_. Note that here the *server_address* locator specifies 65215 as a logical port instead of a physical one. 

A new TCPv4TransportDescriptor must be created and a physical listening port selected. Unlike the client code this listening port (9843 in the example) must be known beforehand for all clients in order to successfully deliver meta traffic to the server. 

HelloWorldExample command line
==============================