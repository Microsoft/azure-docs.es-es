---
title: 'Ventajas: Migración de Apache Hadoop local a Azure HDInsight'
description: Obtenga información acerca de la motivación y ventajas de almacenamiento para migrar clústeres locales de Apache Hadoop a Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: ashishth
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 11/15/2019
ms.openlocfilehash: 1de9fc480c753b2497a1ea4e3438583b3582bc96
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072780"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---motivation-and-benefits"></a>Motivación y ventajas de migrar clústeres locales de Apache Hadoop a Azure HDInsight

Este artículo es el primero de una serie sobre las mejores prácticas para migrar implementaciones locales del ecosistema de Apache Hadoop a Azure HDInsight. Esta serie de artículos está dirigida a las personas responsables del diseño, la implementación y la migración de soluciones de Apache Hadoop a Azure HDInsight. Los roles que pueden beneficiarse de estos artículos incluyen arquitectos de nubes, administradores de Hadoop e ingenieros de DevOps. Los desarrolladores de software, los ingenieros de datos y los científicos de datos también deberían beneficiarse de la explicación de cómo funcionan los diferentes tipos de clústeres en la nube.

## <a name="why-to-migrate-to-azure-hdinsight"></a>Por qué migrar a Azure HDInsight

Azure HDInsight es una distribución de nube de componentes de Hadoop. Azure HDInsight hace que sea fácil, rápido y rentable procesar grandes cantidades de datos. HDInsight incluye los marcos de código abierto más populares, como:

- Apache Hadoop
- Spark de Apache
- Apache Hive con LLAP
- Apache Kafka
- Apache Storm
- HBase Apache
- R

## <a name="azure-hdinsight-advantages-over-on-premises-hadoop"></a>Ventajas de Azure HDInsight sobre las instancias locales de Hadoop

- **Bajo costo**: se pueden reducir los costos mediante la [creación de clústeres a petición](../hdinsight-hadoop-create-linux-clusters-adf.md) y pagando solo por lo que usa. El almacenamiento y proceso desacoplado proporciona flexibilidad al mantener el volumen de datos independiente del tamaño del clúster.

- **Creación automatizada del clúster**: la creación automatizada de clúster requiere una configuración mínima. La automatización puede usarse para clústeres a petición.

- **Hardware y configuración administrados**: no es necesario preocuparse por la infraestructura ni por el hardware físico con un clúster de HDInsight. Simplemente especifique la configuración del clúster, y Azure lo configurará.

- **Fácilmente escalable**: HDInsight le permite [escalar o reducir](../hdinsight-administer-use-portal-linux.md) verticalmente las cargas de trabajo. Azure se encarga de la redistribución de datos y del reequilibrio de la carga de trabajo sin interrumpir los trabajos de procesamiento de datos.

