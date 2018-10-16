.. include:: ../Includes.txt
.. highlight:: shell

.. _testing:

================
Testing the Core
================

For every patch that is uploaded to the Gerrit review server, a number
of tests are run automatically using Bamboo.

.. hint::
   For more in-depth information about how TYPO3 runs the automatic tests with Bamboo
   see the series "Testing TYPO3's Core" on the TYPO3 Blog. You can
   find the links in this article:
   `Serious software testing: TYPO3 runs its 20,000th build!
   <https://typo3.com/blog/serious-software-testing-typo3-runs-its-20000th-build>`__

You can run these tests locally with docker using the same setup Bamboo uses.


.. _testing-install-docker:

Install Docker
==============

All you need for this is `Docker installed locally.
<https://www.docker.com/get-docker>`__


.. _testing-docker-pull:

docker pull
===========

You have to pull the necessary docker image for PHP from `Bitbucket T3COM bamboo-
remote-agent <https://bitbucket.typo3.com/projects/T3COM/repos/bamboo-remote-agent/browse>`__.
There are various images for testing with different PHP versions and
different databases and the functional tests. As an example, if you want to run
the unit tests for current master you have to load the php72 container::

   docker pull typo3gmbh/php72:latest

.. _testing-docker-run:

docker run
==========

Start the container like this. Replace 
`<absolute-local-path-where-your typo3-checkout-is>` with your 
current directory::

   docker run -it --rm \
      --name=typo3_core_test \
      -v <absolute-local-path-where-your typo3-checkout-is>:/srv/tmp/cms \
      -w /srv/tmp/cms \
      typo3gmbh/php72:latest \
      bash

Or, for more convenience, you can just use this, which determines
your current directory with pwd::

   docker run -it --rm \
      --name=typo3_core_test \
      -v `pwd`:/srv/tmp/cms \
      -w /srv/tmp/cms \
      typo3gmbh/php72:latest \
      bash

.. note::
   The docker run command has changed, it used to contain /sbin/my_init 
   which has been discontinued from the typo3gmb/php72:latest docker 
   container

.. _testing-docker-run-unit:

Run all Unit Tests
==================

*From inside the container*, to run all unit tests::


    HOME=/root \
    bin/phpunit -c vendor/typo3/testing-framework/Resources/Core/Build/UnitTests.xml

.. testing-docker-run-functional:

Run all Functional Tests
========================

Running the functional tests is almost the same. It is just the phpunit command
that looks a bit different. For the MySQL setup use::

   export typo3DatabaseName="func" \
          typo3DatabaseUsername="funcu" \
          typo3DatabasePassword="funcp" \
          typo3DatabaseHost="localhost" \
          typo3InstallToolPassword="klaus" \
          && bin/phpunit -c vendor/typo3/testing-framework/Resources/Core/Build/FunctionalTests.xml

When you want to test against other databases like PostgreSQL or MSSQL you will
find the database credentials inside the :file:`Build` folder of the TYPO3
core in the file :file:`Build/bamboo/src/main/java/core/AbstractCoreSpec.java`.

Search for 'typo3DatabaseUsername' in this Java file to find the definitions of
the different database credentials.

.. hint::

   Be sure to have a cup of coffee, a good book or other things to be done to
   span the waiting time. Functional tests will take a lot more time than unit
   tests. Depending on the power of your local machine you can expect about 45
   minutes or more.

.. _testing-docker-run-acceptance:

Run Acceptance Tests
=====================

Ensure temporary test directory exists or create it::

    mkdir -p typo3temp/var/tests

Start webserver and chrome driver in background::

    php -S localhost:8000 >/dev/null 2>&1 &
    bin/chromedriver --url-base=/wd/hub >/dev/null 2>&1 &

Run all acceptance tests
------------------------

::

   export typo3DatabaseName="func" \
          typo3DatabaseUsername="funcu" \
          typo3DatabasePassword="funcp" \
          typo3DatabaseHost="localhost" \
          && bin/codecept run Acceptance -c vendor/typo3/testing-framework/Resources/Core/Build/AcceptanceTests.yml

.. hint::

   Depending on the power of your local machine you can expect about 30 minutes
   or more.

Run a single acceptance test
----------------------------

::

   export typo3DatabaseName="func" \
          typo3DatabaseUsername="funcu" \
          typo3DatabasePassword="funcp" \
          typo3DatabaseHost="localhost" \
          && bin/codecept run Acceptance -c vendor/typo3/testing-framework/Resources/Core/Build/AcceptanceTests.yml typo3/sysext/core/Tests/Acceptance/Backend/Topbar/LogoCest.php

Run a single acceptance test with debug output
----------------------------------------------

::

    bin/codecept run Acceptance -c vendor/typo3/testing-framework/Resources/Core/Build/AcceptanceTests.yml --debug typo3/sysext/core/Tests/Acceptance/Backend/Topbar/LogoCest.php

Results
-------

Reports will be stored in :file:`typo3temp/var/tests/AcceptanceReports` with
screenshots from browser.
