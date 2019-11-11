---
title: OS-level virtualization systems
linktitle: Containers
toc: true
type: docs
draft: false
date: "2019-11-10"
lastmod: "2019-11-10"
menu:
  data_science:
    parent: Data Science
    weight: 2

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---

OS-level virtualization is a key feature of data science. The idea is to have a contained environment that provides functionality, security, and replicability.

For single machines one of the best implementations of OS-level virtualization is [Docker](https://www.docker.com/). However, in multi-user systems such a high-performance computing (HPC), the preferred solution is [Singularity](https://sylabs.io/).

<center>
  <a href="https://www.docker.com/">
    <img
      src="/img/docker.png"
      alt="Docker: Moby Dock whale logo"
      width="350px"
    />
  </a>
</center>

The easiest way to start becoming familiar with Docker is through the GUI application [Kitematic](https://kitematic.com/) part of the Docker Desktop [application](https://docs.docker.com/install/).
