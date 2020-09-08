---
title: Configuración de clústeres en HDInsight con Apache Hadoop, Apache Spark, Apache Kafka, etc.
description: Configure clústeres de Hadoop, Kafka, Spark, HBase, R Server o Storm para HDInsight desde un explorador, la CLI de Azure clásica, Azure PowerShell, REST o el SDK.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017,seodec18
ms.date: 08/06/2020
ms.openlocfilehash: f9c5b8ae16cb43576d788f72478e2cfba521a736
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749872"
---
# <a name="set-up-clusters-in-hdinsight-with-apache-hadoop-apache-spark-apache-kafka-and-more"></a>Configuración de clústeres en HDInsight con Apache Hadoop, Apache Spark, Apache Kafka, etc.

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Aprenda a instalar y configurar Apache Hadoop, Apache Spark, Apache Kafka, Interactive Query, Apache HBase, ML Services o Apache Storm en HDInsight. Además, aprenda a personalizar clústeres y a aumentar la seguridad al asociarlos a un dominio.

Un clúster de Hadoop se compone de varias máquinas virtuales (nodos) que se usan para el procesamiento distribuido de tareas. Azure HDInsight se ocupa de los detalles de implementación de la instalación y configuración de los nodos individuales, por lo que solo tiene que proporcionar información de configuración general.

> [!IMPORTANT]  
> La facturación del clúster de HDInsight se inicia una vez creado el clúster y solo se detiene cuando se elimina. Se facturan por minuto realizando una prorrata, por lo que siempre debe eliminar aquellos que ya no se estén utilizando. Aprenda a [eliminar un clúster](hdinsight-delete-cluster.md).

Si usa varios clústeres juntos, querrá crear una red virtual y, si está usando un clúster de Spark, también querrá usar Hive Warehouse Connector. Para más información, consulte [Planificación de una red virtual para Azure HDInsight](./hdinsight-plan-virtual-network-deployment.md) e [Integración de Apache Spark y Apache Hive con el conector de Hive Warehouse](interactive-query/apache-hive-warehouse-connector.md).

## <a name="cluster-setup-methods"></a>Métodos de configuración de clústeres

La tabla siguiente muestra los distintos métodos que se pueden usar para configurar un clúster de HDInsight.

| Clústeres creados con | Explorador web | Línea de comandos | API DE REST | SDK |
| --- |:---:|:---:|:---:|:---:|
| [Azure Portal](hdinsight-hadoop-create-linux-clusters-portal.md) |✔ |&nbsp; |&nbsp; |&nbsp; |
| [Azure Data Factory](hdinsight-hadoop-create-linux-clusters-adf.md) |✔ |✔ |✔ |✔ |
| [CLI de Azure](hdinsight-hadoop-create-linux-clusters-azure-cli.md) |&nbsp; |✔ |&nbsp; |&nbsp; |
| [Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md) |&nbsp; |✔ |&nbsp; |&nbsp; |
| [cURL](hdinsight-hadoop-create-linux-clusters-curl-rest.md) |&nbsp; |✔ |✔ |&nbsp; |
| [Plantillas del Administrador de recursos de Azure](hdinsight-hadoop-create-linux-clusters-arm-templates.md) |&nbsp; |✔ |&nbsp; |&nbsp; |

