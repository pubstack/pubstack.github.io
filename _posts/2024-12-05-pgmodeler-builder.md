---
layout: post
title: "PostgreSQL Modeling Workflow with pgModeler and binaries from pgmodeler-builder"
author: "Carlos Camacho"
categories:
  - blog
tags:
  # - draft
  - cloud
  - databases
  - pgmodeler
  - PostgreSQL
  - engineering
# hidden: true
favorite: false
commentIssueId: 96
refimage: '/static/pgmodeler.png'
---

If you work with PostgreSQL databases and have a passion for
open-source tools, you’ve likely heard of
**pgModeler**—a powerful, multiplatform database modeler.
But what if you could have or need consistent, automated builds of this essential
tool across major platforms like Linux, Windows, and macOS?
Enter
[**pgmodeler-builder**](https://github.com/ccamacho/pgmodeler-builder)
[https://github.com/ccamacho/pgmodeler-builder](https://github.com/ccamacho/pgmodeler-builder),
a repository designed to
make monthly builds of pgModeler seamless and accessible.

### What is **pgmodeler-builder**?

**pgmodeler-builder** is a GitHub Actions-powered workflow
designed to generate monthly builds of **pgModeler**.
It ensures that the tool is always up-to-date for users on **Linux**, **Windows**, and **macOS**.
The releases includes:

- **.tarball, and .zip** files for compiled sources.
- **.AppImage** for Linux users.
- **.dmg** for ARM-based macOS.
- **.exe** for Windows.

### Why pgModeler?

**pgModeler** is a standout in the PostgreSQL ecosystem.
As an open-source and cross-platform database modeler,
it simplifies database design and management with
features tailored for PostgreSQL.
It’s the go-to solution for developers and DBAs who prioritize free and open-source software.

### Key Benefits of pgmodeler-builder

This repository brings several advantages to the table:

1. **Automated, Regular Builds**: Ensure you’re using the latest version with scheduled monthly releases.
2. **Cross-Platform Compatibility**: Access builds tailored to your operating system.
3. **Streamlined Updates**: The repository makes it easy to add new releases or versions with minimal effort.
4. **Community-Driven Improvements**: Contributions are welcomed, allowing developers to refine and optimize workflows further.
