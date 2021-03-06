NOT CONVERTED TO SYMFONY2 YET
=============================

Want to help out?
Fork it on `Github <https://github.com/sftuts/jobeet-docs>`_

Day 5: The Routing
==================

If you've completed day 4, you should now be familiar with the MVC
pattern and it should be feeling like a more and more natural way
of coding. Spend a bit more time with it and you won't look back.
To practice a bit, we customized the Jobeet pages and in the
process, also reviewed several symfony concepts, like the layout,
helpers, and slots.

Today, we will dive into the wonderful world of the symfony routing
framework.

URLs
---------------

If you click on a job on the Jobeet homepage, the URL looks like
this: ``/job/show/id/1``. If you have already developed PHP
websites, you are probably more accustomed to URLs like
``/job.php?id=1``. How does symfony make it work? How does symfony
determine the action to call based on this URL? Why is the ``id``
of the job retrieved with ``$request->getParameter('id')``? Here,
we will answer all these questions.

But first, let's talk about URLs and what exactly they are. In a
web context, a URL is the unique identifier of a web resource. When
you go to a URL, you ask the browser to fetch a resource identified
by that URL. So, as the URL is the interface between the website
and the user, it must convey some meaningful information about the
resource it references. But "traditional" URLs do not really
describe the resource, they expose the internal structure of the
application. The user does not care that your website is developed
with the PHP language or that the job has a certain identifier in
the database. Exposing the internal workings of your application is
also quite bad as far as security is
concerned: What if the user tries to guess the URL for resources he
does not have access to? Sure, the developer must secure them the
proper way, but you'd better hide sensitive information.

URLs are so important in symfony that it has an entire framework
dedicated to their management: the **routing**
framework. The routing manages internal URIs and external URLs.
When a request comes in, the routing parses the URL and converts it
to an internal URI.

You have already seen the internal URI of the job page in the
``indexSuccess.php`` template:

::

    'job/show?id='.$job->getId()

The ~``url_for()`` helper~ converts this internal URI to a proper
URL:

::

    /job/show/id/1

The internal URI is made of several parts: ``job`` is the module,
``show`` is the action and the query string adds parameters to pass
to the action. The generic pattern for internal URIs is:

::

    MODULE/ACTION?key=value&key_1=value_1&...

As the symfony routing is a two-way process, you can change the
URLs without changing the technical implementation. This is one of
the main advantages of the front-controller design pattern.

Routing Configuration
---------------------

The mapping between internal URIs and external URLs is done in the
``routing.yml`` configuration file:

