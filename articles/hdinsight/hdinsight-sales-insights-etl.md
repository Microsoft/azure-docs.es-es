---
title: 'Tutorial: Creación de una canalización de ETL de un extremo a otro para derivar información de ventas en Azure HDInsight'
description: Aprenda a crear canalizaciones de ETL con Azure HDInsight para obtener información sobre los datos de ventas mediante clústeres a petición de Spark y Power BI.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive
ms.date: 04/15/2020
ms.openlocfilehash: a0f081e0f8df00bbc99d2163fb54a2f15d92a159
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87006439"
---
# <a name="tutorial-create-an-end-to-end-data-pipeline-to-derive-sales-insights-in-azure-hdinsight"></a>Tutorial: Creación de una canalización de datos de un extremo a otro para derivar información de ventas en Azure HDInsight

En este tutorial compilará una canalización de datos integral que realice operaciones de extracción, transformación y carga de datos (ETL). La canalización usará clústeres de [Apache Spark](./spark/apache-spark-overview.md) y Apache Hive que se ejecutan en Azure HDInsight tanto para realizar consultas en los datos como para manipularlos. También usará tecnologías como Azure Data Lake Storage Gen2 para el almacenamiento de datos y Power BI para la visualización.

Esta canalización de datos combina los datos de varios almacenes, quita los datos no deseados, y anexa nuevos datos y los carga de nuevo en su almacenamiento para visualizar información empresarial. Consulte [Extracción, transformación y carga de datos (ETL) a escala](./hadoop/apache-hadoop-etl-at-scale.md) para más información sobre las canalizaciones ETL.

![Arquitectura de ETL](./media/hdinsight-sales-insights-etl/architecture.png)

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* CLI de Azure (2.2.0 como mínimo). Consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

* jq, un procesador JSON de línea de comandos.  Vea [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/).

* Un miembro del [rol integrado de Azure: propietario](../role-based-access-control/built-in-roles.md).