- **Disponibilidad global**: HDInsight está disponible en más [regiones](https://azure.microsoft.com/regions/services/) que ninguna otra oferta de análisis de macrodatos. También está disponible en Azure Government, China y Alemania, lo que le permite satisfacer las necesidades de su empresa en áreas soberanas clave.

- **Seguro y compatible**: HDInsight le permite proteger los recursos de datos de la empresa mediante [Azure Virtual Network](../hdinsight-plan-virtual-network-deployment.md), el  [cifrado](../hdinsight-hadoop-create-linux-clusters-with-secure-transfer-storage.md) y la integración con  [Azure Active Directory](../domain-joined/hdinsight-security-overview.md). HDInsight también cumple con los  [estándares de cumplimiento normativo](https://azure.microsoft.com/overview/trusted-cloud) más conocidos del sector y de la administración.

- **Administración de versiones simplificada**: Azure HDInsight administra la versión de componentes del ecosistema de Hadoop y los mantiene actualizados. Las actualizaciones de software suelen ser un proceso complejo para las implementaciones locales.

- **Clústeres más pequeños optimizados para cargas de trabajo específicas con menos dependencias entre los componentes**: un programa de instalación de Hadoop local típico usa un único clúster que sirve para muchos propósitos. Con Azure HDInsight, se pueden crear clústeres específicos para cargas de trabajo. La creación de clústeres para cargas de trabajo específicas elimina la complejidad de mantener un solo clúster con complejidad cada vez mayor.

- **Productividad**: puede usar varias herramientas de Hadoop y Spark en su entorno de desarrollo preferido.

- **Extensibilidad con herramientas personalizadas o aplicaciones de terceros**: los clústeres de HDInsight se pueden ampliar con componentes instalados y también pueden integrarse con otras soluciones de macrodatos utilizando implementaciones de [un solo clic](https://azure.microsoft.com/services/hdinsight/partner-ecosystem/) del Marketplace de Azure.

- **Fácil administración y supervisión**: Azure HDInsight se integra con los  [registros de Azure Monitor](../hdinsight-hadoop-oms-log-analytics-tutorial.md)  para proporcionar una única interfaz con la que puede supervisar todos los clústeres.

- **Integración con otros servicios de Azure**: HDInsight puede integrarse fácilmente con otros servicios populares de Azure como los siguientes:

    - Azure Data Factory (ADF)
    - Azure Blob Storage
    - Azure Data Lake Storage Gen2
    - Azure Cosmos DB
    - Azure SQL Database
    - Azure Analysis Services

- **Componentes y procesos de recuperación automática**: HDInsight comprueba constantemente los componentes de la infraestructura y de código abierto con su propia infraestructura de supervisión. También se recupera automáticamente de errores críticos como la falta de disponibilidad de nodos y componentes de código abierto. Las alertas se activan en Ambari si se produjo un error en cualquier componente de OSS.

Para obtener más información, vea el artículo [Qué son Azure HDInsight y la pila de tecnología de Apache Hadoop](../hadoop/apache-hadoop-introduction.md).

## <a name="migration-planning-process"></a>Proceso de planeamiento de migración

Se recomiendan los pasos siguientes para planear una migración de los clústeres locales de Hadoop a Azure HDInsight:

1. Comprender las topologías y la implementación local actual.
2. Comprender el ámbito del proyecto actual, las escalas de tiempo y la experiencia del equipo.
3. Comprender los requisitos de Azure.
4. Elaborar un plan detallado basado en los procedimientos recomendados.

## <a name="gathering-details-to-prepare-for-a-migration"></a>Recopilación de detalles para preparar una migración

En este sección se proporcionan plantillas de cuestionarios para ayudar a reunir información importante acerca de:

- Implementación local
- Detalles del proyecto
- Requisitos de Azure

### <a name="on-premises-deployment-questionnaire"></a>Cuestionario de implementación local

| **Pregunta** | **Ejemplo** | **Respuesta** |
|---|---|---|
|**Tema**: **Entorno**|||
|Versión de distribución de clúster|HDP 2.6.5, CDH 5.7|
|Componentes de ecosistema de big Data|HDFS, Yarn, Hive, LLAP, Impala, Kudu, HBase, Spark, MapReduce, Kafka, Zookeeper, Solr, Sqoop, Oozie, Ranger, Atlas, Falcon, Zeppelin, R|
|Tipos de clúster|Hadoop, Spark, Confluent Kafka, Storm, Solr|
|Número de clústeres|4|
|Número de nodos maestros|2|
|Número de nodos de trabajo|100|
|Número de nodos perimetrales| 5|
|Espacio en disco total|100 TB|
|Configuración del nodo maestro|m/y, CPU, disco, etc.|
|Configuración de los nodos de datos|m/y, CPU, disco, etc.|
|Configuración de los nodos perimetrales|m/y, CPU, disco, etc.|
|¿Cifrado HDFS?|Sí|
|Alta disponibilidad|Alta disponibilidad de HDFS HA, alta disponibilidad de Metastore|
|Recuperación ante desastres / Copia de seguridad|¿Copia de seguridad del clúster?|  
|Sistemas que dependen de clúster|SQL Server, Teradata, Power BI, MongoDB|
|Integración con productos de terceros|Tableau GridGain, Qubole, Informatica, Splunk|
|**Tema**: **Seguridad**|||
|Seguridad del perímetro|Firewalls|
|Autenticación y autorización de clúster|Active Directory, Ambari, Cloudera Manager, sin autenticación|
|Control de acceso de HDFS|  Manual, usuarios SSH|
|Autenticación y autorización de Hive|Sentry, LDAP, AD con Kerberos, Ranger|
|Auditoría|Ambari, Cloudera Navigator, Ranger|
|Supervisión|Graphite, collectd, statsd, Telegraf, InfluxDB|
|Alertas|Kapacitor, Prometheus, Datadog|
|Duración de retención de datos| 3 años, 5 años|
|Administradores de clúster|Administrador único, varios administradores|

### <a name="project-details-questionnaire"></a>Cuestionario de detalles del proyecto

|**Pregunta**|**Ejemplo**|**Respuesta**|
|---|---|---|
|**Tema**: **cargas de trabajo y frecuencia**|||
|Trabajos MapReduce|10 trabajos, dos veces al día||
|Trabajos de Hive|100 trabajos, cada hora||
|Trabajos por lotes de Spark|50 trabajos, cada 15 minutos||
|Trabajos de Spark Streaming|5 trabajos, cada 3 minutos||
|Trabajos de Structured Streaming|5 trabajos, cada minuto||
|Trabajos de entrenamiento del modelo de Machine Learning|2 trabajos, una vez a la semana||
|Lenguajes de programación|Python, Scala, Java||
|Scripting|Shell, Python||
|**Tema**: **Data**|||
|Orígenes de datos|Archivos sin formato, Json, Kafka, RDBMS||
|Orquestación de datos|Flujos de trabajo de Oozie, flujo de aire||
|Búsquedas en memoria|Apache Ignite, Redis||
|Destinos de datos|HDFS, RDBMS, Kafka, MPP ||
|**Tema**: **Metadatos**|||
|Tipo de base de datos de Hive|Mysql, Postgres||
|Número de metastores de Hive|2||
|Número de tablas de Hive|100||
|Número de directivas de Ranger|20||
|Número de flujos de trabajo de Oozie|100||
|**Tema**: **Escala**|||
|Volumen de datos, incluida la replicación|100 TB||
|Volumen diario de ingesta|50 GB||
|Tasa de crecimiento de datos|10 % al año||
|Tasa de crecimiento de los nodos de clúster|5 % al año
|**Tema**: **utilización del clúster**|||
|% medio de CPU usada|60%||
|% medio de memoria usada|75 %||
|Espacio en disco usado|75 %||
|% medio de red usada|25 %
|**Tema**: **personal**|||
|Número de administradores|2||
|Número de desarrolladores|10||
|Número de usuarios finales|100||
|Aptitudes|Hadoop, Spark||
|Número de recursos disponibles para los esfuerzos de migración|2||
|**Tema**: **Limitaciones**|||
|Limitaciones actuales|La latencia es alta||
|Desafíos actuales|Problema de simultaneidad||

### <a name="azure-requirements-questionnaire"></a>Cuestionario de los requisitos de Azure

|**Pregunta**|**Ejemplo**|**Respuesta**|
|---|---|---|
|**Tema**: **Infraestructura** |||
| Región preferida|Este de EE. UU.||
|¿Red virtual preferida?|Sí||
|¿Es necesaria alta disponibilidad o recuperación ante desastres?|Sí||
|¿Integración con otros servicios en la nube?|ADF, CosmosDB||
|**Tema**:   **Movimiento de datos**  |||
|Preferencia de carga inicial|DistCp, Data box, ADF, WANDisco||
|Transferencia de datos delta|DistCp, AzCopy||
|Transferencia de datos incremental en curso|DistCp, Sqoop||
|**Tema**:   **supervisión y alertas** |||
|Usar la supervisión y las alertas de Azure en lugar de supervisión de terceros|Usar supervisión y alertas de Azure||
|**Tema**:   **preferencias de seguridad** |||
|¿Canalización de datos privada y protegida?|Sí||
|¿Un clúster unido a un dominio (ESP)?|     Sí||
|¿Sincronización de AD local en la nube?|     Sí||
|¿Número de usuarios de AD para sincronizar?|          100||
|¿Aceptar sincronizar contraseñas en la nube?|    Sí||
|¿Solo usuarios en la nube?|                 Sí||
|¿MFA necesario?|                       No|| 
|¿Requisitos de autorización de datos?|  Sí||
|¿Control de acceso basado en roles?|        Sí||
|¿Auditoría necesaria?|                  Sí||
|¿Cifrado de datos en reposo?|          Sí||
|¿Cifrado de datos en tránsito?|       Sí||
|**Tema**:   **preferencias reestructuración de la arquitectura** |||
|Clúster único frente a determinados tipos de clúster|Tipos de clústeres específicos||
|¿Almacenamiento remoto frente almacenamiento colocado?|Almacenamiento remoto||
|¿Tamaño de clúster más pequeño ya que los datos se almacenan de forma remota?|Tamaño de clúster más pequeño||
|¿Usar varios clústeres más pequeños en lugar de un solo clúster grande?|Uso de varios clústeres más pequeños||
|¿Usar una tienda de metadatos remota?|Sí||
|¿Compartir tiendas de metadatos entre clústeres diferentes?|Sí||
|¿Deconstruir las cargas de trabajo?|Reemplace los trabajos de Hive con trabajos de Spark||
|¿Usar ADF para orquestación de datos?|No||

## <a name="next-steps"></a>Pasos siguientes

Lea el siguiente artículo de esta serie:

- [Architecture best practices for on-premises to Azure HDInsight Hadoop migration](apache-hadoop-on-premises-migration-best-practices-architecture.md) (Procedimientos recomendados de arquitectura para migrar clústeres locales de Hadoop a Azure HDInsight)
