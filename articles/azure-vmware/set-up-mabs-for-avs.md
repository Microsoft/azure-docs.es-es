---
title: Configuración de Azure Backup Server para Azure VMware Solution
description: Configure el entorno de Azure VMware Solution para realizar copias de seguridad de máquinas virtuales mediante Azure Backup Server.
ms.topic: how-to
ms.date: 06/09/2020
ms.openlocfilehash: 0dd2b16254e697a08d0ff542a5ddcb3fc7e4103d
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750623"
---
# <a name="set-up-azure-backup-server-for-azure-vmware-solution"></a>Configuración de Azure Backup Server para Azure VMware Solution

Azure Backup Server es un sistema de copia de seguridad y recuperación empresarial eficaz que contribuye a su estrategia de continuidad empresarial y recuperación ante desastres (BCDR). En la versión preliminar de Azure VMware Solution, solo puede configurar la copia de seguridad de nivel de máquina virtual (VM) mediante Azure Backup Server. 

Azure Backup Server puede almacenar datos de copia de seguridad en:

- **Disco**: para el almacenamiento a corto plazo, Azure Backup Server realiza copias de seguridad de los datos en grupos de discos.
- **Azure**: para el almacenamiento a corto y a largo plazo fuera de las instalaciones, pueden realizarse copias de seguridad de los datos de Azure Backup Server que estén almacenados en grupos de discos en la nube de Microsoft Azure mediante Azure Backup.

Cuando se producen interrupciones y los datos de origen no están disponibles, puede usar Azure Backup Server para restaurar fácilmente los datos en el origen o en una ubicación alternativa. De este modo, si los datos originales no están disponibles debido a problemas planeados o inesperados, puede restaurar fácilmente los datos en una ubicación alternativa.

En este artículo, le ayudaremos a preparar el entorno de Azure VMware Solution para realizar copias de seguridad de máquinas virtuales mediante Azure Backup Server. Le guiaremos a través de los pasos para: 

> [!div class="checklist"]
> * Determinar el tipo de disco de máquina virtual recomendado y el tamaño que se usará.
> * Crear un almacén de Recovery Services que almacene los puntos de recuperación.
> * Establecer la replicación de almacenamiento para un almacén de Recovery Services.
> * Agregar almacenamiento a Azure Backup Server.

## <a name="supported-vmware-features"></a>Características admitidas de VMware

- **Copia de seguridad sin agente:** Azure Backup Server no requiere que se instale un agente en el servidor vCenter o ESXi para crear una copia de seguridad de la máquina virtual. En su lugar, solo tiene que proporcionar la dirección IP o el nombre de dominio completo (FQDN) y las credenciales de inicio de sesión que se usan para autenticar el servidor VMware con Azure Backup Server.
- **Copia de seguridad integrada en la nube:** Azure Backup Server protege las cargas de trabajo en el disco y en la nube. El flujo de trabajo de copia de seguridad y recuperación de Azure Backup Server ayuda a administrar la retención a largo plazo y la copia de seguridad fuera del sitio.
- **Detección y protección de máquinas virtuales administradas por vCenter:** Azure Backup Server detecta y protege las máquinas virtuales implementadas en un servidor vCenter o ESXi. Azure Backup Server también detecta las máquinas virtuales administradas por vCenter para que se puedan proteger las implementaciones de gran tamaño.
- **Protección automática en el nivel de carpeta:** vCenter permite organizar las máquinas virtuales en carpetas de máquinas virtuales. Azure Backup Server detecta esas carpetas, y el usuario puede usarlo para proteger las máquinas virtuales en el nivel de carpeta, lo que incluye todas las subcarpetas. Al proteger las carpetas, Azure Backup Server no solo protege las máquinas virtuales que se encuentran en esa carpeta, sino también las máquinas virtuales que se agreguen después. Azure Backup Server detecta a diario nuevas máquinas virtuales y las protege de manera automática. Cuando organice las máquinas virtuales en carpetas recursivas, Azure Backup Server detectará y protegerá automáticamente las nuevas máquinas virtuales que se implementen en estas carpetas.
- **Azure Backup Server sigue protegiendo las máquinas virtuales de vMotion dentro del clúster:** Cuando las máquinas virtuales se colocan en vMotion para el equilibrio de carga en el clúster, Azure Backup Server detecta y continúa automáticamente la protección de las máquinas virtual.
- **Recuperación con mayor rapidez de los archivos necesarios:** Azure Backup Server puede recuperar archivos o carpetas de una máquina virtual Windows sin necesidad de recuperar toda la máquina virtual.

