---
title: Copia de seguridad sin conexión para DPM y Azure Backup Server
description: Con Azure Backup, puede enviar datos fuera de la red mediante el servicio Azure Import/Export. En este artículo se explica el flujo de trabajo de la copia de seguridad sin conexión para DPM y Azure Backup Server.
ms.topic: conceptual
ms.date: 05/24/2020
ms.openlocfilehash: 368ae846a24ec04ee4b7da9b5971c00180be611d
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378464"
---
# <a name="offline-backup-workflow-for-dpm-and-azure-backup-server-mabs"></a>Flujo de trabajo de copia de seguridad sin conexión para DPM y Azure Backup Server (MABS)

>[!IMPORTANT]
> Estos pasos son aplicables para DPM 2019 UR1 (o versiones superiores) y MABS v3 UR1 (o versiones superiores).

System Center Data Protection Manager y Azure Backup Server (MABS) se integran con Azure Backup y usan varias eficacias integradas que ahorran costos de redes y almacenamiento durante las copias de seguridad completas de datos iniciales en Azure. Las copias de seguridad iniciales completas transfieren grandes cantidades de datos y requieren un mayor ancho de banda de red en comparación con las copias de seguridad sucesivas que solo transfieren los cambios diferenciales e incrementales. Azure Backup permite comprimir las copias de seguridad iniciales. A través del proceso de propagación sin conexión, Azure Backup puede usar discos para cargar los datos comprimidos iniciales de copia de seguridad sin conexión en Azure.

El proceso de propagación sin conexión de Azure Backup se integra estrechamente con el [servicio Azure Import/Export](../storage/common/storage-import-export-service.md). Puede usar este servicio para transferir datos a Azure mediante el uso de discos. Si tiene terabytes (TB) de datos de copia de seguridad inicial que se deben transferir a través de una red de latencia alta y ancho de banda de red bajo, puede usar el flujo de trabajo de propagación sin conexión para enviar la copia de seguridad inicial de una o varias unidades de disco duro a un centro de datos de Azure. En este artículo se proporciona información general y pasos adicionales que finalizan este flujo de trabajo para System Center Data Protection Manager (DPM) y Microsoft Azure Backup Server (MABS).

> [!NOTE]
> El proceso de copia de seguridad sin conexión para el agente de Microsoft Azure Recovery Services (MARS) es distinto del de DPM y el de MABS. Para obtener información sobre cómo usar la copia de seguridad sin conexión con el agente de MARS, consulte el artículo que explica el [flujo de trabajo de la copia de seguridad sin conexión en Azure Backup](backup-azure-backup-import-export.md). No se admite la copia de seguridad sin conexión para copias de seguridad de estado del sistema realizadas mediante el agente de Azure Backup.
>
> La actualización de MABS UR1 también aporta la versión preliminar para Copia de seguridad sin conexión con Azure Data Box en MABS. Póngase en contacto con [SystemCenterFeedback@microsoft.com](mailto:SystemCenterFeedback@microsoft.com) para más información.

## <a name="overview"></a>Información general

Con la funcionalidad de propagación sin conexión de Azure Backup y el servicio Azure Import/Export, cargar los datos sin conexión en Azure mediante discos es un proceso sencillo. El proceso de copia de seguridad sin conexión implica los pasos siguientes:

> [!div class="checklist"]
>
> * Los datos de copia de seguridad, en lugar de enviarse a través de la red, se escriben en una ubicación de almacenamiento provisional.
> * A continuación, los datos de la ubicación de almacenamiento provisional se escriben en uno o más discos SATA mediante la utilidad *AzureOfflineBackupDiskPrep*.
> * La utilidad crea automáticamente un trabajo de importación de Azure.
> * Las unidades SATA se envían posteriormente al centro de datos de Azure más próximo.
> * Cuando finaliza la carga de los datos de copia de seguridad en Azure, el servicio Azure Backup copia los datos de copia de seguridad en el almacén de copia de seguridad y se programan las copias de seguridad incrementales.

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de que se cumplen los siguientes requisitos previos antes de iniciar el flujo de trabajo de copia de seguridad sin conexión:

