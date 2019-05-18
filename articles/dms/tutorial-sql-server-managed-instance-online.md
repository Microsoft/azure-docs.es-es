---
title: 'Tutorial: Uso de Azure Database Migration Service para realizar una migración en línea de SQL Server a una instancia administrada de Azure SQL Database | Microsoft Docs'
description: Aprenda a realizar una migración en línea de SQL Server local a una instancia administrada de Azure SQL Database mediante Azure Database Migration Service.
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
ms.openlocfilehash: 12a0ebeebbc3bdc205816c5534f59b1385cecb12
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413751"
---
# <a name="tutorial-migrate-sql-server-to-an-azure-sql-database-managed-instance-online-using-dms"></a>Tutorial: Migración de SQL Server a una instancia administrada de Azure SQL Database en línea mediante DMS

Azure Database Migration Service se puede usar para migrar las bases de datos de una instancia de SQL Server local a una [instancia administrada de Azure SQL Database](../sql-database/sql-database-managed-instance.md) con un tiempo de inactividad mínimo. Para ver otros métodos que requieren realizar acciones manuales, consulte [Migración de una instancia de SQL Server a una instancia administrada de Azure SQL Database](../sql-database/sql-database-managed-instance-migrate.md).

En este tutorial, migrará la base de datos **Adventureworks2012** desde una instancia local de SQL Server a una instancia administrada de Azure SQL Database con un tiempo de inactividad mínimo mediante Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> - Crear una instancia de Azure Database Migration Service.
> - Crear un proyecto de migración e iniciar una migración en línea mediante Azure Database Migration Service.
> - Supervisar la migración
> - Realizar la transición de la migración cuando esté listo.

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium.
> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se describe una migración en línea de SQL Server a una instancia administrada de Azure SQL Database. Para migraciones sin conexión, consulte [Migración de SQL Server a una instancia administrada de Azure SQL Database sin conexión mediante DMS](tutorial-sql-server-to-managed-instance.md).

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

