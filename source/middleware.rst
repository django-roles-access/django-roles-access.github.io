.. _`Using Django roles access middleware`:

====================================
Using Django roles access middleware
====================================

In :ref:`Quick start` was explained how to configure in two steps a security
access for a view; but this requires *new code*: in the present context it is
assumed that adding a decorator or a mixin is adding new code.

It is also possible to use ``django_roles_access`` as middleware, for example in
one of the following two scenarios:

* You do not want to add new code to the project.

* The size of the project demands a solution for many applications, including
  third-party solutions

For example, a requirement may be that all users must log in to access any v
iew of a particular application. Possible solutions to this requirement
(among others) can be:

* Use *login_required*. in all applications views.

* Use a hook in URLConf configuration.

* Use the ``django_roles_access`` middleware and declare the application of
  *SECURED* type

.. note::

   To know more about the classifications of the applications read after
   installation and configuration: :ref:`Applications type`.

------------
Installation
------------

In Django's site *settings* file add
*django_roles_access.middleware.RolesMiddleware* to MIDDLEWARE variable:

::

    MIDDLEWARE = [
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django_roles_access.middleware.RolesMiddleware',
        ...
    ]


.. warning::

   Once the middleware is installed, to change the behavior of a view
   belonging to a particular application, at least one
   :class:`django_roles_access.models.ViewAccess` object needs to be created
   for that view. Otherwise there will be no change in the behavior of any view
   unless an application is classified with a type as explained below
   (:ref:`Applications type`).


-------------
Configuration
-------------

The last step is to classify installed applications in one of the security
groups in Django site's *settings* file.

When an application is classified in one of the security groups, and the
middleware is installed, all accesses to the application views will have access
restricted, or not, according to the default behavior of the selected
security group.


.. _`Applications type`:

=================
Applications type
=================

To classify the applications in security groups (*applications type*), and with
it their security by default, it is necessary to add at least some of the
following variables to the *settings file*:

* NOT_SECURED

* PUBLIC

* SECURED

* DISABLED

.. note::

  By default if none of these variables is declared in the *settings file*, the
  views of the applications will still have the same type of access they had
  before installing ``django_roles_access`` and / or its middleware.

-----------
NOT_SECURED
-----------
List of applications without security. The views of these applications will
continue to have the same security they had had before installing
``django_roles_access``.

The concept for this security group is to contain all those applications that
do not have views and therefore it is not necessary to restrict access.

.. warning::

    If an application is classified as NOT_SECURED, and has views, it does not
    matter if there are :class:`django_roles_access.models.ViewAccess` objects
    for the views, the security of those objects will be ignored.


------
PUBLIC
------
List of applications used mainly with public access.

PUBLIC applications have their views accessible to anonymous users. The
exception would be if :class:`django_roles_access.models.ViewAccess` object
exist for one or more of its views, and this configuration results in more
restrictive access.

.. note::

   If a view of an application classified as PUBLIC has its access restricted
   by some other means (for example login_required decorator), the
   restricted access will take precedence.


-------
SECURED
-------
List of applications that require users to be *Authorized*.

The default behavior of applications classified as SECURED is to require the
user to log in.

--------
DISABLED
--------

List of disabled applications. Applications listed as DISABLED will have all
their views as forbidden for any user.

If an installed application, for whatever reasons, it is required not to be
accessible, for example: maintenance, correction, etc; this type of
classification allows denying access to the mass without the need to modify the
code of all those applications that make use of models or other
functions/classes implemented by the application that would result in a server
failure if the application is simply removed from the list of installed
applications.
