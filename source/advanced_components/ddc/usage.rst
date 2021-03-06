.. _ddc_usage:

==================
OneProvision Usage
==================

CLI Commands
============

This section covers the available commands of the ``oneprovision`` tool.

.. warning::

    Commands should be run as the ``oneadmin`` user on your frontend.

.. note::

    Additional CLI arguments ``--verbose/-d`` and ``--debug/-D`` (applicable for all commands of the ``oneprovision`` tool) provide additional levels of logging. Check :ref:`Logging Modes <ddc_usage_log>` for the detailed description.

Create
------

All deployment steps (create, provision, configuration) are covered by a single run of the command ``oneprovision create``. It's necessary to provide a :ref:`provision template <ddc_provision_template>` (with information about what to create, provision and how to configure the hosts). The OpenNebula provision ID is returned after successful provision.

Deployment of a new provision is a 4 step process:

- **Add**. OpenNebula infrastructure objects (cluster, hosts, datastores, networks) are created, but disabled for general use.
- :ref:`Provision <ddc_template>`. Resources are allocated on the remote provider (e.g. use the provider's API to get clean new hosts).
- :ref:`Configure <ddc_config>`. Resources are reconfigured for a particular use (e.g. install virtualization tools on new hosts).
- **Add**. OpenNebula virtual objects(images, marketplace apps, VM Templates, VNet templates, OneFlow service templates) are created.
- **Enable**. Ready-to-use resources are enabled in OpenNebula.

Parameters:

+---------------------------+----------------------------------------------------+-----------+
| Parameter                 | Description                                        | Mandatory |
+===========================+====================================================+===========+
| ``FILENAME``              | File with                                          | **YES**   |
|                           | :ref:`provision template <ddc_provision_template>` |           |
+---------------------------+----------------------------------------------------+-----------+
| ``--ping-retries`` number | Number of SSH connection retries (default: 10)     | NO        |
+---------------------------+----------------------------------------------------+-----------+
| ``--ping-timeout`` number | Seconds between each SSH retry (default: 20)       | NO        |
+---------------------------+----------------------------------------------------+-----------+
| ``--wait``                | Wait virtual objects to be ready in OpenNebula     | NO        |
+---------------------------+----------------------------------------------------+-----------+
| ``--wait-timeout`` number | Seconds to wait virtual objects (default: 60)      | NO        |
+---------------------------+----------------------------------------------------+-----------+
| ``--skip-provision``      | Skip hosts provision and configuration             | NO        |
+---------------------------+----------------------------------------------------+-----------+
| ``--skip-config``         | Skip hosts configuration                           | NO        |
+---------------------------+----------------------------------------------------+-----------+

Example:

.. prompt:: bash $ auto

    $ oneprovision create myprovision.yaml -d
    2018-11-27 11:32:03 INFO  : Creating provision objects
    WARNING: This operation can take tens of minutes. Please be patient.
    2018-11-27 11:32:05 INFO  : Deploying
    2018-11-27 11:34:42 INFO  : Monitoring hosts
    2018-11-27 11:34:46 INFO  : Checking working SSH connection
    2018-11-27 11:34:49 INFO  : Configuring hosts
    ID: 8fc831e6-9066-4c57-9ee4-4b11fea98f00

Validate
--------

The ``validate`` command checks the provided :ref:`provision template <ddc_provision_template>` is correct. Returns exit code 0 if the template is valid.

Parameters:

+--------------+----------------------------------------------------+-----------+
| Parameter    | Description                                        | Mandatory |
+==============+====================================================+===========+
| ``FILENAME`` | File with                                          | **YES**   |
|              | :ref:`provision template <ddc_provision_template>` |           |
+--------------+----------------------------------------------------+-----------+
| ``--dump``   | Show complete provision template on standard output| NO        |
+--------------+----------------------------------------------------+-----------+

Examples:

.. prompt:: bash $ auto

    $ oneprovision validate simple.yaml
    $ oneprovision validate simple.yaml --dump | head -4
    ---
    name: myprovision
    playbook: default

List
----

The ``list`` command lists all provisions.

.. prompt:: bash $ auto

    $ oneprovision list
                                      ID NAME                      CLUSTERS HOSTS VNETS DATASTORES STAT
    8fc831e6-9066-4c57-9ee4-4b11fea98f00 myprovision                      1     1     1          2 configured

Show
----

The ``show`` command lists all provisioned objects of the particular provision.

Parameters:

+------------------+---------------------+-----------+
| Parameter        | Description         | Mandatory |
+==================+=====================+===========+
| ``provision ID`` | Valid provision ID  | **YES**   |
+------------------+---------------------+-----------+
| ``--csv``        | Show output as CSV  | NO        |
+------------------+---------------------+-----------+

Examples:

.. prompt:: bash $ auto

    $ oneprovision show 8fc831e6-9066-4c57-9ee4-4b11fea98f00
    PROVISION  INFORMATION
    ID                : 8fc831e6-9066-4c57-9ee4-4b11fea98f00
    NAME              : myprovision
    STATUS            : configured

    CLUSTERS
    184

    HOSTS
    766

    VNETS
    135

    DATASTORES
    318
    319

Configure
---------

.. warning::

    It's important to understand that the (re)configuration can happen only on physical hosts that aren't actively used (e.g., no virtual machines running on the host) and with the operating system/services configuration untouched since the last (re)configuration. It's not possible to (re)configure the host with a manually modified OS/services configuration. Also it's not possible to fix a seriously broken host. Such a situation needs to be handled manually by an experienced systems administrator.

The ``configure`` command offlines the OpenNebula hosts (making them unavailable to users) and triggers the deployment configuration phase. If the provision was already successfully configured before, the argument ``--force`` needs to be used. After successful configuration, the OpenNebula hosts are re-enabled.

Parameters:

+------------------+-----------------------+-----------+
| Parameter        | Description           | Mandatory |
+==================+=======================+===========+
| ``provision ID`` | Valid provision ID    | **YES**   |
+------------------+-----------------------+-----------+
| ``--force``      | Force reconfiguration | NO        |
+------------------+-----------------------+-----------+

Examples:

.. prompt:: bash $ auto

    $ oneprovision configure 8fc831e6-9066-4c57-9ee4-4b11fea98f00 -d
    ERROR: Hosts are already configured

    $ oneprovision configure 8fc831e6-9066-4c57-9ee4-4b11fea98f00 -d --force
    2018-11-27 12:43:31 INFO  : Checking working SSH connection
    2018-11-27 12:43:34 INFO  : Configuring hosts

Delete
------

The ``delete`` command releases the physical resources to the remote provider and deletes the provisioned OpenNebula objects.

.. prompt:: bash $ auto

    $ oneprovision delete 8fc831e6-9066-4c57-9ee4-4b11fea98f00 -d
    2018-11-27 12:45:21 INFO  : Deleting provision 8fc831e6-9066-4c57-9ee4-4b11fea98f00
    2018-11-27 12:45:21 INFO  : Undeploying hosts
    2018-11-27 12:45:23 INFO  : Deleting provision objects

Only provisions with no running VMs or images in the datastores can be easily deleted. You can force ``oneprovision`` to terminate VMs running on provisioned hosts and delete all images in the datastores with the ``--cleanup`` parameter.

Parameters:

+------------------+---------------------------------------------+-----------+
| Parameter        | Description                                 | Mandatory |
+==================+=============================================+===========+
| ``provision ID`` | Valid provision ID                          | **YES**   |
+------------------+---------------------------------------------+-----------+
| ``--delete-all`` | Delete all contained objects (VMs, images)  | NO        |
+------------------+---------------------------------------------+-----------+

Examples:

.. prompt:: bash $ auto

    $ oneprovision delete 8fc831e6-9066-4c57-9ee4-4b11fea98f00 -d
    2018-11-27 13:44:40 INFO  : Deleting provision 8fc831e6-9066-4c57-9ee4-4b11fea98f00
    ERROR: Provision with running VMs can't be deleted

.. prompt:: bash $ auto

    $ oneprovision delete 8fc831e6-9066-4c57-9ee4-4b11fea98f00 -d --cleanup
    2018-11-27 13:56:39 INFO  : Deleting provision 8fc831e6-9066-4c57-9ee4-4b11fea98f00
    2018-11-27 13:56:44 INFO  : Undeploying hosts
    2018-11-27 13:56:51 INFO  : Deleting provision objects

Host Management
---------------

Individual hosts from the provision can be managed by the ``oneprovision host`` subcommands.

List
^^^^

The ``host list`` command lists all provisioned hosts, and ``host top`` command periodically refreshes the list until it's terminated.

.. prompt:: bash $ auto

    $ oneprovision host list
      ID NAME            CLUSTER   RVM PROVIDER VM_MAD   STAT
     766 147.75.33.113   conf-prov   0 packet   kvm      on

    $ oneprovision host top

Host Power Off
^^^^^^^^^^^^^^

The ``host poweroff`` command offlines the host in OpenNebula (making it unavailable to users) and powers off the physical resource.

.. prompt:: bash $ auto

    $ oneprovision host poweroff 766 -d
    2018-11-27 12:21:40 INFO  : Powering off host: 766
    HOST 766: disabled

Host Resume
^^^^^^^^^^^

The ``host resume`` command powers on the physical resource, and re-enables the OpenNebula host (making it available again to users).

.. prompt:: bash $ auto

    $ oneprovision host resume 766 -d
    2018-11-27 12:22:57 INFO  : Resuming host: 766
    HOST 766: enabled

Host Reboot
^^^^^^^^^^^

The ``host reboot`` command offlines the OpenNebula host (making it unavailable for users), cleanly reboots the physical resource and re-enables the OpenNebula host (making it available again for users after successful OpenNebula host monitoring).

.. prompt:: bash $ auto

    $ oneprovision host reboot 766 -d
    2018-11-27 12:25:10 INFO  : Rebooting host: 766
    HOST 766: enabled

Host Reset
^^^^^^^^^^

The ``host reboot --hard`` command offlines the OpenNebula host (making it unavailable for users), resets the physical resource and re-enables the OpenNebula host.

.. prompt:: bash $ auto

    $ oneprovision host reboot --hard 766 -d
    2018-11-27 12:27:55 INFO  : Resetting host: 766
    HOST 766: enabled

Host SSH
^^^^^^^^

The ``host ssh`` command opens an interactive SSH connection on the physical resource to the (privileged) remote user used for configuration.

.. prompt:: bash $ auto

    $ oneprovision host ssh 766
    Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-20-generic x86_64)

     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage

    Last login: Tue Nov 27 10:37:42 2018 from 213.175.39.66
    root@myprovision-host1:~#

