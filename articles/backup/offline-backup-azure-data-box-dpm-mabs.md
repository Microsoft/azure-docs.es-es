---
title: Copia de seguridad sin conexión con Azure Data Box para DPM y MABS
description: Puede usar Azure Data Box para propagar datos de copia de seguridad inicial sin conexión desde DPM y MABS.
ms.topic: conceptual
ms.date: 08/12/2020
ms.openlocfilehash: 3755a4cc77455b37ae1545247992a3c8c4f25653
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89177693"
---
# <a name="offline-seeding-using-azure-data-box-for-dpm-and-mabs-preview"></a>Propagación sin conexión mediante Azure Data Box para DPM y MABS (versión preliminar)

> [!NOTE]
> Esta característica se aplica a Data Protection Manager (DPM) 2019 UR2 y versiones posteriores.<br><br>
> Esta característica está actualmente en versión preliminar para Microsoft Azure Backup Server (MABS). Si le interesa usar Azure Data Box para la propagación sin conexión con MABS, puede ponerse en contacto con nosotros en [systemcenterfeedback@microsoft.com](mailto:systemcenterfeedback@microsoft.com).

En este artículo se explica cómo usar Azure Data Box para propagar datos de copia de seguridad inicial sin conexión desde DPM y MABS a un almacén de Azure Recovery Services.

Puede usar [Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-overview) para propagar copias de seguridad iniciales de gran tamaño de DPM o MABS sin conexión (sin usar la red) a un almacén de Recovery Services. Este proceso ahorra tiempo y ancho de banda de red que, de otro modo, se consumirían al mover grandes cantidades de datos de copia de seguridad en línea a través de una red de alta latencia. Esta funcionalidad actualmente está en su versión preliminar.

La copia de seguridad sin conexión basada en Azure Data Box proporciona dos ventajas diferenciadas sobre la [copia de seguridad sin conexión basada en el servicio Azure Import/Export](backup-azure-backup-server-import-export.md):

