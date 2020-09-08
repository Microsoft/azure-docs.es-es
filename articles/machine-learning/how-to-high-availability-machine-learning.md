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
ms.openlocfilehash: 9b298e10d3eb95bcb0ef525eb973259a3ab1dbbb
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88852545"
---
# <a name="increase-azure-machine-learning-resiliency"></a>Aumento de la resistencia Azure Machine Learning

[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá cómo hacer que los recursos relacionados con Microsoft Azure Machine Learning sean más resistentes mediante configuraciones de alta disponibilidad. Puede configurar los servicios de Azure de los que depende Azure Machine Learning para tener una alta disponibilidad. En este artículo se identifican los servicios que se pueden configurar para alta disponibilidad y los vínculos a información adicional sobre la configuración de estos recursos.

> [!NOTE]
> Azure Machine Learning no ofrece una opción de recuperación ante desastres.

## <a name="understand-azure-services-for-azure-machine-learning"></a>Descripción de los servicios de Azure para Azure Machine Learning

Azure Machine Learning depende de varios servicios de Azure y tiene varias capas. Algunos de estos servicios se aprovisionan en su suscripción (cliente). El usuario es responsable de la configuración de alta disponibilidad de estos servicios. Otros se crean en una suscripción de Microsoft y son administrados por Microsoft. 

Los servicios de Azure incluyen:

* **Infraestructura de Azure Machine Learning**: Un entorno administrado por Microsoft para el área de trabajo de Azure Machine Learning.

* **Recursos asociados**: recursos aprovisionados en la suscripción durante la creación del área de trabajo de Azure Machine Learning. Estos recursos incluyen Azure Storage, Azure Key Vault, Azure Container Registry y Application Insights. El usuario es responsable de la configuración de alta disponibilidad de estos recursos.
  * El almacenamiento predeterminado contiene datos, como el modelo, los datos de registro de entrenamiento y el conjunto de datos.
  * Key Vault tiene credenciales para Azure Storage, Container Registry y almacenes de datos.
  * Container Registry tiene una imagen de Docker para entornos de aprendizaje e inferencia.
  * Application Insights se usa para la supervisión de Azure Machine Learning.

* **Recursos de proceso**: recursos creados después de la implementación del área de trabajo. Por ejemplo, puede crear una instancia de proceso o un clúster de proceso para entrenar un modelo de Machine Learning.
  * Instancia de proceso y clúster de proceso: Entornos de desarrollo de modelos administrado por Microsoft.
  * Otros recursos: Los recursos de computación que se pueden asociar a Azure Machine Learning, como Azure Kubernetes Service (AKS), Azure Databricks, Azure Container Instances y Azure HDInsight. El usuario es responsable de la configuración de alta disponibilidad de estos recursos.

* **Almacenes de datos adicionales**: Azure Machine Learning puede montar almacenes de datos adicionales, como Azure Storage, Azure Data Lake Storage y Azure SQL Database para los datos de entrenamiento.  Estos almacenes de datos se aprovisionan en la suscripción. El usuario es responsable de la configuración de las opciones de alta disponibilidad.

En la tabla siguiente se muestran cuáles son los servicios de Azure que administra Microsoft, los que administra el usuario y los que tienen alta disponibilidad de forma predeterminada.

| Servicio | Administrado por | Alta disponibilidad de forma predeterminada |
| ----- | ----- | ----- |
| **Infraestructura de Azure Machine Learning** | Microsoft | |
| **Recursos asociados** |
| Azure Storage | Los | |
| Key Vault | Los | ✓ |
| Container Registry | Los | |
| Application Insights | Los | N/D |
| **Recursos de proceso** |
| Instancia de proceso | Microsoft |  |
| Clúster de proceso | Microsoft |  |
| Otros recursos de proceso, como AKS, <br>Azure Databricks, Container Instances, HDInsight | Usted |  |
| **Almacenes de datos adicionales**, como Azure Storage, SQL Database,<br> Azure Database for PostgreSQL, Azure Database for MySQL, <br>Sistema de archivos de Azure Databricks | Usted | |

En el resto de este artículo se describen las acciones que el usuario tiene que realizar para que cada uno de estos servicios sea de alta disponibilidad.

## <a name="associated-resources"></a> Recursos asociados

> [!IMPORTANT]
> Azure Machine Learning no admite la conmutación por error de la cuenta de almacenamiento predeterminada mediante ninguno de estos almacenamientos: almacenamiento con redundancia geográfica (GRS), almacenamiento con redundancia de zona geográfica (GZRS), almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) o almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS).

