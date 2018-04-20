.. EGI Federated Cloud Integration documentation master file, created by
   sphinx-quickstart on Thu Apr 19 16:18:33 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

EGI Federated Cloud Integration
===============================

.. toctree::
   :maxdepth: 2
   :caption: Contents:


Common prerequirements and documentation
----------------------------------------

General minimal requirements are:

* Hardware requirements greatly depend on your cloud infrastructure, EGI components in general do lightweigth operations by interacting with your services APIs. `cloudkeeper` requires enough disk space to download and convert images before uploading into your local catalogue. The number and size of images which will be downloaded depends on the communities you plan to support. For the piloting VO `fedcloud.egi.eu`, 100GB of disk should be enough.

* Servers need to authenticate each other in the EGI Federated Cloud context using X.509 certificates. So a Resource Centre should be able to obtain server certificates for some services.

* User and research communities are called Virtual Organisations (VO). Resource Centres are expected to join:

  - ``ops`` and ``dteam`` VOs, used for operational purposes as per RC OLA

  - a community-VO that supports EGI users (e.g. ``fedcloud.egi.eu`` for piloting)

* EGI provides packages for the following operating systems (others may work but we are not providing packages):

  - CentOS 7 (and in general RHEL-compatible)

  - Ubuntu 16.04(and in general Debian-based)

.. TODO:
   In order to configure Virtual Organisations and private image lists, please consider the following guides to:
   * [[HOWTO16|enable a Virtual Organisation on a EGI Federated Cloud site]]
   * [https://wiki.appdb.egi.eu/main:faq:how_to_get_access_to_vo-wide_image_lists get access to VO wide image lists]
   * [https://wiki.appdb.egi.eu/main:faq:how_to_subscribe_to_a_private_image_list_using_the_vmcatcher subscribe to a private image list]

Integration
-----------

You can follow dedicated integration guides for each cloud management framework:

.. toctree::
   :maxdepth: 1

   opennebula
   openstack

.. From MAN10
   * OpenNebula
   ** [[Federated_Cloud_OpenNebula_guide | OpenNebula v5.2.x/v5.4.x]]
   * OpenStack:
   ** [[Federated_Cloud_Mitaka_guide | OpenStack Mitaka]] -- LTS under Ubuntu 16.04 (otherwise EOL)
   ** [[Federated_Cloud_Ocata_guide | OpenStack Ocata]]
   ** [[Federated_Cloud_Pike_guide | OpenStack Pike]]
   ** [[Federated_Cloud_Queens_guide | OpenStack Queens]] -- ''Support for Keystone-VOMS is not available''
   * OpenStack EOL'd versions ('''not recommended in production''')
   ** [[Federated_Cloud_Newton_guide | OpenStack Newton]]
   See http://releases.openstack.org/ for more details on the OpenStack releases.

Registration of services in GOCDB
---------------------------------

Site cloud services must be registered in `EGI Configuration Management Database (GOCDB) <https://goc.egi.eu>`_. If you are creating a new site for your cloud services, check the `PROC09 Resource Centre Registration and Certification <https://wiki.egi.eu/wiki/PROC09>`_ procedure. Services can also coexist within an existing (grid) site.

These are the expected services for a working site:

* **Site-BDII**. This service collects and publishes site's data for the Information System. Existing sites should already have this registered.

* **eu.egi.cloud.accounting**. Register here the host sending the records to the accounting repository (executing SSM send).

* **eu.egi.cloud.vm-metadata.vmcatcher** for the VMI replication mechanism. Register here the host providing the replication.

If offering OCCI interface, sites should register:

* **eu.egi.cloud.vm-management.occi** for the OCCI endpoint offered by the site. Please note the special endpoint URL syntax described at `GOCDB usage in FedCloud <https://wiki.egi.eu/wiki/Federated_Cloud_Technology#eu.egi.cloud.vm-management.occi>`_

If offering native OpenStack access, register:

* **org.openstack.nova** for the Nova endpoint of the site.  Please note the special endpoint URL syntax described at [[Federated_Cloud_Technology#org.openstack.nova|GOCDB usage in FedCloud]]

* **TODO** (swift)

.. TODO: CLARIFY IF THIS IS TRUE, not bringing any value atm

Site should also declare the following properties using the *Site Extension Properties* feature:

  #. Max number of virtual cores for VM with parameter name: ``cloud_max_cores4VM``

  #. Max amount of RAM for VM with parameter name: ``cloud_max_RAM4VM`` using the format: value+unit, e.g. "16GB".

  #. Max amount of storage that could be mounted in a VM with parameter name: ``cloud_max_storage4VM`` using the format: value+unit, e.g. "16GB".

Installation Validation
-----------------------

.. TODO: quite outdated

Once the site services are registered in GOCDB (and flagged as "monitored") they will appear in the EGI service monitoring tools. EGI will check the status of the services (see `Infrastructure Status <https://wiki.egi.eu/wiki/Federated_Cloud_infrastructure_status>`_ for details). Check if your services are present in the EGI service monitoring tools and passing the tests; if you experience any issues (services not shown, services are not OK...) please contact back EGI Operations or your reference Resource Infrastructure.

Extra checks for your installation:

* Check in `ARGO-Mon2 <https://argo-mon2.egi.eu/nagios>`_ that your services are listed and are passing the tests. If all the tests are OK, your installation is already in good shape.

* Check that you are publishing cloud information in your site BDII:

::

    ldapsearch -x -h <site bdii host> -p 2170 -b Glue2GroupID=cloud,Glue2DomainID=<your site name>,o=glue

* Check that all the images listed in the `AppDB page for fedlcoud.egi.eu VO <https://appdb.egi.eu/store/vo/fedcloud.egi.eu>`_ are listed in your BDII. This sample query will return all the template IDs registered in your BDII:

::

    ldapsearch -x -h <site bdii host> -p 2170 -b Glue2GroupID=cloud,Glue2DomainID=<your site name>,o=glue objectClass=GLUE2ApplicationEnvironment GLUE2ApplicationEnvironmentRepository

* Try to start one of those images in your cloud. You can do it with `onetemplate instantiate` or OCCI commands, the result should be the same.

* Execute the `site certification manual tests <https://wiki.egi.eu/wiki/HOWTO04_Site_Certification_Manual_tests#Check_the_functionality_of_the_cloud_elements>`_ against your endpoints.

* Check in the `accounting portal <http://accounting-devel.egi.eu/cloud.php>`_ that your site is listed and the values reported look consistent with the usage of your site.
