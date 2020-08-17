---
title: Aumento de la resistencia
titleSuffix: Azure Machine Learning
description: Aprenda cómo hacer que los recursos relacionados con Azure Machine Learning sean más resistentes a las interrupciones mediante una configuración de alta disponibilidad.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jhirono
author: jhirono
ms.reviewer: larryfr
ms.date: 07/16/2020
ms.openlocfilehash: 4f2bf239e1157f5c927c4449857ad5f7793ccb49
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095335"
---
# <a name="increase-the-resiliency-of-azure-machine-learning"></a>Aumento de la resistencia de Azure Machine Learning

[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Aprenda cómo hacer que los recursos relacionados con Azure Machine Learning sean más resistentes mediante configuraciones de alta disponibilidad. Los servicios de Azure de los que depende Azure Machine Learning pueden configurarse para tener una alta disponibilidad. En este artículo se proporciona información sobre los servicios que se pueden configurar para una alta disponibilidad, junto con vínculos a información sobre la configuración de estos recursos.

> [!NOTE]
> Azure Machine Learning no ofrece una opción de recuperación ante desastres.

## <a name="understand-azure-services-for-azure-machine-learning"></a>Descripción de los servicios de Azure para Azure Machine Learning

Azure Machine Learning depende de varios servicios de Azure y tiene varias capas. Algunos de ellos se aprovisionan en la suscripción del cliente. El usuario es responsable de la configuración de alta disponibilidad de estos servicios. Otros se crean en una suscripción de Microsoft y son administrados por dicha empresa.

* **Infraestructura de Azure Machine Learning**: entorno administrado por Microsoft para el área de trabajo de Azure Machine Learning.

* **Recursos asociados**: recursos aprovisionados en la suscripción durante la creación del área de trabajo de Azure Machine Learning. Se incluyen Azure Storage, Azure Key Vault, Azure Container Registry (ACR) y App Insights. El usuario es responsable de la configuración de alta disponibilidad de estos recursos.
  * El almacenamiento predeterminado contiene datos, como el modelo, los datos de registro de entrenamiento y el conjunto de datos.
  * Key Vault incluye credenciales para el almacenamiento, ACR y los almacenes de datos.
  * ACR cuenta con una imagen de Docker para el entorno de entrenamiento e inferencia.
  * App Insights se usa para la supervisión de Azure Machine Learning.

* **Recursos de proceso**: recursos creados después de la implementación del área de trabajo. Por ejemplo, puede crear una instancia de proceso o un clúster de proceso para entrenar un modelo de aprendizaje automático.
  * Instancia de proceso y clúster de proceso: entorno de desarrollo de modelos administrado por Microsoft.
  * Otros recursos: Son los recursos informáticos que se pueden asociar a Azure Machine Learning, como Azure Kubernetes Service (AKS), Azure Databricks, Azure Container Instances (ACI) y HDInsight. El usuario es responsable de la configuración de alta disponibilidad de tales recursos.

* **Almacenes de datos adicionales**: Azure Machine Learning puede montar almacenes de datos adicionales, como Azure Storage, Azure Data Lake Storage y Azure SQL Database para los datos de entrenamiento.  Estos almacenes están dentro de la suscripción y el usuario es responsable de la configuración de alta disponibilidad.

En la tabla siguiente se muestran los servicios que administra Microsoft, los que administra el usuario y los que tienen alta disponibilidad de forma predeterminada:

| Servicio | Administrado por | Alta disponibilidad de forma predeterminada |
| ----- | ----- | ----- |
| **Infraestructura de Azure Machine Learning** | Microsoft | |
| **Recursos asociados** |
| Azure Storage | Los | |
| Azure Key Vault | Los | ✓ |
| Azure Container Registry | Los | |
| Application Insights | Los | N/D |
| **Recursos de proceso** |
| Instancia de proceso | Microsoft |  |
| Clúster de proceso | Microsoft |  |
| Otros recursos, como Azure Kubernetes Service, <br>Azure Databricks, Azure Container Instance, Azure HDInsight | Los |  |
| **Almacenes de datos adicionales**, como Azure Storage, Azure SQL Database,<br> Azure Database for PostgreSQL, Azure Database for MySQL, <br>sistema de archivos de Azure Databricks | Los | |

Utilice la información del resto de este documento para conocer las acciones necesarias para dotar de alta disponibilidad a cada uno de estos servicios.

## <a name="associated-resources"></a>Recursos asociados

> [!IMPORTANT]
> Azure Machine Learning no admite la conmutación por error de la cuenta de almacenamiento predeterminada mediante ninguno de estos almacenamientos: almacenamiento con redundancia geográfica (GRS), almacenamiento con redundancia de zona geográfica (GZRS), almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS).

