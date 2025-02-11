# QGIS Enhancement: Dameng Database Now Available as QGIS Data Provider

**Date** 2025/02/11

**Author** DMZHY

**Contact** zhaohaiyang at dameng dot com

**Version** QGIS 3.42

# Summary

This proposal aims to include a new database source as data provider in QGIS. The new features include:

1. Create and edit connections to Dameng databases, with support for various connection parameter options.
2. Edit and preview schema, table, and field information in Dameng databases.
3. Identify and view information about spatial data tables (Geometry, Geography, Topology), including data types and attributes.
4. Preview graphics through spatial data tables, and create and edit layers.
5. Use the SQL editor to create and execute SQL statements.
6. Support layer editing and modifications, with synchronization to the Dameng database.
7. Enable the import and export of layers, supporting project storage and loading, as well as layer style storage and loading.
8. Support browser projects and open Dameng data sources in the form of a data source manager.

## My Section

*(optional)* Insert custom sections wherever needed

## Proposed Solution

To address the support for spatial data in Dameng databases, this proposal aims to add support for Dameng databases as a database source provider. Without affecting existing functionalities, the Dameng database, as a data source, should incorporate as many features as possible that QGIS provides for database providers.

Similar to PostgreSQL and Oracle, the connection to the Dameng database, execution of SQL statements, and retrieval of result sets will be achieved in QGIS using the interface tool dmdpi provided by Dameng. Interaction with the Dameng database will be carried out by writing and executing SQL statements to access relevant data. Spatial functionalities can be implemented by calling Dameng Spatial functions or stored procedures.

For the UI, new interfaces for connecting to Dameng databases and project storage will be added. Simplified Chinese translations have been included (refer to `qgis_zh-Hans.ts`), and it is recommended to support as many languages as possible.

## Deliverables

**Connecting to Dameng**

