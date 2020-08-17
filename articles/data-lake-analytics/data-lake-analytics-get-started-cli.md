---
title: 'Creación y consulta de Azure Data Lake Analytics: CLI de Azure'
description: Aprenda cómo utilizar la interfaz de la línea de comandos de Azure para crear una cuenta de Azure Data Lake Analytics y enviar un trabajo de U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 06/18/2017
ms.custom: devx-track-azurecli
ms.openlocfilehash: be7f9d66a10f3f0f4a1ae0f9e28e226f268a7f7c
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497450"
---
# <a name="get-started-with-azure-data-lake-analytics-using-azure-cli"></a>Introducción al uso de la CLI de Azure por parte de Azure Data Lake Analytics

[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

En este artículo se describe cómo utilizar la interfaz de la línea de comandos de la CLI de Azure para crear cuentas de Azure Data Lake Analytics, enviar trabajos de U-SQL y catálogos. El trabajo lee un archivo de valores separados por tabulaciones (TSV) y lo convierte en un otro de valores separados por comas (CSV).

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, necesita los siguientes elementos:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* En este artículo se requiere que ejecute la versión de la CLI de Azure 2.0 o una versión posterior. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para iniciar sesión en la suscripción de Azure:

```azurecli
az login
```

Se le solicita que vaya a una dirección URL y que escriba un código de autenticación.  Y, después, debe seguir las instrucciones para escribir sus credenciales.

Una vez que haya iniciado sesión, el comando de inicio de sesión enumera las suscripciones.

Para usar una suscripción concreta:

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-data-lake-analytics-account"></a>Creación de una cuenta de Análisis de Data Lake

Para poder ejecutar cualquier trabajo es preciso tener una cuenta de Data Lake Analytics. Para crearla, debe especificar los siguientes elementos:

* **Grupo de recursos de Azure**. Se debe crear una cuenta de Data Lake Analytics en un grupo de recursos de Azure. [Azure Resource Manager](../azure-resource-manager/management/overview.md) permite trabajar con los recursos de la aplicación como un grupo. Puede implementar, actualizar o eliminar todos los recursos de la aplicación en una operación única coordinada.  

Para enumerar los grupos de recursos que contiene su suscripción:

```azurecli
az group list
```

Para crear un nuevo grupo de recursos:

```azurecli
az group create --name "<Resource Group Name>" --location "<Azure Location>"
```

* **Nombre de la cuenta de Data Lake Analytics**. Cada cuenta de Data Lake Analytics tiene un nombre.
* **Ubicación**. Use uno de los centros de datos de Azure que admita Data Lake Analytics.
* **Cuenta predeterminada de Data Lake Store**: cada cuenta de Data Lake Analytics tiene una cuenta de Data Lake Store predeterminada.

Para mostrar la cuenta de Data Lake Store existente:

```azurecli
az dls account list
```

Para crear una nueva cuenta de Data Lake Store:

```azurecli
az dls account create --account "<Data Lake Store Account Name>" --resource-group "<Resource Group Name>"
```

Use la siguiente sintaxis para crear una cuenta de Data Lake Analytics:

```azurecli
az dla account create --account "<Data Lake Analytics Account Name>" --resource-group "<Resource Group Name>" --location "<Azure location>" --default-data-lake-store "<Default Data Lake Store Account Name>"
```

Después de crear una cuenta, puede usar los comandos siguientes para enumerar las cuentas y mostrar los detalles de la misma:

```azurecli
az dla account list
az dla account show --account "<Data Lake Analytics Account Name>"
```

## <a name="upload-data-to-data-lake-store"></a>Carga de datos en el Almacén Data Lake

En este tutorial, va a procesar algunos registros de búsqueda.  El registro de búsqueda se puede almacenar en el Almacén de Data Lake o en el almacenamiento de blobs de Azure.

Azure Portal proporciona una interfaz de usuario para copiar algunos archivos de datos de ejemplo a la cuenta predeterminada de Data Lake Store, entre los que se incluye un archivo de registro de búsqueda. Consulte [Preparar los datos de origen](data-lake-analytics-get-started-portal.md) para cargar los datos en la cuenta del Almacén Data Lake.

Para cargar archivos con la CLI de Azure, use el siguiente comando:

```azurecli
az dls fs upload --account "<Data Lake Store Account Name>" --source-path "<Source File Path>" --destination-path "<Destination File Path>"
az dls fs list --account "<Data Lake Store Account Name>" --path "<Path>"
```

Análisis de Data Lake también puede acceder al almacenamiento de blobs de Azure.  Para cargar datos a Azure Blob Storage, consulte [Uso de la CLI de Azure con Azure Storage](../storage/common/storage-azure-cli.md).

## <a name="submit-data-lake-analytics-jobs"></a>Envío de trabajos de Análisis de Data Lake

Los trabajos de Análisis de Data Lake se escriben en el lenguaje U-SQL. Para más información sobre U-SQL, consulte la [introducción al lenguaje U-SQL](data-lake-analytics-u-sql-get-started.md) y la [referencia del lenguaje U-SQL](https://docs.microsoft.com/u-sql/).

### <a name="to-create-a-data-lake-analytics-job-script"></a>Para crear un script de trabajo de Data Lake Analytics

Cree un archivo de texto con el siguiente script U-SQL y guarde el archivo de texto en la estación de trabajo:

```usql
@a  =
    SELECT * FROM
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
```

Este script de U-SQL lee el archivo de datos de origen mediante **Extractors.Tsv()** y crea un archivo csv con **Outputters.Csv()** .

No modifique ninguna de las dos rutas a menos que copie el archivo de origen en una ubicación diferente.  Data Lake Analytics creará la carpeta de salida si no existe.

Es más fácil usar rutas de acceso relativas para los archivos almacenados en las cuentas predeterminadas de Data Lake Store. También puede usar rutas de acceso absolutas.  Por ejemplo:

```usql
adl://<Data LakeStorageAccountName>.azuredatalakestore.net:443/Samples/Data/SearchLog.tsv
```

Debe usar rutas de acceso absolutas para acceder a los archivos de cuentas de almacenamiento vinculadas.  La sintaxis de los archivos almacenados en la cuenta de Azure Storage vinculada es:

```usql
wasb://<BlobContainerName>@<StorageAccountName>.blob.core.windows.net/Samples/Data/SearchLog.tsv
```

> [!NOTE]
> El contenedor de blobs de Azure con blobs públicos no se admite.
> El contenedor de blobs de Azure con contenedores públicos no se admite.
>

### <a name="to-submit-jobs"></a>Para enviar trabajos

Para enviar un trabajo, use la sintaxis siguiente.

```azurecli
az dla job submit --account "<Data Lake Analytics Account Name>" --job-name "<Job Name>" --script "<Script Path and Name>"
```

Por ejemplo:

```azurecli
az dla job submit --account "myadlaaccount" --job-name "myadlajob" --script @"C:\DLA\myscript.txt"
```

### <a name="to-list-jobs-and-show-job-details"></a>Para enumerar los trabajos y mostrar los detalles de un trabajo

```azurecli
az dla job list --account "<Data Lake Analytics Account Name>"
az dla job show --account "<Data Lake Analytics Account Name>" --job-identity "<Job Id>"
```

### <a name="to-cancel-jobs"></a>Para cancelar trabajos

```azurecli
az dla job cancel --account "<Data Lake Analytics Account Name>" --job-identity "<Job Id>"
```

## <a name="retrieve-job-results"></a>Recuperación de los resultados de un trabajo

Una vez que se completa un trabajo, puede usar los siguientes comandos para enumerar los archivos de salida y descargar los archivos:

```azurecli
az dls fs list --account "<Data Lake Store Account Name>" --source-path "/Output" --destination-path "<Destination>"
az dls fs preview --account "<Data Lake Store Account Name>" --path "/Output/SearchLog-from-Data-Lake.csv"
az dls fs preview --account "<Data Lake Store Account Name>" --path "/Output/SearchLog-from-Data-Lake.csv" --length 128 --offset 0
az dls fs download --account "<Data Lake Store Account Name>" --source-path "/Output/SearchLog-from-Data-Lake.csv" --destination-path "<Destination Path and File Name>"
```

Por ejemplo:

```azurecli
az dls fs download --account "myadlsaccount" --source-path "/Output/SearchLog-from-Data-Lake.csv" --destination-path "C:\DLA\myfile.csv"
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver el documento de referencia de la CLI de Azure de Data Lake Analytics, consulte [Data Lake Analytics](/cli/azure/dla).
* Para ver el documento de referencia de la CLI de Azure de Data Lake Store, consulte [Data Lake Store](/cli/azure/dls).
* Para ver una consulta más compleja, consulte la página sobre el [análisis de registros de sitio web mediante Análisis de Azure Data Lake](data-lake-analytics-analyze-weblogs.md).
