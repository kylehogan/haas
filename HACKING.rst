Setup Test Environment
++++++++++++++++++++++

Install Prerequisite Software
=============================

HaaS requires a number of packages available through the CentOS RPM
repositories, as well as the EPEL repository. EPEL can be enabled via::

    yum install epel-release

Then, the rest of the packages can be installed via::

    yum install libvirt bridge-utils ipmitool telnet httpd mod_wsgi python-pip qemu-kvm python-virtinst virt-install python-psycopg2 vconfig net-tools

In addition, HaaS depends on a number of python libraries. Many of these are
available as RPMs as well, but we recommend installing them with pip, since
this will install the versions that HaaS has been tested with.  This is done
automatically by the instructions below.

Creating HaaS System User
=========================

Create a system user ``haas`` with::

  sudo useradd haas_user -d /var/lib/haas -m -r
  passwd haas_user <password>

Setup Virtual Environment and Install HaaS
==========================================

Set up a clean test environment::

  virtualenv .venv

Next, and each time you start working, enter the environment::

  source .venv/bin/activate
  
Clone HaaS::

  git clone https://github.com/CCI-MOC/haas
  cd haas

Then, proceed with installing the HaaS and its dependencies into the virtual
environment::

  pip install -e .


Configuring HaaS
================

Now the ``haas`` executable should be in your path.  First, create a
configuration file ``haas.cfg``. There are two examples for you to work from,
``examples/haas.cfg.dev-example``, which is oriented towards development, and
``examples/haas.cfg.example`` which is more production oriented.  These config
files are well commented; read them carefully.

HaaS can be configured to not perform state-changing operations on nodes,
headnodes and networks, allowing developers to run and test parts of a haas
server without requiring physical hardware. To suppress actual node and headnode
operations, set ``dry_run = True`` in the ``[devel]`` section. For suppressing
actual network switch operations, use the ``mock`` switch driver.

The file should be placed at ``/etc/haas.cfg``; The ``haas.wsgi``
script, described below, requires this. Awkwardly, the ``haas``
command line tool expects the file to be present in its current
working directory. This will be fixed in the next release, but for
now, put the file in ``/etc`` and create a symlink to it in the
HaaS user's home directory::

  sudo ln -s /etc/haas.cfg /var/lib/haas/
  
Additionally, the ``/etc/haas.cfg`` should have its permissions set to read-only and ownership
set to the ``haas_user``::

  sudo chown haas_user:haas_user /etc/haas.cfg
  sudo chmod 400 /etc/haas.cfg

Setting up Database
===================

If using SQLite no additional setup is necessary. Should ensure that `uri` in `database` setion of 'haas.cfg' is set to `sqlite:///haas.db` and can skip to initializing the database.

If using PostgreSQL, follow these steps to setup the database:

log into postgres account::

  sudo -i -u postgres

create user for the haas database::

  createuser -Pd haas_user

 -d allows haas_user to create databases
 -P prompts to create a password

create a database owned by the haas_user::

  createdb haas_db -O haas_user
  
setup haas.cfg to access Postgres database:

the ``uri`` option in haas.cfg must be set to::

   postgresql://<user>:<password>@<address>/<dbname>

Where ``<user>`` is the name of the postgres user you created (here this is ``haas_user``), ``<password>`` is its password, ``<dbname>`` is the name of the database you created (here this is ``haas_db``), and ``<address>`` is the address which HaaS should use to connect to postgres (In a typical default postgres setup, the right value is ``localhost``).
 
Initialize Database and Test Haas
=================================

**Note:** all haas commands should be run from /var/lib/haas signed in as haas_user

Run ``haas-admin db create`` to initialize database with required tables.

Run the server with ``haas serve`` and ``haas serve_networks`` in separate
terminals.  Finally, ``haas help`` lists the various API commands one can use.
Here is an example session, testing ``headnode_delete_hnic``::

  haas project_create proj
  haas headnode_create hn proj
  haas headnode_create_hnic hn hn-eth0
  haas headnode_delete_hnic hn hn-eth0

Additionally, before each commit, run the automated test suite with ``py.test
tests/unit``. If at all possible, run the deployment tests as well (``py.test
tests/deployment``), but this requires access to a sepcialized setup, so if the
patch is sufficiently unintrusive it may be acceptable to skip this step.

`testing.md <docs/testing.md>`_ contains more information about testing HaaS.
`migrations.md <docs/migrations.md>`_ dicsusses working with database migrations
and schema changes.

Possible Errors
===============

On systems with older versions of ``pip``, such as ``Debian Wheezy`` and ``Ubuntu 12.04``, the installation of HaaS will fail with the following error::

  AttributeError: 'NoneType' object has no attribute 'skip_requirements_regex'

Fix this by upgrading pip within the virtual environment::

  pip install --upgrade pip

Versions of ``python`` prior to 2.7 don't have ``importlib`` as part of their standard library, but it is possible to install it separately. If you're using python 2.6 (which is what is available on ``CentOS 6``, for example), you may need to run::

  pip install importlib

You may get an error 'psycopg2 package not found' when you do 'haas-admin db create'
in the next step if you are using PostgreSQL database. 

if its Centos::  

  yum install postgresql-devel

if its Ubuntu::
  
  sudo apt-get install libpq-dev

before installing ``psycopg2`` in the virtualenv for HaaS::

  pip install psycopg2