## <a name="limitations"></a>Limitaciones

- Hay que instalar el paquete acumulativo de actualizaciones 1 para Azure Backup Server v3.
- No se pueden realizar copias de seguridad de instantáneas de usuario antes de la primera copia de seguridad de Azure Backup Server. Una vez que Azure Backup Server completa la primera copia de seguridad, puede crear copias de seguridad de las instantáneas de usuario.
- Azure Backup Server no puede proteger las máquinas virtuales VMware con discos de acceso directo y asignaciones de dispositivos físicos sin formato (pRDM).
- Azure Backup Server no puede detectar ni proteger VMware vApps.

Para configurar Azure Backup Server para Azure VMware Solution, debe completar los pasos siguientes:

- Configure los requisitos previos y el entorno.
- Cree un almacén de Recovery Services.
- Descargue e instale Azure Backup Server.
- Agregue almacenamiento a Azure Backup Server.

### <a name="deployment-architecture"></a>Arquitectura de implementación

Azure Backup Server se implementa como una máquina virtual de infraestructura como servicio (IaaS) de Azure para proteger las máquinas virtuales de Azure VMware Solution.

:::image type="content" source="media/avs-backup/deploy-mabs-avs-diagram.png" alt-text="Azure Backup Server se implementa como una máquina virtual de infraestructura como servicio (IaaS) de Azure para proteger las máquinas virtuales de Azure VMware Solution." border="false":::

## <a name="prerequisites-for-the-azure-backup-server-environment"></a>Requisitos previos para el entorno de Azure Backup Server

Tenga en cuenta las recomendaciones de esta sección cuando instale Azure Backup Server en el entorno de Azure.

### <a name="azure-virtual-network"></a>Azure Virtual Network

Asegúrese de [configurar la red para la nube privada de VMware en Azure](tutorial-configure-networking.md).

### <a name="determine-the-size-of-the-virtual-machine"></a>Determinación del tamaño de la máquina virtual

Debe crear una máquina virtual Windows en la red virtual que creó en el paso anterior. Al elegir un servidor para ejecutar Azure Backup Server, se recomienda comenzar con una imagen de la galería de Windows Server 2019 Datacenter. El tutorial [Creación de la primera máquina virtual de Windows en Azure Portal](../virtual-machines/windows/quick-create-portal.md) le permite empezar a trabajar con la máquina virtual recomendada en Azure, aunque nunca haya usado Azure.

En la tabla siguiente se resume el número máximo de cargas de trabajo protegidas para cada tamaño de máquina virtual de Azure Backup Server. La información se basa en pruebas de escala y rendimiento internas con valores canónicos correspondientes al tamaño y la renovación de la carga de trabajo. El tamaño real de la carga de trabajo puede ser mayor, pero debe adaptarse a los discos conectados a la máquina virtual de Azure Backup Server.

| Número máximo de cargas de trabajo protegidas | Tamaño promedio de la carga de trabajo | Promedio de renovación de la carga de trabajo (diario) | Número máximo de IOPS de almacenamiento | Tipo y tamaño de disco recomendados      | Tamaño de máquina virtual recomendado |
|-------------------------|-----------------------|--------------------------------|------------------|-----------------------------------|---------------------|
| 20                      | 100 GB                | Renovación neta del 5 %                   | 2\.000             | HDD estándar (tamaño de 8 TB o más por disco)  | A4V2       |
| 40                      | 150 GB                | Renovación neta del 10 %                  | 4500             | SSD Premium (tamaño de 1 TB o más por disco) | DS3_V2     |
| 60                      | 200 GB                | Renovación neta del 10 %                  | 10 500            | SSD Premium (tamaño de 8 TB o más por disco) | DS3_V2     |

* Para obtener el número necesario de IOPS, use discos del tamaño mínimo recomendado o superior. Los discos de menor tamaño ofrecen menos IOPS.

