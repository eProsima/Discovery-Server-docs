Example application
###################

**Table of Contents**

* :ref:`UDP transport attribute settings`
* :ref:`TCP transport attribute settings`
* :ref:`HelloWorldExample command line syntax`

The Fast-RTPS **HelloWorldExample** has been updated to illustrate the client-server functionality. Its installation
details are explained in the `installation section <installation.html>`_. Basically, the publisher and subscriber
participants are now *clients* and can only discover each other when a *server* participant is created.

As usual, we launch publishers and subscribers by running HelloWorldExampleDS.exe with the corresponding **publisher**
or **subscriber** argument. Each publisher and subscriber is launched within its own participant, but now the
:code:`HelloWorldPublisher::init()` and :code:`HelloWorldSubscriber::init()` methods are modified to create clients
and add the server address specified by command line (`see examples <helloworldexample#helloworldexample_command_line_syntax>`_). 

UDP transport attribute settings
--------------------------------

In order to use UDP, we can rely on the default transport where the locators are actual ports and IP addresses.

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

    // placeholder values for the server address
    Locator_t server_address(LOCATOR_KIND_UDPv4, 64863);
    IPLocator::setIPv4(server_address, 192, 168, 1, 113);

    ratt.metatrafficUnicastLocatorList.push_back(server_address);
    PParam.rtps.builtin.discovery_config.m_DiscoveryServers.push_back(ratt);

    mp_participant = Domain::createParticipant(PParam);

Note that according to `former attributes explanation <command_line#rtps-attributes-dealing-with-discovery-services>`_
we must populate the **DiscoverySettings discovery_config** specifying we want to create a
**DiscoveryProtocol_t::CLIENT** and adding a new *RemoteServerAttributes* object to the *m_DiscoveryServers* list.
In this case the UDP port 64863 is hardcoded as is the server prefix.

UDP transport code setup for a server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: c++

    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::SERVER;
    PParam.rtps.ReadguidPrefix("4D.49.47.55.45.4c.5f.42.41.52.52.4f");
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_server");

    // placeholder values for the server address
    Locator_t server_address(LOCATOR_KIND_UDPv4, 64863);
    IPLocator::setIPv4(server_address, 192, 168, 1, 113);

    PParam.rtps.builtin.metatrafficUnicastLocatorList.push_back(server_address);

    mp_participant = Domain::createParticipant(PParam);

Note that according to `former attributes explanation <command_line#rtps-attributes-dealing-with-discovery-services>`_
we must populate the **DiscoverySettings discovery_config** specifying we want to create a
**DiscoveryProtocol_t::SERVER** and adding a new listening locator to any **BuiltinAttributes** metatraffic lists
(this locator or locators must be known by the clients). In this case, the UDP port 64863 is hardcoded as is the
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

The **DiscoverySettings discovery_config** is almost the same as in
`UDP client case <#udp-transport-code-setup-of-a-client>`_.  Note that here the *server_address* locator
specifies 65215 as a logical port and 9843 as a physical one. The reason behind this is that TCP transport was
devised in order to allow a single TCP connection tunnel several participants traffic through it.
In order to differentiate each participant sharing the connection, a *logical port concept* was introduced.
The transport will understand that must connect to the physical port (using TCP protocol) and relay meta traffic
to the logical port 65215 which is the meta traffic mailbox of the server we are interested in.

A new TCPv4TransportDescriptor must be created and a physical listening port selected. In this case, each
HelloWorldExample instance creates a single participant thus the linked process ID is a suitable seed to make up
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

The **DiscoverySettings discovery_config** is almost the same as in
`UDP server case <#udp-transport-code-setup-of-a-server>`_. Note that here the *server_address* locator specifies
64863 as a logical port instead of a physical one.

A new TCPv4TransportDescriptor must be created and a physical listening port selected. Unlike the client code, this
listening port (9843 in the example) must be known beforehand for all clients in order to successfully deliver
meta traffic to the server.

HelloWorldExample command line syntax
=====================================

The environmental variables must be appropriately set up as explained in the
`installation steps <installation.html#installation-steps>`_ by employing a colcon generated script file.
For colcon builds the relative path to the script from the example directory would be:

Linux::

	$ . ../../../../../local_setup.bash

Windows::

	>..\..\..\..\..\local_setup.bat

otherwise, modify the console PATH or terminal LIB_PATH_DIR environmental variables to allow the example binary to locate
fast shared libraries.

The command-line syntax is the usual one for the HelloWorldExample although a new flag **-t** or **--tcp** is introduced to
enforce the use of TCP transport:

Linux:

.. code-block:: bash

	$ ./HelloWorldExampleDS publisher|subscriber|server [ -h | -t | -c [<num>] | -i [<num>]  | -l ip[:port] ]

Windows:

.. code-block:: bat

	> HelloWorldExampleDS publisher|subscriber|server [ -h | -t | -c [<num>] | -i [<num>] | -l ip[:port] ]