* Si usa PowerShell para desencadenar la canalización de Data Factory, necesitará el [módulo AZ](https://docs.microsoft.com/powershell/azure/).

* [Power BI Desktop](https://aka.ms/pbiSingleInstaller) para visualizar la información empresarial que aparece al final de este tutorial.

## <a name="create-resources"></a>Crear recursos

### <a name="clone-the-repository-with-scripts-and-data"></a>Clonación del repositorio con scripts y datos

1. Inicie sesión en la suscripción de Azure. Si va a usar Azure Cloud Shell, seleccione **Probar** en la esquina superior derecha del bloque de código. En caso contrario, escriba el siguiente comando:

    ```azurecli-interactive
    az login

    # If you have multiple subscriptions, set the one to use
    # az account set --subscription "SUBSCRIPTIONID"
    ```

1. Asegúrese de que es miembro del rol de Azure [propietario](../role-based-access-control/built-in-roles.md). Reemplace `user@contoso.com` por su cuenta y escriba el comando:

    ```azurecli
    az role assignment list \
    --assignee "user@contoso.com" \
    --role "Owner"
    ```

    Si no se devuelve ningún registro, significa que no es miembro y, por consiguiente, no podrá realizar este tutorial.

1. Descargue los datos y scripts de este tutorial del [repositorio de ETL de información de ventas de HDInsight](https://github.com/Azure-Samples/hdinsight-sales-insights-etl). Escriba el comando siguiente:

    ```bash
    git clone https://github.com/Azure-Samples/hdinsight-sales-insights-etl.git
    cd hdinsight-sales-insights-etl
    ```

1. Asegúrese de que se han creado `salesdata scripts templates`. Realice la comprobación con el comando siguiente:

   ```bash
   ls
   ```

### <a name="deploy-azure-resources-needed-for-the-pipeline"></a>Implementación de recursos de Azure necesarios para la canalización

1. Agregue permisos de ejecución para todos los scripts escribiendo:

    ```bash
    chmod +x scripts/*.sh
    ````

1. Establezca la variable para el grupo de recursos. Reemplace `RESOURCE_GROUP_NAME` por el nombre de un grupo de recursos nuevo o existente y, a continuación, escriba el comando:

    ```bash
    resourceGroup="RESOURCE_GROUP_NAME"
    ```

1. Ejecute el script. Reemplace `LOCATION` por el valor deseado y escriba el comando:

    ```bash
    ./scripts/resources.sh $resourceGroup LOCATION
    ```

    Si no está seguro de qué región especificar, puede recuperar una lista de regiones admitidas para la suscripción con el comando [az account list-locations](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-list-locations).

    El comando implementará los siguientes recursos:

    * Una cuenta de Azure Blob Storage. Esta cuenta contendrá los datos de ventas de la empresa.
    * Una cuenta de Azure Data Lake Storage Gen2. Esta cuenta servirá como cuenta de almacenamiento para ambos clústeres de HDInsight. Consulte [Integración de Azure HDInsight con Data Lake Storage Gen2](https://azure.microsoft.com/blog/azure-hdinsight-integration-with-data-lake-storage-gen-2-preview-acl-and-security-update/) para más información sobre HDInsight y Data Lake Storage Gen2.
    * Una identidad administrada asignada por el usuario. Esta cuenta proporciona a los clústeres de HDInsight acceso a la cuenta de Data Lake Storage Gen2.
    * Un clúster Apache Spark. Este clúster se usará para limpiar y transformar los datos sin procesar.
    * Un clúster de Apache Hive [Interactive Query](./interactive-query/apache-interactive-query-get-started.md). Este clúster permitirá consultar los datos de ventas y visualizarlos con Power BI.
    * Una red virtual de Azure compatible con las reglas del grupo de seguridad de red (NSG). Esta red virtual permite a los clústeres comunicarse y proteger sus comunicaciones.

La creación del clúster puede tardar unos 20 minutos.

La contraseña predeterminada para el acceso SSH a los clústeres es `Thisisapassword1`. Si desea cambiar la contraseña, vaya al archivo `./templates/resourcesparameters_remainder.json` y cámbiela para los parámetros `sparksshPassword`, `sparkClusterLoginPassword`, `llapClusterLoginPassword` y `llapsshPassword`.

### <a name="verify-deployment-and-collect-resource-information"></a>Comprobación de la implementación y recopilación de información de recursos

1. Si desea comprobar el estado de la implementación, vaya al grupo de recursos en Azure Portal. En **Configuración**, seleccione **Implementaciones** y, a continuación, su implementación. Aquí puede ver los recursos que se han implementado correctamente y los que todavía están en curso.

1. Para ver los nombres de los clústeres, escriba el siguiente comando:

    ```bash
    sparkClusterName=$(cat resourcesoutputs_remainder.json | jq -r '.properties.outputs.sparkClusterName.value')
    llapClusterName=$(cat resourcesoutputs_remainder.json | jq -r '.properties.outputs.llapClusterName.value')

    echo "Spark Cluster" $sparkClusterName
    echo "LLAP cluster" $llapClusterName
    ```

1. Para ver la cuenta de almacenamiento y la clave de acceso de Azure, escriba el siguiente comando:

    ```azurecli
    blobStorageName=$(cat resourcesoutputs_storage.json | jq -r '.properties.outputs.blobStorageName.value')

    blobKey=$(az storage account keys list \
        --account-name $blobStorageName \
        --resource-group $resourceGroup \
        --query [0].value -o tsv)

    echo $blobStorageName
    echo $blobKey
    ```

1. Para ver la cuenta de almacenamiento y la clave de acceso de Data Lake Storage Gen2, escriba el siguiente comando:

    ```azurecli
    ADLSGen2StorageName=$(cat resourcesoutputs_storage.json | jq -r '.properties.outputs.adlsGen2StorageName.value')

    adlsKey=$(az storage account keys list \
        --account-name $ADLSGen2StorageName \
        --resource-group $resourceGroup \
        --query [0].value -o tsv)

    echo $ADLSGen2StorageName
    echo $adlsKey
    ```

### <a name="create-a-data-factory"></a>Crear una factoría de datos

Azure Data Factory es una herramienta que ayuda a automatizar Azure Pipelines. No es la única forma de realizar estas tareas, pero es una excelente manera de automatizar los procesos. Consulte la [documentación de Azure Data Factory](https://azure.microsoft.com/services/data-factory/) para más información sobre Azure Data Factory.

Esta factoría de datos tendrá una canalización con dos actividades:

* La primera actividad copiará los datos de Azure Blob Storage en la cuenta de almacenamiento de Data Lake Storage Gen 2 para simular la ingesta de datos.
* La segunda actividad transformará los datos en el clúster de Spark. El script transforma los datos al quitar las columnas no deseadas. También anexa una nueva columna que calcula los ingresos que genera una transacción.

Para configurar una canalización de Azure Data Factory, ejecute el siguiente comando.  Todavía debería estar en el directorio `hdinsight-sales-insights-etl`.

```bash
blobStorageName=$(cat resourcesoutputs_storage.json | jq -r '.properties.outputs.blobStorageName.value')
ADLSGen2StorageName=$(cat resourcesoutputs_storage.json | jq -r '.properties.outputs.adlsGen2StorageName.value')

./scripts/adf.sh $resourceGroup $ADLSGen2StorageName $blobStorageName
```

El script hace lo siguiente:

1. Crea una entidad de servicio con permisos de `Storage Blob Data Contributor` en la cuenta de almacenamiento de Data Lake Storage Gen2.
1. Obtiene un token de autenticación para autorizar solicitudes POST a la [API REST del sistema de archivos de Data Lake Storage Gen2](https://docs.microsoft.com/rest/api/storageservices/datalakestoragegen2/filesystem/create).
1. Rellena el nombre de la cuenta de almacenamiento de Data Lake Storage Gen2 en los archivos `sparktransform.py` y `query.hql`.
1. Obtiene las claves de almacenamiento para las cuentas de Data Lake Storage Gen2 y Blob Storage.
1. Crea otra implementación de recursos para crear una canalización de Azure Data Factory, con sus actividades y servicios vinculados asociados. Pasa las claves de almacenamiento como parámetros al archivo de plantilla para que los servicios vinculados puedan acceder a las cuentas de almacenamiento correctamente.

## <a name="run-the-data-pipeline"></a>Ejecución de la canalización de datos

### <a name="trigger-the-data-factory-activities"></a>Desencadenador de las actividades de Data Factory

La primera actividad de la canalización de Data Factory que ha creado mueve los datos de Blob Storage a Data Lake Storage Gen2. La segunda actividad aplica las transformaciones de Spark a los datos y guarda los archivos CSV transformados en una nueva ubicación. La canalización puede tardar unos minutos en completarse.

Para recuperar el nombre de Data Factory, escriba el siguiente comando:

```azurecli
cat resourcesoutputs_adf.json | jq -r '.properties.outputs.factoryName.value'
```

Para desencadenar la canalización, puede:

* Desencadene la canalización de Data Factory en PowerShell. Reemplace `RESOURCEGROUP` y `DataFactoryName` por los valores adecuados y, a continuación, ejecute los siguientes comandos:

    ```powershell
    # If you have multiple subscriptions, set the one to use
    # Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

    $resourceGroup="RESOURCEGROUP"
    $dataFactory="DataFactoryName"

    $pipeline =Invoke-AzDataFactoryV2Pipeline `
        -ResourceGroupName $resourceGroup `
        -DataFactory $dataFactory `
        -PipelineName "IngestAndTransform"

    Get-AzDataFactoryV2PipelineRun `
        -ResourceGroupName $resourceGroup  `
        -DataFactoryName $dataFactory `
        -PipelineRunId $pipeline
    ```

    Vuelva a ejecutar `Get-AzDataFactoryV2PipelineRun` según sea necesario para supervisar el progreso.

    Or

* Abra la factoría de datos y seleccione **Author & Monitor** (Creación y supervisión). Desencadene la canalización `IngestAndTransform` desde el portal. Consulte [Creación de clústeres de Apache Hadoop a petición en HDInsight mediante Azure Data Factory](hdinsight-hadoop-create-linux-clusters-adf.md#trigger-a-pipeline) para más información sobre cómo desencadenar canalizaciones mediante el portal.

Para comprobar que se ha ejecutado la canalización, puede realizar uno de los pasos siguientes:

* Vaya a la sección **Monitor** (Supervisar) de la factoría de datos mediante el portal.
* En Explorador de Azure Storage, vaya a la cuenta de almacenamiento de Data Lake Storage Gen 2. Vaya al sistema de archivos `files` y a la carpeta `transformed`, y compruebe su contenido para ver si la canalización se ha realizado correctamente.

Consulte [este artículo sobre el uso de Jupyter Notebook](/azure/hdinsight/spark/apache-spark-load-data-run-query) para otras maneras de transformar datos con HDInsight.

### <a name="create-a-table-on-the-interactive-query-cluster-to-view-data-on-power-bi"></a>Creación de una tabla en el clúster de Interactive Query para ver datos en Power BI

1. Copie el archivo `query.hql` en el clúster de LLAP mediante SCP. Escriba el comando:

    ```bash
    llapClusterName=$(cat resourcesoutputs_remainder.json | jq -r '.properties.outputs.llapClusterName.value')
    scp scripts/query.hql sshuser@$llapClusterName-ssh.azurehdinsight.net:/home/sshuser/
    ```

    Recordatorio: La contraseña predeterminada es `Thisisapassword1`.

1. Use SSH para acceder al clúster de LLAP. Escriba el comando:

    ```bash
    ssh sshuser@$llapClusterName-ssh.azurehdinsight.net
    ```

1. Use el siguiente comando para ejecutar el script:

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f query.hql
    ```

    Este script creará una tabla administrada en el clúster de Interactive Query a la que puede acceder desde Power BI.

### <a name="create-a-power-bi-dashboard-from-sales-data"></a>Creación de un panel de Power BI a partir de datos de ventas

1. Abra Power BI Desktop.

1. En el menú, vaya a **Obtenga datos** > **Más...**  > **Azure** > **HDInsight Interactive Query**.

1. Seleccione **Conectar**.

1. En el cuadro de diálogo **HDInsight Interactive Query**:
    1. En el cuadro de texto **Servidor**, escriba el nombre del clúster de LLAP en el formato de `https://LLAPCLUSTERNAME.azurehdinsight.net`.
    1. En el cuadro de texto **base de datos**, escriba `default`.
    1. Seleccione **Aceptar**.

1. En el cuadro de diálogo **AzureHive**:
    1. En el cuadro de texto **Nombre de usuario**, escriba `admin`.
    1. En el cuadro de texto **Contraseña**, escriba `Thisisapassword1`.
    1. Seleccione **Conectar**.

1. En **Navegador**, seleccione `sales` o `sales_raw` para obtener una vista previa de los datos. Una vez cargados los datos, puede experimentar con el panel que desee crear. Consulte los siguientes vínculos para ver una introducción a los paneles de Power BI:

* [Introducción a los paneles para diseñadores de Power BI](https://docs.microsoft.com/power-bi/service-dashboards)
* [Tutorial: introducción al servicio Power BI](https://docs.microsoft.com/power-bi/service-get-started)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine todos los recursos asociados mediante el comando siguiente para que no se le facturen.

1. Para eliminar el grupo de recursos, escriba el comando:

    ```azurecli
    az group delete -n $resourceGroup
    ```

1. Para eliminar la entidad de servicio, escriba los comandos:

    ```azurecli
    servicePrincipal=$(cat serviceprincipal.json | jq -r '.name')
    az ad sp delete --id $servicePrincipal
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Extracción, transformación y carga de datos (ETL) a escala](./hadoop/apache-hadoop-etl-at-scale.md)