- Crear una instancia de Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). [Conozca las topologías de red para migrar a una instancia administrada de Azure SQL Database con Azure Database Migration Service](https://aka.ms/dmsnetworkformi). Para más información acerca de cómo crear una red virtual, consulte la [documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/) y, concretamente, los artículos de inicio rápido que proporcionan instrucciones paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de redes con Microsoft, agregue los siguientes [puntos de conexión](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) de servicio a la subred en la que se aprovisionará el servicio:
    > - Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > - Punto de conexión de Storage
    > - Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque la instancia de Azure Database Migration Service carece de conectividad a internet.

- Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445, 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte el artículo [Planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg).
- Configure el [Firewall de Windows para acceder al motor de base de datos de origen](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
- Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda a la instancia de SQL Server de origen que, de manera predeterminada, es el puerto TCP 1433.
- Si se ejecutan varias instancias con nombre de SQL Server con puertos dinámicos, puede ser conveniente habilitar el servicio SQL Browser y permitir el acceso al puerto UDP 1434 mediante los firewalls para que Azure Database Migration Service pueda conectarse a una instancia con nombre en el servidor de origen.
- Si va a usar un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración, así como archivos a través del puerto SMB 445.
- Cree una instancia administrada de Azure SQL Database mediante los pasos que se describen en el artículo [Creación de una instancia administrada de Azure SQL Database en Azure Portal](https://aka.ms/sqldbmi).
- Asegúrese de que los inicios de sesión usados para conectar la instancia de SQL Server de origen y la instancia administrada de destino sean miembros del rol de servidor sysadmin.
- Especifique un recurso compartido de red SMB que contenga todos los archivos de copia de seguridad de la base de datos completa y los posteriores archivos de copia de seguridad del registro de transacciones que puede usar Azure Database Migration Service para la migración de la base de datos.
- Asegúrese de que la cuenta de servicio que ejecuta la instancia de SQL Server de origen tiene privilegios de escritura en el recurso compartido de red que haya creado, y que la cuenta del equipo del servidor de origen tiene acceso de lectura y escritura para el mismo recurso compartido.
- Anote un usuario de Windows (y su contraseña) que tenga privilegios de control total sobre el recurso compartido de red que creó anteriormente. Azure Database Migration Service suplanta la credencial de usuario para cargar los archivos de copia de seguridad en el contenedor de Azure Storage para la operación de restauración.
- Cree un identificador de aplicación de Azure Active Directory que genere la clave del identificador de aplicación que puede usar el servicio DMS para conectarse a la instancia administrada de Azure Database y al contenedor de Azure Storage. Para más información, consulte el artículo [Uso del portal para crear una aplicación de Azure Active Directory y una entidad de servicio con acceso a los recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal).
- Cree o anote el **nivel de rendimiento estándar**, cuenta de almacenamiento de Azure, que permite que el servicio DMS cargue los archivos de copia de seguridad de la base de datos a y los use para migrar las bases de datos.  Asegúrese de crear la cuenta de Azure Storage en la misma región que el servicio DMS creado.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

    ![Mostrar suscripciones en el portal](media/tutorial-sql-server-to-managed-instance-online/portal-select-subscriptions.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-sql-server-to-managed-instance-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-sql-server-to-managed-instance-online/portal-register-resource-provider.png)   

## <a name="create-an-azure-database-migration-service-instance"></a>Creación de una instancia de Azure Database Migration Service

1. En Azure Portal, seleccione **+ Crear un recurso**, busque **Azure Database Migration Service** y, a continuación, seleccione **Azure Database Migration Service** en la lista desplegable.

     ![Azure Marketplace](media/tutorial-sql-server-to-managed-instance-online/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-sql-server-to-managed-instance-online/dms-create1.png)

3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que desea crear la instancia de DMS.

5. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de origen de SQL Server y a la instancia administrada de Azure SQL Database de destino.

    Para obtener más información sobre cómo crear una VNet en Azure Portal, consulte el artículo [Create a virtual network using the Azure portal](https://aka.ms/DMSVnet) (Crear una red virtual con Azure Portal).

    Para obtener detalles adicionales, consulte el artículo [Topologías de red para migraciones a la instancia administrada de Azure SQL Database con Azure Database Migration Service](https://aka.ms/dmsnetworkformi).

6. Seleccione una SKU del plan de tarifa Premium.

    > [!NOTE]
    > Las migraciones en línea solo se admiten cuando se usa el nivel Premium.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Creación de servicio DMS](media/tutorial-sql-server-to-managed-instance-online/dms-create-service3.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear una instancia del servicio, búsquela en Azure Portal, ábrala y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

    ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-sql-server-to-managed-instance-online/dms-search.png)

2. En la pantalla **Azure Database Migration Service**, busque el nombre de la instancia que ha creado y selecciónela.

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).

4. En la pantalla **New migration project** (Nuevo proyecto de migración), especifique el nombre del proyecto; en el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **SQL Server**; en el cuadro de texto **Target server type** (Tipo de servidor de destino), seleccione **Azure SQL Database Managed Instance** (Instancia administrada de Azure SQL Database) y, finalmente, en **Choose type of activity** (Elegir tipo de actividad), seleccione **Online data migration** (Migración de datos en línea).

   ![Creación de proyecto DMS](media/tutorial-sql-server-to-managed-instance-online/dms-create-project3.png)

5. Seleccione **Crear y ejecutar una actividad** para crear el proyecto.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Detalles del origen de la migración**, especifique los detalles de conexión de SQL Server de origen.

2. Si no ha instalado ningún certificado de confianza en el servidor, seleccione la casilla **Certificado de servidor de confianza**.

    Si no hay ningún certificado de confianza instalado, SQL Server genera un certificado autofirmado cuando se inicia la instancia. Este certificado se usa para cifrar las credenciales de las conexiones del cliente.

    > [!CAUTION]
    > Las conexiones SSL cifradas mediante un certificado autofirmado no proporcionan una gran seguridad. Son susceptibles de sufrir ataques de tipo "Man in the middle". No se debe confiar en SSL mediante certificados autofirmados en un entorno de producción o en servidores que están conectados a Internet.

   ![Detalles del origen](media/tutorial-sql-server-to-managed-instance-online/dms-source-details2.png)

3. Seleccione **Guardar**.

4. En la pantalla **Seleccionar las bases de datos de origen**, seleccione la base de datos **Adventureworks2012** para la migración.

   ![Selección de las bases de datos de origen](media/tutorial-sql-server-to-managed-instance-online/dms-source-database1.png)

    > [!IMPORTANT]
    > Si usa SQL Server Integration Services (SSIS), DMS no admite actualmente la migración de la base de datos de catálogo de los proyectos o paquetes de SSIS (SSISDB) desde SQL Server hasta la instancia administrada de Azure SQL Database. Sin embargo, puede aprovisionar SSIS en Azure Data Factory (ADF) y volver a implementar los proyectos o paquetes de SSIS en la SSISDB de destino hospedada por la instancia administrada de Azure SQL Database. Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages).

5. Seleccione **Guardar**.

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. En la pantalla **Detalles del destino de la migración**, especifique el **identificador de la aplicación** y la **clave** que la instancia de DMS puede usar para conectarse a la instancia de destino de la instancia administrada de Azure SQL Database y la cuenta de Azure Storage.

    Para más información, consulte el artículo [Uso del portal para crear una aplicación de Azure Active Directory y una entidad de servicio con acceso a los recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal).

2. Seleccione la **suscripción** que contiene la instancia de destino de la instancia administrada de Azure SQL Database y, después, seleccione la instancia de destino.

    Si aún no ha aprovisionado la instancia administrada de Azure SQL Database, seleccione el [vínculo](https://aka.ms/SQLDBMI) que le ayudará a aprovisionar la instancia. Cuando la instancia administrada de Azure SQL Database esté lista, regrese a este proyecto específico para ejecutar la migración.

3. Especifique los valores de **Usuario de SQL** y **Contraseña** para conectarse a la instancia administrada de Azure SQL Database.

    ![Selección del destino](media/tutorial-sql-server-to-managed-instance-online/dms-target-details3.png)

4. Seleccione **Guardar**.

## <a name="select-source-databases"></a>Selección de las bases de datos de origen

1. En la pantalla **Seleccionar las bases de datos de origen**, seleccione las bases de datos de origen que quiera migrar.

    ![Selección de las bases de datos de origen](media/tutorial-sql-server-to-managed-instance-online/dms-select-source-databases2.png)

2. Seleccione **Guardar**.

## <a name="configure-migration-settings"></a>Configuración de valores de migración

1. En la pantalla **Configurar los valores de la migración**, proporcione los detalles siguientes:

    | | |
    |--------|---------|
    |**Recurso compartido de la ubicación de la red SMB** | El recurso compartido de red SMB que contiene los archivos de la copia de seguridad de la base de datos completa y los archivos de la copia de seguridad del registro de transacciones que Azure Database Migration Service puede usar para la migración. La cuenta de servicio que ejecuta la instancia de SQL Server de origen debe tener privilegios de lectura y escritura en este recurso compartido de red. Proporcione un FQDN o direcciones IP del servidor en el recurso compartido de red como, por ejemplo, "\\\servername.domainname.com\backupfolder" o "\\\IP address\backupfolder".|
    |**Nombre de usuario** | Asegúrese de que el usuario de Windows tiene privilegio de control total sobre el recurso compartido de red que especificó anteriormente. Azure Database Migration Service suplanta la credencial de usuario para cargar los archivos de copia de seguridad en el contenedor de almacenamiento de Azure para la operación de restauración. |
    |**Contraseña** | Contraseña del usuario. |
    |**Suscripción de la cuenta de Azure Storage** | Seleccione la suscripción que contiene la cuenta de Azure Storage. |
    |**Cuenta de Azure Storage** | Seleccione la cuenta de Azure Storage en la que DMS puede cargar los archivos de copia de seguridad del recurso compartido de red SMB y úsela para la migración de base de datos.  Para que el rendimiento de la carga de archivos sea óptimo, se recomienda seleccionar la cuenta de almacenamiento en la misma región que el servicio DMS. |

    ![Configuración de valores de migración](media/tutorial-sql-server-to-managed-instance-online/dms-configure-migration-settings4.png)

2. Seleccione **Guardar**.

## <a name="review-the-migration-summary"></a>Examen del resumen de la migración

1. En la pantalla **Migration summary** (Resumen de migración), en el cuadro de texto **Nombre de actividad**, especifique un nombre para la actividad de migración.

2. Revise y compruebe los detalles relacionados con el proyecto de migración.

    ![Resumen del proyecto de migración](media/tutorial-sql-server-to-managed-instance-online/dms-project-summary3.png)

## <a name="run-and-monitor-the-migration"></a>Ejecución y supervisión de la migración

1. Seleccione **Ejecutar migración**.

2. En la pantalla de actividad de migración, seleccione **Actualizar** para actualizar la pantalla.

   ![Actividad de migración en curso](media/tutorial-sql-server-to-managed-instance-online/dms-monitor-migration2.png)

    Puede ampliar aún más las bases de datos y las categorías de inicio de sesión para supervisar el estado de migración de los respectivos objetos de servidor.

   ![Actividad de migración en curso](media/tutorial-sql-server-to-managed-instance-online/dms-monitor-migration-extend2.png)

## <a name="performing-migration-cutover"></a>Realización de la migración total

Una vez restaurada la copia de seguridad de la base de datos completa en la instancia de destino de la instancia administrada de Azure SQL Database, la base de datos está disponible para realizar una migración total.

1. Cuando esté listo para completar la migración de la base de datos en línea, seleccione **Iniciar transición**.

2. Detenga todo el tráfico de entrada a las bases de datos de origen.

3. Realice la [copia del final del registro], proporcione el archivo de copia de seguridad en el recurso compartido de red SMB y espere a que se restaure la copia de seguridad del registro de transacciones final.

    En ese momento, verá que el valor de **Cambios pendientes** es 0.

4. Seleccione **Confirmar** y, después, **Aplicar**.

    ![Preparación de la migración completa](media/tutorial-sql-server-to-managed-instance-online/dms-complete-cutover.png)

5. Cuando el estado de la migración de la base de datos muestre **Completado**, conecte las aplicaciones a la nueva instancia destino de la instancia administrada de Azure SQL Database.

    ![Migración completa](media/tutorial-sql-server-to-managed-instance-online/dms-cutover-complete.png)

## <a name="next-steps"></a>Pasos siguientes

- Para consultar un tutorial que muestra cómo migrar una base de datos a una instancia administrada mediante el comando T-SQL RESTORE, consulte [Restauración de una copia de seguridad para una instancia administrada mediante el comando Restore](../sql-database/sql-database-managed-instance-restore-from-backup-tutorial.md).
- Para más información acerca de la instancia administrada, consulte [¿Qué es una instancia administrada?](../sql-database/sql-database-managed-instance.md).
- Para más información sobre cómo conectar las aplicaciones a una instancia administrada, consulte [Conexión de aplicaciones](../sql-database/sql-database-managed-instance-connect-app.md).
