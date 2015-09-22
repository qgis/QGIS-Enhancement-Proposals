.. _qep14:

=============================================================================
QGIS Enhancement 14: Authentication configuration system with master password
=============================================================================

:Date: 2015/01/01
:Author: Larry Shaffer
:Contact: lshaffer at boundlessgeo dot com
:Last Edited: 2015/09/20
:Status: Draft
:Version: QGIS 2.12
:Sponsor: Boundless - New York, NY  USA
:Sponsor URL: http://boundlessgeo.com/

1. Summary
----------

See `Pull Request (PR) #2330 <https://github.com/qgis/QGIS/pull/2330>`_

.. note::

  The PR is already a completely functional core implementation of this
  proposal, with Python bindings, unit tests, integration with PostGIS and OWS
  service connections using username/password and support for PKI client
  certificates (in PEM, DER or PKCS#12 format) for OWS HTTPS connections. In
  addition, it has been built and run on most major platforms and has received
  *extensive* functional testing over several months.

Currently, QGIS lacks any facility to store/retrieve authentication credentials
in a secure manner. Users either choose to save credentials insecurely in
connection profiles and/or as plain text in their project files, or choose to
not save credentials, thereby burdening themselves with correctly reentering the
credentials per work session.

Connection info can also be insecurely saved as part of a data resource's URI,
which is used to help define full connection profiles in a single URI, to aid
drag/drop operations, and to offer the user convenient/quick connections to data
sources. This can lead to accidental exposure of authentication credentials if a
project file, with credentials saved in it, is shared by a user who is unaware
of the consequences.

Similarly, there is no simple methodology in QGIS of representing complex
authentication configuration components (beyond just ``username`` and
``password``) within the scope of a single data source URI.

2. Proposed Technical Solution
------------------------------

Introduce a core singleton authentication (auth) manager that securely oversees
storage and retrieval of user auth configurations (configs) in a dedicated
SQLite database file, where such configs are encrypted on storage and decrypted
upon retrieval using a master password, known only to the user, that has its
salted and iterated SHA256 hash stored in the database for verification.

No access or usage of an auth config can take place without the user first
entering the correct master password, which unlocks access to the auth database
until the app is quit, or the cached master password is cleared mid-session.

Utilize an auth config's ID (a hash-like 7-character string, e.g. ``j2r5tvq``),
generated during initial storage to the auth database, to abstractly represent
the config as a parameter in data source URIs and in application and plugin
settings; thereby allowing auth configs to be securely stored in plain text
application components, e.g. project and settings files, without disclosure of
credentials.

Add a variety of common authentication methods *as plugins*, e.g. HTTP Basic,
SSL/PKI, etc., and integrate resource access routines with calls to the auth
manager, which marshals an appropriate call to the method plugin associated to
any auth config ID assigned to the resource.

For example, if a WMS data source URI string contains ``authcfg=j2r5tvq`` and
such an auth config's method type was related to HTTP[S] connections, then
calls to the manager might update the connection's ``QNetworkRequest`` as
necessary. The details on how or why the method handles the update to network
objects are abstracted, with regards to the integrated routine creating the
network connection. Such a routine merely calls the manager when any updates to
network objects *might* need to happen.

User interaction with the auth system would be done via GUI widgets for:

* *inputting* the master password in a cross-thread manner (and via console)
* *maintaining* the auth configs in the database
* *creating/editing* individual auth configs, based upon ID
* *selecting* existing auth config IDs to associate with resources/servers

Here is `example user documentation (from Boundless) for a PKI workflow, with an overview of the
authentication system <https://github.com/dakcarto/QGIS-Enhancement-Proposals/blob/auth-system/extras/auth-system/pkiuser.rst>`_.

.. _PR: https://github.com/qgis/QGIS/pull/2330

3. Implementation Details
-------------------------

This describes the completed implementation in the current PR. Additional
possible refactorings are listed in `6. Further Improvements`_.

The authentication database defaults to being created at::

  ~/.qgis2/qgis-auth.db

and contains two tables, one for the master password hash (single row only) and
another for auth configs. This is generated upon authentication manager's
``init()``, if it does not already exist.

Upon command line launch of ``qgis``, the user can define::

  --authdbdirectory | -a  "path to directory for authentication database"

into which a ``qgis-auth.db`` will be created, if it does not already exist.

Alternatively, you can define the ``QGIS_AUTH_DB_DIR_PATH`` environment
variable, which has the same effect as the ``--authdbdirectory`` option.

3.1 Added Dependencies
......................

Adds required build/package dependency upon GPL2-licensed `Qt Cryptographic
Architecture`_ (QCA) 2.0.3+, and a run-time dependency upon QCA's OpenSSL plugin
(qca-ossl). Latest QCA is 2.1.0, which builds on all major platforms using CMake
and includes all available plugins in the build process. A new ``FindQCA.cmake``
module is added in the `PR`_ implementation, as well as build-time CMake
functions that check for qca-ossl if unit tests are to be built.

See `PR`_ for more details on building QCA 2.1.0. QCA's `source repo browser`_
at KDE.

QCA 2.1.0 *is already* `compatible with Qt5`_, while QCA 2.0.3 is not.

.. _Qt Cryptographic Architecture: http://delta.affinix.com/qca/
.. _source repo browser: http://quickgit.kde.org/?p=qca.git
.. _compatible with Qt5: https://projects.kde.org/projects/kdesupport/qca/repository/revisions/master/entry/README

3.1. Main Added Classes/Files
.............................

1. ``QgsAuthManager`` [src/core/auth/qgsauthmanager.h]

   - Singleton that oversees all master password and auth database functions and
     marshalling of auth methods
   - Instantiates in ``QgsApplication::initQgis()`` and cleans up in
     ``QgsApplication::exitQgis()``

2. ``QgsAuthCrypto`` [src/core/auth/qgsauthcrypto.h]

   - Simple interface for hashing/verifying master password and encrypt/decrypt
     operations on auth configs with master password.
   - Currently uses QCA, though originally designed for `CryptoPP`_, which was
     found to be *way too finicky* to `build on Windows`_, especially for
     non-devs.

   .. _CryptoPP: http://www.cryptopp.com/
   .. _build on Windows: http://www.codeproject.com/Articles/16388/Compiling-and-Integrating-Crypto-into-the-Microsof

3. ``QgsAuthMethodConfig*`` [src/core/auth/qgsauthmethodconfig.h]

   - Class representing auth method configs
   - Has public properties that can generally be queried *without* requiring the
     user to input the master password
   - Has sensitive properties that become semi-public once the master password
     is set/verified and the config has been retrieved and decrypted from the
     auth database by the auth manager
   - Has sensitive properties that can be set and then encrypted and stored in
     the auth database by the auth manager

4. ``QgsAuthMethod`` [src/core/auth/qgsauthmethod.h] and ``QgsAuthMethodEdit``

   - Class and edit widget that comprise an auth config method plugin
   - Each method accepts marshaled calls from the auth manager to update
     authentication-specific objects when needed, e.g. ``QNetworkRequest`` and
     ``DataSourceURI``, during resource connections
   - Each method has an in-memory cache of authentication objects, generated
     during the processing of an auth config, that are stored upon first
     access/load of the config. Subsequent calls use the cached resource, e.g.
     generated SSL certificate, key and CA chain objects.

5. ``QgsAuthMethodRegistry`` [src/core/auth/qgsauthmethodregistry.h]  and
   ``QgsAuthMethodMetadata``

   - Singleton plugin registry modeled after ``QgsProviderRegistry``
   - Loads plugins with ``lib*authmethod.(so|dll)`` name pattern

3.2 Main Added GUI Classes
..........................

1. Master password input dialog [src/gui/qgscredentialdialog.h]

   User is prompted whenever accessing the auth system, or whenever a layer is
   loaded/dragged/programmatically added that has an associated ``authcfg``.

   .. figure:: extras/auth-system/img/auth-password-new_enter.png
      :align: center

      When master password has not been set, nor its hash stored in auth
      database. **The master password can NOT be retrieved if the user looses
      it.**

   .. figure:: extras/auth-system/img/auth-password-invalid-3times.png
      :align: center

      After master password has been configured and 3 incorrect attempts

2. ``QgsAuthConfigEditor`` [src/gui/auth/qgsauthconfigeditor.h]

   - An embeddable or standalone widget for directly managing auth configs in
     the auth database
   - Uses ``QSqlTableModel`` for its ``QTableView`` model
   - Offers utility functions for managing the auth database and master password

   .. figure:: extras/auth-system/img/auth-editor.png
      :align: center

3. ``QgsAuthConfigEdit`` [src/gui/auth/qgsauthconfigedit.h]

   - An embeddable or standalone widget for creating/editing auth configs
     directly in the auth database
   - Depending upon method, does lightweight validation, e.g. cert issue dates

   .. figure:: extras/auth-system/img/auth-configwidget_create.png
      :align: center

      Standalone config creation

   .. figure:: extras/auth-system/img/auth-config-create_pkcs12-paths.png
      :align: center

      Standalone with existing config in edit mode

4. ``QgsAuthConfigSelect`` [src/gui/auth/qgsauthconfigselect.h]

   - An embeddable or standalone widget for selecting/adding/editing/removing
     auth configs in the auth database

   .. figure:: extras/auth-system/img/auth-selector_noselection.png
      :align: center

      Standalone with no selection defined

   .. figure:: extras/auth-system/img/auth-selector_wms-integration.png
      :align: center

      Integrated in WMS connection dialog, with config defined

5. Sundry GUI classes

   - ``QgsMasterPasswordResetDialog`` Embeddable or standalone widget for
     resetting master password and re-encrypting auth configs into a new auth
     database, with optional backup of old database (no Python binding)
   - ``QgsAuthGuiUtils`` Utility functions for managing the auth database and
     master password, and passing any messages to user via ``QgsMessageBar``

3.3 PKI/SSL-related Added Classes
.................................

There are several editor widgets and included functionalities within QgsAuthManger
that support PKI and SSL authentication and configuration.

1. ``QgsAuthAuthoritiesEditor`` [src/gui/auth/qgsauthauthoritieseditor.h]

   - Manager for system Certificate Authorities and their trust policies
   - Offers ability to add new CAs from file or load into database

2. ``QgsAuthIdentitiesEditor`` [src/gui/auth/qgsauthidentitieseditor.h]

   - Manager for personal certificate/key PKI components stored in the database

3. ``QgsAuthSslErrorsDialog`` [src/gui/auth/qgsauthsslerrorsdialog.h]

   - New SSL errors dialog that allows saving of server certificate exceptions
   - User can review the SSL certificates associated with the error's connection

4. ``QgsAuthServersEditor`` [src/gui/auth/qgsauthserverseditor.h]

   - Manager for SSL server certificate configurations, e.g exceptions.

3.4 Authentication Methods Plugins
..................................

In the `PR`_ there are three auth method plugins. Additional plugins are now
easy to make and add. The system *can* also be updated to allow methods to be
registered via general application plugins (C++ or PyQGIS).

The current plugins in [src/auth] are:

**Basic**

* Basic username/password for HTTP[S] connections and database credentials

**PKI-Paths**

* Pure Qt SSL code, does not use QCA (CryptoPP was going to be lib supporting
  crypto functions in original implementation)
* Supports CA and client certificate/key in PEM or DER format (PEM is Qt native)
* Client key can be passphrase-protected

**PKI-PKCS#12**

* Uses QCA
* Supports client certificate/key bundles in .p12 or .pfx formats
* Bundle should not include any signing (CA) cert chain (this can be supported)
* Bundle can be passphrase-protected

**Identity-Cert**

* Uses PKI cert/key imported from PEM or PKCS#12 files into Identities table
* Uses QCA for importing PKCS#12

Adding more methods requires subclassing an existing base or method and
adding any new virtual functions to the base class that will handle new means of
applying authentication to integrated code elsewhere in the code base. Any new
virtual functions will need a single, similar marshaling function in the auth
manager to provide an abstracted call based solely upon the ``authcfg``.

3.5 TLS/SSL Connection Authentication
.....................................

QGIS leverages `QNetworkAccessManager`_ via a custom subclass
``QgsNetworkAccessManager`` for managing most network connections. This is a
higher level manager and does not offer a good means of responding to a TLS/SSL
server when it requests that a client provide a certificate for authentication,
e.g. like when Firefox prompts you to select a client cert from its embedded
cert manager in the middle of a connection.

A better TLS/SSL connection solution might be to use Qt's `QSslSocket`_ or
`QCA's TLS class`_. ``QCA::TLA`` has a nice ``certificateRequested`` signal.
However, implementing a new TLS/SSL client socket is beyond the scope of this
QEP/PR.

Instead, I chose to take a 'pre-configure' approach, where users need to define
auth configs for TLS/SSL connections *prior* to a server asking for the client
cert. This should generally not be an issue, though QGIS will not act
Web-browser-like in this regard.

Auth configs currently have an unused 'resource URI' property, which was
designed to be utilized later via a custom user-selected auth config type, e.g.
"Select configuration based upon URI", once support for that is
implemented. Such a feature would mitigate some of the annoyance of the
'pre-configure' approach.

.. _QNetworkAccessManager: http://qt-project.org/doc/qt-4.8/qnetworkaccessmanager.html
.. _QSslSocket: http://qt-project.org/doc/qt-4.8/qsslsocket.html
.. _QCA's TLS class: http://delta.affinix.com/docs/qca/classQCA_1_1TLS.html

3.6 Python Bindings
...................

All classes and public functions have sip bindings, except ``QgsAuthCrypto``,
since management of the master password hashing and auth database encryption
should be handled by the main app, and not via Python.
See `5. Security Considerations`_ concerning Python access below.

4. Test Coverage
----------------

Most coverage is provided by the current unit tests. However, a heavy amount of
functional testing was done for the OWS integration against current real-world
installs of PKI/SSL-enabled GeoServer installs (see `PR`_ for pre-configured
GeoServer test install). Those functional tests can be supplanted with automated
tests set up against a pre-configured ``lighttpd`` instance, which spawns
QGIS Server via FCGI.

5. Security Considerations
--------------------------

Once the master password is entered, the API is open to access auth configs in
the auth database, similar to how Firefox works. However, in the initial
implementation, *no* wall against PyQGIS access has been defined. This may lead
to issues where a user downloads/installs a malicious PyQGIS plugin or
standalone app that gains access to auth credentials.

The quick solution for initial release of feature is to just not include most
PyQGIS bindings for the auth system.

Another simple, though not robust, fix is to add a combobox in
``Options -> Authentication`` (defaults to "never")::

  "Allow Python access to authentication system"
  Choices: [ confirm once per session | always confirm | always allow | never ]

Such an option's setting would need to be saved in a location non-accessible to
Python, e.g. the auth database, and encrypted with the master password.

Another option may be to track which plugins the user has specifically allowed
to access the auth system, though it may be tricky to deduce which plugin is
actually making the call.

Sandboxing plugins, possibly in their own virtual environments, would reduce
'cross-plugin' hacking of auth configs from another plugin that is authorized.
This might mean limiting cross-plugin communication as well, but maybe only
between third-party plugins.

Another good solution is to issue code-signing certificates to vetted plugin
authors. Then validate the plugin's certificate upon loading. If need be the
user can also directly set an untrusted policy for the certificate associated
with the plugin using existing certificate management dialogs.

Alternatively, access to sensitive auth system data from Python could *never* be
allowed, and only the use of QGIS core widgets, or duplicating auth system
integrations, would allow the plugin to work with resources that have an
``authcfg``, while keeping master password and auth config loading in the realm
of the main app.

The same security concerns apply to C++ plugins, though it will be harder to
restrict access, since there is no function binding to simply be removed as with
Python.

6. Further Improvements
-----------------------

General auth system improvements to be considered (no particular order):

* Have security guru/firm audit implementation (I'm no guru)
* Integrate auth system with more database connection configs
* Integrate auth system with Plugin Manager connections
* Integrate auth system with all HTTP connection classes
* Integrate auth system into QGIS Server, where user prompts are not supported
* Integrate master password entry with platform-specific password managers, so
  user does not need to enter it and automated pre-population scripts are easier
  to code
* Finish implementation of auth config 'resource' (auto-auth via matched
  resource URI)
* Try moving auth system integration code (when authcfg has been assigned) into
  ``QgsNetworkAccessManager`` instead of within individual data providers and
  pass authcfg to network manager
* Warn user when secure parts of auth system are accessed by Python (see
  above)
* PyQGIS plugins may need sandboxed to protect against, and selectively allow,
  access to auth configs
* Migrate all 'core' PyQGIS plugins that need to manage their own connections to
  using core auth system calls and/or core connections (which would auto-manage
  auth system calls), or isolate them from third-party plugins, if possible
* Switch master password memory allocation from QString (insecure) to
  ``QCA::SecureArray``
* Add conversion button, to convert existing plain auth to auth config
* Add optional ability to edit the auth id for a configuration (user must
  confirm before operation)
* Add ability to change, edit or remove authcfg from existing layer in
  properties dialog
* Add simple read-only text field in layer properties dialog to quickly copy
  authcfg
* Add copy/paste/add/edit/remove of authcfg in layer contextual menu in Legend
  panel
* On layer load, notify user if any associated authcfg is missing in auth
  database
* Add layer authcfg (re)assignment in Handle Bad Layers dialog (can currently
  edit URI)
* Add authcfg as attribute of base QgsMapLayer class, so it can be queried
* Add checkValidity(bool verbose = false) to auth methods that emits
  messages
* Add Test Connection functions/buttons and connection debug dialog
* Add support for no-master-password encryption (or never add this?)
* Better auth system integration for all browser dock functions
* Find means of clearing cached connections in QgsNetworkManager

Specific to PKI and SSL certificate management:

* Add auth method for directly accessing user's OS-specific cert store
  (problematic if client key has passphrase on Windows)
* Better cert/key/trust chain validation in edit widget
* Check for expired/invalid cert/chain prior to connection

7. Restrictions
---------------

The confusing `licensing and exporting issues`_ associated with OpenSSL apply.
In order for Qt to work with SSL certificates, it needs access to the OpenSSL
libraries. Depending upon how Qt was compiled, the default is to dynamically
link to the OpenSSL libs at run-time (to avoid the export limitations).

QCA follows a similar tactic, whereby linking to QCA incurs no restrictions,
because the ``qca-ossl`` (OpenSSL) plugin is loaded at run-time. The qca-ossl
plugin is directly linked to the OpenSSL libs. Packagers would be the ones
needing to ensure any OpenSSL-linking restrictions are met, *if* they ship the
plugin. Maybe. I don't really know. I'm not a lawyer.

The auth system safely disables itself when ``qca-ossl`` is not found at
run-time.

.. _licensing and exporting issues: http://www.opensslfoundation.com/export/README.blurb

8. Backwards Compatibility
--------------------------

The proposed auth system causes no regressions nor backwards incompatibility.
With its initial PostGIS and OWS connection support it is side-by-side with the previous
username/password form widget in connection setup forms, allowing any existing,
older configurations to continue to work. Likewise, in the ``QNetworkRequest``
and ``DataSourceURI`` updates, the new auth system configuration is only
prioritized once it has been utilized.

9. References
-------------

* `Qt Cryptographic Architecture <http://delta.affinix.com/qca/>`_
* `QCA API Docs <http://delta.affinix.com/docs/qca/index.html>`_
* `QCA and Qt5 <https://github.com/JPNaude/dev_notes/wiki/Using-the-Qt-Cryptographic-Architecture-with-Qt5>`_
* `Salted Password Hashing - Doing it Right <http://www.codeproject.com/Articles/704865/Salted-Password-Hashing-Doing-it-Right>`_
* `QNetwork Module (SSL classes) <http://qt-project.org/doc/qt-4.8/qtnetwork.html>`_

10. Voting History
------------------

(required)
