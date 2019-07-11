Executable use
##############

**Table of Contents**

* :ref:`Basic concepts`
* :ref:`Directions for use`

Basic concepts
**************

Under the new client-server discovery paradigm, the metatraffic (message exchange among participants to identify each
other) is centralized in one or several server participants (right figure), as opposed to simple discovery
(left figure), where metatraffic is exchanged using a message broadcast mechanism like an IP multicast protocol.

.. image:: ds_uml.png
    :align: center
    :alt: client-server diagrams

Clients must be aware of how to reach the server, usually by specifying an IP address and a transport protocol like UDP
or TCP. Servers don't need any beforehand knowledge of their clients but, we must specify where they may be reached by
them, usually by specifying a listening IP address and transport protocol.

One of the design goals of the current implementation was to keep both the discovery messages structure and standard
RTPS writer and reader behavior unchanged. In order to do so, clients must be aware of their server's GuidPrefix.
GuidPrefix is the RTPS standard participant unique identifier (basically 12 bytes) which allows clients to assess
whether they are receiving messages from the right server, as each standard RTPS message contains this piece of
information. Note that the server's IP address may not be a reliable server's identifier because several can be specified
and multicast addresses are acceptable. In future implementations, any other more convenient and non-standard identifier
may substitute the GuidPrefix at the expense of adding non-standard members to the RTPS discovery messages structure.

RTPS Attributes dealing with discovery services
===============================================

Several Fast-RTPS configuration structures have been updated in order to deal with the new client-server discovery
strategy. Note that the following elements belong exclusively to fast RTPS builtin discovery architecture and that
the discovery server application just profits from the capabilities provided by Fast-RTPS library.

RTPSParticipantAttributes
-------------------------

+ A `GuidPrefix_t guidPrefix` member specifies the server's identity.  This member has only significance if
  `discovery_config.discoveryProtocol` is **SERVER** or **BACKUP**. There is a `ReadguidPrefix` method to easily fill
  in this member from a string formatted like `"4D.49.47.55.45.4c.5f.42.41.52.52.4f"` (note that each byte must be a
  valid hexadecimal figure).

BuiltinAttributes
-----------------

+ All discovery related info is gathered in a `DiscoverySettings discovery_config` member.

+ In order to receive client metatraffic, `metatrafficUnicastLocatorList` or `metatrafficMulticastLocatorList` must
  be populated with the addresses that were given to the clients.

DiscoverySettings
-----------------

+ A **DiscoveryProtocol_t discoveryProtocol** member specifies the participant's discovery kind:

    - **SIMPLE** generates a standard participant with complete backward compatibility with any other RTPS
      implementation.
    - **CLIENT** generates a *client* participant, which relies on a server to be notified of other *clients* presence.
      This participant can create publishers and subscribers of any topic (static or dynamic) as ordinary participants do.
    - **SERVER** generates a *server* participant, which receives, manages and spreads its linked *clients* metatraffic
      assuring any single one is aware of the others. This participant can create publishers and subscribers of any topic
      (static or dynamic) as ordinary participants do. Servers can link to other servers in order to share its clients
      information.
    - **BACKUP** generates a *server* participant with additional functionality over **SERVER**. Specifically, it uses
      a database to backup its client information, so that if for whatever reason it disappears, it can be automatically
      restored and continue spreading metatraffic to late joiners. A **SERVER** in the same scenario ought to collect
      client information again, introducing a recovery delay.