An additional argument may specify a command to run on the remote side.

.. prompt:: bash $ auto

    $ oneprovision host ssh 766 hostname
    ip-172-30-4-47.ec2.internal

Host Configure
^^^^^^^^^^^^^^

The physical host :ref:`configuration <ddc_config>` is part of the initial deployment, but it's possible to trigger the reconfiguration on provisioned hosts anytime later (e.g. when a configured service stopped running, or the host needs to be reconfigured differently). Based on the initially-provided connection and configuration parameters in the :ref:`provision template <ddc_provision_template_configuration>`, the configuration steps are applied again.

The ``host configure`` command offlines the OpenNebula host (making it unavailable for users) and re-triggers the deployment configuration phase. If the provisioned the host was already successfully configured, the argument ``--force`` needs to be used. After successful configuration, the OpenNebula host is re-enabled.

.. prompt:: bash $ auto

    $ oneprovision host configure 766 -d
    ERROR: Hosts are already configured

    $ oneprovision host configure 766 -d --force
    2018-11-27 12:36:18 INFO  : Checking working SSH connection
    2018-11-27 12:36:21 INFO  : Configuring hosts
    HOST 766:

Cluster Management
------------------

Individual clusters from the provision can be managed by the ``oneprovision cluster`` subcommands.

Cluster List
^^^^^^^^^^^^

