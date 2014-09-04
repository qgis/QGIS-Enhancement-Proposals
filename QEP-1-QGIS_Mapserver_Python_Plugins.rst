.. _qep#[.#]:

========================================================================
QGIS RFC 2: QGIS Server Python Plugins
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

QGIS Desktop takes enormous advantages from the additional features implemented with Python Plugins, this proposal is about extending Python plugins to the server side.

The rationale behind server plugins is doublefold: first they will provide additional services without the need to touch the C++ codebase, second, they will allow for GUI-based configuration from QGIS Desktop.

Possible applications:

* web clients configuration
* authentication/authorization
* additional services (WPS etc.)

Python plugins for the server side are not meant to replace the existing professional solutions available for heavy traffic websites but will provide ready to use, easy to install and easy to configure alternatives for the average user.

Server plugins will also broaden the number of developers for server extensions, that could eventually become part of the core, as happened many times for QGIS desktop.


#. Proposed [Technical] Solution | Change
-----------------------------------------

The proposed (simplest and unobtrusive) solution is to load Python plugins that declare a *service* metatag and match the *SERVICE* parameter in the query string to run the plugin. The plugin must also expose the *REQUEST* methods it provides.

A possible further enhancement would allow the plugins to listen to particular signals emitted, for example:

* at FCGI startup
* after the request object is created
* before the response is sent to the client

#. Implementation Details
-------------------------

The proposed solution is to load Python plugins when FCGI application starts and maintain an hash of services declared by the plugins.

When the FCGI loop ends without a match for the available C++ services (WMS, WFS, WCS), if (and only) python plugins have been registered in the hash and of course Python support is enabled, 404 handler runs and matches the SERVICE parameter with the *service* metatag declared by the plugin.

If a plugin exists for the required *SERVICE*, additional checks are made to verify if it can run:

* the *REQUEST* parameter must match the CSV list of plugin's exposed methods, as declared in the *methods* metadata
* the plugin loads without errors
* the plugin has a method named as the *REQUEST*


Example query string::

    http://localhost/cgi-bin/qgis_mapserv.fcgi?SERVICE=HELLO&REQUEST=SayHello

When all checks are done the plugin runs by calling the method in the *REQUEST*, the method receives the project path and the query string as parameters.


Example metadata::

    service=HELLO
    methods=GetCapabilities,GetOutput,RemoteConsole,SayHello


The plugin (static) method has access to global python environment and can run arbitrary python commands, it can optionally return a content type string (the server defaults to ``text/plain``).

The plugin CGI-style output is captured diverting ``stdout`` and ``stderr`` to a custom buffer which becomes the server response.

Example method::

    @staticmethod
    def SayHello(project_path, parameters):
        """Just say something"""
        print "HelloServer"
        return 'text/plain'



Example response::

    200 OK
    Connection: close
    Date: Thu, 04 Sep 2014 09:56:36 GMT
    Server: Apache/2.4.7 (Ubuntu)
    Vary: Accept-Encoding
    Content-Length: 12
    Content-Type: text/plain
    Client-Date: Thu, 04 Sep 2014 09:56:36 GMT
    Client-Peer: 127.0.0.1:80
    Client-Response-Num: 1

    HelloServer


Because in the proposed implementation the server plugins are not separated from the desktop plugins, the user could take advantage (without being forced to) of the well known QGIS Desktop plugins interface system to configure the behaviour of the QGIS Server side of the plugin.

The proposed implementation do not enforce the fact that a plugin must have a Desktop interface just allows for it.

Since we would be mixing Desktop and Server side, the environment for those plugins that have a Desktop interface should be carefully configured with proper permissions to allow information (configuration files) sharing from the desktop user to the webserver user.



#.# Example(s)
..............

An example plugin is available:

https://github.com/elpaso/qgis-helloserver


#.# Python Bindings
...................

Only in case we decide to pass cached parsed project to the plugins, a python wrapper to the cached projects would be needed.


#.# Affected Files
..................

The proposed implementation adds a cmake flag to enable the functionality and just a few lines of code to ``qgis_map_serv.cpp``, the implementation is done in a separated class contained in a single implementation/header C++ couple.