> [!NOTE]
> Azure Backup Server está diseñado para ejecutarse en un servidor dedicado de objetivo único. No se puede instalar Azure Backup Server en un equipo que:
> * Se ejecute como controlador de dominio.
> * Tenga instalado el rol Servidor de aplicaciones.
> * Sea un servidor de administración de System Center Operations Manager.
> * Ejecute Exchange Server.
> * Sea nodo de un clúster.

### <a name="disks-and-storage"></a>Discos y almacenamiento

Azure Backup Server requiere discos para su instalación, lo que incluye los archivos del sistema, los de instalación, el software necesario, los archivos de base de datos y los discos dedicados para el bloque de almacenamiento.

| Requisito                      | Tamaño recomendado  |
|----------------------------------|-------------------------|
| Instalación de Azure Backup Server                | Ubicación de la instalación: 3 GB<br />Unidad de archivos de base de datos: 900 MB<br />Unidad del sistema: 1 GB para la instalación de SQL Server<br /><br />También necesitará espacio para que Azure Backup Server copie el catálogo de archivos en una ubicación de instalación temporal al realizar el archivado.      |
| Disco para el bloque de almacenamiento<br />(Usa volúmenes básicos; no puede estar en un disco dinámico) | Tamaño entre dos y tres veces superior al de los datos protegidos.<br />Para más información sobre el cálculo de almacenamiento, consulte el artículo sobre [Azure Site Recovery Capacity Planner para DPM](https://www.microsoft.com/download/details.aspx?id=54301).   |

Para más información sobre cómo conectar un nuevo disco de datos administrado a una máquina virtual de Azure existente, consulte [Conexión de un disco de datos administrado a una máquina virtual Windows en Azure Portal](../virtual-machines/windows/attach-managed-disk-portal.md).

> [!NOTE]
> Una sola instancia de Azure Backup Server tiene un límite flexible de 120 TB para el bloque de almacenamiento.

### <a name="store-backup-data-on-local-disk-and-in-azure"></a>Almacenamiento de datos de copia de seguridad en un disco local y en Azure

El almacenamiento de datos de copia de seguridad de Azure reduce la infraestructura de copia de seguridad en la máquina virtual de Azure Backup Server. En el caso de una recuperación operativa (copia de seguridad), Azure Backup Server almacena los datos de copia de seguridad en discos de Azure conectados a la máquina virtual. Una vez que los discos y el espacio de almacenamiento se conectan a la máquina virtual, Azure Backup Server administra el almacenamiento automáticamente. La cantidad de almacenamiento de datos de copia de seguridad depende del número y tamaño de los discos conectados a cada VM de Azure. Para cada tamaño de la VM de Azure hay un número máximo de discos que se pueden conectar a ella. Por ejemplo, a A2 le corresponden 4 discos, a A3, 8 discos y a A4, 16 discos. De nuevo, el tamaño y número de discos determina la capacidad total del bloque de almacenamiento de copia de seguridad.

> [!IMPORTANT]
> *No* debe conservar datos de recuperación operativa en discos conectados directamente a Azure Backup Server durante más de cinco días. Si los datos tienen más de cinco días de antigüedad, almacénelos en un almacén de Recovery Services.

Para almacenar datos de copia de seguridad en Azure, cree o use un almacén de Recovery Services. Cuando se prepare para hacer una copia de seguridad de la carga de trabajo de Azure Backup Server, [configure el almacén de Recovery Services](#create-a-recovery-services-vault). Una vez configurado, cada vez que se ejecuta un trabajo de copia de seguridad en línea, se crea un punto de recuperación en el almacén. Cada almacén de Recovery Services contiene hasta 9999 puntos de recuperación. En función del número de puntos de recuperación creados y del tiempo que se conserven, se pueden conservar los datos de copias de seguridad durante muchos años. Por ejemplo, podría crear puntos de recuperación mensuales y conservarlos durante cinco años.

> [!IMPORTANT]
> Tanto si envía datos de copia de seguridad a Azure como si los mantiene localmente, debe registrar Azure Backup Server en un almacén de Recovery Services.

### <a name="scale-deployment"></a>Escalado de la implementación

Si quiere escalar su implementación, tiene las siguientes opciones:

- **Escalado vertical**: aumente el tamaño de la máquina virtual de Azure Backup Server de la serie A a la serie DS3 y aumente el almacenamiento local.
- **Descarga de datos**: envíe los datos antiguos a Azure y conserve solo los datos más recientes en el almacenamiento conectado a la máquina de Azure Backup Server.
- **Escalado horizontal**: agregue más máquinas de Azure Backup Server para proteger las cargas de trabajo.

### <a name="net-framework"></a>.NET Framework

La máquina virtual necesita tener instalado .NET Framework 3.5 SP1 o una versión posterior.

### <a name="join-a-domain"></a>Unión a un dominio

La máquina virtual de Azure Backup Server debe estar unida a un dominio, y un usuario de dominio con privilegios de administrador en la máquina virtual debe instalar Azure Backup Server.

Aunque no se admite en la versión preliminar, Azure Backup Server implementado en una máquina virtual de Azure puede realizar copias de seguridad de las cargas de trabajo en las máquinas virtuales de Azure VMware Solution. Las cargas de trabajo deben estar en el mismo dominio para habilitar la operación de copia de seguridad.

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Un almacén de Recovery Services es una entidad de almacenamiento que almacena los puntos de recuperación creados a lo largo del tiempo. También contiene directivas de copia de seguridad asociadas a elementos protegidos.

1. Inicie sesión en su suscripción en [Azure Portal](https://portal.azure.com/).

1. En el menú izquierdo, seleccione **Todos los servicios**.

   ![En el menú izquierdo, seleccione Todos los servicios.](../backup/media/backup-create-rs-vault/click-all-services.png)

1. En el cuadro de diálogo **Todos los servicios**, escriba **Recovery Services** y, en la lista, seleccione **Almacenes de Recovery Services**.

   ![Escribir y elegir almacenes de Recovery Services.](../backup/media/backup-create-rs-vault/all-services.png)

   Aparece la lista de almacenes de Recovery Services de la suscripción.

1. En el panel **Almacenes de Recovery Services**, seleccione **Agregar**.

   ![Agregar un almacén de Recovery Services.](../backup/media/backup-create-rs-vault/add-button-create-vault.png)

   Se abre el cuadro de diálogo **Almacén de Recovery Services**.

1. Especifique los valores de **Nombre**, **Suscripción**, **Grupo de recursos** y **Ubicación**.

   ![Configurar el almacén de Recovery Services.](../backup/media/backup-create-rs-vault/create-new-vault-dialog.png)

   - **Name**: escriba un nombre descriptivo que identifique el almacén. El nombre debe ser único para la suscripción de Azure. Escriba un nombre que tenga un mínimo de 2 caracteres y un máximo de 50. El nombre debe comenzar por una letra y consta solo de letras, números y guiones.
   - **Suscripción**: elija la suscripción que desee usar. Si es miembro de una sola suscripción, verá solo ese nombre. Si no está seguro de qué suscripción va a utilizar, use la predeterminada (sugerida). Solo hay varias opciones si la cuenta profesional o educativa está asociada a más de una suscripción de Azure.
   - **Grupo de recursos**: Use un grupo de recursos existente o cree uno. Para ver la lista de grupos de recursos disponibles en una suscripción, seleccione **Usar existente** y, después, seleccione un recurso en la lista desplegable. Para crear un grupo de recursos, seleccione **Crear nuevo** y escriba el nombre.
   - **Ubicación**: seleccione la región geográfica del almacén. Para crear un almacén con el fin de proteger las máquinas virtuales de Azure VMware Solution, dicho almacén *debe* estar en la misma región que la nube privada de Azure VMware Solution.

1. Cuando esté listo para crear el almacén de Recovery Services, seleccione **Crear**.

   ![Crear el almacén de Recovery Services.](../backup/media/backup-create-rs-vault/click-create-button.png)

   La creación del almacén de Recovery Services puede tardar unos minutos. Supervise las notificaciones de estado en el área **Notificaciones** de la parte superior derecha del portal. Tras crear el almacén, se ve en la lista de almacenes de Recovery Services. Si no lo ve, haga clic en **Actualizar**.

   ![Actualizar la lista de almacenes de copia de seguridad.](../backup/media/backup-create-rs-vault/refresh-button.png)

## <a name="set-storage-replication"></a>Configuración de la replicación del almacenamiento

La opción de replicación de almacenamiento permite elegir entre almacenamiento con redundancia geográfica (el valor predeterminado) y almacenamiento con redundancia local. El almacenamiento con redundancia geográfica copia los datos de la cuenta de almacenamiento en una región secundaria, lo que hace que los datos sean duraderos. El almacenamiento con redundancia local es una opción más económica que no es tan duradera. Para más información sobre las opciones de almacenamiento con redundancia geográfica y redundancia local, consulte [Redundancia de Azure Storage](../storage/common/storage-redundancy.md).

> [!IMPORTANT]
> El cambio del valor de **Tipo de replicación de almacenamiento** (con redundancia local o con redundancia geográfica) para un almacén de Recovery Services se debe realizar antes de configurar las copias de seguridad en el almacén. Tras la configuración de las copias de seguridad, la opción para modificarla está deshabilitada y el tipo de replicación del almacenamiento no se puede cambiar.

1. En **Almacenes de Recovery Services**, seleccione el nuevo almacén. 

1. En **Configuración**, seleccione **Propiedades**. En **Configuración de copia de seguridad**, seleccione **Actualizar**.

1. Seleccione el tipo de replicación almacenamiento y seleccione **Guardar**.

   ![Establecer la configuración de almacenamiento del almacén nuevo.](../backup/media/backup-try-azure-backup-in-10-mins/recovery-services-vault-backup-configuration.png)

## <a name="download-and-install-the-software-package"></a>Descarga e instalación del paquete de software

Siga los pasos de esta sección para descargar, extraer e instalar el paquete de software.

### <a name="download-the-software-package"></a>Descarga del paquete de software

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Si ya tiene abierto un almacén de Recovery Services, vaya al siguiente paso. Si no tiene abierto un almacén de Recovery Services, pero está en Azure Portal, en el menú principal, seleccione **Examinar**.

   1. En la lista de recursos, escriba **Recovery Services**.

   1. Cuando comience a escribir, la lista se filtrará en función de la entrada. Cuando vea la opción **Almacenes de Recovery Services**, haga clic en ella.

   ![Creación del almacén de Recovery Services (paso 1)](../backup/media/backup-azure-microsoft-azure-backup/open-recovery-services-vault.png)

1. En la lista de almacenes de Recovery Services, seleccione un almacén.

   Se abre el panel del almacén seleccionado.

   ![Se abre el panel del almacén seleccionado.](../backup/media/backup-azure-microsoft-azure-backup/vault-dashboard.png)

   De forma predeterminada, la opción **Configuración** abre el almacén. Si está cerrado, seleccione **Configuración** para abrirlo.

   ![De forma predeterminada, la opción Configuración abre el almacén. Si está cerrado, seleccione Configuración para abrirlo.](../backup/media/backup-azure-microsoft-azure-backup/vault-setting.png)

1. Seleccione **Copia de seguridad** para abrir el **Asistente para introducción**.

   ![Seleccione Copia de seguridad para abrir el Asistente para introducción.](../backup/media/backup-azure-microsoft-azure-backup/getting-started-backup.png)

1. En la ventana que se abre, haga lo siguiente:

   1. En el menú **¿Dónde se ejecuta su carga de trabajo?** , seleccione **Local**.

      :::image type="content" source="media/avs-backup/deploy-mabs-on-premises-workload.png" alt-text="¿Dónde se ejecuta su carga de trabajo?":::

   1. En el menú **¿De qué desea hacer una copia de seguridad?** , seleccione las cargas de trabajo que quiere proteger mediante Azure Backup Server.

   1. Seleccione **Preparar infraestructura** para descargar e instalar Azure Backup Server y las credenciales del almacén.

      :::image type="content" source="media/avs-backup/deploy-mabs-prepare-infrastructure.png" alt-text="Preparación de la infraestructura":::

1. En la ventana **Preparar infraestructura** que se abre, haga lo siguiente:

   1. Seleccione el vínculo **Descargar** para instalar Azure Backup Server.

   1. Descargue las credenciales del almacén activando la casilla **Ya se ha descargado o se usa la última instalación de Azure Backup Server** y seleccione **Descargar**. Utilice las credenciales del almacén durante el registro de Azure Backup Server en el almacén de Recovery Services. Los vínculos redirigen al Centro de descarga, donde se puede descargar el paquete de software.

   :::image type="content" source="media/avs-backup/deploy-mabs-prepare-infrastructure2.png" alt-text="Preparación de la infraestructura: Azure Backup Server":::

1. En la página de descarga, seleccione todos los archivos y elija **Siguiente**.

   > [!NOTE]
   > Debe descargar todos los archivos en la misma carpeta. Puesto que el tamaño de descarga de todos los archivos juntos es superior a 3 GB, la descarga podría tardar hasta 60 minutos en completarse. 

   ![En la página de descarga, seleccione todos los archivos y elija Siguiente.](../backup/media/backup-azure-microsoft-azure-backup/downloadcenter.png)

### <a name="extract-the-software-package"></a>Extracción del paquete de software

Si descargó el paquete de software en otro servidor, copie los archivos en la máquina virtual que creó para implementar Azure Backup Server.

> [!WARNING]
> Se requieren al menos 4 GB de espacio libre para extraer los archivos de instalación.

1. Cuando haya descargado todos los archivos, haga doble clic en **MicrosoftAzureBackupInstaller.exe** para abrir el Asistente para instalación de **Microsoft Azure Backup** y, luego, elija **Siguiente**.

1. Seleccione la ubicación en la que se van a extraer los archivos y elija **Siguiente**.

1. Seleccione **Extraer** para iniciar el proceso de extracción.

   ![Seleccione Extraer para iniciar el proceso de extracción.](../backup/media/backup-azure-microsoft-azure-backup/extract/03.png)

1. Al concluir la extracción, seleccione la opción para **ejecutar setup.exe** y, luego, elija **Finalizar**.

> [!TIP]
> También puede buscar el archivo setup.exe en la carpeta donde extrajo el paquete de software.

### <a name="install-the-software-package"></a>Instalación del paquete de software

1. En la ventana de instalación, en **Instalar**, seleccione **Microsoft Azure Backup** para abrir el Asistente para instalación.

   ![En la ventana de instalación, en Instalar, seleccione Microsoft Azure Backup para abrir el Asistente para instalación.](../backup/media/backup-azure-microsoft-azure-backup/launch-screen2.png)

1. En la **pantalla de bienvenida**, seleccione **Siguiente** para ir a la página **Comprobaciones de requisitos previos**.

1. Seleccione **Comprobar de nuevo** para determinar si se cumplen los requisitos previos de hardware y software para Azure Backup Server. Si se cumplen, seleccione **Siguiente**.

   ![ Seleccione Comprobar de nuevo para determinar si se cumplen los requisitos previos de hardware y software para Azure Backup Server. Si se cumplen, seleccione Siguiente.](../backup/media/backup-azure-microsoft-azure-backup/prereq/prereq-screen2.png)

1. En el paquete de instalación de Azure Backup Server se incluyen los archivos binarios de SQL Server que se necesitan. Al iniciar una nueva instalación de Azure Backup Server, seleccione la opción **Instalar una nueva instancia de SQL Server con esta instalación**. Después, seleccione **Comprobar e instalar**.

   ![En el paquete de instalación de Azure Backup Server se incluyen los archivos binarios de SQL Server que se necesitan.](../backup/media/backup-azure-microsoft-azure-backup/sql/01.png)

   > [!NOTE]
   > Si quiere usar su propia instancia de SQL Server, las versiones de SQL Server admitidas son SQL Server 2014 SP1 o posterior, 2016 y 2017. Todas las versiones de SQL Server deben ser Estándar o Enterprise de 64 bits. Azure Backup Server no funciona con una instancia remota de SQL Server. La instancia que Azure Backup Server usa debe ser local. Si utiliza una instancia de SQL Server existente para Azure Backup Server, el programa de instalación solo admite el uso de *instancias con nombre* de SQL Server.

   Si se produce un error que incluye una recomendación para reiniciar el equipo, hágalo y seleccione **Comprobar de nuevo**. Si hay algún problema de configuración de SQL Server, reconfigure SQL Server según las directrices de SQL Server. A continuación, intente instalar o actualizar Azure Backup Server mediante la instancia existente de SQL Server.

   **Configuración manual**

   Cuando use su propia instancia de SQL Server, asegúrese de agregar builtin\Administrators al rol Sysadmin de la base de datos maestra.

   **Configuración de SSRS con SQL Server 2017**

   Cuando use su propia instancia de SQL Server 2017, debe configurar SQL Server 2017 Reporting Services (SSRS) manualmente. Tras la configuración de SSRS, asegúrese de que la propiedad **IsInitialized** de SSRS esté establecida en **True**. Cuando esta propiedad se establece en **True**, Azure Backup Server presupone que SSRS ya está configurado y omite la configuración de SSRS.

   Para comprobar el estado de configuración de SSRS, ejecute el comando siguiente:

   ```powershell
   $configset =Get-WmiObject –namespace 
   "root\Microsoft\SqlServer\ReportServer\RS_SSRS\v14\Admin" -class 
   MSReportServer_ConfigurationSetting -ComputerName localhost

   $configset.IsInitialized
   ```

   Se usarán los siguientes valores para la configuración de SSRS:

   * **Cuenta de servicio**: **Usar cuenta integrada** debe estar en **Servicio de red**.
   * **Dirección URL del servicio web**: **Directorio virtual** debe estar en **ReportServer_\<SQLInstanceName>** .
   * **Base de datos**: **DatabaseName** debe estar en **ReportServer$\<SQLInstanceName>** .
   * **Dirección URL del Portal web**: **Directorio virtual** debe estar en **Reports_\<SQLInstanceName>** .

   [Obtenga más información](/sql/reporting-services/report-server/configure-and-administer-a-report-server-ssrs-native-mode?view=sql-server-2017) acerca de la configuración de SSRS.

   > [!NOTE]
   > [Términos de Microsoft Online Services](https://www.microsoft.com/licensing/product-licensing/products) (OST) controla las licencias de SQL Server que se usan como base de datos para Azure Backup Server. Según OST, SQL Server en conjunto con Azure Backup Server solo se puede usar como base de datos para Azure Backup Server.

1. Cuando la instalación se haya completado, seleccione **Siguiente**.

1. Especifique una ubicación para la instalación de los archivos de Microsoft Azure Backup Server y seleccione **Siguiente**.

   > [!NOTE]
   > La ubicación temporal es necesaria para realizar copias de seguridad en Azure. Asegúrese de que la ubicación temporal sea al menos el 5% de los datos cuya copia de seguridad se planea hacer en la nube. Para la protección de disco, hay que configurar discos independientes una vez completada la instalación. Para más información sobre los bloques de almacenamiento, consulte [Configuración de bloques de almacenamiento y almacenamiento en disco](/previous-versions/system-center/system-center-2012-r2/hh758075(v=sc.12)).

   ![Especifique una ubicación para la instalación de los archivos de Microsoft Azure Backup Server y seleccione Siguiente.](../backup/media/backup-azure-microsoft-azure-backup/space-screen.png)

1. Proporcione una contraseña segura para las cuentas de usuario locales con permisos restringidos y seleccione **Siguiente**.

   ![Proporcione una contraseña segura para las cuentas de usuario locales con permisos restringidos y seleccione Siguiente.](../backup/media/backup-azure-microsoft-azure-backup/security-screen.png)

1. Seleccione si quiere usar Microsoft Update para buscar actualizaciones y elija **Siguiente**.

   > [!NOTE]
   > Se recomienda que Windows Update se redirija a Microsoft Update, que ofrece actualizaciones importantes y de seguridad para Windows y otros productos como Azure Backup Server.

   ![Seleccione si quiere usar Microsoft Update para buscar actualizaciones y elija Siguiente.](../backup/media/backup-azure-microsoft-azure-backup/update-opt-screen2.png)

1. Revise la página **Resumen de la configuración** y seleccione **Instalar**.

   La instalación se realiza en fases. En la primera fase se instala el servicio Agente de Microsoft Azure Recovery Services, mientras que en la segunda se comprueba la conectividad a Internet. Si se dispone de conectividad a Internet, puede continuar con la instalación. En caso contrario, debe proporcionar detalles del proxy para conectarse a Internet. La fase final comprueba el software necesario. Si algo no se ha instalado, el software que falte se instalará junto con el servicio Agente de Microsoft Azure Recovery Services.

1. Seleccione **Examinar** para buscar las credenciales del almacén a fin de registrar la máquina en el almacén de Recovery Services y, luego, elija **Siguiente**.

1. Elija una frase de contraseña para cifrar y descifrar los datos enviados entre Azure y sus instalaciones.

   > [!TIP]
   > Puede generar una frase de contraseña automáticamente o proporcionar la suya propia, con un mínimo de 16 caracteres.

1. Escriba la ubicación para guardar la frase de contraseña y, luego, seleccione **Siguiente** para registrar el servidor.

   > [!IMPORTANT]
   > Guarde la frase de contraseña en una ubicación segura que no sea el servidor local. Se recomienda encarecidamente usar Azure Key Vault para almacenar la frase de contraseña.

   Una vez finalizada la instalación del servicio Agente de Microsoft Azure Recovery Services, el paso de instalación sigue con la instalación y configuración de SQL Server, y los componentes de Azure Backup Server.

   ![Una vez finalizada la instalación del servicio Agente de Microsoft Azure Recovery Services, el paso de instalación sigue con la instalación y configuración de SQL Server, y los componentes de Azure Backup Server.](../backup/media/backup-azure-microsoft-azure-backup/final-install/venus-installation-screen.png)

1. Cuando finalice la instalación, seleccione **Cerrar**.

### <a name="install-update-rollup-1"></a>Instalación del paquete acumulativo de actualizaciones 1

La instalación del paquete acumulativo de actualizaciones 1 para Azure Backup Server v3 es obligatoria para proteger las cargas de trabajo. Para ver la lista de correcciones de errores y las instrucciones de instalación para el paquete acumulativo de actualizaciones 1 de Azure Backup Server v3, consulte el artículo [4534062](https://support.microsoft.com/en-us/help/4534062/) de Knowledge Base.

## <a name="add-storage-to-azure-backup-server"></a>Adición de almacenamiento a Azure Backup Server

Azure Backup Server v3 admite Modern Backup Storage, que ofrece:

-  Ahorro de almacenamiento del 50 %.
-  Copias de seguridad tres veces más rápidas.
-  Almacenamiento más eficiente.
-  Almacenamiento con reconocimiento de la carga de trabajo.

### <a name="volumes-in-azure-backup-server"></a>Volúmenes en Azure Backup Server

Agregue los discos de datos con la capacidad de almacenamiento necesaria a la máquina virtual de Azure Backup Server si aún no se han agregado.

Azure Backup Server v3 solo acepta volúmenes de almacenamiento. Al agregar un volumen, Azure Backup Server le asigna formato en Sistema de archivos resistente (ReFS), que requiere Modern Backup Storage.

### <a name="add-volumes-to-azure-backup-server-disk-storage"></a>Adición de volúmenes al almacenamiento de disco de Azure Backup Server

1. En el panel **Administración**, vuelva a examinar el almacenamiento y seleccione **Agregar**. 

1. Entre los volúmenes disponibles, seleccione los que quiere agregar al bloque de almacenamiento. 

1. Después de agregarlos, asígneles un nombre descriptivo que le ayude a administrarlos. 

1. Seleccione **Aceptar** para dar formato a estos volúmenes en ReFS a fin de que Azure Backup Server pueda usar las ventajas de Modern Backup Storage.

![Adición de volúmenes disponibles](../backup/media/backup-mabs-add-storage/mabs-add-storage-7.png)

## <a name="next-steps"></a>Pasos siguientes

Continúe con el siguiente tutorial para aprender a configurar la copia de seguridad de máquinas virtuales de VMware que se ejecutan en Azure VMware Solution mediante Azure Backup Server.

> [!div class="nextstepaction"]
> [Configuración de la copia de seguridad de máquinas virtuales de Azure VMware Solution](backup-avs-vms-with-mabs.md)
