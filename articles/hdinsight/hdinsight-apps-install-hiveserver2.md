---
title: Escalado de HiveServer2 en Azure HDInsight
description: Escale horizontalmente HiveServer2 en clústeres de Azure HDInsight con nodos perimetrales para aumentar la tolerancia a errores y la disponibilidad.
services: hdinsight
ms.service: hdinsight
ms.topic: conceptual
ms.reviewer: hrasheed-msft
ms.author: kecheung
author: kcheeeung
ms.date: 08/12/2020
ms.openlocfilehash: d0a0df4791492c1c9f0d600630d723024c46c1b8
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226967"
---
# <a name="scale-hiveserver2-on-azure-hdinsight-clusters-for-high-availability"></a>Escalado de HiveServer2 en clústeres de Azure HDInsight para alta disponibilidad

Aprenda a implementar un elemento HiveServer2 adicional en el clúster para aumentar la disponibilidad y la distribución de la carga. Cuando no se desea aumentar el tamaño de nodo principal, también puede usar nodos perimetrales para implementar HiveServer2. 

> [!NOTE]
> En función del uso, el aumento del número de HiveServer2 puede aumentar el número de conexiones a la metastore de Hive. Asegúrese también de que su base de datos de Azure SQL tenga el tamaño adecuado.

## <a name="prerequisites"></a>Prerrequisitos

Para usar esta guía, debe comprender el siguiente artículo:
- [Uso de nodos perimetrales vacíos en clústeres de Apache Hadoop en HDInsight](hdinsight-apps-use-edge-node.md)

## <a name="install-hiveserver2"></a>Instalación de HiveServer2

En esta sección, instalará un componente HiveServer2 adicional en los hosts de destino.

1. Abra Ambari en el explorador y haga clic en el host de destino.

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-a.png" alt-text="Menú Hosts de Ambari.":::

2. Haga clic en el botón de agregar y haga clic en HiveServer2.

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-b.png" alt-text="Agregue el panel de HiveServer2 del host.":::

3. Confirme y se ejecutará el proceso. Repita 1-3 para todos los hosts deseados.

4. Cuando haya terminado de instalar, reinicie todos los servicios con configuraciones obsoletas e inicie HiveServer2.

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-c.png" alt-text="Inicie el panel de HiveServer2.":::

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a instalar HiveServer2 en el clúster. Para obtener más información sobre los nodos perimetrales y las aplicaciones, consulte los siguientes artículos:

* [Instalación de nodo perimetral](hdinsight-apps-use-edge-node.md): aprenda a instalar un nodo perimetral en el clúster de HDInsight.
* [Instalación de aplicaciones de HDInsight](hdinsight-apps-install-applications.md): aprenda a instalar una aplicación de HDInsight en sus clústeres.
* [Límites de conexión de DTU de Azure SQL](../azure-sql/database/resource-limits-dtu-single-databases.md): obtenga información sobre los límites de base de datos de Azure SQL mediante DTU.
* [Límites de conexión de núcleo virtual de Azure SQL](../azure-sql/database/resource-limits-vcore-elastic-pools.md): obtenga información sobre los límites de la base de datos de Azure SQL mediante núcleos virtuales.
