---
title: 'Tutorial: Uso de Azure Database Migration Service para migrar MongoDB a la API de Azure Cosmos DB para MongoDB sin conexión | Microsoft Docs'
description: Aprenda a migrar desde el entorno local de MongoDB a la API de Azure Cosmos DB para MongoDB sin conexión mediante Azure Database Migration Service.
services: dms
author: HJToland3
ms.author: jtoland
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc, tutorial
ms.topic: article
ms.date: 05/08/2019
ms.openlocfilehash: ad0d990554d9ff49bed3e9da7097c87c06c7152f
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415532"
---
# <a name="tutorial-migrate-mongodb-to-azure-cosmos-dbs-api-for-mongodb-offline-using-dms"></a>Tutorial: Migración de MongoDB a la API de Azure Cosmos DB para MongoDB sin conexión mediante DMS
Puede usar Azure Database Migration Service para realizar una migración sin conexión (de una sola vez) de las bases de datos desde una instancia local o en la nube de MongoDB a la API de Azure Cosmos DB para MongoDB.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración

En este tutorial, migrará un conjunto de datos de MongoDB hospedado en una máquina virtual de Azure a la API de Azure Cosmos DB para MongoDB mediante Azure Database Migration Service. Si no tiene un origen de MongoDB ya configurado, vea el artículo [Instalación y configuración de MongoDB en una máquina virtual Windows en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/install-mongodb).

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

* [Realizar los pasos previos a la migración](../cosmos-db/mongodb-pre-migration.md) como son estimar el rendimiento, elegir una clave de partición y seleccionar la directiva de indexación.
* [Crear una cuenta para la API de Azure Cosmos DB para MongoDB](https://ms.portal.azure.com/#create/Microsoft.DocumentDB).
* Crear una instancia de Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). Para más información acerca de cómo crear una red virtual, consulte la [documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/) y, concretamente, los artículos de inicio rápido que proporcionan instrucciones paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de redes con Microsoft, agregue los siguientes [puntos de conexión](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) de servicio a la subred en la que se aprovisionará el servicio:

    > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > * Punto de conexión de Storage
    > * Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque la instancia de Azure Database Migration Service carece de conectividad a internet.

* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445 y 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte el artículo [Planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda al servidor de MongoDB de origen que, de manera predeterminada, es el puerto TCP 27017.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-mongodb-to-cosmosdb/portal-select-subscription1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-mongodb-to-cosmosdb/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-mongodb-to-cosmosdb/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>Creación de una instancia

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-mongodb-to-cosmosdb/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service. 

5. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de MongoDB de origen y a la cuenta de Azure Cosmos DB de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual mediante Azure Portal](https://aka.ms/DMSVnet).

6. Seleccione un plan de tarifa.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb/dms-settings2.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

      ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).

4. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **MongoDB**, en el cuadro de texto **Target server type** (Tipo de servidor de destino), seleccione **CosmosDB (API de MongoDB)** y, por último, en **Elegir tipo de actividad**, seleccione **Migración de datos sin conexión**. 

    ![Creación de un proyecto de Database Migration Service](media/tutorial-mongodb-to-cosmosdb/dms-create-project.png)

5. Seleccione **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Detalles del origen**, especifique los detalles de conexión del servidor de MongoDB de origen.

   También puede usar el modo de cadena de conexión y proporcionar una ubicación para un contenedor de archivos de almacén de blobs en el que haya volcado los datos de la colección que se van a migrar.

   > [!NOTE]
   > Azure Database Migration Service también puede migrar documentos BSON o JSON a colecciones de la API de Azure Cosmos DB para MongoDB.

   También puede usar la dirección IP en los casos en que no sea posible la resolución de nombres de DNS.

   ![Especificación de los detalles de origen](media/tutorial-mongodb-to-cosmosdb/dms-specify-source.png)

2. Seleccione **Guardar**.

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. En la pantalla **Detalles del destino de la migración**, especifique los detalles de conexión de la cuenta de Azure Cosmos DB de destino, que es la cuenta de la API de Azure Cosmos DB para MongoDB aprovisionada previamente a la que se van a migrar los datos de MongoDB.

    ![Especificación de los detalles de destino](media/tutorial-mongodb-to-cosmosdb/dms-specify-target.png)

