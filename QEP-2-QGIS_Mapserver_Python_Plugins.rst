.. _qep#[.#]:

========================================================================
QGIS RFC 2: QGIS Server Python Plugins
========================================================================

:Date: 2014/09/04
:Author: Alessandro Pasotti
:Contact: apasotti at itopen dot it
:Last Edited: 2014/09/05
:Status:  Draft
:Version: QGIS 2.5

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

#. Summary
----------

QGIS Desktop takes enormous advantages from the additional features implemented with Python Plugins, this proposal is about extending Python plugins to the server side.

The rationale behind server plugins is doublefold: first they lower the barrier for adding new functionalities to existing core services and they provide additional services without the need to touch the C++ codebase, second, they will optionally allow for GUI-based configuration from QGIS Desktop.

Possible applications:

* web clients configuration
* authentication/authorization
* additional services (WPS etc.)
* new output formats

Python plugins for the server side are not meant to replace the existing professional solutions available for heavy traffic websites but will provide ready to use, easy to install and easy to configure alternatives for the average user.

Server plugins will also broaden the number of developers for server extensions, that could eventually become part of the core, as happened many times for QGIS desktop.


#. Proposed [Technical] Solution | Change
-----------------------------------------

* Provide hooks to **modify** existing services by manipulating input and output
* Provide hooks to **extend** QGIS server by adding new services

Plugins will follow the **Observer pattern** and listen to a couple of signals emitted from the main loop. The plugins will receive an *interface* object that will allow them to manipulate the request and/or the response and to provide new services in case there is no match with any of the core C++ services (404 handler).


Signals will be emitted at:

* FCGI startup
* after the request object is created
* before the response is sent to the client
* when there is no match with the core services

For a discussion of the different options, please see the slides of the presentation held at Essen QGIS HF 2014: http://www.itopen.it/qgis-server-plugins/


#. Implementation Details
-------------------------

The proposed solution is to load Python plugins and start them when FCGI application starts.

Server-enabled plugins will be created calling a ``serverClassFactory(iface)`` method with a ``serverInterface`` object as parameter.

The plugins will then listen to signals coming from the loop and will be able to manipulate input and output of existing core services and provide new services by calling methods of an ``QgsServerInterface`` object.

Because in the proposed implementation the server plugins **can** coexists with the desktop plugins, the user could take advantage (without being forced to) of the well known QGIS Desktop plugins interface system to configure the behaviour of the QGIS Server side of the plugin.

The proposed implementation do not enforce the fact that a plugin must have a Desktop interface just allows for it.

In case the plugin author decides to provide both a server and desktop interface within the same plugins, the environment for those plugins should be carefully configured with proper permissions to allow information (configuration files) sharing from the desktop user to the webserver user.


#.# Python Bindings
...................

Python bindings will be created for (at least) the following classes:

  * ``QgsCapabilitiesCache``
  * ``QgsMapServiceException``
  * ``QgsRequestHandler``
  * ``QgsServerInterface``
  * ``QgsServerLogger``

#.# Affected Files
..................

The proposed implementation adds a cmake flag to enable the functionality and needs some refactoring of ``qgis_map_serv.cpp`` and the ``QgsRequestHandler`` some more classes are added for the plugin handling and the python bindings.

In order to allow headless loading of plugins, is was also necessary to provide a new environment variable wich defines ``mConfigPath`` that normally defaults to a path descending from the user home directory (the web server user doesn't define one on most Linux distributions). The addition of this environment variable ``QGIS_CUSTOM_CONFIG_PATH`` has no impact on the rest of QGIS.

#. Test Coverage
----------------



#. Performance Implications
---------------------------

Loading the Python machinery at FCGI startup causes a small delay, subsequent calls to non python services (WMS, WFS etc.) will cause a tiny (probably negligible) delay to scan the listeners connected to the emitted signals.

#. Restrictions
---------------

If the plugin has a Desktop interface it cannot usually access to user's ``QSettings``, this means that plugins options have to be stored somewhere else in order to be accessible by the server side.

#. Backwards Compatibility
--------------------------

None

#. Documentation
----------------

None

#. Issue Tracking ID(s)
-----------------------



#. Voting History
-----------------


