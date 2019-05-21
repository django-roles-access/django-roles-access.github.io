============
Introduction
============

``django_roles_access`` is a Django app for securing access to views. It's
built on top of :class:`django.contrib.auth.models.Group` interpreted as role.

The objective of the app is to provide secure access to views without requiring
new code. Security can be managed at runtime. The app also provides a
security report by registering ``checkviewaccess`` action.

-------------
Main features
-------------

* Once configured, all administration for secure access is done from
  admin site at runtime.

* Not extra code required:

  * When using ``django_roles_access`` middleware no more code is required.

  * In case of using decorator or mixin of ``django_roles_access`` all that is
    needed is to decorate the view function or to prepend the mixin in class
    based view definition.

* Access to views can be controlled with two elements:

  1. View access object: An object created and configured in admin site.

  2. Application classifications in project settings file.

* If ``django_roles_access`` is installed but nothing else is done (only add
  ``django_roles_access`` to settings.INSTALLED_APPS) there will be no change
  in the behavior of the views.

* ``django_roles_access`` register an action called ``checkviewaccess`` that
  will report Django sites views access security when using
  ``django_roles_access`` tools. To use it just:

  ``django_roles_access`` registers an action called ``checkviewaccess`` that
  will report the security to access the views that uses the tools
  provided by ``django_roles_access``

::

    python manage.py checkviewaccess


============
Requirements
============

``django_roles_access`` uses :class:`django.contrib.auth.models.Group`,
:class:`django.contrib.auth.models.User`. Also *Django admin interface* is
necessary to create and administrate *views access*
(:class:`django_roles_access.models.ViewAccess`).
So Django roles access is dependent of *Django admin site* and because of
this it have the same requirements than it. This can be checked in the
official documentation: `Django admin site`_.

.. _`Django admin site`: https://docs.djangoproject.com/en/dev/ref/contrib/admin/

Last requirement is to give names to views and applications. Read more about
this in :ref:`Required views and app name`.

.. _`Quick start`:

===========
Quick start
===========

------------------------------
Installation and configuration
------------------------------

1. Install ``django_roles_access`` from pypi:

::

   pip install django-roles-access

2. Add *'django_roles_access'* to your INSTALLED_APPS setting:

::

   INSTALLED_APPS = [
       ...
       'django_roles_access',
   ]


3. Run migrations to create the ``django_roles_access`` models:
::

    python manage.py migrate

.. note::

   If nothing else is done, then Django site security keeps without
   modification.

--------------------
Access configuration
--------------------

Configuration in two steps.

Step 1
======

In *Django admin* interface create a
:class:`django_roles_access.models.ViewAccess` object and configure it:

1. **view** attribute: Name of the view you want to secure. More about
   this in :ref:`Namespace and View Name`.

2. **type** attribute: select the *access type* for the view:

   * **Public**: Any visitor can access the view.

   * **Authorized**: Only authorized (logged)
     :class:`django.contrib.auth.models.User` can access the view.

   * **By roles**: Only :class:`django.contrib.auth.models.User` belonging to
     any added :class:`django.contrib.auth.models.Group` will access the view.

3. **roles** attribute: When *By roles* is selected as *access type*, this
   attribute hold any :class:`django.contrib.auth.models.Group`
   whose members will access the view.


Step 2
======

Use :func:`django_roles_access.decorators.access_by_role` decorator or
:class:`django_roles_access.mixin.RolesMixin` mixin in the view to be secured.

For example:

In case the view is a function:
::

    from django_roles_access.decorators import access_by_role

    @access_by_role()
    def myview(request):
        ...


In case of classes based views use mixin:
::

    from django_roles_access.mixin import RolesMixin

    class MyView(RolesMixin, View):

        ...

.. note::

   When user has no access to a view, by default ``django_roles_access``
   will response with :class:`django.http.HttpResponseForbidden`.

.. note::

   Pre existent security behavior will be modified if a ``django_role_access``
   configuration for the same view results in a more restrictive access; for
   example forbidden access.




