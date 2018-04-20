OpenStack Ocata
```````````````

Integration with FedCloud requires a *working OpenStack installation* as a pre-requirement (see http://docs.openstack.org/ for details). This manual provides information on how to set up a Resource Centre providing cloud resources in the EGI infrastructure, using **OpenStack Ocata**. EGI expect the following OpenStack services to be available and accessible from outside the site:

* Keystone

* Nova

* Cinder

* Glance

* Neutron

* Swift (if providing Object Storage)

FedCloud components are distributed through `CMD (Cloud Middleware Distribution) <https://wiki.egi.eu/wiki/EGI_Cloud_Middleware_Distribution>`_ or docker container images available in dockerhub. These docker containers come pre-packaged and ready to use in the EGI FedCloud Appliance so you do not need to install any extra components on your site but just run a VM and configure it approprietely to interact with your services.

The integration is performed by a set of EGI components that interact with the OpenStack services APIs:

.. image:: /_static/OpenStackSite.png

* **cASO** collects accounting data from OpenStack and uses **SSM** to send the records to the central accounting database on the EGI Accounting service (APEL)

* **cloud-info-provider** registers the RC configuration and description through the EGI Information System to facilitate service discovery

* **cloudkeeper** (and **cloudkeeper-os**) synchronises with `EGI AppDB <https://appdb.egi.eu/browse/cloud>`_  so new or updated images can be provided by the RC to user communities (VO).

Not all EGI components need to share the same credentials. They are individually configured, you can use different credentials and permissions if desired.

Authentication by EGI users into your system is performed by configuring the native OpenID Connect support of Keystone. Support for legacy VOs using VOMS requires the installation of the **Keystone-VOMS Authorization plugin** to  allow users with a valid VOMS proxy to obtain tokes to access your OpenStack deployment.

Optionally, **ooi (OpenStack OCCI Interface)** translates between OpenStack API and OCCI.


.. TODO
   PORTS?


EGI AAI
:::::::

