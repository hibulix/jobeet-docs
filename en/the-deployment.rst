NOT CONVERTED TO SYMFONY2 YET
=============================

Want to help out?
Fork it on `Github <https://github.com/sftuts/jobeet-docs>`_

Day 22: The Deployment
======================

With the configuration of the cache system in the 21st day, the
Jobeet website is ready to be deployed on
the production servers.

During twenty-two days, we have developed Jobeet on a development
machine, and for most of you, it probably means your local machine;
except if you develop on the production server directly, which is
of course a very bad idea. Now, it is time to move the website to a
production server.

Now, we will see what needs to be done before going to production,
what kind of deploying strategies you can
use, and also the tools you need for a successful deployment.

Preparing the Production Server
---------------------------------------------

Before deploying the project to production, we need to be sure the
production server is configured correctly. You can re-read day 1,
where we explained how to configure the web server.

In this section, we assume that you have already installed the web
server, the database server, and PHP 5.2.4 or later.

    **NOTE** If you don't have an SSH access to the web server, skip
    the part where you need to have access to the command line.


Server Configuration
~~~~~~~~~~~~~~~~~~~~

First, you need to check that PHP is installed with all the needed
extensions and is correctly configured. As for day 1, we will use
the ``check_configuration.php`` script provided with symfony. As we
won't install symfony on the production server, download the file
directly from the symfony website:

::

    http://trac.symfony-project.org/browser/branches/1.4/data/bin/check_configuration.php?format=raw

Copy the file to the web root directory and run it from your
browser **and** from the command line:

::

    $ php check_configuration.php

Fix any fatal error the script finds and repeat the process until
everything works fine in **both** environments.

PHP Accelerator
~~~~~~~~~~~~~~~~~~~~~~~~~~

For the production server, you probably want the best performance
possible. Installing a
`PHP accelerator <http://en.wikipedia.org/wiki/PHP_accelerator>`_
will give you the best improvement for your money.

    **NOTE** From Wikipedia: A PHP accelerator works by caching the
    compiled bytecode of PHP scripts to avoid the overhead of parsing
    and compiling source code on each request.


`APC <http://www.php.net/apc>`_ is one of the most
popular one, and it is quite simple to install:

::

    $ pecl install APC

Depending on your Operating System, you will also be able to
install it with the OS native package manager.

    **NOTE** Take some time to learn how to
    `configure APC <http://www.php.net/manual/en/apc.configuration.php>`_.


The symfony Libraries
---------------------

Embedding symfony
~~~~~~~~~~~~~~~~~

One of the great strengths of symfony is that a project is
self-contained. All the files needed for the project to work are
under the main root project directory. And you can move around the
project in another directory without changing anything in the
project itself as symfony only uses relative paths. It means that
the directory on the production server does not have to be the same
as the one on your development machine.

The only absolute path that can possibly be found is in the
``config/ProjectConfiguration.class.php`` file; but we took care of
it during day 1. Check that it actually contains a relative path to
the symfony core autoloader:

::

    <?php
    // config/ProjectConfiguration.class.php
    require_once dirname(__FILE__).'/../lib/vendor/symfony/lib/autoload/sfCoreAutoload.class.php';

Upgrading symfony
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Even if everything is self-contained in a single directory,
upgrading symfony to a newer release is nonetheless insanely easy.

You will want to upgrade symfony to the latest minor release from
time to time, as we constantly fix bugs and possibly security
issues. The good news is that all symfony versions are maintained
for at least a year and during the maintenance period, we never
ever add new features, even the smallest one. So, it is always
fast, safe, and secure to upgrade from one minor release to
another.

Upgrading symfony is as simple as changing the content of the
``lib/vendor/symfony/`` directory. If you have installed symfony
with the archive, remove the current files and replace them with
the newest ones.

If you use Subversion for your project, you can also
link your project to the latest symfony 1.4 tag:

::

    $ svn propedit svn:externals lib/vendor/
      # symfony http://svn.symfony-project.com/tags/RELEASE_1_4_3/

Upgrading symfony is then as simple as changing the tag to the
latest symfony version.

You can also use the 1.4 branch to have fixes in real-time:

::

    $ svn propedit svn:externals lib/vendor/
      # symfony http://svn.symfony-project.com/branches/1.4/

Now, each time you do an ``svn up``, you will have the latest
symfony 1.4 version.

When upgrading to a new version, you are advised to always clear
the cache, especially in the production environment:

::

    $ php symfony cc

    **TIP** If you also have an FTP access to the production server,
    you can simulate a ``symfony cc`` by simply removing all the files
    and directories under the ``cache/`` directory.


You can even test a new symfony version without replacing the
existing one. If you just want to test a new release, and want to
be able to rollback easily, install symfony in another directory
(``lib/vendor/symfony_test`` for instance), change the path in the
``ProjectConfiguration`` class, clear the cache, and you are done.
Rollbacking is as simple as removing the directory, and change back
the path in ``ProjectConfiguration``.

