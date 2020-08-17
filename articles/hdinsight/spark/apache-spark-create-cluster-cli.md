---
title: 'Inicio rápido: Clústeres de Apache Spark con la CLI de Azure: Azure HDInsight'
description: En este inicio rápido se muestra cómo usar la CLI de Azure para crear un clúster de Apache Spark en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: quickstart
ms.date: 02/03/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9827c50d034b1c49700869c3274800c194c631a8
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499456"
---
# <a name="quickstart-create-apache-spark-cluster-in-azure-hdinsight-using-azure-cli"></a>Inicio rápido: Creación de un clúster de Apache Spark en Azure HDInsight mediante la CLI de Azure

En este inicio rápido, aprenderá a crear un clúster de Apache Spark en Azure HDInsight mediante la interfaz de la línea de comandos (CLI) de Azure. Azure HDInsight es un servicio administrado, de espectro completo y de análisis de código abierto para empresas. El marco de Apache Spark para HDInsight permite un análisis de datos y un procesamiento de clústeres rápidos mediante el procesamiento en memoria. La CLI de Azure es la experiencia de línea de comandos multiplataforma de Microsoft para administrar los recursos de Azure.

Si usa varios clústeres juntos, querrá crear una red virtual y, si está usando un clúster de Spark, también querrá usar Hive Warehouse Connector. Para más información, consulte [Planificación de una red virtual para Azure HDInsight](../hdinsight-plan-virtual-network-deployment.md) e [Integración de Apache Spark y Apache Hive con el conector de Hive Warehouse](../interactive-query/apache-hive-warehouse-connector.md).

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- La [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli), si no desea usar Azure Cloud Shell.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-an-apache-spark-cluster"></a>Creación de un clúster de Apache Spark

1. Inicie sesión en la suscripción de Azure. Si planea usar Azure Cloud Shell, seleccione **Probar** en la esquina superior derecha del siguiente bloque de código. De lo contrario, escriba el comando siguiente:

    ```azurecli-interactive
    az login

    # If you have multiple subscriptions, set the one to use
    # az account set --subscription "SUBSCRIPTIONID"
    ```

2. Establezca las variables de entorno. El uso de variables en este inicio rápido se basa en Bash. Se necesitarán ligeras variaciones con otros entornos. Reemplace CLUSTERNAME, contraseña, ubicación, RESOURCEGROUPNAME y STORAGEACCOUNTNAME en el fragmento de código siguiente por los valores deseados. Luego, escriba los comandos de la CLI para establecer las variables de entorno.

    ```azurecli-interactive
    export resourceGroupName=RESOURCEGROUPNAME
    export location=LOCATION
    export clusterName=CLUSTERNAME
    export AZURE_STORAGE_ACCOUNT=STORAGEACCOUNTNAME
    export httpCredential='PASSWORD'
    export sshCredentials='PASSWORD'
    
    export AZURE_STORAGE_CONTAINER=$clusterName
    export clusterSizeInNodes=1
    export clusterVersion=3.6
    export clusterType=spark
    export componentVersion=Spark=2.3
    ```

3. Cree el grupo de recursos con el comando siguiente:

    ```azurecli-interactive
    az group create \
        --location $location \
        --name $resourceGroupName
    ```

4. Cree una cuenta de almacenamiento de Azure con el comando siguiente:

    ```azurecli-interactive
    az storage account create \
        --name $AZURE_STORAGE_ACCOUNT \
        --resource-group $resourceGroupName \
        --https-only true \
        --kind StorageV2 \
        --location $location \
        --sku Standard_LRS
    ```

5. Extraiga la clave principal de la cuenta de almacenamiento de Azure y almacénela en una variable con el comando siguiente:

    ```azurecli-interactive
    export AZURE_STORAGE_KEY=$(az storage account keys list \
        --account-name $AZURE_STORAGE_ACCOUNT \
        --resource-group $resourceGroupName \
        --query [0].value -o tsv)
    ```

6. Cree un contenedor de almacenamiento de Azure con el comando siguiente:

    ```azurecli-interactive
    az storage container create \
        --name $AZURE_STORAGE_CONTAINER \
        --account-key $AZURE_STORAGE_KEY \
        --account-name $AZURE_STORAGE_ACCOUNT
    ```

7. Cree el clúster de Apache Spark con el comando siguiente:

    ```azurecli-interactive
    az hdinsight create \
        --name $clusterName \
        --resource-group $resourceGroupName \
        --type $clusterType \
        --component-version $componentVersion \
        --http-password $httpCredential \
        --http-user admin \
        --location $location \
        --workernode-count $clusterSizeInNodes \
        --ssh-password $sshCredentials \
        --ssh-user sshuser \
        --storage-account $AZURE_STORAGE_ACCOUNT \
        --storage-account-key $AZURE_STORAGE_KEY \
        --storage-container $AZURE_STORAGE_CONTAINER \
        --version $clusterVersion
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de completar el inicio rápido, puede ser conveniente eliminar el clúster. Con HDInsight, los datos se almacenan en Azure Storage, por lo que puede eliminar un clúster de forma segura cuando no se esté usando. Los clústeres de HDInsight se cobran aunque no se estén usando. Como en muchas ocasiones los cargos por el clúster son mucho más elevados que los cargos por el almacenamiento, desde el punto de vista económico tiene sentido eliminar clústeres cuando no se usen.

Escriba todos o algunos de los siguientes comandos para quitar recursos:

```azurecli-interactive
# Remove cluster
az hdinsight delete \
    --name $clusterName \
    --resource-group $resourceGroupName

# Remove storage container
az storage container delete \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --name $AZURE_STORAGE_CONTAINER

# Remove storage account
az storage account delete \
    --name $AZURE_STORAGE_ACCOUNT \
    --resource-group $resourceGroupName

# Remove resource group
az group delete \
    --name $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, aprendió a crear un clúster de Apache Kafka en Azure HDInsight con la CLI de Azure.  Pase al siguiente tutorial, donde aprenderá a usar un clúster de HDInsight para ejecutar consultas interactivas en datos de ejemplo.

> [!div class="nextstepaction"]
> [Ejecución de consultas interactivas en Apache Spark](./apache-spark-load-data-run-query.md)
