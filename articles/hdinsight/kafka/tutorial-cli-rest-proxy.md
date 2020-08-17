---
title: 'Tutorial: Creación de un clúster habilitado para el proxy REST de Apache Kafka en HDInsight con la CLI de Azure'
description: Obtenga información sobre cómo realizar operaciones de Apache Kafka mediante un proxy de REST de Kafka en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: tutorial
ms.date: 02/27/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 44951fc19f36bb6652caf79ded96484bcc4b38f1
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503147"
---
# <a name="tutorial-create-an-apache-kafka-rest-proxy-enabled-cluster-in-hdinsight-using-azure-cli"></a>Tutorial: Creación de un clúster habilitado para el proxy REST de Apache Kafka en HDInsight con la CLI de Azure

En este tutorial, aprenderá a crear un clúster de [habilitado para un proxy REST](./rest-proxy.md) de Apache Kafka en Azure HDInsight mediante la interfaz de la línea de comandos (CLI) de Azure. Azure HDInsight es un servicio administrado, de espectro completo y de análisis de código abierto para empresas. Apache Kafka es una plataforma de streaming distribuida y de código abierto. A menudo se usa como agente de mensajes, ya que proporciona una funcionalidad similar a una cola de mensajes de publicación o suscripción. El proxy REST de Kafka le permite interactuar con el clúster de Kafka mediante una [API REST](https://docs.microsoft.com/rest/api/hdinsight-kafka-rest-proxy/) a través de HTTP. La CLI de Azure es la experiencia de línea de comandos multiplataforma de Microsoft para administrar los recursos de Azure.

Solo los recursos dentro de la misma red virtual pueden tener acceso a la API de Apache Kafka. Puede acceder al clúster directamente mediante SSH. Para conectar otros servicios, redes o máquinas virtuales con Apache Kafka, primero debe crear una red virtual y, a continuación, crear los recursos dentro de la red. Para más información, consulte [Conexión a Apache Kafka con una red virtual](./apache-kafka-connect-vpn-gateway.md).

En este tutorial, aprenderá:

> [!div class="checklist"]
> * Requisitos previos para el proxy REST de Kafka
> * Creación de un clúster de Apache Kafka con la CLI de Azure

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* Una aplicación registrada con Azure AD. Las aplicaciones cliente que escriba para interactuar con el proxy de REST de Kafka usarán el identificador y el secreto de la aplicación para autenticarse en Azure. Para más información, consulte [Registro de una aplicación con la plataforma de identidad de Microsoft](../../active-directory/develop/quickstart-register-app.md).

* Un grupo de seguridad de Azure AD con la aplicación registrada como miembro. Este grupo de seguridad se usará para controlar qué aplicaciones pueden interactuar con el proxy de REST. Para obtener más información sobre cómo crear grupos de Azure AD, consulte [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

* CLI de Azure. Asegúrese de tener la versión 2.0.79, como mínimo. Consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

## <a name="create-an-apache-kafka-cluster"></a>Creación de un clúster de Apache Kafka

1. Inicie sesión en la suscripción de Azure.

    ```azurecli
    az login
    
    # If you have multiple subscriptions, set the one to use
    # az account set --subscription "SUBSCRIPTIONID"
    ```

1. Establezca las variables de entorno. En este tutorial, el uso de variables se basa en Bash. Se necesitarán ligeras variaciones con otros entornos.

    |Variable | Descripción |
    |---|---|
    |resourceGroupName|Reemplace RESOURCEGROUPNAME por el nombre del nuevo grupo de recursos.|
    |ubicación|Reemplace LOCATION por la región donde se creará el clúster. Para obtener una lista de ubicaciones válidas, use el comando `az account list-locations`|
    |clusterName|Reemplace CLUSTERNAME por un nombre único global para el nuevo clúster.|
    |StorageAccount|Reemplace STORAGEACCOUNTNAME por el nombre de la nueva cuenta de almacenamiento.|
    |httpPassword|Reemplace PASSWORD por la contraseña del inicio de sesión del clúster, **admin**.|
    |sshPassword|Reemplace PASSWORD por la contraseña para el nombre de usuario del shell seguro, **sshuser**.|
    |securityGroupName|Reemplace SECURITYGROUPNAME por el nombre del grupo de seguridad de AAD para el proxy REST de Kafka. La variable se pasará al parámetro `--kafka-client-group-name` para `az-hdinsight-create`.|
    |securityGroupID|Reemplace SECURITYGROUPID por el identificador del grupo de seguridad AAD del cliente para el proxy REST de Kafka. La variable se pasará al parámetro `--kafka-client-group-id` para `az-hdinsight-create`.|
    |storageContainer|Contenedor de almacenamiento que usará el clúster; déjelo tal cual en este tutorial. Esta variable se establecerá con el nombre del clúster.|
    |workernodeCount|Número de nodos de trabajo en el clúster; déjelo tal cual en este tutorial. Para garantizar una alta disponibilidad, Kafka requiere un mínimo de tres nodos de trabajo.|
    |clusterType|Tipo del clúster de HDInsight; déjelo tal cual en este tutorial.|
    |clusterVersion|Versión del clúster de HDInsight; déjela tal cual en este tutorial. El proxy REST de Kafka requiere la versión mínima de clúster 4.0.|
    |componentVersion|Versión de Kafka; déjela tal cual en este tutorial. El proxy REST de Kafka requiere la versión mínima de componente 2.1.|

    Actualice las variables con los valores deseados. Luego, escriba los comandos de la CLI para establecer las variables de entorno.

    ```azurecli
    export resourceGroupName=RESOURCEGROUPNAME
    export location=LOCATION
    export clusterName=CLUSTERNAME
    export storageAccount=STORAGEACCOUNTNAME
    export httpPassword='PASSWORD'
    export sshPassword='PASSWORD'
    export securityGroupName=SECURITYGROUPNAME
    export securityGroupID=SECURITYGROUPID

    export storageContainer=$(echo $clusterName | tr "[:upper:]" "[:lower:]")
    export workernodeCount=3
    export clusterType=kafka
    export clusterVersion=4.0
    export componentVersion=kafka=2.1
    ```

1. [Cree el grupo de recursos](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-create) con el comando siguiente:

    ```azurecli
     az group create \
        --location $location \
        --name $resourceGroupName
    ```

1. [Cree una cuenta de Azure Storage](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create) con el comando siguiente:

    ```azurecli
    # Note: kind BlobStorage is not available as the default storage account.
    az storage account create \
        --name $storageAccount \
        --resource-group $resourceGroupName \
        --https-only true \
        --kind StorageV2 \
        --location $location \
        --sku Standard_LRS
    ```

1. [Extraiga la clave principal](https://docs.microsoft.com/cli/azure/storage/account/keys?view=azure-cli-latest#az-storage-account-keys-list) de la cuenta de Azure Storage y almacénela en una variable con el comando siguiente:

    ```azurecli
    export storageAccountKey=$(az storage account keys list \
        --account-name $storageAccount \
        --resource-group $resourceGroupName \
        --query [0].value -o tsv)
    ```

1. [Cree un contenedor de Azure Storage](https://docs.microsoft.com/cli/azure/storage/container?view=azure-cli-latest#az-storage-container-create) con el comando siguiente:

    ```azurecli
    az storage container create \
        --name $storageContainer \
        --account-key $storageAccountKey \
        --account-name $storageAccount
    ```

1. [Creación del clúster de HDInsight](https://docs.microsoft.com/cli/azure/hdinsight?view=azure-cli-latest#az-hdinsight-create). Antes de escribir el comando, tenga en cuenta los siguientes parámetros:

    1. Parámetros necesarios para los clústeres de Kafka:

        |Parámetro | Descripción|
        |---|---|
        |--type|El valor debe ser **Kafka**.|
        |--workernode-data-disks-per-node|El número de discos de datos que se van a usar por nodo de trabajo. HDInsight Kafka solo se admite con discos de datos. En este tutorial se usa el valor **2**.|

    1. Parámetros necesarios para el proxy REST de Kafka:

        |Parámetro | Descripción|
        |---|---|
        |--kafka-management-node-size|El tamaño del nodo. En este tutorial se usa el valor **Standard_D4_v2**.|
        |--kafka-client-group-id|El identificador del grupo de seguridad AAD del cliente para el proxy REST de Kafka. El valor se pasa desde la variable **$securityGroupID**.|
        |--kafka-client-group-name|El nombre del grupo de seguridad AAD del cliente para el proxy REST de Kafka. El valor se pasa desde la variable **$securityGroupName**.|
        |--version|La versión del clúster de HDInsight debe ser la 4.0, como mínimo. El valor se pasa desde la variable **$clusterVersion**.|
        |--component-version|La versión de Kafka debe ser la 2.1, como mínimo. El valor se pasa desde la variable **$componentVersion**.|
    
        Si desea crear el clúster sin el proxy REST, elimine `--kafka-management-node-size`, `--kafka-client-group-id`y `--kafka-client-group-name` del comando `az hdinsight create`.

    1. Si tiene una red virtual existente, agregue los parámetros `--vnet-name` y `--subnet`, y sus valores.

    Escriba el comando siguiente para crear el clúster:

    ```azurecli
    az hdinsight create \
        --name $clusterName \
        --resource-group $resourceGroupName \
        --type $clusterType \
        --component-version $componentVersion \
        --http-password $httpPassword \
        --http-user admin \
        --location $location \
        --ssh-password $sshPassword \
        --ssh-user sshuser \
        --storage-account $storageAccount \
        --storage-account-key $storageAccountKey \
        --storage-container $storageContainer \
        --version $clusterVersion \
        --workernode-count $workernodeCount \
        --workernode-data-disks-per-node 2 \
        --kafka-management-node-size "Standard_D4_v2" \
        --kafka-client-group-id $securityGroupID \
        --kafka-client-group-name "$securityGroupName"
    ```

    El proceso de creación del clúster puede tardar varios minutos en completarse. de creación del clúster.

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de completar el artículo, puede eliminar el clúster. Con HDInsight, los datos se almacenan en Azure Storage, por lo que puede eliminar un clúster de forma segura cuando no se esté usando. También se le cobrará por un clúster de HDInsight aunque no se esté usando. Como en muchas ocasiones los cargos por el clúster son mucho más elevados que los cargos por el almacenamiento, desde el punto de vista económico tiene sentido eliminar clústeres cuando no se usen.

Escriba todos o algunos de los siguientes comandos para quitar recursos:

```azurecli
# Remove cluster
az hdinsight delete \
    --name $clusterName \
    --resource-group $resourceGroupName

# Remove storage container
az storage container delete \
    --account-name $storageAccount  \
    --name $storageContainer

# Remove storage account
az storage account delete \
    --name $storageAccount  \
    --resource-group $resourceGroupName

# Remove resource group
az group delete \
    --name $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado correctamente un clúster habilitado para el proxy REST de Apache Kafka en Azure HDInsight mediante la CLI de Azure, use el código de Python para interactuar con el proxy REST:

> [!div class="nextstepaction"]
> [Creación de la aplicación de ejemplo](./rest-proxy.md#client-application-sample)
