---
title: Copia de seguridad de cargas de trabajo de SQL Server en Azure Stack
description: En este artículo, aprenda a configurar Microsoft Azure Backup Server (MABS) para proteger las bases de datos SQL Server en Azure Stack.
ms.topic: conceptual
ms.date: 06/08/2018
ms.openlocfilehash: 706050fa37e4234a0ffc902f6b696ebd84e6701e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87032653"
---
# <a name="back-up-sql-server-on-azure-stack"></a>Copia de seguridad de SQL Server en Azure Stack

Use este artículo para configurar Microsoft Azure Backup Server (MABS) para proteger las bases de datos SQL Server en Azure Stack.

La administración de la copia de seguridad de bases de datos de SQL Server en Azure implica tres pasos:

1. Crear una directiva de copia de seguridad para proteger las bases de datos de SQL Server
2. Crear copias de seguridad a petición
3. Recuperar la base de datos de discos y de Azure

## <a name="prerequisites-and-limitations"></a>Requisitos previos y limitaciones

* Si tiene una base de datos con archivos en un recurso compartido de archivos remoto, la protección no se realizará correctamente y generará el identificador de error 104. MABS no admite la protección de datos de SQL Server ubicados en un recurso compartido de archivos remoto.
* MABS no puede proteger las bases de datos almacenadas en recursos compartidos de SMB remotos.
* Asegúrese de que las [réplicas del grupo de disponibilidad están configuradas como de solo lectura](/sql/database-engine/availability-groups/windows/configure-read-only-access-on-an-availability-replica-sql-server?view=sql-server-ver15).
* Deberá agregar explícitamente la cuenta del sistema **NTAuthority\System** al grupo Sysadmin en SQL Server.
* Al realizar una recuperación desde una ubicación alternativa de una base de datos parcialmente independiente, debe asegurarse de que la instancia SQL de destino tenga habilitada la característica [Bases de datos independientes](/sql/relational-databases/databases/migrate-to-a-partially-contained-database?view=sql-server-ver15#enable).
* Al realizar una recuperación desde una ubicación alternativa de una base de datos de flujo de archivos, debe asegurarse de que la instancia de SQL de destino tenga habilitada la característica de [base de datos de flujo de archivos](/sql/relational-databases/blob/enable-and-configure-filestream?view=sql-server-ver15).
* Protección para SQL Server AlwaysOn:
  * MABS detecta los grupos de disponibilidad cuando se ejecuta una consulta en la creación del grupo de protección.
  * MABS detecta una conmutación por error y continúa la protección de la base de datos.
  * MABS admite configuraciones de clúster de varios sitios en una instancia de SQL Server.
* Al proteger las bases de datos que utilizan la característica AlwaysOn, MABS tiene las siguientes limitaciones:
  * MABS respetará la directiva de copia de seguridad de grupos de disponibilidad establecida en SQL Server según las preferencias de copia de seguridad, como se indica a continuación:
    * Preferir secundaria: las copias de seguridad se deben realizar en una réplica secundaria, excepto cuando la réplica principal sea la única réplica en línea. Si hay varias réplicas secundarias disponibles, se seleccionará el nodo con la mayor prioridad de copia de seguridad para realizar la copia de seguridad. En el caso de que solo esté disponible la réplica principal, la copia de seguridad se deberá realizar en ella.
    * Solo secundaria: la copia de seguridad no se debe realizar en la réplica principal. Si la réplica principal es la única en línea, no se deberá realizar la copia de seguridad.
    * Principal: las copias de seguridad se deben realizar siempre en la réplica principal.
    * Cualquier réplica: las copias de seguridad se pueden realizar en cualquier réplica disponible en el grupo de disponibilidad. El nodo desde el que se va a realizar la copia de seguridad se basará en las prioridades de copia de seguridad de cada uno de los nodos.
  * Tenga en cuenta lo siguiente:
    * Las copias de seguridad se pueden realizar desde cualquier réplica legible, es decir, principal, secundaria sincrónica o secundaria asincrónica.
    * Si se excluye alguna réplica de la copia de seguridad, por ejemplo, si se ha habilitado **Excluir de la réplica** o está marcada como no legible, entonces esa réplica no se seleccionará para la copia de seguridad en ninguna de las opciones.
    * Si hay varias réplicas disponibles y legibles, se seleccionará el nodo con la mayor prioridad de copia de seguridad para realizar la copia de seguridad.
    * Si se produce un error de la copia de seguridad en el nodo seleccionado, la operación de copia de seguridad no se realizará.
    * No se admite la recuperación en la ubicación original.
* Problemas de copia de seguridad de SQL Server 2014 o superior:
  * SQL Server 2014 agregó una nueva característica para crear una [base de datos de SQL Server local en el almacenamiento de blobs de Microsoft Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure?view=sql-server-ver15). No se puede usar MABS para proteger esta configuración.
  * Hay algunos problemas conocidos con la preferencia de copia de seguridad "Preferir secundaria" de la opción SQL AlwaysOn. MABS siempre hace una copia de seguridad de la secundaria. Si no se puede encontrar una secundaria, la copia de seguridad no se realiza.

## <a name="before-you-start"></a>Antes de comenzar

[Instale y prepare Azure Backup Server](backup-mabs-install-azure-stack.md).

## <a name="create-a-backup-policy-to-protect-sql-server-databases-to-azure"></a>Crear una directiva de copia de seguridad para proteger las bases de datos SQL Server en Azure

1. En la IU de Azure Backup Server, haga clic en el área de trabajo **Protección**.

2. En la cinta de herramientas, haga clic en **Nuevo** para crear un nuevo grupo de protección.

    ![Creación de un grupo de protección](./media/backup-azure-backup-sql/protection-group.png)

    Azure Backup Server inicia el asistente para crear nuevo grupo de protección que lo guía por el proceso de creación de un **grupo de protección**. Haga clic en **Next**.

3. En la pantalla **Seleccionar tipo de grupo de protección**, seleccione **Servidores**.

    ![Selección de tipo de grupo de protección - 'Servidores'](./media/backup-azure-backup-sql/pg-servers.png)

4. En la pantalla **Seleccionar a miembros del grupo**, la lista de miembros disponibles muestra los distintos orígenes de datos. Haga clic en **+** para expandir una carpeta y mostrar las subcarpetas. Active la casilla de verificación para seleccionar un elemento.

    ![Selección de base de datos SQL](./media/backup-azure-backup-sql/pg-databases.png)

    Todos los elementos seleccionados aparecen en la lista de miembros seleccionados. Después de seleccionar los servidores o las bases de datos que quiere proteger, haga clic en **Siguiente**.

5. En la pantalla **Seleccionar método de protección de datos**, indique un nombre para el grupo de protección y active la casilla de verificación **Deseo protección en línea**.

    ![Método de protección de datos: disco a corto plazo y en línea de Azure](./media/backup-azure-backup-sql/pg-name.png)

6. En la pantalla **Especificar objetivos a corto plazo**, incluya las entradas necesarias para crear puntos de copia de seguridad en el disco y haga clic en **Siguiente**.

    En el ejemplo, **Duración de retención** es **5 días**, **Frecuencia de sincronización** es una vez cada **15 minutos**, que es la frecuencia de copia de seguridad. **Copia de seguridad completa rápida** está establecido en **8:00 p.m**.

    ![Objetivos a corto plazo](./media/backup-azure-backup-sql/pg-shortterm.png)

   > [!NOTE]
   > En el ejemplo que se muestra, todos los días a las 8:00 p.m. se crea un punto de copia de seguridad mediante la transferencia de los datos modificados a partir del punto de recuperación del día anterior a las 8:00 p.m. Este proceso se denomina **Copia de seguridad completa rápida**. Los registros de transacciones se sincronizan cada 15 minutos. Si tiene que recuperar la base de datos a las 9:00 p.m., el punto se crea a partir de los registros del último punto de copia de seguridad completa rápida (en este caso, a las 8:00 p.m.).
   >
   >

7. En la pantalla **Revisar la asignación de disco**, compruebe el espacio de almacenamiento general disponible y el posible espacio en disco. Haga clic en **Next**.

8. En **Elegir método de creación de réplica**, elija cómo crear el primer punto de recuperación. Puede transferir manualmente la copia de seguridad inicial (fuera de la red) para evitar la congestión del ancho de banda o a través de la red. Si elige esperar para transferir la primera copia de seguridad, puede especificar la hora de la transferencia inicial. Haga clic en **Next**.

    ![Método de replicación inicial](./media/backup-azure-backup-sql/pg-manual.png)

    La copia de seguridad inicial requiere la transferencia de todo el origen de datos (base de datos SQL Server) desde el servidor de producción (equipo con SQL Server) a Azure Backup Server. Estos datos pueden ser grandes y la transferencia de los datos a través de la red podría superar el ancho de banda. Por este motivo, puede optar por transferir la copia de seguridad inicial: **Manualmente** (mediante un soporte físico extraíble) para evitar la congestión del ancho de banda, o **Automáticamente a través de la red** (en un momento determinado).

    Una vez completada la copia de seguridad inicial, el resto de las copias de seguridad son copias de seguridad incrementales sobre la copia de seguridad inicial. Las copias de seguridad incrementales tienden a ser pequeñas y se transfieren fácilmente a través de la red.

9. Elija cuándo quiere que se ejecute la comprobación de coherencia y haga clic en **Siguiente**.

    ![Comprobación de coherencia](./media/backup-azure-backup-sql/pg-consistent.png)

    Azure Backup Server puede realizar una comprobación de coherencia de la integridad del punto de copia de seguridad. Azure Backup Server calcula la suma de comprobación del archivo de copia de seguridad en el servidor de producción (equipo con SQL Server en este escenario) y los datos de copia de seguridad para ese archivo. Si surge un conflicto, se da por sentado que el archivo con copia de seguridad en Azure Backup Server está dañado. Azure Backup Server rectifica los datos de copia de seguridad mediante el envío de los bloques correspondientes a la suma de comprobación no coincidente. Como las comprobaciones de coherencia requieren un alto rendimiento, puede programarla o ejecutarla de manera automática.

10. Para especificar la protección en línea de los orígenes de datos, seleccione las bases de datos que se van proteger en Azure y haga clic en **Siguiente**.

    ![Selección de orígenes de datos](./media/backup-azure-backup-sql/pg-sqldatabases.png)

11. Elija las programaciones de copia de seguridad y las directivas de retención que se ajusten a las directivas de la organización.

    ![Programación y retención](./media/backup-azure-backup-sql/pg-schedule.png)

    En este ejemplo, las copias de seguridad se realizan una vez a las 12:00 p.m. y a las 8 p.m. (parte inferior de la pantalla)

    > [!NOTE]
    > Es recomendable tener algunos puntos de recuperación a corto plazo en disco para una recuperación rápida. Estos puntos de recuperación se usan para la recuperación operacional. Azure actúa como una ubicación válida fuera de sitio con unos contratos de nivel de servicio mayores y una disponibilidad garantizada.
    >
    >

    **Procedimiento recomendado**: si programa que las copias de seguridad en Azure se inicien después de que se completen las copias de seguridad del disco local, las copias de seguridad más recientes del disco se copiarán siempre en Azure.

12. Seleccione la programación de la directiva de retención. Se proporcionan detalles sobre el funcionamiento de la directiva de retención en [Usar Azure Backup para cambiar su infraestructura de cintas](backup-azure-backup-cloud-as-tape.md).

    ![Directiva de retención](./media/backup-azure-backup-sql/pg-retentionschedule.png)

    En este ejemplo:

    * Las copias de seguridad se realizan una vez a las 12:00 p.m. y a las 8 p.m. (parte inferior de la pantalla) y se conservan durante 180 días.
    * La copia de seguridad es el sábado a las 12:00 P.M. se conserva durante 104 semanas
    * La copia de seguridad es el último sábado de 12:00 P.M. se conserva durante 60 meses
    * La copia de seguridad es el último sábado de marzo a las 12:00 P.M. se conserva durante 10 años
13. Haga clic en **Siguiente** y seleccione la opción adecuada para transferir la copia de seguridad inicial a Azure. Puede elegir **Automáticamente por encima de la red**.

14. Una vez revisados los detalles de la directiva en la pantalla **Resumen**, haga clic en **Crear grupo** para completar el flujo de trabajo. Puede hacer clic en **Cerrar** y supervisar el progreso del trabajo en el área de trabajo de supervisión.

    ![Creación de un grupo de protección en curso](./media/backup-azure-backup-sql/pg-summary.png)

## <a name="on-demand-backup-of-a-sql-server-database"></a>Copia de seguridad a petición de una base de datos SQL Server

Aunque los pasos anteriores crean una directiva de copia de seguridad, solo se crea un “punto de recuperación” cuando se produce la primera copia de seguridad. En lugar de esperar a que se inicie el programador, los pasos siguientes activarán la creación de un punto de recuperación manualmente.

1. Espere hasta que el estado del grupo de protección muestre **Correcto** para la base de datos antes de crear el punto de recuperación.

    ![Miembros del grupo de protección](./media/backup-azure-backup-sql/sqlbackup-recoverypoint.png)
2. Haga clic con el botón derecho en la base de datos y seleccione **Crear punto de recuperación**.

    ![Creación de punto de recuperación en línea](./media/backup-azure-backup-sql/sqlbackup-createrp.png)
3. Elija **Protección en línea** en el menú desplegable y haga clic en **Aceptar** para empezar a crear un punto de recuperación en Azure.

    ![Crear punto de recuperación](./media/backup-azure-backup-sql/sqlbackup-azure.png)
4. Vea el progreso del trabajo en el área de trabajo de **supervisión**.

    ![Consola de supervisión](./media/backup-azure-backup-sql/sqlbackup-monitoring.png)

## <a name="recover-a-sql-server-database-from-azure"></a>Recuperación de una base de datos SQL Server de Azure

Los pasos siguientes son necesarios para recuperar una entidad protegida (base de datos SQL) de Azure.

1. Abra la consola de administración de Azure Backup Server. Vaya al área de trabajo **Recuperación** en la que puede ver los servidores protegidos. Examine la base de datos requerida (en este caso ReportServer$MSDPM2012). Seleccione una hora en la opción **Recovery from** (Recuperación desde) que se especifique como un punto **Online** (En línea).

    ![Selección de punto de recuperación](./media/backup-azure-backup-sql/sqlbackup-restorepoint.png)
2. Haga clic con el botón secundario en el nombre de la base de datos y haga clic en **Recuperar**.

    ![Recuperación desde Azure](./media/backup-azure-backup-sql/sqlbackup-recover.png)
3. MABS muestra los detalles del punto de recuperación. Haga clic en **Next**. Para sobrescribir la base de datos, seleccione el tipo de recuperación **Recuperar en instancia original de servidor SQL Server**. Haga clic en **Next**.

    ![Recuperación en ubicación original](./media/backup-azure-backup-sql/sqlbackup-recoveroriginal.png)

    En este ejemplo, MABS recupera la base de datos en otra instancia de SQL Server o en una carpeta de red independiente.

4. En la pantalla **Especificar opciones de recuperación** , puede seleccionar las opciones de recuperación como Límite de uso del ancho de banda de red para limitar el ancho de banda usado por la recuperación. Haga clic en **Next**.

5. En la pantalla **Resumen** , podrá ver todas las configuraciones de recuperación proporcionadas hasta ahora. Haga clic en **Recuperar**.

    El estado de recuperación muestra que la base de datos se está recuperando. Puede hacer clic en **Cerrar** para cerrar el asistente y ver el progreso en el área de trabajo **Supervisión**.

    ![Inicio de proceso de recuperación](./media/backup-azure-backup-sql/sqlbackup-recoverying.png)

    Una vez completada la recuperación, la base de datos restaurada será coherente con la aplicación.

## <a name="next-steps"></a>Pasos siguientes

Consulte el artículo sobre [copia de seguridad de archivos y aplicaciones](backup-mabs-files-applications-azure-stack.md).
Consulte el artículo sobre [copia de seguridad de SharePoint en Azure Stack](backup-mabs-sharepoint-azure-stack.md).
