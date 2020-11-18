.. _getting_started:

Getting started
###############

This section explains the basic concepts of the Discovery Server discovery mechanism.
For more information on the Discovery Server mechanism, please refer to the
`Fast DDS documentation <https://fast-dds.docs.eprosima.com/en/latest/fastdds/discovery/server_client.html>`__.

Basic concepts
**************

Under the new client-server discovery paradigm, the metatraffic (message exchange among participants to identify each
other) is centralized in one or several server participants (right figure), as opposed to simple discovery
(left figure), where metatraffic is exchanged using a message broadcast mechanism like an IP multicast protocol.

.. image:: /01-figures/ds_uml.png
    :align: center
    :alt: Discovery Server discovery mechanism

Clients must be aware of how to reach the server, usually by specifying an IP address and a transport protocol like UDP
or TCP. Servers do not need any beforehand knowledge of their clients but, we must specify where they may be reached by
them, usually by specifying a listening IP address and transport protocol.

One of the design goals of the current implementation was to keep both the discovery messages structure and standard
RTPS writer and reader behavior unchanged. In order to do so, clients must be aware of their server's GuidPrefix.
GuidPrefix is the RTPS standard participant unique identifier (basically 12 bytes) which allows clients to assess
whether they are receiving messages from the right server, as each standard RTPS message contains this piece of
information. Note that the server's IP address may not be a reliable server's identifier because several can be
specified and multicast addresses are acceptable. In future implementations, any other more convenient and non-standard
identifier may substitute the GuidPrefix at the expense of adding non-standard members to the RTPS discovery messages
structure.

Finally, the discovery between clients is only performed in case of a match at the topic level between publishers and
subscribers.
That is, customers with publishers and subscribers on different topics will never discover each other.
This implies a notorious reduction in the number of messages exchanged.

.. _rtps_attr:

RTPS attributes dealing with discovery services
===============================================

Several *Fast DDS* configuration structures have been updated in order to deal with the new client-server discovery
strategy. Note that the following elements belong exclusively to fast RTPS builtin discovery architecture and that
the discovery server application just profits from the capabilities provided by *Fast DDS* library.

RTPSParticipantAttributes
-------------------------

-   ``GuidPrefix_t guidPrefix`` member specifies the server's identity.  This member has only significance if
    `discovery_config.discoveryProtocol` is **SERVER** or **BACKUP**. There is a `ReadguidPrefix` method to easily fill
    in this member from a string formatted like ``"4D.49.47.55.45.4c.5f.42.41.52.52.4f"`` (note that each byte must
    be a valid hexadecimal figure).

BuiltinAttributes
-----------------

+   All discovery related info is gathered in a ``DiscoverySettings discovery_config`` member.

+   In order to receive client metatraffic, `metatrafficUnicastLocatorList` or `metatrafficMulticastLocatorList` must
    be populated with the addresses that were given to the clients.

.. _getting_started_discovery_settings:

DiscoverySettings
-----------------

+   ``DiscoveryProtocol_t discoveryProtocol`` member specifies the participant's discovery kind:

    -   **SIMPLE** generates a standard participant with complete backward compatibility with any other RTPS
        implementation.
    -   **CLIENT** generates a *client* participant, which relies on a server to be notified of other *clients*
        presence.
        This participant can create publishers and subscribers of any topic (static or dynamic) as ordinary participants
        do.
    -   **SERVER** generates a *server* participant, which receives, manages and spreads its linked *clients*
        metatraffic assuring any single one is aware of the others. This participant can create publishers and
        subscribers of any topic (static or dynamic) as ordinary participants do. Servers can link to other servers
        in order to share its clients information.
    -   **BACKUP** generates a *server* participant with additional functionality over **SERVER**. Specifically, it uses
        a database to backup its client information, so that if for whatever reason it disappears, it can be
        automatically restored and continue spreading metatraffic to late joiners. A **SERVER** in the same scenario
        ought to collect client information again, introducing a recovery delay.

+   ``RemoteServerList_t m_DiscoveryServers`` lists the servers linked to the participant. This member has only
    significance if ``discoveryProtocol`` is **CLIENT**, **SERVER** or **BACKUP**. These member elements are
    ``RemoteServerAttributes`` objects that identify each server and report where the servers can be reached:

    -   ``GuidPrefix_t guidPrefix`` is the RTPS unique identifier of the server participant we want to link to.
        There is a `ReadguidPrefix` method to easily fill in this member from a string formatted like
        `"4D.49.47.55.45.4c.5f.42.41.52.52.4f"` (note that each octet must be a valid hexadecimal figure).
    -   ``metatrafficUnicastLocatorList`` and `metatrafficMulticastLocatorList` are ordinary `LocatorList_t`
        (see *Fast DDS* documentation) where the server's locators must be specified. At least one of them should be
        populated.
    -   ``Duration_t discoveryServer_client_syncperiod`` specifies the time span of PDP metatraffic exchange,
        and has only significance if ``discoveryProtocol`` is **CLIENT**, **SERVER** or **BACKUP**.
        The default value is half a second.

.. _getting_started_rtps_schema:

RTPS schema elements dealing with discovery services
=====================================================

Each of the attributes in *Fast DDS* has its equivalent in the XML profiles. XML profiles make it possible to avoid
tiresome hard-coded settings within application sources using XML configuration files. The fast XML schema was duly
updated to accommodate the new client-server attributes:

+   The participant profile ``rtps`` tag contains a new optional ``prefix`` tag where the server ``GuidPrefix_t``
    must be specified.
    Any other discovery selection as simple or clients may disregard this member.

-   The participant profile ``builtin`` tag contains a ``discovery_config`` tag where all discovery-related info is
    gathered. This new tag contains the following new XML child elements:
    -   ``<discoveryProtocol>``: specifies the discovery type through the ``DiscoveryProtocol_t`` enumeration.
    -   ``<discoveryServersList>``: specifies the server or servers linked with a Client/Server.
    -   ``<clientAnnouncementPeriod>``: specifies the time span between PDP metatraffic exchange.

An XML profiles examples using this new tags can be found :ref:`here <basic_config_file>`.
