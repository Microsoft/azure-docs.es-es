---
title: Copia de seguridad sin conexión con Azure Data Box
description: Aprenda a usar Azure Data Box para propagar los datos de grandes copias de seguridad iniciales sin conexión desde el agente de MARS hasta un almacén de Recovery Services.
ms.topic: conceptual
ms.date: 1/27/2020
ms.openlocfilehash: 6ad97ee60c3c7debea72357cf7fc8d483a3c1d46
ms.sourcegitcommit: f1b18ade73082f12fa8f62f913255a7d3a7e42d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88761566"
---
# <a name="azure-backup-offline-backup-by-using-azure-data-box"></a>Copia de seguridad sin conexión de Azure Backup con Azure Data Box

Puede usar [Azure Data Box](../databox/data-box-overview.md) para propagar sus grandes copias de seguridad de MARS iniciales sin conexión (sin usar la red) a un almacén de Recovery Services. Este proceso ahorra tiempo y ancho de banda de red que, de otro modo, se consumirían al mover grandes cantidades de datos de copia de seguridad en línea a través de una red de alta latencia. Esta mejora está actualmente en versión preliminar. La copia de seguridad sin conexión basada en Azure Data Box proporciona dos ventajas diferenciadas sobre la [copia de seguridad sin conexión basada en el servicio Azure Import/Export](./backup-azure-backup-import-export.md):

