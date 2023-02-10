---
layout: default
title: Project Creation
parent: CI Fuzz
nav_order: 4
permalink: ci-fuzz-project-creation
---
# **Project Creation**
{: .no_toc }

This section will guide you through creating a new project in the UI of CI Fuzz.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Prerequisites

Before creating a project, you should have already:

* [Installed CI Fuzz]({{ site.baseurl }}{% link docs/ci-fuzz/installation.md %}).
* [Configured authentication]({{ site.baseurl }}{% link docs/ci-fuzz/authentication.md %}).


## Add New Project

Start by logging into CI Fuzz. From the Dashboard, you can either:
* Click the **Select Project** dropdown and then **Add Project**
* Click the **NEW PROJECT** button

![](/assets/images/ci-fuzz-project-creation/new-project.png)

This will spawn a dialog box to enter information about the project:

* Projects can be customized with a logo of your choosing. If you do not select a logo, then CI Fuzz will generate one for you.
* If a project is marked as Personal, it will not belong to an organization.
* Enter a name for the project.
* If a project is not marked as Personal, then it must belong to an organization.
* If your project currently resides in a Git repository, you can add the URL. This is optional, but recommended if possible.
* You can add a description of your project. 