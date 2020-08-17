---
title: 'Plantilla: clúster de HDInsight con Data Lake Storage Gen1'
description: Use las plantillas de Azure Resource Manager para crear y usar clústeres de HDInsight con Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 33c54738b1ab3c90118c86bbf78bdcc3348658e0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87048715"
---
# <a name="create-an-hdinsight-cluster-with-azure-data-lake-storage-gen1-using-azure-resource-manager-template"></a>Creación de un clúster de HDInsight con Azure Data Lake Storage Gen1 mediante las plantillas de Azure Resource Manager
> [!div class="op_single_selector"]
> * [Uso del portal](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Uso de PowerShell (para el almacenamiento predeterminado)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Uso de PowerShell (para el almacenamiento adicional)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Uso de Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
>
>

Aprenda a usar Azure PowerShell para configurar un clúster de HDInsight con Azure Data Lake Storage Gen1, **como almacenamiento adicional**.

Para los tipos de clúster compatibles, Data Lake Storage Gen1 se puede usar como un almacenamiento predeterminado o como una cuenta de almacenamiento adicional. Cuando Data Lake Storage Gen1 se usa como almacenamiento adicional, la cuenta de almacenamiento predeterminada para los clústeres seguirá siendo Azure Storage Blobs (WASB) y los archivos relacionados con clústeres (como registros, etc.) seguirán escribiéndose en el almacenamiento predeterminado, mientras que los datos que quiere procesar pueden almacenarse en una cuenta de Data Lake Storage Gen1. Utilizar Data Lake Storage Gen1 como una cuenta de almacenamiento adicional no afecta al rendimiento ni a la capacidad de lectura y escritura en el almacenamiento del clúster.

## <a name="using-data-lake-storage-gen1-for-hdinsight-cluster-storage"></a>Uso de Data Lake Storage Gen1 para el almacenamiento de clústeres de HDInsight

Estas son algunos puntos importantes que se deben tener en cuenta al usar HDInsight con Data Lake Storage Gen1:

* Opción para crear clústeres de HDInsight con acceso a Data Lake Storage Gen1 como almacenamiento predeterminado está disponible para la versión 3.5 y 3.6 de HDInsight.

* La opción para crear clústeres de HDInsight con acceso a Data Lake Storage Gen1 como almacenamiento está disponible para las versiones 3.2, 3.4, 3.5 y 3.6 de HDInsight.

En este artículo, aprovisionamos un clúster de Hadoop con Data Lake Storage Gen1 como almacenamiento adicional. Para instrucciones sobre cómo crear un clúster de Hadoop con Data Lake Storage Gen1 como almacenamiento predeterminado, consulte [Creación de un clúster de HDInsight con Data Lake Storage Gen1 mediante Azure Portal](data-lake-store-hdinsight-hadoop-use-portal.md).

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Antes de empezar este tutorial, debe contar con lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1.0 o versiones posteriores**. Consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).
* **Entidad de servicio de Azure Active Directory** En los pasos de este tutorial se proporcionan instrucciones sobre cómo crear entidades de servicio de Azure AD. Sin embargo, debe ser administrador de Azure AD para poder crearlas. Si ya lo es, puede hacer caso omiso a este requisito previo y continuar con el tutorial.

    **Si no lo es**, no podrá realizar los pasos necesarios para crear una entidad de servicio. En este caso, su administrador de Azure AD debe generar primero una entidad de servicio antes de crear un clúster de HDInsight con Data Lake Storage Gen1. Además, la entidad de servicio debe crearse con un certificado, tal y como se describe en [Creación de una entidad de servicio](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-certificate-from-certificate-authority).

## <a name="create-an-hdinsight-cluster-with-data-lake-storage-gen1"></a>Creación de un clúster de HDInsight con Data Lake Storage Gen1 mediante el Portal de Azure
Las plantillas de Resource Manager y los requisitos previos para usarla están disponibles en GitHub: [Deploy a HDInsight Linux cluster with new Data Lake Store](https://github.com/Azure/azure-quickstart-templates/tree/master/201-hdinsight-datalake-store-azure-storage) (Implementación de un clúster Linux de HDInsight con la nueva versión de Data Lake Storage Gen1). Siga las instrucciones de este vínculo para crear un clúster de HDInsight con Data Lake Storage Gen1 como almacenamiento adicional.

Estas instrucciones requieren el uso de PowerShell. Antes de comenzar con las instrucciones, asegúrese de iniciar sesión en su cuenta de Azure. En el escritorio, abra una nueva ventana de Azure PowerShell y escriba los siguientes fragmentos. Cuando se le solicite iniciar sesión, asegúrese de iniciarla como uno de los administradores o propietarios de la suscripción:

```
# Log in to your Azure account
Connect-AzAccount

# List all the subscriptions associated to your account
Get-AzSubscription

# Select a subscription
Set-AzContext -SubscriptionId <subscription ID>
```

La plantilla implementa estos tipos de recursos:

* [Microsoft.DataLakeStore/accounts](/azure/templates/microsoft.datalakestore/accounts)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.HDInsight/clusters](/azure/templates/microsoft.hdinsight/clusters)