Tweaking the Configuration
-------------------------------------

Database Configuration
~~~~~~~~~~~~~~~~~~~~~~

Most of the time, the production database has different credentials
than the local one. Thanks to the symfony environments, it is quite
simple to have a different configuration for the production
database:

::

    $ php symfony configure:database
       ➥ "mysql:host=localhost;dbname=prod_dbname" prod_user prod_pass

You can also edit the ``databases.yml`` configuration file
directly.

Assets
~~~~~~~~~~~~~~~~~

As Jobeet uses plugins that embed assets, symfony
created relative symbolic links in the ``web/`` directory. The
``plugin:publish-assets`` task regenerates or creates them if you
install plugins without the ``plugin:install`` task:

::

    $ php symfony plugin:publish-assets

Customizing Error Pages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before going to production, it is better to customize ~default
symfony pages\|Default symfony Pages~, like the "~Page Not
Found\|404 Error~" page, or the default exception page.

We have already configured the error page for the ``YAML`` format
during day 15, by creating an ``error.yaml.php`` and an
``exception.yaml.php`` files in the ``config/error/`` directory.
The ``error.yaml.php`` file is used by symfony in the ``prod``
environment, whereas ``exception.yaml.php`` is used in the ``dev``
environment.

So, to customize the default exception page
for the HTML format, create two files:
``config/error/error.html.php`` and
``config/error/exception.html.php``.

The ``404`` page (page not found) can be customized by changing the
``error_404_module`` and ``error_404_action`` settings:

::

    [yml]
    # apps/frontend/config/settings.yml
    all:
      .actions:
        error_404_module: default
        error_404_action: error404

Customizing the Directory Structure
----------------------------------------------

To better structure and standardize your code, symfony has a
default directory structure with pre-defined names. But sometimes,
you don't have the choice but to change the structure because of
some external constraints.

Configuring the directory names can be done in the
``config/ProjectConfiguration.class.php`` class.

The Web Root Directory
~~~~~~~~~~~~~~~~~~~~~~~~

On some web hosts, you cannot change the web root directory name.
Let's say that on your web host, it is named ``public_html/``
instead of ``web/``:

::

    <?php
    // config/ProjectConfiguration.class.php
    class ProjectConfiguration extends sfProjectConfiguration
    {
      public function setup()
      {
        $this->setWebDir($this->getRootDir().'/public_html');
      }
    }

The ``setWebDir()`` method takes the absolute path of the web root
directory. If you also move this directory elsewhere, don't forget
to edit the controller scripts to check that paths to the
``config/ProjectConfiguration.class.php`` file are still valid:

::

    <?php
    require_once(dirname(__FILE__).'/../config/ProjectConfiguration.class.php');

The Cache`\  and \ :sub:`Log Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The symfony framework only writes in two directories: ``cache/``
and ``log/``. For security reasons, some web
hosts do not set write permissions in the main
directory. If this is the case, you can move these directories
elsewhere on the filesystem:

::

    <?php
    // config/ProjectConfiguration.class.php
    class ProjectConfiguration extends sfProjectConfiguration
    {
      public function setup()
      {
        $this->setCacheDir('/tmp/symfony_cache');
        $this->setLogDir('/tmp/symfony_logs');
      }
    }

As for the ``setWebDir()`` method, ``setCacheDir()`` and
``setLogDir()`` take an absolute path to the ``cache/`` and
``log/`` directories respectively.

Customizing symfony core Objects (aka factories)
------------------------------------------------

During day 16, we talked a bit about the symfony factories. Being
able to customize the factories means that you can use a custom
class for symfony core objects instead of the default one. You can
also change the default behavior of these classes by changing the
parameters send to them.

Let's take a look at some classic customizations you may want to
do.

Cookie Name
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To handle the user session, symfony uses a cookie. This
cookie has a default name of ``symfony``, which can be changed in
``factories.yml``. Under the ``all`` key, add the following
configuration to change the cookie name to ``jobeet``:

::

    [yml]
    # apps/frontend/config/factories.yml
    storage:
      class: sfSessionStorage
      param:
        session_name: jobeet

Session Storage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The default session storage class is ``sfSessionStorage``. It uses
the filesystem to store the session information. If you have
several web servers, you would want to store the sessions in a
central place, like a database table:

::

    [yml]
    # apps/frontend/config/factories.yml
    storage:
      class: sfPDOSessionStorage
      param:
        session_name: jobeet
        db_table:     session

database: propel database: doctrine db\_id\_col: id db\_data\_col:
data db\_time\_col: time

Session Timeout
~~~~~~~~~~~~~~~

By default, the user session timeout if
``1800`` seconds. This can be changed by editing the ``user``
entry:

