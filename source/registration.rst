Registration of services in GOCDB
---------------------------------

Site endpoints must be registered in `EGI Configuration Management Database (GOCDB) <https://goc.egi.eu>`_. If you are creating a new site for your cloud services, check the `PROC09 Resource Centre Registration and Certification <https://wiki.egi.eu/wiki/PROC09>`_ procedure. Services can also coexist within an existing (grid) site.

These are the expected services for a working site:

* **Site-BDII**. This service collects and publishes site's data for the Information System. Existing sites should already have this registered.

* **eu.egi.cloud.accounting**. Register here the host sending the records to the accounting repository (executing SSM send).

* **eu.egi.cloud.vm-metadata.vmcatcher** for the VMI replication mechanism. Register here the host providing the replication (i.e. the host with cloudkeeper installation)

If offering OCCI interface, sites should register:

* **eu.egi.cloud.vm-management.occi** for the OCCI endpoint offered by the site. The endpoint URL must follow this syntax:

    ::

        https://hostname:port/?image=<image_name>&resource=<resource_name>

    where ``<image_name>`` and ``<resource_name>`` cannot contain spaces. These attributes map to os_tpl and resource_tpl respectively and will be the ones used for monitoring purposes.

If offering native OpenStack access (nova), register:

* **org.openstack.nova** for the Nova endpoint of the site.  The endpoint URL must contain the Keystone URL.

If offering native OpenStack access (swift), register:

* **org.openstack.swift** for the swift endpoint of the site. The endpoint URL field must contain Keystone URL:

    ::

        https://hostname:port/url

.. TODO: CLARIFY IF THIS IS TRUE, not bringing any value atm
    Site should also declare the following properties using the *Site Extension Properties* feature:
      #. Max number of virtual cores for VM with parameter name: ``cloud_max_cores4VM``
      #. Max amount of RAM for VM with parameter name: ``cloud_max_RAM4VM`` using the format: value+unit, e.g. "16GB".
      #. Max amount of storage that could be mounted in a VM with parameter name: ``cloud_max_storage4VM`` using the format: value+unit, e.g. "16GB".
