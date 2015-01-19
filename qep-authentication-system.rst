.. _qep#[.#]:

=============================================================================
QGIS Enhancement ##: Authentication configuration system with master password
=============================================================================

:Date: 2015/01/01
:Author: Larry Shaffer
:Contact: lshaffer at boundlessgeo dot com
:Last Edited: 2015/01/18
:Status: Draft
:Version: QGIS 2.8
:Sponsor: Boundless - New York, NY  USA
:Sponsor URL: http://boundlessgeo.com/

1. Summary
----------

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
settings, thereby allowing auth configs to be securely stored in plain text
application components, e.g. project and settings files, without disclosure of
credentials.

Add a variety of common authentication providers, e.g. HTTP Basic, SSL/PKI,
etc.; and, integrate resource access routines with calls to the auth manager,
which marshals an appropriate call to the provider linked to any auth config ID
assigned to the resource.

For example, if a WMS data source URI string contains ``authid=j2r5tvq`` and
such an auth config's provider type was related to HTTP[S] connections, then
calls to the manager might update the connection's ``QNetworkRequest`` and maybe
handle parts of ``QNetworkReply`` (e.g. override expected SSL errors), as
necessary. The details on how or why the provider handles the update to network
objects are abstracted, with regards to the integrated routine creating the
network connection. Such a routine merely calls the manager when any updates to
network objects *might* need to happen.

User interaction with the auth system would be done via GUI widgets for:

* *inputting* the master password in a cross-thread manner (and via console)
* *maintaining* the auth configs in the database
* *creating/editing* individual auth configs, based upon ID
* *selecting* existing auth config IDs to associate with resources/servers

See `Pull Request (PR) #1838 <https://github.com/qgis/QGIS/pull/1838>`_

