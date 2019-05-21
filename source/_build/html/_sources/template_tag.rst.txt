=========================
Django roles template tag
=========================

``django_roles_access`` *template tag* is a method to restrict access by role to
content.

``django_roles_access`` decorator, mixin, or middleware, restrict access to a
view. In this way, restriction is applied to all content of the view. With
``django_roles_access`` *template tag* is possible to restrict access to
portions of the content in a view.

``django_roles_access`` *template tag* is used in Django templates.

Example for using ``django_roles_access`` template tag:
::

    {% load roles_tags %}
    ...
    {% if request.user|check_role:'reports_menu' %}
        restricted content
    {% endif %}

If exists a :class:`django_roles_access.models.TemplateAccess` object with a
value in *flag* attribute equal to *reports_menu*; then only users belonging to
any :class:`django.contrib.auth.models.Group` which has been added to *roles*
attribute of the :class:`django_roles_access.models.TemplateAccess` object
will see the restricted content.

.. note::

   ``django_roles_access`` *template_tag* require DjangoTemplate backend to be
   configured in *settings file*. If not, when trying to use it, an exception
   will be raised:
   ::

       django.core.exceptions.ImproperlyConfigured: No DjangoTemplates
       backend is configured.