The ``oneprovision cluster list`` command lists all provisioned clusters.

.. prompt:: bash $ auto

    $ oneprovision cluster list
       ID NAME                      HOSTS VNETS DATASTORES
      184 myprovision                   1     1          2

Cluster Delete
^^^^^^^^^^^^^^

The ``oneprovision cluster delete`` command deletes the cluster.

.. prompt:: bash $ auto

    $ oneprovision cluster delete 184 -d
    CLUSTER 184: deleted

The cluster needs to have no datastores, virtual networks, or hosts. Please see the ``oneprovision delete`` command to remove all the related objects.

.. prompt:: bash $ auto

    $ oneprovision cluster delete 184 -d
    ERROR: [one.cluster.delete] Cannot delete cluster. Cluster 185 is not empty, it contains 1 datastores.


Datastore Management
--------------------

Individual datastores from the provision can be managed by the ``oneprovision datastore`` subcommands.

Datastore List
^^^^^^^^^^^^^^

The ``oneprovision datastore list`` command lists all provisioned datastores.

.. prompt:: bash $ auto

    $ oneprovision datastore list
      ID NAME                SIZE AVAIL CLUSTERS     IMAGES TYPE DS      PROVIDER TM      STA
     318 conf-provisio     271.1G 7%    184               0 img  fs      packet   ssh     on
     319 conf-provisio         0M -     184               0 sys  -       packet   ssh     on