OpenID Connect Support
''''''''''''''''''''''

TO IMPORT FROM AAI GUIDE


VOMS Support
''''''''''''

Support for authenticating users with X.509 certificates with VOMS extensions is achieved with Keystone-VOMS extension. Documentation is available at https://keystone-voms.readthedocs.io/en/stable-ocata/

Notes:

* **You need a host certificate from a recognised CA for your keystone server**.

* Take into account that using keystone-voms plugin will **enforce the use of https for your Keystone service**, you will need to update your URLs in the configuration of your services if your current installation is not using https:

  * you will probably need to include your CA to your system's CA bundle to avoid certificate validation issues: Check the `Federated Cloud OpenStack Client guide <https://wiki.egi.eu/wiki/Federated_Cloud_APIs_and_SDKs#CA_CertificatesCheck>`_ on how to do it.
  * replace http with https in ``auth_[protocol|uri|url]`` and ``auth_[host|uri|url]`` in the nova, cinder, glance and neutron config files (``/etc/nova/nova.conf``, ``/etc/nova/api-paste.ini``, ``/etc/neutron/neutron.conf``, ``/etc/neutron/api-paste.ini``, ``/etc/neutron/metadata_agent.ini``, ``/etc/cinder/cinder.conf``, ``/etc/cinder/api-paste.ini``, ``/etc/glance/glance-api.conf``, ``/etc/glance/glance-registry.conf``, ``/etc/glance/glance-cache.conf``) and any other service that needs to check keystone tokens.

  * Update the URLs of the services directly in the database:

::

    mysql> use keystone;
    mysql> update endpoint set url="https://<keystone-host>:5000/v2.0" where url="http://<keystone-host>:5000/v2.0";
    mysql> update endpoint set url="https://<keystone-host>:35357/v2.0" where url="http://<keystone-host>:35357/v2.0";

* Most sites should enable the ``autocreate_users`` option in the ``[voms]`` section of `Keystone-VOMS configuration <https://keystone-voms.readthedocs.org/en/latest/configuration.html>`_. This will enable new users to be automatically created in your local keystone the first time they login into your site.

* if (and only if) you need to configure the Per-User Subproxy (PUSP) feature, please follow the `specific guide <https://wiki.egi.eu/wiki/Long-tail_of_science_-_information_for_providers#Instructions_for_OpenStack_providers>`_.

EGI VM Management (optional)
::::::::::::::::::::::::::::

Follow the `installation and configuration manual of ooi <http://ooi.readthedocs.org/en/stable/index.html>`_.

.. TODO: packages?

Once the OCCI interface is installed, you should register it on your installation (adapt the region and URL to your deployment), e.g.:

::

    $ openstack service create --name occi --description "OCCI Interface" occi
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | OCCI Interface                   |
    | enabled     | True                             |
    | id          | 6dfd6a56c9a6456b84e8c86038e58f56 |
    | name        | occi                             |
    | type        | occi                             |
    +-------------+----------------------------------+

    $ openstack endpoint create --region RegionOne occi --publicurl http://172.16.4.70:8787/occi1.1

    +-------------+----------------------------------+
    |   Property  |              Value               |
    +-------------+----------------------------------+
    | description |           OCCI service           |
    |      id     | 8e6de5d0d7624584bed6bec9bef7c9e0 |
    |     name    |             occi_api             |
    |     type    |               occi               |
    +-------------+----------------------------------+


Other components
::::::::::::::::

There are two options to install the remaining Accounting, Information System and Image Management components:

* Using the EGI FedCloud Appliance (recommended), which uses docker containers to bundle an OpenStack deployment of the corresponding services

* Using individual components.

Follow the guides below according to your preferences

FedCloud Appliance
''''''''''''''''''

The EGI FedCloud Appliance is available at `AppDB <https://appdb.egi.eu/store/vappliance/fedcloud.integration.appliance.openstack>`_ as an OVA file. You can easily extract the VMDK disk by untaring and optionally converting it to your preferred format with qemu-img:

::

    # get image and extract VMDK
    curl https://cephrgw01.ifca.es:8080/swift/v1/egi_endorsed_vas/FedCloud-Appliance.Ubuntu.16.04-2017.08.09.ova | \
           tar x FedCloud-Appliance.Ubuntu.16.04-2017.08.09-disk001.vmdk
    # convert to qcow2
    qemu-img convert -O qcow2 FedCloud-Appliance.Ubuntu.16.04-2017.08.09-disk001.vmdk fedcloud-appliance.qcow2

The VM running at your OpenStack must:

* Be accessible via public IP with port 2170 open for external connections.

* Have a host certificate to send the accounting information to the accounting repository. DN of the host certificate must be registered in GOCDB service type eu.egi.cloud.accounting. The host certificate and key in PEM format are expected in /etc/grid-security/hostcert.pem and /etc/grid-security/hostkey.pem respectively.

* Have enough disk space for handling the VM image replication (~ 100GB for `fedcloud.egi.eu` VO). By default these are stored at /image_data. You can mount a volume at that location.

EGI Accounting
~~~~~~~~~~~~~~

There are two different processes handling the accounting integration:

* cASO, which connects to the OpenStack deployment to get the usage information, and,

* ssmsend, which sends that usage information to the central EGI accounting repository.

They are run by cron every hour (cASO) and every six hours (ssmsend).

`cASO configuration <http://caso.readthedocs.org/en/latest/configuration.html>`_ is stored at  ``/etc/caso/caso.conf``. Most default values are ok, but you must set:

* ``site_name`` (line 12)

* ``projects`` (line 20)

* credentials to access the accounting data (lines 28-47, more options also available). Check the `cASO documentation <http://caso.readthedocs.org/en/latest/configuration.html#openstack-configuration>`_ for the expected permissions of the user configured here.

The cron job will use the voms mapping file at ``/etc/voms.json``.

cASO will write records to ``/var/spool/apel`` from where ssmsend will take them.

SSM configuration is available at ``/etc/apel``. Defaults should be ok for most cases. The cron file uses ``/etc/grid-security`` for the CAs and the host certificate and private keys (``/etc/grid-security/hostcert.pem`` and ``/etc/grid-security/hostkey.pem``).

Running the services
""""""""""""""""""""

Both caso and ssmsend are run via cron scripts. They are located at ``/etc/cron.d/caso`` and ``/etc/crond.d/ssmsend`` respectively. For convenience there are also two scripts ``/usr/loca/bin/caso-extract.sh`` and ``/usr/local/bin/ssm-send.sh`` that run the docker container with the proper volumes.

EGI Information System
~~~~~~~~~~~~~~~~~~~~~~

Information discovery provides a real-time view about the actual images and flavors available at the OpenStack for the federation users. It has two components:

* Resource-Level BDII: which queries the OpenStack deployment to get the information to publish

* Site-Level BDII: gathers information from several resource-level BDIIs (in this case only 1) and makes it publicly available for the EGI information system.

Resource-level BDII
"""""""""""""""""""

This is provided by container ``egifedcloud/cloudbdii``. You need to configure:

* ``/etc/cloud-info-provider/openstack.rc``, with the credentials to query your OpenStack. The user configured just needs to be able to access the lists of images and flavors.

* ``/etc/cloud-info-provider/openstack.yaml``, this file includes the static information of your deployment. Make sure to set the ``SITE-NAME`` as defined in GOCDB.

Site-level BDII
"""""""""""""""

The ``egifedcloud/sitebdii`` container runs this process. Configuration files:

* `/etc/sitebdii/glite-info-site-defaults.conf`. Set here the name of your site (as defined in GOCDB) and the public hostname where the appliance will be available.

* `/etc/sitebdii/site.cfg`. Include here basic information on your site.

Running the services
""""""""""""""""""""

There is a `bdii.service` unit for systemd available in the appliance. This leverages docker-compose for running the containers. You can start the service with:

::

    systemctl start bdii

Check the status with:

::

    systemctl status bdii

And stop with:

::

    systemctl stop bdii

You should be able to get the BDII information with an LDAP client, e.g.:

::

    ldapsearch -x -p 2170 -h <yourVM.hostname.domain.com> -b o=glue

EGI VM Image Management
~~~~~~~~~~~~~~~~~~~~~~~

The appliance provides VMI replication with cloudkeeper. Every 4 hours, the appliance will perform the following actions:

* download the configured lists in ``/etc/cloudkeeper/image-lists.conf`` and verify its signature

* check any changes in the lists and download new images

* synchronise this information to the configured glance endpoint

cloudkeeper has two components:

* fronted dealing the with image lists and downloading the needed images

* backend dealing with your glance catalogue

First you need to configure and start the backend. Edit ``/etc/cloudkeeper/cloudkeeper-os.conf`` and add the authentication parameters from line 117 to 136.
Then add as many image lists (one per line) as you would like to subscribe to ``/etc/cloudkeeper/image-lists.conf``. Use URLs with your AppDB token for authentication.

Running the services
""""""""""""""""""""

cloudkeeper-os should run permanently, there is a ``cloudkeeper-os.service`` for systemd in the appliance. Manage as usual:

::

    systemctl <start|stop|status> cloudkeeper-os

cloudkeeper core is run every 4 hours with a cron script.

Individual Components
'''''''''''''''''''''

