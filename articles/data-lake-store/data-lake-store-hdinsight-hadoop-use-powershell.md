---
title: 'PowerShell para HDInsight con Data Lake Storage Gen1: complemento de almacenamiento de Azure'
description: Aprenda a usar Azure PowerShell para configurar un clúster de HDInsight con Azure Data Lake Storage Gen1 como almacenamiento adicional.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: b1a228e9d15a349f032291fb0678bdd0ad9d95b3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87048855"
---
# <a name="use-azure-powershell-to-create-an-hdinsight-cluster-with-azure-data-lake-storage-gen1-as-additional-storage"></a>Uso de Azure PowerShell para crear clústeres de HDInsight con Azure Data Lake Storage Gen1 (como almacenamiento adicional)

> [!div class="op_single_selector"]
> * [Uso del portal](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Uso de PowerShell (para el almacenamiento predeterminado)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Uso de PowerShell (para el almacenamiento adicional)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Uso de Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
>
>

Aprenda a usar Azure PowerShell para configurar un clúster de HDInsight con Azure Data Lake Storage Gen1, **como almacenamiento adicional**. Para obtener instrucciones acerca de cómo crear un clúster de HDInsight con Data Lake Storage Gen1 como almacenamiento predeterminado, consulte [Creación de un clúster de HDInsight con Data Lake Storage Gen1 mediante Azure PowerShell](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md).

> [!NOTE]
> Si va a usar Data Lake Storage Gen1 como almacenamiento adicional para un clúster de HDInsight, se recomienda encarecidamente hacer esto al crear el clúster como se describe en este artículo. La adición de Data Lake Storage Gen1 como almacenamiento adicional a un clúster de HDInsight existente es un proceso complicado y en la que es fácil que se produzcan errores.
>

Para los tipos de clúster compatibles, Data Lake Storage Gen1 se puede usar como un almacenamiento predeterminado o una cuenta de almacenamiento adicional. Cuando Data Lake Storage Gen1 se usa como almacenamiento adicional, la cuenta de almacenamiento predeterminada para los clústeres seguirá siendo Azure Storage Blobs (WASB) y los archivos relacionados con clústeres (como registros, etc.) seguirán escribiéndose en el almacenamiento predeterminado, mientras que los datos que quiere procesar pueden almacenarse en una cuenta de Data Lake Storage Gen1. Utilizar Data Lake Storage Gen1 como una cuenta de almacenamiento adicional no afecta al rendimiento ni a la capacidad de lectura y escritura en el almacenamiento del clúster.

## <a name="using-data-lake-storage-gen1-for-hdinsight-cluster-storage"></a>Uso de Data Lake Storage Gen1 para el almacenamiento de clústeres de HDInsight

Estas son algunos puntos importantes que se deben tener en cuenta al usar HDInsight con Data Lake Storage Gen1:

* La opción para crear clústeres de HDInsight con acceso a Data Lake Storage Gen1 como almacenamiento está disponible para las versiones 3.2, 3.4, 3.5 y 3.6 de HDInsight.

La configuración de HDInsight para trabajar con Data Lake Storage Gen1 mediante PowerShell consta de los pasos siguientes:

* Creación de una cuenta de Data Lake Storage Gen1
* Configuración de la autenticación para el acceso basado en roles a Data Lake Storage Gen1
* Creación de un clúster de HDInsight con autenticación en Data Lake Storage Gen1
* Ejecución de un trabajo de prueba en el clúster

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Antes de empezar este tutorial, debe contar con lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1.0 o versiones posteriores**. Consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).
* **Windows SDK**. Se puede instalar desde [aquí](https://dev.windows.com/en-us/downloads). Úselo para crear un certificado de seguridad.
* **Entidad de servicio de Azure Active Directory** En los pasos de este tutorial se proporcionan instrucciones sobre cómo crear entidades de servicio de Azure AD. Sin embargo, debe ser administrador de Azure AD para poder crearlas. Si ya lo es, puede hacer caso omiso a este requisito previo y continuar con el tutorial.

    **Si no lo es**, no podrá realizar los pasos necesarios para crear una entidad de servicio. En este caso, su administrador de Azure AD debe generar primero una entidad de servicio antes de crear un clúster de HDInsight con Data Lake Storage Gen1. Además, la entidad de servicio debe crearse con un certificado, tal y como se describe en [Creación de una entidad de servicio](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-certificate-from-certificate-authority).

## <a name="create-a-data-lake-storage-gen1-account"></a>Creación de una cuenta de Data Lake Storage Gen1
Siga estos pasos para crear una cuenta de Data Lake Storage Gen1.

1. En el escritorio, abra una nueva ventana de Azure PowerShell y escriba el siguiente fragmento. Cuando se le solicite iniciar sesión, asegúrese de iniciarla como uno de los administradores o propietario de la suscripción:

    ```azurepowershell
    # Log in to your Azure account
    Connect-AzAccount

    # List all the subscriptions associated to your account
    Get-AzSubscription

    # Select a subscription
    Set-AzContext -SubscriptionId <subscription ID>

    # Register for Data Lake Storage Gen1
    Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
    ```

   > [!NOTE]
   > Si recibe un error similar a `Register-AzResourceProvider : InvalidResourceNamespace: The resource namespace 'Microsoft.DataLakeStore' is invalid` al registrar el proveedor de recursos de Data Lake Storage Gen1, es posible que su suscripción no esté en la lista de permitidas de Data Lake Storage Gen1. Asegúrese de habilitar su suscripción de Azure en la versión preliminar pública de Data Lake Storage Gen1 siguiendo estas [instrucciones](data-lake-store-get-started-portal.md).
   >
   >
2. Una cuenta de Data Lake Storage Gen1 está asociada con un grupo de recursos de Azure. Comience creando un grupo de recursos de Azure.

    ```azurepowershell
    $resourceGroupName = "<your new resource group name>"
    New-AzResourceGroup -Name $resourceGroupName -Location "East US 2"
    ```

    La salida debe ser parecida a la siguiente:

    ```output
    ResourceGroupName : hdiadlgrp
    Location          : eastus2
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/<subscription-id>/resourceGroups/hdiadlgrp
    ```

3. Cree una cuenta de Data Lake Storage Gen1. El nombre de cuenta que especifique debe contener solo letras minúsculas y números.

    ```azurepowershell
    $dataLakeStorageGen1Name = "<your new Data Lake Storage Gen1 account name>"
    New-AzDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStorageGen1Name -Location "East US 2"
    ```

    Debe ver algo parecido a lo siguiente:

    ```output
    ...
    ProvisioningState           : Succeeded
    State                       : Active
    CreationTime                : 5/5/2017 10:53:56 PM
    EncryptionState             : Enabled
    ...
    LastModifiedTime            : 5/5/2017 10:53:56 PM
    Endpoint                    : hdiadlstore.azuredatalakestore.net
    DefaultGroup                :
    Id                          : /subscriptions/<subscription-id>/resourceGroups/hdiadlgrp/providers/Microsoft.DataLakeStore/accounts/hdiadlstore
    Name                        : hdiadlstore
    Type                        : Microsoft.DataLakeStore/accounts
    Location                    : East US 2
    Tags                        : {}
    ```

5. Carga de algunos datos de ejemplo en Data Lake Storage Gen1. Los usaremos más adelante en este artículo para comprobar que se puede acceder a los datos desde un clúster de HDInsight. Si busca datos de ejemplo para cargar, puede obtener la carpeta **Ambulance Data** en el [repositorio Git de Azure Data Lake](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData).

    ```azurepowershell
    $myrootdir = "/"
    Import-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name -Path "C:\<path to data>\vehicle1_09142014.csv" -Destination $myrootdir\vehicle1_09142014.csv
    ```

## <a name="set-up-authentication-for-role-based-access-to-data-lake-storage-gen1"></a>Configuración de la autenticación para el acceso basado en roles a Data Lake Storage Gen1

Cada una de las suscripciones de Azure está asociada a un Azure Active Directory. Los usuarios y los servicios que acceden a los recursos de la suscripción mediante Azure Portal o la API de Azure Resource Manager deben autenticarse primero en ese Azure Active Directory. Para conceder acceso a servicios y suscripciones de Azure, se les asigna el rol adecuado en un recurso de Azure.  Para los servicios, una entidad de servicio identifica al servicio en Azure Active Directory (AAD). En esta sección, se muestra cómo conceder a un servicio de aplicación, como HDInsight, acceso a un recurso de Azure (la cuenta de Data Lake Storage Gen1 creada antes) mediante la creación de una entidad de servicio para la aplicación y la asignación de roles a ella a través de Azure PowerShell.

Para configurar la autenticación de Active Directory para Data Lake Storage Gen1, debe realizar las siguientes tareas.

* Creación de un certificado autofirmado
* Creación de una aplicación en Azure Active Directory y una entidad de servicio

### <a name="create-a-self-signed-certificate"></a>Creación de un certificado autofirmado

Asegúrese de que tiene [Windows SDK](https://dev.windows.com/en-us/downloads) instalado antes de continuar con los pasos de esta sección. También debe crear un directorio, como **C:\mycertdir**, donde se creará el certificado.

1. En la ventana de PowerShell, vaya a la ubicación donde ha instalado Windows SDK (normalmente, `C:\Program Files (x86)\Windows Kits\10\bin\x86`) y use la utilidad [MakeCert][makecert] para crear un certificado autofirmado y una clave privada. Use los comandos siguientes.

    ```azurepowershell
    $certificateFileDir = "<my certificate directory>"
    cd $certificateFileDir

    makecert -sv mykey.pvk -n "cn=HDI-ADL-SP" CertFile.cer -r -len 2048
    ```

    Se le pedirá que escriba la contraseña de la clave privada. Después de que el comando se ejecute correctamente, debería ver los archivos **CertFile.cer** y **mykey.pvk** en el directorio de certificado especificado.
2. Use la utilidad [Pvk2Pfx][pvk2pfx] para convertir los archivos .pvk y .cer que MakeCert ha creado en un archivo .pfx. Ejecute el siguiente comando:

    ```azurepowershell
    pvk2pfx -pvk mykey.pvk -spc CertFile.cer -pfx CertFile.pfx -po <password>
    ```

    Cuando se le pida, escriba la contraseña de la clave privada que especificó antes. El valor especificado para el parámetro **-po** es la contraseña asociada al archivo .pfx. Cuando el comando se complete correctamente, debería ver un archivo CertFile.pfx en el directorio de certificado especificado.

### <a name="create-an-azure-active-directory-and-a-service-principal"></a>Creación de una aplicación en Azure Active Directory y una entidad de servicio

En esta sección, llevará a cabo los pasos para crear una entidad de servicio para una aplicación de Azure Active Directory, asignar un rol a la entidad de servicio y autenticarse como la entidad de servicio proporcionando un certificado. Ejecute los comandos siguientes para crear una aplicación en Azure Active Directory.

1. Pegue los siguientes cmdlets en la ventana de consola de PowerShell. Asegúrese de que el valor especificado para la propiedad **- DisplayName** sea único. Además, los valores de **-HomePage** e **-IdentiferUris** son valores de marcador de posición y no se comprueban.

    ```azurepowershell
    $certificateFilePath = "$certificateFileDir\CertFile.pfx"

    $password = Read-Host -Prompt "Enter the password" # This is the password you specified for the .pfx file

    $certificatePFX = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($certificateFilePath, $password)

    $rawCertificateData = $certificatePFX.GetRawCertData()

    $credential = [System.Convert]::ToBase64String($rawCertificateData)

    $application = New-AzADApplication `
        -DisplayName "HDIADL" `
        -HomePage "https://contoso.com" `
        -IdentifierUris "https://mycontoso.com" `
        -CertValue $credential  `
        -StartDate $certificatePFX.NotBefore  `
        -EndDate $certificatePFX.NotAfter

    $applicationId = $application.ApplicationId
    ```

2. Cree una entidad de servicio con el identificador de aplicación.

    ```azurepowershell
    $servicePrincipal = New-AzADServicePrincipal -ApplicationId $applicationId

     $objectId = $servicePrincipal.Id
    ```

3. Conceda acceso de entidad de servicio al archivo o la carpeta de Data Lake Storage Gen1 a la que accederá desde el clúster de HDInsight. El fragmento de código siguiente proporciona acceso a la raíz de la cuenta de Data Lake Storage Gen1 (donde copió el archivo de datos de ejemplo) y al propio archivo.

    ```azurepowershell
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path / -AceType User -Id $objectId -Permissions All
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path /vehicle1_09142014.csv -AceType User -Id $objectId -Permissions All
    ```

## <a name="create-an-hdinsight-linux-cluster-with-data-lake-storage-gen1-as-additional-storage"></a>Creación de un clúster de HDInsight en Linux con Data Lake Storage Gen1 como almacenamiento adicional

En esta sección, se creará un clúster de Hadoop en HDInsight basado en Linux con Data Lake Storage Gen1 como almacenamiento adicional. En esta versión, el clúster de HDInsight y la cuenta de Data Lake Storage Gen1 deben estar en la misma ubicación.

1. Comience recuperando el identificador del inquilino de la suscripción. Lo necesitará más adelante.

    ```azurepowershell
    $tenantID = (Get-AzContext).Tenant.TenantId
    ```

2. En esta versión, para un clúster de Hadoop, Data Lake Storage Gen1 solo sirve como almacenamiento adicional para el clúster. El almacenamiento predeterminado seguirá siendo los blobs de almacenamiento de Azure (WASB). En primer lugar, crearemos la cuenta de almacenamiento y los contenedores de almacenamiento necesarios para el clúster.

    ```azurepowershell
    # Create an Azure storage account
    $location = "East US 2"
    $storageAccountName = "<StorageAccountName>"   # Provide a Storage account name

    New-AzStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName -Location $location -Type Standard_GRS

    # Create an Azure Blob Storage container
    $containerName = "<ContainerName>"              # Provide a container name
    $storageAccountKey = (Get-AzStorageAccountKey -Name $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
    $destContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
    New-AzStorageContainer -Name $containerName -Context $destContext
    ```

3. Cree el clúster de HDInsight. Use los siguientes cmdlets.

    ```azurepowershell
    # Set these variables
    $clusterName = $containerName                   # As a best practice, have the same name for the cluster and container
    $clusterNodes = <ClusterSizeInNodes>            # The number of nodes in the HDInsight cluster
    $httpCredentials = Get-Credential
    $sshCredentials = Get-Credential

    New-AzHDInsightCluster -ClusterName $clusterName -ResourceGroupName $resourceGroupName -HttpCredential $httpCredentials -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.windows.net" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainer $containerName  -ClusterSizeInNodes $clusterNodes -ClusterType Hadoop -Version "3.4" -OSType Linux -SshCredential $sshCredentials -ObjectID $objectId -AadTenantId $tenantID -CertificateFilePath $certificateFilePath -CertificatePassword $password
    ```

    Una vez que el cmdlet se complete correctamente, debería ver un resultado con la información del clúster.


## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-the-data-lake-storage-gen1-account"></a>Ejecución de trabajos de prueba en el clúster de HDInsight para usar la cuenta de Data Lake Storage Gen1
Después de configurar un clúster de HDInsight, puede ejecutar trabajos de prueba en el clúster para probar que el clúster de HDInsight pueda acceder a Data Lake Storage Gen1. Para hacerlo, ejecutaremos un trabajo de Hive de ejemplo que crea una tabla con los datos de ejemplo que cargó antes en la cuenta de Data Lake Storage Gen1.

En esta sección, aprenderá a usar SSH en el clúster de Linux en HDInsight que creó y ejecutará una consulta de Hive de ejemplo.

* Si usa un cliente Windows para usar SSH en el clúster, consulte [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).
* Si usa un cliente Linux para usar SSH en el clúster, consulte [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. Una vez conectado, inicie la CLI de Hive con el siguiente comando:

    ```azurepowershell
    hive
    ```

2. Mediante la CLI, especifique las siguientes instrucciones para crear una tabla nueva denominada **vehicles** con los datos de ejemplo de Data Lake Storage Gen1:

    ```azurepowershell
    DROP TABLE vehicles;
    CREATE EXTERNAL TABLE vehicles (str string) LOCATION 'adl://<mydatalakestoragegen1>.azuredatalakestore.net:443/';
    SELECT * FROM vehicles LIMIT 10;
    ```

    Debería ver una salida similar a la siguiente:

    ```output
    1,1,2014-09-14 00:00:03,46.81006,-92.08174,51,S,1
    1,2,2014-09-14 00:00:06,46.81006,-92.08174,13,NE,1
    1,3,2014-09-14 00:00:09,46.81006,-92.08174,48,NE,1
    1,4,2014-09-14 00:00:12,46.81006,-92.08174,30,W,1
    1,5,2014-09-14 00:00:15,46.81006,-92.08174,47,S,1
    1,6,2014-09-14 00:00:18,46.81006,-92.08174,9,S,1
    1,7,2014-09-14 00:00:21,46.81006,-92.08174,53,N,1
    1,8,2014-09-14 00:00:24,46.81006,-92.08174,63,SW,1
    1,9,2014-09-14 00:00:27,46.81006,-92.08174,4,NE,1
    1,10,2014-09-14 00:00:30,46.81006,-92.08174,31,N,1
    ```

## <a name="access-data-lake-storage-gen1-using-hdfs-commands"></a>Acceso a Data Lake Storage Gen1 mediante comandos de HDFS
Una vez que configure el clúster de HDInsight para que use Data Lake Storage Gen1, puede usar los comandos de shell de HDFS para acceder al almacén.

En esta sección, aprenderá a usar SSH en el clúster de Linux en HDInsight que creó y ejecutará los comandos de HDFS.

* Si usa un cliente Windows para usar SSH en el clúster, consulte [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).
* Si usa un cliente Linux para usar SSH en el clúster, consulte [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

Una vez conectado, utilice el siguiente comando del sistema de archivos HDFS para enumerar los archivos de la cuenta de Data Lake Storage Gen1.

```azurepowershell
hdfs dfs -ls adl://<Data Lake Storage Gen1 account name>.azuredatalakestore.net:443/
```

Se debería incluir el archivo que cargó antes en Data Lake Storage Gen1.

```output
15/09/17 21:41:15 INFO web.CaboWebHdfsFileSystem: Replacing original urlConnectionFactory with org.apache.hadoop.hdfs.web.URLConnectionFactory@21a728d6
Found 1 items
-rwxrwxrwx   0 NotSupportYet NotSupportYet     671388 2015-09-16 22:16 adl://mydatalakestoragegen1.azuredatalakestore.net:443/mynewfolder
```

También puede usar el comando `hdfs dfs -put` para cargar algunos archivos en Data Lake Storage Gen1 y después usar `hdfs dfs -ls` para comprobar si los archivos se cargaron correctamente.

## <a name="see-also"></a>Consulte también
* [Usar Data Lake Storage Gen1 con clústeres de Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-store.md)
* [Portal: Creación de un clúster de HDInsight para usar Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)

[makecert]: https://msdn.microsoft.com/library/windows/desktop/ff548309(v=vs.85).aspx
[pvk2pfx]: https://msdn.microsoft.com/library/windows/desktop/ff550672(v=vs.85).aspx
