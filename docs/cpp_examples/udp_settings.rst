.. _cpp_udp_settings:

UDP transport attribute settings
################################

To use UDP, the application relies on the default transport where the locators are actual ports and IP addresses.

.. _udp_transport_client:

UDP transport code setup for a Client
*************************************

According to the `former RTPS attributes explanation <rtps_attr>`_,
the ``DiscoverySettings discovery_config`` must be populated specifying ``DiscoveryProtocol_t::CLIENT``
and adding a new ``RemoteServerAttributes`` object to the ``m_DiscoveryServers`` list.
In this case the UDP port 64863 is set as is the server prefix.

.. code-block:: c++

    RemoteServerAttributes ratt;
    ratt.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::CLIENT;
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_pub");

    // Placeholder values for the server address
    Locator_t server_address(LOCATOR_KIND_UDPv4, 64863);
    IPLocator::setIPv4(server_address, 192, 168, 1, 113);

    ratt.metatrafficUnicastLocatorList.push_back(server_address);
    PParam.rtps.builtin.discovery_config.m_DiscoveryServers.push_back(ratt);

    mp_participant = Domain::createParticipant(PParam);

.. _udp_transport_server:

UDP transport code setup for a server
*************************************

According to the `former RTPS attributes explanation <rtps_attr>`_,
the ``DiscoverySettings discovery_config`` specifying we want to create a
``DiscoveryProtocol_t::SERVER`` and adding a new listening locator to any ``BuiltinAttributes`` metatraffic lists
(this locator or locators must be known by the Clients).
In this case, the UDP port 64863 is hardcoded as is the Server prefix.

.. code-block:: c++

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::SERVER;
    PParam.rtps.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_server");

    // Placeholder values for the server address
    Locator_t server_address(LOCATOR_KIND_UDPv4, 64863);
    IPLocator::setIPv4(server_address, 192, 168, 1, 113);

    PParam.rtps.builtin.metatrafficUnicastLocatorList.push_back(server_address);

    mp_participant = Domain::createParticipant(PParam);
