---
title: Restricción del acceso mediante firmas de acceso compartido en Azure HDInsight
description: Obtener información acerca de cómo usar firmas de acceso compartido para restringir el acceso de HDInsight a datos almacenados en blobs de Almacenamiento de Azure.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/28/2020
ms.openlocfilehash: 8ab181eb72b5a3ab54ad8dba19d23288926b8969
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87006320"
---
# <a name="use-azure-storage-shared-access-signatures-to-restrict-access-to-data-in-hdinsight"></a>Uso de firmas de acceso compartido de Azure Storage para restringir el acceso a datos en HDInsight

HDInsight tiene acceso total a los datos de las cuentas de Azure Storage asociadas con el clúster. Puede usar firmas de acceso compartido en el contenedor de blobs para restringir el acceso a los datos. Las firmas de acceso compartido (SAS) son una característica de las cuentas de Almacenamiento de Azure que permite limitar el acceso a los datos. Por ejemplo, al proporcionar acceso de solo lectura a los datos.

> [!IMPORTANT]  
> Para una solución con Apache Ranger, considere la posibilidad de usar HDInsight unido a un dominio. Para más información, consulte el documento [Configuración de clústeres de HDInsight unidos a un dominio](./domain-joined/apache-domain-joined-configure.md).

> [!WARNING]  
> HDInsight debe tener acceso total al almacenamiento predeterminado para el clúster.

## <a name="prerequisites"></a>Prerrequisitos

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](./hdinsight-hadoop-linux-use-ssh-unix.md).

* Un [contenedor de almacenamiento](../storage/blobs/storage-quickstart-blobs-portal.md) existente.  

