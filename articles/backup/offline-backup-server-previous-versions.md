---
title: 'Copia de seguridad sin conexión para Data Protection Manager (DPM) y Microsoft Azure Backup Server (MABS): versiones anteriores'
description: Con Azure Backup, puede enviar datos fuera de la red mediante el servicio Azure Import/Export. En este artículo se explica el flujo de trabajo de la copia de seguridad sin conexión para las versiones anteriores de DPM y Azure Backup Server.
ms.topic: conceptual
ms.date: 06/08/2020
ms.openlocfilehash: b747fd3c682dc1caf7312ba7279470a1e6b38bd5
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88890100"
---
# <a name="offline-backup-workflow-for-dpm-and-azure-backup-server-previous-versions"></a>Flujo de trabajo de copia de seguridad sin conexión para DPM y Azure Backup Server (versiones anteriores)

>[!IMPORTANT]
>Estos pasos son aplicables para DPM 2019 RTM y versiones anteriores y MABS v3 RTM y versiones anteriores.

El servicio Azure Backup presenta varias eficiencias integradas para ahorrar costos de almacenamiento y red durante las copias de seguridad iniciales 'completas' de datos en Azure. Las copias de seguridad iniciales completas transfieren grandes cantidades de datos y requieren un mayor ancho de banda de red en comparación con las copias de seguridad sucesivas que solo transfieren los cambios diferenciales e incrementales. Azure Backup permite comprimir las copias de seguridad iniciales. A través del proceso de propagación sin conexión, Azure Backup puede usar discos para cargar los datos comprimidos iniciales de copia de seguridad sin conexión en Azure.

El proceso de propagación sin conexión de Azure Backup se integra estrechamente con el [servicio Azure Import/Export](../storage/common/storage-import-export-service.md). Puede usar este servicio para transferir datos a Azure mediante el uso de discos. Si tiene terabytes (TB) de datos de copia de seguridad inicial que se deben transferir a través de una red de latencia alta y ancho de banda de red bajo, puede usar el flujo de trabajo de propagación sin conexión para enviar la copia de seguridad inicial de una o varias unidades de disco duro a un centro de datos de Azure. En este artículo se proporciona información general y pasos adicionales que finalizan este flujo de trabajo para System Center Data Protection Manager (DPM) y Microsoft Azure Backup Server (MABS).

> [!NOTE]
> El proceso de copia de seguridad sin conexión para el agente de Microsoft Azure Recovery Services (MARS) es distinto del de DPM y el de MABS. Para obtener información sobre cómo usar la copia de seguridad sin conexión con el agente de MARS, consulte el artículo que explica el [flujo de trabajo de la copia de seguridad sin conexión en Azure Backup](backup-azure-backup-import-export.md). No se admite la copia de seguridad sin conexión para copias de seguridad de estado del sistema realizadas mediante el agente de Azure Backup.
>

## <a name="overview"></a>Información general

Con la funcionalidad de propagación sin conexión de Azure Backup y el servicio Azure Import/Export, cargar los datos sin conexión en Azure mediante discos es un proceso sencillo. El proceso de copia de seguridad sin conexión implica los pasos siguientes:

> [!div class="checklist"]
>
> * Los datos de copia de seguridad, en lugar de enviarse a través de la red, se escriben en una ubicación de almacenamiento provisional.
> * A continuación, los datos de la ubicación de almacenamiento provisional se escriben en uno o más discos SATA mediante la utilidad *AzureOfflineBackupDiskPrep*.
> * La utilidad crea automáticamente un trabajo de importación de Azure.
> * Las unidades SATA se envían posteriormente al centro de datos de Azure más próximo.
> * Cuando finaliza la carga de los datos de copia de seguridad en Azure, el servicio Azure Backup copia los datos de copia de seguridad en el almacén de copia de seguridad y se programan las copias de seguridad incrementales.

## <a name="supported-configurations"></a>Configuraciones admitidas

