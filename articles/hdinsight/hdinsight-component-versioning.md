---
title: 'Componentes y versiones de Apache Hadoop: Azure HDInsight'
description: Conozca los componentes y las versiones de Apache Hadoop disponibles en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 08/13/2020
ms.openlocfilehash: 5c3586e9ba405cb5a7338b8f7f22858a238b9271
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717258"
---
# <a name="apache-components-and-versions-available-for-azure-hdinsight"></a>Componentes y versiones de Apache disponibles para Azure HDInsight

En este artículo, conozca los componentes y las versiones del entorno [Apache Hadoop](https://hadoop.apache.org/) que se incluyen en Microsoft Azure HDInsight, así como Enterprise Security Package. Además, obtenga información sobre cómo comprobar las versiones de los componentes de Hadoop en HDInsight.

## <a name="apache-components-available-with-different-hdinsight-versions"></a>Componentes de Apache disponibles con diferentes versiones de HDInsight

HDInsight de Azure es compatible con varias versiones de clústeres de Hadoop que se pueden implementar en cualquier momento. Desde el 4 de abril de 2017, la versión de clúster predeterminada que se usa en Azure HDInsight es la 3.6.

En la tabla siguiente se enumeran las versiones de componente asociadas a las versiones de clúster de HDInsight.

> [!NOTE]
> La versión predeterminada del servicio HDInsight puede cambiar sin previo aviso. Si tiene una dependencia de la versión, especifique la versión de HDInsight al crear clústeres con el SDK de .NET con Azure PowerShell y la CLI de Azure clásica.

| Componente              | HDInsight 4.0 | HDInsight 3.6 (predeterminado)     |
|------------------------|---------------|-----------------------------|
| Apache Hadoop y YARN | 3.1.1         | 2.7.3                       |
| Apache Tez             | 0.9.1         | 0.7.0                       |
| Apache Pig             | 0.16.0        | 0.16.0                      |
| Apache Hive            | 3.1.0         | 1.2.1 (2.1.0 en ESP Interactive Query) |
| Apache Tez Hive2       | -             | 0.8.4                       |
| Apache Ranger          | 1.1.0         | 0.7.0                       |
| HBase Apache           | 2.1.6         | 1.1.2                       |
| Apache Sqoop           | 1.4.7         | 1.4.6                       |
| Apache Oozie           | 4.3.1         | 4.2.0                       |
| Apache Zookeeper       | 3.4.6         | 3.4.6                       |
| Apache Storm           | -             | 1.1.0                       |
| Apache Mahout          | -             | 0.9.0+                      |
| Apache Phoenix         | 5             | 4.7.0                       |
| Spark de Apache           | 2.4.4         | 2.3.2                      |
| Apache Livy            | 0.5           | 0.4                        |
| Apache Kafka           | 2.1.1         | 1.1                         |
| Apache Ambari          | 2.7.0         | 2.6.0                       |
| Apache Zeppelin        | 0.8.0         | 0.7.3                       |
| Mono                   | 4.2.1         | 4.2.1                       |

> [!NOTE]
> HDInsight 4.0: Spark 2.4 y Kafka 2.1 son totalmente compatibles. Pero los tipos de clúster Spark 2.3 y Kafka 1.1 no tienen servicio. HDInsight 3.6: Spark 2.3 y Kafka 1.1 son totalmente compatibles.  

## <a name="check-for-current-apache-component-version-information"></a>Comprobación de la información de la versión de los componentes actuales de Apache

Las versiones de los componentes del entorno de Hadoop asociadas a las versiones de los clústeres de HDInsight pueden cambiar en futuras actualizaciones de HDInsight. Para comprobar los componentes de Hadoop disponibles y comprobar las versiones que se están usando para un clúster, use la API REST de Ambari. El comando **GetComponentInformation** recupera información sobre componentes de servicio. Para más información, consulte la [documentación de Apache Ambari](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md).

### <a name="release-notes"></a>Notas de la versión

Consulte las [Notas de la versión de HDInsight](hdinsight-release-notes.md) para conocer otras notas de las últimas versiones de HDInsight.

## <a name="supported-hdinsight-versions"></a>Versiones compatibles de HDInsight

### <a name="support-expiration-and-retirement-for-hdinsight-versions"></a>Expiración de la compatibilidad y retirada de versiones de HDInsight

La **Expiración del soporte técnico** significa que Microsoft ya no proporcionará soporte técnico para la versión especificada de HDInsight. Y ya no está disponible mediante Azure Portal para la creación del clúster. Sin embargo, estas versiones todavía se pueden crear mediante la CLI de Azure o los diversos SDK.

La **retirada** de una versión de HDInsight significa que los clústeres existentes seguirán ejecutándose tal cual. Los nuevos clústeres de esta versión no se pueden crear por ningún medio (ni con la CLI ni con los SDK). Otras características del plano de control, como el escalado manual y la escalabilidad automática, puede que tampoco funcionen después de la retirada de la versión. El soporte técnico no está disponible para las versiones retiradas.

Las tablas siguientes enumeran las versiones de HDInsight. Si se conocen, también se proporcionan las fechas de expiración y retirada.

### <a name="available-versions"></a>Versiones disponibles

En la tabla siguiente se enumeran las versiones de HDInsight que están disponibles en Azure Portal y otros métodos de implementación como PowerShell y SDK de .NET.

| Versión de HDInsight | SISTEMA OPERATIVO DE LA MÁQUINA VIRTUAL | Fecha de la versión | Fecha de expiración del soporte técnico | Fecha de retirada | Alta disponibilidad |  Disponibilidad en Azure Portal |
| --- | --- | --- | --- | --- | --- | --- |
| HDInsight 4.0 |Ubuntu 16.0.4 LTS |24 de septiembre de 2018 | | |Sí |Sí |
| HDInsight 3.6 |Ubuntu 16.0.4 LTS |4 de abril de 2017      | *31 de junio de 2021 |31 de junio de 2021 |Sí |Sí |

*Estamos ampliando el período de soporte técnico para determinados tipos de clúster de HDInsight 3.6

| Tipo de clúster                    | Versión del marco | Expiración del soporte técnico actual        | Nueva fecha de expiración del soporte técnico |
|---------------------------------|-------------------|-----------------------------------|-----------------------------|
| HDInsight 3.6 Hadoop            | 2.7.3             | 31 de diciembre de 2020                      | 30 de junio de 2021               |
| HDInsight 3.6 Spark             | 2.3               | 31 de diciembre de 2020                      | 30 de junio de 2021               |
| HDInsight 3.6 Spark             | 2,2               | Retirado el 30 de junio de 2020          |                             |
| HDInsight 3.6 Spark             | 2.1               | Retirado el 30 de junio de 2020          |                             |
| HDInsight 3.6 Kafka             | 1,1               | 31 de diciembre de 2020                      | 30 de junio de 2021               |
| HDInsight 3.6 Kafka             | 1,0               | Retirado el 30 de junio de 2020         |                             |
| HDInsight 3.6 HBase             | 1,1               | 31 de diciembre de 2020                      | 30 de junio de 2021               |
| HDInsight 3.6 Interactive Query | 2.1               | 31 de diciembre de 2020                      | 30 de junio de 2021               |
| HDInsight 3.6 Storm             | 1,1               | 31 de diciembre de 2020                      | 30 de junio de 2021               |
| HDInsight 3.6 ML Services      | 9.3               | 31 de diciembre de 2020                      | 31 de diciembre de 2020                |

> [!NOTE]
> Cuando expira la compatibilidad de una versión de clúster, es posible que deje de estar disponible en Azure Portal. En algunos casos, las versiones del clúster seguirán estando disponibles mediante el parámetro **Version** del comando [New-AzHDInsightCluster](https://docs.microsoft.com/powershell/module/az.hdinsight/new-azhdinsightcluster) de Windows PowerShell y en el SDK de .NET hasta la fecha de retirada de la versión.

### <a name="retired-versions"></a>Versiones retiradas

En la tabla siguiente se enumeran las versiones de HDInsight que no están disponibles en Azure Portal.

| Versión de HDInsight | Versión de HDP | SISTEMA OPERATIVO DE LA MÁQUINA VIRTUAL | Fecha de la versión | Fecha de expiración del soporte técnico | Fecha de retirada | Alta disponibilidad |  Disponibilidad en Azure Portal |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HDInsight 3.5 |HDP 2.5 |Ubuntu 16.0.4 LTS |30 de septiembre de 2016 |5 de septiembre de 2017 |28 de junio de 2018 |Sí |No |
| HDInsight 3.4 |HDP 2.4 |Ubuntu 14.0.4 LTS |29 de marzo de 2016 |29 de diciembre de 2016 |9 de enero de 2018 |Sí |No |
| HDInsight 3.3 |HDP 2.3 |Windows Server 2012 R2 |2 de diciembre de 2015 |27 de junio de 2016 |31 de julio de 2018 |Sí |No |
| HDInsight 3.3 |HDP 2.3 |Ubuntu 14.0.4 LTS |2 de diciembre de 2015 |27 de junio de 2016 |31 de julio de 2017 |Sí |No |
| HDInsight 3.2 |HDP 2.2 |Ubuntu 12.04 LTS o Windows Server 2012 R2 |18 de febrero de 2015 |1 de marzo de 2016 |1 de abril de 2017 |Sí |No |
| HDInsight 3.1 |HDP 2,1 |Windows Server 2012 R2 |24 de junio de 2014 |18 de mayo de 2015 |30 de junio de 2016 |Sí |No |
| HDInsight 3.0 |HDP 2,0 |Windows Server 2012 R2 |11 de febrero de 2014 |17 de septiembre de 2014 |30 de junio de 2015 |Sí |No |
| HDInsight 2.1 |HDP 1,3 |Windows Server 2012 R2 |28 de octubre de 2013 |12 de mayo de 2014 |31 de mayo de 2015 |Sí |No |
| HDInsight 1.6 |HDP 1.1 | |28 de octubre de 2013 |26 de abril de 2014 |31 de mayo de 2015 |No |No |

> [!NOTE]
> Los clústeres de alta disponibilidad con dos nodos principales se implementan de forma predeterminada para los clústeres de HDInsight 2.1 y versiones posteriores. No están disponibles para los clústeres de HDInsight versión 1.6.

## <a name="service-level-agreement-for-hdinsight-cluster-versions"></a>Contrato de nivel de servicio para las versiones de clúster de HDInsight

El contrato de nivel de servicio se define en términos de _ventana de soporte técnico_. Una ventana de soporte técnico se refiere al período durante el cual la versión de HDInsight es compatible con el Soporte técnico y el Servicio al cliente de Microsoft. Si la versión tiene una _fecha de expiración de soporte técnico_ que ya ha pasado, el clúster de HDInsight está fuera de la ventana de soporte técnico. La expiración del soporte técnico de la versión X de HDInsight (después de que haya disponible una versión más reciente X + 1) es la más reciente:

- **Fórmula 1:** agregue 180 días a la fecha en la que se lanzó la versión X del clúster de HDInsight.
- **Fórmula 2:** Agregue 90 días a la fecha en la que la versión del clúster de HDInsight X + 1 se encuentra disponible en Azure Portal.

La _fecha de retirada_ es la fecha tras la cual no se puede crear la versión del clúster en HDInsight. A partir del 31 de julio de 2017, no se puede cambiar el tamaño de un clúster de HDInsight después de su fecha de retirada.

## <a name="default-node-configuration-and-virtual-machine-sizes-for-clusters"></a>Configuración de nodo predeterminada y tamaños de máquina virtual para clústeres

Para obtener más información sobre qué SKU de máquina virtual se deben seleccionar para su clúster, consulte [Detalles de configuración de clúster de Azure HDInsight](hdinsight-supported-node-configuration.md).

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de clúster para Apache Hadoop, Spark, etc. en HDInsight](hdinsight-hadoop-provision-linux-clusters.md)
- [Trabajo en Apache Hadoop en HDInsight desde un equipo Windows](hdinsight-hadoop-windows-tools.md)
- [Notas de la versión de Hortonworks asociadas a las versiones de HDInsight](./hortonworks-release-notes.md)
- [Paquete de seguridad de la empresa](./enterprise-security-package.md)
