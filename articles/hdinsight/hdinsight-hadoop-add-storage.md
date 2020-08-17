---
title: Adición de más cuentas de Azure Storage a HDInsight
description: Aprenda a agregar cuentas adicionales de Azure Storage a un clúster de HDInsight existente.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 04/27/2020
ms.openlocfilehash: 23e7b0f8dcb0c64259627d5350511ebdc48d6fac
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078977"
---
# <a name="add-additional-storage-accounts-to-hdinsight"></a>Adición de más cuentas de almacenamiento a HDInsight

Aprenda a usar acciones de script para agregar *cuentas* adicionales de Azure Storage a HDInsight. Los pasos descritos en este documento agregan una *cuenta* de almacenamiento a un clúster de HDInsight existente. Este artículo se aplica a las *cuentas* de almacenamiento (no la cuenta de almacenamiento predeterminada del clúster) y tampoco al almacenamiento adicional, como [`Azure Data Lake Storage Gen1`](hdinsight-hadoop-use-data-lake-store.md) y [`Azure Data Lake Storage Gen2`](hdinsight-hadoop-use-data-lake-storage-gen2.md).

> [!IMPORTANT]  
> La información de este documento trata sobre cómo agregar cuentas de almacenamiento adicionales a un clúster después de que se ha creado. Para información sobre cómo agregar cuentas de almacenamiento durante la creación de clústeres, consulte [Configuración de clústeres en HDInsight con Apache Hadoop, Apache Spark, Apache Kafka, etc](hdinsight-hadoop-provision-linux-clusters.md).

## <a name="prerequisites"></a>Requisitos previos