2. Seleccione **Guardar**.

## <a name="map-to-target-databases"></a>Asignación a bases de datos de destino

1. En la pantalla **Map to target databases** (Asignar a bases de datos de destino), asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    Si la cadena **Crear** aparece junto al nombre de la base de datos, indica que Azure Database Migration Service no encontró la base de datos de destino, y el servicio creará la base de datos automáticamente.

    En este momento de la migración, puede [aprovisionar rendimiento](https://docs.microsoft.com/azure/cosmos-db/set-throughput). En Cosmos DB, puede aprovisionar rendimiento a nivel de base de datos o individualmente para cada colección. El rendimiento se mide en [unidades de solicitud](https://docs.microsoft.com/azure/cosmos-db/request-units) (RU). Obtenga más información sobre los [precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

    ![Asignación a bases de datos de destino](media/tutorial-mongodb-to-cosmosdb/dms-map-target-databases.png)

2. Seleccione **Guardar**.
3. En la pantalla **Configuración de colecciones**, expanda la lista de colecciones y, a continuación, revise la lista de las colecciones que se van a migrar.

    Tenga en cuenta que Azure Database Migration Service selecciona automáticamente todas las colecciones existentes en la instancia de MongoDB de origen que no existen en la cuenta de Azure Cosmos DB de destino. Si quiere volver a migrar las colecciones que ya incluyen datos, necesita seleccionar explícitamente las colecciones de esta hoja.

    Puede especificar la cantidad de unidades de solicitud que desea que las colecciones usen. Azure Database Migration Service sugiere valores predeterminados inteligentes en función del tamaño de la colección.

    También puede especificar una clave de partición para aprovechar las ventajas de la [creación de particiones en Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/partitioning-overview) para una escalabilidad óptima. No olvide revisar los [procedimientos recomendados para seleccionar una clave de partición](https://docs.microsoft.com/azure/cosmos-db/partitioning-overview#choose-partitionkey).

    ![Selección de tablas de colecciones](media/tutorial-mongodb-to-cosmosdb/dms-collection-setting.png)

4. Seleccione **Guardar**.

5. En la pantalla **Migration summary** (Resumen de migración), en el cuadro de texto **Nombre de actividad**, especifique un nombre para la actividad de migración.

    ![Resumen de migración](media/tutorial-mongodb-to-cosmosdb/dms-migration-summary.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

    Aparecerá la ventana de actividad de migración y el **estado** de la actividad será **Sin iniciar**.

    ![Estado de la actividad](media/tutorial-mongodb-to-cosmosdb/dms-activity-status.png)

## <a name="monitor-the-migration"></a>Supervisión de la migración

* En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **Completado**.

   > [!NOTE]
   > Puede seleccionar la actividad para obtener los detalles de las métricas de migración a nivel de base de datos y de colección.

    ![Estado de actividad completado](media/tutorial-mongodb-to-cosmosdb/dms-activity-completed.png)

## <a name="verify-data-in-cosmos-db"></a>Verificación de datos en Cosmos DB

* Después de finalizar la migración, podrá consultar la cuenta de Azure Cosmos DB para comprobar que todas las colecciones se han migrado correctamente.

    ![Estado de actividad completado](media/tutorial-mongodb-to-cosmosdb/dms-cosmosdb-data-explorer.png)

## <a name="post-migration-optimization"></a>Optimización posterior a la migración

Después de migrar los datos almacenados en la base de datos de MongoDB a la API de Azure Cosmos DB para MongoDB, puede conectarse a Azure Cosmos DB y administrar los datos. También puede realizar otros pasos de optimización posteriores a la migración, como optimizar la directiva de indexación, actualizar el nivel de coherencia predeterminado o configurar la distribución global de la cuenta de Azure Cosmos DB. Para más información, consulte el artículo [Optimización posterior a la migración](../cosmos-db/mongodb-post-migration.md). 

## <a name="additional-resources"></a>Recursos adicionales

* [Información sobre el servicio Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)

## <a name="next-steps"></a>Pasos siguientes

* Revise las directrices de migración en la [Guía de migración de bases de datos](https://datamigration.microsoft.com/) para consultar escenarios adicionales.
