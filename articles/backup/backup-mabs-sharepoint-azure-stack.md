---
title: Copia de seguridad de una granja de SharePoint en Azure Stack
description: Use Azure Backup Server para crear una copia de seguridad de los datos de SharePoint y restaurarlos en Azure Stack. En este artículo se proporciona la información sobre cómo configurar la granja de SharePoint para almacenar los datos deseados en Azure. Puede restaurar los datos protegidos de SharePoint desde disco o desde Azure.
ms.topic: conceptual
ms.date: 06/07/2020
ms.openlocfilehash: 1e237e63b92468fafff4f8f8f525d1388840d162
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378328"
---
# <a name="back-up-a-sharepoint-farm-on-azure-stack"></a>Copia de seguridad de una granja de SharePoint en Azure Stack

La copia de seguridad de una granja de SharePoint de Azure Stack en Microsoft Azure se crea mediante Microsoft Azure Backup Server (MABS) casi de la misma manera que realiza la copia de seguridad de otros orígenes de datos. Azure Backup ofrece flexibilidad en la programación de copias de seguridad para crear puntos de copia de seguridad diarios, semanales, mensuales o anuales, y le ofrece diferentes opciones de directiva de retención para varios puntos de copia de seguridad. También ofrece la posibilidad de almacenar copias en discos locales para conseguir objetivos de tiempo de recuperación (RTO) más rápidos y de almacenar copias en Azure, para una retención económica más a largo plazo.

## <a name="sharepoint-supported-versions-and-related-protection-scenarios"></a>Las versiones compatibles de SharePoint y relacionadas con escenarios de protección

Azure Backup para MABS admite los siguientes escenarios:

| Carga de trabajo | Versión | Implementación de SharePoint | Protección y recuperación |
| --- | --- | --- | --- |
| SharePoint |SharePoint 2016, SharePoint 2013, SharePoint 2010 |SharePoint implementado como máquina virtual de Azure Stack <br> -------------- <br> SQL AlwaysOn | Opciones de protección de recuperación de la granja de SharePoint: granja de servidores de recuperación, base de datos y archivo, o elemento de la lista de puntos de recuperación de disco.  Recuperación de base de datos y granja de servidores a partir de puntos de recuperación de Azure. |

## <a name="before-you-start"></a>Antes de comenzar

Antes de realizar una copia de seguridad de una granja de SharePoint en Azure, hay algunas cuantas cosas que debe confirmar.

### <a name="whats-not-supported"></a>Lo que no se admite

* Que MABS proteja una granja de SharePoint y no proteja índices de búsqueda o bases de datos de servicios de aplicaciones. Deberá configurar la protección de estas bases de datos por separado.

* Que MABS no proporcione copia de seguridad de bases de datos SQL Server de SharePoint hospedadas en recursos compartidos de servidor de archivos de escalabilidad horizontal (SOFS).

### <a name="prerequisites"></a>Requisitos previos

