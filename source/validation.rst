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