Datastore Delete
^^^^^^^^^^^^^^^^

The ``oneprovision datastore delete`` command deletes the datastore.

.. prompt:: bash $ auto

    $ oneprovision datastore delete 318 -d
    2018-11-27 13:01:08 INFO  : Deleting datastore 318
    DATASTORE 318: deleted

Virtual Networks Management
---------------------------

Individual virtual networks from the provision can be managed by the ``oneprovision vnet`` subcommands.

Vnet List
^^^^^^^^^

The ``oneprovision vnet list`` command lists all virtual networks.

.. prompt:: bash $ auto

    $ oneprovision vnet list
      ID USER            GROUP        NAME                CLUSTERS   BRIDGE   PROVIDER LEASES
     136 oneadmin        oneadmin     myprovision-hostonl 184        br0      packet        0

Vnet Delete
^^^^^^^^^^^

The ``oneprovision vnet delete`` command deletes the virtual network.

.. prompt:: bash $ auto

    $ oneprovision vnet delete 136 -d
    2018-11-27 13:02:08 INFO  : Deleting vnet 136
    VNET 136: deleted

.. _ddc_usage_log:

Logging Modes
=============

The ``oneprovision`` tool in the default mode returns only minimal requested output (e.g., provision IDs after create), or errors. Operations on the remote providers or the host configuration are complicated and time-consuming tasks. For better insight and for debugging purposes there are 2 logging modes available, providing more information on the standard error output.

* **verbose** (``--verbose/-d``). Only the main steps are logged.

Example:

.. prompt:: bash $ auto

    $ oneprovision host reboot 766 -d
    2018-11-27 12:58:32 INFO  : Rebooting host: 766
    HOST 766: disabled

* **debug** (``--debug/-D``). All internal actions, including generated configurations with **sensitive data**, are logged.

Example:

.. prompt:: bash $ auto

    $ oneprovision host reboot 766 -D
    2018-11-27 12:59:02 DEBUG : Offlining OpenNebula host: 766
    2018-11-27 12:59:02 INFO  : Rebooting host: 766
    2018-11-27 12:59:02 DEBUG : Command run: /var/lib/one/remotes/pm/packet/reboot fa65c328-57c3-4890-831e-172c9d730b04 147.75.33.113 767 147.75.33.113
    2018-11-27 12:59:09 DEBUG : Command succeeded
    2018-11-27 12:59:09 DEBUG : Enabling OpenNebula host: 766