Note about PR:

  The PR is a completely functional core implementation, with Python bindings,
  unit tests, integration with OWS service connections and support for PKI
  client certificates (in PEM, DER or PKCS#12 format) for HTTPS connections. In
  addition, it has been built and run on most major platforms and has received
  extensive functional testing over several months.

.. _PR: https://github.com/qgis/QGIS/pull/1838

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

  --authdbdirectory | -a  "directory for authentication database"

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

3.1. Added Classes/Files
........................

1. ``QgsAuthManager`` [core/qgsauthenticationmanager.h]

   - Singleton that oversees all master password and auth database functions and
     marshalling of auth providers
   - Inherits from new ``QgsSingleton<type>`` template setup
   - Instantiates in ``QgsApplication::initQgis()`` and cleans up in
     ``QgsApplication::exitQgis()``

2. ``QgsAuthCrypto`` [core/qgsauthenticationcrypto.h]

   - Simple interface for hashing/verifying master password and encrypt/decrypt
     operations on auth configs with master password.
   - Currently uses QCA, though originally designed for `CryptoPP`_, which was
     found to be *way too finicky* to `build on Windows`_, especially for
     non-devs.

   .. _CryptoPP: http://www.cryptopp.com/
   .. _build on Windows: http://www.codeproject.com/Articles/16388/Compiling-and-Integrating-Crypto-into-the-Microsof

3. ``QgsAuthConfigBase``, ``QgsAuthConfig*`` [core/qgsauthenticationconfig.h]

   - Base and subclasses for representing auth configs
   - Has public properties that can generally be queried without requiring the
     user to input the master password
   - Has sensitive properties that become semi-public once the master password
     is set/verified and the config has been retrieved and decrypted from the
     auth database by the auth manager
   - Has sensitive properties that can be set and then encrypted and stored in
     the auth database by the auth manager

4. ``QgsAuthProvider``, ``QgsAuthProvider*`` [core/qgsauthenticationprovider.h]

   - Base and subclasses for representing auth config providers
   - Each provider accepts marshaled calls from the auth manager to update
     authentication-specific objects when needed, e.g. ``QNetworkRequest`` and
     ``QNetworkReply`` during HTTP[S] connections.
   - Each provider has an in-memory cache of authentication objects, generated
     during the processing of an auth config, that are stored upon first
     access/load of the config. Subsequent calls use the cached resource, e.g.
     generated SSL certificate, key and CA chain objects.

5. ``QgsAuthType`` [core/qgsauthenticationconfig.h]

   - Utility class for handling mapping between textual and enum representation
     of a authentication type (for both config and provider)

6. ``QgsAuthUtils`` [gui/qgsauthenticationutils.h]

   - Utility functions for managing the auth database and master password, and
     passing any messages to user via ``QgsMessageBar``

3.2 Added Widgets
.................

1. Master password input dialog [gui/qgscredentialdialog.h]

   User is prompted whenever accessing the auth system, or whenever a layer is
   loaded/dragged/programmatically added that has an associated ``authid``.

   .. figure:: figures/auth-system/masterpass_new.png
      :align: center

      When master password has not been set, nor its hash stored in auth
      database. **The master password can NOT be retrieved if the user looses
      it.**

   .. figure:: figures/auth-system/masterpass_current.png
      :align: center

      After master password has been configured

2. ``QgsAuthConfigEditor`` [gui/qgsauthenticationconfigeditor.h]

   - An embeddable or standalone widget for directly managing auth configs in
     the auth database
   - Uses ``QSqlTableModel`` for its ``QTableView`` model
   - Offers utility functions for managing the auth database and master password

   .. figure:: figures/auth-system/auth-editor.png
      :align: center

3. ``QgsAuthConfigWidget`` [gui/qgsauthenticationconfigwidget.h]

   - An embeddable or standalone widget for creating/editing auth configs
     directly in the auth database
   - Depending upon provider, does lightweight validation, e.g. cert issue dates

   .. figure:: figures/auth-system/auth-configwidget_create.png
      :align: center

      Standalone config creation

   .. figure:: figures/auth-system/auth-configwidget_edit.png
      :align: center

      Standalone with existing config in edit mode

4. ``QgsAuthConfigSelect`` [gui/qgsauthenticationconfigselect.h]

   - An embeddable or standalone widget for selecting/adding/removing auth
     configs in the auth database

   .. figure:: figures/auth-system/auth-selector_noselection.png
      :align: center

      Standalone with no selection defined

   .. figure:: figures/auth-system/auth-selector_wms-integration.png
      :align: center

      Integrated in WMS connection dialog, with config defined

5. Settings -> Authentication menu actions [gui/qgsauthenticationutils.h]

   - Utility functions for managing the auth database and master password

   .. figure:: figures/auth-system/settings-menu_auth.png
      :align: center

6. ``QgsMasterPasswordResetDialog`` [gui/qgsauthenticationutils.h]

   - An embeddable or standalone widget for resetting master password and
     re-encrypting auth configs into a new auth database, with optional backup
     of old database (no Python binding)
   - Redundant validation of master password input is required, regardless of
     whether master password has already been entered.

   .. figure:: figures/auth-system/auth-resetpass.png
      :align: center

3.3 Python Bindings
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
implementation, *no* wall against PyQGIS access has been defined. This will lead
to issues where a user downloads/installs a malicious PyQGIS plugin or
standalone app that gains access to auth credentials.

A simple, though not robust, fix is to add a combobox in
``Options -> Authentication`` (defaults to "never")::

  "Allow Python access to authentication system"
  Choices: [ confirm once per session | always confirm | always allow | never ]

Such an option's setting would need to be saved in a location non-accessible to
Python, e.g. the auth database, and encrypted with the master password.

Another option may be to track which plugins the user has specifically allowed
to access the auth system, though it may be tricky to deduce which plugin is
actually making the call.

Alternatively, access to sensitive auth system data from Python could *never* be
allowed, and only the use of QGIS core widgets, or duplicating auth system
integrations, would allow the plugin to work with resources that have an
``authid``, while keeping master password and auth config loading in the realm
of the main app.

The same security concerns apply to C++ plugins, though it will be harder to
restrict access, since there is no function binding to simply be removed as with
Python.

6. Further Improvements
-----------------------

General auth system improvements to be considered:

* Have security guru/firm audit implementation (I'm no guru)
* Integrate auth system with database connection configs
* Integrate auth system with Plugin Manager connections
* Integrate auth system with all HTTP connection classes
* Integrate auth system into QGIS Server, where user prompts are not supported
* Finish implementation of auth config 'resource' (auto-auth via matched
  resource URI)
* Warn user when secure parts of auth system are accessed by Python (see
  above)
* Add conversion button, to convert existing plain auth to auth config
* Add optional ability to edit the auth id for a configuration (user must
  confirm before operation)
* Add ability to change, edit or remove authid from existing layer in
  properties dialog
* Add simple read-only text field in layer properties dialog to quickly copy
  authid
* Add copy/paste/add/edit/remove of authid in layer contextual menu in Legend
  panel
* On layer load, notify user if any associated authid is missing in auth
  database
* Add layer authid (re)assignment in Handle Bad Layers dialog (can currently
  edit URI)
* Add authid as attribute of base QgsMapLayer class, so it can be queried
* Add checkValidity(bool verbose = false) to auth providers that emits
  messages
* Add Test Connection functions/buttons and connection debug dialog
* Add support for no-master-password encryption (or never add this?)
* Better auth system integration for all browser dock functions
* Find means of clearing cached connections in QgsNetworkManager

Specific to PKI and SSL certificate management:

* Add certificate manager (like Firefox's) to store personal, server and CA
  certs
* Store certificates (input via manager) in new table in auth database
* Add auth provider to work with certificates in auth database
* Allow user-created auth config and provider subclasses to be registered with
  auth manager (necessary? browsers don't do it; currently hard-coded)
* Add auth provider for directly accessing user's OS-specific cert store
  (problematic if client key has passphrase on Windows)
* Better cert/key/trust chain validation in edit widget
* Add cert/key/trust chain detailed info dialogs
* Check for expired/invalid cert/chain prior to connection
* Add access to peer cert and local cert info in error dialogs

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
With its initial OWS connection support it is side-by-side with the previous
username/password form widget in connection setup forms, allowing any existing,
older configurations to continue to work. Likewise, in the ``QNetworkRequest``
and ``QNetworkReply`` updates, the new auth system configuration is only
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
