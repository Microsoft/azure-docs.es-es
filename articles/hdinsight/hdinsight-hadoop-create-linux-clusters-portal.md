---
title: 'Creación de clústeres de Apache Hadoop con un explorador web: Azure HDInsight'
description: Aprenda a crear clústeres de Apache Hadoop, Apache HBase, Apache Storm o Apache Spark en HDInsight. Utilice un explorador web y Azure Portal.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 08/06/2020
ms.openlocfilehash: 1ba2f5e4b88ae6ae0ed15dbfbbc4fa5c55c45a77
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87874034"
---
# <a name="create-linux-based-clusters-in-hdinsight-by-using-the-azure-portal"></a>Creación de clústeres basados en Linux en HDInsight mediante Azure Portal

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

El Portal de Azure es una herramienta de administración basada en web para servicios y recursos hospedados en la nube de Microsoft Azure. En este artículo aprenderá a crear clústeres de Azure HDInsight basados en Linux mediante el portal. Puede encontrar más detalles en [Creación de clústeres de HDInsight](./hdinsight-hadoop-provision-linux-clusters.md).

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

El Portal de Azure expone la mayor parte de las propiedades del clúster. Mediante las plantillas de Azure Resource Manager puede ocultar una gran cantidad de detalles. Para más información, consulte [Creación de clústeres de Apache Hadoop en HDInsight con plantillas de Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-clusters"></a>Creación de clústeres

[!INCLUDE [secure-transfer-enabled-storage-account](../../includes/hdinsight-secure-transfer.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el menú superior, seleccione **+ Crear un recurso**.

    ![Creación de un clúster en Azure Portal](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-create-resource.png "Creación de un clúster en Azure Portal")

1. Seleccione **Analytics** > **Azure HDInsight** para ir a la página **Crear clúster de HDInsight**.

## <a name="basics"></a>Aspectos básicos

![Conceptos básicos de creación de clústeres de HDInsight](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-basics.png "Creación de un clúster en Azure Portal")

En la pestaña **Básico**, especifique la siguiente información:

|Propiedad |Descripción |
|---|---|
|Suscripción|En la lista desplegable, seleccione la suscripción de Azure que se usa para el clúster.|
|Resource group|En la lista desplegable, seleccione el grupo de recursos existente o seleccione **Crear**.|
|Nombre del clúster|Escriba un nombre único global.|
|Region|En la lista desplegable, seleccione una región donde crear el clúster.|
|Tipo de clúster|Haga clic en **Seleccionar tipo de clúster** para abrir una lista. En la lista, seleccione el tipo de clúster que desee. Los clústeres de HDInsight se presentan en distintos tipos. Estos corresponden a la carga de trabajo o la tecnología para los que el clúster está optimizado. No hay ningún método compatible para crear un clúster que combine varios tipos.|
|Versión|En la lista desplegable, seleccione una **versión**. Use la versión predeterminada si no sabe qué opción elegir. Para obtener más información, consulte [Versiones de clústeres de HDInsight](hdinsight-component-versioning.md).|
|Nombre de usuario de inicio de sesión del clúster|Proporcione el nombre de usuario; el valor predeterminado es **admin**.|
|Contraseña de inicio de sesión de clúster|Proporcione la contraseña.|
|Confirmar la contraseña de inicio de sesión del clúster|Reescriba la contraseña.|
|Nombre de usuario de Secure Shell (SSH)|Proporcione el nombre de usuario; el valor predeterminado es **sshuser**.|
|Uso de la contraseña de inicio de sesión del clúster para SSH|Si desea la misma contraseña de SSH que la contraseña de administrador que especificó anteriormente, active la casilla **Usar la contraseña de inicio de sesión del clúster para SSH**. De no ser así, especifique una **CONTRASEÑA** o **CLAVE PÚBLICA** para autenticar al usuario de SSH. Es recomendable usar una clave pública. Elija **Seleccionar** en la parte inferior para guardar la configuración de las credenciales.  Para más información, consulte [Connect to HDInsight (Apache Hadoop) by using SSH](hdinsight-hadoop-linux-use-ssh-unix.md) (Conexión a través de SSH con HDInsight [Apache Hadoop]).|

Seleccione **Siguiente: Almacenamiento>>** para avanzar a la pestaña siguiente.

## <a name="storage"></a>Storage

> [!WARNING] 
> A partir del 15 de junio de 2020 los clientes no podrán crear una nueva entidad de servicio mediante HDInsight. Consulte [Creación de una entidad de servicio y certificados](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal) mediante Azure Active Directory.

![Almacenamiento en creación de clústeres de HDInsight](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-storage.png "Creación de un clúster en Azure Portal: almacenamiento")

### <a name="primary-storage"></a>Almacenamiento principal

En la lista desplegable de **Tipo de almacenamiento principal**, seleccione el tipo de almacenamiento predeterminado. Los campos siguientes que va a completar variarán en función de la selección. Para **Azure Storage**:

1. En **Método de selección**, elija **Seleccionar un elemento de la lista** o **Usar clave de acceso**.
    * En **Seleccionar un elemento de la lista**, seleccione la **Cuenta de almacenamiento principal** en la lista desplegable o seleccione **Crear**.
    * En **Usar clave de acceso**, escriba su **Nombre de la cuenta de almacenamiento**. A continuación, proporcione la **Clave de acceso**.

1. En **Contenedor**, acepte el valor predeterminado o escriba uno nuevo.

### <a name="additional-azure-storage"></a>Azure Storage adicional

Opcional: Seleccione **Agregar Azure Storage** para un almacenamiento de clúster adicional. No se admite el uso de una cuenta de almacenamiento adicional en una región diferente a la del clúster de HDInsight.

### <a name="metastore-settings"></a>Configuración de metastores

Opcional: Especifique una instancia existente de SQL Database para guardar los metadatos de Apache Hive, Apache Oozie o Apache Ambari fuera del clúster. La instancia de Azure SQL Database usada para el metastore debe permitir la conectividad con otros servicios de Azure, incluido Azure HDInsight. Al crear una instancia de Metastore, no asigne nombre a una base de datos con guiones. Estos caracteres podrían provocar que el proceso de creación del clúster produzca errores.

> [!IMPORTANT]
> En el caso de las formas de clúster que admiten metastores, el metastore predeterminado proporciona una instancia de Azure SQL Database con un **límite de 5 DTU de nivel básico (no actualizable)** . Esto resulta adecuado para la realización de pruebas básicas. En el caso de las cargas de trabajo de producción o de gran volumen, se recomienda migrar a un metastore externo.

Seleccione **Siguiente: Seguridad y redes >>** para avanzar a la pestaña siguiente.

## <a name="security--networking"></a>Seguridad y redes

![Creación de clúster de HDInsight en seguridad y redes](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-security-networking.png "Creación de clúster de HDInsight en seguridad y redes")

En la pestaña **Seguridad y redes**, especifique la información siguiente:

|Propiedad |Descripción |
|---|---|
|Paquete de seguridad de la empresa|Opcional: Active la casilla para usar **Enterprise Security Package**. Para obtener más información, consulte [Configuración de clúster de HDInsight con Enterprise Security Package mediante Azure Active Directory Domain Services](./domain-joined/apache-domain-joined-configure-using-azure-adds.md).|
|TLS|Opcional: Seleccione una versión de TLS de la lista desplegable. Para más información, consulte [Seguridad de la capa de transporte](./transport-layer-security.md).|
|Virtual network|Opcional: Seleccione una red virtual y una subred existentes de la lista desplegable. Para obtener más información, consulte [Planeamiento de la implementación de una red virtual para clústeres de Azure HDInsight](hdinsight-plan-virtual-network-deployment.md). El artículo incluye los requisitos de configuración específicos de la red virtual.|
|Configuración de cifrado de disco|Opcional: Active la casilla para usar el cifrado. Para obtener más información, consulte [Cifrado de disco mediante claves administradas por el cliente](./disk-encryption.md).|
|Proxy de REST de Kafka|Esta opción solo está disponible para el tipo de clúster Kafka. Para más información, consulte [Uso de un proxy de REST](./kafka/rest-proxy.md).|
|Identidad|Opcional: Seleccione una identidad de servicio asignada por el usuario existente de la lista desplegable. Para obtener más información, consulte [Identidades administradas en Azure HDInsight](./hdinsight-managed-identities.md).|

Seleccione **Siguiente: Configuración y precios >>** para avanzar a la pestaña siguiente.

## <a name="configuration--pricing"></a>Configuración y precios

![Creación de un clúster de HDInsight en configuración](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-configuration.png "Pestaña Configuración y precios")

En la pestaña **Configuración y precios**, especifique la información siguiente:

|Propiedad |Descripción |
|---|---|
|+ Agregar aplicación|Opcional: Seleccione las aplicaciones que desee. De Microsoft, de proveedores de software independientes (ISV) o también puede desarrollarlas. Para más información, consulte [Instalación de aplicaciones durante la creación del clúster](hdinsight-apps-install-applications.md#install-applications-during-cluster-creation).|
|Tamaño del nodo|Opcional: Seleccione un nodo de otro tamaño.|
|Número de nodos|Opcional: Especifique el número de nodos para el tipo de nodo especificado. Si planea crear más de 32 nodos de trabajo, seleccione un tamaño de nodo principal con al menos ocho núcleos y 14 GB de RAM. Planee los nodos al crear el clúster o mediante el escalado del clúster después de la creación.|
|Habilitar escalado automático|Opcional: Active la casilla para habilitar la característica. Para más información, consulte [Escalado automático de clústeres de Azure HDInsight](./hdinsight-autoscale-clusters.md).|
|+ Agregar acción de script|Opcional: Esta opción funciona si quiere usar un script personalizado para personalizar un clúster, conforme se crea el clúster. Para más información sobre las acciones de script, consulte [Personalización de clústeres de HDInsight basados en Linux mediante acciones de script](hdinsight-hadoop-customize-cluster-linux.md).|

Seleccione **Revisar y crear >>** para validar la configuración del clúster y avanzar a la pestaña final.

## <a name="review--create"></a>Revisar y crear

![Resumen de creación de clústeres de HDInsight](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-review-create-hadoop.png "Especificación del número de nodos del clúster")

Revise la configuración. Seleccione **Crear** para crear el clúster.

El clúster tarda algo de tiempo en crearse, normalmente unos 20 minutos. Supervise las **notificaciones** para comprobar el proceso de aprovisionamiento.

## <a name="post-creation"></a>Después de la creación

Una vez completado el proceso de creación, seleccione **Ir al recurso** en la notificación **Implementación correcta**. La ventana del clúster ofrece la siguiente información.

![Información general de HDI en Azure Portal](./media/hdinsight-hadoop-create-linux-clusters-portal/hdinsight-create-cluster-completed.png "Propiedades de clúster")

Algunos de los iconos de la ventana se explican de la siguiente manera:

|Propiedad | Descripción |
|---|---|
|Información general|Proporciona toda la información esencial sobre el clúster. Algunos ejemplos son el nombre, el grupo de recursos al que pertenece, la ubicación, el sistema operativo y la dirección URL del panel del clúster.|
|Paneles de clúster|Le dirige al portal de Ambari asociado al clúster.|
|SSH e inicio de sesión del clúster|Proporciona la información necesaria para acceder al clúster mediante SSH.|
|Eliminar|Elimina el clúster de HDInsight.|

## <a name="delete-the-cluster"></a>Eliminación del clúster

Consulte [Eliminación de un clúster de HDInsight con el explorador, PowerShell o la CLI de Azure](./hdinsight-delete-cluster.md).

## <a name="troubleshoot"></a>Solución de problemas

Si experimenta problemas con la creación de clústeres de HDInsight, consulte los [requisitos de control de acceso](./hdinsight-hadoop-customize-cluster-linux.md#access-control).

## <a name="next-steps"></a>Pasos siguientes

Ha creado correctamente un clúster de HDInsight. Ahora puede aprender a trabajar con el clúster.

* [Uso de Apache Hive con HDInsight](hadoop/hdinsight-use-hive.md)
* [Introducción a Apache HBase en HDInsight](hbase/apache-hbase-tutorial-get-started-linux.md)
* [Personalización de clústeres de HDInsight basados en Linux mediante acciones de script](hdinsight-hadoop-customize-cluster-linux.md)