Asegúrese de configurar los valores de alta disponibilidad de cada recurso mediante la referencia a la siguiente documentación:

* **Azure Storage**: Para configurar la opción de alta disponibilidad, consulte [Redundancia de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-redundancy).
* **Key Vault**: Key Vault proporciona alta disponibilidad de forma predeterminada y no requiere ninguna acción del usuario.  Consulte [Redundancia y disponibilidad de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/disaster-recovery-guidance).
* **Container Registry** (Registro de contenedor): Elija la opción del registro Premium para la replicación geográfica. Consulte [Replicación geográfica en Azure Container Registry](https://docs.microsoft.com/azure/container-registry/container-registry-geo-replication).
* **Application Insights**: Application Insights no proporciona configuraciones de alta disponibilidad. Para ajustar el período de retención de datos y los detalles consulte [Recopilación, retención y almacenamiento de datos en Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/data-retention-privacy#how-long-is-the-data-kept).

## <a name="compute-resources"></a>Recursos de proceso

Asegúrese de configurar los valores de alta disponibilidad de cada recurso mediante la referencia a la siguiente documentación:

* **Azure Kubernetes Service**: consulte [Procedimientos recomendados para continuidad empresarial y recuperación ante desastres en Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/operator-best-practices-multi-region) y [Creación de un clúster de Azure Kubernetes Service (AKS) que use zonas de disponibilidad](https://docs.microsoft.com/azure/aks/availability-zones). Si el clúster de AKS se creó en Azure Machine Learning Studio, el SDK o la CLI, no se admite alta disponibilidad entre regiones.
* **Azure Databricks**: consulte [Recuperación ante desastres regional para clústeres de Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/howto-regional-disaster-recovery).
* **Container Instances**. Un orquestador es responsable de la conmutación por error. Consulte [Azure Container Instances y orquestadores de contenedores](https://docs.microsoft.com/azure/container-instances/container-instances-orchestrator-relationship).
* **HDInsight**: consulte [Servicios de alta disponibilidad admitidos en Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-high-availability-components).

## <a name="additional-data-stores"></a>Almacenes de datos adicionales

Asegúrese de configurar los valores de alta disponibilidad de cada recurso mediante la referencia a la siguiente documentación:

* **Contenedor de blobs de Azure/Azure Files/Data Lake Storage Gen2**: igual que el almacenamiento predeterminado.
* **Data Lake Storage Gen1**: Consulte [Guía de alta disponibilidad y recuperación ante desastres para Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-disaster-recovery-guidance).
* **SQL Database**: Consulte [Alta disponibilidad para Azure SQL Database e Instancia administrada de SQL](https://docs.microsoft.com/azure/sql-database/sql-database-high-availability).
* **Azure Database for PostgreSQL**: consulte [Conceptos de alta disponibilidad en Azure Database for PostgreSQL: servidor único](https://docs.microsoft.com/azure/postgresql/concepts-high-availability).
* **Azure Database for MySQL**: consulte [Información acerca de la continuidad empresarial en Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-business-continuity).
* **Sistema de archivos de Azure Databricks**: consulte [Recuperación ante desastres regional para clústeres de Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/howto-regional-disaster-recovery).

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

Si proporciona su propia clave administrada por el cliente para implementar el área de trabajo de Azure Machine Learning, Azure Cosmos DB también se aprovisiona dentro de la suscripción. En ese caso, es responsable de configurar la configuración de alta disponibilidad. Consulte [Alta disponibilidad con Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/high-availability).

## <a name="next-steps"></a>Pasos siguientes

Para implementar Azure Machine Learning con los recursos asociados con la configuración de alta disponibilidad, use una [plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-advanced).
