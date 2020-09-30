---
title: Instalación de Azure Backup Server en Azure Stack
description: En este artículo, aprenderá a usar Azure Backup Server para proteger cargas de trabajo o realizar una copia de seguridad de ellas en Azure Stack.
ms.topic: conceptual
ms.date: 01/31/2019
ms.openlocfilehash: a95ee42c3229e642bd48b812eb6bc01636af3ba2
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018134"
---
# <a name="install-azure-backup-server-on-azure-stack"></a>Instalación de Azure Backup Server en Azure Stack

En este artículo se explica cómo instalar Azure Backup Server en Azure Stack. Con Azure Backup Server, puede proteger cargas de trabajo de Infraestructura como servicio (IaaS), como por ejemplo, máquinas virtuales que se ejecutan en Azure Stack. Una ventaja de usar Azure Backup Server para proteger las cargas de trabajo es que puede administrar toda la protección de las cargas de trabajo desde una sola consola.

> [!NOTE]
> Para información sobre las funcionalidades de seguridad, consulte la [documentación sobre las características de seguridad de Azure Backup](backup-azure-security-feature.md).
>

## <a name="azure-backup-server-protection-matrix"></a>Matriz de protección de Azure Backup Server

Azure Backup Server protege las siguientes cargas de trabajo de máquina virtual de Azure Stack:

| Origen de datos protegido | Protección y recuperación |
| --------------------- | ----------------------- |
| Canal semianual de Windows Server: Datacenter/Enterprise/Standard | Volúmenes, archivos, carpetas |
| Windows Server 2016: Datacenter/Enterprise/Standard | Volúmenes, archivos, carpetas |
| Windows Server 2012 R2: Datacenter/Enterprise/Standard | Volúmenes, archivos, carpetas |
| Windows Server 2012: Datacenter/Enterprise/Standard | Volúmenes, archivos, carpetas |
| Windows Server 2008 R2: Datacenter/Enterprise/Standard | Volúmenes, archivos, carpetas |
| SQL Server 2016 | Base de datos |
| SQL Server 2014 | Base de datos |
| SQL Server 2012 SP1 | Base de datos |
| SharePoint 2016 | Granja, base de datos, front-end, servidor web |
| SharePoint 2013 | Granja, base de datos, front-end, servidor web |
| SharePoint 2010 | Granja, base de datos, front-end, servidor web |

## <a name="prerequisites-for-the-azure-backup-server-environment"></a>Requisitos previos para el entorno de Azure Backup Server

Tenga en cuenta las recomendaciones de esta sección cuando instale Azure Backup Server en el entorno de Azure Stack. El instalador de Azure Backup Server revisa si el entorno tiene los requisitos previos necesarios, pero puede prepararse antes de la instalación para ahorrar tiempo.

### <a name="determining-size-of-virtual-machine"></a>Determinar el tamaño de la máquina virtual

Para ejecutar Azure Backup Server en una máquina virtual de Azure Stack, use el tamaño A2 o superior. Si quiere obtener ayuda para elegir un tamaño de máquina virtual, descargue la [Calculadora de tamaño de VM de Azure Stack](https://www.microsoft.com/download/details.aspx?id=56832).

### <a name="virtual-networks-on-azure-stack-virtual-machines"></a>Redes virtuales en máquinas virtuales de Azure Stack

Toda máquina virtual que se usa en una carga de trabajo de Azure Stack debe pertenecer a la misma red virtual de Azure y suscripción a Azure.

### <a name="azure-backup-server-vm-performance"></a>Rendimiento de VM de Azure Backup Server

Si se comparte con otras máquinas virtuales, el tamaño de la cuenta de almacenamiento y los límites de IOPS afectan el rendimiento de la máquina virtual de Azure Backup Server. Por este motivo, debe utilizar una cuenta de almacenamiento independiente para la máquina virtual de Azure Backup Server. El agente de Azure Backup que se ejecuta en Azure Backup Server necesita almacenamiento temporal para:

- su propio uso (una ubicación en caché),
- datos restaurados de la nube (área de almacenamiento provisional local)

### <a name="configuring-azure-backup-temporary-disk-storage"></a>Configurar el almacenamiento temporal en el disco de Azure Backup

Cada máquina virtual de Azure Stack viene con un almacenamiento temporal en el disco, que está disponible para el usuario como volumen `D:\`. El área de almacenamiento provisional local necesaria para Azure Backup se puede configurar para residir en `D:\`, y la ubicación en caché puede colocarse en `C:\`. De esta manera, no es necesario reducir el almacenamiento de los discos de datos conectados a la máquina virtual de Azure Backup Server.

### <a name="storing-backup-data-on-local-disk-and-in-azure"></a>Almacenamiento de datos de copia de seguridad en un disco local y en Azure

Azure Backup Server almacena los datos de copia de seguridad en discos de Azure conectados a la máquina virtual, para la recuperación operativa. Una vez que los discos y el espacio de almacenamiento están conectados a la máquina virtual, Azure Backup Server gestiona el almacenamiento por usted. La cantidad de almacenamiento de datos de copia de seguridad depende del número y tamaño de los discos conectados a cada [máquina virtual de Azure Stack](/azure-stack/user/azure-stack-storage-overview). Cada tamaño de VM de Azure Stack tiene un número máximo de discos que se pueden conectar a la máquina virtual. Por ejemplo, A2 son cuatro discos. A3 son ocho discos. A4 son 16 discos. De nuevo, el tamaño y número de discos determina el grupo de almacenamiento de copia de seguridad total.

> [!IMPORTANT]
> **No** debería conservar datos de recuperación operativa (copia de seguridad) en discos conectados directamente a Azure Backup Server durante más de cinco días.
>

El almacenamiento de datos de copia de seguridad de Azure reduce la infraestructura de copia de seguridad en Azure Stack. Si los datos tienen más de cinco días de antigüedad, se deben almacenar en Azure.

Para almacenar datos de copia de seguridad en Azure, cree o use un almacén de Recovery Services. Cuando se prepare para hacer una copia de seguridad de la carga de trabajo de Azure Backup Server, [configure el almacén de Recovery Services](backup-azure-microsoft-azure-backup.md#create-a-recovery-services-vault). Una vez configurado, cada vez que se ejecuta un trabajo de copia de seguridad, se crea un punto de recuperación en el almacén. Cada almacén de Recovery Services contiene hasta 9999 puntos de recuperación. En función del número de puntos de recuperación creados y del tiempo que se conserven, se pueden conservar los datos de copias de seguridad durante muchos años. Por ejemplo, podría crear puntos de recuperación mensuales y conservarlos durante cinco años.

### <a name="scaling-deployment"></a>Escalado de la implementación

Si quiere escalar su implementación, tiene las siguientes opciones:

- Escalar verticalmente: aumente el tamaño de la máquina virtual Azure Backup Server de la serie A a la serie D y aumente el almacenamiento local [según las instrucciones de la máquina virtual de Azure Stack](/azure-stack/user/azure-stack-manage-vm-disks).
- Descargar datos: envíe los datos antiguos a Azure y conserve solo los datos más recientes en el almacenamiento conectado a Azure Backup Server.
- Escalar horizontalmente: agregue más instancias de Azure Backup Server para proteger las cargas de trabajo.

### <a name="net-framework"></a>.NET Framework

.NET Framework 3.5 SP1 o una versión posterior debe estar instalado en la máquina virtual.

### <a name="joining-a-domain"></a>Unión a un dominio

La máquina virtual de Azure Backup Server debe unirse a un dominio. El usuario de un dominio con privilegios de administrador debe instalar Azure Backup Server en la máquina virtual.

## <a name="using-an-iaas-vm-in-azure-stack"></a>Uso de una máquina virtual de IaaS en Azure Stack

Al elegir un servidor para Azure Backup Server, empiece con una imagen de la galería de Windows Server 2012 R2 Datacenter o Windows Server 2016 Datacenter. En el artículo [Create your first Windows virtual machine in the Azure portal](../virtual-machines/windows/quick-create-portal.md?toc=/azure/virtual-machines/windows/toc.json) (Creación de la primera máquina virtual de Windows en Azure Portal), se proporciona un tutorial de introducción a la máquina virtual recomendada. Los requisitos mínimos recomendados para la máquina virtual servidor deben ser: A2 estándar con 2 núcleos y 3,5 GB de RAM.

La protección de cargas de trabajo con Azure Backup Server tiene muchos matices. La [matriz de protección de MABS](./backup-mabs-protection-matrix.md) ayuda a explicar estos matices. Lea este artículo completamente antes de implementar la máquina.

> [!NOTE]
> Azure Backup Server está diseñado para ejecutarse en una máquina virtual dedicada para un único objetivo. No se puede instalar Azure Backup Server en:
>
> - Un equipo que se ejecuta como controlador de dominio
> - Un equipo en el que está instalado el rol del servidor de aplicaciones
> - Un equipo en el que se ejecute Exchange Server
> - Un equipo que sea un nodo de un clúster

Siempre una Azure Backup Server a un dominio. Si necesita mover Azure Backup Server a otro dominio, instálelo primero y luego únalo al dominio nuevo. Una vez que implemente Azure Backup Server, no puede moverlo a un dominio nuevo.

[!INCLUDE [backup-create-rs-vault.md](../../includes/backup-create-rs-vault.md)]

### <a name="set-storage-replication"></a>Configuración de la replicación de almacenamiento

La opción de replicación de almacenamiento de almacén de Recovery Services permite elegir entre almacenamiento con redundancia geográfica y almacenamiento con redundancia local. De forma predeterminada, los almacenes de Recovery Services usan almacenamiento con redundancia geográfica. Si este almacén es su almacén principal, deje la opción de almacenamiento establecida en almacenamiento con redundancia geográfica. Elija un almacenamiento con redundancia local si quiere una opción más económica que sea menos duradera. Para más información sobre las opciones de almacenamiento [con redundancia geográfica](../storage/common/storage-redundancy.md) y [con redundancia local](../storage/common/storage-redundancy.md), consulte [Replicación de Azure Storage](../storage/common/storage-redundancy.md).

Para editar la configuración de replicación de almacenamiento:

1. Seleccione el almacén para abrir su panel y el menú de configuración. Si el menú **Configuración** no se abre, seleccione **Toda la configuración** en el panel del almacén.
2. En el menú **Configuración**, seleccione **Infraestructura de Backup** > **Configuración de copia de seguridad** para abrir el menú **Configuración de copia de seguridad**. En el menú **Configuración de copia de seguridad**, elija la opción de replicación del almacenamiento para su almacén.

    ![Lista de copias de seguridad](./media/backup-azure-vms-first-look-arm/choose-storage-configuration-rs-vault.png)

## <a name="download-azure-backup-server-installer"></a>Descarga del instalador de Azure Backup Server

Hay dos formas de descargar el instalador de Azure Backup Server. Puede descargar el instalador de Azure Backup Server del [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=55269). También puede descargarlo mientras configura un almacén de Recovery Services. Los pasos siguientes lo guiarán por el proceso de descarga del instalador desde Azure Portal mientras configura un almacén de Recovery Services.

1. En la máquina virtual de Azure Stack, [inicie sesión en su suscripción de Azure en Azure Portal](https://portal.azure.com/).
2. En el menú izquierdo, seleccione **Todos los servicios**.

    ![Elija la opción Todos los servicios en el menú principal](./media/backup-mabs-install-azure-stack/click-all-services.png)

3. En el cuadro de diálogo **Todos los servicios**, escriba *Recovery Services*. En cuanto empiece a escribirlo, la entrada filtra la lista de recursos. Cuando vea la opción **Almacenes de Recovery Services**, selecciónela.

    ![Escriba Recovery Services en el cuadro de diálogo Todos los servicios](./media/backup-mabs-install-azure-stack/all-services.png)

    Aparece la lista de almacenes de Recovery Services de la suscripción.

4. En la lista de almacenes de Recovery Services, seleccione su almacén para abrir su panel.

    ![Seleccionar el almacén para abrir el panel](./media/backup-mabs-install-azure-stack/rs-vault-dashboard.png)

5. En el menú Introducción del almacén, seleccione **Copia de seguridad** para abrir el Asistente de introducción.

    ![Introducción a la copia de seguridad](./media/backup-mabs-install-azure-stack/getting-started-backup.png)

    Se abre el menú de la copia de seguridad.

    ![Backup-goals-default-opened](./media/backup-mabs-install-azure-stack/getting-started-menu.png)

6. En el menú **¿Dónde se ejecuta su carga de trabajo?** del menú de la copia de seguridad, seleccione **Local**. En el menú desplegable **¿De qué quiere realizar una copia de seguridad?** , seleccione las cargas de trabajo que quiere proteger con Azure Backup Server. Si no está seguro de las cargas de trabajo que quiere seleccionar, elija **Máquinas virtuales de Hyper-V** y seleccione **Preparar infraestructura**.

    ![entorno local y cargas de trabajo como objetivos](./media/backup-mabs-install-azure-stack/getting-started-menu-onprem-hyperv.png)

    Se abre el menú **Prepare infrastructure** (Preparar infraestructura).

7. En el menú **Preparar infraestructura**, seleccione **Descargar** para abrir una página web donde descargar los archivos de instalación de Azure Backup Server.

    ![Cambio del Asistente para introducción](./media/backup-mabs-install-azure-stack/prepare-infrastructure.png)

    Se abre la página web de Microsoft que hospeda los archivos descargables para Azure Backup Server.

8. En la página de descarga de Microsoft Azure Backup Server, seleccione un idioma y, después, **Descargar**.

    ![Se abre el centro de descarga](./media/backup-mabs-install-azure-stack/mabs-download-center-page.png)

9. El instalador de Azure Backup Server consta de ocho archivos: un instalador y siete archivos .bin. Active **Nombre de archivo** para seleccionar todos los archivos necesarios y seleccione **Siguiente**. Descargue todos los archivos en la misma carpeta.

    ![Centro de descarga, archivos seleccionados](./media/backup-mabs-install-azure-stack/download-center-selected-files.png)

    El tamaño de descarga de todos los archivos de instalación es superior a 3 GB. En un vínculo de descarga de 10 Mbps, descargar todos los archivos de instalación puede tardar hasta 60 minutos. Los archivos se descargan en la ubicación de descarga que especificó.

## <a name="extract-azure-backup-server-install-files"></a>Extracción de los archivos de instalación de Azure Backup Server

Una vez que descargue todos los archivos en la máquina virtual de Azure Stack, vaya a la ubicación de descarga. La primera fase de la instalación de Azure Backup Server es extraer los archivos.

![Descargar el instalador de MABS](./media/backup-mabs-install-azure-stack/download-mabs-installer.png)

1. Para empezar la instalación, en la lista de archivos descargados, seleccione **MicrosoftAzureBackupserverInstaller.exe**.

    > [!WARNING]
    > Se requieren al menos 4 GB de espacio libre para extraer los archivos de instalación.
    >

2. En el Asistente de Azure Backup Server, seleccione **Siguiente** para continuar.

    ![Asistente para instalación de Microsoft Azure Backup](./media/backup-mabs-install-azure-stack/mabs-install-wiz-1.png)

3. Elija la ruta de acceso de los archivos de Azure Backup Server y seleccione **Siguiente**.

   ![Seleccionar destino para los archivos](./media/backup-mabs-install-azure-stack/mabs-install-wizard-select-destination-1.png)

4. Compruebe la ubicación de extracción y seleccione **Extraer**.

   ![Comprobar ubicación de extracción](./media/backup-mabs-install-azure-stack/mabs-install-wizard-extract-2.png)

5. El asistente extrae los archivos y prepara el proceso de instalación.

   ![El asistente extrae los archivos](./media/backup-mabs-install-azure-stack/mabs-install-wizard-install-3.png)

6. Una vez completado el proceso de extracción, seleccione **Finalizar**. De manera predeterminada, se selecciona la opción **Execute setup.exe** (Ejecutar setup.exe). Cuando seleccione **Finalizar**, Setup.exe instalará Microsoft Azure Backup Server en la ubicación especificada.

   ![El archivo de instalación extrae los archivos de Microsoft Azure Backup Server](./media/backup-mabs-install-azure-stack/mabs-install-wizard-finish-4.png)

## <a name="install-the-software-package"></a>Instalación del paquete de software

En el paso anterior, seleccione **Finalizar** para salir de la fase de extracción e iniciar el Asistente para instalación de Azure Backup Server.

![El Asistente para instalación de Microsoft Azure Backup se inicia](./media/backup-mabs-install-azure-stack/mabs-install-wizard-local-5.png)

Azure Backup Server comparte código con Data Protection Manager. En el instalador de Azure Backup Server verá referencias a Data Protection Manager y a DPM. Si bien Azure Backup Server y Data Protection Manager son productos independientes, son productos estrechamente relacionados.

1. Para iniciar el Asistente para la instalación, haga clic en **Microsoft Azure Backup Server**.

   ![Seleccionar Microsoft Azure Backup Server](./media/backup-mabs-install-azure-stack/mabs-install-wizard-local-5b.png)

2. En la pantalla de **bienvenida**, seleccione **Siguiente**.

    ![Pantalla de bienvenida de Azure Backup Server](./media/backup-mabs-install-azure-stack/mabs-install-wizard-setup-6.png)

3. En la pantalla **Comprobaciones de requisitos previos**, seleccione **Comprobar** para determinar si se cumplieron los requisitos previos de hardware y software para Azure Backup Server.

    ![Comprobación de requisitos previos de Azure Backup Server](./media/backup-mabs-install-azure-stack/mabs-install-wizard-pre-check-7.png)

    Si el entorno cuenta con los requisitos previos necesarios, verá un mensaje que indica que la máquina cumple con los requisitos. Seleccione **Next** (Siguiente).  

    ![Azure Backup Server - Comprobación de requisitos previos aprobada](./media/backup-mabs-install-azure-stack/mabs-install-wizard-pre-check-passed-8.png)

    Si el entorno no cumple con los requisitos previos necesarios, se especificarán los problemas. Los requisitos previos que no se cumplieron también aparecen en el archivo DpmSetup.log. Resuelva los errores de requisitos previos y ejecute **Check Again** (Volver a comprobar). No es posible continuar con la instalación si no se cumplen todos los requisitos previos.

    ![Azure Backup Server - Requisitos previos de instalación no cumplidos](./media/backup-mabs-install-azure-stack/installation-errors.png)

4. Microsoft Azure Backup Server requiere SQL Server. El paquete de instalación de Azure Backup Server se incluye con los archivos binarios de SQL Server adecuados. Si quiere usar su propia instalación de SQL, puede hacerlo. Sin embargo, la opción recomendada es permitir que el instalador agregue una instancia de SQL Server nueva. Para asegurarse de que la opción que elige funciona con el entorno, seleccione **Comprobar e instalar**.

   > [!NOTE]
   > Azure Backup Server no funcionará con una instancia remota de SQL Server. La instancia que Azure Backup Server usa debe ser local.
   >

    ![Azure Backup Server (configuración de SQL)](./media/backup-mabs-install-azure-stack/mabs-install-wizard-sql-install-9.png)

    Después de realizar la comprobación, si la máquina virtual cuenta con los requisitos previos necesarios para instalar Azure Backup Server, seleccione **Siguiente**.

    ![Azure Backup Server (requisitos cumplidos)](./media/backup-mabs-install-azure-stack/mabs-install-wizard-sql-ready-10.png)

    Si se produce un error con una recomendación para reiniciar la máquina, proceda a reiniciarla. Después de reiniciar la máquina, reinicie el instalador y, cuando vea la pantalla **Configuración de SQL**, seleccione **Volver a comprobar**.

5. En **Configuración de la instalación**, indique una ubicación para la instalación de los archivos de Microsoft Azure Backup Server y seleccione **Siguiente**.

    ![Proporcionar la ubicación para la instalación de los archivos](./media/backup-mabs-install-azure-stack/mabs-install-wizard-settings-11.png)

    La ubicación temporal es un requisito para crear copias de seguridad en Azure. Asegúrese de que el tamaño de la ubicación temporal sea equivalente al menos al 5 % de los datos cuya copia de seguridad planea hacer en Azure. Para la protección de disco, deben configurarse discos independientes una vez completada la instalación. Para más información sobre los grupos de almacenamiento, vea [Preparación del almacenamiento de datos](/system-center/dpm/plan-long-and-short-term-data-storage?view=sc-dpm-2019).

6. En la pantalla **Configuración de seguridad**, escriba una contraseña segura para las cuentas de usuario locales con permisos restringidos y seleccione **Siguiente**.

    ![Pantalla de configuración de seguridad](./media/backup-mabs-install-azure-stack/mabs-install-wizard-security-12.png)

7. En la pantalla **Participación en Microsoft Update**, seleccione si quiere usar *Microsoft Update* para buscar actualizaciones y seleccione **Siguiente**.

   > [!NOTE]
   > Se recomienda que Windows Update se redirija a Microsoft Update, que ofrece actualizaciones importantes y de seguridad para Windows y otros productos como Microsoft Azure Backup Server.
   >

    ![Pantalla de Participación en Microsoft Update](./media/backup-mabs-install-azure-stack/mabs-install-wizard-update-13.png)

8. Revise la página *Resumen de la configuración* y seleccione **Instalar**.

    ![Resumen de la configuración](./media/backup-mabs-install-azure-stack/mabs-install-wizard-summary-14.png)

    Cuando finalice la instalación de Azure Backup Server, el instalador inicia de inmediato el instalador del agente de Microsoft Azure Recovery Services.

9. Se abre el instalador del agente de Microsoft Azure Recovery Services y comprueba la conectividad de Internet. Si se dispone de conectividad a Internet, continúe con la instalación. Si no hay conectividad, proporcione detalles del proxy para conectarse a Internet. Una vez que haya especificado la configuración del proxy, seleccione **Siguiente**.

    ![Configuración de proxy](./media/backup-mabs-install-azure-stack/mabs-install-wizard-proxy-15.png)

10. Para instalar el agente de Microsoft Azure Recovery Services Agent, seleccione **Instalar**.

    ![Instalación del agente](./media/backup-mabs-install-azure-stack/mabs-install-wizard-mars-agent-16.png)

    El agente de Microsoft Azure Recovery Services, también llamado agente de Azure Backup, configura Azure Backup Server en el almacén de Recovery Services. Una vez configurado, Azure Backup Server siempre creará copias de seguridad de los datos en el mismo almacén de Recovery Services.

11. Cuando el agente de Microsoft Azure Recovery Services finalice la instalación, seleccione **Siguiente** para iniciar la fase siguiente: registrar Azure Backup Server con el almacén de Recovery Services.

    ![Instalación del agente completada correctamente](./media/backup-mabs-install-azure-stack/mabs-install-wizard-complete-16.png)

    El instalador inicia el **Asistente para registrar servidor**.

12. Cambie a su suscripción de Azure y su almacén de Recovery Services. En el menú **Preparar infraestructura**, seleccione **Descargar** para descargar las credenciales del almacén. Si el botón **Descargar** del paso 2 no está activo, seleccione **Already downloaded or using the latest Azure Backup Server installation** (Ya descargado o usando la instalación de Azure Backup Server más reciente) para activarlo. Las credenciales del almacén se descargan en la ubicación donde almacena las descargas. Tenga en cuenta esta ubicación porque la necesitará para el próximo paso.

    ![Descarga de las credenciales de almacén](./media/backup-mabs-install-azure-stack/download-mars-credentials-17.png)

13. En el menú **Identificación del almacén**, seleccione **Examinar** para buscar las credenciales del almacén de Recovery Services.

    ![Menú Identificación del almacén](./media/backup-mabs-install-azure-stack/mabs-install-wizard-vault-id-18.png)

    En el cuadro de diálogo **Seleccionar credenciales de almacén**, vaya a la ubicación de descarga, seleccione las credenciales del almacén y, después, **Abrir**.

    La ruta de acceso a las credenciales aparece en el menú de identificación del almacén. Seleccione **Siguiente** para avanzar a la **Configuración de cifrado**.

14. En el cuadro de diálogo **Configuración de cifrado**, indique una frase de contraseña para el cifrado de copia de seguridad y una ubicación para almacenar la frase de contraseña y seleccione **Siguiente**.

    ![Configuración de cifrado](./media/backup-mabs-install-azure-stack/mabs-install-wizard-encryption-19.png)

    Puede proporcionar su propia frase de contraseña o usar el generador de frases de contraseña para crear una. La frase de contraseña es suya, y Microsoft no la guarda ni administra. Para prepararse para un desastre, guarde la frase de contraseña en una ubicación accesible.

    Cuando seleccione **Siguiente**, Azure Backup Server se registrará con el almacén de Recovery Services. El instalador continúa la instalación de SQL Server y Azure Backup Server.

    ![El programa de instalación instala SQL y Azure Backup Server](./media/backup-mabs-install-azure-stack/mabs-install-wizard-sql-still-installing-20.png)

15. Cuando el instalador se complete, el **estado** mostrará que todo el software se instaló correctamente.

    ![El software se instaló correctamente](./media/backup-mabs-install-azure-stack/mabs-install-wizard-done-22.png)

    Cuando se complete la instalación, la consola de Azure Backup Server y los iconos de PowerShell de Azure Backup Server se crean en el escritorio del servidor.

## <a name="add-backup-storage"></a>Incorporación de almacenamiento de copia de seguridad

La primera copia de seguridad se mantiene en el almacenamiento conectado a la máquina de Azure Backup Server. Para obtener más información acerca de cómo agregar discos, consulte [Add Modern Backup storage](/system-center/dpm/add-storage) (Agregar almacenamiento de copia de seguridad moderno).

> [!NOTE]
> Debe agregar el almacenamiento de copia de seguridad incluso si tiene pensado enviar los datos a Azure. En la arquitectura de Azure Backup Server, el almacén de Recovery Services contiene la *segunda* copia de los datos mientras que el almacenamiento local contiene la primera (y obligatoria) copia de seguridad.
>
>

## <a name="network-connectivity"></a>Conectividad de red

Azure Backup Server requiere conectividad al servicio Azure Backup para que el producto funcione correctamente. Para validar si la máquina tiene conectividad a Azure, use el cmdlet ```Get-DPMCloudConnection``` en la consola de PowerShell de Azure Backup Server. Si la salida del cmdlet es TRUE, entonces existe conectividad, de lo contrario, no hay conectividad.

Además, la suscripción de Azure debe encontrarse en un estado correcto. Para averiguar el estado de la suscripción y administrarla, inicie sesión en el [portal de suscripción](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

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

Si hay un firewall o un proxy que impide el acceso a Azure, agregue las direcciones de dominio siguientes a la lista de perfiles permitidos de firewall y proxy:

- `http://www.msftncsi.com/ncsi.txt`
- \*.Microsoft.com
- \*.WindowsAzure.com
- \*.microsoftonline.com
- \*.windows.net

Una vez restaurada la conectividad a Azure en Azure Backup Server, el estado de la suscripción de Azure determina las operaciones que se pueden realizar. Una vez que el servidor esté **conectado**, use la tabla que aparece en [Network connectivity](backup-mabs-install-azure-stack.md#network-connectivity) (Conectividad de red) para ver las operaciones disponibles.

### <a name="handling-subscription-states"></a>Control de los estados de la suscripción

Es posible cambiar una suscripción de Azure desde un estado *Expirado* o *Desaprovisionado* hasta un estado *Activo*. Mientras el estado de la suscripción no es *Activo*:

- Mientras el estado de la suscripción es *Desaprovisionado*, pierde funcionalidad. Al restaurar el estado de la suscripción a *Activo* revive la funcionalidad de copia de seguridad/restauración. Si los datos de copia de seguridad en el disco local se mantuvieron durante un período de retención lo suficientemente amplio, se pueden recuperar. No obstante, los datos de copia de seguridad de Azure se pierden irremediablemente una vez que la suscripción pasa al estado *Desaprovisionado*.
- Mientras el estado de la suscripción es *Desaprovisionado*, pierde funcionalidad. Las copias de seguridad programadas no se ejecutan mientras una suscripción tiene el estado *Expirado*.

## <a name="troubleshooting"></a>Solución de problemas

Si Microsoft Azure Backup Server presenta un error durante la fase de configuración (o copia de seguridad o restauración), consulte el [documento de códigos de error](https://support.microsoft.com/kb/3041338).
También puede consultar [Azure Backup - Preguntas más frecuentes](backup-azure-backup-faq.md)

## <a name="next-steps"></a>Pasos siguientes

El artículo sobre la [preparación del entorno para DPM](/system-center/dpm/prepare-environment-for-dpm) contiene información sobre las configuraciones admitidas de Azure Backup Server.

Puede usar los artículos siguientes para mejorar la comprensión sobre la protección de cargas de trabajo mediante Microsoft Azure Backup Server.

- [Copia de seguridad de SQL Server](./backup-mabs-sql-azure-stack.md)
- [Copia de seguridad de una granja de SharePoint](./backup-mabs-sharepoint-azure-stack.md)
- [Copia de seguridad de otro servidor](backup-azure-alternate-dpm-server.md)
