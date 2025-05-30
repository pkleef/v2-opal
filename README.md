# Creating an OPAL instance using Docker

_Copyright (C) 2025 OpenLink Software <support@openlinksw.com>_

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Introduction](#introduction)
- [Downloading and Running the Example](#downloading-and-running-the-example)
  - [Starting the Example for the first time](#starting-the-example-for-the-first-time)
  - [Start this Docker instance in the background](#start-this-docker-instance-in-the-background)
  - [Stop this Docker Instance](#stop-this-docker-instance)
- [Description of Files and Directories](#description-of-files-and-directories)
  - [The docker-compose.yml File](#the-docker-composeyml-file)
  - [The database Directory](#the-database-directory)
    - [The database/virtuoso.lic File](#the-databasevirtuosolic-file)
  - [The scripts Directory](#the-scripts-directory)
    - [The scripts/10-init-opal.sql File](#the-scripts10-init-opalsql-file)
- [See Also](#see-also)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Introduction

The
[OpenLink Virtuoso Commercial v8.3 Docker Image](https://hub.docker.com/repository/docker/openlink/virtuoso-closedsource-8)
(openlink/virtuoso-commercial-8)
allows users to run a combination of
[shell](https://en.wikipedia.org/wiki/Shell_script) scripts and
[SQL](https://en.wikipedia.org/wiki/SQL) scripts
to initialize a new database.

This example shows how a Virtuoso Docker instance, started by the
`docker compose` command, can install the OpenLink Personal Assistant
layer (OPAL) and related VAD packages.

This example has been tested on both Ubuntu Noble Numbat 24.04
(x86_64) and macOS Big Sur 11.6 (x86_64 and Apple Silicon).

Most modern Linux distributions provide Docker packages as part of
their repository.

For Apple macOS and Microsoft Windows Docker installers can be downloaded from the
[Docker website](https://docker.com/products).

**Note**: Installing software like git, Docker and Docker Compose is left as an exercise for
the reader.


## Downloading and Running the Example

The source code for this example can be cloned from its repository
on GitHub using the following command:

```shell
$ git clone https://github.com/OpenLinkSoftware/virtuoso-opal-docker-example
```

### Starting the Example for the first time

The first time, start this example using the following commands:

```shell
$ cd virtuoso-opal-docker-example
$ docker compose pull
$ docker compose up
```

**Note**: On older releases of Docker, you need to use the
`docker-compose` command.

The first time it takes a couple of minutes to initialize the database,
load all the required packages, and finish by starting up the instance
after which the engine is listening online for requests:

```text
virtuoso-opal-docker-example-virtuoso_db-1  | 13:19:46 SSL server online at 1112
virtuoso-opal-docker-example-virtuoso_db-1  | 13:19:46 HTTP Server threads exceed the number of licensed connections. Setting to 1
virtuoso-opal-docker-example-virtuoso_db-1  | 13:19:46 HTTP/WebDAV server online at 8890
virtuoso-opal-docker-example-virtuoso_db-1  | 13:19:46 HTTPS server online at 8891
virtuoso-opal-docker-example-virtuoso_db-1  | 13:19:46 Server online at 1111 (pid 1)
```

At this point, you can use a browser to connect to the local OPAL
endpoint using https:

```text
https://{CNAME}:8891/chat
```

where `{CNAME}` is either the name the machine on which you installed
docker, or `localhost` if your docker instance and browser are
running on the same machine.

As this instance is using a `Self-Signed SSL Certificate` your
browser will show a warning screen, which allows you to 'Proceed
to {CNAME} (unsafe)`

Once the `OPAL chat` page is loaded, press the `login` button, log in
with your `dba` account using the `secret` password, and press the
`autorise` button to allow the app to use this account.

Next, enter your `OpenAI API key`, press the `Set` button and you
are ready to use the OpenLink Personal Assistant.

To stop Virtuoso, you can press the `CTRL-C` keys in your terminal
window.

### Start this Docker instance in the background

This Docker instance is set up to write the database to your hard
disk in the `database` directory. You can start and stop it as many
times as you want in the background:

```shell
$ cd virtuoso-opal-docker-example
$ docker compose up -d
```

### Stop this Docker Instance

```
$ cd virtuoso-opal-docker-example
$ docker compose down
```

## Description of Files and Directories

### The docker-compose.yml File

```yaml
version: "3.3"
services:
  virtuoso_db:
    image: openlink/virtuoso-closedsource-8
    volumes:
      - ./scripts:/opt/virtuoso/initdb.d
      - ./database:/database
    environment:
      - DBA_PASSWORD=secret
    ports:
      - "1111:1111"
      - "1112:1112"
      - "8890:8890"
      - "8891:8891"
```

### The database Directory

This directory will permanently store your `virtuoso.db` database,
the `virtuoso.ini` configuration file, and other files required by
Virtuoso. The files are created on the first startup of this example,
allowing you to retain data stored in the database between restarts
of this Docker instance.

#### The database/virtuoso.lic File

This example uses the
[OpenLink Virtuoso Commercial 8 Docker image](https://hub.docker.com/repository/docker/openlink/virtuoso-closedsource-8)
and runs by default with a demo license, which allows a single expiring
HTTPS connection to the Virtuoso Commercial Edition database engine.

While this should suffice for initially running this example, you may
want to register an account with us and
[get a 15-day evaluation license](https://shop.openlinksw.com/license_generator/virtuoso/)
emailed to you.

Once you receive this license in your email, you can store the
`virtuoso.lic` file in the database directory of this example and restart
this Docker instance.

For continued use beyond the demo or evaluation period, you need to
purchase a commercial license from the
[OpenLink Software Shop](https://shop.openlinksw.com/shop.vsp?dbms=http%3A%2F%2Fwww.openlinksw.com%2Fontology%2Fsoftware%23OpenLinkVirtuoso)
to obtain a one-year license.

Please contact [the OpenLink Support Team](mailto:support@openlinksw.com)
if you have any questions regarding licensing Virtuoso.


### The scripts Directory

This directory can contain a mix of shell (.sh) scripts and Virtuoso
PL (.sql) scripts that can perform functions such as:

  * Installing additional Ubuntu packages.
  * Loading data from remote locations such as an Amazon S3 bucket, Google Drive, or other locations.
  * Bulk loading data into the Virtuoso database.
  * Installing additional `VAD` packages into the database.
  * Adding new Virtuoso users.
  * Granting permissions to Virtuoso users.
  * Regenerating freetext indexes or other initial data.

The scripts are run only once during the initial database creation;
subsequent restarts of the Docker image will not cause these scripts
to be rerun.

The scripts are run in alphabetical order, so we suggest starting
the script name with a sequence number, to make the ordering explicit.

For security purposes, Virtuoso will run the `.sql` scripts in a
special mode and will not respond to connections on its SQL (1111)
and/or HTTP (8890) ports.

At the end of each `.sql` script, Virtuoso automatically performs
a `checkpoint` to make sure the changes are fully written back to
the database. This is very important for scripts that use the bulk
loader function `rdf_loader_run()` or for any reason manually change
the [ACID](https://en.wikipedia.org/wiki/ACID) mode of the database.

After all the initialization scripts have run to completion, Virtuoso
will be started normally and start listening to requests on its SQL
(1111), SQL SSL (1112), HTTP (8890), and HTTPS (8891) ports.


#### The scripts/10-init-opal.sql File

This script installs the following commercial VAD packages into a new database:

  * OpenLink Virtuoso Faceted Browser (FCT)
  * OpenLink Virtuoso Authentication Layer (VAL)
  * OpenLink Personal AI Layer (OPAL)

```SQL
--
--  Copyright (C) 2025 OpenLink Software
--

VAD_INSTALL ('../vad/fct_dav.vad');
VAD_INSTALL ('../vad/val_dav.vad');
VAD_INSTALL ('../vad/personal_assistant_dav.vad');

--
--  End of script
--
```

## See Also
 * [The OpenLink AI Layer](https://opal.openlinksw.com/)
 * [The OpenLink Community Forum](https://community.openlinksw.com/)
 * [The OpenLink Virtuoso Commercial Edition Docker image](https://hub.docker.com/repository/docker/openlink/virtuoso-closedsource-8/general/) on Docker Hub
 * [The OpenLink Virtuoso Universal Server Documentation](https://docs.openlinksw.com/virtuoso/)
 * [The OpenLink Virtuoso Website](https://virtuoso.openlinksw.com/)
 * [The OpenLink Shop Server](https://shop.openlinksw.com/)