* Un clúster de Hadoop en HDInsight. Consulte [Introducción a HDInsight en Linux](./hadoop/apache-hadoop-linux-tutorial-get-started.md).
* Nombre y clave de la cuenta de almacenamiento. Vea [Administración de las claves de acceso de la cuenta de almacenamiento](../storage/common/storage-account-keys-manage.md).
* Si usa PowerShell, necesitará el módulo AZ.  Consulte [Introducción a Azure PowerShell](https://docs.microsoft.com/powershell/azure/).

## <a name="how-it-works"></a>Funcionamiento

Durante el procesamiento, el script realiza las siguientes acciones:

* Si la cuenta de almacenamiento ya existe en la configuración de core-site.xml para el clúster, el script se cierra y no se lleva a cabo ninguna otra acción.

* Comprueba que la cuenta de almacenamiento existe y que se puede acceder a ella mediante la clave.

* Cifra la clave con la credencial del clúster.

* Agrega la cuenta de almacenamiento al archivo core-site.xml.

* Detiene y reinicia los servicios Apache Oozie, Apache Hadoop YARN, Apache Hadoop MapReduce2 y Apache Hadoop HDFS. Detener e iniciar estos servicios permite que usen la nueva cuenta de almacenamiento.

> [!WARNING]  
> No se admite el uso de una cuenta de almacenamiento en una ubicación diferente a la del clúster de HDInsight.

## <a name="add-storage-account"></a>Agregar cuenta de almacenamiento

Use [Acción de script](hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster) para aplicar los cambios con las siguientes consideraciones:

|Propiedad | Value |
|---|---|
|URI de script de Bash|`https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh`|
|Tipos de nodo|Head|
|Parámetros|`ACCOUNTNAME` `ACCOUNTKEY` `-p` (opcional)|

* `ACCOUNTNAME` es el nombre de la cuenta de almacenamiento que se agregará al clúster de HDInsight.
* `ACCOUNTKEY` es la clave de acceso para `ACCOUNTNAME`.
* `-p` es opcional. si se especifica, la clave no se cifra y se almacena en el archivo core-site.xml como texto sin formato.

## <a name="verification"></a>Comprobación

Cuando vea el clúster de HDInsight en Azure Portal, si selecciona la entrada __Cuentas de almacenamiento__ en __Propiedades__, no se mostrarán las cuentas de almacenamiento agregadas mediante esta acción de script. Azure PowerShell y la CLI de Azure tampoco mostrarán la cuenta de almacenamiento adicional. Esto se debe a que el script solo modifica la configuración de `core-site.xml` del clúster. Esta información no se usa al recuperar la información del clúster mediante las API de administración de Azure.

Para comprobar el almacenamiento adicional, use uno de los métodos que se muestran a continuación:

### <a name="powershell"></a>PowerShell

El script devolverá los nombres de cuenta de almacenamiento asociados al clúster especificado. Reemplace `CLUSTERNAME` por el nombre de clúster real y después ejecute el script.

```powershell
# Update values
$clusterName = "CLUSTERNAME"

$creds = Get-Credential -UserName "admin" -Message "Enter the cluster login credentials"

$clusterName = $clusterName.ToLower();

# getting service_config_version
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName`?fields=Clusters/desired_service_config_versions/HDFS" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content

$configVersion=$respObj.Clusters.desired_service_config_versions.HDFS.service_config_version

$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=$configVersion" `
    -Credential $creds
$respObj = ConvertFrom-Json $resp.Content

# extract account names
$value = ($respObj.items.configurations | Where type -EQ "core-site").properties | Get-Member -membertype properties | Where Name -Like "fs.azure.account.key.*"
foreach ($name in $value ) { $name.Name.Split(".")[4]}
```

### <a name="apache-ambari"></a>Apache Ambari

1. En un explorador web, vaya a `https://CLUSTERNAME.azurehdinsight.net`, donde `CLUSTERNAME` es el nombre del clúster.

1. Vaya a **HDFS** > **Configs** > **Advanced** > **Custom core-site** (HDFS > Configuraciones > Avanzado > Sitio principal personalizado).

1. Observe las claves que comienzan por `fs.azure.account.key`. El nombre de la cuenta formará parte de la clave, tal como se muestra en esta imagen de ejemplo:

   ![comprobación a través de Apache Ambari](./media/hdinsight-hadoop-add-storage/apache-ambari-verification.png)

## <a name="remove-storage-account"></a>Eliminación de la cuenta de almacenamiento

1. En un explorador web, vaya a `https://CLUSTERNAME.azurehdinsight.net`, donde `CLUSTERNAME` es el nombre del clúster.

1. Vaya a **HDFS** > **Configs** > **Advanced** > **Custom core-site** (HDFS > Configuraciones > Avanzado > Sitio principal personalizado).

1. Quite las siguientes claves:
    * `fs.azure.account.key.<STORAGE_ACCOUNT_NAME>.blob.core.windows.net`
    * `fs.azure.account.keyprovider.<STORAGE_ACCOUNT_NAME>.blob.core.windows.net`

Después de quitar estas claves y guardar la configuración, debe reiniciar Oozie, Yarn, MapReduce2, HDFS y Hive uno a uno.

## <a name="known-issues"></a>Problemas conocidos

### <a name="storage-firewall"></a>Firewall de almacenamiento

Si decide proteger la cuenta de almacenamiento con las restricciones de **Firewalls y redes virtuales** en **Redes seleccionadas**, asegúrese de habilitar la excepción **Permitir servicios de Microsoft de confianza...** para que HDInsight pueda acceder a su cuenta de almacenamiento.`.`

### <a name="unable-to-access-storage-after-changing-key"></a>No se puede acceder a almacenamiento después de cambiar la clave

Si cambia la clave de una cuenta de almacenamiento, HDInsight ya no podrá acceder a dicha cuenta. HDInsight usa una copia en caché de clave del archivo core-site.xml para el clúster. Esta copia en caché debe actualizarse para que coincida con la nueva.

Volver a ejecutar la acción de script **no** actualizará la clave, ya que el script comprueba si ya existe una entrada para la cuenta de almacenamiento. Si ya existe una entrada, no realiza ningún cambio.

Para solucionar temporalmente este problema:  
1. Elimine la cuenta de almacenamiento.
1. Agregue la cuenta de almacenamiento.

> [!IMPORTANT]  
> No se admite la rotación de la clave de almacenamiento de la cuenta de almacenamiento principal asociada a un clúster.

## <a name="next-steps"></a>Pasos siguientes

En este documento aprendió a agregar más cuentas de almacenamiento a un clúster de HDInsight. Para más información sobre las acciones de script, consulte [Personalización de clústeres de HDInsight basados en Linux mediante la acción de script](hdinsight-hadoop-customize-cluster-linux.md).
