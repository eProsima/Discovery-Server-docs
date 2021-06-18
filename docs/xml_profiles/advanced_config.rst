.. _advanced_config_file:

Advanced XML configuration
**************************

This XML configures an advanced Discovery Server topology in which multiple servers, with publishers and subscribers,
have multiple clients with publishers and subscribers in turn.
This example also shows how to launch participants and endpoints during the execution of the Discovery Server tool.
Under the snapshots tag are specified the times at which a snapshot of the discovery state will be taken.

.. literalinclude:: ../02-resources/tests/test_14_disposals_remote_servers.xml
    :language: xml
    :linenos:
