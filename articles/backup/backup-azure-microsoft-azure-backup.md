---
title: Uso de Azure Backup Server para realizar copias de seguridad de cargas de trabajo
description: En este artículo, aprenderá a preparar su entorno para proteger las cargas de trabajo y hacer copias de seguridad de ellas mediante Microsoft Azure Backup Server (MABS).
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: 74706e772371c39b96b0cb02bd09ec70ad4d43f6
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86539132"
---
# <a name="install-and-upgrade-azure-backup-server"></a>Instalación y actualización de Azure Backup Server

> [!div class="op_single_selector"]
>
> * [Azure Backup Server](backup-azure-microsoft-azure-backup.md)
> * [SCDPM](backup-azure-dpm-introduction.md)
>
>

> Se aplica a: MABS v3. (MABS v2 ya no se admite. Si usa una versión anterior a MABS v3, actualice a la versión más reciente).

En este artículo se explica cómo preparar el entorno para la copia de seguridad de las cargas de trabajo mediante Microsoft Azure Backup Server (MABS). Con Azure Backup Server, puede proteger cargas de trabajo de aplicaciones como máquinas virtuales de Hyper-V, Microsoft SQL Server, SharePoint Server, Microsoft Exchange y clientes Windows desde una única consola.

> [!NOTE]
> Azure Backup Server ahora puede proteger máquinas virtuales de VMware y proporciona mejores funcionalidades de seguridad. Instale el producto tal y como se explica en las secciones de debajo y el agente de Azure Backup más reciente. Para obtener más información sobre la copia de seguridad de servidores VMware con Azure Backup Server, vea el artículo [Uso de Azure Backup Server para hacer copia de seguridad de un servidor de VMware](backup-azure-backup-server-vmware.md). Para obtener información sobre las funcionalidades de seguridad, consulte la [documentación de características de seguridad de copia de seguridad de Azure](backup-azure-security-feature.md).
>
>

Una instancia de MABS implementada en una máquina virtual de Azure puede crear copias de seguridad de máquinas virtuales en Azure, pero deben encontrarse en el mismo dominio para habilitar dicha operación. El proceso para realizar una copia de una máquina virtual de Azure es el mismo que al realizar una copia de seguridad de máquinas virtuales locales, aunque la implementación de MABS en Azure tiene algunas limitaciones. Para más información sobre las limitaciones, consulte [DPM como máquina virtual de Azure](/system-center/dpm/install-dpm#setup-prerequisites).

> [!NOTE]
> Azure cuenta con dos modelos de implementación para crear recursos y trabajar con ellos: [Resource Manager y el clásico](../azure-resource-manager/management/deployment-models.md). En este artículo se proporcionan información y procedimientos para restaurar las máquinas virtuales implementadas mediante el modelo de Resource Manager.
>
>

Azure Backup Server hereda gran parte de la funcionalidad de copia de seguridad de las cargas de trabajo de Data Protection Manager (DPM). Este artículo incluye vínculos a documentación de DPM para explicar algunas de las funcionalidades compartidas. Aunque Azure Backup Server comparte muchas funcionalidades con DPM, Azure Backup Server no realiza copias de seguridad en cinta, ni se integra con System Center.

## <a name="choose-an-installation-platform"></a>Elección de una plataforma de instalación

El primer paso para que funcione Azure Backup Server es configurar un equipo con Windows Server. El servidor puede estar en Azure o en el entorno local.

* Para proteger las cargas de trabajo locales, el servidor MABS debe ubicarse en el entorno local.
* Para proteger las cargas de trabajo de máquinas virtuales de Azure, el servidor MABS debe estar ubicado en Azure y se debe ejecutar como una máquina virtual de Azure.

### <a name="using-a-server-in-azure"></a>Uso de un servidor en Azure

Al elegir un servidor para ejecutar Azure Backup Server, se recomienda comenzar con una imagen de la galería de Windows Server 2016 Datacenter o Windows Server 2019 Datacenter. En el artículo [Creación de la primera máquina virtual de Windows en el Portal de Azure](../virtual-machines/windows/quick-create-portal.md?toc=/azure/virtual-machines/windows/toc.json), se proporciona un tutorial de introducción a la máquina virtual recomendada en Azure, incluso si nunca ha usado Azure antes. Los requisitos mínimos recomendados para la máquina virtual servidor deben ser: Standard_A4_v2 con cuatro núcleos y 8 GB de RAM.

La protección de cargas de trabajo con Azure Backup Server tiene muchos matices. La [matriz de protección de MABS](./backup-mabs-protection-matrix.md) ayuda a explicar estos matices. Lea este artículo completamente antes de implementar la máquina.

### <a name="using-an-on-premises-server"></a>Uso de un servidor local

Si no desea ejecutar el servidor de base de Azure, puede ejecutar el servidor en una máquina virtual de Hyper-V, una máquina virtual de VMware o un host físico. Los requisitos mínimos recomendados para el hardware de servidor son dos núcleos y 8 GB de RAM. En la tabla siguiente se muestran los sistemas operativos compatibles:

| Sistema operativo | Plataforma | SKU |
|:--- | --- |:--- |
| Windows Server 2019 |64 bits |Standard, Datacenter, Essentials |
| Windows Server 2016 y SP más recientes |64 bits |Standard, Datacenter, Essentials  |

Puede desduplicar el almacenamiento de DPM con la desduplicación de Windows Server. Más información sobre cómo funcionan juntos [DPM y la desduplicación](/system-center/dpm/deduplicate-dpm-storage?view=sc-dpm-2019) al implementarlos en máquinas virtuales de Hyper-V.

> [!NOTE]
> Azure Backup Server está diseñado para ejecutarse en un servidor dedicado de objetivo único. No se puede instalar Azure Backup Server en:
>
> * Un equipo que se ejecuta como controlador de dominio
> * Un equipo en el que está instalado el rol del servidor de aplicaciones
> * Un equipo que sea un grupo de administración de System Center Operations Manager
> * Un equipo en el que se ejecute Exchange Server
> * Un equipo que sea un nodo de un clúster
>
> La instalación de Azure Backup Server no se admite en Windows Server Core ni Microsoft Hyper-V Server.

Siempre una Azure Backup Server a un dominio. Si piensa mover el servidor a un dominio diferente, instale primero Azure Backup Server y luego una el servidor al nuevo dominio. *No se permite*mover una máquina de Azure Backup Server existente a un dominio nuevo después de la implementación.

Si envía datos de copia de seguridad a Azure o los mantiene localmente, Azure Backup Server debe registrarse en un almacén de Recovery Services.

[!INCLUDE [backup-create-rs-vault.md](../../includes/backup-create-rs-vault.md)]

### <a name="set-storage-replication"></a>Configuración de la replicación de almacenamiento

La opción de replicación de almacenamiento permite elegir entre almacenamiento con redundancia geográfica y almacenamiento con redundancia local. De forma predeterminada, los almacenes de Recovery Services usan almacenamiento con redundancia geográfica. Si este almacén es su almacén principal, deje la opción de almacenamiento establecida en almacenamiento con redundancia geográfica. Elija el almacenamiento con redundancia local si desea una opción más económica que no sea tan duradera. Para más información sobre las opciones de almacenamiento [con redundancia geográfica](../storage/common/storage-redundancy.md) y [con redundancia local](../storage/common/storage-redundancy.md), consulte [Replicación de Azure Storage](../storage/common/storage-redundancy.md).

Para editar la configuración de replicación de almacenamiento:

1. En la hoja **Almacenes de Recovery Services**, haga clic en el almacén nuevo. En la sección **Configuración**, haga clic en **Propiedades**.
2. En **Propiedades**, en **Configuración de copia de seguridad**, haga clic en **Actualizar**.

3. Seleccione el tipo de replicación de almacenamiento y haga clic en **Guardar**.

     ![Establecimiento de la configuración de almacenamiento del nuevo almacén](./media/backup-try-azure-backup-in-10-mins/recovery-services-vault-backup-configuration.png)

## <a name="software-package"></a>Paquete de software

### <a name="downloading-the-software-package"></a>Descarga del paquete de software

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Si ya tiene abierto un almacén de Recovery Services, vaya al paso 3. Si no tiene abierto un almacén de Recovery Services pero está en Azure Portal, en el menú principal, haga clic en **Examinar**.

   * En la lista de recursos, escriba **Recovery Services**.
   * Cuando comience a escribir, la lista se filtrará en función de la entrada. Haga clic en **Almacenes de Recovery Services**cuando lo vea.

     ![Creación del almacén de Recovery Services, paso 1](./media/backup-azure-microsoft-azure-backup/open-recovery-services-vault.png)

     Aparece la lista de almacenes de Recovery Services.
   * En la lista de almacenes de Recovery Services, seleccione un almacén.

     Se abre el panel del almacén seleccionado.

     ![Hoja del almacén abierta](./media/backup-azure-microsoft-azure-backup/vault-dashboard.png)
3. La hoja **Configuración** se abre de forma predeterminada. Si está cerrada, haga clic en **Configuración** para abrirla.

    ![Hoja del almacén abierta](./media/backup-azure-microsoft-azure-backup/vault-setting.png)
4. Haga clic en **Copia de seguridad** para abrir el Asistente para introducción.

    ![Introducción a la copia de seguridad](./media/backup-azure-microsoft-azure-backup/getting-started-backup.png)

    En la hoja **Introducción a Backup** que se abre, **Backup Goals** (Objetivos de Backup) se selecciona automáticamente.

    ![Backup-goals-default-opened](./media/backup-azure-microsoft-azure-backup/getting-started.png)

5. En la hoja **Backup Goals**, en el menú desplegable **¿Dónde se ejecuta su carga de trabajo?** , seleccione **Local**.

    ![entorno local y cargas de trabajo como objetivos](./media/backup-azure-microsoft-azure-backup/backup-goals-azure-backup-server.png)

    En el menú desplegable **¿De qué quiere realizar una copia de seguridad?** , seleccione las cargas de trabajo que quiere proteger con Azure Backup Server y, después, haga clic en **Aceptar**.

    El asistente **Introducción a Backup** cambia la opción **Preparar infraestructura** para realizar copias de seguridad de las cargas de trabajo en Azure.

   > [!NOTE]
   > Si solo desea hacer copia de seguridad de archivos y carpetas, se recomienda utilizar el agente de Azure Backup y seguir las instrucciones del artículo [Primer análisis: Copia de seguridad de archivos y carpetas](./backup-windows-with-mars-agent.md). Si va a proteger más que archivos y carpetas o tiene pensado expandir las necesidades de protección en un futuro, seleccione esas cargas de trabajo.
   >
   >

    ![Cambio del Asistente para introducción](./media/backup-azure-microsoft-azure-backup/getting-started-prep-infra.png)

6. En la hoja **Preparar la infraestructura** que se abre, haga clic en los vínculos de **Descarga** para instalar Azure Backup Server y descargar las credenciales del almacén. Utilice las credenciales del almacén durante el registro de Azure Backup Server en el almacén de Servicios de recuperación. Los vínculos le llevan al Centro de descarga donde se puede descargar el paquete de software.

    ![Preparación de la infraestructura de Azure Backup Server](./media/backup-azure-microsoft-azure-backup/azure-backup-server-prep-infra.png)

7. Seleccione todos los archivos y haga clic en **Siguiente**. Descargue todos los archivos procedentes de la página de descarga de Microsoft Azure Backup y colóquelos en la misma carpeta.

    ![Centro de descarga 1](./media/backup-azure-microsoft-azure-backup/downloadcenter.png)

    Puesto que el tamaño de descarga de todos los archivos juntos es de más de 3 GB, con un vínculo de descarga a 10 Mbps, se puede tardar hasta sesenta minutos en completarla.

### <a name="extracting-the-software-package"></a>Extracción del paquete de software

Después de descargar todos los archivos, haga clic en **MicrosoftAzureBackupInstaller.exe**. Se inicia el **Asistente para instalación de Microsoft Azure Backup** , que extraerá los archivos de instalación en una ubicación especificada por el usuario. Siga con el asistente y haga clic en el botón **Extraer** para comenzar el proceso de extracción.

> [!WARNING]
> Se requieren al menos 4 GB de espacio libre para extraer los archivos de instalación.
>
>

![Asistente para instalación de Microsoft Azure Backup](./media/backup-azure-microsoft-azure-backup/extract/03.png)

Después de completar el proceso de extracción, active la casilla para iniciar el archivo *setup.exe* recién extraído y empezar a instalar Microsoft Azure Backup Server. Luego, haga clic en el botón **Finalizar**.

### <a name="installing-the-software-package"></a>Instalación del paquete de software

1. Haga clic en **Microsoft Azure Backup** para iniciar el asistente para la instalación.

    ![Asistente para instalación de Microsoft Azure Backup](./media/backup-azure-microsoft-azure-backup/launch-screen2.png)
2. En la pantalla de bienvenida, haga clic en **Siguiente**. Irá a la sección *Prerequisite Checks* (Comprobaciones de requisitos previos). En esta pantalla, haga clic en **Comprobar** para determinar si se cumplieron los requisitos previos de hardware y software para Azure Backup Server. Si se cumplieron correctamente todos los requisitos previos, verá un mensaje que indica que la máquina los cumple. Haga clic en el botón **Siguiente** .

    ![Azure Backup Server - Bienvenida y requisitos previos](./media/backup-azure-microsoft-azure-backup/prereq/prereq-screen2.png)
3. El paquete de instalación de Azure Backup Server se incluye con los archivos binarios de SQL Server necesarios. Cuando comience una nueva instalación de Azure Backup Server, debe elegir la opción **Instalar una nueva instancia de SQL Server con esta instalación** y hacer clic en el botón **Comprobar e instalar**. Una vez que los requisitos previos se instalen correctamente, haga clic en **Next**(Siguiente).

    >[!NOTE]
    >Si desea usar su propio servidor SQL Server, las versiones de SQL Server admitidas son SQL Server 2014 SP1 o posterior, 2016 y 2017.  Todas las versiones de SQL Server deben ser Estándar o Enterprise de 64 bits.
    >Azure Backup Server no funcionará con una instancia remota de SQL Server. La instancia que se utiliza en Azure Backup Server debe ser local. Si utiliza un servidor SQL Server existente para MABS, el programa de instalación de MABS solo permitirá utilizar *instancias con nombre* de SQL Server.

    ![Azure Backup Server - Comprobación de SQL](./media/backup-azure-microsoft-azure-backup/sql/01.png)

    Si se produce un error con una recomendación para reiniciar el equipo, proceda a reiniciar y haga clic en **Check Again**(Comprobar de nuevo). Si hay algún problema de configuración de SQL, vuelva a configurar SQL según las instrucciones de SQL y vuelva a intentar la instalación o actualización de MABS con la instancia existente de SQL.

   **Configuración manual**

   Cuando use su propia instancia de SQL, asegúrese de agregar builtin\Administrators al rol de administrador del sistema de la base de datos maestra.

    **Configuración de SSRS con SQL 2017**

    Cuando use su propia instancia de SQL 2017, deberá configurar SSRS manualmente. Después de configurar SSRS, asegúrese de que la propiedad de SSRS *IsInitialized* está establecida en *True*. Cuando se establece en True, MABS supone que SSRS ya está configurado y omite la configuración de SSRS.

    Se usarán los siguientes valores para la configuración de SSRS:
    * Cuenta de servicio: "Usar cuenta integrada" debe ser Servicio de red
    * URL de servicio web: "Directorio virtual" debe ser ReportServer_\<SQLInstanceName>
    * Base de datos: DatabaseName debe ser ReportServer$\<SQLInstanceName>
    * URL del portal web: "Virtual Directory" debe ser Reports_\<SQLInstanceName>

    [Obtenga más información](/sql/reporting-services/report-server/configure-and-administer-a-report-server-ssrs-native-mode) acerca de la configuración de SSRS.

    > [!NOTE]
    > Las licencias de SQL Server usadas como base de datos para MABS se rigen por los [Términos de los servicios en línea de Microsoft](https://www.microsoft.com/licensing/product-licensing/products) (OST). Según OST, SQL Server en conjunto con MABS solo se puede usar como base de datos para MABS.

4. Proporcione una ubicación donde instalar los archivos del Microsoft Azure Backup Server y haga clic en **Next**(Siguiente).

    ![Requisitos previos de Microsoft Azure Backup2](./media/backup-azure-microsoft-azure-backup/space-screen.png)

    La ubicación temporal es un requisito para hacer copias de seguridad en Azure. Asegúrese de que la ubicación temporal sea al menos el 5% de los datos cuya copia de seguridad se planea hacer en la nube. Para la protección de disco, deben configurarse discos independientes una vez completada la instalación. Para más información sobre los grupos de almacenamiento, consulte [Preparación del almacenamiento de datos](/system-center/dpm/plan-long-and-short-term-data-storage?view=sc-dpm-2019).
5. Proporcione una contraseña segura para las cuentas de usuario locales con permisos restringidos y haga clic en **Next**(Siguiente).

    ![Requisitos previos de Microsoft Azure Backup2](./media/backup-azure-microsoft-azure-backup/security-screen.png)
6. Seleccione si desea usar *Microsoft Update* para comprobar si hay actualizaciones y haga clic en **Next**(Siguiente).

   > [!NOTE]
   > Se recomienda que Windows Update se redirija a Microsoft Update, que ofrece actualizaciones importantes y de seguridad para Windows y otros productos como Microsoft Azure Backup Server.
   >
   >

    ![Requisitos previos de Microsoft Azure Backup2](./media/backup-azure-microsoft-azure-backup/update-opt-screen2.png)
7. Revise *Summary of Settings* (Resumen de la configuración) y haga clic en **Install**(Instalar).

    ![Requisitos previos de Microsoft Azure Backup2](./media/backup-azure-microsoft-azure-backup/summary-screen.png)
8. La instalación se realiza en fases. En la primera fase, se instala el agente de Microsoft Azure Recovery Services en el servidor. El asistente comprueba igualmente la conectividad a Internet. Si la conectividad a Internet está disponible, puede continuar con la instalación; de lo contrario, debe proporcionar los detalles del proxy para conectarse a Internet.

    El siguiente paso es configurar el agente de Microsoft Azure Recovery Services. Como parte de la configuración, tendrá que proporcionar las credenciales del almacén para registrar la máquina en el almacén de Servicios de recuperación. También proporcionará una frase de contraseña para cifrar y descifrar los datos enviados entre Azure y sus instalaciones. Puede generar una frase de contraseña automáticamente o proporcionar la suya propia, con un mínimo de 16 caracteres. Continúe con el asistente hasta que se haya configurado el agente.

    ![Requisitos previos de Azure Backup Server2](./media/backup-azure-microsoft-azure-backup/mars/04.png)
9. Una vez que se complete correctamente el registro de Microsoft Azure Backup Server, el asistente para instalación global prosigue con la instalación y configuración de los componentes de SQL Server y de Microsoft Azure Backup Server. Tras completarse la instalación de los componentes de SQL Server, se instalan los componentes de Azure Backup Server.

    ![Servidor de Azure Backup](./media/backup-azure-microsoft-azure-backup/final-install/venus-installation-screen.png)

Cuando el paso de instalación haya finalizado, se habrán creado también los iconos de escritorio del producto. Haga doble clic en el icono para iniciar el producto.

### <a name="add-backup-storage"></a>Incorporación de almacenamiento de copia de seguridad

La primera copia de seguridad se mantiene en el almacenamiento conectado a la máquina de Azure Backup Server. Para obtener más información acerca de los discos, consulte [Configuración de bloques de almacenamiento y almacenamiento en disco](./backup-mabs-add-storage.md).

> [!NOTE]
> Debe agregar el almacenamiento de copia de seguridad incluso si tiene pensado enviar los datos a Azure. En la arquitectura de Azure Backup Server actual, el almacén de Azure Backup contiene la *segunda* copia de los datos, mientras que el almacenamiento local contiene la primera (y obligatoria) copia de seguridad.
>
>

### <a name="install-and-update-the-data-protection-manager-protection-agent"></a>Actualización e instalación del agente de protección de Data Protection Manager

MABS usa el agente de protección de System Center Data Protection Manager. [Estos son los pasos](/system-center/dpm/deploy-dpm-protection-agent) para instalar el agente de protección en los servidores de protección.

En las secciones siguientes se describe cómo actualizar los agentes de protección para los equipos cliente.

1. En la consola de administrador de Backup Server, seleccione **Administración** > **Agentes**.

2. En el panel de información, seleccione los equipos cliente cuyo agente de protección quiere actualizar.

   > [!NOTE]
   > En la columna **Actualizaciones del agente** se indica cuándo está disponible una actualización del agente de protección para cada equipo protegido. En el panel **Acciones**, la acción **Actualización** solo está disponible cuando se selecciona un equipo protegido y las actualizaciones están disponibles.
   >
   >

3. Para instalar agentes de protección actualizados en los equipos seleccionados, en el panel **Acciones**, seleccione **Actualizar**.

4. Si un equipo cliente no está conectado a la red, aparecerá el estado **Actualización pendiente** en la columna **Estado del agente** mientras el equipo no se conecte a la red.

   Una vez que el equipo cliente se conecte a la red, aparecerá el estado **Actualizando** en la columna **Actualizaciones del agente** para el equipo cliente.

## <a name="move-mabs-to-a-new-server"></a>Migración de MABS a un servidor nuevo

Estos son los pasos en caso de que necesite migrar MABS a un nuevo servidor sin dejar de conservar el almacenamiento. Esto puede hacerse únicamente si todos los datos se encuentran en Modern Backup Storage.

  > [!IMPORTANT]
  >
  > * El nombre nuevo del servidor debe ser el mismo que el de la instancia original de Azure Backup Server. No puede cambiar el nombre de la nueva instancia de Azure Backup Server si desea usar el anterior bloque de almacenamiento y base de datos de MABS (DPMDB) para conservar los puntos de recuperación.
  > * Debe tener una copia de seguridad de la base de datos de MABS (DPMDB). Tendrá que restaurar la base de datos.

1. En el panel de información, seleccione los equipos cliente cuyo agente de protección quiere actualizar.
2. Cierre la instancia original de Azure Backup Server o desconéctela.
3. Restablezca la cuenta de máquina en Active Directory.
4. Instale Server 2016 en un equipo nuevo y asígnele el mismo nombre de máquina que la instancia original de Azure Backup Server.
5. Únase al dominio
6. Instale Azure Backup Server V3 o versiones posteriores (mueva los discos del grupo de almacenamiento de MABS desde el servidor antiguo e impórtelos).
7. Restaure la base de datos de DPM realizada en el paso 1.
8. Conecte el almacenamiento del servidor de copia de seguridad original al nuevo servidor.
9. Desde SQL, restaure la base de datos DPM
10. Desde la línea de comandos de administrador en el nuevo servidor, cambie al directorio de la ubicación de instalación de Microsoft Azure Backup y a la carpeta bin

    Ejemplo de ruta de acceso: C:\windows\system32>cd "c:\Program Files\Microsoft Azure Backup\DPM\DPM\bin\"

11. A copia de seguridad de Azure. Ejecute DPMSYNC -SYNC

    Si ha agregado NUEVOS discos al bloque de almacenamiento DPM en lugar de mover los antiguos, ejecute DPMSYNC -Reallocatereplica

## <a name="network-connectivity"></a>Conectividad de red

Azure Backup Server requiere conectividad al servicio Azure Backup para que el producto funcione correctamente. Para validar si la máquina tiene conectividad a Azure, use el cmdlet ```Get-DPMCloudConnection``` en la consola de PowerShell de Azure Backup Server. Si la salida del cmdlet es TRUE, entonces existe conectividad, de lo contrario, no hay conectividad.

Además, la suscripción de Azure debe encontrarse en un estado correcto. Para averiguar el estado de la suscripción y administrarla, inicie sesión en el [portal de suscripción](https://account.windowsazure.com/Subscriptions).

Una vez que conozca el estado de la conectividad y suscripción de Azure, puede usar la tabla siguiente para saber el impacto en la funcionalidad de copia de seguridad y restauración que se ofrece.

| Estado de conectividad | Suscripción de Azure | Copia de seguridad en Azure | Copia de seguridad en disco | Restauración desde Azure | Restauración desde disco |
| --- | --- | --- | --- | --- | --- |
| Conectado |Active |Permitida |Permitida |Permitida |Permitida |
| Conectado |Expirada |Detenido |Detenido |Permitida |Permitida |
| Conectado |Desaprovisionada |Detenido |Detenido |Detenida y puntos de recuperación de Azure eliminados |Detenido |
| Pérdida de conectividad > 15 días |Active |Detenido |Detenido |Permitida |Permitida |
| Pérdida de conectividad > 15 días |Expirada |Detenido |Detenido |Permitida |Permitida |
| Pérdida de conectividad > 15 días |Desaprovisionada |Detenido |Detenido |Detenida y puntos de recuperación de Azure eliminados |Detenido |

### <a name="recovering-from-loss-of-connectivity"></a>Recuperación de una pérdida de conectividad

Si tiene un firewall o un proxy que impiden el acceso a Azure, deberá permitir primero las siguientes direcciones de dominio en el perfil del firewall/proxy:

* `http://www.msftncsi.com/ncsi.txt`
* \*.Microsoft.com
* \*.WindowsAzure.com
* \*.microsoftonline.com
* \*.windows.net

Si usa el emparejamiento de Microsoft de ExpressRoute, seleccione los siguientes servicios o regiones:

* Azure Active Directory (12076:5060)
* Región de Microsoft Azure (según la ubicación del almacén de Recovery Services)
* Azure Storage (según la ubicación del almacén de Recovery Services)

Para más información, consulte [Requisitos de enrutamiento de ExpressRoute](../expressroute/expressroute-routing.md).

Una vez restaurada la conectividad a Azure en la máquina de Azure Backup Server, las operaciones que pueden realizarse dependen del estado de la suscripción de Azure. La tabla anterior incluye detalles acerca de las operaciones permitidas una vez que la máquina esté "Conectada".

### <a name="handling-subscription-states"></a>Control de los estados de la suscripción

Es posible llevar una suscripción de Azure desde un estado *Expirado* o *Desaprovisionado* hasta un estado *Activo*. Sin embargo, esto tiene algunos efectos sobre el comportamiento del producto mientras el estado no sea *Activo*:

* Una suscripción con estado *Desaprovisionado* pierde la funcionalidad durante el período en que está desaprovisionada. Al pasar a *Activo*, se reactiva la funcionalidad del producto de copia de seguridad y restauración. Los datos de copia de seguridad del disco local también pueden recuperarse en caso de que se haya mantenido con un período de retención lo suficientemente amplio. No obstante, los datos de copia de seguridad de Azure se pierden irremediablemente una vez que la suscripción pasa al estado *Desaprovisionado* .
* Una suscripción con estado *Expirado* solo pierde la funcionalidad hasta que pase de nuevo al estado *Activo*. Las copias de seguridad programadas para el período en el que la suscripción tenía el estado *Expirado* no se ejecutarán.

## <a name="upgrade-mabs"></a>Actualización de MABS

Use los procedimientos siguientes para actualizar MABS.

### <a name="upgrade-from-mabs-v2-to-v3"></a>Actualización de MABS V2 a V3

> [!NOTE]
>
> MABS V2 no es un requisito previo para la instalación de MABS V3. Sin embargo, puede solo puede actualizar a MABS V3 desde MABS V2.

Para actualizar MABS, realice los siguientes pasos:

1. Para actualizar MABS V2 a MABS V3, actualice su sistema operativo a Windows Server 2016 o Windows Server 2019 si es necesario.

2. Actualice su servidor. Los pasos son similares a los de la [instalación](#install-and-upgrade-azure-backup-server). Sin embargo, para la configuración de SQL, obtendrá una opción para actualizar la instancia de SQL a SQL 2017, o para usar su propia instancia de SQL Server 2017.

   > [!NOTE]
   >
   > No salga mientras se está actualizando la instancia de SQL; si lo hace, la instancia de informes de SQL se desinstalará y, por tanto, se producirá un error cuando intente volver a actualizar MABS.

   > [!IMPORTANT]
   >
   >  Como parte de la actualización a SQL 2017, se realiza una copia de seguridad de las claves de cifrado de SQL y se desinstalan los servicios de informes. Después de la actualización de SQL Server, se instala el servicio de informes (14.0.6827.4788) y se restauran las claves de cifrado.
   >
   > Si configura manualmente SQL 2017, consulte la sección *Configuración de SSRS con SQL 2017* de las Instrucciones de instalación.

3. Actualice los agentes de protección de los servidores protegidos.
4. Las copias de seguridad deben continuar sin necesidad de reiniciar los servidores de producción.
5. Puede comenzar a proteger sus datos ahora mismo. Si va a actualizar a Modern Backup Storage, al tiempo que protege, también puede elegir en qué volúmenes quiere almacenar las copias de seguridad y comprobar la existencia de espacio insuficiente. [Más información](backup-mabs-add-storage.md).

## <a name="troubleshooting"></a>Solución de problemas

Si el servidor de Microsoft Azure Backup produce un error durante la fase de instalación (o copia de seguridad o restauración), consulte este [documento de códigos de error](https://support.microsoft.com/kb/3041338) para más información.
También puede consultar [Azure Backup - Preguntas más frecuentes](backup-azure-backup-faq.md)

## <a name="next-steps"></a>Pasos siguientes

Puede obtener información detallada sobre la [preparación del entorno para DPM](/system-center/dpm/prepare-environment-for-dpm?view=sc-dpm-2019). También contiene información sobre las configuraciones admitidas en las que se puede implementar y usar Azure Backup Server. Puede usar una serie de [cmdlets de PowerShell](/powershell/module/dataprotectionmanager/) para realizar diversas operaciones.

Puede usar estos artículos para mejorar la comprensión sobre la protección de cargas de trabajo mediante el servidor Microsoft Azure Backup Server.

* [Copia de seguridad de SQL Server](backup-azure-backup-sql.md)
* [Copia de seguridad de una granja de SharePoint](backup-azure-backup-sharepoint.md)
* [Copia de seguridad de otro servidor](backup-azure-alternate-dpm-server.md)
