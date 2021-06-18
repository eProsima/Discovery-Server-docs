.. _cpp_tcp_settings:

TCP transport attribute settings
################################

For TCP transport is mandatory to disable the default transport setting the
``RTPSParticipantAttributes::useBuiltinTransports`` as ``false`` and creating a new *transport descriptor* thus
*Fast DDS* framework might create a suitable transport object.

TCP transport code setup for a client
*************************************

The ``DiscoverySettings discovery_config`` is almost the same as in
:ref:`UDP client case <udp_transport_client>`.
Note that here the ``server_address`` locator specifies 65215 as the logical port and 9843 as the physical one.
The reason behind this is that TCP transport was
devised in order to allow a single TCP connection tunnel several participants traffic through it.
In order to differentiate each participant sharing the connection, a *logical port concept* was introduced.
The transport will understand that must connect to the physical port (using TCP protocol) and relay metatraffic
to the logical port 65215, which is the metatraffic mailbox of the Server.

A new ``TCPv4TransportDescriptor`` must be created and a physical listening port selected.
In this case, each `HelloWorldExample` instance creates a single participant thus the linked process ID is a
suitable seed to make up a listening port number
(this way each time a new Client is created a different port is selected).

.. code-block:: c++

    RemoteServerAttributes ratt;
    ratt.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::CLIENT;
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_pub");

    // Placeholder values for the server address
    Locator_t server_address;
    server_address.kind = LOCATOR_KIND_TCPv4;
    IPLocator::setLogicalPort(server_address, 64863);
    IPLocator::setPhysicalPort(server_address, 9843);
    IPLocator::setIPv4(server_address, 192, 168, 1, 113);

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

TCP transport code setup for a server
*************************************

The ``DiscoverySettings discovery_config`` is almost the same as in
:ref:`UDP server case <udp_transport_server>`.
Here the *server_address* locator specifies 64863 as the logical port instead of the physical one.

A new ``TCPv4TransportDescriptor`` must be created and a physical listening port selected.
Unlike the client code, this
listening port (9843 in the example) must be known beforehand for all Clients in order to successfully deliver
metatraffic to the server.

.. code-block:: c++

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::SERVER;
    PParam.rtps.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_server");

    // Placeholder values for the server address
    Locator_t server_address;
    server_address.kind = LOCATOR_KIND_TCPv4;
    IPLocator::setLogicalPort(server_address, 64863);
    IPLocator::setIPv4(server_address, 192, 168, 1, 113);

    PParam.rtps.builtin.metatrafficUnicastLocatorList.push_back(server_address);

    std::shared_ptr<TCPv4TransportDescriptor> descriptor = std::make_shared<TCPv4TransportDescriptor>();
    descriptor->wait_for_tcp_negotiation = false;
    descriptor->add_listener_port(9843);

    PParam.rtps.useBuiltinTransports = false;
    PParam.rtps.userTransports.push_back(descriptor);

    mp_participant = Domain::createParticipant(PParam);
