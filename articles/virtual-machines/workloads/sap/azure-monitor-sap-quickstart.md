---
title: Implementación de Azure Monitor para soluciones de SAP con Azure Portal
description: Implementación de Azure Monitor para soluciones de SAP con Azure Portal
author: sameeksha91
ms.author: sakhare
ms.topic: how-to
ms.service: virtual-machines
ms.date: 08/17/2020
ms.openlocfilehash: 2c3b11ca0c2bb916a5a3fcaf50eb99c7db3aa8e7
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642562"
---
# <a name="deploy-azure-monitor-for-sap-solutions-with-azure-portal"></a>Implementación de Azure Monitor para soluciones de SAP con Azure Portal

Los recursos de Azure Monitor para soluciones de SAP se pueden crear mediante [Azure Portal](https://azure.microsoft.com/features/azure-portal). Este método proporciona una interfaz de usuario basada en explorador para implementar Azure Monitor para soluciones de SAP y configurar los proveedores.

## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-monitoring-resource"></a>Creación de recurso de supervisión

1. Seleccione **Azure Monitor para soluciones de SAP** en **Azure Marketplace**.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-1.png" alt-text="La imagen muestra la selección de la oferta Azure Monitor para soluciones de SAP en Azure Marketplace." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-1.png":::

2. En la pestaña **Aspectos básicos**, proporcione los valores necesarios. Si corresponde, puede usar un área de trabajo de Log Analytics existente.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-2.png" alt-text="Visualización de las opciones de configuración de Azure Portal." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-2.png":::

3. Al seleccionar una red virtual, asegúrese de que los sistemas que quiere supervisar son accesibles desde dentro de esa red virtual. 

   > [!IMPORTANT]
   > La selección de **Compartir** para el uso compartido de datos con Microsoft permite que nuestros equipos de soporte técnico proporcionen soporte adicional.

## <a name="configure-providers"></a>Configurar proveedores

### <a name="sap-hana-provider"></a>Proveedor de SAP HANA 

1. Seleccione la pestaña **Proveedor** para agregar los proveedores que quiere configurar. Puede agregar varios proveedores uno tras otro o agregarlos después de implementar el recurso de supervisión. 

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-3.png" alt-text="Muestra la pestaña de proveedor para agregar proveedores adicionales a Azure Monitor para soluciones de SAP." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-3.png":::

2. Seleccione **Agregar proveedor** y elija **SAP HANA** en el menú desplegable. 

3. Escriba la dirección IP privada del servidor HANA.

4. Escriba el nombre del inquilino de la base de datos que quiere usar. Puede elegir cualquier inquilino, pero se recomienda usar **SYSTEMDB** porque permite una matriz más amplia de áreas de supervisión. 

5. Escriba el número de puerto de SQL asociado con la base de datos HANA. El número de puerto debe tener el formato **[3]**  +  **[número_de_instancia]**  +  **[13]** o **[3]**  +  **[número_de_instancia]**  +  **[15]** . Por ejemplo, 30013 o 30015. 

6. Escriba el nombre de usuario de la base de datos que quiere usar. Asegúrese de que el usuario de la base de datos tiene asignados los roles de **supervisión** y **lectura de catálogo**. 

7. Cuando termine, seleccione **Agregar proveedor**. Siga agregando proveedores adicionales según sea necesario o seleccione **Revisar y crear** para completar la implementación.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-4.png" alt-text="Imagen de las opciones de configuración al agregar información del proveedor." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-4.png":::

### <a name="high-availability-cluster-pacemaker-provider"></a>Proveedor de clúster de alta disponibilidad (Pacemaker)

1. Seleccione **Clúster de alta disponibilidad (Pacemaker)** en la lista desplegable. 

   > [!IMPORTANT]
   > Para configurar el proveedor de clúster de alta disponibilidad (Pacemaker), asegúrese de que ha_cluster_provider esté instalado en cada nodo. Para más información, consulte el artículo sobre el [exportador de clúster de alta disponibilidad](https://github.com/ClusterLabs/ha_cluster_exporter#installation).

2. Escriba el punto de conexión de Prometheus con el formato http://IP:9664/metrics. 
 
3. Escriba el id. del sistema (SID), el nombre de host y el nombre del clúster.

4. Cuando termine, seleccione **Agregar proveedor**. Siga agregando proveedores adicionales según sea necesario o seleccione **Revisar y crear** para completar la implementación.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-5.png" alt-text="La imagen muestra las opciones relacionadas con el proveedor de Pacemaker del clúster de alta disponibilidad." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-5.png":::


### <a name="microsoft-sql-server-provider"></a>Proveedor de Microsoft SQL Server

1. Antes de agregar el proveedor de Microsoft SQL Server, debe ejecutar el script siguiente en SQL Server Management Studio para crear un usuario con los permisos adecuados necesarios para configurar el proveedor.

   ```sql
   USE [<Database to monitor>]
   DROP USER [AMS]
   GO
   USE [master]
   DROP USER [AMS]
   DROP LOGIN [AMS]
   GO
   CREATE LOGIN [AMS] WITH PASSWORD=N'<password>', DEFAULT_DATABASE=[<Database to monitor>], DEFAULT_LANGUAGE=[us_english], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
   CREATE USER AMS FOR LOGIN AMS
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GRANT CONNECT TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW ANY DEFINITION TO AMS
   GRANT EXEC ON xp_readerrorlog TO AMS
   GO
   USE [<Database to monitor>]
   CREATE USER [AMS] FOR LOGIN [AMS]
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GO
   ``` 

2. Seleccione **Agregar proveedor** y elija **Microsoft SQL Server** en el menú desplegable. 

3. Rellene los campos con la información asociada a su instancia de Microsoft SQL Server. 

4. Cuando termine, seleccione **Agregar proveedor**. Siga agregando proveedores adicionales según sea necesario o seleccione **Revisar y crear** para completar la implementación.

     :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-6.png" alt-text="La imagen muestra información relacionada con la incorporación del proveedor de Microsoft SQL Server." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-6.png":::

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [Azure Monitor para soluciones de SAP](azure-monitor-overview.md)
