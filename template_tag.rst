=========================
Django roles template tag
=========================

Django roles template tag is



For using Django roles template tag:
::

    {% load roles_tags %}
    ...
    {% if request.user|check_role:'reports_menu' %}
        check_access
    {% endif %}


Is required to configure a DjangoTemplate backend in *settings* file. If not,
when trying to use it, an exception will be raised:
::

    django.core.exceptions.ImproperlyConfigured: No DjangoTemplates backend is
    configured.

