HaaS Dev Setup
++++++++++++++

Installing and Setting Up HaaS
------------------------------

Install Necessary Programs
==========================

 ::

  apt-get install git python-pip sqlite3
  pip install virtualenv

Setup Virtual Environment
=========================

 ::

  virtualenv .haas
  source .haas/bin/activate

Setup HaaS
==========

 add a HaaS user:
  ::
  
   useradd haas_user -d /var/lib/haas -m -r
   passwd haas_user <password>

 clone + install HaaS:
  ::

   git clone https://github.com/CCI-MOC/haas
   cd haas
   pip install -e .

 put haas.cfg in expected location:
  ::

   mv /haas/examples/haas.cfg.dev-no-hardware /etc/haas.cfg

 /etc/haas.cfg should have its permissions set to read-only and ownership set to the haas_user:
  ::

   chown haas_user:haas_user /etc/haas.cfg
   chmod 400 /etc/haas.cfg

 create symbolic link to haas.cfg in haas_user home directory:
  ::

   ln -s /etc/haas.cfg /var/lib/haas/

 **Note:** all haas commands should be run from /var/lib/haas signed in as haas_user
  ::

   su haas_user
   cd ~

 initialize the database:
 
   **Note:** If using PostgreSQL complete steps for alternative database setup below before initializing the database

 ::

    haas init_db

Test
====

 in one window run:
  ::

   haas serve 5000

 in another run:
  ::

   haas serve_networks

 can then execute ``haas`` commands, e.g.
  ::

   haas project_create proj
   haas headnode_create hn proj
   haas headnode_create_hnic hn hn-eth0
   haas headnode_delete_hnic hn hn-eth0

 which tests ``headnode_delete_hnic``

 Additionally, before each commit, run the automated test suite with ``py.test tests/unit``. If at all possible, run the deployment tests as well (``py.test tests/deployment``), but this requires access to a sepcialized setup, so if the patch is sufficiently unintrusive it may be acceptable to skip this step.

Notes and Comments
------------------

Configuration
=============

 HaaS can be configured to not perform state-changing operations on nodes, headnodes and networks, allowing developers to run and test parts of a haas server without requiring physical hardware. To supress actual node and headnode operations, set ``dry_run = True`` in the ``[devel]`` section. For supressing actual network switch operations, use the ``mock`` switch driver and the ``null`` network allocator.

Possible Errors
===============

 On systems with older versions of ``pip``, such as ``Debian Wheezy`` and ``Ubuntu 12.04``, the installation of HaaS will fail with the following error:
  ::

   AttributeError: 'NoneType' object has no attribute 'skip_requirements_regex'

 Fix this by upgrading pip within the virtual environment:
  ::

   pip install --upgrade pip

 Versions of ``python`` prior to 2.7 don't have ``importlib`` as part of their standard library, but it is possible to install it separately. If you're using python 2.6 (which is what is available on ``CentOS 6``, for example), you may need to run:
  ::

   pip install importlib

Alternative Setup for Use With PostgreSQL
-----------------------------------------

install PostgreSQL:
 ::
  
  sudo apt-get install postgresql

log into postgres account:
 ::

  sudo -i -u postgres

create user for the haas database:
 ::

  createuser -Pd haas_user

 -d allows haas_user to create databases
 -P prompts to create a password

create a database:
 ::

  createdb haas_db

Complete steps in `Installing and Setting Up HaaS` except do not yet initialize the database.

setup haas.cfg to access Postgres database:

 the ``uri`` option in haas.cfg must be set to:
  ::

   postgresql://<user>:<password>@<address>/<dbname>

 Where ``<user>`` is the name of the postgres user you created, ``<password>`` is its password, ``<dbname>`` is the name of the database you created, and ``<address>`` is the address which HaaS should use to connect to postgres (In a typical default postgres setup, the right value is ``localhost``).
 
Can then initialize the database.
