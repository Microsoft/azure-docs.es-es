---
title: Creación de una copia de seguridad de aplicaciones del clúster de Red Hat OpenShift 4 en Azure
description: Aprenda a crear una copia de seguridad de las aplicaciones del clúster de Red Hat OpenShift en Azure con Velero
ms.service: container-service
ms.topic: article
ms.date: 06/22/2020
author: troy0820
ms.author: b-trconn
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc
ms.openlocfilehash: 046cd30c0f93a468287c73573a3d18f4ba66221b
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690228"
---
# <a name="create-an-azure-red-hat-openshift-4-cluster-application-backup"></a>Creación de una copia de seguridad de aplicaciones del clúster de Red Hat OpenShift en Azure 4

En este artículo, prepará el entorno para crear una copia de seguridad de aplicaciones del clúster de Red Hat OpenShift en Azure 4. Aprenderá a:

> [!div class="checklist"]
> * Configurar los requisitos previos e instalar las herramientas necesarias
> * Creación de una copia de seguridad de aplicaciones de Red Hat OpenShift 4 en Azure

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.6.0, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="before-you-begin"></a>Antes de empezar

### <a name="install-velero"></a>Instalación de Velero

Para [instalar](https://velero.io/docs/main/basic-install/) Velero en el sistema, siga el proceso recomendado para su sistema operativo.

### <a name="set-up-azure-storage-account-and-blob-container"></a>Configuración de un contenedor de blobs y una cuenta de almacenamiento de Azure

En este paso se creará un grupo de recursos fuera del grupo de recursos del clúster de ARO.  Este grupo de recursos permitirá que las copias de seguridad se conserven y se puedan restaurar las aplicaciones a los clústeres nuevos.

```bash
AZURE_BACKUP_RESOURCE_GROUP=Velero_Backups
az group create -n $AZURE_BACKUP_RESOURCE_GROUP --location eastus

AZURE_STORAGE_ACCOUNT_ID="velero$(uuidgen | cut -d '-' -f5 | tr '[A-Z]' '[a-z]')"
az storage account create \
    --name $AZURE_STORAGE_ACCOUNT_ID \
    --resource-group $AZURE_BACKUP_RESOURCE_GROUP \
    --sku Standard_GRS \
    --encryption-services blob \
    --https-only true \
    --kind BlobStorage \
    --access-tier Hot

BLOB_CONTAINER=velero
az storage container create -n $BLOB_CONTAINER --public-access off --account-name $AZURE_STORAGE_ACCOUNT_ID
```

## <a name="set-permissions-for-velero"></a>Establecimiento de permisos de Velero

### <a name="create-service-principal"></a>Creación de una entidad de servicio

Velero necesita permisos para realizar copias de seguridad y restauraciones. Al crear una entidad de servicio, se le está asignando el permiso de Velero para acceder al grupo de recursos que definió en el paso anterior. Este paso obtendrá el grupo de recursos del clúster:

```bash
export AZURE_RESOURCE_GROUP=aro-$(az aro show --name <name of cluster> --resource-group <name of resource group> | jq -r '.clusterProfile.domain')
```


```bash
AZURE_SUBSCRIPTION_ID=$(az account list --query '[?isDefault].id' -o tsv)

AZURE_TENANT_ID=$(az account list --query '[?isDefault].tenantId' -o tsv)
```

```bash
AZURE_CLIENT_SECRET=$(az ad sp create-for-rbac --name "velero" --role "Contributor" --query 'password' -o tsv \
--scopes  /subscriptions/$AZURE_SUBSCRIPTION_ID)
AZURE_CLIENT_ID=$(az ad sp list --display-name "velero" --query '[0].appId' -o tsv)

```

```bash
cat << EOF  > ./credentials-velero.yaml
AZURE_SUBSCRIPTION_ID=${AZURE_SUBSCRIPTION_ID}
AZURE_TENANT_ID=${AZURE_TENANT_ID}
AZURE_CLIENT_ID=${AZURE_CLIENT_ID}
AZURE_CLIENT_SECRET=${AZURE_CLIENT_SECRET}
AZURE_RESOURCE_GROUP=${AZURE_RESOURCE_GROUP}
AZURE_CLOUD_NAME=AzurePublicCloud
EOF
```

## <a name="install-velero-on-azure-red-hat-openshift-4-cluster"></a>Instalación de un clúster de Red Hat OpenShift 4 en Azure

En este paso se instalará Velero en su propio proyecto y las [definiciones de recursos personalizados](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) necesarios para realizar copias de seguridad y restauraciones con Velero. Asegúrese de que ha iniciado sesión correctamente en un clúster de Red Hat OpenShift 4 en Azure.


```bash
velero install \
--provider azure \
--plugins velero/velero-plugin-for-microsoft-azure:v1.1.0 \
--bucket $BLOB_CONTAINER \
--secret-file ~/path/to/credentials-velero.yaml \
--backup-location-config resourceGroup=$AZURE_BACKUP_RESOURCE_GROUP,storageAccount=$AZURE_STORAGE_ACCOUNT_ID \
--snapshot-location-config apiTimeout=15m \
--velero-pod-cpu-limit="0" --velero-pod-mem-limit="0" \
--velero-pod-mem-request="0" --velero-pod-cpu-request="0"
```

## <a name="create-a-backup-with-velero"></a>Crear una copia de seguridad con Velero

Para crear una copia de seguridad de aplicaciones con Velero, debe incluir el espacio de nombres en el que se encuentra esta aplicación.  Si tiene un espacio de nombres `nginx-example` y desea incluir todos los recursos de ese espacio de nombres en la copia de seguridad, ejecute el siguiente comando en el terminal:

```bash
velero create backup <name of backup> --include-namespaces=nginx-example
```
Puede comprobar el estado de la copia de seguridad ejecutando:

```bash
oc get backups -n velero <name of backup> -o yaml
```

Una copia de seguridad correcta generará `phase:Completed` y los objetos residirán en el contenedor de la cuenta de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha realizado una copia de seguridad de la aplicación del clúster de Red Hat OpenShift 4en Azure. Ha aprendido a:

> [!div class="checklist"]
> * Crear una copia de seguridad de aplicaciones del clúster de OpenShift 4 con Velero


Vaya al siguiente artículo para aprender a crear una restauración de aplicaciones del clúster de Red Hat OpenShift 4 en Azure.

* [Creación de una restauración de aplicaciones del clúster de Red Hat OpenShift 4 en Azure](howto-create-a-restore.md)