::

    [yml]
    # apps/frontend/config/factories.yml
    user:
      class: myUser
      param:
        timeout: 1800

Logging
~~~~~~~~~~~~~~~~~~

By default, there is no logging in the ``prod``
environment because the logger class name
is ``sfNoLogger``:

::

    [yml]
    # apps/frontend/config/factories.yml
    prod:
      logger:
        class:   sfNoLogger
        param:
          level:   err
          loggers: ~

You can for instance enable logging on the filesystem by changing
the logger class name to ``sfFileLogger``:

::

    [yml]
    # apps/frontend/config/factories.yml
    logger:
      class: sfFileLogger
      param:
        level:   err
        loggers: ~
        file:    %SF_LOG_DIR%/%SF_APP%_%SF_ENVIRONMENT%.log

    **NOTE** In the ``factories.yml`` configuration file, ``%XXX%``
    strings are replaced with their corresponding value from the
    ``sfConfig`` object. So, ``%SF_APP%`` in a configuration file is
    equivalent to ``sfConfig::get('sf_app')`` in PHP code. This
    notation can also be used in the ``app.yml`` configuration file. It
    is very useful when you need to reference a path in a configuration
    file without hardcoding the path (``SF_ROOT_DIR``, ``SF_WEB_DIR``,
    ...).


Deploying
--------------------

What to deploy?
~~~~~~~~~~~~~~~

When deploying the Jobeet website to the production server, we need
to be careful not to deploy unneeded files or override files
uploaded by our users, like the company logos.

In a symfony project, there are three directories to exclude from
the transfer: ``cache/``, ``log/``, and ``web/uploads/``.
Everything else can be transfered as is.

For security reasons, you also don't want to transfer the
"non-production" front controllers, like the ``frontend_dev.php``,
``backend_dev.php`` and ``frontend_cache.php`` scripts.

Deploying Strategies
~~~~~~~~~~~~~~~~~~~~

In this section, we will assume that you have full control over the
production server(s). If you can only access the server with a FTP
account, the only deployment solution possible is to transfer all
files every time you deploy.

The simplest way to deploy your website is to use the built-in
``project:deploy`` task. It uses
``SSH```\  and \ :sub:```rsync`` to connect and transfer
the files from one computer to another one.

Servers for the ``project:deploy`` task can be configured in the
``config/properties.ini`` configuration file:

::

    [ini]
    # config/properties.ini
    [production]
      host=www.jobeet.org
      port=22
      user=jobeet
      dir=/var/www/jobeet/

To deploy to the newly configured ``production`` server, use the
``project:deploy`` task:

::

    $ php symfony project:deploy production

    **NOTE** Before running the ``project:deploy`` task for the first
    time, you need to connect to the server manually to add the key in
    the known hosts file.


-

    **TIP** If the command does not work as expected, you can pass the
    ``-t`` option to see the real-time output of the ``rsync``
    command.


If you run this command, symfony will only simulate the transfer.
To actually deploy the website, add the ``--go`` option:

::

    $ php symfony project:deploy production --go

    **NOTE** Even if you can provide the SSH password in the
    ``properties.ini`` file, it is better to configure your server with
    a SSH key to allow password-less connections.


By default, symfony won't transfer the directories we have talked
about in the previous section, nor it will transfer the ``dev``
front controller script. That's because the ``project:deploy`` task
exclude files and directories are configured in the
``config/rsync_exclude.txt`` file:

::

    # config/rsync_exclude.txt
    .svn
    /web/uploads/*
    /cache/*
    /log/*
    /web/*_dev.php

For Jobeet, we need to add the ``frontend_cache.php`` file:

::

    # config/rsync_exclude.txt
    .svn
    /web/uploads/*
    /cache/*
    /log/*
    /web/*_dev.php
    /web/frontend_cache.php

    **TIP** You can also create a ``config/rsync_include.txt`` file to
    force some files or directories to be transfered.


Even if the ``project:deploy`` task is very flexible, you might
want to customize it even further. As deploying can be very
different based on your server configuration and topology, don't
hesitate to extend the default task.

Each time you deploy a website to production, don't forget to at
least clear the configuration cache on the production server:

::

    $ php symfony cc --type=config

If you have changed some routes, you will also need to clear the
routing cache:

::

    $ php symfony cc --type=routing

    **NOTE** Clearing the cache selectively allows to keep some parts
    of the cache, such as the template cache.


Final Thoughts
--------------

The deployment of a project is the very last step of the symfony
development life-cycle. It does not mean that you are done. This is
quite the contrary. A website is something that has a life by
itself. You will probably have to fix bugs and you will also want
to add new features over time. But thanks to the symfony structure
and the tools at your disposal, upgrading your website is simple,
fast, and safe.

Tomorrow, will be the last day of the Jobeet tutorial. It will be
time to take a step back and have a look at what you learned during
the twenty-three days of Jobeet.

**ORM**