Running Modes
=============

The ``oneprovision`` tool is ready to deal with common problems during execution. It's able to retry some actions or clean up an incomplete provision. Depending on where and how the tool is used, it offers 2 running modes:

* **interactive** (default). If the unexpected condition appears, the user is asked how to continue.

Example:

.. prompt:: bash $ auto

    $ oneprovision host poweroff 0
    ERROR: Driver action '/var/lib/one/remotes/pm/packet/shutdown' failed
    Shutdown of Packet host 147.75.33.123 failed due to "{"errors"=>["Device must be powered on"]}"
    1. quit
    2. retry
    3. skip
    Choose failover method: 2
    ERROR: Driver action '/var/lib/one/remotes/pm/packet/shutdown' failed
    Shutdown of Packet host 147.75.33.123 failed due to "{"errors"=>["Device must be powered on"]}"
    1. quit
    2. retry
    3. skip
    Choose failover method: 1
    $

* **batch** (``--batch``). It's expected to be run from scripts. No questions are asked, and the tool tries to deal automatically with the problem according to the failover method specified as a command line parameter:

+-------------------------+------------------------------------------------+
| Parameter               | Description                                    |
+=========================+================================================+
| ``--fail-quit``         | Set batch failover mode to quit (default)      |
+-------------------------+------------------------------------------------+
| ``--fail-retry`` number | Set batch failover mode to number of retries   |
+-------------------------+------------------------------------------------+
| ``--fail-cleanup``      | Set batch failover mode to clean up and quit   |
+-------------------------+------------------------------------------------+
| ``--fail-skip``         | Set batch failover mode to skip failing part   |
+-------------------------+------------------------------------------------+

Example of automatic retry:

.. prompt:: bash $ auto

    $ oneprovision host poweroff 0 --batch --fail-retry 2
    ERROR: Driver action '/var/lib/one/remotes/pm/packet/shutdown' failed
    Shutdown of Packet host 147.75.33.123 failed due to "{"errors"=>["Device must be powered on"]}"
    ERROR: Driver action '/var/lib/one/remotes/pm/packet/shutdown' failed
    Shutdown of Packet host 147.75.33.123 failed due to "{"errors"=>["Device must be powered on"]}"
    ERROR: Driver action '/var/lib/one/remotes/pm/packet/shutdown' failed
    Shutdown of Packet host 147.75.33.123 failed due to "{"errors"=>["Device must be powered on"]}"

Example of non-interactive provision with automatic clean up in case of failure:

.. prompt:: bash $ auto

    $ oneprovision create simple.yaml -d --batch --fail-cleanup
    2018-11-27 13:48:53 INFO  : Creating provision objects
    WARNING: This operation can take tens of minutes. Please be patient.
    2018-11-27 13:48:54 INFO  : Deploying
    2018-11-27 13:51:32 INFO  : Monitoring hosts
    2018-11-27 13:51:36 INFO  : Checking working SSH connection
    2018-11-27 13:51:38 INFO  : Configuring hosts
    2018-11-27 13:52:02 WARN  : Command FAILED (code=2): ANSIBLE_CONFIG=/tmp/d20181127-11335-ktlqrb/ansible.cfg ansible-playbook --ssh-common-args='-o UserKnownHostsFile=/dev/null' -i /tmp/d20181127-11335-ktlqrb/inventory -i /usr/share/one/oneprovision/ansible/inventories/default/ /usr/share/one/oneprovision/ansible/default.yml
    ERROR: Configuration failed
    - 147.75.33.125   : TASK[opennebula-repository : Add OpenNebula repository (Ubuntu)] - MODULE FAILURE
    2018-11-27 13:52:02 INFO  : Deleting provision 18e85ef4-b29f-4391-8d89-c72702ede54e
    2018-11-27 13:52:02 INFO  : Undeploying hosts
    2018-11-27 13:52:05 INFO  : Deleting provision objects
