---
title: Inicialización de la copia de seguridad sin conexión mediante el servicio Azure Import/Export
description: Obtenga información sobre cómo puede usar Azure Backup para enviar datos fuera de la red mediante el servicio Azure Import/Export. Este artículo explica la propagación sin conexión de los datos de copia de seguridad iniciales mediante el servicio de Azure Import/Export.
ms.reviewer: saurse
ms.topic: conceptual
ms.date: 05/17/2018
ms.openlocfilehash: f3cf44a34babab79d135923db040630a1c8e3dfe
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88892021"
---
# <a name="offline-backup-workflow-in-azure-backup"></a>Flujo de trabajo de copia de seguridad sin conexión en Azure Backup

El servicio Azure Backup presenta varias eficiencias integradas para ahorrar costos de almacenamiento y red durante las copias de seguridad iniciales 'completas' de datos en Azure. Las copias de seguridad iniciales completas transfieren grandes cantidades de datos y requieren un mayor ancho de banda de red en comparación con las copias de seguridad sucesivas que solo transfieren los cambios diferenciales e incrementales. A través del proceso de propagación sin conexión, Azure Backup puede usar discos para cargar los datos de copia de seguridad sin conexión en Azure.

El proceso de propagación sin conexión de Azure Backup se integra estrechamente con el [servicio Azure Import/Export](../storage/common/storage-import-export-service.md). Puede usar este servicio para transferir datos de la copia de seguridad inicial a Azure mediante el uso de discos. Si tiene terabytes (TB) de datos de copia de seguridad inicial que se deben transferir a través de una red de latencia alta y ancho de banda de red bajo, puede usar el flujo de trabajo de propagación sin conexión para enviar la copia de seguridad inicial de una o varias unidades de disco duro a un centro de datos de Azure. En la imagen siguiente se proporciona información general de los pasos del flujo de trabajo.

  ![Información general del proceso del flujo de trabajo de importación sin conexión](./media/backup-azure-backup-import-export/offlinebackupworkflowoverview.png)

El proceso de copia de seguridad sin conexión conlleva estos pasos:

1. En lugar de enviar los datos de la copia de seguridad a través de la red, escríbalos en una ubicación de almacenamiento provisional.
1. Use la utilidad *AzureOfflineBackupDiskPrep* para escribir los datos de la ubicación de almacenamiento provisional en uno o más discos SATA.
1. Como parte del trabajo de preparación, la utilidad *AzureOfflineBackupDiskPrep* crea un trabajo de importación de Azure. Envíe las unidades de disco SATA al centro de datos de Azure más cercano y haga referencia al trabajo de importación para conectar las actividades.
1. En el centro de datos de Azure, los datos de los discos se copian en una cuenta de almacenamiento de Azure.
1. Azure Backup copia los datos de la copia de seguridad de la cuenta de almacenamiento al almacén de Recovery Services y se programan copias de seguridad incrementales.

## <a name="supported-configurations"></a>Configuraciones admitidas

Las características o cargas de trabajo de Azure Backup siguientes admiten el uso de la copia de seguridad sin conexión:

> [!div class="checklist"]
>
> * Copia de seguridad de archivos y carpetas con el agente de Microsoft Azure Recovery Services (MARS), también conocido como el agente de Azure Backup.
> * Copia de seguridad de todas las cargas de trabajo y archivos con System Center Data Protection Manager (DPM).
> * Copia de seguridad de todas las cargas de trabajo y archivos con Microsoft Azure Backup Server.

   > [!NOTE]
   > No se admite la copia de seguridad sin conexión para copias de seguridad de estado del sistema realizadas mediante el agente de Azure Backup.

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="prerequisites"></a>Requisitos previos

  > [!NOTE]
  > El flujo de trabajo y los requisitos previos siguientes solo se aplican a la copia de seguridad sin conexión de archivos y carpetas con el [agente Azure Recovery Services más reciente](https://aka.ms/azurebackup_agent). Para crear copias de seguridad sin conexión de las cargas de trabajo con System Center DPM o Azure Backup Server, consulte [Flujo de trabajo de copia de seguridad sin conexión para DPM y Azure Backup Server](backup-azure-backup-server-import-export.md).

Antes de iniciar el flujo de trabajo de copia de seguridad sin conexión, complete estos requisitos previos:

* Cree un [almacén de Recovery Services](backup-azure-recovery-services-vault-overview.md). Para crear un almacén, siga los pasos descritos en [Creación de un almacén de Recovery Services](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault).
* Asegúrese de que solo la [versión más reciente del agente de Azure Backup](https://aka.ms/azurebackup_agent) se instaló en el cliente de Windows Server o Windows, según corresponda, y que el equipo está registrado con el almacén de Recovery Services.
* Se requiere Azure PowerShell 3.7.0 en el equipo donde se ejecuta el agente de Azure Backup. Descargue e [Instale la versión 3.7.0 de Azure PowerShell](https://github.com/Azure/azure-powershell/releases/tag/v3.7.0-March2017).
* En el equipo donde se ejecuta el agente de Azure Backup, asegúrese de que esté instalado Microsoft Edge o Internet Explorer 11 y que JavaScript esté habilitado.
* Cree una cuenta de Azure Storage en la misma suscripción que el almacén de Recovery Services.
* Asegúrese de tener los [permisos necesarios](../active-directory/develop/howto-create-service-principal-portal.md) para crear la aplicación de Azure Active Directory. El flujo de trabajo de la copia de seguridad sin conexión crea una aplicación de Azure Active Directory en la suscripción asociada con la cuenta de Azure Storage. El objetivo de la aplicación es proporcionar Azure Backup con el acceso seguro y limitado al servicio Azure Import/Export que se requiere para el flujo de trabajo de la copia de seguridad sin conexión.
* Registre el proveedor de recursos *Microsoft.ImportExport* con la suscripción que contiene la cuenta de Azure Storage. Para registrar el proveedor de recursos:
    1. En el menú principal, seleccione **Suscripciones**.
    1. Si está suscrito a varias suscripciones, seleccione la suscripción que usa para la copia de seguridad sin conexión. Si solo usa una, aparecerá esa suscripción.
    1. En el menú de la suscripción, seleccione **Proveedores de recursos** para ver la lista de proveedores.
    1. En la lista de proveedores, desplácese hacia abajo hasta *Microsoft.ImportExport*. Si el **estado** es **No registrado**, seleccione **Registrar**.

        ![Registrar el proveedor de recursos](./media/backup-azure-backup-import-export/registerimportexport.png)

* Se crea una ubicación de almacenamiento provisional, que puede ser un recurso compartido de red o cualquier unidad adicional en el equipo, interna o externa, con suficiente espacio en disco para almacenar la copia inicial. Por ejemplo, si intenta realizar una copia de seguridad en un servidor de archivos de 500 GB, asegúrese de que el área de ensayo es de al menos 500 GB. (Se utilizará una cantidad menor gracias a la compresión).
* Cuando envíe los discos a Azure, use solo SSD de 2,5 pulgadas o discos duros internos SATA II/III de 2,5 o 3,5 pulgadas. Puede utilizar unidades de disco duro de hasta 10 TB. Vea la [documentación del servicio Azure Import/Export](../storage/common/storage-import-export-requirements.md#supported-hardware) para conocer el conjunto más reciente de unidades de disco que admite el servicio.
* Las unidades de disco SATA deben estar conectadas a un equipo (llamado *equipo de copia*) desde donde se realiza la copia de los datos de copia de seguridad de la ubicación de almacenamiento provisional a SATA. Asegúrese de que BitLocker está habilitado en el equipo de copia.

## <a name="workflow"></a>Flujo de trabajo

En esta sección se describe el flujo de trabajo de la copia de seguridad sin conexión, por lo que los datos se pueden entregar a un centro de datos de Azure y cargarse en Azure Storage. Si tiene alguna pregunta sobre el servicio de importación o cualquier aspecto del proceso, consulte la [documentación sobre la información general del servicio Azure Import/Export](../storage/common/storage-import-export-service.md).

## <a name="initiate-offline-backup"></a>Inicio de la copia de seguridad sin conexión

1. Al programar una copia de seguridad en el agente de Recovery Services, verá esta página.

    ![Página Importar](./media/backup-azure-backup-import-export/offlinebackup_inputs.png)

1. Seleccione la opción **Transfer using my own disks** (Transferir con mis propios discos).

    > [!NOTE]
    > Use la opción Azure Data Box para transferir los datos de copia de seguridad iniciales sin conexión. Esta opción ahorra el esfuerzo necesario para adquirir sus propios discos compatibles con Azure. Proporciona dispositivos de Azure Data Box seguros, con mecanismo a prueba de manipulaciones y con derechos de propiedad de Microsoft en los que el agente de Recovery Services puede escribir directamente los datos de copia de seguridad.

1. Seleccione **Siguiente**y rellene las casillas con cuidado.

    ![Escribir los detalles del disco](./media/backup-azure-backup-import-export/your-disk-details.png)

   Las casillas que rellenará son las siguientes:

    * **Ubicación de almacenamiento provisional**: la ubicación de almacenamiento temporal en la que se escribe la copia de seguridad inicial. La ubicación de almacenamiento provisional podría estar en un recurso compartido de red o en un equipo local. Si el equipo de copia y el equipo de origen son diferentes, especifique la ruta de acceso completa de red de la ubicación de ensayo.
    * **Cuenta de almacenamiento de Azure Resource Manager**: El nombre de la cuenta de almacenamiento de tipo Resource Manager (de uso general v1 o v2) en cualquier suscripción de Azure.
    * **Contenedor de Azure Storage**: el nombre del contenedor de almacenamiento del blob de destino de la cuenta de Azure Storage en la que se importan los datos de la copia de seguridad antes de que se copien en el almacén de Recovery Services.
    * **Identificador de suscripción de Azure**: el identificador de la suscripción de Azure en la que se crea la cuenta de Azure Storage.
    * **Nombre de trabajo de Azure Import**: el nombre único mediante el que el servicio Azure Import/Export y Azure Backup realizan el seguimiento de la transferencia de los datos enviados en discos a Azure.
  
   Después de rellenar las casillas, seleccione **Siguiente**. Guarde la **ubicación de almacenamiento provisional** y el **nombre del trabajo de importación de Azure**. Es necesario preparar los discos.

1. Cuando se le solicite, inicie sesión en su suscripción de Azure. Debe iniciar sesión para que Azure Backup pueda crear la aplicación de Azure Active Directory. Escriba los permisos necesarios para acceder al servicio Azure Import/Export.

    ![Página de inicio de sesión de la suscripción de Azure](./media/backup-azure-backup-import-export/azure-login.png)

1. Finalice el flujo de trabajo. En la consola del agente de Azure Backup, seleccione **Hacer copia de seguridad ahora**.

    ![Hacer copia de seguridad ahora](./media/backup-azure-backup-import-export/backupnow.png)

1. En la página **Confirmación** del asistente, haga clic en **Hacer copia de seguridad ahora**. La copia de seguridad inicial se escribe en el área de ensayo como parte de esta configuración.

   ![Confirmación de que está listo para hacer una copia de seguridad](./media/backup-azure-backup-import-export/backupnow-confirmation.png)

    Una vez finalizada la operación, la ubicación de ensayo estará lista para usarse en la preparación del disco.

   ![Página del Asistente para hacer copia de seguridad ahora](./media/backup-azure-backup-import-export/opbackupnow.png)

## <a name="prepare-sata-drives-and-ship-to-azure"></a>Preparación de unidades SATA y envío a Azure

La utilidad *AzureOfflineBackupDiskPrep* prepara las unidades de disco SATA que se envían al centro de datos de Azure más próximo. Esta utilidad está disponible en el directorio de instalación del agente de Azure Backup (en la ruta de acceso siguiente):

```*\Microsoft Azure Recovery Services Agent\Utils\\*```

1. Vaya al directorio y copie el directorio *AzureOfflineBackupDiskPrep* en otro equipo donde estén conectadas las unidades de disco SATA. En el equipo con las unidades de disco SATA conectadas, asegúrese de lo siguiente:

   * El equipo de copia puede acceder a la ubicación de ensayo del flujo de trabajo de propagación sin conexión mediante la misma ruta de acceso de red proporcionada durante el flujo de trabajo de la sección "Inicio de la copia de seguridad sin conexión".
   * BitLocker está habilitado en el equipo de copia.
   * Azure PowerShell 3.7.0 está instalado.
   * Los exploradores compatibles más recientes (Microsoft Edge o Internet Explorer 11) están instalados y JavaScript, habilitado.
   * El equipo de copia puede acceder a Azure Portal. Si es necesario, el equipo de copia puede ser el mismo que el equipo de origen.

     > [!IMPORTANT]
     > Si el equipo de origen es una máquina virtual, el equipo de copia debe estar en una máquina cliente o un servidor físico distinto del equipo de origen.

1. Abra un símbolo del sistema con privilegios elevados en el equipo de copia con el directorio de la utilidad *AzureOfflineBackupDiskPrep* como directorio actual. Ejecute el siguiente comando:

    ```.\AzureOfflineBackupDiskPrep.exe s:<Staging Location Path>```

    | Parámetro | Descripción |
    | --- | --- |
    | s:&lt;*Ruta de acceso de la ubicación de ensayo*&gt; |Esta entrada obligatoria se usa para proporcionar la ruta de acceso a la ubicación de almacenamiento provisional que escribió en el flujo de trabajo en la sección "Iniciar copia de seguridad sin conexión". |
    | p:&lt;*Ruta de acceso a PublishSettingsFile*&gt; |Esta entrada opcional se usa para proporcionar la ruta de acceso al archivo de configuración de publicación de Azure.  |

    Cuando se ejecuta el comando, la utilidad solicita la selección del trabajo de importación de Azure correspondiente a las unidades que deben estar preparadas. Si hay un solo trabajo de importación asociado con la ubicación de ensayo proporcionada, aparece una pantalla como esta.

    ![Entrada de la herramienta de preparación de discos de Azure](./media/backup-azure-backup-import-export/diskprepconsole0_1.png) <br/>

1. Escriba la letra de la unidad del disco montado (sin los dos puntos finales) que quiera preparar para la transferencia a Azure.
1. Proporcione la confirmación del formato de la unidad cuando se le solicite.
1. Se le indica que inicie sesión en su suscripción o suscripciones de Azure. Escriba sus credenciales.

    ![Inicio de sesión en la suscripción de Azure](./media/backup-azure-backup-import-export/signindiskprep.png) <br/>

    La herramienta comienza entonces a preparar el disco y a copiar los datos de copia de seguridad. Puede que deba conectar discos adicionales si se lo pide la herramienta, si el disco proporcionado no tiene espacio suficiente para los datos de copia de seguridad. <br/>

    Al terminar la ejecución correcta de la herramienta, el símbolo del sistema proporciona tres informaciones distintas:

   1. Uno o varios de los discos que proporcionó están preparados para enviarlos a Azure.
   1. Recibe la confirmación de que se creó el trabajo de importación. El trabajo de importación usa el nombre que proporcionó.
   1. La herramienta muestra la dirección de envío del centro de datos de Azure.

      ![Preparación de discos de Azure finalizada](./media/backup-azure-backup-import-export/console2.png)<br/>

1. Cuando se complete la ejecución del comando, puede actualizar la información de envío.

1. Envíe los discos a la dirección proporcionada por la herramienta. Tenga a mano el número de seguimiento por si lo necesita más adelante.

   > [!IMPORTANT]
   > El número de seguimiento no puede ser el mismo para distintos trabajos de importación de Azure. Asegúrese de que las unidades preparadas con la utilidad en un trabajo de importación de Azure se envían juntas en un único paquete y de que el paquete tenga un número de seguimiento exclusivo. No combine unidades preparadas como parte de trabajos de importación de Azure independientes en un solo paquete.

## <a name="update-shipping-details-on-the-azure-import-job"></a>Actualización de los detalles de envío del trabajo de importación de Azure

El procedimiento siguiente actualiza los detalles del envío del trabajo de importación de Azure. Esta información incluye detalles sobre:

* el nombre del operador que entrega los discos a Azure.
* Los detalles de envío en caso de devolución de los discos.

1. Inicie sesión en la suscripción de Azure.
1. En el menú principal, seleccione **Todos los servicios**. En el cuadro de diálogo **Todos los servicios**, escriba **Importar**. Cuando vea **Trabajos de Import/Export**, selecciónelo.

    ![Especificación de la información de envío](./media/backup-azure-backup-import-export/search-import-job.png)<br/>

    Se abre el menú de **Trabajos de Import/Export** y aparecen todos los trabajos de Import/Export de la suscripción seleccionada.

1. Si tiene varias suscripciones, seleccione la suscripción que se usó para importar los datos de la copia de seguridad. Luego, seleccione el trabajo de importación recién creado para abrir sus detalles.

    ![Revisión de la información de envío](./media/backup-azure-backup-import-export/import-job-found.png)<br/>

1. En el menú **Configuración** del trabajo de importación, seleccione **Administrar la información de envío**. Escriba los detalles de envío de devolución.

    ![Almacenamiento de la información de envío](./media/backup-azure-backup-import-export/shipping-info.png)<br/>

1. Cuando tenga el número de seguimiento del transportista de envío, seleccione el banner en la página de información general del trabajo de importación de Azure y escriba estos detalles.

   > [!IMPORTANT]
   > Asegúrese de que la información del transportista y el número de seguimiento se actualicen en dos semanas a contar de la creación del trabajo de importación de Azure. No comprobar esta información dentro de dos semanas puede generar que se elimine el trabajo y que no se procese.

   ![Alerta de actualización de información de seguimiento](./media/backup-azure-backup-import-export/joboverview.png)<br/>

   ![Información del operador y número de seguimiento](./media/backup-azure-backup-import-export/tracking-info.png)

### <a name="time-to-process-the-drives"></a>Tiempo para procesar las unidades

La cantidad de tiempo que se tarda en procesar un trabajo de importación de Azure varía. El tiempo depende de factores como el tiempo de envío, el tipo de trabajo, el tipo y el tamaño de los datos que se copian, y el tamaño de los discos proporcionados. El servicio Azure Import/Export no tiene un Acuerdo de Nivel de Servicio. Una vez que se reciben los discos, el servicio trata de finalizar la copia de los datos de copia de seguridad en la cuenta de almacenamiento de Azure en 7 y 10 días.

### <a name="monitor-azure-import-job-status"></a>Supervisión del estado del trabajo de importación de Azure

Puede supervisar el estado del trabajo de importación desde Azure Portal. Vaya a la página **Trabajos de Import/Export** y seleccione el trabajo. Para obtener más información sobre el estado de los trabajos de importación, consulte [¿Qué es el servicio Azure Import/Export?](../storage/common/storage-import-export-service.md).

### <a name="finish-the-workflow"></a>Finalización del flujo de trabajo

Una vez que el trabajo de importación se completa correctamente, los datos de la copia de seguridad inicial están disponibles en la cuenta de almacenamiento. En la siguiente copia de seguridad programada, Azure Backup copia el contenido de los datos de la cuenta de almacenamiento en el almacén de Recovery Services.

   ![Copia de datos en el almacén de Recovery Services](./media/backup-azure-backup-import-export/copyingfromstorageaccounttoazurebackup.png)<br/>

En la siguiente copia de seguridad programada, Azure Backup realiza una copia de seguridad incremental.

### <a name="clean-up-resources"></a>Limpieza de recursos

Una vez que se completa la copia de seguridad inicial, puede eliminar de manera segura los datos importados al contenedor de Azure Storage y los datos de la copia de seguridad de la ubicación de almacenamiento provisional.

## <a name="next-steps"></a>Pasos siguientes

* Si tiene dudas del flujo de trabajo del servicio Azure Import/Export, consulte [Uso del servicio Microsoft Azure Import/Export para transferir datos a Blob Storage](../storage/common/storage-import-export-service.md).