The spatial features in Dameng Spatial aid users in managing geographic and location data in a native type within an Dameng database. In addition to some of the options in [Creating a stored Connection](https://docs.qgis.org/3.34/en/docs/user_manual/managing_data_source/opening_data.html#vector-create-stored-connection), the connection dialog proposes:

- **Name**： A name for this connection. It can be the same as **Database**.
- **Host**：Name of the database host. This must be a resolvable host name such as would be used to open a TCP/IP connection or ping the host. If the database is on the same computer as QGIS, simply enter *localhost* here.
- **Port**：Port number the Dameng database server listens on. The default port for Dameng is `5236`.

**Authentication**, basic.

- **User name**: User name used to log in to the database.
- **Password**: Password used with *Username* to connect to the database.

Optionally, depending on the type of database, you can activate the following checkboxes:

- [![复选框](https://docs.qgis.org/3.34/en/_images/checkbox.png)](https://docs.qgis.org/3.34/en/_images/checkbox.png) **Don’t resolve type of unrestricted columns (GEOMETRY)**
- [![复选框](https://docs.qgis.org/3.34/en/_images/checkbox.png)](https://docs.qgis.org/3.34/en/_images/checkbox.png)**Only look in the ‘SYSDBA’ schema**
- [![复选框](https://docs.qgis.org/3.34/en/_images/checkbox.png)](https://docs.qgis.org/3.34/en/_images/checkbox.png) **Also list tables with no geometry**
- [![复选框](https://docs.qgis.org/3.34/en/_images/checkbox.png)](https://docs.qgis.org/3.34/en/_images/checkbox.png) **Use estimated table metadata**
- [![复选框](https://docs.qgis.org/3.34/en/_images/checkbox.png)](https://docs.qgis.org/3.34/en/_images/checkbox.png) **Allow saving/loading QGIS projects in the database**

1. QgsDamengConn

   Connection to Dameng Database；

   Executes SQL Statements；

   Conversion and Processing of Dameng Spatial Data Types:；

   Database Transaction Management.

2. QgsDamengConnectionItem

   Connection feature in the "Browser" panel.

3. QgsDamengConnPool

   Creates and initializes Dameng connections.

4. QgsDamengConnPoolGroup

   Manages the Dameng database connection pool, supporting automatic management for timed-out connections.

5. QgsDamengDataItemGuiProvider

   UI dialog for Dameng data item functionality in the "Browser" panel.

6. QgsDamengDataItemProvider

   Provider for Dameng data items.

7. QgsDamengExpressionCompiler

   Expression compiler for the Dameng provider.

8. QgsDamengFeatureIterator

   Feature iterator for vector data, retrieving and processing spatial data from the Dameng database.

9. QgsDamengFeatureSource

   Retrieves feature source data information from the Dameng database.

10. QgsDamengGeomColumnTypeThread

    Detects geometry types and SRIDs of Dameng Spatial tables in a separate thread.

11. QgsDamengLayerItem

    Layer (database table) functionality item in the "Browser" panel.

12. QgsDamengNewConnection

    Dialog for creating new Dameng database connections;

    Handling connection parameters.

13. QgsDamengProjectStorage

    Reads, writes, and deletes QGIS project data from the database.

14. QgsDamengProjectStorageDialog

    Manages and configures parameters for the project storage dialog.

15. QgsDamengProvider

    Dameng Provider provides database connection and transaction operation;

    Manages database table and field information;

    Supports CRUD operations for database tables and spatial data tables;

    Handles QGIS layer operations and retrieves layer information.

16. QgsDamengProviderConnection

    Implements Dameng connection functionality in the "Browser" panel：

    Executes SQL statements and retrieves result sets；

    Creates and deletes database schemas；

    Creates and deletes spatial data indexes;

    Retrieves schema, table, and field information under the current user.

17. QgsDamengProviderGuiMetadata

    GUI management for Dameng provider data sources.

18. QgsDamengProviderMetadata

    Metadata for the Dameng database provider's data source information.

19. QgsDamengProviderResultIterator

    Iterator for SQL result sets.

20. QgsDamengResult

    Wrapper for QgsDMResult, used to retrieve SQL execution results.

21. QgsDamengRootItem

    Root directory item for Dameng databases in the "Browser" panel.

22. QgsDamengSchemaItem

    Schema functionality item in the "Browser" panel.

23. QgsDamengSharedData

    Data shared between the provider class and its feature sources (e.g., FID, feature counts).

24. QgsDamengSourceSelect

    Implements connection and management functionalities for Dameng databases in the Data Source Manager.

25. QgsDamengSourceSelectDelegate

    Creates an editor for Dameng in the Data Source Manager;

    Configures UI display information.

26. QgsDamengTableModel

    Manages and configures table information under Dameng database schemas in the Data Source Manager.

27. QgsDamengTransaction

    Starts, commits, and rolls back transactions;

    Creates and rolls back to transaction savepoints.

28. QgsDamengUtils

    Handles formatting for Dameng arrays and strings.

29. QgsDMDriver

    Wrapper for the `dmdpi` interface, implementing Dameng database connection;

    SQL execution functionalities.

30. QgsDMResult

    Wrapper for the `dmdpi` interface, retrieving and processing SQL execution result sets.

31. QgsDMResultPrivate

    Handles `dmdpi` statement handles.

32. QgsDMDriverPrivate

    Handles `dmdpi` connection handles.

33. QgsPoolDamengConn

    Acquires and releases connections from the connection pool.

### Example(s)

*(optional)*

1. Database Connection

   ![image-20250206152013618](.\images\image-newdmconn.png)

2. Browser and Data Source Manager

   ![image-20250206153720750](.\images\image-browser.png)

   ![image-20250207091020899](.\images\image-datasource.png)

3. Preview Layer Properties

   ![image-20250206153855595](.\images\image-layerproperty.png)

4. Layer Editing (Adding Data) and Synchronization to Dameng Database

   ![image-20250206154047212](.\images\image-addfeature.png)

   ![image-20250206154139863](.\images\image-synchronization.png)

5. SQL Editor

   ![image-20250206154308405](.\images\image-execsql.png)

6. Export Project/Layer Style to Database

   ![image-20250206163709155](.\images\image-saveproj.png)

   ![image-20250206163757322](.\images\image-savestyle.png)

### Affected Files

*(optional)*

## Risks

Only Chinese and English have been localized；

New features may not have the same level of coverage as databases like PostgreSQL and Oracle.

## Performance Implications

**(required if known at design time)**

## Further Considerations/Improvements

Some notes for the possible future improvements:

- Dameng Provider support for raster、PointCloud data types;
- Dameng Provider compatibility with the **DB Manager** plugin;
- If feasible, expand support for more localized languages in the Dameng Provider interface.

## Backwards Compatibility

**(required if applicable)**

## Issue Tracking ID(s)

*(optional)*