* Si usa PowerShell, necesitará el [Módulo Az](https://docs.microsoft.com/powershell/azure/).

* Si quiere usar la CLI de Azure y todavía no la ha instalado, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

* Si usa [Python](https://www.python.org/downloads/), versión 2.7 o superior.

* Si usa C#, debe usar la versión 2013 de Visual Studio o superior.

* El esquema de URI para la cuenta de almacenamiento. Este esquema sería `wasb://` para Azure Storage, `abfs://` para Azure Data Lake Storage Gen2 o `adl://` para Azure Data Lake Storage Gen1. Si se habilita la transferencia segura para Azure Storage, el identificador URI sería `wasbs://`.

* Un clúster de HDInsight existente al que agregar una firma de acceso compartido. Si no es así, puede usar Azure PowerShell para crear un clúster y agregar una firma de acceso compartido durante la creación del clúster.

* Los archivos de ejemplo de [https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature). Este repositorio contiene los siguientes elementos:

  * Un proyecto de Visual Studio que puede crear un contenedor de almacenamiento, una directiva almacenada y una SAS para su uso con HDInsight.
  * Un script de Python que puede crear un contenedor de almacenamiento, una directiva almacenada y una SAS para su uso con HDInsight.
  * Un script de PowerShell que puede crear un clúster de HDInsight y configurarlo para que use la SAS. Más adelante se usa una versión actualizada.
  * Un archivo de ejemplo: `hdinsight-dotnet-python-azure-storage-shared-access-signature-master\sampledata\sample.log`

## <a name="shared-access-signatures"></a>Las firmas de acceso compartido

Hay dos formas de firmas de acceso compartido:

* `Ad hoc`: la hora de inicio, la hora de expiración y los permisos para la firma de acceso compartido se especifican en el URI de esta.

* `Stored access policy`: se define una directiva de acceso almacenada en un contenedor de recursos, como un contenedor de blobs. Una directiva puede usarse para administrar las restricciones de una o varias firmas de acceso compartido. Cuando asocia una SAS a una directiva de acceso almacenada, la SAS hereda las restricciones (hora de inicio, hora de expiración y permisos) definidas para la directiva de acceso almacenada.

La diferencia entre las dos formas es importante para un escenario principal: revocación. Una SAS es una dirección URL, por lo que cualquier persona que obtenga la SAS puede usarla. No importa quién la solicitó originalmente. Si una SAS se encuentra disponible públicamente, cualquier persona del mundo puede usarla. Una SAS distribuida es válida hasta que se produzca una de las cuatro situaciones:

1. Se alcanza el tiempo de expiración especificado en la SAS.

2. Se alcanza la hora de expiración especificada en la directiva de acceso almacenada a la que hace referencia la SAS. Los escenarios siguientes hacen que se alcance la hora de expiración:

    * Ha transcurrido el intervalo de tiempo.
    * La directiva de acceso almacenada se ha modificado para que la hora de expiración haya pasado. Cambiar la hora de expiración es una manera de revocar la firma de acceso compartido.

3. Se elimina la directiva de acceso almacenada a la que hace referencia la SAS, que es otra forma de revocar la SAS. Si se vuelve a crear la directiva de acceso almacenada con el mismo nombre, todos los tokens de SAS de la directiva anterior son válidos (si no se ha superado el tiempo de expiración de la SAS). Si prevé revocar la SAS, asegúrese de usar un nombre distinto si vuelve a crear la directiva de acceso con una hora de expiración futura.

4. Se vuelve a generar la clave de cuenta que se usó para crear la SAS. Regenerar la clave hace que todas las aplicaciones que usan la clave anterior no se puedan autenticar. Actualice todos los componentes con la nueva clave.

> [!IMPORTANT]  
> Los URI de firma de acceso compartido están asociados a la clave de la cuenta que se utiliza para crear la firma y a la directiva de acceso almacenada correspondiente (en su caso). Si no se especifica una directiva de acceso almacenada, la única forma de revocar una firma de acceso compartido es cambiar la clave de la cuenta.

Se recomienda usar siempre las directivas de acceso almacenadas. Al utilizar las directivas almacenadas, puede revocar las firmas o ampliar la fecha de expiración según sea necesario. Los pasos descritos en este documento utilizan directivas de acceso almacenadas para generar las SAS.

Para más información sobre firmas de acceso compartido, consulte [Firmas de acceso compartido, Parte 1: Descripción del modelo de firmas de acceso compartido](../storage/common/storage-dotnet-shared-access-signature-part-1.md)

## <a name="create-a-stored-policy-and-sas"></a>Creación de una directiva almacenada y una SAS

Guarde el token de SAS que se crea al final de cada método. El token será similar al siguiente resultado:

```output
?sv=2018-03-28&sr=c&si=myPolicyPS&sig=NAxefF%2BrR2ubjZtyUtuAvLQgt%2FJIN5aHJMj6OsDwyy4%3D
```

### <a name="using-powershell"></a>Usar PowerShell

Reemplace `RESOURCEGROUP`, `STORAGEACCOUNT` y `STORAGECONTAINER` por los valores adecuados para el contenedor de almacenamiento existente. Cambie el directorio a `hdinsight-dotnet-python-azure-storage-shared-access-signature-master` o revise el parámetro `-File` para que incluya la ruta de acceso absoluta para `Set-AzStorageblobcontent`. Escriba el siguiente comando de PowerShell:

```powershell
$resourceGroupName = "RESOURCEGROUP"
$storageAccountName = "STORAGEACCOUNT"
$containerName = "STORAGECONTAINER"
$policy = "myPolicyPS"

# Login to your Azure subscription
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Get the access key for the Azure Storage account
$storageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $storageAccountName)[0].Value

# Create an Azure Storage context
$storageContext = New-AzStorageContext `
                                -StorageAccountName $storageAccountName `
                                -StorageAccountKey $storageAccountKey

# Create a stored access policy for the Azure storage container
New-AzStorageContainerStoredAccessPolicy `
   -Container $containerName `
   -Policy $policy `
   -Permission "rl" `
   -ExpiryTime "12/31/2025 08:00:00" `
   -Context $storageContext

# Get the stored access policy or policies for the Azure storage container
Get-AzStorageContainerStoredAccessPolicy `
    -Container $containerName `
    -Context $storageContext

# Generates an SAS token for the Azure storage container
New-AzStorageContainerSASToken `
    -Name $containerName `
    -Policy $policy `
    -Context $storageContext

<# Removes a stored access policy from the Azure storage container
Remove-AzStorageContainerStoredAccessPolicy `
    -Container $containerName `
    -Policy $policy `
    -Context $storageContext
#>

# upload a file for a later example
Set-AzStorageblobcontent `
    -File "./sampledata/sample.log" `
    -Container $containerName `
    -Blob "samplePS.log" `
    -Context $storageContext
```

### <a name="using-azure-cli"></a>Uso de la CLI de Azure

El uso de variables en esta sección se basa en un entorno de Windows. Se necesitarán ligeras variaciones para Bash u otros entornos.

1. Reemplace `STORAGEACCOUNT` y `STORAGECONTAINER` con los valores adecuados para el contenedor de almacenamiento existente.

    ```azurecli
    # set variables
    set AZURE_STORAGE_ACCOUNT=STORAGEACCOUNT
    set AZURE_STORAGE_CONTAINER=STORAGECONTAINER

    #Login
    az login

    # If you have multiple subscriptions, set the one to use
    # az account set --subscription SUBSCRIPTION

    # Retrieve the primary key for the storage account
    az storage account keys list --account-name %AZURE_STORAGE_ACCOUNT% --query "[0].{PrimaryKey:value}" --output table
    ```

2. Establezca la clave principal recuperada en una variable para su uso posterior. Reemplace `PRIMARYKEY` con el valor recuperado en el paso anterior y, después, escriba el comando siguiente:

    ```console
    #set variable for primary key
    set AZURE_STORAGE_KEY=PRIMARYKEY
    ```

3. Cambie el directorio a `hdinsight-dotnet-python-azure-storage-shared-access-signature-master` o revise el parámetro `--file` para que incluya la ruta de acceso absoluta para `az storage blob upload`. Ejecute los comandos restantes:

    ```azurecli
    # Create stored access policy on the containing object
    az storage container policy create --container-name %AZURE_STORAGE_CONTAINER% --name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT% --expiry 2025-12-31 --permissions rl

    # List stored access policies on a containing object
    az storage container policy list --container-name %AZURE_STORAGE_CONTAINER% --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # Generate a shared access signature for the container
    az storage container generate-sas --name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # Reversal
    # az storage container policy delete --container-name %AZURE_STORAGE_CONTAINER% --name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # upload a file for a later example
    az storage blob upload --container-name %AZURE_STORAGE_CONTAINER% --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT% --name sampleCLI.log --file "./sampledata/sample.log"
    ```

### <a name="using-python"></a>Usar Python

Abra el archivo `SASToken.py` y reemplace `storage_account_name`, `storage_account_key` y `storage_container_name` con los valores adecuados para el contenedor de almacenamiento existente, y después ejecute el script.

Es posible que tenga que ejecutar `pip install --upgrade azure-storage` si recibe el mensaje de error `ImportError: No module named azure.storage`.

### <a name="using-c"></a>Uso de C\#

1. Abra la solución en Visual Studio.

2. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **SASExample** y seleccione **Propiedades**.

3. Seleccione **Configuración** y agregue valores para las siguientes entradas:

    |Elemento |Descripción |
    |---|---|
    |StorageConnectionString|la cadena de conexión de la cuenta de almacenamiento para la que desea crear una directiva almacenada y una SAS. El formato debe ser `DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey` donde `myaccount` es el nombre de la cuenta de almacenamiento y `mykey` es la clave para la cuenta de almacenamiento.|
    |ContainerName|El contenedor de la cuenta de almacenamiento a la que desea restringir el acceso.|
    |SASPolicyName|El nombre que se usará para la directiva almacenada que se va a crear.|
    |FileToUpload|La ruta de acceso a un archivo que se carga en el contenedor.|

4. Ejecute el proyecto. Guarde el token de directiva de SAS, el nombre de la cuenta de almacenamiento y el nombre del contenedor. Estos valores se usan al asociar la cuenta de almacenamiento con el clúster de HDInsight.

## <a name="use-the-sas-with-hdinsight"></a>Uso de las SAS con HDInsight

Al crear un clúster de HDInsight, debe especificar una cuenta de almacenamiento principal. También puede especificar cuentas de almacenamiento adicionales. Estos dos métodos para agregar almacenamiento requieren tener un acceso total a las cuentas de almacenamiento y los contenedores que se utilizan.

Use una firma de acceso compartido para limitar el acceso al contenedor. Agregue una entrada personalizada a la configuración de **core-site** para el clúster. Puede agregar la entrada durante la creación del clúster mediante PowerShell, o bien con Ambari después de crearlo.

### <a name="create-a-cluster-that-uses-the-sas"></a>Crear un clúster que usa la SAS

Reemplace `CLUSTERNAME`, `RESOURCEGROUP`, `DEFAULTSTORAGEACCOUNT`, `STORAGECONTAINER`, `STORAGEACCOUNT` y `TOKEN` por los valores adecuados. Escriba los comandos de PowerShell:

```powershell
$clusterName = 'CLUSTERNAME'
$resourceGroupName = 'RESOURCEGROUP'

# Replace with the Azure data center you want to the cluster to live in
$location = 'eastus'

# Replace with the name of the default storage account TO BE CREATED
$defaultStorageAccountName = 'DEFAULTSTORAGEACCOUNT'

# Replace with the name of the SAS container CREATED EARLIER
$SASContainerName = 'STORAGECONTAINER'

# Replace with the name of the SAS storage account CREATED EARLIER
$SASStorageAccountName = 'STORAGEACCOUNT'

# Replace with the SAS token generated earlier
$SASToken = 'TOKEN'

# Default cluster size (# of worker nodes), version, and type
$clusterSizeInNodes = "4"
$clusterVersion = "3.6"
$clusterType = "Hadoop"

# Login to your Azure subscription
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Create an Azure Storage account and container
New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableHttpsTrafficOnly 1

$defaultStorageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $defaultStorageAccountName)[0].Value

$defaultStorageContext = New-AzStorageContext `
                                -StorageAccountName $defaultStorageAccountName `
                                -StorageAccountKey $defaultStorageAccountKey

# Create a blob container. This holds the default data store for the cluster.
New-AzStorageContainer `
    -Name $clusterName `
    -Context $defaultStorageContext

# Cluster login is used to secure HTTPS services hosted on the cluster
$httpCredential = Get-Credential `
    -Message "Enter Cluster login credentials" `
    -UserName "admin"

# SSH user is used to remotely connect to the cluster using SSH clients
$sshCredential = Get-Credential `
    -Message "Enter SSH user credentials" `
    -UserName "sshuser"

# Create the configuration for the cluster
$config = New-AzHDInsightClusterConfig

$config = $config | Add-AzHDInsightConfigValue `
    -Spark2Defaults @{} `
    -Core @{"fs.azure.sas.$SASContainerName.$SASStorageAccountName.blob.core.windows.net"=$SASToken}

# Create the HDInsight cluster
New-AzHDInsightCluster `
    -Config $config `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $clusterName `
    -Location $location `
    -ClusterSizeInNodes $clusterSizeInNodes `
    -ClusterType $clusterType `
    -OSType Linux `
    -Version $clusterVersion `
    -HttpCredential $httpCredential `
    -SshCredential $sshCredential `
    -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
    -DefaultStorageAccountKey $defaultStorageAccountKey `
    -DefaultStorageContainer $clusterName

<# REVERSAL
Remove-AzHDInsightCluster `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $clusterName

Remove-AzStorageContainer `
    -Name $clusterName `
    -Context $defaultStorageContext

Remove-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName

Remove-AzResourceGroup `
    -Name $resourceGroupName
#>
```

> [!IMPORTANT]  
> Cuando se le pida el nombre de usuario SSH o HTTP/s y la contraseña, debe proporcionar una contraseña que cumpla los criterios siguientes:
>
> * Debe tener como mínimo 10 caracteres.
> * Debe contener al menos un dígito.
> * Debe incluir al menos un carácter no alfanumérico.
> * Debe contener al menos una mayúscula o una minúscula.

Este script tarda un tiempo en completarse, normalmente unos 15 minutos. Una vez finalizado el script sin errores, se creará el clúster.

### <a name="use-the-sas-with-an-existing-cluster"></a>Usar la SAS con un clúster existente

Si tiene un clúster existente, puede agregar la SAS para la configuración **core-site** mediante los pasos siguientes:

1. Abra la interfaz de usuario web Ambari del clúster. La dirección de esta página es `https://YOURCLUSTERNAME.azurehdinsight.net`. Cuando se le solicite, autentíquese en el clúster con el nombre de administrador (admin) y la contraseña que usó al crear el clúster.

1. Vaya a **HDFS** > **Configs** > **Advanced** > **Custom core-site** (HDFS > Configuraciones > Avanzado > Sitio principal personalizado).

1. Expanda la sección **Custom core-site** (Sitio principal personalizado), desplácese hasta el final y seleccione **Add property...** (Agregar propiedad...). Utilice los siguientes valores para los campos **Key** (Clave) y **Value** (Valor):

    * **Clave**: `fs.azure.sas.CONTAINERNAME.STORAGEACCOUNTNAME.blob.core.windows.net`
    * **Valor**: La SAS devuelta por uno de los métodos que se han ejecutado antes.

    Reemplace `CONTAINERNAME` por el nombre del contenedor que ha usado con la aplicación de C# o de SAS. Reemplace `STORAGEACCOUNTNAME` por el nombre de la cuenta de almacenamiento que ha usado.

    Seleccione **Add** (Agregar) para guardar esta clave y valor.

1. Seleccione el botón **Save** (Guardar) para guardar los cambios en la configuración. Cuando se le solicite, agregue una descripción del cambio ("Agregar acceso de almacenamiento de SAS", por ejemplo) y, a continuación, seleccione **Save** (Guardar).

    Seleccione **OK** (Aceptar) cuando se hayan completado los cambios.

   > [!IMPORTANT]  
   > Debe reiniciar varios servicios para que el cambio surta efecto.

1. Aparecerá una lista desplegable **Restart** (Reiniciar). Seleccione **Restart All Affected** (Reiniciar todos los afectados) en la lista desplegable y, a continuación, seleccione __Confirm Restart All__ (Confirmar reiniciar todos).

    Repita este proceso para **MapReduce2** y **YARN**.

1. Una vez reiniciados los servicios, seleccione cada uno de ellos y deshabilite el modo de mantenimiento en la lista desplegable **Service Actions** (Acciones del servicio).

## <a name="test-restricted-access"></a>Prueba de acceso restringido

Siga los pasos siguientes para comprobar que solo puede leer y enumerar elementos de la cuenta de almacenamiento de SAS.

1. Conéctese al clúster. Reemplace `CLUSTERNAME` por el nombre del clúster y escriba el comando siguiente:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Para mostrar el contenido del contenedor, utilice el siguiente comando desde el símbolo del sistema:

    ```bash
    hdfs dfs -ls wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/
    ```

    Reemplace `SASCONTAINER` por el nombre del contenedor creado para la cuenta de almacenamiento de SAS. Reemplace `SASACCOUNTNAME` por el nombre de la cuenta de almacenamiento que se ha usado para la firma de acceso compartido.

    La lista incluye el archivo que se cargó al crear el contenedor y la firma de acceso compartido.

3. Use el siguiente comando para comprobar que puede leer el contenido del archivo. Reemplace `SASCONTAINER` y `SASACCOUNTNAME` como en el paso anterior. Reemplace `sample.log` por el nombre del archivo que se muestra en el comando anterior:

    ```bash
    hdfs dfs -text wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/sample.log
    ```

    Este comando muestra el contenido del archivo.

4. Use el siguiente comando para descargar el archivo en el sistema de archivos local:

    ```bash
    hdfs dfs -get wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/sample.log testfile.txt
    ```

    Este comando descarga el archivo en un archivo local denominado **testfile.txt**.

5. Use el siguiente comando para cargar el archivo local en un nuevo archivo denominado **testupload.txt** en el almacenamiento de SAS:

    ```bash
    hdfs dfs -put testfile.txt wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/testupload.txt
    ```

    Recibirá un mensaje similar al texto siguiente:

    ```output
    put: java.io.IOException
    ```

    Este error se produce porque la ubicación de almacenamiento es solo de lectura y lista. Use el siguiente comando para colocar los datos en el almacenamiento predeterminado para el clúster, que tiene permiso de escritura:

    ```bash
    hdfs dfs -put testfile.txt wasbs:///testupload.txt
    ```

    Esta vez, la operación debe completarse correctamente.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a agregar almacenamiento de acceso limitado al clúster de HDInsight, obtenga información acerca de otras maneras de trabajar con datos en el clúster:

* [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md)
* [Autorización de usuarios para vistas de Apache Ambari](hdinsight-authorize-users-to-ambari.md)
