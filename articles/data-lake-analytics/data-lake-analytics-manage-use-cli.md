---
title: Administración de Azure Data Lake Analytics mediante la CLI de Azure
description: En este artículo se describe cómo usar la CLI de Azure para administrar trabajos, orígenes de datos y usuarios de Data Lake Analytics.
services: data-lake-analytics
ms.assetid: 4e5a3a0a-6d7f-43ed-aeb5-c3b3979a1e0a
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 01/29/2018
ms.openlocfilehash: d4b9a7be36bd4c0a4044f3e76b96b21a16eb80de
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132524"
---
# <a name="manage-azure-data-lake-analytics-using-the-azure-command-line-interface-cli"></a>Administración de Azure Data Lake Analytics mediante la Interfaz de la línea de comandos (CLI) de Azure

[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Aprenda a administrar cuentas, orígenes de datos, usuarios y trabajos de Azure Data Lake Analytics con la CLI de Azure. Para ver los temas de administración con otras herramientas, haga clic en el selector de pestañas de arriba.


## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial, debe contar con los recursos siguientes:

* Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* CLI de Azure. Consulte [Instalación y configuración de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

  * Descargue e instale la **versión preliminar** de las [herramientas de la CLI de Azure](https://github.com/MicrosoftBigData/AzureDataLake/releases) para completar esta demostración.

* Para realizar la autenticación, use el comando `az login` y seleccione la suscripción que desea usar. Para más información acerca de cómo autenticarse con una cuenta profesional o educativa, consulte [Conexión a una suscripción de Azure desde la CLI de Azure](/cli/azure/authenticate-azure-cli).

   ```azurecli
   az login
   az account set --subscription <subscription id>
   ```

   Ahora puede acceder a los comandos de Data Lake Analytics y Data Lake Store. Ejecute el comando siguiente para mostrar los comandos de Data Lake Store y Data Lake Analytics:

   ```azurecli
   az dls -h
   az dla -h
   ```

## <a name="manage-accounts"></a>Administrar cuentas

Antes de ejecutar un trabajo de Análisis de Data Lake, debe tener una cuenta de Análisis de Data Lake. A diferencia de HDInsight de Azure, no se paga por una cuenta de Análisis cuando no está ejecutando un trabajo. Solo se paga por el tiempo en que se ejecuta un trabajo.  Para obtener más información, consulte la página con [información general sobre Análisis con Azure Data Lake](data-lake-analytics-overview.md).  

### <a name="create-accounts"></a>Creación de cuentas

Ejecute el comando siguiente para crear una cuenta de Data Lake. 

   ```azurecli
   az dla account create --account "<Data Lake Analytics account name>" --location "<Location Name>" --resource-group "<Resource Group Name>" --default-data-lake-store "<Data Lake Store account name>"
   ```

### <a name="update-accounts"></a>Actualización de cuentas

El siguiente comando actualiza las propiedades de una cuenta existente de Análisis de Data Lake

   ```azurecli
   az dla account update --account "<Data Lake Analytics Account Name>" --firewall-state "Enabled" --query-store-retention 7
   ```

### <a name="list-accounts"></a>Enumeración de cuentas

Enumerar cuentas de Análisis de Data Lake dentro de un grupo de recursos específico

   ```azurecli
   az dla account list "<Resource group name>"
   ```

## <a name="get-details-of-an-account"></a>Obtención de los detalles de una cuenta

   ```azurecli
   az dla account show --account "<Data Lake Analytics account name>" --resource-group "<Resource group name>"
   ```

### <a name="delete-an-account"></a>Eliminación de una cuenta

   ```azurecli
   az dla account delete --account "<Data Lake Analytics account name>" --resource-group "<Resource group name>"
   ```

## <a name="manage-data-sources"></a>Administración de orígenes de datos

Actualmente, Data Lake Analytics admite estos dos orígenes de datos:

* [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)
* [Almacenamiento de Azure](../storage/common/storage-introduction.md)

Cuando crea una cuenta de Análisis, debe designar una cuenta de Almacén de Azure Data Lake para que sea la cuenta de almacenamiento predeterminada. La cuenta de almacenamiento predeterminada de Data Lake sirve para almacenar los registros de auditoría y de metadatos de trabajos. Una vez creada la cuenta de Análisis, puede agregar más cuentas de Data Lake Store y/o cuentas de Azure Storage. 

### <a name="find-the-default-data-lake-store-account"></a>Búsqueda de la cuenta predeterminada de Almacén de Data Lake

Ejecute el comando `az dla account show` para ver la cuenta predeterminada de Data Lake Store. El nombre de la cuenta predeterminada aparece bajo la propiedad defaultDataLakeStoreAccount.

   ```azurecli
   az dla account show --account "<Data Lake Analytics account name>"
   ```

### <a name="add-additional-blob-storage-accounts"></a>Incorporación de cuentas de Blob Storage adicionales

   ```azurecli
   az dla account blob-storage add --access-key "<Azure Storage Account Key>" --account "<Data Lake Analytics account name>" --storage-account-name "<Storage account name>"
   ```

> [!NOTE]
> Solo se admiten nombres cortos de almacenamiento de blobs. No use el FQDN, por ejemplo "myblob.blob.core.windows.net".
>

### <a name="add-additional-data-lake-store-accounts"></a>Adición de más cuentas de Almacén de Data Lake

El comando siguiente actualiza la cuenta de Data Lake Analytics especificada con una cuenta de Data Lake Store adicional:

   ```azurecli
   az dla account data-lake-store add --account "<Data Lake Analytics account name>" --data-lake-store-account-name "<Data Lake Store account name>"
   ```

### <a name="update-existing-data-source"></a>Actualizar el origen de datos existente

Para actualizar una clave de cuenta de almacenamiento de blobs existente:

   ```azurecli
   az dla account blob-storage update --access-key "<New Blob Storage Account Key>" --account "<Data Lake Analytics account name>" --storage-account-name "<Data Lake Store account name>"
   ```

### <a name="list-data-sources"></a>Enumeración de orígenes de datos:

Para mostrar las cuentas de Data Lake Store:

   ```azurecli
   az dla account data-lake-store list --account "<Data Lake Analytics account name>"
   ```

Para mostrar la cuenta de Blob Storage:

   ```azurecli
   az dla account blob-storage list --account "<Data Lake Analytics account name>"
   ```

![Origen de datos de lista de Análisis de Data Lake](./media/data-lake-analytics-manage-use-cli/data-lake-analytics-list-data-source.png)

### <a name="delete-data-sources"></a>Eliminar orígenes de datos:

Para eliminar una cuenta de Almacén Data Lake:

   ```azurecli
   az dla account data-lake-store delete --account "<Data Lake Analytics account name>" --data-lake-store-account-name "<Azure Data Lake Store account name>"
   ```

Para eliminar una cuenta de Almacenamiento de blobs:

   ```azurecli
   az dla account blob-storage delete --account "<Data Lake Analytics account name>" --storage-account-name "<Data Lake Store account name>"
   ```

## <a name="manage-jobs"></a>Trabajos de administración

Debe tener una cuenta de Análisis de Data Lake para poder crear un trabajo.  Para obtener más información, consulte [Administración de cuentas de Análisis de Data Lake](#manage-accounts).

### <a name="list-jobs"></a>Enumeración de trabajos

   ```azurecli
   az dla job list --account "<Data Lake Analytics account name>"
   ```

   ![Origen de datos de lista de Análisis de Data Lake](./media/data-lake-analytics-manage-use-cli/data-lake-analytics-list-jobs.png)

### <a name="get-job-details"></a>Obtención de detalles del trabajo

   ```azurecli
   az dla job show --account "<Data Lake Analytics account name>" --job-identity "<Job Id>"
   ```

### <a name="submit-jobs"></a>Envío de trabajos

> [!NOTE]
> La prioridad predeterminada de un trabajo es 1000 y el grado predeterminado de paralelismo para un trabajo es 1.
>
>    ```azurecli
>    az dla job submit --account "<Data Lake Analytics account name>" --job-name "<Name of your job>" --script "<Script to submit>"
>    ```

### <a name="cancel-jobs"></a>Cancelación de trabajos
Use el comando de la lista para buscar el identificador del trabajo y, a continuación, use Cancelar para cancelar el trabajo.

   ```azurecli
   az dla job cancel --account "<Data Lake Analytics account name>" --job-identity "<Job Id>"
   ```

## <a name="pipelines-and-recurrences"></a>Canalizaciones y repeticiones

### <a name="get-information-about-pipelines-and-recurrences"></a>Obtención de información sobre canalizaciones y repeticiones

Utilice los comandos `az dla job pipeline` para ver la información de canalización de trabajos enviados previamente.

```azurecli
az dla job pipeline list --account "<Data Lake Analytics Account Name>"

az dla job pipeline show --account "<Data Lake Analytics Account Name>" --pipeline-identity "<Pipeline ID>"
```

Utilice los comandos `az dla job recurrence` para ver la información de repetición de trabajos enviados previamente.

```azurecli
az dla job recurrence list --account "<Data Lake Analytics Account Name>"

az dla job recurrence show --account "<Data Lake Analytics Account Name>" --recurrence-identity "<Recurrence ID>"
```

## <a name="next-steps"></a>Pasos siguientes
* [Información general de Análisis de Microsoft Azure Data Lake](data-lake-analytics-overview.md)
* [Introducción a Data Lake Analytics mediante el portal de Azure](data-lake-analytics-get-started-portal.md)
* [Administración de Azure Data Lake Analytics con Azure Portal](data-lake-analytics-manage-use-portal.md)
* [Supervisión y solución de problemas de trabajos de Azure Data Lake Analytics con Azure Portal](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
