---
title: Introducción a Azure Data Lake Storage Gen1 | PowerShell Microsoft Docs
description: Use Azure PowerShell para crear una cuenta de Azure Data Lake Storage Gen1 y realizar operaciones básicas.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 06/27/2018
ms.author: twooley
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ba51c96e3195f75a0ccf49baded67737f4665615
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89077745"
---
# <a name="get-started-with-azure-data-lake-storage-gen1-using-azure-powershell"></a>Introducción a Azure Data Lake Storage Gen1 mediante Azure PowerShell

> [!div class="op_single_selector"]
> * [Portal](data-lake-store-get-started-portal.md)
> * [PowerShell](data-lake-store-get-started-powershell.md)
> * [CLI de Azure](data-lake-store-get-started-cli-2.0.md)
>
>

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Aprenda a usar Azure PowerShell para crear una cuenta de Azure Data Lake Storage Gen1 y realizar operaciones básicas como crear carpetas, cargar y descargar archivos de datos, eliminar la cuenta, etc. Para más información sobre Data Lake Storage Gen1, consulte [Información general de Data Lake Storage Gen1](data-lake-store-overview.md).

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1.0 o versiones posteriores**. Consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).

## <a name="authentication"></a>Authentication

En este artículo se usa un enfoque de autenticación más sencillo con Data Lake Storage Gen1 en el que se le solicita que escriba las credenciales de la cuenta de Azure. El nivel de acceso a la cuenta de Data Lake Storage Gen1 y al sistema de archivos está determinado por el nivel de acceso del usuario que ha iniciado sesión. Aunque existen otros enfoques para realizar la autenticación con Data Lake Storage Gen1, como la autenticación de usuario final o la autenticación entre servicios. Para obtener instrucciones y más información acerca de cómo realizar la autenticación, consulte [Autenticación de usuario final con Data Lake Store mediante Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md) o [Autenticación entre servicios con Data Lake Store mediante Azure Active Directory](data-lake-store-authenticate-using-active-directory.md).

## <a name="create-a-data-lake-storage-gen1-account"></a>Creación de una cuenta de Data Lake Storage Gen1

1. Desde el escritorio, abra una nueva ventana de Windows PowerShell. Escriba el siguiente fragmento de código para iniciar sesión en su cuenta de Azure, establecer la suscripción y registrar el proveedor de Data Lake Storage Gen1. Cuando se le solicite iniciar sesión, asegúrese de iniciarla como uno de los administradores o propietarios de la suscripción:

    ```PowerShell
    # Log in to your Azure account
    Connect-AzAccount

    # List all the subscriptions associated to your account
    Get-AzSubscription

    # Select a subscription
    Set-AzContext -SubscriptionId <subscription ID>

    # Register for Azure Data Lake Storage Gen1
    Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
    ```

1. Una cuenta de Data Lake Storage Gen1 está asociada a un grupo de recursos de Azure. Comience a crear un grupo de recursos.

    ```PowerShell
    $resourceGroupName = "<your new resource group name>"
    New-AzResourceGroup -Name $resourceGroupName -Location "East US 2"
    ```

    ![Creación de un grupo de recursos de Azure](./media/data-lake-store-get-started-powershell/ADL.PS.CreateResourceGroup.png "Creación de un grupo de recursos de Azure")

1. Cree una cuenta de Data Lake Storage Gen1. El nombre que especifique debe contener solo letras minúsculas y números.

    ```PowerShell
    $dataLakeStorageGen1Name = "<your new Data Lake Storage Gen1 account name>"
    New-AzDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStorageGen1Name -Location "East US 2"
    ```

    ![Creación de una cuenta de Data Lake Storage Gen1](./media/data-lake-store-get-started-powershell/ADL.PS.CreateADLAcc.png "Creación de una cuenta de Data Lake Storage Gen1")

1. Compruebe que la cuenta se creó correctamente.

    ```PowerShell
    Test-AzDataLakeStoreAccount -Name $dataLakeStorageGen1Name
    ```

    La salida del cmdlet debe ser **True**.

## <a name="create-directory-structures"></a>Creación de estructuras de directorio

Puede crear directorios en su cuenta de Data Lake Storage Gen1 para administrar y almacenar datos.

1. Especifique un directorio raíz.

    ```PowerShell
    $myrootdir = "/"
    ```

1. Cree un nuevo directorio denominado **mynewdirectory** en la raíz especificada.

    ```PowerShell
    New-AzDataLakeStoreItem -Folder -AccountName $dataLakeStorageGen1Name -Path $myrootdir/mynewdirectory
    ```

1. Compruebe que el nuevo directorio se creó correctamente.

    ```PowerShell
    Get-AzDataLakeStoreChildItem -AccountName $dataLakeStorageGen1Name -Path $myrootdir
    ```

    La salida debe parecerse a la siguiente captura de pantalla:

    ![Comprobación del directorio](./media/data-lake-store-get-started-powershell/ADL.PS.Verify.Dir.Creation.png "Comprobación del directorio")

## <a name="upload-data"></a>Carga de datos

Puede cargar los datos en Data Lake Storage Gen1 directamente en el nivel raíz o en un directorio que haya creado en la cuenta. Los fragmentos de código de esta sección muestran cómo cargar datos de ejemplo en el directorio (**mynewdirectory**) que creó en la sección anterior.

Si busca datos de ejemplo para cargar, puede obtener la carpeta **Ambulance Data** en el [repositorio Git de Azure Data Lake](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData). Descargue el archivo y almacénelo en un directorio local del equipo, como C:\sampledata\.

```PowerShell
Import-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
   -Path "C:\sampledata\vehicle1_09142014.csv" `
   -Destination $myrootdir\mynewdirectory\vehicle1_09142014.csv
```

## <a name="rename-download-and-delete-data"></a>Cambio de nombre, descarga y eliminación de datos

Para cambiar el nombre de un archivo, use el comando siguiente:

```PowerShell
Move-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Path $myrootdir\mynewdirectory\vehicle1_09142014.csv `
    -Destination $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv
```

Para descargar un archivo, use el comando siguiente:

```PowerShell
Export-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Path $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv `
    -Destination "C:\sampledata\vehicle1_09142014_Copy.csv"
```

Para eliminar un archivo, use el comando siguiente:

```PowerShell
Remove-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Paths $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv
```

Cuando se le solicite, escriba **Y** para eliminar el elemento. Si tiene más de un archivo para eliminar, puede proporcionar todas las rutas de acceso separadas por comas.

```PowerShell
Remove-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Paths $myrootdir\mynewdirectory\vehicle1_09142014.csv, $myrootdir\mynewdirectoryvehicle1_09142014_Copy.csv
```

## <a name="delete-your-account"></a>Eliminación de la cuenta

Para eliminar la cuenta de Data Lake Storage Gen1, use el siguiente comando.

```PowerShell
Remove-AzDataLakeStoreAccount -Name $dataLakeStorageGen1Name
```

Cuando se le solicite, escriba **Y** para eliminar la cuenta.

## <a name="next-steps"></a>Pasos siguientes

* [Guía de ajuste del rendimiento para usar PowerShell con Azure Data Lake Storage Gen1](data-lake-store-performance-tuning-powershell.md)
* [Uso de Azure Data Lake Storage Gen1 para requisitos de macrodatos](data-lake-store-data-scenarios.md)
* [Protección de datos en Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Use Azure Data Lake Analytics with Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md) (Uso de Azure Data Lake Analytics con Data Lake Storage Gen1)
* [Use Azure HDInsight with Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md) (Uso de Azure HDInsight con Data Lake Storage Gen1)