::

    [yml]
    # apps/frontend/config/routing.yml
    homepage:
      url:   /
      param: { module: default, action: index }
    
    default_index:
      url:   /:module
      param: { action: index }
    
    default:
      url:   /:module/:action/*

The ``routing.yml`` file describes routes. A route has a name
(``homepage``), a pattern (``/:module/:action/*``), and some
parameters (under the ``param`` key).

When a request comes in, the routing tries to match a pattern for
the given URL. The first route that matches wins, so the order in
``routing.yml`` is important. Let's take a look at some examples to
better understand how this works.

When you request the Jobeet homepage, which has the ``/job`` URL,
the first route that matches is the ``default_index`` one. In a
pattern, a word prefixed with a colon (``:``) is
a variable, so the ``/:module`` pattern means: Match a ``/``
followed by something. In our example, the ``module`` variable will
have ``job`` as a value. This value can then be retrieved with
``$request->getParameter('module')`` in the action. This route also
defines a default value for the ``action`` variable. So, for all
URLs matching this route, the request will also have an ``action``
parameter with ``index`` as a value.

If you request the ``/job/show/id/1`` page, symfony will match the
last pattern: ``/:module/:action/*``. In a pattern, a star (``*``)
matches a collection of variable/value pairs separated by slashes
(``/``):

\| Request parameter \| Value \| \| ----------------- \| ----- \|
\| module \| job \| \| action \| show \| \| id \| 1 \|

    **NOTE** The ``module|Module`` and
    ``action|Action`` variables are special as they are used
    by symfony to determine the action to execute.


The ``/job/show/id/1`` URL can be created from a template by using
the following call to the ``url_for()`` helper:

::

    <?php
    url_for('job/show?id='.$job->getId())

You can also use the route name by prefixing it by ``@``:

::

    <?php
    url_for('@default?module=job&action=show&id='.$job->getId())

Both calls are equivalent but the latter is much faster as the
routing does not have to parse all routes to find the best match,
and it is less tied to the implementation (the module and action
names are not present in the internal URI).

Route Customizations
--------------------

For now, when you request the ``/`` URL in a browser, you have the
default congratulations page of symfony. That's because this URL
matches the ``homepage`` route. But it makes
sense to change it to be the Jobeet homepage. To make the change,
modify the ``module`` variable of the ``homepage`` route to
``job``:

::

    <?php
    # apps/frontend/config/routing.yml
    homepage:
      url:   /
      param: { module: job, action: index }

We can now change the link of the Jobeet logo in the layout to use
the ``homepage`` route:

::

    <?php
    <!-- apps/frontend/templates/layout.php -->
    <h1>
      <a href="<?php echo url_for('homepage') ?>">
        <img src="/images/logo.jpg" alt="Jobeet Job Board" />
      </a>
    </h1>

That was easy!

    **TIP** When you update the routing configuration, the changes are
    immediately taken into account in the development environment. But
    to make them also work in the production environment, you need to
    clear the cache by calling the ``cache:clear`` task.


For something a bit more involved, let's change the job page URL to
something more meaningful:

::

    /job/sensio-labs/paris-france/1/web-developer

Without knowing anything about Jobeet, and without looking at the
page, you can understand from the URL that Sensio Labs is looking
for a Web developer to work in Paris, France.

    **NOTE** Pretty URLs are important because they convey information
    for the user. It is also useful when you copy and paste the URL in
    an email or to optimize your website for search engines.


The following pattern matches such a URL:

::

    /job/:company/:location/:id/:position

Edit the ``routing.yml`` file and add the ``job_show_user`` route
at the beginning of the file:

::

    [yml]
    job_show_user:
      url:   /job/:company/:location/:id/:position
      param: { module: job, action: show }

If you refresh the Jobeet homepage, the links to jobs have not
changed. That's because to generate a route, you need to pass all
the required variables. So, you need to change the ``url_for()``
call in ``indexSuccess.php`` to:

::

    <?php
    url_for('job/show?id='.$job->getId().'&company='.$job->getCompany().
      '&location='.$job->getLocation().'&position='.$job->getPosition())

An internal URI can also be expressed as an array:

::

    <?php
    url_for(array(
      'module'   => 'job',
      'action'   => 'show',
      'id'       => $job->getId(),
      'company'  => $job->getCompany(),
      'location' => $job->getLocation(),
      'position' => $job->getPosition(),
    ))

Requirements
------------

At the beginning of the book, we talked about validation and error
handling for good reasons. The routing system has a built-in
validation feature. Each pattern variable
can be validated by a regular expression defined using the
``requirements|Requirements`` entry of a
route definition:

::

    [yml]
    job_show_user:
      url:   /job/:company/:location/:id/:position
      param: { module: job, action: show }
      requirements:
        id: \d+

The above ``requirements`` entry forces the ``id`` to be a numeric
value. If not, the route won't match.

Route Class
-----------

Each route defined in ``routing.yml`` is internally
converted to an object of class
```sfRoute`` <http://www.symfony-project.org/api/1_4/sfRoute>`_.
This class can be changed by defining a ``class`` entry in the
route definition. If you are familiar with the HTTP
protocol, you know that it defines several "methods", like
``GET|GET (HTTP Method)``,
``POST|POST (HTTP Method)``,
``HEAD|HEAD (HTTP Method)``,
``DELETE|DELETE (HTTP Method)``, and
``PUT|PUT (HTTP Method)``. The first three are supported
by all browsers, while the other two are not.

To restrict a route to only match for certain request methods, you
can change the route class to
```sfRequestRoute`` <http://www.symfony-project.org/api/1_4/sfRequestRoute>`_
and add a requirement for the virtual ``sf_method`` variable:

::

    [yml]
    job_show_user:
      url:   /job/:company/:location/:id/:position
      class: sfRequestRoute
      param: { module: job, action: show }
      requirements:
        id: \d+
        sf_method: [get]

    **NOTE** Requiring a route to only match for some ~HTTP
    methods\|HTTP Method~ is not totally equivalent to using
    ``sfWebRequest::isMethod()`` in your actions. That's because the
    routing will continue to look for a matching route if the method
    does not match the expected one.


Object Route Class
------------------

The new internal URI for a job is quite long and tedious to write
(``url_for('job/show?id='.$job->getId().'&company='.$job->getCompany().'&location='.$job->getLocation().'&position='.$job->getPosition())``),
but as we have just learned in the previous section, the route
class can be changed. For the ``job_show_user`` route, it is better
to use
```sfPropelRoute`` <http://www.symfony-project.org/api/1_4/sfPropelRoute>`_
as the class is optimized for routes that represent ##ORM## objects
or collections of ##ORM## objects:

::

    [yml]
    job_show_user:
      url:     /job/:company/:location/:id/:position
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: show }
      requirements:
        id: \d+
        sf_method: [get]

The ``options`` entry customizes the behavior of the route. Here,
the ``model`` option defines the ##ORM## model class
(``JobeetJob``) related to the route, and the ``type`` option
defines that this route is tied to one object (you can also use
``list`` if a route represents a collection of objects).

The ``job_show_user`` route is now aware of its relation with
``JobeetJob`` and so we can simplify the
``url_for()`` call to:

::

    <?php
    url_for(array('sf_route' => 'job_show_user', 'sf_subject' => $job))

or just:

::

    <?php
    url_for('job_show_user', $job)

    **NOTE** The first example is useful when you need to pass more
    arguments than just the object.


It works because all variables in the route have a corresponding
accessor in the ``JobeetJob`` class (for instance, the ``company``
route variable is replaced with the value of ``getCompany()``).

If you have a look at generated URLs, they are not quite yet as we
want them to be:

::

    http://www.jobeet.com.localhost/frontend_dev.php/job/Sensio+Labs/Paris%2C+France/1/Web+Developer

We need to "slugify" the column values by
replacing all non ASCII characters by a ``-``. Open the
``JobeetJob`` file and add the following methods to the class:

::

    <?php

// lib/model/JobeetJob.php //
lib/model/doctrine/JobeetJob.class.php public function
getCompanySlug() { return Jobeet::slugify($this->getCompany()); }

::

    public function getPositionSlug()
    {
      return Jobeet::slugify($this->getPosition());
    }
    
    public function getLocationSlug()
    {
      return Jobeet::slugify($this->getLocation());
    }

Then, create the ``lib/Jobeet.class.php`` file and add the
``slugify`` method in it:

::

    <?php
    // lib/Jobeet.class.php
    class Jobeet
    {
      static public function slugify($text)
      {
        // replace all non letters or digits by -
        $text = preg_replace('/\W+/', '-', $text);
    
        // trim and lowercase
        $text = strtolower(trim($text, '-'));
    
        return $text;
      }
    }

    **NOTE** In this tutorial, we never show the opening ``<?php``
    statement in the code examples that only contain pure PHP code to
    optimize space and save some trees. You should obviously remember
    to add it whenever you create a new PHP file. Just remember to not
    add it to template files.


We have defined three new "virtual" accessors:
``getCompanySlug()``, ``getPositionSlug()``, and
``getLocationSlug()``. They return their corresponding column value
after applying it the ``slugify()`` method. Now, you can replace
the real column names by these virtual ones in the
``job_show_user`` route:

::

    [yml]
    job_show_user:
      url:     /job/:company_slug/:location_slug/:id/:position_slug
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: show }
      requirements:
        id: \d+
        sf_method: [get]

You will now have the expected URLs:

::

    http://www.jobeet.com.localhost/frontend_dev.php/job/sensio-labs/paris-france/1/web-developer

But that's only half the story. The route is able to generate a URL
based on an object, but it is also able to find the object related
to a given URL. The related object can be retrieved with the
``getObject()`` method of the route object. When parsing an
incoming request, the routing stores the matching route object for
you to use in the actions. So, change the ``executeShow()`` method
to use the route object to retrieve the ``Jobeet`` object:

::

    <?php
    class jobActions extends sfActions
    {
      public function executeShow(sfWebRequest $request)
      {
        $this->job = $this->getRoute()->getObject();
    
        $this->forward404Unless($this->job);
      }
    
      // ...
    }

If you try to get a job for an unknown ``id``, you will see a 404
error page but the error message has changed:

.. figure:: http://www.symfony-project.org/images/jobeet/1_4/05/404_propel_route.png
   :alt: 404 with sfPropelRoute
   
   404 with sfPropelRoute

That's because the 404 error has been thrown for you
automatically by the ``getRoute()`` method. So, we can simplify the
``executeShow`` method even more:

::

    <?php
    class jobActions extends sfActions
    {
      public function executeShow(sfWebRequest $request)
      {
        $this->job = $this->getRoute()->getObject();
      }
    
      // ...
    }

    **TIP** If you don't want the route to generate a 404 error, you
    can set the ``allow_empty`` routing option to ``true``.


-

    **NOTE** The related object of a route is lazy loaded. It is only
    retrieved from the database if you call the ``getRoute()``
    method.


Routing in Actions and Templates
--------------------------------

In a template, the ``url_for()`` helper converts an internal URI to
an external URL. Some other symfony helpers also take an internal
URI as an argument, like the ``link_to()`` helper which
generates an ``<a>`` tag:

::

    <?php
    <?php echo link_to($job->getPosition(), 'job_show_user', $job) ?>

It generates the following HTML code:

::

    <?php
    <a href="/job/sensio-labs/paris-france/1/web-developer">Web Developer</a>

Both ``url_for()`` and ``link_to()`` can also generate absolute
URLs:

::

    <?php
    url_for('job_show_user', $job, true);
    
    link_to($job->getPosition(), 'job_show_user', $job, true);

If you want to generate a URL from an action, you can use the
``generateUrl()`` method:

::

    <?php
    $this->redirect($this->generateUrl('job_show_user', $job));

    **SIDEBAR** The "redirect" Methods Family

    Yesterday, we talked about the "forward" methods. These methods
    forward the current request to another action without a round-trip
    with the browser.

    The "redirect" methods redirect the user to another URL. As with
    forward, you can use the ``redirect()`` method, or the
    ``redirectIf()`` and ``redirectUnless()`` shortcut methods.


Collection Route Class
----------------------

For the ``job`` module, we have already customized the ``show``
action route, but the URLs for the others methods (``index``,
``new``, ``edit``, ``create``, ``update``, and ``delete``) are
still managed by the ``default`` route:

::

    [yml]
    default:
      url: /:module/:action/*

The ``default`` route is a great way to start coding without
defining too many routes. But as the route acts as a "catch-all",
it cannot be configured for specific needs.

As all ``job`` actions are related to the ``JobeetJob`` model
class, we can easily define a custom ``sfPropelRoute``
route for each as we have already done for the ``show`` action. But
as the ``job`` module defines the classic seven actions possible
for a model, we can also use the
```sfPropelRouteCollection`` <http://www.symfony-project.org/api/1_4/sfPropelRouteCollection>`_
class. Open the ``routing.yml`` file and modify it to read as
follows:

::

    [yml]
    # apps/frontend/config/routing.yml
    job:
      class:   sfPropelRouteCollection
      options: { model: JobeetJob }
    
    job_show_user:
      url:     /job/:company_slug/:location_slug/:id/:position_slug
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: show }
      requirements:
        id: \d+
        sf_method: [get]
    
    # default rules
    homepage:
      url:   /
      param: { module: job, action: index }
    
    default_index:
      url:   /:module
      param: { action: index }
    
    default:
      url:   /:module/:action/*

The ``job`` route above is really just a shortcut that
automatically generates the following seven ``sfPropelRoute``
routes:

::

    [yml]
    job:
      url:     /job.:sf_format
      class:   sfPropelRoute
      options: { model: JobeetJob, type: list }
      param:   { module: job, action: index, sf_format: html }
      requirements: { sf_method: get }
    
    job_new:
      url:     /job/new.:sf_format
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: new, sf_format: html }
      requirements: { sf_method: get }
    
    job_create:
      url:     /job.:sf_format
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: create, sf_format: html }
      requirements: { sf_method: post }
    
    job_edit:
      url:     /job/:id/edit.:sf_format
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: edit, sf_format: html }
      requirements: { sf_method: get }
    
    job_update:
      url:     /job/:id.:sf_format
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: update, sf_format: html }
      requirements: { sf_method: put }
    
    job_delete:
      url:     /job/:id.:sf_format
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: delete, sf_format: html }
      requirements: { sf_method: delete }
    
    job_show:
      url:     /job/:id.:sf_format
      class:   sfPropelRoute
      options: { model: JobeetJob, type: object }
      param:   { module: job, action: show, sf_format: html }
      requirements: { sf_method: get }

    **NOTE** Some routes generated by ``sfPropelRouteCollection`` have
    the same URL. The routing is still able to use them
    because they all have different HTTP method
    requirements.


The ``job_delete`` and ``job_update`` routes requires ~HTTP
methods\|HTTP Method~ that are not supported by browsers
(``DELETE|DELETE (HTTP Method)`` and
``PUT|PUT (HTTP Method)`` respectively). This works
because symfony simulates them. Open the ``_form.php`` template to
see an example:

::

    <?php
    // apps/frontend/modules/job/templates/_form.php
    <form action="..." ...>
    <?php if (!$form->getObject()->isNew()): ?>
      <input type="hidden" name="sf_method" value="PUT" />
    <?php endif; ?>
    
    <?php echo link_to(
      'Delete',
      'job/delete?id='.$form->getObject()->getId(),
      array('method' => 'delete', 'confirm' => 'Are you sure?')
    ) ?>

All the symfony helpers can be told to simulate whatever HTTP
method you want by passing the special ``sf_method`` parameter.

    **NOTE** symfony has other special parameters like ``sf_method``,
    all starting with the ``sf_`` prefix. In the
    generated routes above, you can see another one: ``sf_format``,
    which will be explained further in this book.


Route Debugging
---------------

When you use collection routes, it is sometimes useful to list the
generated routes. The ``app:routes`` task outputs all the routes
for a given application:

::

    $ php symfony app:routes frontend

You can also have a lot of debugging information
for a route by passing its name as an additional argument:

::

    $ php symfony app:routes frontend job_edit

Default Routes
--------------

It is a good practice to define routes for all
your URLs. As the ``job`` route defines all the routes needed to
describe the Jobeet application, go ahead and remove or comment the
default routes from the ``routing.yml`` configuration file:

::

    [yml]
    # apps/frontend/config/routing.yml
    #default_index:
    #  url:   /:module
    #  param: { action: index }
    #
    #default:
    #  url:   /:module/:action/*

The Jobeet application must still work as before.

Final Thoughts
--------------

Today was packed with a lot of new information. You have learned
how to use the routing framework of symfony and how to decouple
your URLs from the technical implementation.

Tomorrow, we won't introduce any new concept, but rather spend time
going deeper into what we've covered so far.

**ORM**


