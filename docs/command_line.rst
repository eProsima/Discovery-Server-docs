Executable use
##############

**Table of Contents**

:ref:`Basic concepts`
:ref:`Running a scenario`
:ref:`Validating a scenario`

Basic concepts
**************

Under the new client-server discovery paradigm, the metatraffic (message exchange among participants to identify each other) is centralized in one or several server participants (right figure), as opposed to simple discovery (left figure), where metatraffic is exchanged using a message broadcast mechanism like an IP multicast protocol.

.. uml::

@startuml

package "Simple Discovery" {

(participant 1) as p1
(participant 2) as p2
(participant 3) as p3
(participant 4) as p4


p1 <-right-> p2
p1 <-left-> p3
p1 <-up-> p4
p2 <-up-> p3
p2 <-> p4
p3 <-> p4

}


package "Discovery Server" {

' node "Discovery Server" as server
cloud "Discovery Server" as server
(participant 1) as ps1
(participant 2) as ps2
(participant 3) as ps3
(participant 4) as ps4

ps1 <-down-> server
ps2 <-up-> server
ps3 <-down-> server
ps4 <-up-> server

}

@enduml

Clients must be aware of how to reach the server, usually by specifying an IP address and a transport protocol like UDP or TCP. Servers don't need any beforehand knowledge of clients but we must specify where they should be reached, usually by specifying a listening IP address and transport protocol.

One of the design goals of the current implementation was to keep both the discovery messages structure and standard RTPS writer and reader behavior unchanged. In order to do so, clients must be aware of their server's GuidPrefix. GuidPrefix is the RTPS standard participant unique identifier (basically 12 bytes), and allows clients to assess whether they are receiving messages from the right server, as each standard RTPS message contains this piece of information. Note that server's IP address may not be a reliable server's identifier because several can be specified and multicast addresses are acceptable. In future implementations any other more convenient and non-standard identifier may substitute the GuidPrefix at the expense of adding non-standard members to the RTPS discovery messages structure.

RTPS Attributes dealing with discovery services
===============================================

Several fast-RTPS configuration structures have been updated in order to deal with the new client-server discovery strategy. Note that the following elements belong exclusively to fast RTPS builtin discovery architecture and that the discovery server application just profits from the capabilities provided by fast RTPS library.

RTPSParticipantAttributes
-------------------------

+ a `GuidPrefix_t guidPrefix` member specifies server's identity.  This member has only significance if `discovery_config.discoveryProtocol` is **SERVER** or **BACKUP**. There is a `ReadguidPrefix` method to easily fill in this member from a string formatted like `"4D.49.47.55.45.4c.5f.42.41.52.52.4f"` (note that each byte must be a valid hexadecimal figure).

BuiltinAttributes
-----------------

+ All discovery related info is gathered in a `DiscoverySettings discovery_config` member.

+ In order to receive client metatraffic, `metatrafficUnicastLocatorList` or `metatrafficMulticastLocatorList` must be populated with the addresses that were given to the clients.

DiscoverySettings
-----------------

+ a `DiscoveryProtocol_t discoveryProtocol` member specifies participant's discovery kind:
	- **SIMPLE** generates a standard participant with complete backward compatibility with any other RTPS implementation.
	- **CLIENT** generates a *client* participant, which relies on a server to be notified of other *clients* presence. This participant can create publishers and subscribers of any topic (static or dynamic) as ordinary participants do.
	- **SERVER** generates a *server* participant, which receives, manages and spreads its linked *clients* metatraffic assuring any single one is aware of the others. This participant can create publishers and subscribers of any topic (static or dynamic) as ordinary participants do. Servers can link to other servers in order to share its clients information.
	- **BACKUP** generates a *server* participant with additional functionality over **SERVER**. Specifically, it uses a database to backup its client information, so that if for whatever reason it disappears, it can be automatically restored and continue spreading metatraffic to late joiners. A **SERVER** in the same scenario ought to collect client information again, introducing a recovery delay.

+ a `RemoteServerList_t  m_DiscoveryServers` lists the servers linked to the participant. This member has only significance if `discoveryProtocol` is **CLIENT**, **SERVER** or **BACKUP**. This member elements are `RemoteServerAttributes` objects that identify each server and report where to reach it:
	- `GuidPrefix_t guidPrefix` is the RTPS unique identifier of the server participant we want to link to. There is a `ReadguidPrefix` method to easily fill in this member from a string formatted like `"4D.49.47.55.45.4c.5f.42.41.52.52.4f"` (note that each octec must be a valid hexadecimal figure).
	- `metatrafficUnicastLocatorList` and `metatrafficMulticastLocatorList` are ordinary `LocatorList_t` (see fast-RTPS documentation) where server's locators must be specified. At least one of them should be populated.
	- `Duration_t discoveryServer_client_syncperiod` specifies the time span between PDP metatraffic exchange, and has only significance if `discoveryProtocol` is **CLIENT**, **SERVER** or **BACKUP**. The default value is half a second.
	
RTPS schema elements dealing with discovery services
=====================================================

Each of the attributes in fast-RTPS has an echo in the XML profiles. XML profiles make it possible to avoid tiresome hard-coded settings within applications sources using XML configuration files. The fast XML schema was duly updated to accommodate the new client-server attributes:

+ The participant profile **rtps** tag contains a new optional **prefix** tag where the server `GuidPrefix_t` must be specified. Any other discovery selection as simple or clients may disregard this member.

+ The participant profile **builtin** tag contains a **discovery_config** tag where all discovery related info is gathered. This new tag contains the following new elements:
	- a **discoveryProtocol** tag, where the discovery type can be specified through the `DiscoveryProtocol_t` enumeration quoted above see `above <DiscoverySettings_>`_.
	- a **discoveryServersList** tag, where the server or servers linked with a participant can be specified.
	- a **clientAnnouncementPeriod** tag, where the time span between PDP metatraffic exchange can be specified.
	
Below we provide an example xml participant profile using this new *tags*:	
	
..literalinclude:: 

