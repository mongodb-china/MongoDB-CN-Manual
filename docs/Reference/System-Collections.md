# System Collections

On this page

- [Synopsis](https://docs.mongodb.com/master/reference/system-collections/#synopsis)
- [Collections](https://docs.mongodb.com/master/reference/system-collections/#collections)

## Synopsis

MongoDB stores system information in collections that use the `<database>.system.*` [namespace](https://docs.mongodb.com/master/reference/glossary/#term-namespace), which MongoDB reserves for internal use. Do not create collections that begin with `system`.

MongoDB also stores some additional instance-local metadata in the [local database](https://docs.mongodb.com/master/reference/local-database/), specifically for replication purposes and in the [config database](https://docs.mongodb.com/master/reference/config-database/) for [sessions information](https://docs.mongodb.com/master/core/read-isolation-consistency-recency/#sessions).

## Collections

System collections include these collections stored in the `admin` database:

- `admin.system.``roles`

  The [`admin.system.roles`](https://docs.mongodb.com/master/reference/system-collections/#admin.system.roles) collection stores custom roles that administrators create and assign to users to provide access to specific resources.

- `admin.system.``users`

  The [`admin.system.users`](https://docs.mongodb.com/master/reference/system-collections/#admin.system.users) collection stores the user’s authentication credentials as well as any roles assigned to the user. Users may define authorization roles in the [`admin.system.roles`](https://docs.mongodb.com/master/reference/system-collections/#admin.system.roles) collection.

- `admin.system.``version`

  Stores the schema version of the user credential documents.

System collections include these collections stored in the `config` database:

- `config.system.``indexBuilds`

  *New in version 4.4.*The [`indexBuilds`](https://docs.mongodb.com/master/reference/system-collections/#config.system.indexBuilds) collection stores information related to in-progress index builds.

System collections also include these collections stored directly in each database:

- `<database>.system.``namespaces`

  REMOVED IN 4.2Starting in MongoDB 4.2, `<database>.system.namespaces` has been removed (access to the collection has been deprecated since 3.0). To list the collections in a database, use the [`listCollections`](https://docs.mongodb.com/master/reference/command/listCollections/#dbcmd.listCollections) command instead.

- `<database>.system.``indexes`

  REMOVED IN 4.2Starting in MongoDB 4.2, `<database>.system.indexes` has been removed (access to the collection has been deprecated since 3.0). To list the inndexes, use the [`listIndexes`](https://docs.mongodb.com/master/reference/command/listIndexes/#dbcmd.listIndexes) command instead.

- `<database>.system.``profile`

  The [`.system.profile`](https://docs.mongodb.com/master/reference/system-collections/#.system.profile) collection stores database profiling information. For information on profiling, see [Database Profiling](https://docs.mongodb.com/master/administration/analyzing-mongodb-performance/#database-profiling).

- `<database>.system.``js`

  The [`.system.js`](https://docs.mongodb.com/master/reference/system-collections/#.system.js) collection holds special JavaScript code for use in [server side JavaScript](https://docs.mongodb.com/master/core/server-side-javascript/). See [Store a JavaScript Function on the Server](https://docs.mongodb.com/master/tutorial/store-javascript-function-on-server/) for more information.

- `<database>.system.``views`

  The [`.system.views`](https://docs.mongodb.com/master/reference/system-collections/#.system.views) collection contains information about each [view](https://docs.mongodb.com/master/core/views/) in the database.