+ A **RemoteServerList_t  m_DiscoveryServers** lists the servers linked to the participant. This member has only
  significance if **discoveryProtocol** is **CLIENT**, **SERVER** or **BACKUP**. These member elements are
  `RemoteServerAttributes` objects that identify each server and report where the servers can be reached:

    - **GuidPrefix_t guidPrefix** is the RTPS unique identifier of the server participant we want to link to.
      There is a `ReadguidPrefix` method to easily fill in this member from a string formatted like
      `"4D.49.47.55.45.4c.5f.42.41.52.52.4f"` (note that each octec must be a valid hexadecimal figure).
    - **metatrafficUnicastLocatorList** and `metatrafficMulticastLocatorList` are ordinary `LocatorList_t`
      (see Fast-RTPS documentation) where the server's locators must be specified. At least one of them should be 
      populated.
    - **Duration_t discoveryServer_client_syncperiod** specifies the time span of PDP metatraffic exchange,
      and has only significance if `discoveryProtocol` is **CLIENT**, **SERVER** or **BACKUP**.
      The default value is half a second.

RTPS schema elements dealing with discovery services
=====================================================

Each of the attributes in Fast-RTPS has its equivalent in the XML profiles. XML profiles make it possible to avoid
tiresome hard-coded settings within application sources using XML configuration files. The fast XML schema was duly
updated to accommodate the new client-server attributes:

+ The participant profile **rtps** tag contains a new optional **prefix** tag where the server **GuidPrefix_t** must be
  specified. Any other discovery selection as simple or clients may disregard this member.

+ The participant profile **builtin** tag contains a **discovery_config** tag where all discovery-related info is
  gathered. This new tag contains the following new elements:

    - a **discoveryProtocol** tag, where the discovery type can be specified through the `DiscoveryProtocol_t`
      enumeration quoted `above <DiscoverySettings_>`_.
    - a **discoveryServersList** tag, where the server or servers linked with a participant can be specified.
    - a **clientAnnouncementPeriod** tag, where the time span between PDP metatraffic exchange can be specified.

Below we provide an example XML participant profile using this new *tags*:

.. literalinclude:: ../tests/test_1_PDP_UDP.xml
    :language: XML
    :start-after: <profiles>
    :end-before: </profiles>

Directions for use
******************

The discovery server binary (named after the pattern :code:`discovery-server-X.X.X(d)` where X.X.X is the version
number and the optional *d* denotes a debug builds) is set up from one or several XML config files passed as command-line
arguments. There are two modes of execution:

+ *Processing* mode. In this mode a single config file answering to the
  `discovery-server.xsd <../../schemas/discovery-server.xsd>` _schema. This schema is basically an extension of the
  fast RTPS one that simplifies the creation of custom servers and provides tools for easily testing specific
  discovery scenarios.

  When using the discovery server with testing purposes one may:

 - immediately validate when test execution is finished. This is the usual case when testing a single process scenario.

 - not validate the test results and generate an XML file with the test results. This results file follows a
   specific `ds-snapshot.xsd <../../schemas/ds-snapshot.xsd>`_ schema. This mode is activated by passing a filename
   in the input config XML. It's the usual case when testing a multiprocess or multimachine scenario. Each process or
   machine will generate a results XML file (with each one's discovery information record) and all of them would be
   validated later by running the discovery server binary in *validation* mode.

 ::

   > discovery-server-X.X.X(d).exe config_file.xml

+ *Validation* mode. Only for testing purposes. In this mode several XML results files generated on *Processing* mode
  would be compared to each other in other to assess if all discovery information was properly propagated by the server
  or servers involved in the testing.

 ::

   > discovery-server-X.X.X(d).exe results_file_1.xml results_file_2.xml results_file_3.xml ...

Note that if the colcon deployment strategy described in section
`Installation steps <installation.html#installation-steps>` was followed, before using the binary we must setup the
appropriate environmental variables using colcon generated scripts:

Linux:

.. code-block:: bash

    [BUILD]/install/discovery-server/bin$ . ../../local_setup.bash

Windows:

.. code-block:: bat

    [BUILD]\install\discovery-server\bin>..\..\local_setup.bat

where:

- the local_setup batch sets up the environment variables for the binary execution.
- the discovery-server binary name depends on build configuration (debug introduces `d` postfix) and version number.
- the config_file.xml is a placeholder for any XML config file that follows
  the `discovery-server.xsd <../../schemas/discovery-server.xsd>`_.