Antes de continuar, asegúrese de que se cumplen todos los [requisitos previos para usar Microsoft Azure Backup](backup-azure-dpm-introduction.md#prerequisites-and-limitations) a fin de proteger las cargas de trabajo. Algunas de las tareas que son requisito previo incluyen: crear un almacén de copia de seguridad, descargar las credenciales de almacén, instalar el agente de copia de seguridad de Azure y registrar Azure Backup Server en el almacén.

Requisitos previos y las limitaciones adicionales:

* De forma predeterminada, al proteger SharePoint, se protegerán todas las bases de datos de contenido (y las bases de datos SharePoint_Config y SharePoint_AdminContent*). Si desea agregar personalizaciones como índices de búsqueda, plantillas o bases de datos de servicios de aplicación, o bien el servicio de perfiles de usuario, deberá configurarlas para la protección por separado. Asegúrese de habilitar la protección para todas las carpetas que incluyan estos tipos de características o archivos de personalización.

* No puede proteger las bases de datos de SharePoint como un origen de datos de SQL Server. Puede recuperar bases de datos individuales desde una copia de seguridad de la granja.

* Recuerde que MABS se ejecuta como **sistema local**, y para realizar una copia de seguridad de las bases de datos de SQL Server, necesita privilegios de administrador del sistema en esa cuenta para SQL Server. En el servidor SQL Server del que desea realizar una copia de seguridad, establezca NT AUTHORITY\SYSTEM en **sysadmin**.

* Para cada 10 millones de elementos del conjunto de servidores, debe haber al menos 2 GB de espacio en el volumen donde se encuentra la carpeta MABS. Este espacio se requiere para la generación del catálogo. Para poder utilizar MABS para realizar una recuperación específica de elementos (colecciones de sitios, sitios, listas, bibliotecas de documentos, carpetas, documentos individuales y elementos de lista), la generación de catálogos crea una lista de las direcciones URL que están dentro de cada base de datos de contenido. Puede ver la lista de direcciones URL en el panel de elementos recuperables en el área de tareas de recuperación de la Consola de administrador MABS.

* En la granja de SharePoint, si tiene bases de datos de SQL Server que están configuradas con alias de SQL Server, instale los componentes del cliente de SQL Server en el servidor web front-end web que MABS protegerá.

* La protección de elementos de la tienda de aplicaciones no es compatible con SharePoint 2013.

* MABS no admite la protección de FILESTREAM remoto. FILESTREAM debe formar parte de la base de datos.

## <a name="configure-backup"></a>Configuración de la copia de seguridad

Para realizar una copia de seguridad de una granja de SharePoint, configure la protección de SharePoint mediante ConfigureSharePoint.exe y, a continuación, cree un grupo de protección en MABS.

1. **Ejecute ConfigureSharePoint.exe**: esta herramienta configura el servicio \(WSS\) de VSS Writer de SharePoint y proporciona el agente de protección con las credenciales de la granja de SharePoint. Después de haber implementado el agente de protección, el archivo ConfigureSharePoint.exe puede encontrarse en la carpeta `<MABS Installation Path\>\bin` del servidor web del front-end.  Si tiene varios servidores WFE, solo tiene que instalarlo en uno de ellos. Ejecute del modo siguiente:

    * En el servidor WFE, en un símbolo del sistema, vaya a `\<MABS installation location\>\\bin\\` y ejecute `ConfigureSharePoint \[\-EnableSharePointProtection\] \[\-EnableSPSearchProtection\] \[\-ResolveAllSQLAliases\] \[\-SetTempPath <path>\]`, donde:

        * **EnableSharePointProtection** habilita la protección de la granja de SharePoint, habilita VSS Writer y registra la identidad del WssCmdletsWrapper de la aplicación DCOM para que se ejecute como un usuario cuyas credenciales se especifican con esta opción. Esta cuenta debe tener el rol de administrador de la granja y también administrador local en el servidor web del front\-end.

        * **EnableSPSearchProtection** habilita la protección de SP Search de WSS 3.0 mediante la clave del Registro SharePointSearchEnumerationEnabled en HKLM\\Software\\Microsoft\\Microsoft Data Protection Manager\\Agent\\2.0\\ en el servidor web del front\-end y registra la identidad del WssCmdletsWrapper de la aplicación DCOM WssCmdletsWrapper para que se ejecute como un usuario cuyas credenciales se especifican con esta opción. Esta cuenta debe tener el rol de administrador de la granja y también administrador local en el servidor web del front\-end.

        * **ResolveAllSQLAliases** muestra todos los alias notificados por VSS Writer de SharePoint y los resuelve en el servidor SQL Server correspondiente. También se muestran sus nombres de instancia resueltos. Si los servidores están reflejados, también muestra el servidor reflejado. Notifica todos los alias que no se están resolviendo en un servidor SQL Server.

        * **SetTempPath** establece las variables de entorno TEMP y TMP en la ruta de acceso especificada. Se produce un error en la recuperación de nivel de elemento si se está realizando una recuperación extensa de un sitio, una colección de sitios, una lista o un elemento y no hay espacio suficiente en la carpeta Temporary del administrador de la granja. Esta opción permite cambiar la ruta de acceso de la carpeta de los archivos temporales a un volumen que tenga espacio suficiente para almacenar el sitio o la colección de sitios objeto de recuperación.

    * Escriba las credenciales de administrador de la granja de servidores. Esta cuenta debe ser miembro del grupo de administradores local en el servidor WFE. Si el administrador de la granja no es un administrador local, conceda los permisos siguientes en el servidor WFE:

        * Conceda al grupo **WSS_Admin_WPG** control total sobre la carpeta de MABS (`%Program Files%\Data Protection Manager\DPM\`).

        * Conceda al grupo **WSS_Admin_WPG** acceso de lectura a la clave de Registro de MABS (`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager`).

        Después de ejecutar ConfigureSharePoint.exe, tendrá que volver a ejecutarlo cada vez que haya un cambio en las credenciales de administrador de la granja de SharePoint.

1. Para crear un grupo de protección, seleccione **Protección** > **Acciones** > **Crear grupo de protección** para abrir el **Asistente para crear nuevo grupo de protección** en la Consola de MABS.

1. En **Seleccionar tipo de grupo de protección**, seleccione **Servidores**.

1. En **Seleccionar miembros del grupo**, expanda el servidor que contiene el rol WFE. Si hay más de un servidor WFE, seleccione aquel en que instaló ConfigureSharePoint.exe.

    Al expandir el servidor de SharePoint, MABS consulta a VSS para ver qué datos puede proteger MABS.  Si la base de datos de SharePoint es remota, MABS se conecta a ella. Si los orígenes de datos de SharePoint no aparecen, compruebe que VSS Writer se está ejecutando en el servidor de SharePoint y en cualquier servidor SQL Server remoto, y asegúrese de que el agente de MABS esté instalado en el servidor de SharePoint y en el servidor SQL Server remoto. Además, asegúrese de que las bases de datos de SharePoint no se están protegiendo en otro lugar como bases de datos de SQL Server.

1. En **Seleccionar método de protección de datos**, especifique cómo desea administrar la copia de seguridad a corto y largo plazo. La copia de seguridad a corto plazo siempre se realiza primero en disco, pero existe la opción de realizar una copia de seguridad desde el disco en la nube de Azure mediante Azure Backup \(a corto o largo plazo\).

1. En **Especificar objetivos a corto plazo**, especifique cómo quiere realizar la copia de seguridad en el almacenamiento a corto plazo en disco.   En **Duración de retención**, especfique cuánto tiempo quiere conservar los datos en el disco. En **Frecuencia de sincronización**, especifique con qué frecuencia quiere ejecutar una copia de seguridad incremental en el disco. Si no desea establecer un intervalo de copia de seguridad, puede marcar Solo antes de un punto de recuperación para que MABS ejecute una copia de seguridad completa rápida solo antes del momento en que esté programado un punto de recuperación.

1. En la página Revisar asignación de disco, revise el espacio en disco del grupo de almacenamiento asignado para el grupo de protección.

    **Total Data size** (Tamaño total de datos) es el tamaño de los datos de los que quiere realizar la copia de seguridad y **Disk space to be provisioned on MABS** (Espacio en disco que se va a aprovisionar en MABS) es el espacio que recomienda MABS para el grupo de protección. MABS elige el volumen ideal para la copia de seguridad, en función de la configuración. Pero puede editar las opciones de volumen de copia de seguridad en los **detalles de asignación del disco**. En el caso de las cargas de trabajo, seleccione el almacenamiento de su preferencia en el menú desplegable. Las modificaciones cambiarán los valores de **Almacenamiento total** y **Free Storage** (Almacenamiento libre) en el panel **Almacenamiento en disco disponible**. El espacio insuficiente es la cantidad de almacenamiento que MABS sugiere agregar al volumen, para continuar las copias de seguridad sin problemas en el futuro.

1. En **Elegir método de creación de réplica**, seleccione cómo quiere controlar la replicación inicial completa de los datos.  Si selecciona replicar a través de la red, se recomienda elegir una hora de menor actividad. Para grandes cantidades de datos o condiciones de red no del todo óptimas, considere la posibilidad de replicar los datos sin conexión con medios extraíbles.

1. En **Elegir las opciones de la comprobación de coherencia**, seleccione cómo quiere automatizar las comprobaciones de coherencia. Puede habilitar una comprobación para que se ejecute solo cuando los datos de réplica no sean coherentes, o bien según una programación. Si no quiere configurar la comprobación de coherencia automática, puede ejecutar una comprobación manual en cualquier momento: para ello, haga clic con el botón derecho en el grupo de protección en el área **Protección** de la consola de MABS y seleccione **Realizar comprobación de coherencia**.

1. Si ha seleccionado realizar una copia de seguridad en la nube mediante Azure Backup, asegúrese de que en la página **Especificar datos de protección en línea** están seleccionadas las cargas de trabajo de las que quiere realizar una copia de seguridad en Azure.

1. En **Especificar la programación de copia de seguridad en línea**, especifique con qué frecuencia se deberían realizar las copias de seguridad incrementales en Azure. Puede programar que las copias de seguridad se ejecuten cada día, semana, mes y año y seleccionar la hora y la fecha en la que se deben ejecutar. Pueden realizarse copias de seguridad hasta dos veces al día. Cada vez que se ejecuta una copia de seguridad, se crea un punto de recuperación de datos en Azure a partir de la copia de los datos de copia de seguridad almacenados en el disco de MABS.

1. En **Especificar la directiva de retención en línea**, puede especificar cómo se retienen en Azure los puntos de recuperación creados a partir de copias de seguridad diarias, semanales, mensuales y anuales.

1. En **Elegir replicación en línea**, especifique cómo se realizará la replicación inicial total de los datos. Puede replicar a través de la red o realizar una copia de seguridad sin conexión (propagación sin conexión). Las copias de seguridad sin conexión usan la característica Azure Import. [Más información](./backup-azure-backup-import-export.md).

1. En la página **Resumen**, revise la configuración. Después de seleccionar **Crear grupo**, se produce la replicación inicial de los datos. Cuando finalice, el estado del grupo de protección se mostrará como **Correcto** en la página**Estado**. Después se realiza una copia de seguridad según la configuración del grupo de protección.

## <a name="monitoring"></a>Supervisión

Después de crear el grupo de protección, se produce la replicación inicial y MABS empieza a realizar la copia de seguridad y la sincronización de los datos de SharePoint. MABS supervisa la sincronización inicial y las copias de seguridad posteriores.  Puede supervisar los datos de SharePoint de dos maneras:

* Mediante la supervisión de MABS predeterminada, puede configurar las notificaciones para la supervisión proactiva mediante la publicación de alertas y la configuración de notificaciones. Puede enviar notificaciones por correo electrónico para las alertas críticas, de advertencia o informativas y para el estado de las recuperaciones con instancias.

* Si usa Operations Manager, puede publicar las alertas de forma centralizada.

### <a name="set-up-monitoring-notifications"></a>Configuración de notificaciones de supervisión

1. En la Consola de administrador MABS, haga clic en **Supervisión** > **Acción** > **Opciones**.

2. Seleccione **Servidor SMTP**, escriba el nombre del servidor, el puerto y la dirección de correo electrónico desde la que se enviarán las notificaciones. La dirección debe ser válida.

3. En **Servidor SMTP autenticado**, escriba un nombre de usuario y una contraseña. El nombre de usuario y la contraseña que escriba deben ser el nombre de cuenta de dominio de la persona cuya dirección "De" se describe en el paso anterior. De lo contrario, se produce un error en la entrega de la notificación.

4. Para probar la configuración del servidor SMTP, seleccione **Enviar correo electrónico de prueba**, escriba la dirección de correo electrónico a la que desea que MABS envíe el mensaje de prueba y, a continuación, seleccione **Aceptar**. Seleccione **Opciones** > **Notificaciones** y seleccione los tipos de alertas sobre qué destinatarios desean recibir notificaciones. En **Destinatarios**, escriba la dirección de correo electrónico para cada destinatario al que desea que MABS envíe copias de las notificaciones.

### <a name="publish-operations-manager-alerts"></a>Importación de alertas de Operations Manager

1. En la Consola de administrador MABS, seleccione **Supervisión** > **Acción** > **Opciones** > **Publicación de alertas** > **Publicar alertas activas**.

2. Después de habilitar **Publicación de alertas**, todas las alertas de MABS existentes que pueden requerir una acción del usuario se publican en el registro de eventos de **Alertas de MABS**. A continuación, el agente de Operations Manager que está instalado en el servidor de MABS publica estas alertas en Operations Manager y continúa actualizando la consola a medida que se generan nuevas alertas.

## <a name="restore-a-sharepoint-item-from-disk-by-using-mabs"></a>Restauración de un elemento de SharePoint desde un disco con MABS

En el ejemplo siguiente, el *elemento de recuperación de SharePoint* se eliminó accidentalmente y es necesario recuperarlo.
![Protección de SharePoint con MABS4](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection5.png)

1. Abra la **Consola de administrador de MABS**. Todas las granjas de SharePoint que están protegidas con MABS se muestran en la pestaña **Protección**.

    ![Protección de SharePoint con MABS3](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection4.png)
2. Para empezar a recuperar el elemento, seleccione la pestaña **Recuperación** .

    ![Protección de SharePoint con MABS5](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection6.png)
3. Puede buscar en SharePoint el *elemento de recuperación* mediante caracteres comodín dentro un intervalo de puntos de recuperación.

    ![Protección de SharePoint con MABS6](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection7.png)
4. Seleccione el punto de recuperación adecuado en los resultados de búsqueda, haga clic con el botón derecho en él y elija **Recuperar**.
5. También puede desplazarse por varios puntos de recuperación y seleccionar una base de datos o un elemento para recuperar. Seleccione **Fecha > Hora de recuperación** y luego el elemento **Base de datos > Granja de SharePoint > Punto de recuperación > adecuado**.

    ![Protección de SharePoint con MABS7](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection8.png)
6. Haga clic con el botón derecho en el elemento y seleccione**Recuperar** para abrir el **Asistente para recuperación**. Seleccione **Next** (Siguiente).

    ![Revisar selección de recuperación](./media/backup-azure-backup-sharepoint/review-recovery-selection.png)
7. Seleccione el tipo de recuperación que quiere realizar y, a continuación, elija **Siguiente**.

    ![Tipo de recuperación](./media/backup-azure-backup-sharepoint/select-recovery-type.png)

   > [!NOTE]
   > La selección de **Recuperar en sitio original** en el ejemplo, recupera el elemento en el sitio de SharePoint original.
   >
   >
8. Seleccione el **proceso de recuperación** que quiere usar.

   * Seleccione **Recuperar sin una granja de servidores de recuperación** si la granja de SharePoint no ha cambiado y es la misma que el punto de recuperación que se restaura.
   * Seleccione **Recuperar con una granja de servidores de recuperación** si la granja de servidores de SharePoint ha cambiado desde que se creó el punto de recuperación.

     ![proceso de recuperación](./media/backup-azure-backup-sharepoint/recovery-process.png)
9. Proporcione una ubicación provisional para la instancia de SQL Server para recuperar la base de datos temporalmente. Proporcione también un recurso compartido de archivos de almacenamiento provisional en MABS y el servidor que ejecuta SharePoint para recuperar el elemento.

    ![Ubicación provisional1](./media/backup-azure-backup-sharepoint/staging-location1.png)

    MABS conecta la base de datos de contenido que hospeda el elemento de SharePoint con la instancia temporal de SQL Server. Desde la base de datos de contenido, recupera el elemento y lo coloca en la ubicación del archivo de almacenamiento provisional en MABS. Ahora, el elemento recuperado en la ubicación de almacenamiento provisional debe exportarse a la ubicación provisional de la granja de SharePoint.

    ![Ubicación provisional2](./media/backup-azure-backup-sharepoint/staging-location2.png)
10. Seleccione **Especificar opciones de recuperación**y aplique la configuración de seguridad a la granja de SharePoint o aplique la configuración de seguridad del punto de recuperación. Seleccione **Next** (Siguiente).

    ![Opciones de recuperación](./media/backup-azure-backup-sharepoint/recovery-options.png)

    > [!NOTE]
    > Puede limitar el uso de ancho de banda de red. Esto minimiza el impacto en el servidor de producción durante las horas de producción.
    >
    >
11. Revise la información de resumen y, a continuación, seleccione **Recuperar** para empezar la recuperación del archivo.

    ![Resumen de recuperación](./media/backup-azure-backup-sharepoint/recovery-summary.png)
12. Ahora seleccione la pestaña **Supervisión** en la **Consola de administrador MABS** para ver el **Estado** de la recuperación.

    ![Estado de recuperación](./media/backup-azure-backup-sharepoint/recovery-monitoring.png)

    > [!NOTE]
    > El archivo ya se ha restaurado. Puede actualizar el sitio de SharePoint para comprobar el archivo restaurado.
    >
    >

## <a name="restore-a-sharepoint-database-from-azure-by-using-mabs"></a>Restauración de una base de datos de SharePoint de Azure con MABS

1. Para recuperar una base de datos de contenido de SharePoint, desplácese por los diversos puntos de recuperación (tal como se mostró anteriormente) y seleccione aquel que quiera recuperar.

    ![Protección de SharePoint con MABS8](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection9.png)
2. Haga doble clic en el punto de recuperación de SharePoint para mostrar la información de catálogo de SharePoint disponible.

   > [!NOTE]
   > Como la granja de SharePoint está protegida para la retención a largo plazo en Azure, no hay información de catálogo (metadatos) disponible en el servidor de MABS. Como resultado, cada vez que deba recuperar una base de datos de contenido de SharePoint en un momento dado, deberá volver a catalogar la granja de SharePoint.
   >
   >
3. Seleccione **Volver a catalogar**.

    ![Protección de SharePoint con MABS10](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection12.png)

    Aparece la ventana de estado **Volver a catalogar la nube** .

    ![Protección de SharePoint con MABS11](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection13.png)

    Cuando finaliza la catalogación, el estado cambia a *Correcto*. Seleccione **Cerrar**.

    ![Protección de SharePoint con MABS12](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection14.png)
4. Seleccione el objeto de SharePoint que se muestra en la pestaña **Recuperación** de MABS para obtener la estructura de la base de datos de contenido. Haga clic con el botón derecho en el elemento y, a continuación, seleccione **Recuperar**.

    ![Protección de SharePoint con MABS13](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection15.png)
5. En este momento, siga los pasos de recuperación que se han indicado en este artículo para recuperar una base de datos de contenido de SharePoint desde el disco.

## <a name="switching-the-front-end-web-server"></a>Cambio del servidor front-end web

Si tiene más de un servidor front-end web y desea cambiar el servidor que MABS usa para proteger la granja, siga las instrucciones:

El siguiente procedimiento usa el ejemplo de una granja de servidores con dos servidores web de front-end: *Servidor1* y *Servidor2*. MABS usa el *Servidor1* para proteger la granja. Cambie el servidor web de front-end que MABS usa para *Servidor2* para poder quitar el *Servidor1* de la granja.

> [!NOTE]
> Si el servidor web de front-end que usa MABS para proteger la granja no está disponible, utilice el siguiente procedimiento para cambiar el servidor web de front-end a partir del paso 4.

### <a name="to-change-the-front-end-web-server-that-mabs-uses-to-protect-the-farm"></a>Para cambiar el servidor front-end web que usa MABS para proteger la granja de servidores

1. Detenga el servicio VSS Writer de SharePoint en el *Servidor1* ejecutando el siguiente comando en un símbolo del sistema:

    ```CMD
    stsadm -o unregisterwsswriter
    ```

1. En el *Servidor1*, abra el Editor del Registro y desplácese hasta la clave siguiente:

   **HKLM\System\CCS\Services\VSS\VssAccessControl**

1. Compruebe todos los valores que figuran en la subclave VssAccessControl. Si alguna entrada tiene un valor de 0 y otra instancia de VSS Writer se está ejecutando en las credenciales de la cuenta asociada, cambie los datos del valor a 1.

1. Instale un agente de protección en el *Servidor2*.

   > [!WARNING]
   > Solo puede cambiar los servidores web de front-end si ambos servidores están en el mismo dominio.

1. En el *Servidor2*, en un símbolo del sistema, cambie el directorio a `_MABS installation location_\bin\` y ejecute **ConfigureSharepoint**. Para obtener más información sobre ConfigureSharePoint, vea [Configurar la copia de seguridad](#configure-backup).

1. Seleccione el grupo de protección al que pertenece la granja de servidores y, a continuación, elija **Modificar grupo de protección**.

1. En el Asistente para modificar grupo, en la página **Seleccionar miembros del grupo**, expanda *Servidor2* y seleccione la granja de servidores. A continuación, complete el asistente.

   Se iniciará una comprobación de coherencia.

1. Si realizó el paso 6, ahora puede quitar el volumen del grupo de protección.

## <a name="next-steps"></a>Pasos siguientes

* Consulte el artículo sobre [copia de seguridad de archivos y aplicaciones](backup-mabs-files-applications-azure-stack.md).
* Consulte el artículo sobre [copia de seguridad de SQL Server en Azure Stack](backup-mabs-sql-azure-stack.md).