Use la siguiente documentación para comprobar la configuración de alta disponibilidad de cada recurso.

* **Azure Storage**: para configurar la opción de alta disponibilidad, consulte [Redundancia de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-redundancy).
* **Azure Key Vault**: proporciona un servicio de alta disponibilidad predeterminada y no se requiere ninguna acción del usuario.  Consulte [Redundancia y disponibilidad de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/disaster-recovery-guidance).
* **Azure Container Registry**: elija SKU Premium para la replicación geográfica. Consulte [Replicación geográfica en Azure Container Registry](https://docs.microsoft.com/azure/container-registry/container-registry-geo-replication).
* **Application Insights**: no proporciona configuración de alta disponibilidad. Puede ajustar el período de retención de datos y los detalles en [Recopilación, retención y almacenamiento de datos en Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/data-retention-privacy#how-long-is-the-data-kept).

## <a name="compute-resources"></a>Recursos de proceso

Use la siguiente documentación para comprobar la configuración de alta disponibilidad de cada recurso.

* **Azure Kubernetes Service**: consulte [Procedimientos recomendados para continuidad empresarial y recuperación ante desastres en Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/operator-best-practices-multi-region) y [Creación de un clúster de Azure Kubernetes Service (AKS) que use zonas de disponibilidad](https://docs.microsoft.com/azure/aks/availability-zones). Si el clúster de AKS se creó en Azure Machine Learning (con Studio, el SDK o la CLI de ML), no se admite alta disponibilidad entre regiones.
* **Azure Databricks**: consulte [Recuperación ante desastres regional para clústeres de Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/howto-regional-disaster-recovery).
* **Azure Container Instances**: el orquestador ACI es responsable de la conmutación por error. Consulte [Azure Container Instances y orquestadores de contenedores](https://docs.microsoft.com/azure/container-instances/container-instances-orchestrator-relationship).
* **Azure HDInsight**: consulte [Servicios de alta disponibilidad admitidos en Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-high-availability-components).

## <a name="additional-data-stores"></a>Almacenes de datos adicionales

Use la siguiente documentación para comprobar la configuración de alta disponibilidad de cada recurso.

* **Contenedor de blobs de Azure/recurso compartido de archivos de Azure/Azure Data Lake Gen2**: igual que el almacenamiento predeterminado.
* **Azure Data Lake Gen1**: consulte [Guía de recuperación ante desastres para datos de Azure Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-disaster-recovery-guidance).
* **Azure SQL Database**: Consulte [Alta disponibilidad y Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-high-availability).
* **Azure Database for PostgreSQL**: consulte [Conceptos de alta disponibilidad en Azure Database for PostgreSQL: servidor único](https://docs.microsoft.com/azure/postgresql/concepts-high-availability).
* **Azure Database for MySQL**: consulte [Información acerca de la continuidad empresarial en Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-business-continuity).
* **Sistema de archivos de Databricks**: consulte [Recuperación ante desastres regional para clústeres de Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/howto-regional-disaster-recovery).

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

Si proporciona su propia clave (clave administrada por el cliente) para implementar el área de trabajo de Azure Machine Learning, Cosmos DB también se aprovisiona dentro de la suscripción. En ese caso, el usuario es responsable de su alta disponibilidad. Consulte [Alta disponibilidad con Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/high-availability).

## <a name="next-steps"></a>Pasos siguientes

Para implementar Azure Machine Learning con los recursos asociados con la configuración de alta disponibilidad, use una [plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-advanced).