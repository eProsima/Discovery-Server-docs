.. _xml_configuration_examples:

Transport protocol configuration
################################

UDP settings
************

The XML basically mimics the `UDP attribute C++ source code <cpp_udp_settings>`_:

.. literalinclude:: ../02-resources/examples/xml/HelloWorld_UDP_config.xml
    :language: xml
    :linenos:

+   Server prefix is specified.
+   Discovery kind set to SERVER.
+   Metatrafic locators set to the UDP listening port.

.. note::

    ``leaseDuration`` is set to ``INFINITY`` in order to mimic the HelloWorldExample participants but can be
    whatever value without affecting the discovery operation.

TCP settings
************

The XML basically mimics the `TCP attribute C++ source code <cpp_tcp_settings>`_:

.. literalinclude:: ../02-resources/examples/xml/HelloWorld_TCP_config.xml
    :language: xml
    :linenos:

+   A TCP transport descriptor is created specifying the physical listening port as 9843.
+   The above transport descriptor is added to the participant user transports.
+   Builtin transport is disabled to avoid UDP operation. This wouldn't disturb TCP communication in any way and is
    specified merely to prove that the actual discovery traffic is not going through UDP.
+   Server prefix is specified
+   Discovery kind set to SERVER.
+   Metatrafic locators set to the logical listening port. The real TCP locator is provided in the transport this one
    is merely a port number that is linked with this particular server.

.. note::

    ``leaseDuration`` is set to ``INFINITY`` in order to mimic the HelloWorldExample participants but can be
    whatever value without affecting the discovery operation.

UDP and TCP simultaneously
**************************

The XML config generates a server able to listen simultaneously on TCP or UDP ports.
It mixes concepts from previous UDP and TCP config files:

.. literalinclude:: ../02-resources/examples/xml/HelloWorld_UDP_TCP_config.xml
    :language: xml
    :linenos:

+   A TCP transport descriptor is created specifying the physical listening port as 9843.
+   The above transport descriptor is added to the participant user transports.
+   Builtin transport is not disabled in order to allow UDP traffic.
+   Server prefix is specified
+   Discovery kind set to SERVER.
+   Metatrafic locators set to the logical TCP listening port and UDP actual IP address and listening port.

Using this last config XML file to generate a server allows, not only that participants with the same transport
(either UDP or TCP) discover each other, but that all participants (disregarding selected transport) discover
each other. A publisher in a TCP participant can match a subscriber in a TCP one (cannot exchange data due to the
configuration of the HelloWorldExample Clients; only one transport is selected).