Este artículo le guía a lo largo del proceso de configuración en [Azure Portal](https://portal.azure.com), donde puede crear un clúster de HDInsight.

## <a name="basics"></a>Aspectos básicos

![opciones de creación rápida personalizada de hdinsight](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-basics-blank-fs.png)

### <a name="project-details"></a>Detalles del proyecto

[Azure Resource Manager](../azure-resource-manager/management/overview.md) lo ayuda a trabajar con los recursos de la aplicación como grupo, al que se conoce como [grupo de recursos](../azure-resource-manager/management/overview.md#resource-groups) de Azure. Puede implementar, actualizar, supervisar o eliminar todos los recursos de la aplicación en una operación única y coordinada.

### <a name="cluster-details"></a>Detalles del clúster

#### <a name="cluster-name"></a>Nombre del clúster

Los nombres de clúster de HDInsight tienen las siguientes restricciones:

* Caracteres permitidos: a-z, 0-9, A-Z
* Longitud máxima: 59
* Nombres reservados: aplicaciones
* El ámbito de nombres del clúster es común para todo Azure, para todas las suscripciones. Por lo tanto, el nombre del clúster debe ser único en todo el mundo.
* Los seis primeros caracteres deben ser únicos en una red virtual.

#### <a name="region"></a>Region

No es preciso que especifique la ubicación del clúster explícitamente: El clúster se encuentra en la misma ubicación que el almacenamiento predeterminado. Si desea obtener una lista de las regiones admitidas, seleccione la lista desplegable **Región** en [Precios de HDInsight](https://go.microsoft.com/fwLink/?LinkID=282635&clcid=0x409).

#### <a name="cluster-type"></a>Tipo de clúster

Actualmente, Azure HDInsight proporciona los siguientes tipos de clúster, cada uno de ellos con un conjunto de componentes que ofrecen determinadas funcionalidades.

> [!IMPORTANT]  
> Los clústeres de HDInsight están disponibles en distintos tipos, cada uno de ellos para una carga de trabajo o una tecnología única. No hay ningún método admitido para crear un solo clúster que combine varios tipos, como Storm y HBase. Si la solución requiere tecnologías repartidas entre varios tipos de clústeres de HDInsight, una [red virtual de Azure](https://docs.microsoft.com/azure/virtual-network) puede conectar los tipos de clústeres necesarios.

| Tipo de clúster | Funcionalidad |
| --- | --- |
| [Hadoop](hadoop/apache-hadoop-introduction.md) |Consulta por lotes y análisis de datos almacenados |
| [HBase](hbase/apache-hbase-overview.md) |Procesamiento de grandes cantidades de datos no SQL sin esquema |
| [Interactive Query](./interactive-query/apache-interactive-query-get-started.md) |Almacenamiento en caché en memoria para realizar consultas de Hive interactivas y más rápidas |
| [Kafka](kafka/apache-kafka-introduction.md) | Una plataforma de streaming distribuida que se puede utilizar para compilar aplicaciones y canalizaciones de datos de streaming en tiempo real. |
| [ML Services](r-server/r-server-overview.md) |Distintas capacidades de estadísticas de macrodatos, modelado de predicción y aprendizaje automático |
| [Spark](spark/apache-spark-overview.md) |Procesamiento en memoria, consultas interactivas, procesamiento de transmisión de microlotes |
| [Storm](storm/apache-storm-overview.md) |Procesamiento de eventos en tiempo real |

#### <a name="version"></a>Versión

Elija la versión de HDInsight para este clúster. Para más información, consulte [Versiones compatibles de HDInsight](hdinsight-component-versioning.md#supported-hdinsight-versions).

### <a name="cluster-credentials"></a>Credenciales del clúster

Con los clústeres de HDInsight, puede configurar dos cuentas de usuario durante la creación del clúster:

* Nombre de usuario de inicio de sesión del clúster: El nombre de usuario predeterminado es *admin*. Emplea la configuración básica en el portal de Azure. A veces, se denomina "Usuario de clúster" o "usuario HTTP".
* Nombre de usuario de Secure Shell (SSH): se usa para conectarse al clúster mediante SSH. Para más información, consulte [Uso SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

El nombre de usuario de HTTP tiene las siguientes restricciones:

* Caracteres especiales que se permiten: `_` y `@`
* Caracteres no permitidos: #;."',\/:`!*?$(){}[]<>|&--=+%~^space
* Longitud máxima: 20

El nombre de usuario de SSH tiene las siguientes restricciones:

* Caracteres especiales que se permiten: `_` y `@`
* Caracteres no permitidos: #;."',\/:`!*?$(){}[]<>|&--=+%~^space
* Longitud máxima: 64
* Nombres reservados: hadoop, users, oozie, hive, mapred, ambari-qa, zookeeper, tez, hdfs, sqoop, yarn, hcat, ams, hbase, storm, administrator, admin, user, user1, test, user2, test1, user3, admin1, 1, 123, a, actuser, adm, admin2, aspnet, backup, console, david, guest, john, owner, root, server, sql, support, support_388945a0, sys, test2, test3, user4, user5, spark

## <a name="storage"></a>Storage

![Configuración de almacenamiento de clúster: Puntos de conexión compatibles con HDFS](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-storage.png)

Aunque una instalación local de Hadoop usa el sistema de archivos distribuido de Hadoop (HDFS) para el almacenamiento en el clúster, en la nube se usan puntos de conexión de almacenamiento conectados al clúster. El empleo de almacenamiento en la nube significa que puede eliminar con seguridad los clústeres de HDInsight usados para el cálculo y, al mismo tiempo, conservar los datos.

Los clústeres de HDInsight pueden usar las siguientes opciones de almacenamiento:

* Azure Data Lake Storage Gen2
* Azure Data Lake Storage Gen1
* Azure Storage de uso general v2
* Azure Storage de uso general v1
* Blob en bloques de Azure Storage (**solo se admite como almacenamiento secundario**)

Para más información sobre las opciones de almacenamiento con HDInsight, consulte [Comparación de opciones de almacenamiento para usar con clústeres de Azure HDInsight](hdinsight-hadoop-compare-storage-options.md).

> [!WARNING]  
> No se admite el uso de una cuenta de almacenamiento adicional en una ubicación diferente a la del clúster de HDInsight.

Durante la configuración se especifica un contenedor de blobs de una cuenta de Azure Storage o de Data Lake Storage para el punto de conexión de almacenamiento predeterminado. El almacenamiento predeterminado contiene los registros del sistema y de la aplicación. De manera opcional, puede especificar más cuentas vinculadas de Azure Storage y cuentas de Data Lake Storage a las que el clúster pueda acceder. El clúster de HDInsight y las cuentas de almacenamiento dependientes deben estar en la misma ubicación de Azure.

[!INCLUDE [secure-transfer-enabled-storage-account](../../includes/hdinsight-secure-transfer.md)]

> [!IMPORTANT]
> Habilitar la transferencia segura del almacenamiento después de crear un clúster puede producir errores al usar la cuenta de almacenamiento, por lo que no se recomienda. Es mejor crear un clúster mediante una cuenta de almacenamiento que ya tenga habilitada la transferencia segura.

> [!Note]  
> Azure HDInsight no transfiere, mueve ni copia automáticamente los datos almacenados en Azure Storage de una región a otra.

### <a name="metastore-settings"></a>Configuración de metastores

Puede crear tiendas de metadatos opcionales de Hive o Apache Oozie. Pero no todos los tipos de clúster admiten las tiendas de metadatos; además, Azure SQL Data Warehouse no es compatible con ellas.

Para más información, consulte [Use external metadata stores in Azure HDInsight](./hdinsight-use-external-metadata-stores.md) (Uso de almacenes externos de metadatos en Azure HDInsight).

> [!IMPORTANT]  
> Cuando cree una tienda de metadatos personalizada, no use guiones ni espacios en el nombre de la base de datos. Podría provocar que el proceso de creación del clúster diera error.

#### <a name="sql-database-for-hive"></a>Base de datos SQL para Hive

Si quiere conservar las tablas de Hive después de eliminar un clúster de HDInsight, use una tienda de metadatos personalizada. Luego puede adjuntar la tienda de metadatos a otro clúster de HDInsight.

Un metastore de HDInsight creado para una versión del clúster de HDInsight no se puede compartir entre diferentes versiones del clúster de HDInsight. Para obtener una lista de versiones de HDInsight, consulte [Versiones compatibles de HDInsight](hdinsight-component-versioning.md#supported-hdinsight-versions).

> [!IMPORTANT]
> El metastore predeterminado proporciona una instancia de Azure SQL Database con un **límite de 5 DTU de nivel básico (no actualizable)** . Esto resulta adecuado para la realización de pruebas básicas. En el caso de las cargas de trabajo de producción o de gran volumen, se recomienda migrar a un metastore externo.

#### <a name="sql-database-for-oozie"></a>Base de datos SQL para Oozie

Para aumentar el rendimiento al usar Oozie, utilice un Metastore personalizado. Una tienda de metadatos también puede proporcionar acceso a datos de trabajo de Oozie después de eliminar el clúster.

#### <a name="sql-database-for-ambari"></a>Base de datos SQL para Ambari

Ambari se usa para supervisar clústeres de HDInsight, realizar cambios en la configuración y almacenar información de administración del clúster, así como el historial de trabajos. La característica de bases de datos personalizadas de Ambari permite implementar un nuevo clúster y configurar Ambari en una base de datos externa que usted administra. Para obtener más información, consulte [Base de datos de Ambari personalizada](./hdinsight-custom-ambari-db.md).

> [!IMPORTANT]  
> No puede volver a usar un Oozie Metastore personalizado. Para usar un Oozie Metastore personalizado, debe proporcionar una base de datos de Azure SQL vacía al crear el clúster HDInsight.

## <a name="security--networking"></a>Seguridad y redes

![Opciones de creación de hdinsight para elegir el paquete de seguridad de la empresa](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-security-networking.png)

### <a name="enterprise-security-package"></a>Paquete de seguridad de la empresa

Para los tipos de clúster de Hadoop, Spark, HBase, Kafka e Interactive Query, puede elegir la opción para habilitar **Enterprise Security Package**. Este paquete ofrece la opción de tener una configuración de clúster más segura mediante Apache Ranger y la integración con Azure Active Directory. Para más información, consulte el [Introducción a la seguridad de la empresa en Azure HDInsight](./domain-joined/hdinsight-security-overview.md).

El paquete de seguridad de la empresa le permite integrar HDInsight con Active Directory y Apache Ranger. Se pueden crear varios usuarios con el paquete de seguridad de la empresa.

Para más información sobre la creación de un clúster de HDInsight unido a un dominio, consulte la [creación de un entorno de espacio aislado de HDInsight unido a un dominio](./domain-joined/apache-domain-joined-configure.md).

### <a name="tls"></a>TLS

Para más información, consulte [Seguridad de la capa de transporte](./transport-layer-security.md).

### <a name="virtual-network"></a>Virtual network

Si la solución requiere tecnologías repartidas entre varios tipos de clústeres de HDInsight, una [red virtual de Azure](https://docs.microsoft.com/azure/virtual-network) puede conectar los tipos de clústeres necesarios. Esta configuración permite que los clústeres, y cualquier otro código que se implemente en ellos, se comuniquen directamente entre sí.

Para más información acerca del uso de una red virtual de Azure con HDInsight, consulte [Planeamiento de una red virtual para HDInsight](hdinsight-plan-virtual-network-deployment.md).

Para ver un ejemplo de cómo usar dos tipos de clúster en una red virtual de Azure, consulte [Uso del flujo estructurado de Apache Spark con Apache Kafka en HDInsight](hdinsight-apache-kafka-spark-structured-streaming.md). Para más información acerca del uso de HDInsight con una red virtual, incluidos los requisitos de configuración específicos de la red virtual, consulte [Planeamiento de una red virtual para HDInsight](hdinsight-plan-virtual-network-deployment.md).

### <a name="disk-encryption-setting"></a>Configuración de cifrado de disco

Para obtener más información, consulte [Cifrado de disco mediante claves administradas por el cliente](./disk-encryption.md).

### <a name="kafka-rest-proxy"></a>Proxy de REST de Kafka

Esta opción solo está disponible para el tipo de clúster Kafka. Para más información, consulte [Uso de un proxy de REST](./kafka/rest-proxy.md).

### <a name="identity"></a>Identidad

Para obtener más información, consulte [Identidades administradas en Azure HDInsight](./hdinsight-managed-identities.md).

## <a name="configuration--pricing"></a>Configuración y precios

![Selección del tamaño de nodo de HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-configuration.png)

Mientras que exista el clúster, se le facturará por el uso de nodos. La facturación se inicia una vez creado el clúster y se detiene cuando se elimina el clúster. Los clústeres no se puede desasignar ni ponerse en espera.

### <a name="node-configuration"></a>Configuración del nodo

Cada tipo de clúster tiene su propio número de nodos, terminología para los nodos y tamaño de máquina virtual predeterminado. En la siguiente tabla, el número de nodos de cada tipo de nodo se muestra entre paréntesis.

| Tipo | Nodos | Diagrama |
| --- | --- | --- |
| Hadoop |Nodo principal (2), nodo de trabajo (1+) |![Nodos de clúster de Hadoop en HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-hadoop-cluster-type-nodes.png) |
| HBase |Servidor principal (2), servidor de región (más de 1), nodo maestro/ZooKeeper (3) |![Configuración del tipo de clúster de HBase en HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-hbase-cluster-type-setup.png) |
| Storm |Nodo Nimbus (2), servidor de supervisor (más de 1), nodo ZooKeeper (3) |![Configuración del tipo de clúster de Storm en HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-storm-cluster-type-setup.png) |
| Spark |Nodo principal (2), nodo de trabajo (más de 1), nodo ZooKeeper (3) (gratis para el tamaño de máquina virtual ZooKeeper A1) |![Configuración del tipo de clúster de Spark en HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-spark-cluster-type-setup.png) |

Para más información, vea [Configuración de nodo predeterminada y tamaños de máquina virtual para clústeres](hdinsight-component-versioning.md#default-node-configuration-and-virtual-machine-sizes-for-clusters) en "¿Cuáles son los componentes y versiones de Hadoop disponibles con HDInsight?"

El costo de los clústeres de HDInsight viene determinado por el número de nodos y por los tamaños de las máquinas virtuales de los nodos.

Los distintos tipos de clúster tienen distintos tipos, número y tamaños de nodos:
* Tipo de clúster predeterminado de Hadoop:
    * Dos *nodos principales*  
    * Cuatro *nodos de trabajo*
* Tipo de clúster predeterminado de Storm:
    * Dos *nodos Nimbus*
    * Tres *nodos ZooKeeper*
    * Cuatro *nodos de supervisor*

Si simplemente está probando HDInsight, se recomienda usar un nodo de trabajo. Para más información sobre los precios de HDInsight, vea [HDInsight Precios](https://go.microsoft.com/fwLink/?LinkID=282635&clcid=0x409).

> [!NOTE]  
> El límite de tamaño del clúster varía entre las suscripciones a Azure. Póngase en contacto con el [servicio de soporte técnico de facturación de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request) para aumentar el límite.

Al usar Azure Portal para configurar el clúster, el tamaño del nodo está disponible en la pestaña **Configuración y precios**. En el portal también puede ver el costo asociado a los diferentes tamaños de nodo.

### <a name="virtual-machine-sizes"></a>Tamaños de máquina virtual

Al implementar clústeres, elija los recursos de proceso en función de la solución que tenga previsto implementar. Las máquinas virtuales siguientes se usan para clústeres de HDInsight:

* Máquinas virtuales de las series A y D1-4: [Tamaños de máquinas virtuales Linux para uso general](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-general)
* Máquinas virtuales de la serie D11-14: [Tamaños de máquinas virtuales Linux optimizadas para memoria](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-memory)

Para averiguar el valor que debe usar para especificar un tamaño de máquina virtual durante la creación de un clúster mediante los distintos SDK o mientras usa Azure PowerShell, vea [Tamaños de máquina virtual para clústeres de HDInsight](../cloud-services/cloud-services-sizes-specs.md#size-tables). Use el valor de la columna **Tamaño** de las tablas de este artículo vinculado.

> [!IMPORTANT]  
> Si necesita más de 32 nodos de trabajo en un clúster, tiene que seleccionar un tamaño de nodo principal con al menos 8 núcleos y 14 GB de RAM.

Para más información, consulte [Tamaños de las máquinas virtuales Linux en Azure](../virtual-machines/windows/sizes.md). Para más información sobre los precios de los diferentes tamaños, consulte [Precios de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight).

### <a name="add-application"></a>Agregar aplicación

Una aplicación de HDInsight es una aplicación que los usuarios pueden instalar en un clúster de HDInsight basado en Linux. Puede usar aplicaciones proporcionadas por Microsoft, de terceros o desarrolladas por sí mismo. Para más información, consulte [Instalación de aplicaciones de Apache Hadoop de terceros en Azure HDInsight](hdinsight-apps-install-applications.md).

La mayoría de las aplicaciones de HDInsight se instalan en un nodo perimetral vacío.  Un nodo perimetral vacío es una máquina virtual Linux con las mismas herramientas cliente instaladas y configuradas que en el nodo principal. Se puede usar el nodo perimetral para acceder al clúster y para probar y hospedar las aplicaciones cliente. Para obtener más información, consulte [Uso de nodos perimetrales vacíos en HDInsight](hdinsight-apps-use-edge-node.md).

### <a name="script-actions"></a>Acciones de script

Puede instalar componentes adicionales o personalizar la configuración del clúster mediante el uso de scripts durante la creación. Tales scripts se invocan mediante la opción de **Acción de script**, una opción de configuración que se puede usar a partir de los cmdlets de Windows PowerShell de HDInsight, en el Portal de Azure o el SDK de .NET para HDInsight. Para obtener más información, consulte [Personalización de un clúster de HDInsight mediante la acción de script](hdinsight-hadoop-customize-cluster-linux.md).

Algunos componentes nativos de Java, como Apache Mahout y Cascading, se pueden ejecutar en el clúster como archivos Java Archive (JAR). Estos archivos JAR se pueden distribuir a Azure Storage y enviarse a clústeres de HDInsight con los mecanismos de envío de trabajos de Hadoop. Para más información, consulte [Envío de trabajos de Apache Hadoop mediante programación](hadoop/submit-apache-hadoop-jobs-programmatically.md).

> [!NOTE]  
> Si tiene problemas con la implementación de archivos JAR en clústeres de HDInsight o con la llamada a archivos JAR en clústeres de HDInsight, póngase en contacto con el [soporte técnico de Microsoft](https://azure.microsoft.com/support/options/).
>
> Cascading no se admite en HDInsight, y no puede optar a recibir soporte técnico de Microsoft. Para ver las listas de los componentes compatibles, consulte [Novedades en las versiones de clústeres proporcionadas por HDInsight](hdinsight-component-versioning.md).

En ocasiones, querrá configurar los siguientes archivos de configuración durante el proceso de creación:

* clusterIdentity.xml
* core-site.xml
* gateway.xml
* hbase-env.xml
* hbase-site.xml
* hdfs-site.xml
* hive-env.xml
* hive-site.xml
* mapred-site
* oozie-site.xml
* oozie-env.xml
* storm-site.xml
* tez-site.xml
* webhcat-site.xml
* yarn-site.xml

Para más información, consulte [Personalización de los clústeres de HDInsight con Bootstrap](hdinsight-hadoop-customize-cluster-bootstrap.md).

## <a name="next-steps"></a>Pasos siguientes

* [Solución de errores de creación de clústeres con Azure HDInsight](./hadoop/hdinsight-troubleshoot-cluster-creation-fails.md)
* [¿Qué son HDInsight, el ecosistema Apache Hadoop y los clústeres de Hadoop?](hadoop/apache-hadoop-introduction.md)
* [Introducción al uso de Apache Hadoop en HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Trabajo en Apache Hadoop en HDInsight desde un equipo Windows](hdinsight-hadoop-windows-tools.md)
