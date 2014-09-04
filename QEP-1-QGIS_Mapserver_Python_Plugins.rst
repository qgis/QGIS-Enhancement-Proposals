.. _qep#[.#]:

========================================================================
QGIS Enhancement #: QGIS Server Python Plugins
========================================================================

:Date: 2014/09/04
:Author: Alessandro Pasotti
:Contact: apasotti at itopen dot it
:Last Edited: 2014/09/04
:Status:  Draft
:Version: QGIS 2.5

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

#. Summary
----------

QGIS Desktop takes enormous advantages from the additional features implemented with Python Plugins. The rationale behind server plugins is doublefold: first they could provide additional services without the need to touch the C++ codebase, second, they allow for GUI-based configuration from QGIS Desktop.

Possible applications:

* web clients configuration
* authentication/authorization
* additional services (WPS etc.)


#. Proposed [Technical] Solution | Change
-----------------------------------------

The proposed (simplest and unobtrusive) solution is to load Python plugins that declare a *service* metatag and match the *SERVICE* parameter in the query string to run the plugin. The plugin must also expose the *REQUEST* methods it provides.

A possible further enhancement would allow the plugins to listen to particular signals emitted at least:

* after the request object is created
* before the response is sent to the client

#. Implementation Details
-------------------------

The proposed (simplest and unobtrusive) solution is to load Python plugins when FCGI application starts and maintain an hash of services declared by the plugins.

When the FCGI loops ends without a match for the available C++ services (WMS, WFS, WCS), if (and only) python plugins have been registered in the hash and of course Python support is enabled, 404 handler runs and matches the SERVICE parameter with the *service* metatag declared by the plugin.

If a plugin exists for the required *SERVICE*, additional checks are made to verify if it can run:

* the *REQUEST* parameter must match the CSV list of plugin's exposed methods, as declared in the *methods* metadata
* the plugin loads without errors
* the plugin has a method named as the *REQUEST*


When all checks are done the plugin runs by calling the method in the *REQUEST*, the method receives the project path and the query string as parameters.


Example metadata::

    service=HELLO
    methods=GetCapabilities,GetOutput,RemoteConsole,SayHello


The plugin (static) method has access to global python environment and can run arbitrary python commands, it can optionally return a content type string (the server defaults to `text/plain`).

Example method::

    @staticmethod
    def SayHello(project_path, parameters):
        """Just say a sentence"""
        print "HelloServer"
        return 'text/plain'



Because in the proposed implementation the server plugins are not separated from the desktop plugins, the user could take advantage (without being forced to) of the well known QGIS Desktop plugins interface system to configure the behaviour of the QGIS Server side of the plugin.

The proposed implementation do not enforce the fact that a plugin must have a Desktop interface just allows for it.

Since we would be mixing Desktop and Server side, the environment for those plugins that have a Desktop interface should be carefully configured with proper permissions to allow information sharing from the desktop user to the webserver user.



#.# Example(s)
..............

An example plugin is available:

https://github.com/elpaso/qgis-helloserver


#.# Python Bindings
...................

Only in case we decide to pass cached parsed project to the plugins, a python wrapper to the cached projects would be needed.


#.# Affected Files
..................

The proposed implementation adds a cmake flag to enable the functionality and just a few lines of code to `qgis_map_serv.cpp`, the implementation is done in a separated class contained in a single implementation/header C++ couple.

#. Test Coverage
----------------



#. Performance Implications
---------------------------

Loading the Python machinery at FCGI startup causes a small delay, subsequent calls to non python services (WMS, WFS etc.) will not cause any delay because those calls will never hit the 404 handler that obviously comes last.

In case we decide to implement signals/slots bound to request start and response we could expect an impact in performances that would probably be almost negligible in case there is no match with any listening plugin.



#. Further Considerations/Improvements
--------------------------------------

A possible further enhancement would allow the plugins to listen to particular signals emitted at least:

* after the request object is created
* before the response is sent to the client

Other enhancements could wrap the cached parsed project C++ object and pass it to the plugins, this would allow the plugins to interact with the parsed project without the need to parse the project file again.


#. Restrictions
---------------

(optional)

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