## <a name="upload-sample-data-to-data-lake-storage-gen1"></a>Carga de datos de ejemplo en Data Lake Storage Gen1
Las plantilla de Resource Manager crean una cuenta de Data Lake Storage Gen1 y la asocian con el clúster de HDInsight. Ahora, debe cargar algunos datos de ejemplo en Data Lake Storage Gen1. Necesitará estos datos más adelante en el tutorial para ejecutar trabajos desde un clúster de HDInsight que accedan a datos en Data Lake Storage Gen1. Para ver las instrucciones sobre cómo cargar datos, consulte el artículo de [carga de archivos en la cuenta de Data Lake Storage Gen1](data-lake-store-get-started-portal.md#uploaddata). Si busca datos de ejemplo para cargar, puede obtener la carpeta **Ambulance Data** en el [repositorio Git de Azure Data Lake](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData).

## <a name="set-relevant-acls-on-the-sample-data"></a>Establecimiento de listas ACL pertinentes en los datos de ejemplo
Para confirmar que los datos de ejemplo que carga están accesibles desde el clúster de HDInsight, debe asegurarse de que la aplicación de Azure AD que utiliza para establecer la identidad entre el clúster de HDInsight y Data Lake Storage Gen1 tiene acceso al archivo o a la carpeta deseados. Para ello, siga estos pasos.

1. Busque el nombre de la aplicación de Azure AD asociada al clúster de HDInsight y a la cuenta de Data Lake Storage Gen1. Una manera de buscar el nombre es abrir la hoja de clúster de HDInsight que creó mediante la plantilla de Resource Manager, hacer clic en la pestaña **Identidad de AAD del clúster** y buscar el valor del **nombre para mostrar de la entidad de servicio**.
2. Ahora, proporcione acceso a esta aplicación de Azure AD en el archivo o la carpeta a los quiera acceder desde el clúster de HDInsight. Para establecer las listas ACL adecuadas en el archivo o carpeta de Data Lake Storage Gen1, consulte el artículo sobre cómo [proteger los datos de Data Lake Storage Gen1](data-lake-store-secure-data.md#filepermissions).

## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-data-lake-storage-gen1"></a>Ejecución de trabajos de prueba en el clúster de HDInsight para usar Data Lake Storage Gen1
Después de configurar un clúster de HDInsight, puede ejecutar trabajos de prueba en el clúster para probar que el clúster de HDInsight pueda acceder a Data Lake Storage Gen1. Para hacerlo, ejecutaremos un trabajo de Hive de ejemplo que crea una tabla con los datos de ejemplo que cargó antes en la cuenta de Data Lake Storage Gen1.

En esta sección, se explica cómo usar SSH en el clúster de Linux en HDInsight y ejecutar la consulta de Hive de ejemplo. Si está utilizando un cliente de Windows, se recomienda utilizar **PuTTY**, que puede descargarse desde [https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Para obtener más información sobre el uso de PuTTY, consulte [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Linux, Unix u OS X (versión preliminar)](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).

1. Una vez conectado, inicie la CLI de Hive con el siguiente comando:

   ```
   hive
   ```
2. Mediante la CLI, especifique las siguientes instrucciones para crear una tabla nueva denominada **vehicles** con los datos de ejemplo de Data Lake Storage Gen1:

   ```
   DROP TABLE vehicles;
   CREATE EXTERNAL TABLE vehicles (str string) LOCATION 'adl://<mydatalakestoragegen1>.azuredatalakestore.net:443/';
   SELECT * FROM vehicles LIMIT 10;
   ```

   Debería ver un resultado similar al siguiente:

   ```
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

En esta sección, se explica cómo usar SSH en un clúster de Linux en HDInsight y ejecutar los comandos de HDFS. Si está utilizando un cliente de Windows, se recomienda utilizar **PuTTY**, que puede descargarse desde [https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Para obtener más información sobre el uso de PuTTY, consulte [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Linux, Unix u OS X (versión preliminar)](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).

Una vez conectado, utilice el siguiente comando del sistema de archivos HDFS para enumerar los archivos de la cuenta de Data Lake Storage Gen1.

```
hdfs dfs -ls adl://<Data Lake Storage Gen1 account name>.azuredatalakestore.net:443/
```

Se debería incluir el archivo que cargó antes en Data Lake Storage Gen1.

```
15/09/17 21:41:15 INFO web.CaboWebHdfsFileSystem: Replacing original urlConnectionFactory with org.apache.hadoop.hdfs.web.URLConnectionFactory@21a728d6
Found 1 items
-rwxrwxrwx   0 NotSupportYet NotSupportYet     671388 2015-09-16 22:16 adl://mydatalakestoragegen1.azuredatalakestore.net:443/mynewfolder
```

También puede usar el comando `hdfs dfs -put` para cargar algunos archivos en Data Lake Storage Gen1 y después usar `hdfs dfs -ls` para comprobar si los archivos se cargaron correctamente.


## <a name="next-steps"></a>Pasos siguientes
* [Copia de datos de los blobs de Azure Storage en Data Lake Storage Gen1](data-lake-store-copy-data-wasb-distcp.md)
* [Usar Data Lake Storage Gen1 con clústeres de Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-store.md)