EGI Accounting
~~~~~~~~~~~~~~


Every cloud RC should publish utilization data to the EGI accounting database. You will need to install **cASO**, a pluggable extractor of Cloud Accounting Usage Records from OpenStack.

Documentation on how to install and configure cASO is available at https://caso.readthedocs.org/en/latest/

In order to send the records to the accounting database, you will also need to configure **SSM**, whose documentation can be found at https://github.com/apel/ssm


EGI Information System
~~~~~~~~~~~~~~~~~~~~~~

Sites must publish information to EGI information system which is based on BDII. The BDII can be installed easily directly from the distribution repository, the package is usually named "bdii".

There is a common cloud information provider for all cloud management frameworks that collects the information from the used CMF and send them to the aforementioned BDII. It can be installed on the same machine as the BDII or on another machine. The installation and configuration guide for the cloud information provider can be found in the following `Fedclouds BDII instructions <https://wiki.egi.eu/wiki/HOWTO15>`_.

EGI VM Image Management
~~~~~~~~~~~~~~~~~~~~~~~

.. TODO: Where are the docs?

Post-installation
:::::::::::::::::

After the installation of all the needed components, it is recommended to set the following policies on Nova to avoid users accessing other users resources:

::

    sed -i 's|"admin_or_owner":  "is_admin:True or project_id:%(project_id)s",|"admin_or_owner":  "is_admin:True or project_id:%(project_id)s",\n    "admin_or_user":  "is_admin:True or user_id:%(user_id)s",|g' /etc/nova/policy.json
    sed -i 's|"default": "rule:admin_or_owner",|"default": "rule:admin_or_user",|g' /etc/nova/policy.json
    sed -i 's|"compute:get_all": "",|"compute:get": "rule:admin_or_owner",\n    "compute:get_all": "",|g' /etc/nova/policy.json