La copia de seguridad sin conexión es compatible con todos los modelos de implementación de Azure Backup que realizan copias de seguridad de datos sin conexión desde entornos locales a la nube de Microsoft. Estos modelos incluyen lo siguiente:

> [!div class="checklist"]
>
> * Copia de seguridad de archivos y carpetas con el agente de MARS o el agente de Azure Backup.
> * Copia de seguridad de todas las cargas de trabajo y archivos con DPM.
> * Copia de seguridad de todas las cargas de trabajo y archivos con MABS.

>[!NOTE]
>Las suscripciones de CSP de Azure no se admiten para su uso con la propagación sin conexión para DPM 2019 RTM y versiones anteriores, y MABS v3 RTM y versiones anteriores. Todavía se admiten copias de seguridad en línea a través de la red.

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de que se cumplen los siguientes requisitos previos antes de iniciar el flujo de trabajo de copia de seguridad sin conexión:

* Se ha creado [el almacén de Recovery Services](backup-azure-recovery-services-vault-overview.md). Para crear uno, siga los pasos que se indican en [Creación de un almacén de Recovery Services](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault).
* Se ha instalado el agente de Azure Backup, MABS o DPM en Windows Server o en un cliente de Windows, según corresponda, y el equipo está registrado con el almacén de Recovery Services. Asegúrese de que solo se utiliza la [versión más reciente de Azure Backup](https://go.microsoft.com/fwlink/?linkid=229525).
* [Descargue el archivo de configuración de publicación de Azure](https://portal.azure.com/#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) en el equipo desde el que piensa hacer copia de seguridad de los datos. La suscripción desde la que descargue el archivo de configuración de publicación puede ser diferente a la suscripción que contiene el almacén de Recovery Services. Si la suscripción se encuentra en nubes soberanas de Azure, use los vínculos siguientes según corresponda para descargar el archivo de configuración de publicación de Azure.

    | Región de nubes soberanas | Vínculo del archivo de configuración de publicación de Azure |
    | --- | --- |
    | Estados Unidos | [Vínculo](https://portal.azure.us#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) |
    | China | [Vínculo](https://portal.azure.cn/#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) |

* Se ha creado una cuenta de Azure Storage con el modelo de implementación de Resource Manager en la suscripción desde la que descargó el archivo de configuración de publicación. En la cuenta de almacenamiento, cree un contenedor de blobs que se usará como destino.

  ![Creación de una cuenta de almacenamiento con el desarrollo de Resource Manager](./media/offline-backup-dpm-mabs-previous-versions/storage-account-resource-manager.png)

* Se crea una ubicación de almacenamiento provisional, que puede ser un recurso compartido de red o cualquier unidad adicional en el equipo, interna o externa, con suficiente espacio en disco para almacenar la copia inicial. Por ejemplo, si intenta realizar una copia de seguridad en un servidor de archivos de 500 GB, asegúrese de que el área de ensayo es de al menos 500 GB. (Se utilizará una cantidad menor gracias a la compresión).
* Con respecto a los discos que se enviarán a Azure, asegúrese de que solo se utilizan unidades de disco duro SSD de 2,5 pulgadas o SATA II/III de 2,5 o 3,5 pulgadas internas. Puede utilizar unidades de disco duro de hasta 10 TB. Vea la [documentación del servicio Azure Import/Export](../storage/common/storage-import-export-requirements.md#supported-hardware) para conocer el conjunto más reciente de unidades de disco que admite el servicio.
* Las unidades de disco SATA deben estar conectadas a un equipo (llamado *equipo de copia*) desde donde se realiza la copia de los datos de copia de seguridad de la ubicación de almacenamiento provisional a SATA. Asegúrese de que BitLocker está habilitado en el equipo de copia.

## <a name="prepare-the-server-for-the-offline-backup-process"></a>Preparación del servidor para el proceso de copia de seguridad sin conexión

>[!NOTE]
> Si no encuentra las utilidades especificadas, como *AzureOfflineBackupCertGen.exe* en su instalación del agente de MARS, escriba a AskAzureBackupTeam@microsoft.com para obtener acceso a estas.

* Abra un símbolo del sistema con privilegios elevados en el servidor y ejecute el comando siguiente:

    ```cmd
    AzureOfflineBackupCertGen.exe CreateNewApplication SubscriptionId:<Subs ID>
    ```

    La herramienta crea una aplicación de Active Directory de copia de seguridad sin conexión de Azure si no existe ninguna.

    Si ya existe una aplicación, este archivo ejecutable le pedirá que cargue manualmente el certificado en la aplicación en el inquilino. Siga los pasos que se indican en [esta sección](#manually-upload-an-offline-backup-certificate) para cargar el certificado manualmente en la aplicación.

* La herramienta *AzureOfflineBackupCertGen.exe* generará un archivo *OfflineApplicationParams.xml*. Copie este archivo en el servidor con MABS o DPM.
* Instale el [agente de MARS más reciente](https://aka.ms/azurebackup_agent) en la instancia de DPM o el servidor de Azure Backup.
* Registre el servidor en Azure.
* Ejecute el siguiente comando:

    ```cmd
    AzureOfflineBackupCertGen.exe AddRegistryEntries SubscriptionId:<subscriptionid> xmlfilepath:<path of the OfflineApplicationParams.xml file>  storageaccountname:<storageaccountname to be used for offline backup>
    ```

* En el ejemplo anterior se crea el archivo `C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch\MicrosoftBackupProvider\OfflineApplicationParams_<Storageaccountname>.xml`.

## <a name="manually-upload-an-offline-backup-certificate"></a>Carga manual del certificado de copia de seguridad sin conexión

Siga los pasos para cargar manualmente el certificado de copia de seguridad sin conexión en una aplicación de Azure Active Directory creada anteriormente para la copia de seguridad sin conexión.

1. Inicie sesión en Azure Portal.
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones**.
1. En la pestaña **Aplicaciones propias** y busque una aplicación con el formato de nombre para mostrar `AzureOfflineBackup _<Azure User Id`.

    ![Búsqueda de aplicaciones en la pestaña Aplicaciones propias](./media/offline-backup-dpm-mabs-previous-versions/owned-applications.png)

1. Seleccione la aplicación. En la pestaña **Administrar** del panel izquierdo, vaya a **Certificados y secretos**.
1. Compruebe si hay certificados o claves públicas preexistentes. Si no hay ninguno, puede seleccionar el botón **Eliminar** de la página **Información general** de la aplicación para eliminar la aplicación de forma segura. Después, puede volver a intentar realizar los pasos del proceso [Preparación del servidor para el proceso de copia de seguridad sin conexión](#prepare-the-server-for-the-offline-backup-process) y saltarse los pasos siguientes. De lo contrario, siga estos pasos desde la instancia de DPM o el servidor de Azure Backup en el que desea configurar la copia de seguridad sin conexión.
1. En **Iniciar** – **Ejecutar**, escriba *Certlm.msc*. En la ventana **Certificados - Equipo local**, seleccione la pestaña **Certificados – Equipo local** > **Personal**. Busque el certificado con el nombre `CB_AzureADCertforOfflineSeeding_<ResourceId>`.
1. Seleccione el certificado anterior, haga clic con el botón derecho en **Todas las tareas** y, después, en **Exportar**, sin clave privada y en formato .cer.
1. Vaya a la aplicación de copia de seguridad sin conexión de Azure en Azure Portal.
1. Seleccione **Administrar** > **Certificados y secretos** > **Cargar certificado**. Cargue el certificado exportado en el paso anterior.

    ![Carga del certificado](./media/offline-backup-dpm-mabs-previous-versions/upload-certificate.png)

1. En el servidor, escriba **regedit** en la ventana Ejecutar para abrir el Registro.
1. Vaya a la entrada del Registro *Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider*.
1. Haga clic con el botón derecho en **CloudBackupProvider** y agregue un nuevo valor de cadena con el nombre `AzureADAppCertThumbprint_<Azure User Id>`.

    >[!NOTE]
    > Para encontrar el identificador de usuario de Azure, realice uno de los pasos siguientes:
    >
    >* Desde la instancia de PowerShell conectada a Azure, ejecute el comando `Get-AzureRmADUser -UserPrincipalName "Account Holder's email as appears in the portal"`.
    >* Vaya a la ruta de acceso del registro `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\DbgSettings\OnlineBackup; Name: CurrentUserId;`.

1. Haga clic con el botón derecho en la cadena agregada en el paso anterior y seleccione **Modificar**. En el valor, proporcione la huella digital del certificado exportado en el paso 7. Después, seleccione **Aceptar**.
1. Para obtener el valor de la huella digital, haga doble clic en el certificado. Seleccione la pestaña **Detalles** y desplácese hacia abajo hasta la huella digital. Seleccione **Huella digital** y copie el valor.

    ![Copia del valor del campo Huella digital](./media/offline-backup-dpm-mabs-previous-versions/thumbprint-field.png)

1. Diríjase a la sección [Flujo de trabajo](#workflow) para continuar con el proceso de copia de seguridad sin conexión.

## <a name="workflow"></a>Flujo de trabajo

La información de esta sección lo ayuda a completar el flujo de trabajo de copia de seguridad sin conexión, por lo que los datos se pueden entregar a un centro de datos de Azure y cargarse en Azure Storage. Si tiene alguna pregunta sobre el servicio de importación o cualquier aspecto del proceso, consulte la documentación sobre la [Información general del servicio de importación](../storage/common/storage-import-export-service.md) a la que se ha hecho referencia anteriormente.

### <a name="initiate-offline-backup"></a>Inicio de la copia de seguridad sin conexión

1. Al programar una copia de seguridad, verá la siguiente página en Windows Server, un cliente de Windows o DPM.

    ![Página Importar](./media/offline-backup-dpm-mabs-previous-versions/offline-backup-screen-inputs.png)

    Esta es la página correspondiente en DPM. <br/>

    ![Página de importación de servidor de DPM y Azure Backup Server](./media/offline-backup-dpm-mabs-previous-versions/dpm-offline.png)

    Las casillas que rellenará son las siguientes:

   * **Ubicación de almacenamiento provisional**: la ubicación de almacenamiento temporal en la que se escribe la copia de seguridad inicial. La ubicación de almacenamiento provisional podría estar en un recurso compartido de red o en un equipo local. Si el equipo de copia y el equipo de origen son diferentes, especifique la ruta de acceso completa de red de la ubicación de ensayo.
   * **Configuración de publicación de Azure**: la ruta de acceso local al archivo de configuración de publicación.
   * **Nombre de trabajo de Azure Import**: el nombre único mediante el que el servicio Azure Import/Export y Azure Backup realizan el seguimiento de la transferencia de los datos enviados en discos a Azure.
   * **Identificador de suscripción de Azure**: el identificador de la suscripción de Azure con la que descargó el archivo de configuración de publicación de Azure.
   * **Cuenta de Azure Storage**: el nombre de la cuenta de almacenamiento de la suscripción de Azure asociada al archivo de configuración de publicación de Azure.
   * **Contenedor de Azure Storage**: el nombre del blob de almacenamiento de destino de la cuenta de Azure Storage donde se importan los datos de copia de seguridad.

   Guarde la **ubicación de almacenamiento provisional** y el **nombre del trabajo de importación de Azure** proporcionados. Es necesario preparar los discos.

1. Finalice el flujo de trabajo. Para iniciar la copia de seguridad sin conexión, seleccione **Hacer copia de seguridad ahora** en la consola de administración del agente de Azure Backup. La copia de seguridad inicial se escribe en el área de ensayo como parte de este paso.

    ![Realizar copia de seguridad ahora](./media/offline-backup-dpm-mabs-previous-versions/backup-now.png)

    Para finalizar el flujo de trabajo correspondiente en DPM o Azure Backup Server, haga clic con el botón derecho en **Grupo de protección**. Seleccione la opción **Crear punto de recuperación**. A continuación, seleccione la opción **Protección en línea**.

    ![Hacer copia de seguridad ahora en DPM y MABS](./media/offline-backup-dpm-mabs-previous-versions/dpm-backup-now.png)

    Una vez finalizada la operación, la ubicación de ensayo estará lista para usarse en la preparación del disco.

    ![Progreso de la copia de seguridad](./media/offline-backup-dpm-mabs-previous-versions/op-backup-now.png)

### <a name="prepare-sata-drives-and-ship-to-azure"></a>Preparación de unidades SATA y envío a Azure

La utilidad *AzureOfflineBackupDiskPrep* se usa para preparar las unidades SATA que se envían al centro de datos de Azure más próximo. Esta utilidad está disponible en el directorio de instalación del agente de Recovery Services en la ruta de acceso siguiente:

`*\\Microsoft Azure Recovery Services Agent\Utils\*`

1. Vaya al directorio y copie el directorio de *AzureOfflineBackupDiskPrep* en un equipo de copia en el que se conectarán las unidades SATA que se van a preparar. Asegúrese de que:

   * El equipo de copia puede acceder a la ubicación de ensayo del flujo de trabajo de propagación sin conexión mediante la misma ruta de acceso de red proporcionada durante el flujo de trabajo de la sección "Inicio de la copia de seguridad sin conexión".
   * BitLocker está habilitado en el equipo de copia.
   * El equipo de copia puede acceder a Azure Portal. Si es necesario, el equipo de copia puede ser el mismo que el equipo de origen.

     > [!IMPORTANT]
     > Si el equipo de origen es una máquina virtual, es obligatorio utilizar un servidor físico o equipo cliente diferentes como equipo de copia.

1. Abra un símbolo del sistema con privilegios elevados en el equipo de copia con el directorio de la utilidad *AzureOfflineBackupDiskPrep* como directorio actual. Ejecute el siguiente comando:

    `*.\AzureOfflineBackupDiskPrep.exe*   s:<*Staging Location Path*>   [p:<*Path to AzurePublishSettingsFile*>]`

    | Parámetro | Descripción |
    | --- | --- |
    | s:&lt;*Ruta de acceso de la ubicación de ensayo*&gt; |Esta entrada obligatoria se usa para proporcionar la ruta de acceso a la ubicación de almacenamiento provisional que escribió en el flujo de trabajo en la sección "Iniciar copia de seguridad sin conexión". |
    | p:&lt;*Ruta de acceso a PublishSettingsFile*&gt; |Esta entrada obligatoria se utiliza para proporcionar la ruta de acceso al archivo de Configuración de publicación de Azure especificado en el flujo de trabajo de la sección "Inicio de la copia de seguridad sin conexión". |

    > [!NOTE]
    > El valor de &lt;Ruta de acceso a AzurePublishSettingFile&gt; es obligatorio cuando el equipo de copia y el equipo de origen son diferentes.
    >
    >

    Cuando se ejecuta el comando, la utilidad solicita la selección del trabajo de importación de Azure correspondiente a las unidades que deben estar preparadas. Si hay un solo trabajo de importación asociado con la ubicación de ensayo proporcionada, aparece una pantalla como esta.

    ![Entrada de la herramienta de preparación de discos de Azure](./media/offline-backup-dpm-mabs-previous-versions/azure-disk-preparation-tool-drive-input.png) <br/>

1. Escriba la letra de la unidad del disco montado (sin los dos puntos finales) que quiera preparar para la transferencia a Azure. Proporcione la confirmación del formato de la unidad cuando se le solicite.

    La herramienta comienza entonces a preparar el disco y a copiar los datos de copia de seguridad. Puede que deba conectar discos adicionales si se lo pide la herramienta, si el disco proporcionado no tiene espacio suficiente para los datos de copia de seguridad. <br/>

    Al final de la ejecución correcta de la herramienta, uno o varios discos que proporcionó estarán preparados para el envío a Azure. También se creará en Azure un trabajo de importación con el nombre que proporcionó durante el flujo de trabajo en la sección "Inicio de la copia de seguridad sin conexión". Finalmente, la herramienta muestra la dirección de envío al centro de datos de Azure adonde es necesario enviar los discos.

    ![Preparación de discos de Azure finalizada](./media/offline-backup-dpm-mabs-previous-versions/azure-disk-preparation-tool-success.png)<br/>

1. Al final de la ejecución del comando, también verá la opción para actualizar la información de envío.

    ![Opción para actualizar la información de envío](./media/offline-backup-dpm-mabs-previous-versions/update-shipping-utility.png)<br/>

1. Puede escribir los detalles inmediatamente. La herramienta lo guía a través del proceso, que requiere que especifique diversos datos. Si no tiene información como el número de seguimiento u otros datos relacionados con el envío, puede finalizar la sesión. Los pasos para actualizar los detalles de envío se describen más adelante en este artículo.

1. Envíe los discos a la dirección proporcionada por la herramienta. Tenga a mano el número de seguimiento por si lo necesita más adelante.

   > [!IMPORTANT]
   > El número de seguimiento no puede ser el mismo para distintos trabajos de importación de Azure. Asegúrese de que las unidades preparadas con la utilidad en un trabajo de importación de Azure se envían juntas en un único paquete y de que el paquete tenga un número de seguimiento exclusivo. No combine unidades preparadas como parte de diferentes trabajos de importación de Azure en un único paquete.

1. Cuando tenga la información del número de seguimiento, vaya al equipo de origen, que está esperando a que se complete el trabajo de importación. Abra el siguiente símbolo del sistema con privilegios elevados con el directorio de la utilidad *AzureOfflineBackupDiskPrep* como directorio actual.

   `*.\AzureOfflineBackupDiskPrep.exe*  u:`

   También puede ejecutar el siguiente comando desde otro equipo, como el equipo de copia, con el directorio de la utilidad *AzureOfflineBackupDiskPrep* como directorio actual.

   `*.\AzureOfflineBackupDiskPrep.exe*  u:  s:<*Staging Location Path*>   p:<*Path to AzurePublishSettingsFile*>`

    | Parámetro | Descripción |
    | --- | --- |
    | u: | Esta entrada obligatoria se usa para actualizar los detalles de envío de un trabajo de importación de Azure. |
    | s:&lt;*Ruta de acceso de la ubicación de ensayo*&gt; | Esta entrada obligatoria se usa cuando el comando no se ejecuta en el equipo de origen. Se utiliza para proporcionar la ruta de acceso a la ubicación de ensayo especificada en el flujo de trabajo de la sección "Inicio de la copia de seguridad sin conexión". |
    | p:&lt;*Ruta de acceso a PublishSettingsFile*&gt; | Esta entrada obligatoria se usa cuando el comando no se ejecuta en el equipo de origen. Se utiliza para proporcionar la ruta de acceso al archivo de Configuración de publicación de Azure especificado en el flujo de trabajo de la sección "Inicio de la copia de seguridad sin conexión". |

    La utilidad detecta automáticamente el trabajo de importación que está esperando el equipo de origen o los trabajos de importación asociados a la ubicación de almacenamiento provisional cuando el comando se ejecuta en un equipo diferente. Después proporciona la opción para actualizar la información de envío a través de una serie de valores que deben especificarse.

    ![Especificación de la información de envío](./media/offline-backup-dpm-mabs-previous-versions/shipping-inputs.png)<br/>

1. Una vez especificados todos los valores, revise cuidadosamente los detalles y confirme la información de envío que ha proporcionado escribiendo **yes**.

    ![Revisión de la información de envío](./media/offline-backup-dpm-mabs-previous-versions/review-shipping-information.png)<br/>

1. Al actualizar correctamente la información de envío, la utilidad proporciona una ubicación local donde se almacenan los detalles de envío que ha especificado.

    ![Almacenamiento de la información de envío](./media/offline-backup-dpm-mabs-previous-versions/storing-shipping-information.png)<br/>

   > [!IMPORTANT]
   > Asegúrese de que las unidades lleguen el centro de datos de Azure en el plazo de dos semanas tras proporcionar la información de envío mediante la utilidad *AzureOfflineBackupDiskPrep*. De lo contrario, es posible que las unidades no se procesen.

Cuando haya completado los pasos anteriores, el centro de datos de Azure estará listo para recibir las unidades y procesarlas para transferir los datos de copia de seguridad de las unidades a la cuenta de almacenamiento de Azure clásica que ha creado.

### <a name="time-to-process-the-drives"></a>Tiempo para procesar las unidades

La cantidad de tiempo que se tarda en procesar un trabajo de importación de Azure varía. El tiempo depende de factores como el tiempo de envío, el tipo de trabajo, el tipo y el tamaño de los datos que se copian, y el tamaño de los discos proporcionados. El servicio Azure Import/Export no tiene un Acuerdo de Nivel de Servicio. Una vez que se reciben los discos, el servicio trata de finalizar la copia de los datos de copia de seguridad en la cuenta de almacenamiento de Azure en 7 y 10 días. La siguiente sección detalla cómo puede supervisar el estado del trabajo de importación de Azure.

### <a name="monitor-azure-import-job-status"></a>Supervisión del estado del trabajo de importación de Azure

Mientras que las unidades están en tránsito o en el centro de datos de Azure para copiarse en la cuenta de almacenamiento, el agente de Azure Backup, DPM o la consola MABS en el equipo de origen muestran el estado de trabajo siguiente para las copias de seguridad programadas:

  `Waiting for Azure Import Job to complete. Please check on Azure Management portal for more information on job status`

Para comprobar el estado del trabajo de importación:

1. Abra un símbolo del sistema con privilegios elevados en el equipo de origen y ejecute el comando siguiente:

     `AzureOfflineBackupDiskPrep.exe u:`

1. El resultado muestra el estado actual del trabajo de importación.

    ![Comprobación del estado del trabajo de importación](./media/offline-backup-dpm-mabs-previous-versions/import-job-status-reporting.png)<br/>

Para obtener más información sobre los distintos estados del trabajo de importación de Azure, consulte [Visualización del estado de los trabajos de Azure Import/Export](../storage/common/storage-import-export-view-drive-status.md).

### <a name="finish-the-workflow"></a>Finalización del flujo de trabajo

Una vez completado el trabajo de importación, los datos de la copia de seguridad inicial están disponibles en la cuenta de almacenamiento. En la siguiente copia de seguridad programada, Azure Backup copia el contenido de los datos de la cuenta de almacenamiento en el almacén de Recovery Services.

   ![Copia de datos en el almacén de Recovery Services](./media/offline-backup-dpm-mabs-previous-versions/copying-from-storage-account-to-azure-backup.png)<br/>

En la siguiente copia de seguridad programada, Azure Backup realiza la copia de seguridad incremental sobre la copia de seguridad inicial.

## <a name="next-steps"></a>Pasos siguientes

* Si tiene dudas del flujo de trabajo del servicio Azure Import/Export, consulte [Uso del servicio Microsoft Azure Import/Export para transferir datos a Blob Storage](../storage/common/storage-import-export-service.md).