+----------------+------------------+---------------------------------------------------------+
| SHORTCUT       |       FLAG       |                         MEANING                         |
+================+==================+=========================================================+
| -h             | --help           | Produce help message                                    |
+----------------+------------------+---------------------------------------------------------+
| -t             | --tcp            | Use TCP transport instead of the default UDP one        |
+----------------+------------------+---------------------------------------------------------+
| -c <num>       | --count=<num>    | Number of datagrams to send (0=infinite) defaults to 10 |
+----------------+------------------+---------------------------------------------------------+
| -i <num>       | --Interval=<num> | Time between samples in milliseconds defaults to 100    |
+----------------+------------------+---------------------------------------------------------+
| -l <ip[:port]> | --ip=<ip[:port]> | Server address and physical port                        |  
+----------------+------------------+---------------------------------------------------------+

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

The HelloWorldExampleDS server instance can be replaced by a discovery-server instance that creates a suitable server.
Thus instead of calling :code:`HelloWorldExample --server`, we can call:

Windows:

.. code-block:: bat

	> discovery-server-X.X.X(d).exe config-file.xml

Linux:

.. code-block:: bash

	$ ./discovery-server-X.X.X(d) config-file.xml

As an example we show here the configuration files associated with each specific kind of transport:


HelloWorld_UDP_config.xml
-------------------------

.. literalinclude:: ../examples/HelloWorld_UDP_config.xml
    :linenos:
    :language: xml
    :start-after:	<profiles>
    :end-before:	</profiles>

The XML basically mimics the `UDP attribute source code <#upd-transport-code-setup-for-a-server>`_ showed above:

 + server prefix is specified.
 + discovery kind set to SERVER.
 + metatrafic locators set to the UDP listening port.

Note that leaseDuration was set to INFINITY in order to mimic the HelloWorldExample usual participants but can be
whatever value without affecting the discovery operation.

HelloWorld_TCP_config.xml
-------------------------

.. literalinclude:: ../examples/HelloWorld_TCP_config.xml
    :linenos:
    :language: xml
    :start-after:	<profiles>
    :end-before:	</profiles>

The XML basically mimics the `TCP attribute source code <#tcp-transport-code-setup-for-a-server>`_ showed above:

 + a TCP transport descriptor is created specifying the physical listening port as 9843.
 + the above transport descriptor is added to the participant user transports.
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

.. literalinclude:: ../examples/HelloWorld_UDP_TCP_config.xml
    :linenos:
    :language: xml
    :start-after:	<profiles>
    :end-before:	</profiles>

The above XML config generates a server able to listen simultaneously on TCP or UDP ports. It mixes concepts from
previous UDP and TCP config files:

 + a TCP transport descriptor is created specifying the physical listening port as 9843.
 + the above transport descriptor is added to the participant user transports.
 + builtin transport is not disabled in order to allow UDP traffic.
 + server prefix is specified
 + discovery kind set to SERVER.
 + metatrafic locators set to the logical TCP listening port and UDP actual IP address and listening port. 

Using this last config XML file to generate a server allows, not only that participants with the same transport
(either UDP or TCP) discover each other, but that all participants (disregarding selected transport) discover
each other. A publisher in a TCP participant can match a subscriber in a TCP one (they cannot exchange data although
because as HelloWorldExample clients are setup, only one transport is selected).

Testing over a network using HelloWorldExampleDS
------------------------------------------------

We need to now the server address and physical port. In our example it would be `192.168.1.113:64863`. Defaults to UDP,
if we want to use TCP add the `--tcp` flag to the following commands:

Windows:

- Console 1:

.. code-block:: bat

    > ..\..\..\..\..\local_setup.bat
    > HelloWorldExampleDS publisher --count=0 --ip=192.168.1.113:64863

by specifying `--count=0` the publisher keeps publishing samples forever. 

- Console 2:

.. code-block:: bat

    > ..\..\..\..\..\local_setup.bat
    > HelloWorldExampleDS subscriber --ip=192.168.1.113:64863

- Console 3:

.. code-block:: bat

    > ..\..\..\..\..\local_setup.bat
    > HelloWorldExampleDS server --ip=0.0.0.0:64863

Note that by using `0.0.0.0` as ip address we are hinting the server to publish all the local interfaces as metatraffic
(192.168.1.133 would be one of then in this example). The clients, once received the server metadata, would choose the
faster among the server interfaces. Of course we can force the use of a single interface by doing
`--ip=192.168.1.133:64863`. If we specify localhost as the interface (by doing `ip=127.0.0.1:64863`) only local clients
would be able to reach the server (we strongly discourage this).

Note also that if no port number is provided a default one is used.


Linux:

- Terminal 1:

.. code-block:: bash

	$ . ../../../../../local_setup.bash
    $ ./HelloWorldExampleDS publisher --count=0 --ip=192.168.1.113:64863

- Terminal 2:

.. code-block:: bash

	$ . ../../../../../local_setup.bash
    $ ./HelloWorldExampleDS subscriber --ip=192.168.1.113:64863

- Terminal 3:

.. code-block:: bash

	$ . ../../../../../local_setup.bash
    $ ./HelloWorldExampleDS server --ip=0.0.0.0:64863


