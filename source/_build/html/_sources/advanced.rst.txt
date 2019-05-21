.. _`Required views and app name`:

===========================
Required views and app name
===========================

An important requirement is to give names to views and applications.

This is an implicit requirement if you expect to use all characteristics of
``django_roles_access``. This is necessary because the value used by
``django_roles_access`` to identify the views to be protected is it's name.

Also application name, or namespace, are used to get complete view name:

* ``app_name:view_name``.

* ``namespace:view_name``.

Despite what has been said above, you can also protect the view using it's
name directly, if, and only if, no application name or namespace is given.
But if no application name is given; then the view security status will be
reported under *Undefined app* in ``checkviewaccess`` action.

Read more about Naming URL in official Django project documentation
`Naming URL`_. And examples can be found in Django project tutorial at
`Namespacing URL names`_.

.. _`Naming URL`: https://docs.djangoproject.com/en/dev/topics/http/urls/#naming-url-patterns

.. _`Namespacing URL names`: https://docs.djangoproject.com/en/dev/intro/tutorial03/#namespacing-url-names


.. _`Namespace and View Name`:

========================
Namespaces and View Name
========================

When creating a :class:`django_roles_access.models.ViewAccess` object, the
value of **view** attribute can be:

* `view_name`
* `app_name:view_name`
* `namespace:view_name`
* `nest_namespace:namespace:view_name`


============================
Django roles access response
============================

When a user try to access a view, and this access result in a forbidden action,
is possible to setup different responses:

----------------------------
DJANGO_ROLES_ACCESS_REDIRECT
----------------------------

By default ``django_roles_access`` will response with
:class:`django.http.HttpResponseForbidden` when the user has no access to the
view. This behavior can be changed if you add the attribute
`DJANGO_ROLES_ACCESS_REDIRECT` in *settings files* with a value equal to True:
::

    ...
    DJANGO_ROLES_ACCESS_REDIRECT = True
    ...

The answer given to a user without access is a
:class:`django.http.HttpResponseRedirect` to the value in *settings.LOGIN_URL*.

-------------------------------------
DJANGO_ROLES_ACCESS_FORBIDDEN_MESSAGE
-------------------------------------

When ``django_roles_access`` answer with
:class:`django.http.HttpResponseForbidden`, the message used by default is:
``<h1>403 Forbidden</h1>``; but this configuration can also be changed if a new
attribute named ``DJANGO_ROLES_ACCESS_FORBIDDEN_MESSAGE`` is added in the
*settings file* with the message to be returned instead of default one.
