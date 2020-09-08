---
title: Configuración de SQL Data Sync
description: En este tutorial se muestra cómo configurar SQL Data Sync para Azure.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 01/14/2019
ms.openlocfilehash: bd1362db2e70d4f9f46d80b00805856e08aedac4
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87987348"
---
# <a name="tutorial-set-up-sql-data-sync-between-databases-in-azure-sql-database-and-sql-server"></a>Tutorial: Configuración de SQL Data Sync entre Azure SQL Database y SQL Server
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, aprenderá a configurar SQL Data Sync mediante la creación de un grupo de sincronización que contiene instancias de Azure SQL Database y de SQL Server. El grupo de sincronización está configurado de manera personalizada y se sincroniza según el programa establecido.

En este tutorial se asume que como mínimo tiene algo de experiencia previa con SQL Database y con SQL Server.

Para obtener información general sobre SQL Data Sync, vea [Sincronización de datos entre bases de datos locales y de la nube con SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md).

Para ejemplos de PowerShell sobre cómo configurar SQL Data Sync, vea [Cómo sincronizar datos entre varias bases de datos en SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md) o [entre varias bases de datos de Azure SQL Database y SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md).

> [!IMPORTANT]
> SQL Data Sync **no** admite en este momento la Instancia administrada de Azure SQL.

## <a name="create-sync-group"></a>Creación de un grupo de sincronización