* Se ha creado [el almacén de Recovery Services](backup-azure-recovery-services-vault-overview.md). Para crear uno, siga los pasos en [Creación de un almacén de Recovery Services](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault)tutorial-backup-windows-server-to-azure#create-a-recovery-services-vault).
* Asegúrese de que solo la [versión más reciente del agente de Microsoft Azure Recovery Services](https://aka.ms/azurebackup_agent) esté instalada en SC DPM o MABS y registrada en el almacén de Recovery Services.
* El paquete acumulativo de actualizaciones 1 está instalado en SC DPM 2019 o MABS v3.

  > [!NOTE]
  > Con DPM 2019 UR1 y MABS v3 UR1, la inicialización sin conexión se autentica mediante Azure Active Directory.

* En el servidor de DPM o MABS, asegúrese de que esté instalado Microsoft Edge o Internet Explorer 11 y que JavaScript esté habilitado.
* Cree una cuenta de Azure Storage en la misma suscripción que el almacén de Recovery Services.
* Asegúrese de tener los [permisos necesarios](../active-directory/develop/howto-create-service-principal-portal.md) para crear la aplicación de Azure Active Directory. El flujo de trabajo de la copia de seguridad sin conexión crea una aplicación de Azure Active Directory en la suscripción asociada con la cuenta de Azure Storage. El objetivo de la aplicación es proporcionar Azure Backup con el acceso seguro y limitado al servicio Azure Import que se requiere para el flujo de trabajo de la copia de seguridad sin conexión.
* Registre el proveedor de recursos Microsoft.ImportExport con la suscripción que contiene la cuenta de Azure Storage. Para registrar el proveedor de recursos:
    1. En el menú principal, seleccione **Suscripciones**.
    2. Si está suscrito a varias suscripciones, seleccione la suscripción que usa para la copia de seguridad sin conexión. Si solo usa una, aparecerá esa suscripción.
    3. En el menú de la suscripción, seleccione **Proveedores de recursos** para ver la lista de proveedores.
    4. En la lista de proveedores, desplácese hacia abajo hasta Microsoft.ImportExport. Si el estado es NotRegistered, seleccione **Registrar**.

       ![Registro del proveedor de recursos](./media/backup-azure-backup-server-import-export/register-import-export.png)

* Se crea una ubicación de almacenamiento provisional, que puede ser un recurso compartido de red o cualquier unidad adicional en el equipo, interna o externa, con suficiente espacio en disco para almacenar la copia inicial. Por ejemplo, si intenta realizar una copia de seguridad en un servidor de archivos de 500 GB, asegúrese de que el área de ensayo es de al menos 500 GB. (Se utilizará una cantidad menor gracias a la compresión).
* Con respecto a los discos que se enviarán a Azure, asegúrese de que solo se utilizan unidades de disco duro SSD de 2,5 pulgadas o SATA II/III de 2,5 o 3,5 pulgadas internas. Puede utilizar unidades de disco duro de hasta 10 TB. Vea la [documentación del servicio Azure Import/Export](../storage/common/storage-import-export-requirements.md#supported-hardware) para conocer el conjunto más reciente de unidades de disco que admite el servicio.
* Las unidades de disco SATA deben estar conectadas a un equipo (llamado *equipo de copia*) desde donde se realiza la copia de los datos de copia de seguridad de la ubicación de almacenamiento provisional a SATA. Asegúrese de que BitLocker está habilitado en el equipo de copia.

## <a name="workflow"></a>Flujo de trabajo

La información de esta sección le ayuda a completar el flujo de trabajo de copia de seguridad sin conexión, por lo que los datos se pueden entregar a un centro de datos de Azure y cargarse en Azure Storage. Si tiene alguna pregunta sobre el servicio de importación o cualquier aspecto del proceso, consulte la documentación sobre la [Información general del servicio de importación](../storage/common/storage-import-export-service.md) a la que se ha hecho referencia anteriormente.

## <a name="initiate-offline-backup"></a>Inicio de la copia de seguridad sin conexión

1. Cuando cree un grupo de protección nuevo con protección en línea o agregue protección en línea a un grupo de protección existente, consulte la pantalla siguiente. Para seleccionar el método de replicación en línea inicial, seleccione **Transfer using my own disk** (Transferir con mi propio disco) y elija **Siguiente**.

    ![Pantalla de importación](./media/backup-azure-backup-server-import-export/create-new-protection-group.png)

2. Se abrirá la página de inicio de sesión de Azure. Inicie sesión con su cuenta de usuario de Azure, que tiene un permiso de rol de *propietario* en la suscripción de Azure.

    ![Página de inicio de sesión de Azure](./media/backup-azure-backup-server-import-export/choose-initial-online-replication.png)

3. Proporcione las entradas en la página **Usar su propio disco**.

   ![Entradas para Usar su propio disco](./media/backup-azure-backup-server-import-export/use-your-own-disk.png)

   La descripción de las entradas es la siguiente:

   * **Ubicación de almacenamiento provisional**: la ubicación de almacenamiento temporal en la que se escribe la copia de seguridad inicial. La ubicación de almacenamiento provisional podría estar en un recurso compartido de red o en un equipo local. Si el equipo de copia y el equipo de origen son diferentes, especifique la ruta de acceso completa de red de la ubicación de ensayo.
   * **Cuenta de almacenamiento de Azure Resource Manager**: El nombre de la cuenta de almacenamiento de tipo Resource Manager (de uso general v1 o v2) en cualquier suscripción de Azure.
   * **Contenedor de Azure Storage**: el nombre del contenedor de almacenamiento de blobs de destino de la cuenta de Azure Storage donde se importan los datos de copia de seguridad.
   * **Identificador de suscripción de Azure**: el identificador de la suscripción de Azure en la que se crea la cuenta de Azure Storage.
   * **Nombre de trabajo de Azure Import**: el nombre único mediante el que los servicios Azure Import y Azure Backup realizan el seguimiento de la transferencia de los datos enviados en discos a Azure.

    Guarde la **ubicación de almacenamiento provisional** y el **nombre del trabajo de importación de Azure** proporcionados. Es necesario preparar los discos.

4. Complete el flujo de trabajo para crear o actualizar la protección. Y para iniciar la copia de seguridad sin conexión, haga clic con el botón derecho en el **Grupo de protección** y, luego, elija la opción **Crear punto de recuperación**. A continuación, elija la opción **Protección en línea** .

   ![Crear punto de recuperación](./media/backup-azure-backup-server-import-export/create-recovery-point.png)

5. Supervise el trabajo de creación de réplicas en línea en el panel de supervisión. El trabajo se debe completar correctamente con la advertencia *Esperando a que finalice el trabajo de importación de Azure*.

   ![Finalización del punto de recuperación](./media/backup-azure-backup-server-import-export/complete-recovery-point.png)

6. Una vez finalizada la operación, la ubicación de ensayo estará lista para usarse en la preparación del disco.

## <a name="prepare-sata-drives-and-ship-to-azure"></a>Preparación de unidades SATA y envío a Azure

La utilidad *AzureOfflineBackupDiskPrep* prepara las unidades de disco SATA que se envían al centro de datos de Azure más próximo. Esta utilidad está disponible en el directorio de instalación del agente de Azure Backup (en la ruta de acceso siguiente): `*\Microsoft Azure Recovery Services Agent\Utils\\*`

1. Vaya al directorio y copie el directorio **AzureOfflineBackupDiskPrep** en otro equipo donde estén conectadas las unidades de disco SATA. En el equipo con las unidades de disco SATA conectadas, asegúrese de lo siguiente:

   * El equipo de copia puede acceder a la ubicación de ensayo del flujo de trabajo de propagación sin conexión mediante la misma ruta de acceso de red proporcionada durante el flujo de trabajo de la sección "Inicio de la copia de seguridad sin conexión".
   * BitLocker está habilitado en el equipo de copia.
   * Azure PowerShell 3.7.0 está instalado en el equipo de copia (no es necesario si ejecuta la utilidad AzureOfflineBackupDiskPrep en el servidor de DPM o MABS).
   * Los exploradores compatibles más recientes (Microsoft Edge o Internet Explorer 11) están instalados y JavaScript, habilitado.
   * El equipo de copia puede acceder a Azure Portal. Si es necesario, el equipo de copia puede ser el mismo que el equipo de origen.

     > [!IMPORTANT]
     > Si el equipo de origen es una máquina virtual, es obligatorio utilizar un servidor físico o equipo cliente diferentes como equipo de copia.

1. Abra un símbolo del sistema con privilegios elevados en el equipo de copia con el directorio de la utilidad *AzureOfflineBackupDiskPrep* como directorio actual. Ejecute el siguiente comando:

    ```console
    .\AzureOfflineBackupDiskPrep.exe s:<Staging Location Path>
    ```

    | Parámetro | Descripción |
    | --- | --- |
    | s:&lt;*Ruta de acceso de la ubicación de ensayo*&gt; |Esta entrada obligatoria se usa para proporcionar la ruta de acceso a la ubicación de almacenamiento provisional que escribió en el flujo de trabajo en la sección "Iniciar copia de seguridad sin conexión". |
    | p:&lt;*Ruta de acceso a PublishSettingsFile*&gt; |Esta entrada opcional se usa para proporcionar la ruta de acceso al archivo de configuración de publicación de Azure. |

    Cuando se ejecuta el comando, la utilidad solicita la selección del trabajo de importación de Azure correspondiente a las unidades que deben estar preparadas. Si hay un solo trabajo de importación asociado con la ubicación de ensayo proporcionada, aparece una pantalla como esta.

      ![Consola de preparación de disco](./media/backup-azure-backup-server-import-export/disk-prep-console.png)

1. Escriba la letra de la unidad del disco montado (sin los dos puntos finales) que quiera preparar para la transferencia a Azure.
1. Proporcione la confirmación del formato de la unidad cuando se le solicite.
1. Se le indica que inicie sesión en su suscripción de Azure. Proporcione sus credenciales.

    ![Pantalla de inicio de sesión de Azure](./media/backup-azure-backup-server-import-export/signin-disk-prep.png)

    La herramienta comienza entonces a preparar el disco y a copiar los datos de copia de seguridad. Puede que deba conectar discos adicionales si se lo pide la herramienta, si el disco proporcionado no tiene espacio suficiente para los datos de copia de seguridad. <br/>

    Al terminar la ejecución correcta de la herramienta, el símbolo del sistema proporciona tres informaciones distintas:
    * Uno o varios de los discos que proporcionó están preparados para enviarlos a Azure.
    * Recibe la confirmación de que se creó el trabajo de importación. El trabajo de importación usa el nombre que proporcionó.
    * La herramienta muestra la dirección de envío del centro de datos de Azure.

     ![Preparación de discos de Azure finalizada](./media/backup-azure-backup-server-import-export/console.png)

1. Al final de la ejecución del comando, también verá la opción para actualizar la información de envío.

1. Envíe los discos a la dirección proporcionada por la herramienta y guarde el número de seguimiento como referencia futura.

   > [!IMPORTANT]
   > El número de seguimiento no puede ser el mismo para distintos trabajos de importación de Azure. Asegúrese de que las unidades preparadas con la utilidad en un trabajo de importación de Azure se envían juntas en un único paquete y de que el paquete tenga un número de seguimiento exclusivo. No combine unidades preparadas como parte de diferentes trabajos de importación de Azure en un único paquete.

## <a name="update-shipping-details-on-the-azure-import-job"></a>Actualización de los detalles de envío del trabajo de importación de Azure

El procedimiento siguiente actualiza los detalles del envío del trabajo de importación de Azure. Esta información incluye detalles sobre:

* el nombre del operador que entrega los discos a Azure
* los detalles de envío en caso de devolución de los discos

   1. Inicie sesión en la suscripción de Azure.
   2. En el menú principal, seleccione **Todos los servicios** y, en el cuadro de diálogo Todos los servicios, escriba Importar. Cuando vea la opción **Trabajos de importación o exportación**, selecciónela.
       ![Especificación de la información de envío](./media/backup-azure-backup-server-import-export/search-import-job.png)

       Se abre el menú de la lista de **Trabajos de Import/Export** y aparecen todos los trabajos de Import/Export de la suscripción seleccionada.

   3. Si tiene varias suscripciones, asegúrese de seleccionar la suscripción que se usó para importar los datos de la copia de seguridad. Luego, seleccione el trabajo de importación recién creado para abrir sus detalles.

       ![Revisión de la información de envío](./media/backup-azure-backup-server-import-export/import-job-found.png)

   4. En el menú Configuración del trabajo de importación, seleccione **Administrar la información de envío** y escriba los detalles para el envío en caso de devolución.

       ![Almacenamiento de la información de envío](./media/backup-azure-backup-server-import-export/shipping-info.png)

   5. Cuando tenga el número de seguimiento del transportista de envío, seleccione el banner en la página de información general del trabajo de importación de Azure y escriba estos detalles:

      > [!IMPORTANT]
      > Asegúrese de que la información del transportista y el número de seguimiento se actualicen en dos semanas a contar de la creación del trabajo de importación de Azure. No comprobar esta información dentro de dos semanas puede generar que se elimine el trabajo y que no se procese.

      ![Información general del trabajo](./media/backup-azure-backup-server-import-export/job-overview.png)

      ![Información de seguimiento](./media/backup-azure-backup-server-import-export/tracking-info.png)

### <a name="time-to-process-the-drives"></a>Tiempo para procesar las unidades

La cantidad de tiempo que se tarda en procesar un trabajo de importación de Azure varía. El tiempo depende de factores como el tiempo de envío, el tipo de trabajo, el tipo y el tamaño de los datos que se copian, y el tamaño de los discos proporcionados. El servicio Azure Import/Export no tiene un Acuerdo de Nivel de Servicio. Una vez que se reciben los discos, el servicio trata de finalizar la copia de los datos de copia de seguridad en la cuenta de almacenamiento de Azure en 7 y 10 días. La siguiente sección detalla cómo puede supervisar el estado del trabajo de importación de Azure.

### <a name="monitor-azure-import-job-status"></a>Supervisión del estado del trabajo de importación de Azure

Para supervisar el estado del trabajo de importación en Azure Portal, vaya a la página **Trabajos de Import/Export** y seleccione el trabajo. Para más información sobre el estado de los trabajos de importación, consulte el artículo sobre el [servicio de Import/Export de Storage](../storage/common/storage-import-export-service.md).

### <a name="complete-the-workflow"></a>Finalización del flujo de trabajo

Una vez completado el trabajo de importación, los datos de la copia de seguridad inicial están disponibles en la cuenta de almacenamiento. En la siguiente copia de seguridad programada, Azure Backup copia el contenido de los datos de la cuenta de almacenamiento en el almacén de Recovery Services.

En el momento del trabajo de creación de réplica en línea programado siguiente, Data Protection Manager realiza una copia de seguridad incremental sobre la copia de seguridad inicial.

## <a name="next-steps"></a>Pasos siguientes

* Si tiene dudas del flujo de trabajo del servicio Azure Import/Export, consulte [Uso del servicio Microsoft Azure Import/Export para transferir datos a Blob Storage](../storage/common/storage-import-export-service.md).
