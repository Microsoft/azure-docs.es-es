---
title: Instalación de aplicaciones de terceros en Azure HDInsight
description: Aprenda a instalar aplicaciones de Apache Hadoop de terceros en Azure HDInsight.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/17/2019
ms.author: hrasheed
ms.openlocfilehash: e0d1f9ad99e1b64560321312a22f61f5a2ef3dea
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89016043"
---
# <a name="install-third-party-apache-hadoop-applications-on-azure-hdinsight"></a>Instalación de aplicaciones de Apache Hadoop de terceros en Azure HDInsight

Aprenda a instalar una aplicación de [Apache Hadoop](https://hadoop.apache.org/) de terceros en Azure HDInsight. Para obtener instrucciones sobre cómo instalar su propia aplicación, consulte [Instalación de aplicaciones de HDInsight personalizadas](hdinsight-apps-install-custom-applications.md).

Una aplicación de HDInsight es una aplicación que los usuarios pueden instalar en un clúster de HDInsight. Estas aplicaciones puede desarrollarlas Microsoft, fabricantes de software independientes (ISV) o el propio usuario.  

En la lista siguiente se muestran las aplicaciones publicadas:

|Application |Tipos de clúster | Descripción |
|---|---|---|
|[Plataforma de inteligencia AtScale](https://azuremarketplace.microsoft.com/marketplace/apps/atscaleinc.atscale) |Hadoop |AtScale convierte el clúster de HDInsight en un servidor OLAP de escalabilidad horizontal, lo que permite consultar miles de millones de filas de datos de forma interactiva mediante las herramientas de inteligencia empresarial que ya conoce, posee y prefiere: desde Microsoft Excel, Power BI o Tableau Software hasta QlikView. |
|[CDAP para HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/cask.cdap-for-hdinsight) |HBase |CDAP es la primera plataforma de integración unificada para macrodatos que acelera la rentabilidad de Hadoop y permite a TI proporcionar datos de autoservicio. De código abierto y extensible, CDAP elimina las barreras para la innovación. Requisitos: 4 nodos de región, mín. D3 v2. |
|[Datameer](https://azuremarketplace.microsoft.com/marketplace/apps/datameer.datameer) |Hadoop |La plataforma escalable de autoservicio de Datameer para preparar, explorar y administrar los datos para el análisis acelera la conversión de datos complejos de varios orígenes en información empresarial valiosa, con una entrega más rápida e inteligente de información detallada a escala empresarial. |
|[Dataiku DSS en HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/dataiku.dss-on-hdi) |Hadoop, Spark |Dataiku DSS en una plataforma de ciencia de datos empresariales que permite a científicos y analistas de datos colaborar para diseñar y ejecutar nuevos servicios y productos de datos de forma más eficaz, con el fin de convertir los datos sin procesar en predicciones de gran impacto. |
|[WANdisco Fusion HDI App](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/wandisco.wandisco-liveanalytics) |Hadoop, Spark, HBase, Storm, Kafka |El mantenimiento de la coherencia de los datos en un entorno distribuido es un desafío de las operaciones de datos masivos. WANdisco Fusion, una plataforma de software de nivel empresarial, soluciona este problema mediante la habilitación de la coherencia de datos no estructurados en cualquier entorno. |
|[H2O SparklingWater para HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/h2o-ai.h2o-sparklingwater) |Spark |H2O Sparkling Water es compatible con los siguientes algoritmos distribuidos: GLM, Naïve Bayes, Distributed Random Forest, Gradient Boosting Machine, redes neurales profundas, aprendizaje profundo, K-means, PCA, Generalized Low Rank Models, Detección de anomalías y Autoencoders. |
|[Striim para la integración de datos en tiempo real en HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/striim.striimbyol) |Hadoop, HBase, Storm, Spark, Kafka |Striim es una plataforma de inteligencia e integración de datos de streaming de un extremo a otro que permite la ingesta continua, el procesamiento y el análisis de flujos de datos dispares. |
|[Jumbune Enterprise: aceleración de análisis de macrodatos](https://azuremarketplace.microsoft.com/marketplace/apps/impetus-infotech-india-pvt-ltd.impetus_jumbune) |Hadoop, Spark |A nivel general, Jumbune ayuda a las empresas mediante, 1. la aceleración del rendimiento de cargas de trabajo Hive, Java, Scala basadas en Tez, MapReduce y el motor de Spark; 2. la supervisión proactiva de clústeres de Hadoop; y 3. el establecimiento de administración de calidad de los datos en sistemas de archivos distribuidos. |
|[Kyligence Enterprise](https://azuremarketplace.microsoft.com/marketplace/apps/kyligence.kyligence) |Hadoop, HBase, Spark |Con tecnología Apache Kylin, Kyligence Enterprise permite la inteligencia empresarial sobre macrodatos. Como un motor OLAP empresarial en Hadoop, Kyligence Enterprise permite a los analistas empresariales diseñar una arquitectura de inteligencia empresarial en Hadoop con metodologías de inteligencia empresarial y almacenamiento de datos estándar del sector. |
|[Starburst Presto para Azure HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/starburstdatainc1579800938563.starburst-presto?tab=Overview) |Hadoop |Presto es un motor de consulta SQL distribuido, rápido y escalable. Diseñado para la separación de almacenamiento y proceso, Presto es perfecto para consultar datos en Azure Data Lake Storage, Azure Blob Storage, bases de datos SQL y NoSQL y otros orígenes de datos. |
|[StreamSets Data Collector para HDInsight en la nube](https://azuremarketplace.microsoft.com/marketplace/apps/streamsets.streamsets-data-collector-hdinsight) |Hadoop, HBase, Spark, Kafka |StreamSets Data Collector es un motor ligero y eficaz que transmite datos en tiempo real. Puede usar Data Collector para enrutar y procesar los datos de los flujos de datos. Incluye una licencia de prueba de 30 días. |
|[Trifacta Wrangler Enterprise](https://azuremarketplace.microsoft.com/marketplace/apps/trifacta.trifacta-db?tab=Overview) |Hadoop, Spark, HBase |Trifacta Wrangler Enterprise para HDInsight admite la limpieza y transformación de datos en toda la empresa para cualquier escala de datos. El costo de ejecutar Trifacta en Azure es una combinación de los costos de suscripción de Trifacta más los costos de infraestructura de Azure para las máquinas virtuales. |
|[Plataforma de datos Unifi](https://unifisoftware.com/platform/) |Hadoop, HBase, Storm, Spark |La plataforma de datos Unifi es un conjunto de herramientas de datos de autoservicio perfectamente integrado diseñado para permitir que el usuario de una empresa afronte los desafíos de datos que impulsan ingresos incrementales y reducen los costos o la complejidad operativa. |
|[Unraveldata APM](https://azuremarketplace.microsoft.com/marketplace/apps/unravel-data.unravel-app) |Spark |Aplicación Unravel Data para clúster Spark de HDInsight. |
|[Catálogo de datos controlado por inteligencia artificial de Waterline](https://azuremarketplace.microsoft.com/marketplace/apps/waterline_data.waterline_data) |Spark |Waterline clasifica, organiza y administra los datos mediante la inteligencia artificial para etiquetarlos de forma automática con términos empresariales. El catálogo alfabetizado empresarial de Waterline es un componente crítico y de éxito para análisis de autoservicio, cumplimiento y gobernanza, así como para iniciativas de administración de TI. |

Para las instrucciones proporcionadas en este artículo se usa Azure Portal. También puede exportar la plantilla de Azure Resource Manager desde el portal u obtener una copia de dicha plantilla de proveedores y usar Azure PowerShell y la CLI de Azure clásica para implementarla.  Consulte [Creación de clústeres de Apache Hadoop en HDInsight con plantillas de Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md).

## <a name="prerequisites"></a>Prerrequisitos
Si desea instalar aplicaciones de HDInsight en un clúster de HDInsight existente, debe tener un clúster de HDInsight. Para crearlo, consulte [Crear clúster](hadoop/apache-hadoop-linux-tutorial-get-started.md). También puede instalar aplicaciones de HDInsight al crear un clúster de HDInsight.

## <a name="install-applications-to-existing-clusters"></a>Instalación de aplicaciones en clústeres existentes
En el siguiente procedimiento, se muestra cómo instalar aplicaciones de HDInsight en un clúster de HDInsight existente.

**Instalación de una aplicación HDInsight**

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Desde el menú de la izquierda, vaya a **Todos los servicios** > **Analytics** > **HDInsight clusters** (Clústeres de HDInsight).
3. Seleccione un clúster de HDInsight de la lista.  Si no tiene ninguno, debe crearlo primero.  Consulte [Crear clúster](hadoop/apache-hadoop-linux-tutorial-get-started.md).
4. En la categoría **Configuración**, seleccione **Aplicaciones**. Puede ver una lista de las aplicaciones instaladas en la ventana principal. 
   
    ![Menú del portal de aplicaciones de HDInsight](./media/hdinsight-apps-install-applications/hdinsight-apps-portal-menu.png)
5. Seleccione **+Agregar** en el menú. Verá una lista de las aplicaciones disponibles.  Si **+Agregar** está atenuado, significa que no hay aplicaciones para esta versión del clúster de HDInsight.
   
    ![Aplicaciones disponibles para aplicaciones de HDInsight](./media/hdinsight-apps-install-applications/hdinsight-apps-list1.png)
6. Seleccione una de las aplicaciones disponibles y siga las instrucciones para aceptar los términos legales.

En las notificaciones del portal puede ver el estado de la instalación (seleccione el icono de campana de la parte superior del portal). Una vez instalada la aplicación, aparece en la lista Aplicaciones instaladas.

## <a name="install-applications-during-cluster-creation"></a>Instalación de aplicaciones durante la creación del clúster

Tiene la opción de instalar aplicaciones de HDInsight al crear un clúster. Durante el proceso, se instalan aplicaciones de HDInsight una vez que se haya creado el clúster y se encuentre en ejecución. Para instalar aplicaciones durante la creación del clúster mediante Azure Portal, en la pestaña **Configuración y precios**, seleccione **+ Agregar aplicación**.

![Aplicaciones de configuración de clústeres de Azure Portal](./media/hdinsight-apps-install-applications/azure-portal-cluster-configuration-applications.png)

## <a name="list-installed-hdinsight-apps-and-properties"></a>Lista de las aplicaciones de HDInsight instaladas y sus propiedades
El portal muestra una lista de las aplicaciones de HDInsight instaladas para un clúster y las propiedades de cada una.

**Lista de las aplicaciones de HDInsight y visualización de sus propiedades**

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Desde el menú de la izquierda, vaya a **Todos los servicios** > **Analytics** > **HDInsight clusters** (Clústeres de HDInsight).
3. Seleccione un clúster de HDInsight de la lista.
4. En la categoría **Configuración**, seleccione **Aplicaciones**. Puede ver una lista de las aplicaciones instaladas en la ventana principal. 
   
    ![Aplicaciones instaladas en aplicaciones de HDInsight](./media/hdinsight-apps-install-applications/hdinsight-apps-installed-apps-with-apps.png)
5. Seleccione una de las aplicaciones instaladas para mostrar la propiedad. En la propiedad se indican los siguientes elementos:

    |Propiedad | Descripción |
    |---|---|
    |Nombre de la aplicación |Nombre de la aplicación. |
    |Status |Estado de la aplicación. |
    |Página web |la dirección URL de la aplicación web que ha implementado en el nodo perimetral. La credencial coincide con las credenciales de usuario HTTP que ha configurado para el clúster. |
    |Punto de conexión SSH |puede usar SSH para conectarse al nodo perimetral. Las credenciales de SSH coinciden con las credenciales de usuario SSH que ha configurado para el clúster. Para más información, consulte [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md). |
    |Descripción | Descripción de la aplicación. |

6. Para eliminar una aplicación, haga clic con el botón derecho en ella y haga clic en **Eliminar** en el menú contextual.

## <a name="connect-to-the-edge-node"></a>Conexión con el nodo perimetral
Puede conectar con el nodo perimetral mediante HTTP y SSH. Encontrará la información de punto de conexión en el [portal](#list-installed-hdinsight-apps-and-properties). Para más información, consulte [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

Las credenciales del punto de conexión HTTP son las credenciales de usuario HTTP que ha configurado para el clúster de HDInsight; las del punto de conexión SSH son las credenciales SSH que ha configurado para el clúster de HDInsight.

## <a name="troubleshoot"></a>Solución de problemas
Consulte [Solución de problemas de instalación](hdinsight-apps-install-custom-applications.md#troubleshoot-the-installation).

## <a name="next-steps"></a>Pasos siguientes
* [Instalación de aplicaciones de HDInsight personalizadas](hdinsight-apps-install-custom-applications.md): aprenda a implementar en HDInsight una aplicación de HDInsight no publicada.
* [Publicación de aplicaciones de HDInsight en Azure Marketplace](hdinsight-apps-publish-applications.md): aprenda a publicar aplicaciones de HDInsight personalizadas en Azure Marketplace.
* [MSDN: Instalación de una aplicación HDInsight](https://msdn.microsoft.com/library/mt706515.aspx): Obtenga información sobre cómo definir las aplicaciones de HDInsight.
* [Personalización de clústeres de HDInsight mediante la acción de scripts (Linux)](hdinsight-hadoop-customize-cluster-linux.md): aprenda a usar acciones de script para instalar otras aplicaciones.
* [Creación de clústeres de Apache Hadoop basados en Linux en HDInsight con plantillas de Azure Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md): aprenda a llamar a plantillas de Resource Manager para crear clústeres de HDInsight.
* [Use empty edge nodes in HDInsight](hdinsight-apps-use-edge-node.md)(Utilización de nodos perimetrales vacíos en HDInsight): aprenda a usar un nodo perimetral vacío para acceder a los clústeres de HDInsight, probar aplicaciones de este y hospedarlas.

