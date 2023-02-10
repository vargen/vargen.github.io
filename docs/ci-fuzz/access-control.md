---
layout: default
title: Access Control
parent: CI Fuzz
nav_order: 3
permalink: ci-fuzz-access-control
---
# **Access Control**
{: .no_toc }

CI Fuzz enables you to collaborate with others through `organizations` and `projects`. An `organization` can be a company, team, or any other group of people that are working on the same set of `projects`. Each `project` belongs to an `organization`. 

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Organization Roles

`Organizations` have two roles: `administrators` and `members`.

*   `Administrators` have complete administrative access to the organization.  
*   `Members` are the default for everybody else. The project role that `members` gain in a project within an organization is configurable by the administrator.

Capabilities of each `organization` role:

| Organization action       | Member    | Administrator |
| :-----------------------: | :-------: | :-----------: |
| List org members          |           |       X       |
| Add member to org         |           |       X       |
| Remove member from org    |           |       X       |
| Delete Org                |           |       X       |
| Manage member permissions |           |       X       |
| View member permissions   |           |       X       |
| View all org projects     |     X     |       X       |
| Add project to org        |     X*    |       X       |

**\*** `organization members` can only perform the `Add project to org` action if the default project role is `Developer` or `Administrator`.


## Creating Organizations

To create an organization:

1. Login to CI Fuzz
2. On the left sidebar select **Organizations**. This will open a new pane listing all the current organizations you belong to.
3. On the new pane in the top right click **NEW ORGANIZATION**.
4. Select the Add manually option by clicking **+ ADD**. 
5. Enter your desired organization name and click **+ ADD**.

## Managing Organizations

Organization management features are accessible by selecting **Organizations** from the left sidebar and then selecting the appropriate organization. The creator of an organization is an `organization administrator` and can manage members of the organization. 

For a given organization, an `organization administrator` can set the default role for project members. This option is located under the **GENERAL** tab in the organization pane. **Note:** All `members` of an `organization` will have this role across all `projects`.

![](/assets/images/ci-fuzz-access-control/set-default-project-role.png)

Under the **MEMBERS** tab in the organization pane, `organization administrators` can add and remove members from the organization. They can also promote other `members` to `administrators`. **Note:** To add a member this way, they must have logged in to CI Fuzz at least once.

![](/assets/images/ci-fuzz-access-control/manage-members.png)

## Customizing Organizations

Visual aspects of each organization can be customized to suit your preferences. After you have created your organization, The options below are all available under a specific organization's settings provided you are an `organization administrator`.

![](/assets/images/ci-fuzz-customize-organizations.png)

### Add Custom Logo

You can add a logo for a specific organization. Whenever someone is interacting with this organization, the logo in the top left of the UI will be the one you applied. You can set a different logo for light mode and dark mode. The appropriate logo will be displayed based on the mode a user has specified in their user settings.

### Create Custom Color Scheme

You can specify a custom color scheme for the UI. Similarly to logos, you can specify a color scheme for an organization based on which mode a user has specified, either light mode or dark mode. Colors can be selected using a color palette by clicking on one of the colored boxes or specifying the appropriate hex code. 

When selecting with the color palette, you can click **SAVE COLOR** in the popup box to preview the color. Colors will not be permanently saved until you select **SAVE SETTINGS** in the bottom right. You can restore the default color scheme at any time by clicking **RESET** and then **SAVE SETTINGS**.


## Project Roles

`Projects` have the following roles:

*   `Observers` have read-only access to a project.
*   `Developers` have read-write access to a project.
*   `Administrators` have full access to a project.

Capabilities of each `project` role:

| Project Action    	| Observer 	| Developer 	| Administrator 	|
| :-------------------:	| :-------:	| :-----------:	| :---------------:	|
| View Findings     	|     X    	|     X     	|       X       	|
| Download Report  	    |     X    	|     X     	|       X       	|
| Start Fuzzing   	    |          	|     X     	|       X       	|
| Configure Fuzzing	    |          	|     X     	|       X       	|
| Configure Project	    |          	|     X     	|       X       	|
| Delete Findings  	    |          	|           	|       X       	|
| Delete Project  	    |          	|           	|       X       	|
| List Members   	    |          	|           	|       X       	|
| Add Members    	    |          	|           	|       X       	|
| Delete Members  	    |          	|           	|       X       	|

## Creating Projects

There are two ways to add a `project`:

1. Go to the **PROJECTS** tab for an `organization` you are a member of and click **ADD PROJECT**.
2. From the left sidebar, under **PROJECTS**, click the **Select Project** dropdown menu and select **Add Project**.

For either method, an **Add Project** windows pops up where you can specify:

* Project name
* Organization (required)
* Git URL (HTTPS, .git) (required)
* Description