# Exercise 5 - How to use Collections on Red Hat Ansible Controller

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
    - [Step 1 - Create a Credential for Private Automation Hub](#step-1---write-a-requirementsyml)
    - [Step 2 - Define Collection lookup on Organization](#step-2---create-job-template)
- [Troubleshooting](#troubleshooting)

# Objective

Red Hat Ansible Controller will automatically install and configure collections for you from the project. To make sure Ansible Collections are recognized by Red Hat Ansible Controller a requirements file is needed and has to be stored in the proper directory.

Ansible Galaxy is already configured by default, however if you want your Red Hat Ansible Controller to prefer and fetch content from the Red Hat Automation Hub, additional configuration changes are required.

# Guide

In this exercise you will learn how to define an Ansible Collection as a requirement in a format recognized by Red Hat Ansible Controller and how to configure the Controller to use the Private Automation Hub to pull from.

## Step 1 - Write a requirements.yml

Red Hat Ansible Controller can download and install Ansible Collections automatically before executing a Job Template. If a `collections/requirements.yml` exists, it will be parsed and Ansible Collections specified in this file will be automatically installed.

> **NOTE**: Since the Ansible Collection is downloaded into this directory before the Job Template is executed, you will find temporary files of your Ansible Collection in `/var/lib/awx/projects/`.

The format of the `requirements.yml` for Ansible Collections is very similar to the one for roles, however it is very important to store in the folder `collections`.

Here is what our Ansible Collection requirements looks like that has been pre created for us to use with AAP:

```yaml
---
collections:
- ansible.posix
- workshop.demo_collection
```

## Step 2 - Create a credential for the Private Automation Hub

We need to create a credential that will have the url of the repo and the api token needed to authenticate.

1. Log into your Controller node this can be found on your workshop portal, with a link to the url the username is admin and password is provided on the portal.

   ![Load token|845x550,20%](screenshots/controller_details.png)

Once logged in, navigate to Credentials

   ![Load token|845x550,20%](screenshots/credentials1.png)

Next Add a new credential

   ![Load token|845x550,20%](screenshots/credentials2.png)

Set the credential as follows

   ![Load token|845x550,20%](screenshots/create_credential.png)


Name: Private Automation Hub - Published
Organization: Default
Credential Type: Ansible Galaxy/Automation Hub API Token

Details we get from the private automation hub
Galaxy Server URL: is the link from the Published Repo

   ![Load token|845x550,20%](screenshots/automation_hub_repo.png)

Token: is the API token we got earlier and populated it into your .ansible.cfg if you didnt create one and regenerate it from the UI note you will need to repopulate the ansible.cfg as the user can only have 1 active API token.


## Step 3 - Configure the Organization to pull from the Private Automation Hub
The Credential we created behaves the same as what we used in the CLI, we create one for each repo we would pull from, and therefore we need to set the order in which we call them, in the Controller this is set at the Organization level.

Navigate to Organizations and select Default

   ![Load token|845x550,20%](screenshots/select_default_org.png)

Select edit

   ![Load token|845x550,20%](screenshots/edit_default_org.png)

Click on the magnifying glass next to Galaxy Credentials.
Put a tick into the Private automation hub credential you created and then you will see it populate at the top, then you want to drag it above Ansible Galaxy so the order in which it looks up is correct.

   ![Load token|845x550,20%](screenshots/order_default_org.png)

Press Select and Save.

## Step 3 - Create a Project 

Since Red Hat Ansible Tower does only check for updates in the repository in which you stored your Playbook, it might not do a refresh if there was a change in the Ansible Collection used by your Playbook. This happens particularly if you also combine Roles and Collections.

In this case you should check the option **Delete on Update** which will delete the entire local directory during a refresh.

If there is a problem while parsing your `requirements.yml` it worth testing it with the `ansible-galaxy` command. As a reminder, Red Hat Ansible Controller basically also just runs the command for you with the appropriate parameters, so testing this works manually makes a lot of sense.



----
**Navigation**
<br>
[Previous Exercise](../4-collections-from-roles/)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md)
