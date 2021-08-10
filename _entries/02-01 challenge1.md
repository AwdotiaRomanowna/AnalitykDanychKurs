---
sectionid: deploy
sectionclass: h2
title: Deploy Azure Database for PostgreSQL with Azure Portal
parent-id: azure
---

Azure Database for PostgreSQL is a fully-managed database as a service with built-in capabilities, such as high availability and intelligence. 

### Tasks

* Use [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview) to deploy Azure Database for PostgreSQL - single server.
* Use [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) to connected to the created database instance.

#### Setup Database

**Deploy DB using ARM**

Deploy Azure Database for PostgreSQL, which is managed service that you can use to run, manage, and scale in the cloud.

{% collapsible %}

* If your environment meets the prerequisites and you're familiar with using ARM templates, [follow this link and a template will open in the Azure portal](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.dbforpostgresql%2Fmanaged-postgresql-with-vnet%2Fazuredeploy.json).

![Create Azure DB](media/create-azure-db-pg.png)

* Continue to the next page by clicking **Review + create**

* The review page should look like the following:

![Review Azure DB](media/review-pg-create.png)

{% endcollapsible %}

