Integration
-----------

Integration of cloud stacks into EGI FedCloud follows a well-defined path, with certain steps which need to be taken, depending on the cloud stack in question.
By integration here, we refer to the proper interoperation with EGI infrastructure services such as accounting, monitoring, authentication and authorisation, *etc*.
These configurations make your site discoverable and usable by the communities you wish to support, and allow EGI to support you in operational and technical matters.

Integration of these services implies specific configuration actions which you need to take on your site.
These aim to be unintrusive and are mostly to facilitate access to your site by the communities you wish to support, without interfering with normal operations. This can be summarised essentially as :

#. Network configuration
#. Permissions configuration
#. AAI configuration
#. Accounting configuration
#. Information system integration
#. VM and appliance repository configuration

If at any time you experience technical difficulties or need support, please `open a ticket <https://ggus.eu>`_ or discuss the matter with us `on the forum <https://community.egi.eu>`_

You can follow dedicated integration guides for each cloud management frameworks:


.. toctree::
   :maxdepth: 2

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