- No es necesario aportar discos ni conectores propios compatibles con Azure. Azure Data Box envía los discos asociados a la [SKU de Data Box](https://azure.microsoft.com/services/databox/data/) seleccionada.

- Azure Backup (agente de MARS) puede escribir directamente los datos de copia de seguridad en las SKU admitidas de Azure Data Box. Esta funcionalidad elimina la necesidad de aprovisionar una ubicación de almacenamiento provisional para los datos de copia de seguridad iniciales. Tampoco necesita utilidades para dar formato a los datos de los discos y copiarlos.

## <a name="supported-platforms"></a>Plataformas compatibles

Se admiten las siguientes plataformas:

- Windows Server 2019 64 bits (Standard, Datacenter y Essentials)
- Windows Server 2016 64 bits (Standard, Datacenter y Essentials)

## <a name="backup-data-size-and-supported-data-box-skus"></a>Tamaño de los datos de copia de seguridad y SKU de Data Box admitidas

Se admiten las siguientes SKU de Data Box:

| Tamaño de los datos de copia de seguridad (tras compresión por MARS)\* por servidor | SKU de Azure Data Box admitidas |
| --- | --- |
| \<= 7,2 TB | [Azure Data Box Disk](https://docs.microsoft.com/azure/databox/data-box-disk-overview) |
| > 7,2 TB y <= 80 TB\*\* | [Azure Data Box (100 TB)](https://docs.microsoft.com/azure/databox/data-box-overview) |

\*Las tasas de compresión típicas varían entre el 10 y el 20 % <br>
\*\*Póngase en contacto con [SystemCenterFeedback@microsoft.com](mailto:SystemCenterFeedback@microsoft.com) si espera tener más de 80 TB de datos de copia de seguridad inicial de un único origen de datos.

> [!IMPORTANT]
> Los datos de copia de seguridad inicial de un único origen de datos deben estar incluidos en un único dispositivo Azure Data Box o Azure Data Box Disk, y no se pueden compartir entre varios dispositivos con la misma SKU u otras. Por otro lado, un dispositivo Azure Data Box puede contener copias de seguridad iniciales de varios orígenes de datos.

## <a name="before-you-begin"></a>Antes de empezar

El agente de MARS que se ejecuta en DPM o MABS debe actualizarse a la [versión más reciente](https://aka.ms/azurebackup_agent) (2.0.9171.0 o posterior).

Asegúrese de lo siguiente:

### <a name="azure-subscription-and-required-permissions"></a>Suscripción a Azure y permisos necesarios

- Una suscripción válida a Azure.
- El usuario designado para realizar la directiva de copia de seguridad sin conexión debe ser un propietario de la suscripción de Azure.
- El trabajo de Data Box y el almacén de Recovery Services al que se deben propagar los datos deben estar disponibles en las mismas suscripciones.
    > [!NOTE]
    > Se recomienda que la cuenta de almacenamiento de destino y el almacén de Recovery Services estén en la misma región, aunque no es obligatorio.

### <a name="order-and-receive-the-data-box-device"></a>Pedir y recibir el dispositivo Data Box

Asegúrese de que los dispositivos Data Box necesarios estén en estado *Entregado* antes de desencadenar la copia de seguridad sin conexión. Vea [Tamaño de los datos de copia de seguridad y SKU de Data Box admitidas](#backup-data-size-and-supported-data-box-skus) para pedir la SKU más adecuada a sus requisitos. Siga los pasos de [este artículo](https://docs.microsoft.com/azure/databox/data-box-disk-deploy-ordered) para pedir y recibir los dispositivos Data Box.

> [!IMPORTANT]
> No seleccione *BlobStorage* como **Tipo de cuenta**. El servidor de DPM o MABS necesita una cuenta que admita blobs en páginas, algo que no se admite cuando *BlobStorage* está seleccionado. Seleccione **Storage V2 (general purpose v2)** (Almacenamiento V2 (fin general v2)) como **Tipo de cuenta** cuando cree la cuenta de almacenamiento de destino para el trabajo de Azure Data Box.

![Instalación de Azure Data Box](./media/offline-backup-azure-data-box-dpm-mabs/setup-azure-databox.png)

## <a name="setup-azure-data-box-devices"></a>Instalación de dispositivos Azure Data Box

Cuando reciba el dispositivo Azure Data Box, según la SKU de Azure Data Box que haya pedido, siga los pasos de las secciones correspondientes siguientes para instalar y preparar los dispositivos Data Box para que el servidor de DPM o MABS identifique y transfiera los datos de copia de seguridad inicial.

### <a name="setup-azure-data-box-disk"></a>Instalación de Azure Data Box Disk

Si ha pedido uno varios dispositivos Azure Data Box Disk (hasta 8 TB cada uno), siga los pasos indicados [aquí](https://docs.microsoft.com/azure/databox/data-box-disk-deploy-set-up) para desempaquetar, conectar y desbloquear el dispositivo Data Box Disk.

> [!NOTE]
> Es posible que el servidor de DPM o MABS no tenga puerto USB. En tal caso, puede conectar Azure Data Box Disk a otro servidor o cliente y exponer la raíz del dispositivo como recurso compartido de red.

## <a name="setup-azure-data-box"></a>Instalación de Azure Data Box

Si ha pedido un dispositivo Azure Data Box (hasta 100 TB), siga los pasos indicados [aquí](https://docs.microsoft.com/azure/databox/data-box-deploy-set-up) para instalarlo.

### <a name="mount-your-azure-data-box-as-local-system"></a>Montaje de Azure Data Box como sistema local

El servidor de DPM o MABS funciona en el contexto del sistema y, por tanto, requiere que se proporcione el mismo nivel de privilegios a la ruta de acceso de montaje donde se conecta Azure Data Box. Siga los pasos siguientes para asegurarse de que puede montar el dispositivo Data Box como sistema local mediante el protocolo NFS.

1. Habilite el cliente de la característica NFS en el servidor de DPM o MABS.
Especifique un origen alternativo: *WIM: D: \Sources\Install.wim: 4*
2. Descargue **PSExec** desde [https://download.sysinternals.com/files/PSTools.zip](https://download.sysinternals.com/files/PSTools.zip) en el servidor de DPM o MABS.
3. Abra un símbolo del sistema con privilegios elevados y ejecute el comando siguiente con el directorio que contiene *PSExec.exe* como directorio actual.

   ```cmd
   psexec.exe  -s  -i  cmd.exe
   ```

4. La ventana de comandos que se abre como consecuencia del comando anterior se encuentra en el contexto del sistema local. Use esta ventana de comandos para ejecutar los pasos para montar el recurso compartido de blobs en páginas de Azure como unidad de red en el servidor de Windows.
5. Siga [estos pasos](https://docs.microsoft.com/azure/databox/data-box-deploy-copy-data-via-nfs#connect-to-data-box) para conectar el servidor de DPM o MABS al dispositivo Data Box a través de NFS, y ejecute el siguiente comando en el símbolo del sistema local para montar el recurso compartido de blobs en páginas de Azure:

    ```cmd
    mount -o nolock \\<DeviceIPAddres>\<StorageAccountName_PageBlob X:
    ```

6. Una vez montado, compruebe si puede acceder a X: desde el servidor. En caso afirmativo, continúe con la sección siguiente de este artículo.

## <a name="transfer-initial-backup-data-to-azure-data-box-devices"></a>Transferencia de datos de copia de seguridad inicial a dispositivos de Azure Data Box

1. En el servidor de DPM o MABS, siga los pasos para [crear un nuevo grupo de protección](https://docs.microsoft.com/system-center/dpm/create-dpm-protection-groups?view=sc-dpm-2019). Si va a agregar una protección en línea al grupo de protección existente, haga clic con el botón derecho en el grupo de protección existente, seleccione **Agregar protección en línea** y comience desde el **paso 8**.
2. En la página **Seleccionar miembros del grupo**, especifique los equipos y orígenes cuya copia de seguridad quiere realizar.
3. En la página **Seleccionar método de protección de datos**, especifique cómo quiere controlar la copia de seguridad a corto y largo plazo. Asegúrese de seleccionar **Deseo protección en línea**.

   ![Creación de nuevo grupo de protección](./media/offline-backup-azure-data-box-dpm-mabs/create-new-protection-group.png)

4. En la página **Especificar objetivos a corto plazo**, especifique cómo quiere realizar la copia de seguridad en el almacenamiento a corto plazo en disco.
5. En la página **Revisar asignación de disco**, revise el espacio en disco del bloque de almacenamiento asignado para el grupo de protección.
6. En la página **Seleccionar método de creación de réplicas**, seleccione **Automáticamente a través de la red**.
7. En la página **Elegir las opciones de la comprobación de coherencia**, seleccione cómo quiere automatizar las comprobaciones de coherencia.
8. En la página **Especificar datos de protección en línea**, seleccione el miembro para el que quiere habilitar la protección en línea.

    ![Especificar datos de protección en línea](./media/offline-backup-azure-data-box-dpm-mabs/specify-online-protection-data.png)

9. En la página **Especificar la programación de copia de seguridad en línea**, especifique con qué frecuencia se deberían realizar copias de seguridad incrementales en Azure.
10. En la página **Especificar la directiva de retención en línea**, especifique cómo se conservan en Azure los puntos de recuperación creados a partir de copias de seguridad diarias, semanales, mensuales y anuales.
11. En la pantalla **Elegir replicación en línea** del asistente, seleccione la opción **Transferir mediante discos propiedad de Microsoft** y luego **Siguiente**.

    ![Selección de replicación en línea inicial](./media/offline-backup-azure-data-box-dpm-mabs/choose-initial-online-replication.png)

    >[!NOTE]
    > La opción para seleccionar **Transferir mediante discos propiedad de Microsoft** no está disponible en MABS v3, ya que la característica está en versión preliminar. Póngase en contacto con nosotros en [systemcenterfeedback@microsoft.com](mailto:systemcenterfeedback@microsoft.com) si quiere usar esta característica en MABS v3.

12. Inicie sesión en Azure cuando se le pida con unas credenciales de usuario con acceso de propietario en la suscripción de Azure. Tras un inicio de sesión correcto, se muestra la siguiente pantalla:

    ![Tras un inicio de sesión correcto](./media/offline-backup-azure-data-box-dpm-mabs/after-successful-login.png)

     Entonces, el servidor de DPM o MABS captura los trabajos de Data Box de la suscripción que se encuentran en estado *Entregado*.

     > [!NOTE]
     > El primer inicio de sesión tarda más de lo habitual. El módulo de Azure PowerShell se instala en segundo plano y, además, se registra la aplicación Azure AD.
     >
     >  - Se instalan los siguientes módulos de PowerShell:<br>
          - AzureRM.Profile     *5.8.3*<br>
          - AzureRM.Resources   *6.7.3*<br>
          - AzureRM.Storage     *5.2.0*<br>
          - Azure.Storage       *4.6.1*<br>
     >  - La aplicación Azure AD se registra como *AzureOfflineBackup_\<object GUID of the user>* .

13. Seleccione el pedido correcto de Data Box para el que se ha desempaquetado, conectado y desbloqueado el disco de Data Box. Seleccione **Next** (Siguiente).

    ![Selección de Data Box](./media/offline-backup-azure-data-box-dpm-mabs/select-databox.png)

14. En la pantalla **Detect the DataBox** (Detectar Data Box), escriba la ruta de acceso del dispositivo Data Box y seleccione **Detectar el dispositivo**.

    ![Especificación de ruta de acceso de red](./media/offline-backup-azure-data-box-dpm-mabs/enter-network-path.png)

    > [!IMPORTANT]
    > Proporcione la ruta de acceso de red al directorio raíz del disco de Azure Data Box. Este directorio debe contener un directorio con el nombre *PageBlob*, como se muestra a continuación:
    >
    > ![Unidad USB](./media/offline-backup-azure-data-box-dpm-mabs/usb-drive.png)
    >
    > Por ejemplo, si la ruta de acceso del disco es `\\mydomain\myserver\disk1\` y *disk1* contiene un directorio denominado *PageBlob*, la ruta de acceso que se va a proporcionar en el asistente del servidor de DPM o MABS es `\\mydomain\myserver\disk1\`.
    > Si [instala un dispositivo Azure Data Box de 100 TB](https://docs.microsoft.com/azure/backup/offline-backup-azure-data-box#setup-azure-data-box), proporcione la siguiente como ruta de acceso de red al dispositivo `\\<DeviceIPAddress>\<StorageAccountName>_PageBlob`.

15. Seleccione **Next** (Siguiente). En la página **Resumen**, revise la configuración y seleccione **Crear grupo**.

    ![Detección de Data Box](./media/offline-backup-azure-data-box-dpm-mabs/detect-databox.png)

    En la pantalla siguiente se confirma que el grupo de protección se ha creado correctamente.

    ![Grupo de protección creado](./media/offline-backup-azure-data-box-dpm-mabs/protection-group-created.png)

16. Seleccione **Cerrar** en la pantalla anterior.

    Con esto, se produce la replicación inicial de los datos en el disco de DPM o MABS. Cuando termina la protección, el estado del grupo se muestra como **Correcto** en la página **Protección**.

17. Para iniciar la copia de seguridad sin conexión en el dispositivo Azure Data Box, haga clic con el botón derecho en el **Grupo de protección** y seleccione la opción **Crear punto de recuperación**. A continuación, elija la opción **Protección en línea** .

    ![Crear punto de recuperación](./media/offline-backup-azure-data-box-dpm-mabs/create-recovery-point.png)

    ![Punto de recuperación](./media/offline-backup-azure-data-box-dpm-mabs/recovery-point.png)

   El servidor de DPM o MABS comienza a hacer la copia de seguridad de los datos seleccionados en el dispositivo Azure Data Box. Esto puede tardar entre varias horas y unos días, según el tamaño de los datos y la velocidad de conexión entre el servidor de DPM o MABS y el dispositivo Azure Data Box Disk.

   Puede supervisar el estado del trabajo en el panel **Supervisión**. Una vez que finaliza la copia de seguridad de los datos, se ve una pantalla parecida a la siguiente:

   ![Consola de administrador](./media/offline-backup-azure-data-box-dpm-mabs/administrator-console.png)

## <a name="post-backup-steps"></a>Pasos posteriores a la copia de seguridad

Siga estos pasos una vez que la copia de seguridad de los datos en Azure Data Box Disk sea correcta.

- Siga los pasos de [este artículo](https://docs.microsoft.com/azure/databox/data-box-disk-deploy-picked-up) para enviar Azure Data Box Disk a Azure. Si ha usado un dispositivo Azure Data Box de 100 TB, siga [estos pasos](https://docs.microsoft.com/azure/databox/data-box-deploy-picked-up) para enviar Azure Data Box a Azure.
- [Supervise el trabajo de Data Box](https://docs.microsoft.com/azure/databox/data-box-disk-deploy-upload-verify) en Azure Portal Una vez que el trabajo de Azure Data Box está *Completo*, el servidor de DPM o MABS mueve automáticamente los datos de la cuenta de almacenamiento al almacén de Recovery Services en el momento de la siguiente copia de seguridad programada. Luego marca el trabajo de copia de seguridad como *Trabajo completado* si se ha creado correctamente un punto de recuperación.

  > [!NOTE]
  > El servidor de DPM o MABS desencadena las copias de seguridad a las horas programadas durante la creación del grupo de protección. Pero estos trabajos se marcan como *Waiting for Azure Data Box job to be completed* (Esperando a que se complete el trabajo de Azure Data Box) hasta que se completa el trabajo.

- Una vez que el servidor de DPM o MABS crea correctamente un punto de recuperación correspondiente a la copia de seguridad inicial, se puede eliminar la cuenta de almacenamiento (o contenido específico) asociada al trabajo de Azure Data Box.

## <a name="troubleshooting"></a>Solución de problemas

El agente de Microsoft Azure Backup (MAB) en el servidor de DPM crea automáticamente una aplicación de Azure AD en el inquilino. Esta aplicación requiere un certificado con fines de autenticación, que se creará y se cargará al configurar la directiva de propagación sin conexión.

Usamos Azure PowerShell para crear y cargar el certificado en la aplicación de Azure AD.

### <a name="issue"></a>Incidencia

En el momento de configurar la copia de seguridad sin conexión, debido a un defecto de código conocido en el cmdlet de Azure PowerShell, no se pueden agregar varios certificados a la misma aplicación de Azure AD creada por el agente de MAB. Esto le afecta si ha configurado una directiva de propagación sin conexión para el mismo servidor o para otro.

### <a name="verify-if-the-issue-is-caused-by-this-specific-root-cause"></a>Verificación de que el problema se debe a esta causa principal concreta

Para asegurarse de que el error se debe a la [Incidencia](#issue) anterior, siga uno de estos pasos:

#### <a name="step-1"></a>Paso 1

Compruebe si aparece el siguiente mensaje de error en la consola de DPM o MABS en el momento de configurar la copia de seguridad sin conexión:

![Agente de Azure Recovery Services](./media/offline-backup-azure-data-box-dpm-mabs/azure-recovery-services-agent.png)

#### <a name="step-2"></a>Paso 2

1. Abra la carpeta **Temp** en la ruta de instalación (la ruta de acceso predeterminada de la carpeta Temp es *C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Temp*. Busque el archivo *CBUICurr* y ábralo.
2. En el archivo *CBUICurr*, vaya a la última línea y compruebe si el error se debe a "No se puede crear una credencial de aplicación Azure AD en la cuenta del cliente. Excepción: no se permite actualizar a una credencial existente con KeyId \<some guid>".

### <a name="workaround"></a>Solución alternativa

Para solucionar este problema, siga estos pasos y vuelva a intentar la configuración de la directiva.

1. Inicie sesión en la página de inicio de sesión de Azure que aparece en la interfaz de usuario del servidor de DPM o MABS con otra cuenta con acceso de administrador en la suscripción en la que se va a crear el trabajo de importación y exportación.
2. Si ningún otro servidor tiene configurada la propagación sin conexión y ningún otro depende de la aplicación `AzureOfflineBackup_<Azure User Id>`, elimine esta aplicación de **Azure Portal> Azure Active Directory > Registros de aplicaciones**.

   > [!NOTE]
   > Compruebe si la aplicación `AzureOfflineBackup_<Azure User Id>` no tiene configurada ninguna otra propagación sin conexión y que tampoco haya otro servidor que dependa de esta aplicación. Vaya a **Configuración > Claves** en la sección Claves públicas. No debe haber ninguna otra **clave pública** agregada. Como referencia, observe la siguiente captura de pantalla:
   >
   > ![Claves públicas](./media/offline-backup-azure-data-box-dpm-mabs/public-keys.png)

#### <a name="step-3"></a>Paso 3

En el servidor de DPM o MABS donde intenta configurar la copia de seguridad sin conexión, realice las siguientes acciones:

1. Abra la pestaña **Manage computer certificate application** > **Personal** (Administrar aplicación de certificados del equipo > Personal) y busque el certificado denominado `CB_AzureADCertforOfflineSeeding_<ResourceId>`.
2. Seleccione el certificado anterior, haga clic con el botón derecho en **Todas las tareas** y **Exportar** sin clave privada y en formato .cer.
3. Vaya a la aplicación de copia de seguridad sin conexión de Azure mencionada en el **punto 2**. En **Configuración** > **Claves** > **Cargar clave pública**, cargue el certificado exportado en el paso anterior.

   ![Carga de claves públicas](./media/offline-backup-azure-data-box-dpm-mabs/upload-public-keys.png)

4. En el servidor, escriba **regedit** en la ventana **Ejecutar** para abrir el Registro.
5. Vaya a la entrada del Registro *Computer\HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider*. Haga clic con el botón derecho en **CloudBackupProvider** y agregue un nuevo valor de cadena con el nombre `AzureADAppCertThumbprint_<Azure User Id>`.

    >[!NOTE]
    > Para obtener el identificador de usuario de Azure, realice una de estas acciones:
    >
    >- Desde la instancia de PowerShell conectada a Azure, ejecute el comando `Get-AzureRmADUser -UserPrincipalName "Account Holder's email as defined in the portal"`.
    > - Navegue hasta la ruta de acceso del registro `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\DbgSettings\OnlineBackup` con el nombre *CurrentUserId*.

6. Haga clic con el botón derecho en la cadena agregada en el paso anterior y seleccione **Modificar**. En el valor, proporcione la huella digital del certificado exportado en el **punto 2** y seleccione **Aceptar**.
7. Para obtener el valor de la huella digital, haga doble clic en el certificado, seleccione **Detalles** y vaya hacia abajo hasta que vea el campo Huella digital. Seleccione **Huella digital** y copie el valor.

   ![Valor de la huella digital](./media/offline-backup-azure-data-box-dpm-mabs/certificate.png)

## <a name="next-steps"></a>Pasos siguientes

- [Propagación sin conexión mediante un disco propio (con el servicio Azure Import/Export)](backup-azure-backup-server-import-export.md)
