Configuration files
####################

Discovery-server operation is managed from an XML config file that follows the **discovery-server.xsd** schema located in
**[SOURCES]**/discovery-server/resources/xsd (this is an extension of the Fast-RTPS XML schema). The discovery-server
main goals are:

-   Simplify the configuration of Fast-RTPS servers. Using a Fast-RTPS participant profile for each server is tiresome,
    given the large number of boilerplate code to move around. New XML syntax extensions are introduced to ease this task.

-   Provide a flexible testing tool for the client-server discovery implementation. Testing the discovery involves
    creating a large number of participants, publishers, subscribers that use specific topics and types (static or dynamic
    ones) over different transports. Besides, all these entities may appear or disappear at different times, and we need to
    be able to check the discovery status (collective participant knowledge) at any of these times.

The outermost XML tag is **DS**. It admits an optional boolean attribute called **user_shutdown** that defaults to
*true*. By default, the discover-server binary runs indefinitely until the user decides to shutdown. This default
behavior is suitable for practical applications but not for testing. Test XML files use :code:`user_shutdown="false"`,
which grants that the discovery server is closed as soon as the test is fulfilled. The **DS** tag can contain the
following tags:

+   **profiles** is plainly the Fast-RTPS profiles. We can use them to fine-tune the server operation. The associated
    documentation can be found `fast RTPS docs <https://eprosima-Fast-RTPS.readthedocs.io/en/latest/xmlprofiles.html>`_,
    note the updates introduced in the
    `attributes section <command_line.html#rtps-attributes-dealing-with-discovery-services>`_.

+   **servers** is a list of servers that the discovery-server must create and setup. Must contain at least a **server**
    tag. Each server admits the following attributes:

 -  **name** non-mandatory but advisable for debugging purposes.
 -  **prefix** server unique identifier. It's optional because it may be specified in the profile but using this
    attribute we can avoid generating server profiles that only differ in prefix.
 -  **profile_name** identifies the profile associated with this server is a mandatory one.
 -  **persist** specifies if the participant is a `SERVER <command_line.html#discoverysettings>`_) or a
    `BACKUP  <command_line.html#discoverysettings>`_.
 -  **creation_time** introduced for testing purposes specifies in seconds when a server must be created.
 -  **removal_time** introduced for testing purposes specifies in seconds when a server must be destroyed.

 Each server element admits the following tags:

 - **ListeningPorts** contains lists of locators where this server will listen for incoming client metatraffic.
 - **ServersList** contains at least one **RServer** tag that references the servers this one wants to link to.
   **RServer** only has a prefix attribute. Based on this prefix the discover-server parser would search for the
   corresponding server locators within the config file.
 - **publisher** introduced for testing purposes. Creates a dummy publisher characterized by *profile_name*,*topic*,
   *creation_time* and *removal_time*.
 - **subscriber** introduced for testing purposes. Creates a dummy publisher characterized by *profile_name*, *topic*,
   *creation_time* and *removal_time*.

+   **clients** introduced for testing purposes. It's a list of dummy clients that the discovery-server must create and
    setup. Must contain at least a **client** tag.

 Each client admits the following attributes:

 -  **name** non-mandatory but advisable for debugging purposes.
 -  **profile_name** identifies the profile associated with this server is a mandatory one.
 -  **server** specifies the prefix of the server we want to link to. This optional attribute saves us the nuisance
    of creating a **ServerList** (only if this client references a single server). Based on this prefix the
    discover-server
    parser would search for the corresponding server locators within the config file.
 -  **listening_port** specifies a physical port where to listen for incoming traffic. This attribute is mandatory in
    TCP transport (client wouldn't receive other clients traffic without it). When using the TCPv4 the format is:
    [XXX.XXX.XXX.XXX:]XXXX where the IP address is the client's WAN address that must be specified if we want the
    client to
    be reachable from outside a local NAT.
 -  **creation_time** introduced for testing purposes specifies in seconds when a client must be created.
 -  **removal_time** introduced for testing purposes specifies in seconds when a client must be destroyed.

 Each client element admits the following tags:

 - **ServersList** contains at least one **RServer** tag that references the servers this one wants to link to.
   **RServer** only has a prefix attribute. Based on this prefix the discover-server parser would search for the
   corresponding server locators within the config file.
 - **publisher** introduced for testing purposes. Creates a publisher characterized by *profile_name*, *topic*,
   *creation_time* and *removal_time*.
 - **subscriber** introduced for testing purposes. Creates a publisher characterized by *profile_name*, *topic*,
   *creation_time* and *removal_time*.

+   **types** is plainly the
    `Fast-RTPS types <https://eprosima-Fast-RTPS.readthedocs.io/en/latest/xmlprofiles.html#xml-dynamic-types>`_.
    It's introduced here for testing purposes to check how topic and type discovery info is handled by EDP.

+   **snapshots** contains **snapshot** tags. Whenever a discovery-server creates a participant (client or a server) it
    becomes its *listener* in the sense that all discovery info received by the participant is relayed to him. This
    reported discovery info is stored in a database. A *snapshot* is a *commit* of this database in a given time point.
    The **snapshots** element has a **file** attribute that must be filled with the filename of the XML results file if
    immediate validation is not desired (see `instructions <command_line.html#directions for use>`_).

    *snapshots* are useful because within is recorded how much information each participant is aware of. If a
    participant reported no info or incomplete one there is a discovery failure. When the discovery server has finished
    processing the config file (creating and destroying entities or taking snapshots) then all **snapshots** taken are
    checked. If any of them shows discovery info leakages discovery server returns -1 and the unsuccessful *snapshot* is
    logged to the standard error. Note:

 -  this validation can be avoided using the **file** attribute of the **snapshots** collection.
 -  there is a particular case that requires special treatment by using **snapshot**'s attribute **someone**.
    By default **someone** is true and no discovery info reported by any participant is regarded as a failure, yet in
    some tests, we want to validate the absence of discovery if settings are wrong so **someone** may be set false then.

 **snapshot**  tag has a single mandatory attribute **time** which specifies when the snapshot must be taken and an
 optional **someone** whose functionality is explained above. The text content of the tag is regarded as a
 description where the user can specify which event may require validation (participant creation or removal, etc...).

The `tests <tests.html>`_ are probably the best examples of the above XML definitions put into practice.