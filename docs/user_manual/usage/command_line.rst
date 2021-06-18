Usage
#####

Each setting in *Fast DDS* can be configured through XML profiles. XML profiles allows to avoid tiresome
hard-coded settings within applications sources using XML configuration files.
The *Fast DDS* XML schema was duly updated to accommodate the new Discovery Server tool settings.
Please refer to :ref:`config_files` for more information on the new Discovery Server xml configuration files.
Moreover, an XML configuration file example can be found :ref:`here <basic_config_file>`.

The discovery server binary (named after the pattern ``discovery-server-X.X.X(d)`` where ``X.X.X`` is the version
number and the optional *d* denotes a debug builds) is set up from one XML profiles files passed as command-line
arguments.
To have the tool accessible in the terminal session it is necessary to source the setup file.

-   **Linux**

    .. code-block:: bash

        $ source <path/to/discovery-server-ws>/discovery-server-ws/install/setup.bash
        $ discovery-server-X.X.X(d) config_file.xml

-   **Windows**

    .. code-block:: batch

        > . <path\to\discovery-server-ws>\discovery-server-ws\install\setup.bat
        > discovery-server-X.X.X(d).exe config_file.xml