- No es necesario adquirir sus propios discos y conectores compatibles con Azure. Azure Data Box envía los discos asociados a la [SKU de Data Box](https://azure.microsoft.com/services/databox/data/) seleccionada.
- Azure Backup (agente de MARS) puede escribir directamente los datos de copia de seguridad en las SKU admitidas de Azure Data Box. Esta funcionalidad elimina la necesidad de aprovisionar una ubicación de almacenamiento provisional para los datos de copia de seguridad iniciales. Tampoco necesita utilidades para dar formato a los datos de los discos y copiarlos.

## <a name="azure-data-box-with-the-mars-agent"></a>Azure Data Box con el agente de MARS

En este artículo se explica cómo puede usar Azure Data Box para propagar los datos de grandes copias de seguridad iniciales sin conexión desde el agente de MARS hasta un almacén de Recovery Services.

## <a name="supported-platforms"></a>Plataformas compatibles

El proceso para propagar los datos desde el agente de MARS mediante Azure Data Box se admite en las siguientes SKU de Windows.

| **SISTEMA OPERATIVO**                                 | **SKU**                                                      |
| -------------------------------------- | ------------------------------------------------------------ |
| **Estación de trabajo**                        |                                                              |
| Windows 10 64 bits                     | Enterprise, Pro, Home                                       |
| Windows 8.1 64 bits                    | Enterprise, Pro                                             |
| Windows 8 64 bits                      | Enterprise, Pro                                             |
| Windows 7 64 bits                      | Ultimate, Enterprise, Professional, Home Premium, Home Basic, Starter |
| **Server**                             |                                                              |
| Windows Server 2019 64 bits            | Standard, Datacenter, Essentials                            |
| Windows Server 2016 64 bits            | Standard, Datacenter, Essentials                            |
| Windows Server 2012 R2 64 bits         | Standard, Datacenter, Foundation                            |
| Windows Server 2012 64 bits            | Datacenter, Foundation, Standard                            |
| Windows Storage Server 2016 64 bits    | Standard, Workgroup                                         |
| Windows Storage Server 2012 R2 64 bits | Standard, Workgroup y Essential                              |
| Windows Storage Server 2012 64 bits    | Standard, Workgroup                                         |
| Windows Server 2008 R2 SP1 64 bits     | Standard, Enterprise, Datacenter, Foundation                |
| Windows Server 2008 SP2 de 64 bits        | Standard, Enterprise y Datacenter                            |

## <a name="backup-data-size-and-supported-data-box-skus"></a>Tamaño de los datos de copia de seguridad y SKU de Data Box admitidas

| Tamaño de los datos de copia de seguridad (luego de la compresión por MARS)* por servidor | SKU de Azure Data Box admitidas                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <=7,2 TB                                                    | [Azure Data Box Disk](../databox/data-box-disk-overview.md) |
| > 7,2 TB y <= 80 TB**                                      | [Azure Data Box (100 TB)](../databox/data-box-overview.md) |

*Las tasas de compresión típicas varían entre el 10 y el 20 %. <br>
**Póngase en contacto con [AskAzureBackupTeam@microsoft.com](mailto:AskAzureBackupTeam@microsoft.com) si espera tener más de 80 TB de datos de copia de seguridad inicial para un único servidor MARS.

>[!IMPORTANT]
>Los datos de copia de seguridad inicial de un único servidor deben estar contenidos en una única instancia de Azure Data Box o Azure Data Box Disk, y no se pueden compartir entre varios dispositivos con una SKU igual o distinta. Sin embargo, un dispositivo Azure Data Box puede contener copias de seguridad iniciales de varios servidores.

## <a name="prerequisites"></a>Prerrequisitos

### <a name="azure-subscription-and-required-permissions"></a>Suscripción a Azure y permisos necesarios

- El proceso requiere una suscripción a Azure.
- El proceso requiere que el usuario designado para realizar la directiva de copia de seguridad sin conexión sea un propietario de la suscripción a Azure.
- El trabajo de Data Box y el almacén de Recovery Services (al que se deben propagar los datos) deben estar en las mismas suscripciones.
- Se recomienda que la cuenta de almacenamiento de destino asociada con el trabajo de Azure Data Box y el almacén de Recovery Services estén en la misma región. Sin embargo, esto no es necesario.

### <a name="get-azure-powershell-370"></a>Obtener Azure PowerShell 3.7.0

*Este es el requisito previo más importante para el proceso*. Antes de instalar Azure PowerShell, versión 3.7.0, realice las siguientes comprobaciones.

#### <a name="step-1-check-the-powershell-version"></a>Paso 1: Comprobación de la versión de PowerShell

1. Abra Windows PowerShell y ejecute el siguiente comando:

    ```powershell
    Get-Module -ListAvailable AzureRM*
    ```

1. Si la salida muestra una versión superior a 3.7.0, siga con el paso 2. De lo contrario, vaya al paso 3.

#### <a name="step-2-uninstall-the-powershell-version"></a>Paso 2: Desinstalar la versión de PowerShell

Desinstale la versión actual de PowerShell.

1. Quite los módulos dependientes ejecutando el siguiente comando en PowerShell:

    ```powershell
    foreach ($module in (Get-Module -ListAvailable AzureRM*).Name |Get-Unique)  { write-host "Removing Module $module" Uninstall-module $module }
    ```

2. Ejecute el siguiente comando para asegurar la correcta eliminación de todos los módulos dependientes:

    ```powershell
    Get-Module -ListAvailable AzureRM*
    ```

#### <a name="step-3-install-powershell-version-370"></a>Paso 3: Instalar PowerShell versión 3.7.0

Cuando haya verificado que no existe ningún módulo de AzureRM, instale la versión 3.7.0 con uno de los métodos siguientes:

- En GitHub, use [este vínculo](https://github.com/Azure/azure-powershell/releases/tag/v3.7.0-March2017).

También puede:

- Ejecute el comando siguiente en la ventana de PowerShell:

    ```powershell
    Install-Module -Name AzureRM -RequiredVersion 3.7.0
    ```

También se podría haber instalado Azure PowerShell mediante un archivo .msi. Para quitarlo, desinstálelo con la opción **Desinstalar programas** del Panel de control.

### <a name="order-and-receive-the-data-box-device"></a>Pedir y recibir el dispositivo Data Box

El proceso de copia de seguridad sin conexión con MARS y Azure Data Box requiere que los dispositivos Data Box se encuentren en estado Entregado antes de desencadenar la copia de seguridad sin conexión mediante el agente de MARS. Consulte el artículo sobre el [tamaño de los datos de copia de seguridad y la SKU de Data Box admitidas](#backup-data-size-and-supported-data-box-skus) para pedir la SKU más adecuada para sus necesidades. Siga los pasos del [Tutorial: Solicitud de Azure Data Box Disk](../databox/data-box-disk-deploy-ordered.md) para pedir y recibir dispositivos Data Box.

> [!IMPORTANT]
> No seleccione *BlobStorage* como **Tipo de cuenta**. El agente de MARS requiere una cuenta que admita blobs en páginas, lo que no es posible cuando se selecciona *BlobStorage*. Seleccione **Storage V2 (uso general v2)** como **Tipo de cuenta** cuando cree la cuenta de almacenamiento de destino para el trabajo de Azure Data Box.

![Elegir el tipo de cuenta en los detalles de la instancia](./media/offline-backup-azure-data-box/instance-details.png)

## <a name="install-and-set-up-the-mars-agent"></a>Instalación y configuración del agente de MARS

1. Asegúrese de desinstalar toda instalación anterior del agente de MARS.
1. Descargue la última versión del agente de MARS [desde este sitio web](https://aka.ms/azurebackup_agent).
1. Ejecute *MARSAgentInstaller.exe* y siga *solo* los pasos de [instalación y registro del agente](./install-mars-agent.md#install-and-register-the-agent) en el almacén de Recovery Services en el que quiere que se almacenen las copias de seguridad.

   > [!NOTE]
   > El almacén de Recovery Services debe estar en la misma suscripción que el trabajo de Azure Data Box.

   Una vez que registre el agente en el almacén de Recovery Services, siga los pasos de las secciones siguientes.

## <a name="set-up-azure-data-box-devices"></a>Configuración de dispositivos Azure Data Box

En función de la SKU de Azure Data Box que haya pedido, siga los pasos que se describen en las secciones adecuadas. En los pasos se muestra cómo configurar y preparar los dispositivos Data Box para el agente de MARS a fin de identificar y transferir los datos de copia de seguridad inicial.

### <a name="set-up-azure-data-box-disks"></a>Configuración de los discos de Azure Data Box

Si ha pedido uno varios discos de Azure Data Box (hasta 8 TB cada uno), siga los pasos mencionados aquí sobre [Desempaquetado, conexión y desbloqueo de Azure Data Box Disk](../databox/data-box-disk-deploy-set-up.md).

>[!NOTE]
>Es posible que el servidor con el agente de MARS no tenga un puerto USB. En tal caso, puede conectar el disco su instancia de Azure Data Box Disk a otro servidor o cliente y exponer la raíz del dispositivo como recurso compartido de red.

### <a name="set-up-azure-data-box"></a>Configuración de Azure Data Box

Si solicitó una instancia de Azure Data Box (hasta 100 TB), siga los pasos que se indican aquí [para configurar la instancia de Data Box](../databox/data-box-deploy-set-up.md).

#### <a name="mount-your-azure-data-box-instance-as-a-local-system"></a>Montaje de la instancia de Azure Data Box como un sistema local

El agente de MARS funciona en el contexto del sistema local y, por tanto, requiere que se proporcione el mismo nivel de privilegios a la ruta de acceso de montaje donde se conecta Azure Data Box.

Para asegurarse de que puede montar el dispositivo Data Box como un sistema local mediante el protocolo NFS:

1. Habilite el cliente para la característica NFS en el servidor de Windows donde está instalado el agente de MARS. Especifique el origen alternativo *WIM:D:\Sources\Install.wim:4*.
1. Descargue PsExec desde la página [Sysinternals](/sysinternals/downloads/psexec) del servidor con el agente de MARS instalado.
1. Abra un símbolo del sistema con privilegios elevados y ejecute el comando siguiente con el directorio que contiene *PSExec.exe* como directorio actual.

    ```cmd
    psexec.exe  -s  -i  cmd.exe
    ```

   La ventana de comandos que se abre porque el comando anterior se encuentra en el contexto del sistema local. Use esta ventana de comandos para ejecutar los pasos para montar el recurso compartido de blobs en páginas de Azure como unidad de red en el servidor de Windows.
1. Para conectar el servidor con el agente de MARS al dispositivo Data Box a través de NFS, siga los pasos descritos en [Conexión a un dispositivo Data Box](../databox/data-box-deploy-copy-data-via-nfs.md#connect-to-data-box). Ejecute el siguiente comando en el símbolo del sistema del sistema local para montar el recurso compartido de blobs en páginas de Azure.

    ```cmd
    mount -o nolock \\<DeviceIPAddress>\<StorageAccountName_PageBlob X:  
    ```

   Una vez montado el recurso compartido, compruebe si puede acceder a X: desde el servidor. En caso afirmativo, continúe con la sección siguiente de este artículo.

## <a name="transfer-initial-backup-data-to-azure-data-box-devices"></a>Transferencia de datos de copia de seguridad inicial a dispositivos de Azure Data Box

1. Abra la aplicación **Microsoft Azure Backup** en el servidor.
1. En el panel **Acciones**, seleccione **Programar copia de seguridad**.

    ![Selección de Programar copia de seguridad](./media/offline-backup-azure-data-box/schedule-backup.png)

1. Siga los pasos del **Asistente para programar copias de seguridad**.

1. Para agregar elementos, seleccione el botón **Agregar elementos**. Mantenga el tamaño total de los elementos dentro de los [límites de tamaño admitidos por la SKU de Azure Data Box](#backup-data-size-and-supported-data-box-skus) que ha pedido y recibido.

    ![Agregar elementos para la copia de seguridad](./media/offline-backup-azure-data-box/add-items.png)

1. Seleccione la directiva de retención y programación de copia de seguridad adecuada para **Archivos y carpetas** y **Estado del sistema**. El estado del sistema solo es aplicable a los servidores de Windows, no a los clientes de Windows.
1. En la página **Elegir el tipo de copia de seguridad inicial (archivos y carpetas)** del asistente, seleccione la opción **Transfer using Microsoft Azure Data Box disks** (Transferir con discos de Microsoft Azure Data Box) y elija **Siguiente**.

    ![Elección del tipo de copia de seguridad inicial](./media/offline-backup-azure-data-box/initial-backup-type.png)

1. Cuando se le solicite, inicie sesión en Azure con las credenciales de usuario que tienen acceso de propietario a la suscripción de Azure. Si esta operación se realiza correctamente, verá una página parecida a la siguiente.

    ![Creación de recursos y aplicación de los permisos necesarios](./media/offline-backup-azure-data-box/creating-resources.png)

   A continuación, el agente de MARS captura los trabajos de Data Box de la suscripción que se encuentran en estado Entregado.

    ![Captura de trabajos de Data Box para el identificador de la suscripción](./media/offline-backup-azure-data-box/fetching-databox-jobs.png)

1. Seleccione el pedido correcto de Data Box para el que ha desempaquetado, conectado y desbloqueado el disco de Data Box. Seleccione **Next** (Siguiente).

    ![Selección de los pedidos de Data Box](./media/offline-backup-azure-data-box/select-databox-order.png)

1. Seleccione **Detectar el dispositivo** en la página **Detección de dispositivos Data Box**. Esta acción hace que el agente de MARS busque los discos de Azure Data Box conectados localmente y los detecte.

    ![Detección de dispositivos Data Box](./media/offline-backup-azure-data-box/databox-device-detection.png)

    Si ha conectado la instancia de Azure Data Box como un recurso compartido de red (debido a la falta de disponibilidad de puertos USB o porque ha pedido y montado el dispositivo Data Box de 100 TB), la detección genera un error al principio. Se le ofrece la opción de especificar la ruta de acceso de red al dispositivo Data Box.

    ![Especificación de la ruta de acceso de red](./media/offline-backup-azure-data-box/enter-network-path.png)

    >[!IMPORTANT]
    > Proporcione la ruta de acceso de red al directorio raíz del disco de Azure Data Box. Este directorio debe contener un directorio con el nombre *PageBlob*.
    >
    >![Directorio raíz del disco de Azure Data Box](./media/offline-backup-azure-data-box/root-directory.png)
    >
    >Por ejemplo, si la ruta de acceso del disco es `\\mydomain\myserver\disk1\` y *disk1* contiene un directorio denominado *PageBlob*, la ruta de acceso que se debe proporcionar en el asistente del agente de MARS es `\\mydomain\myserver\disk1\`.
    >
    >Si [configura un dispositivo Azure Data Box de 100 TB](#set-up-azure-data-box-devices), escriba `\\<DeviceIPAddress>\<StorageAccountName>_PageBlob` como la ruta de acceso de red al dispositivo.

1. Seleccione **Siguiente** y **Finalizar** en la página siguiente para guardar la directiva de retención y copia de seguridad con la configuración de la copia de seguridad sin conexión mediante Azure Data Box.

   En la página siguiente se confirma que la directiva se ha guardado correctamente.

    ![La directiva se guardó correctamente](./media/offline-backup-azure-data-box/policy-saved.png)

1. Seleccione **Cerrar** en la página anterior.

1. Seleccione **Hacer Copia Ahora** en el panel **Acciones** de la consola del agente de MARS. Seleccione **Copia de seguridad** en la página del asistente.

    ![Asistente de Hacer Copia Ahora](./media/offline-backup-azure-data-box/backup-now.png)

El agente de MARS comenzará a hacer la copia de seguridad de los datos seleccionados en el dispositivo Azure Data Box. Este proceso puede tardar entre varias horas y unos días. La cantidad de tiempo depende del número de archivos y de la velocidad de la conexión entre el servidor con el agente de MARS y el disco Azure Data Box.

Una vez finalizada la copia de seguridad de los datos, verá una página en el agente de MARS parecida a esta.

![Progreso de la copia de seguridad](./media/offline-backup-azure-data-box/backup-progress.png)

## <a name="post-backup-steps"></a>Pasos posteriores a la copia de seguridad

En esta sección se explican los pasos que se deben llevar a cabo una vez que se complete correctamente la copia de seguridad de los datos en Azure Data Box Disk.

- Siga los pasos de este artículo para [enviar el disco de Azure Data Box a Azure](../databox/data-box-disk-deploy-picked-up.md). Si usó un dispositivo Azure Data Box de 100 TB, siga estos pasos para [enviar el dispositivo Azure Data Box a Azure](../databox/data-box-deploy-picked-up.md).

- [Supervise el trabajo de Data Box](../databox/data-box-disk-deploy-upload-verify.md) en Azure Portal Una vez que el trabajo de Azure Data Box ha finalizado, el agente de MARS mueve automáticamente los datos de la cuenta de almacenamiento al almacén de Recovery Services cuando tenga lugar la siguiente copia de seguridad programada. A continuación, marca el trabajo de copia de seguridad como *Trabajo completado* si se ha creado correctamente un punto de recuperación.

    >[!NOTE]
    >El agente de MARS desencadena las copias de seguridad a las horas programadas durante la creación de la directiva. Estos trabajos se marcan como "Waiting for Azure Data Box job to be completed" (Esperando a que se complete el trabajo de Azure Data Box) hasta que finaliza el trabajo.

- Una vez que el agente de MARS cree correctamente un punto de recuperación correspondiente a la copia de seguridad inicial, puede eliminar la cuenta de almacenamiento o el contenido específico asociado con el trabajo de Azure Data Box.

## <a name="troubleshooting"></a>Solución de problemas

El agente de Microsoft Azure Recovery Services (MARS) crea automáticamente una aplicación de Azure Active Directory (Azure AD) en su inquilino. Esta aplicación requiere un certificado para la autenticación, que se crea y se carga al configurar la directiva de propagación sin conexión. Se usará Azure PowerShell para crear y cargar el certificado en la aplicación de Azure AD.

### <a name="problem"></a>Problema

Al configurar la copia de seguridad sin conexión, es posible que se produzca un problema debido a un error en el cmdlet de Azure PowerShell. Es posible que no pueda agregar varios certificados a la misma aplicación de Azure AD creada por el agente de MAB. Este problema le afectará si ha configurado una directiva de propagación sin conexión para el mismo servidor o para otro.

### <a name="verify-if-the-problem-is-caused-by-this-specific-root-cause"></a>Comprobación de si el problema se debe a esta causa principal específica

Para ver si el problema es el mismo que el descrito anteriormente, realice uno de los pasos siguientes.

#### <a name="step-1-of-verification"></a>Paso 1 de la verificación

Compruebe si aparece el siguiente mensaje de error en la consola de MAB cuando configuró la copia de seguridad sin conexión.

![No se puede crear la directiva Copia de seguridad sin conexión para la cuenta actual de Azure](./media/offline-backup-azure-data-box/unable-to-create-policy.png)

#### <a name="step-2-of-verification"></a>Paso 2 de la verificación

1. Abra la carpeta **Temp** en la ruta de acceso de instalación. La ruta de acceso predeterminada de la carpeta Temp es *C:\Program Files\Microsoft Azure Recovery Services Agent\Temp*. Busque el archivo *CBUICurr* y ábralo.

1. En el archivo *CBUICurr*, desplácese hasta la última línea y compruebe si el problema es el mismo que el de este mensaje de error: `Unable to create an Azure AD application credential in customer's account. Exception: Update to existing credential with KeyId <some guid> is not allowed`.

### <a name="workaround"></a>Solución alternativa

Como solución alternativa para resolver este problema, siga los pasos a continuación y vuelva a intentar la configuración de la directiva.

#### <a name="step-1-of-workaround"></a>Paso 1 de la solución alternativa

Inicie sesión en la instancia de PowerShell que aparece en la interfaz de usuario de MAB con una cuenta diferente con acceso de administrador en la suscripción en la que se creará el trabajo de importación o exportación.

#### <a name="step-2-of-workaround"></a>Paso 2 de la solución alternativa

Si no hay ningún otro servidor configurado para la propagación sin conexión y ningún otro servidor depende de la aplicación `AzureOfflineBackup_<Azure User Id>`, elimine esta aplicación. Seleccione **Azure Portal** > **Azure Active Directory** > **Registros de aplicaciones**.

>[!NOTE]
> Compruebe que la aplicación `AzureOfflineBackup_<Azure User Id>` no tenga configurada ninguna otra propagación sin conexión y que tampoco haya otro servidor que dependa de esta aplicación. Vaya a **Configuración** > **Claves** en la sección **Claves públicas**. No debe haber ninguna otra clave pública agregada. Como referencia, vea la siguiente captura de pantalla.
>
>![Claves públicas](./media/offline-backup-azure-data-box/public-keys.png)

#### <a name="step-3"></a>Paso 3

Desde el servidor donde intenta configurar la copia de seguridad sin conexión, realice las siguientes acciones.

1. Vaya a la pestaña **Manage computer certificate application** (Administrar aplicación de certificado de equipo)  > **Personal** y busque el certificado con el nombre `CB_AzureADCertforOfflineSeeding_<ResourceId>`.

2. Seleccione el certificado, haga clic con el botón derecho en **Todas las tareas** y, después, en **Exportar**, sin clave privada y en formato .cer.

3. Vaya a la aplicación de copia de seguridad sin conexión de Azure mencionada en el paso 2. Seleccione **Configuración** > **Claves** > **Cargar clave pública**. Cargue el certificado exportado en el paso anterior.

    ![Carga de la clave pública](./media/offline-backup-azure-data-box/upload-public-key.png)

4. En el servidor, abra el registro y escriba **regedit** en la ventana de ejecución.

5. Vaya a la entrada del Registro *Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider*. Haga clic con el botón derecho en **CloudBackupProvider** y agregue un nuevo valor de cadena con el nombre `AzureADAppCertThumbprint_<Azure User Id>`.

    >[!NOTE]
    > Para obtener el identificador de usuario de Azure, realice una de estas acciones:
    >
    >- Desde la instancia de PowerShell conectada a Azure, ejecute el comando `Get-AzureRmADUser -UserPrincipalName "Account Holder's email as defined in the portal"`.
    > - Vaya a la ruta de acceso del registro `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\DbgSettings\OnlineBackup` con el nombre *CurrentUserId*.

6. Haga clic con el botón derecho en la cadena agregada en el paso anterior y seleccione **Modificar**. En el valor, proporcione la huella digital del certificado exportado en el paso 2. Seleccione **Aceptar**.

7. Para obtener el valor de la huella digital, haga doble clic en el certificado. Seleccione la pestaña **Detalles** y desplácese hacia abajo hasta la huella digital. Seleccione **Huella digital** y copie el valor.

    ![Campo Huella digital del certificado](./media/offline-backup-azure-data-box/thumbprint-field.png)

## <a name="questions"></a>Preguntas

Si tiene alguna pregunta o aclaraciones sobre los problemas a los que se enfrenta, póngase en contacto con [AskAzureBackupTeam@microsoft.com](mailto:AskAzureBackupTeam@microsoft.com).