In order to allow headless loading of plugins, is was also necessary to provide a new environment variable wich defines ``mConfigPath`` which normally defaults to a path starting from the user home directory (www-data doesn't define one). The addition of this environment variable ``QGIS_CUSTOM_CONFIG_PATH`` has no impact on the rest of QGIS.

#. Test Coverage
----------------



#. Performance Implications
---------------------------

Loading the Python machinery at FCGI startup causes a small delay, subsequent calls to non python services (WMS, WFS etc.) will not cause any delay because those calls will never hit the 404 handler that obviously comes last.

A quick test done with ``ab`` compares the a.m. example call (SayHello) with the standard exception output::

    SAYHELLO REQUEST

    ab -r -n 1000 -c 100  'http://qwc/cgi-bin/qgis_mapserv.fcgi?SERVICE=HELLO&request=SayHello'
    This is ApacheBench, Version 2.3 <$Revision: 1528965 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking qwc (be patient)
    Completed 100 requests
    Completed 200 requests
    Completed 300 requests
    Completed 400 requests
    Completed 500 requests
    Completed 600 requests
    Completed 700 requests
    Completed 800 requests
    Completed 900 requests
    Completed 1000 requests
    Finished 1000 requests


    Server Software:        Apache/2.4.7
    Server Hostname:        qwc
    Server Port:            80

    Document Path:          /cgi-bin/qgis_mapserv.fcgi?SERVICE=HELLO&request=SayHello
    Document Length:        12 bytes

    Concurrency Level:      100
    Time taken for tests:   1.370 seconds
    Complete requests:      1000
    Failed requests:        0
    Total transferred:      187000 bytes
    HTML transferred:       12000 bytes
    Requests per second:    729.93 [#/sec] (mean)
    Time per request:       137.000 [ms] (mean)
    Time per request:       1.370 [ms] (mean, across all concurrent requests)
    Transfer rate:          133.30 [Kbytes/sec] received

    Connection Times (ms)
                min  mean[+/-sd] median   max
    Connect:        0    1   1.5      0       6
    Processing:     2   40  62.3     38    1043
    Waiting:        2   40  62.3     38    1043
    Total:          8   41  62.4     38    1044

    Percentage of the requests served within a certain time (ms)
    50%     38
    66%     39
    75%     39
    80%     40
    90%     40
    95%     41
    98%     42
    99%     46
    100%   1044 (longest request)


    EMPTY REQUEST (Exception)

    ab -r -n 1000 -c 100  'http://qwc/cgi-bin/qgis_mapserv.fcgi'
    This is ApacheBench, Version 2.3 <$Revision: 1528965 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking qwc (be patient)
    Completed 100 requests
    Completed 200 requests
    Completed 300 requests
    Completed 400 requests
    Completed 500 requests
    Completed 600 requests
    Completed 700 requests
    Completed 800 requests
    Completed 900 requests
    Completed 1000 requests
    Finished 1000 requests


    Server Software:        Apache/2.4.7
    Server Hostname:        qwc
    Server Port:            80

    Document Path:          /cgi-bin/qgis_mapserv.fcgi
    Document Length:        225 bytes

    Concurrency Level:      100
    Time taken for tests:   1.111 seconds
    Complete requests:      1000
    Failed requests:        0
    Total transferred:      399000 bytes
    HTML transferred:       225000 bytes
    Requests per second:    899.77 [#/sec] (mean)
    Time per request:       111.139 [ms] (mean)
    Time per request:       1.111 [ms] (mean, across all concurrent requests)
    Transfer rate:          350.59 [Kbytes/sec] received

    Connection Times (ms)
                min  mean[+/-sd] median   max
    Connect:        0    0   0.9      0       3
    Processing:     2   27  62.6     24    1025
    Waiting:        2   27  62.6     24    1025
    Total:          6   27  62.7     24    1025

    Percentage of the requests served within a certain time (ms)
    50%     24
    66%     25
    75%     25
    80%     25
    90%     26
    95%     27
    98%     29
    99%     30
    100%   1025 (longest request)


In case we decide to implement signals/slots bound to request start and response we could expect an impact in performances that would probably be almost negligible in case there is no match with any listening plugin.



#. Further Considerations/Improvements
--------------------------------------

A possible further enhancement would allow the plugins to listen to particular signals emitted at least:

* after the request object is created
* before the response is sent to the client

Other enhancements could wrap the cached parsed project C++ object and pass it to the plugins, this would allow the plugins to interact with the parsed project without the need to parse the project file again.


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


