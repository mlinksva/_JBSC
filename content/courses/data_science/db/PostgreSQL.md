---
title: Relational Database
linktitle: Relational Database
toc: true
type: docs
draft: false
date: "2019-10-01"
lastmod: "2019-10-01"
menu:
  data_science:
    parent: Data Science
    weight: 3

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---

A Relational Database Management System (RDBMS) is software that implements a digital database based on the relational model of data, as proposed by Codd ([1970](https://doi.org/10.1145/362384.362685)). This system allows for efficient storage and operations on tabular data and commonly implements the Structured Query Language (SQL). Other alternatives for storing and managing tabular data includes flat-files (i.e., text files), data formats such as JavaScript Object Notation (JSON), and binary files such as Hierarchical Data Format (e.g., HDF5). For non tabular data other alternatives such as NoSQL approaches are better suited.

<center>
  <img
    src="/img/postgres.png"
    alt="Postgresql elephant: Slonik"
    width="500px"
  />
</center>

For most cases, PostgreSQL (or Postgres) is one of the best solutions and provides a very robust set of functionality. The easiest way to get started is through the desktop [app](https://postgresapp.com/) for OSX or through a Docker image (e.g., [geographica/postgis](https://hub.docker.com/r/geographica/postgis)). Both solutions include the [postgis](https://postgis.net/) extension for spatial functionality.

<center>
  <img
    src="/img/postgis.png"
    alt="Postgis Logo"
    width="200px"
    copyright="Under Fair Use from Refractions Research - Artist: Lana Lounsbury for PostGIS"
  />
</center>


[pgAdmin](https://www.pgadmin.org/): the most popular and feature rich Open Source administration and development platform for PostgreSQL, the most advanced Open Source database in the world, is a great tool to use with PostgreSQL. It can be used through Docker with [dpage/pgadmin4:latest](https://hub.docker.com/r/dpage/pgadmin4/).

Try it out locally!

A few hints,

- Use the Docker desktop app or a Docker compose file.
- For PostgreSQL
  - Make the publishing port the default `5432:5432`
  - Map a volume for persistency (e.g., `/data:~/MEGAsync/db/postgres12`).
- For pgAdmin, don't forget to the required environmental variables
  - `PGADMIN_DEFAULT_EMAIL`
  - `PGADMIN_DEFAULT_PASSWORD`
- The host name/address would be `host.docker.internal`
- The default username and password are `postgres`.

[postgres]: https://en.wikipedia.org/wiki/PostgreSQL#/media/File:Postgresql_elephant.svg "PostgreSQL Logo, "
