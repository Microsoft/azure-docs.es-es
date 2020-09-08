---
title: 'Inicio rápido: Creación de un área de trabajo de Synapse mediante la CLI de Azure'
description: Siga los pasos de esta guía para crear un área de trabajo de Azure Synapse mediante la CLI de Azure.
services: synapse-analytics
author: alehall
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: ''
ms.date: 08/25/2020
ms.author: alehall
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 7f0bf7d409c5e47abaaa5b59271f55952d8ccff4
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871604"
---
# <a name="quickstart-create-an-azure-synapse-workspace-with-azure-cli"></a>Inicio rápido: Creación de un área de trabajo de Azure Synapse con la CLI de Azure

La CLI de Azure es la forma de usar la línea de comandos de Azure para administrar los recursos de Azure. Puede utilizarlo en el explorador con Azure Cloud Shell. También puede instalarla en macOS, Linux o Windows y ejecutarla desde la línea de comandos.

En este inicio rápido aprenderá a crear un área de trabajo de Synapse mediante la CLI de Azure.

Si no tiene una suscripción a Azure, [cree una cuenta gratuita antes de empezar](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Requisitos previos

- Descargar e instalar [jq](https://stedolan.github.io/jq/download/), un procesador JSON de línea de comandos ligero y flexible.
- Una [cuenta de almacenamiento de Azure Data Lake Storage Gen2](../storage/common/storage-account-create.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

    > [!IMPORTANT]
    > El área de trabajo de Azure Synapse debe poder leer y escribir en la cuenta de ADLS Gen2 seleccionada. Además, para cualquier cuenta de almacenamiento que vincule como cuenta de almacenamiento principal, debe haber habilitado el **espacio de nombres jerárquico** al crearla, como se describe en la sección [Creación de una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-portal#create-a-storage-account). 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="install-the-azure-cli-locally"></a>Instalación local de la CLI de Azure

Si elige instalar y usar la CLI de Azure localmente, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Si ejecuta la CLI de Azure localmente, tiene que iniciar sesión y autenticarse. Este paso no es necesario si usa Azure Cloud Shell. Para iniciar sesión en la CLI de Azure, ejecute `az login` y autentíquese en la ventana del explorador:

```azurecli
az login
```

Para más información sobre la autenticación con la CLI de Azure, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).

## <a name="install-azure-synapse-extension-for-azure-cli"></a>Instalación de la extensión de Azure Synapse para la CLI de Azure

```azurecli
az extension add --name synapse
```

> [!WARNING]
> La extensión de Azure Synapse para la CLI de Azure está en versión preliminar.

## <a name="create-an-azure-synapse-workspace-using-the-azure-cli"></a>Creación de un área de trabajo de Azure Synapse mediante la CLI de Azure

1. Defina las variables de entorno necesarias para crear recursos para el área de trabajo de Azure Synapse.

    | Nombre de la variable de entorno | Descripción |
    |---|---|---|
    |StorageAccountName| Asigne un nombre a la cuenta de almacenamiento de ADLS Gen2 existente.|
    |StorageAccountResourceGroup| Nombre del grupo de recursos de la cuenta de almacenamiento de ADLS Gen2 existente. |
    |FileShareName| Nombre del sistema de archivos de almacenamiento existente.|
    |SynapseResourceGroup| Elija un nuevo nombre para el grupo de recursos de Azure Synapse. |
    |Región| Elija una de las [regiones de Azure](https://azure.microsoft.com/global-infrastructure/geographies/#overview). |
    |SynapseWorkspaceName| Elija un nombre único para la nueva área de trabajo de Azure Synapse. |
    |SqlUser| Elija un valor para un nombre de usuario nuevo.|
    |SqlPassword| Elija una contraseña segura.|
    |||

2. Cree un grupo de recursos como contenedor para el área de trabajo de Azure Synapse:
    ```azurecli
    az group create --name $SynapseResourceGroup --location $Region
    ```
3. Recupere la clave de la cuenta de almacenamiento de ADLS Gen 2:
    ```azurecli
    StorageAccountKey=$(az storage account keys list \
      --account-name $StorageAccountName \
      | jq -r '.[0] | .value')
    ```
4. Recupere la dirección URL del punto de conexión de ADLS Gen 2:
    ```azurecli
    StorageEndpointUrl=$(az storage account show \
      --name $StorageAccountName \
      --resource-group $StorageAccountResourceGroup \
      | jq -r '.primaryEndpoints | .dfs')
    ```

5. (Opcional) Siempre puede comprobar cuáles son el punto de conexión y la clave de la cuenta de almacenamiento de ADLS Gen2:
    ```azurecli
    echo "Storage Account Key: $StorageAccountKey"
    echo "Storage Endpoint URL: $StorageEndpointUrl"
    ```

6. Cree un área de trabajo de Azure Synapse:
    ```azurecli
    az synapse workspace create \
      --name $SynapseWorkspaceName \
      --resource-group $SynapseResourceGroup \
      --storage-account $StorageAccountName \
      --file-system $FileShareName \
      --sql-admin-login-user $SqlUser \
      --sql-admin-login-password $SqlPassword \
      --location $Region
    ```

7. Obtenga la dirección URL web y de desarrollo del área de trabajo de Azure Synapse:
    ```azurecli
    WorkspaceWeb=$(az synapse workspace show --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup | jq -r '.connectivityEndpoints | .web')

    WorkspaceDev=$(az synapse workspace show --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup | jq -r '.connectivityEndpoints | .dev')
    ```

8. Cree una regla de firewall que le permita acceder al área de trabajo de Azure Synapse desde su máquina:

    ```azurecli
    ClientIP=$(curl -sb -H "Accept: application/json" "$WorkspaceDev" | jq -r '.message')
    ClientIP=${ClientIP##'Client Ip address : '}
    echo "Creating a firewall rule to enable access for IP address: $ClientIP"

    az synapse workspace firewall-rule create --end-ip-address $ClientIP --start-ip-address $ClientIP --name "Allow Client IP" --resource-group $SynapseResourceGroup --workspace-name $SynapseWorkspaceName
    ```

9. Abra la dirección URL web del área de trabajo de Azure Synapse almacenada en la variable de entorno `WorkspaceWeb` para acceder al área de trabajo:

    ```azurecli
    echo "Open your Azure Synapse Workspace Web URL in the browser: $WorkspaceWeb"
    ```
    
    [ ![Web del área de trabajo de Azure Synapse](media/quickstart-create-synapse-workspace-cli/create-workspace-cli-1.png) ](media/quickstart-create-synapse-workspace-cli/create-workspace-cli-1.png#lightbox)


## <a name="clean-up-resources"></a>Limpieza de recursos

Siga los pasos que se indican a continuación para eliminar el área de trabajo de Azure Synapse.
> [!WARNING]
> Al eliminar un área de trabajo de Azure Synapse, se quitarán también los motores de análisis y los datos almacenados en la base de datos de los grupos de SQL y metadatos de área de trabajo incluidos. Ya no será posible conectarse a los puntos de conexión de SQL o de Apache Spark. Se eliminarán todos los artefactos de código (consultas, cuadernos, definiciones de trabajos y canalizaciones).
>
> La eliminación del área de trabajo **no** afectará a los datos de Data Lake Store Gen2 vinculados al área de trabajo.

Si desea eliminar el área de trabajo de Azure Synapse, siga estos pasos:

```azurecli
az synapse workspace delete --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede [crear grupos de SQL](quickstart-create-sql-pool-studio.md) o [crear grupos de Apache Spark](quickstart-create-apache-spark-pool-studio.md) para empezar a analizar y explorar los datos.