1. Vaya a [Azure Portal](https://portal.azure.com) para encontrar su base de datos en SQL Database. Busque y seleccione **base de datos SQL**.

    ![Búsqueda de bases de datos, Microsoft Azure Portal](./media/sql-data-sync-sql-server-configure/search-for-sql-databases.png)

1. Seleccione la base de datos que desea usar como base de datos central para Data Sync.

    ![Selección de la lista de bases de datos, Microsoft Azure Portal](./media/sql-data-sync-sql-server-configure/select-sql-database.png)

    > [!NOTE]
    > La base de datos central es el punto de conexión central de la topología de sincronización, donde un grupo de sincronización tiene varios puntos de conexión de base de datos. Las demás bases de datos miembros con puntos de conexión del grupo de sincronización se sincronizan con esta.

1. En el menú de la **base de datos SQL** para la base de datos seleccionada, seleccione **Sincronizar con otras bases de datos**.

    ![Sincronización con otras bases de datos, Microsoft Azure Portal](./media/sql-data-sync-sql-server-configure/sync-to-other-databases.png)

1. En la página **Sincronizar con otras bases de datos**, seleccione **Nuevo grupo de sincronización**. La página **Nuevo grupo de sincronización** se abre con el paso 1, con el paso 1,**Crear grupo de sincronización**, resaltado.

   ![Configuración del paso 1](./media/sql-data-sync-sql-server-configure/stepone.png)

   En la página **Crear grupo de sincronización de datos**, cambie la siguiente configuración:

   | Configuración                        | Descripción |
   | ------------------------------ | ------------------------------------------------- |
   | **Nombre del grupo de sincronización** | Escriba el nombre del nuevo grupo de sincronización. Este nombre es distinto del nombre de la base de datos. |
   | **Base de datos de metadatos de sincronización** | Elija esta opción para crear una base de datos (recomendación) o para usar una existente.<br/><br/>Si ha elegido **Nueva base de datos**, seleccione **Crear nueva base de datos**. Después, en la página **SQL Database**, asigne un nombre a la nueva base de datos, configúrela y seleccione **Aceptar**.<br/><br/>Si ha elegido **Usar base de datos existente**, selecciónela de la lista. |
   | **Sincronización automática** | Utilice **Activado** o **Desactivado**.<br/><br/>Si ha elegido **Activado**, escriba un número y seleccione **Segundos**, **Minutos**, **Horas** o **Días** en la sección **Frecuencia de sincronización**.<br/> La primera sincronización comienza después de que transcurre el período de intervalo seleccionado desde el momento en que se guarda la configuración.|
   | **Resolución de conflictos** | Seleccione **Concentrador Win** o **Miembro Win**.<br/><br/>**Concentrador Win** significa que, cuando se produce un conflicto, los datos de la base de datos central sobrescriben los datos en conflicto en la base de datos miembro.<br/><br/>**Miembro Win** significa que, cuando se produce un conflicto, los datos de la base de datos miembro sobrescriben los datos en conflicto en la base de datos central. |

   > [!NOTE]
   > Microsoft recomienda crear una vacía para usarla como **Base de datos de metadatos de sincronización**. Data Sync crea tablas en esta base de datos y ejecuta una carga de trabajo frecuente. Esta base de datos se comparte como la **base de datos de metadatos de sincronización** para todos los grupos de sincronización en un región y una suscripción seleccionadas. No puede cambiar la base de datos ni su nombre sin quitar todos los grupos de sincronización y agentes de sincronización de la región.

   Seleccione **Aceptar** y espere a que se cree e implemente el grupo de sincronización.

## <a name="add-sync-members"></a>Incorporación de miembros de sincronización

Una vez creado e implementado el nuevo grupo de sincronización, el paso 2, **Incorporación de miembros de sincronización**, se resalta en la página **Nuevo grupo de sincronización**.

En la sección **Base de datos central**, escriba las credenciales existentes del servidor en el que se encuentra la base de datos central. No escriba *nuevas* credenciales en esta sección.

![Configuración del paso 2](./media/sql-data-sync-sql-server-configure/steptwo.png)

### <a name="to-add-a-database-in-azure-sql-database"></a>Adición de una base de datos en Azure SQL Database

En la sección **Base de datos de miembros**, tiene la opción de agregar una base de datos de Azure SQL Database al grupo de sincronización si selecciona **Add an Azure SQL Database** (Agregar una base de datos de Azure SQL Database). Se abre la página **Configurar Azure SQL Database**.

  ![Paso 2: Configuración de la base de datos](./media/sql-data-sync-sql-server-configure/steptwo-configure.png)

  En la página **Configurar Azure SQL Database**, cambie las siguientes opciones de configuración:

  | Configuración                       | Descripción |
  | ----------------------------- | ------------------------------------------------- |
  | **Nombre del miembro de sincronización** | Proporcione un nombre para el nuevo miembro de sincronización. Este nombre es distinto del de la base de datos. |
  | **Suscripción** | Seleccione la suscripción de Azure asociada para fines de facturación. |
  | **Azure SQL Server** | Seleccione el servidor. |
  | **Azure SQL Database** | Seleccione la base de datos existente en SQL Database. |
  | **Direcciones de sincronización** | Seleccione **Sincronización bidireccional**, **Al concentrador** o **Desde el concentrador**. |
  | **Nombre de usuario** y **Contraseña** | Escriba las credenciales existentes para el servidor en el que se encuentra la base de datos miembro. No escriba *nuevas* credenciales en esta sección. |

  Seleccione **Aceptar** y espere a que el nuevo miembro de sincronización se cree e implemente.

<a name="add-on-prem"></a>

### <a name="to-add-a-sql-server-database"></a>Para agregar una base de datos de SQL Server

En la sección **Base de datos de miembros**, tiene la opción de agregar una base de datos de SQL Server al grupo de sincronización; para ello, seleccione **Agregar una base de datos local**. Se abrirá la página **Configurar localmente**, donde podrá hacer lo siguiente:

1. Seleccione **Elegir Sync Agent Gateway**. Se abre la página **Seleccionar agente de sincronización**.

   ![Creación de un agente de sincronización](./media/sql-data-sync-sql-server-configure/steptwo-agent.png)

1. En la página **Seleccionar agente de sincronización**, elija si quiere usar un agente existente o crear uno.

   Si ha elegido **Agentes existentes**, seleccione el agente existente de la lista.

   Si ha elegido **Crear un agente nuevo**, haga lo siguiente:

   1. Descargue el agente de sincronización de datos del vínculo facilitado e instálelo en el equipo en que se encuentra SQL Server. También puede descargar el agente directamente desde [SQL Azure Data Sync Agent](https://www.microsoft.com/download/details.aspx?id=27693).

      > [!IMPORTANT]
      > Tendrá que abrir el puerto TCP 1433 saliente en el firewall para permitir que el agente cliente se comunique con el servidor.

   1. Escriba un nombre para el agente.

   1. Seleccione **Crear y generar clave** y copie la clave del agente en el portapapeles.

   1. Seleccione **Aceptar** para cerrar la página **Seleccionar agente de sincronización**.

1. En el equipo de SQL Server, localice y ejecute la aplicación del Agente de sincronización cliente.

   ![La aplicación del agente cliente de Data Sync](./media/sql-data-sync-sql-server-configure/datasync-preview-clientagent.png)

    1. En la aplicación del agente de sincronización, seleccione **Enviar clave del agente**. Se abre el cuadro de diálogo **Configuración de base de datos de metadatos de sincronización**.

    1. En el cuadro de diálogo **Configuración de base de datos de metadatos de sincronización**, pegue la clave del agente copiada de Azure Portal, Proporcione también las credenciales existentes para el servidor en el que se encuentra la base de datos de metadatos. (Si ha creado una base de datos de metadatos, esta se encuentra en el mismo servidor que la base de datos central). Seleccione **Aceptar** y espere a que la configuración finalice.

        ![Definición de las credenciales del servidor y de la clave del agente](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-enterkey.png)

        > [!NOTE]
        > Si recibe un error de firewall, cree una regla de firewall en Azure para permitir el tráfico entrante desde el equipo de SQL Server. Puede crear manualmente la regla en el portal o en SQL Server Management Studio (SSMS). En SSMS, conéctese a la base de datos central en Azure; para ello, escriba el nombre como <nombre_de_la_base_de_datos_central>.database.windows.net.

    1. Seleccione **Registrar** para registrar una instancia de SQL Server Database en el agente. Se abre el cuadro de diálogo **Configuración de SQL Server**.

        ![Adición y configuración de una instancia de SQL Server Database](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-adddb.png)

    1. En el cuadro de diálogo **Configuración de SQL Server**, elija conectarse mediante autenticación de SQL Server o autenticación de Windows. Si elige la autenticación de SQL Server, escriba las credenciales existentes. Proporcione el nombre de SQL Server y el nombre de la base de datos que desea sincronizar y seleccione **Prueba de conexión** para probar la configuración. A continuación, seleccione **Guardar** y la base de datos registrada aparecerá en la lista.

        ![La base de datos de SQL Server ya está registrada](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-dbadded.png)

    1. Cierre la aplicación del Agente de sincronización cliente.

1. En el portal, en la página **Configurar localmente**, seleccione **Seleccionar la base de datos**.

1. En la página **Seleccionar base de datos**, en el campo **Nombre del miembro de sincronización**, proporcione un nombre para el nuevo miembro de sincronización. Este nombre es distinto del nombre de la base de datos. Seleccione la base de datos de la lista. En el campo **Direcciones de sincronización**, seleccione **Sincronización bidireccional**, **Al concentrador** o **Desde el concentrador**.

    ![Seleccione la base de datos local](./media/sql-data-sync-sql-server-configure/datasync-preview-selectdb.png)

1. Seleccione **Aceptar** para cerrar la página **Seleccionar base de datos**. Después, seleccione **Aceptar** para cerrar la página **Configurar localmente** y espere a que el nuevo miembro de sincronización se cree e implemente. Por último, seleccione **Aceptar** para cerrar la página **Seleccionar miembros de sincronización**.

> [!NOTE]
> Para conectarse a SQL Data Sync y al agente local, agregue el nombre de usuario al rol *DataSync_Executor*. Data Sync crea este rol en la instancia de SQL Server.

## <a name="configure-sync-group"></a>Configuración del grupo de sincronización

Una vez creados e implementados los nuevos miembros del grupo de sincronización, el paso 3, **Configuración del grupo de sincronización**, se resalta en la página **Nuevo grupo de sincronización**.

![Configuración del paso 3](./media/sql-data-sync-sql-server-configure/stepthree.png)

1. En la página **Tablas**, seleccione una base de datos de la lista de miembros del grupo de sincronización y **Actualizar esquema**.

1. En la lista, seleccione las tablas que desea sincronizar. De forma predeterminada se seleccionan todas las columnas; deshabilite la casilla de las columnas que no desee sincronizar. Asegúrese de dejar seleccionada la columna de clave principal.

1. Seleccione **Guardar**.

1. De forma predeterminada, las bases de datos no se sincronizan hasta la ejecución programada o manual. Para la sincronización manual, vaya a la base de datos de SQL Database en Azure Portal, seleccione **Sincronizar con otras bases de datos** y elija el grupo de sincronización. La página **Data Sync** se abre. Seleccione **Sincronizar**.

    ![Sincronización manual](./media/sql-data-sync-sql-server-configure/datasync-sync.png)

## <a name="faq"></a>Preguntas más frecuentes

**¿Con qué frecuencia puede sincronizar mis datos Data Sync?**

El tiempo mínimo entre una sincronización y otra es de cinco minutos.

**¿Crea SQL Data Sync tablas completas?**

Si faltan tablas del esquema de sincronización en la base de datos de destino, SQL Data Sync las crea con las columnas que haya seleccionado. Sin embargo, esto no genera un esquema de total fidelidad por las razones siguientes:

- Solo las columnas seleccionadas se crean en la tabla de destino. Las que no haya seleccionado se omiten.
- En la tabla de destino solo se crean los índices de las columnas seleccionadas. Los de las columnas sin seleccionar se ignoran.
- No se crean índices de las columnas de tipo XML.
- No se crean restricciones CHECK.
- No crean desencadenadores en las tablas de origen.
- No se crean vistas ni procedimientos almacenados.

Debido a estas limitaciones, se recomienda lo siguiente:

- Para entornos de producción, cree el esquema de total fidelidad personalmente.
- Al experimentar con el servicio, use la característica de aprovisionamiento automático.

**¿Por qué veo tablas que yo no he creado?**

Data Sync crea tablas adicionales en la base de datos para hacer un seguimiento de los cambios. No las elimine, ya que Data Sync dejaría de funcionar.

**¿Son los datos homogéneos después de una sincronización?**

No necesariamente. En un grupo de sincronización con un concentrador y tres radios (A, B y C), se sincroniza el concentrador con A, con B y con C. Si se realiza un cambio en la base de datos A *después* de que se haya sincronizado el concentrador con A, ese cambio no se aplica en las bases de datos B o C hasta la siguiente tarea de sincronización.

**¿Cómo se realizan cambios de esquema en un grupo de sincronización?**

Realice todos los cambios de esquema y propáguelos manualmente.

1. Replique los cambios de esquema manualmente en el centro y en todos los miembros de la sincronización.
1. Actualice el esquema de sincronización.

Para agregar nuevas tablas y columnas:

Las nuevas tablas y columnas no afectan a la sincronización actual y Data Sync las omite hasta que se agregan al esquema de sincronización. Al agregar nuevos objetos de base de datos, siga este orden:

1. Agregue las nuevas tablas o columnas al concentrador y a todos los miembros de la sincronización.
1. Agregue las nuevas tablas o columnas al esquema de sincronización.
1. Empiece a insertar valores en las nuevas tablas y columnas.

Para cambiar el tipo de datos de una columna:

Al cambiar el tipo de datos de una columna existente, Data Sync funcionará siempre que los nuevos valores se ajusten al tipo de datos original definido en el esquema de sincronización. Por ejemplo, si cambia el tipo de la base de datos de origen de **int** a **bigint**, Data Sync continúa funcionando hasta que inserte un valor demasiado grande para el tipo de datos **int**. Para completar el cambio, replique el cambio de esquema manualmente en el concentrador y en todos los miembros de la sincronización y actualice el esquema de sincronización.

**¿Cómo se exporta e importa una base de datos con Data Sync?**

Después de exportar una base de datos como archivo *.bacpac* y de importar dicho archivo para crear una base de datos, realice lo siguiente para usar Data Sync en la nueva base de datos:

1. Limpie los objetos de Data Sync y las tablas adicionales en la nueva base de datos mediante [este script](https://github.com/vitomaz-msft/DataSyncMetadataCleanup/blob/master/Data%20Sync%20complete%20cleanup.sql). Este script elimina todos los objetos de Data Sync necesarios de la base de datos.
1. Vuelva a crear el grupo de sincronización con la nueva base de datos. Si ya no necesita el grupo de sincronización antiguo, elimínelo.

**¿Dónde puedo encontrar información sobre el agente cliente?**

Para ver las preguntas más frecuentes sobre el agente de cliente, consulte [Preguntas más frecuentes sobre el agente](sql-data-sync-agent-overview.md#agent-faq).

## <a name="next-steps"></a>Pasos siguientes

¡Enhorabuena! Ha creado un grupo de sincronización que incluye una instancia de SQL Database y una base de datos de SQL Server.

Para más información sobre SQL Data Sync, consulte:

- [Data Sync Agent para Azure SQL Data Sync](sql-data-sync-agent-overview.md)
- [Procedimientos recomendados](sql-data-sync-best-practices.md) y [Solución de problemas de Azure SQL Data Sync](sql-data-sync-troubleshoot.md)
- [Monitor SQL Data Sync with Azure Monitor logs](sql-data-sync-monitor-sync.md) (Supervisión de SQL Data Sync con registros de Azure Monitor)
- [Actualización del esquema de sincronización con Transact-SQL](sql-data-sync-update-sync-schema.md) o [PowerShell](scripts/update-sync-schema-in-sync-group.md)

Para más información sobre SQL Database, consulte:

- [Información general de SQL Database](sql-database-paas-overview.md)
- [Administración del ciclo de vida de las aplicaciones](https://msdn.microsoft.com/library/jj907294.aspx)
