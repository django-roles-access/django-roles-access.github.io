=================
Check view access
=================

Django roles access register an action with manage.py: **checkviewaccess**
::

    python manage.py checkviewaccess

This action will analyze all reachable views from settings.ROOT_URLCONF and
print to standard output a *security report* of access to each view.

.. _django_algorithm: https://docs.djangoproject.com/en/dev/topics/http/urls/#how-django-processes-a-request

.. warning::

   Django documentation explain the algorithm the system follows to determine
   which Python code to execute django_algorithm_:

   ... but if the incoming HttpRequest object has a urlconf attribute (set by
   middleware), its value will be used in place of the ROOT_URLCONF setting.
   ...

   Django roles access *checkviewaccess* takes from ROOT_URLCONF the root for
   all site's views. If an installed middleware change this root, registered
   action may not work as expected.

.. note::

   Django roles access middleware do not change ROOT_URLCONF or any other
   attribute of the HTTPRequest. HTTPResponse is changed only when access
   denied is reached.

----------------------
Security report format
----------------------

Printed *security report* can have two possible formats:

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

Is possible to export site’s view access in csv format with the next columns:

* App Name: Application name to which belong the view being reported.

* Type: With any of this values: 'no type','NOT_SECURED', 'PUBLIC', 'SECURED'.

* View Name: The name of the vie or None.

* Url: The regex (django 1.10 and django 1.11) or pattern (django 2+)

* Status: With any of this values: Normal, Warning, Error.

* Status description: If there is a description for the state it is
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

This indicate if Django roles access middleware is being used or not. When
**True** means middleware is *active* and all views are *protected by Django
role*. When **False** means Django roles access middleware is not installed.

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
If a view belong to an application without configured type and no
:class:`django_roles_access.models.ViewAccess` associated, an *ERROR* will be
reported because there is no default behavior for the view. The access to it
will depends on it implementation, this mean, for example, the view could be
decorated with Django *login_required decorator*.

Views without application will be classified under an application with
name *Undefined app*.

Applications type
-----------------

When application has no configured type is reported as `app_name has no type`
. In any other case the configured type for the application is reported for
example `app_name is SECURED type`.

--------
Analysis
--------

The used analysis follow next algorithm:

1. All views are collected and grouped by application. In the created report
   this step si called **gathering information**.

2. Is checked if Django roles access middleware is active or not. When active
   the report will indicate **Django roles active for site: True**. When not
   active it will br reported as **Django roles active for site: False**. Also
   analysis will keep track of this state in it's **site_active** variable.

3. For each installed application (settings.INSTALLED_APPLICATION) is checked
   if the application was classified as explained in
   :ref:`Applications type`. The result of this is reported to standard output.

4. For each view of the analyzed application selected in step 3, is checked
   the access security by analyzing the callable associated with the view. This
   analysis include:

   * Evaluate if view is decorated with ``django_roles_access`` decorator, or
     if mixin was used.

   * Search any :class:`django_roles_access.models.ViewAccess` object for the
     view.

   * Take in consideration if **site_active** is True or not.

   * Take in consideration the :ref:`Applications type` of the application
     holding the view.

5. Report from selected view will indicate:

   * View Name.

   * Declared URL.

   * Access security status.

------
Method
------

The used method to determine **Access security status** of a view is:

1. A :class:`django_roles_access.models.ViewAccess` object is searched for the
   view.

2. If **site_active** is True:

   a. If an object was found in step 1, object security is reported for the
      view. If object security is type `By role` and no roles were added an
      ERROR is reported (no one, except superuser, can access de view).

   b. If no object was found; default behavior for view's application is
      reported as explained in :ref:`Applications type`.

   c. If no object was found in step 1. And no application type is
      defined for view's application (or view has no application defined). An
      ERROR of configuration is reported.

3. If **site_active** is False and ``django_roles_access`` decorator or mixin
   was used:

   a. In case exist object found in step 1, object security is reported.

   b. In case no object were found in step 1. And view's application has a
      type as defined in :ref:`Applications type`. Default behavior for the
      application type is reported as view access security.

   c. In case no object were found in step 1. And no application type is
      defined for view's application (or view has no application defined). An
      ERROR of configuration is reported.
