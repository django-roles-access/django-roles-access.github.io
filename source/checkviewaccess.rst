=================
Check view access
=================

``django_roles_access`` register an action with manage.py: **checkviewaccess**
::

    python manage.py checkviewaccess

This action will analyze all reachable views from settings.ROOT_URLCONF and
print to standard output a *security report* of access to each view.

.. _django_algorithm: https://docs.djangoproject.com/en/dev/topics/http/urls/#how-django-processes-a-request

.. warning::

   Django documentation explain the algorithm the system follows to determine
   which Python code to execute (django_algorithm_):

   ... but if the incoming HttpRequest object has a urlconf attribute (set by
   middleware), its value will be used in place of the ROOT_URLCONF setting.
   ...

   *checkviewaccess* action takes from ROOT_URLCONF the root
   for all site's views. If an installed middleware change this root, registered
   action may not work as expected.

.. note::

   ``django_roles_access`` middleware do not change ROOT_URLCONF or any other
   attribute of the HTTPRequest. HTTPResponse is changed only when access
   denied is reached.

----------------------
Security report format
----------------------

Printed *security report* can have two formats:

* Console format.

* CSV format.


Console format
--------------

When **checkviewaccess** is used without any argument, the output format will
be like:

.. code-block:: text

    python manage.py checkviewaccess
    Start checking views access.
    Start gathering information.
    Finish gathering information.
    Django roles access middleware is active: False.

    ...

        Analyzing django.contrib.messages:
            django.contrib.messages has no type.
            django.contrib.messages does not have configured views.
        Finish analyzing django.contrib.messages.

    ...

        Analyzing app_name:
            app_name is SECURED type.

            Analysis for view: app_name:index
            View url: app_name/
            View access is of type By role.
                Roles with access: role-1, role-2

            Analysis for view: app_name:status
            View url: app_name/status/
            No Django roles tool used. Access to view depends on its implementation.
        Finish analyzing app_name.
    End checking view access.



CSV format
----------

CSV format exports site’s view access with next columns:

* **App Name**: Application name to which belong the view being reported.

* **Type**: Will have one of next values: 'no type','NOT_SECURED', 'PUBLIC',
  'SECURED', 'DISABLED'.

* **View Name**: Name of the view or None.

* **Url**: Regex (django 1.10 and django 1.11) or pattern (django 2+)

* **Status**: Will have one of next values: Normal, Warning, Error.

* **Status description**: If there is a description for the **Status** it will
  reported here; eg cause of error or warning.


To get the report with csv format, execute:

.. code-block:: text

    python manage.py checkviewaccess --output-format csv

Also could be useful to send output to a file and then use other application
to read it's content:

.. code-block:: text

    python manage.py checkviewaccess --output-format csv > checkviewaccess.csv

When **checkviewaccess** is used with csv format, the output format will
be like:

.. code-block:: text

    Reported: 2019-04-09 14:28:04.712023+00:00
    Django roles access middleware is active: True.
    App Name,Type,View Name,Url,Status,Status description
    app1,SECURED,app1:view1,app1/view1/,Normal,
    app2,PUBLIC,app2:view1,app2/,Normal,View access is of typeAuthorized.
    app2,PUBLIC,app2:view2,app2/view2/<int:pk>/,Normal,	Public access ...
    Undefined app,no type,start,,Normal,



----------------------------------------
Django roles access middleware is active
----------------------------------------

::

    Django roles access middleware is active


This indicate if ``django_roles_access`` middleware is being used or not.
When is **True** means middleware is *active* and all views are *protected by*
``django_roles_access``, if and only if the applications have been categorized
in a security group (:ref:`Applications type`) or a
:class:`django_roles_access.models.ViewAccess` objects has been created. When is
**False** means ``django_roles_access`` middleware is not installed.

.. code-block:: text

    python manage.py checkviewaccess
    Start checking views access.
    Start gathering information.
    Finish gathering information.
    Django roles access middleware is active: True.

    ...

        Analyzing django.contrib.messages:
            django.contrib.messages has no type.
            django.contrib.messages does not have configured views.
        Finish analyzing django.contrib.messages.

    ...

        Analyzing app_name:
        Analyzing devices:
            app_name is SECURED type.

            Analysis for view: app_name:index
            View url: app_name/
            No security configured for the view (ViewAccess object) and application type is "SECURED". User is required to be authenticated to access the view.

            Analysis for view: app_name:status
            View url: app_name/status/
            View access is of type By role.
                Roles with access: role-1, role-2
        Finish analyzing app_name.

    ...

        Analyzing Undefined app:
            Undefined app has no type.

            Analysis for view: other-index
            View url: /
            ERROR: Django roles middleware is active; or view is protected with Django roles decorator or mixin, and has no application or application has no type. There are no View Access object for the view. Is not possible to determine behavior for access view. Access to view is determined by view implementation.
        Finish analyzing Undefined app.

    End checking view access.

In above example `Django roles access middleware is active:` is **True**.
If a view belong to an application without configured type
(:ref:`Applications type`) and there is no
:class:`django_roles_access.models.ViewAccess` associated with the view, an
*ERROR* will be reported because there is no default security behavior for
the view. The access to the view depends on it implementation; for example, the
view is decorated with Django *login_required decorator*.

Views not belonging to an application will be classified under an application
with name *Undefined app*. This happens when no name is given to an application.

Applications type
-----------------

If an application has no configured type (:ref:`Applications type`), will be
reported as `app_name has no type`. In any other case the configured type for
the application is reported; for example, `app_name is SECURED type` .
