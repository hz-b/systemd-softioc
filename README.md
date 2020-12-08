This repository is derived from: https://github.com/epicsdeb/sysv-rc-softioc

This repository contains a systemd style initialization script for a softioc using procServ.

= Usage =

Use "install.sh" (with sudo access) to install this package. By default, the source files are installed in /usr/local/systemd-softioc and a symlink "/usr/bin/manage-iocs -> /usr/local/systemd-softioc/manage-iocs" is created. Since this package is dependant on procServ, install.sh will try its best to get procServ's github repository, then build it and install the executable to /usr/bin/. 

Source files:

    epics-softioc.conf: this is where you can customize a few things; 

    manage-iocs: the main script; 

    library.sh: functions used in the script manage-iocs;

The following are provided as information:

    config.example: a template showing how to create a file named 'config' for an IOC instance/application; 

    softioc-example.service: the unit file for the IOC named "example"

    README.md (this file)

== manage-iocs ==

The script 'manage-iocs' has several sub-commands
to help in managing softiocs.  See its manpage for
details.

$ manage-iocs help

    Usage: manage-iocs [-v] [-x] cmd

    Available commands:

      help            - display this message

      report [ioc]    - Show config of all/an IOC

      status          - Check if IOCs are running

      nextport        - Find the next unused procServ port

      install <ioc>   - Create /etc/systemd/system/softioc-[ioc].service

      uninstall <ioc> - Remove /etc/systemd/system/softioc-[ioc].service

      start <ioc>     - Start the IOC <ioc>

      stop <ioc>      - Stop the IOC <ioc>

      startall        - Start all IOCs installed for this system

      stopall         - Stop all IOCs installed for this system

== Initial Setup ==

1) Choose a location for all softIoc instances / applications

This will be a single directory which will contain a subdirectory for each softIoc instance.  A good choice would be '/epics/iocs' (or /opt/epics/iocs). Set this path in 'epics-softioc.conf':

    #base directory for IOC instances

    IOCPATH=/epics/iocs

    #IOCPATH=/epics/iocs:/opt/epics/iocs

2) Create a Unix group softIocs

Altough each softIoc can run as a separate user/group, it is recommended using a single username 'softioc' (or all softIoc users be in the same group).  This provides a nice division and allows Channel Access security to distinguish all the instances on a given machine.

    #useradd softioc

    (#groupadd softioc)

3) Create the directory for softIoc instances

    #mkdir -p /epics

    #chgrp softioc /epics

    #chmod g+s /epics

    #mkdir -p /epics/iocs

    #chmod g+ws /epics/iocs

4) (optional) Install Conserver

Conserver is a process which connects to the telnet servers provided by all the softIocs on a host.  It then allows (secure) remote access and continuous logging.

    #apt-get install conserver-server

    Edit /etc/conserver/conserver.cf to include the following line.

    default softioc { type host; host localhost;}

    #include /etc/conserver/iocs.cf

See the conserver documentation for information on access control.

Note: Conserver uses the tcpd for authentication.

== Per-Instance Setup ==

1) Choose a name

Try to pick something more creative than example1 ;)

2) Create an IOC instance directory

    #mkdir /epics/iocs/example1

    (option: Create a user: #useradd -c 'softioc' -d /epics/iocs/example1 -g softioc -N example1

3) Create and configure the instance's config file

Each instance must have a name and a port number (for procServ).  It is also a good idea to include the server's hostname to prevent accidentally running it on the wrong server.

    #cat << EOF > /epics/iocs/example1/config
    NAME=example1

    PORT=4051

    HOST=myserver

    #USER=softioc
    EOF

Note: The manage-iocs script has a subcommand 'nextport' which tries to pick an unused port number from procServ.

If given, the hostname is required to match the result of running 'hostname -s' or 'hostname -f'.

See config.example for allowed items in a IOC config file.

4) Create/replace the systemd unit file (/etc/systemd/system/softioc-example.service). The IOC will automatically startup after the host server boots up.

    #manage-iocs install example1

    (# manage-iocs uninstall example1: this removes the unit file /etc/systemd/system/softioc-example.service)

5) Manually starting the instance

    #manage-iocs start example1

6) Telnet to the IOC example1's EPICS shell

    $ telnet localhost 4051


== IOC runtime environment ==

The following environment variables are automatically set before procServ
is started.  They may be overridden in the IOC shell.

    IOCNAME - The IOC name.

    USER - The system user account name which runs the IOC.

    HOSTNAME - The short name of the host running the IOC.

    TOP - The absolute path of the base IOC directory.  Contains the 'config' file

    EPICS_HOST_ARCH - Host target name (eg. linux-x86)
