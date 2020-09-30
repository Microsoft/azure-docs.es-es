---
title: Solución de problemas de Azure Backup Server
description: Solucionar problemas de instalación, registro de Azure Backup Server y copia de seguridad y restauración de las cargas de trabajo de la aplicación.
ms.reviewer: srinathv
ms.topic: troubleshooting
ms.date: 07/05/2019
ms.openlocfilehash: 70431ee42566d1cbba5ed239b9da55c2ff7a2afe
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88999230"
---
# <a name="troubleshoot-azure-backup-server"></a>Solución de problemas de Azure Backup Server

Use la información de las tablas siguientes para solucionar los errores detectados al usar Azure Backup Server.

## <a name="basic-troubleshooting"></a>Pasos básicos para solucionar problemas

Antes de empezar a solucionar problemas relacionados con Microsoft Azure Backup Server (MABS), se recomienda realizar las siguientes comprobaciones:

- [Asegúrese de que el agente de Microsoft Azure Recovery Services (MARS) esté actualizado](https://go.microsoft.com/fwlink/?linkid=229525&clcid=0x409).
- [Asegúrese de que haya conectividad de red entre el agente de MARS y Azure](./backup-azure-mars-troubleshoot.md#the-microsoft-azure-recovery-service-agent-was-unable-to-connect-to-microsoft-azure-backup)
- Asegúrese de que se ejecuta Microsoft Azure Recovery Services (en la consola del servicio). Si es necesario, reinicie y vuelva a intentar la operación.
- [Asegúrese de que haya disponible entre un 5 % y un 10 % en la ubicación de la carpeta temporal](./backup-azure-file-folder-backup-faq.md#whats-the-minimum-size-requirement-for-the-cache-folder)
- Si se produce un error en el registro, asegúrese de que el servidor en el que está intentando instalar Azure Backup Server no esté registrado en otro almacén.
- Si se produce un error de instalación automática, compruebe si el agente de DPM ya está presente. En caso afirmativo, desinstale al agente y vuelva a intentar la instalación.
- [Asegúrese de que ningún otro proceso o software antivirus interfiera con Azure Backup](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-another-process-or-antivirus-software-interfering-with-azure-backup).<br>
- Asegúrese de que el servicio del Agente SQL se esté ejecutando y esté establecido en el modo automático en el servidor MABS.<br>

## <a name="configure-antivirus-for-mabs-server"></a>Configuración del antivirus para el servidor MABS

MABS es compatible con la mayoría de los productos de software antivirus más utilizados. Se recomienda aplicar los pasos siguientes para evitar conflictos:

1. **Deshabilitar la supervisión en tiempo real**: deshabilite la supervisión en tiempo real mediante el software antivirus para lo siguiente:
    - `C:\Program Files<MABS Installation path>\XSD`
    - `C:\Program Files<MABS Installation path>\Temp`
    - Letra de unidad del volumen de Modern Backup Storage
    - Registros de réplicas y de transferencias: Para ello, deshabilite la supervisión en tiempo real de **dpmra.exe**, que se encuentra en la carpeta `Program Files\Microsoft Azure Backup Server\DPM\DPM\bin`. La supervisión en tiempo real degrada el rendimiento porque el software antivirus examina todas las réplicas cada vez que MABS se sincroniza con el servidor protegido y examina todos los archivos afectados cada vez que MABS aplica cambios en las réplicas.
    - Consola de administrador: Para evitar que afecte al rendimiento, deshabilite la supervisión en tiempo real del proceso **csc.exe**. El proceso **csc.exe** es el compilador C\#, y la supervisión en tiempo real puede degradar el rendimiento porque el software antivirus examina los archivos que el proceso **csc.exe** emite al generar mensajes XML. El proceso **csc.exe** se encuentra en las siguientes rutas de acceso:
        - `\Windows\Microsoft.net\Framework\v2.0.50727\csc.exe`
        - `\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe`
    - Para el agente de MARS instalado en el servidor de MABS, se recomienda excluir los siguientes archivos y ubicaciones:
        - `C:\Program Files\Microsoft Azure Backup Server\DPM\MARS\Microsoft Azure Recovery Services Agent\bin\cbengine.exe` como proceso
        - `C:\Program Files\Microsoft Azure Backup Server\DPM\MARS\Microsoft Azure Recovery Services Agent\folder`
        - Ubicación temporal (si no está usando la ubicación estándar)
2. **Deshabilitar la supervisión en tiempo real en el servidor protegido**: Deshabilite la supervisión en tiempo real de **dpmra.exe**, que se encuentra en la carpeta `C:\Program Files\Microsoft Data Protection Manager\DPM\bin`, en el servidor protegido.
3. **Configurar el software antivirus para eliminar los archivos infectados en los servidores protegidos y el servidor de MABS**: A fin de evitar daños en las réplicas y los puntos de recuperación, configure el software antivirus para que elimine los archivos infectados en lugar de eliminarlos o enviarlos a cuarentena automáticamente. La limpieza automática y la puesta en cuarentena pueden hacer que el software antivirus modifique los archivos y que realice cambios que MABS no pueda detectar.

Debe ejecutar una sincronización manual con una regularidad. Compruebe el trabajo cada vez que el software antivirus elimine un archivo de la réplica, incluso si la réplica esté marcada como incoherente.

### <a name="mabs-installation-folders"></a>Carpetas de instalación de MABS

Las carpetas de instalación predeterminadas para DPM son las siguientes:

- `C:\Program Files\Microsoft Azure Backup Server\DPM\DPM`

También puede ejecutar el siguiente comando para buscar la ruta de acceso de la carpeta de instalación:

```cmd
Reg query "HKLM\SOFTWARE\Microsoft\Microsoft Data Protection Manager\Setup"
```

## <a name="invalid-vault-credentials-provided"></a>Se han proporcionado credenciales de almacén no válidas

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Registrar en un almacén | Se han proporcionado credenciales de almacén no válidas. El archivo está dañado o no tiene asociadas las credenciales más recientes para el servicio de recuperación. | Acción recomendada: <br> <ul><li> Descargue el archivo de credenciales más reciente del almacén y vuelva a intentarlo. <br>O BIEN</li> <li> Si la acción anterior no funciona, intente descargar las credenciales en un directorio local diferente o cree un nuevo almacén. <br>O BIEN</li> <li> Intente actualizar la configuración de fecha y hora como se describe en [este artículo](./backup-azure-mars-troubleshoot.md#invalid-vault-credentials-provided). <br>O BIEN</li> <li> Compruebe si c:\windows\temp tiene más de 65000 archivos. Mueva los archivos obsoletos a otra ubicación o elimine los elementos de la carpeta Temp. <br>O BIEN</li> <li> Compruebe el estado de los certificados. <br> a. Abra **Administrar certificados de equipo** (en el Panel de control). <br> b. Expanda el nodo **Personal** y su nodo secundario **Certificados**.<br> c.  Quite el certificado **Windows Azure Tools**. <br> d. Vuelva a intentar el registro en el cliente de Azure Backup. <br> O BIEN </li> <li> Compruebe si hay alguna directiva de grupo implementada. </li></ul> |

## <a name="replica-is-inconsistent"></a>La réplica es incoherente

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Copia de seguridad | La réplica es incoherente | Compruebe que está activada la opción para habilitar esa coherencia automática en el Asistente para nuevo grupo de protección. Para más información sobre las opciones de replicación y las comprobaciones de coherencia, consulte [este artículo](/system-center/dpm/create-dpm-protection-groups?view=sc-dpm-2019).<br> <ol><li> En el caso de una copia de seguridad de BMR o estado del sistema, compruebe si Windows Server Backup está instalado en el servidor protegido.</li><li> Compruebe si hay problemas relacionados con el espacio en el grupo de almacenamiento de DPM en DPM o Microsoft Azure Backup Server, y asigne almacenamiento según sea necesario.</li><li> Compruebe el estado del Servicio de instantáneas de volumen en el servidor protegido. Si se encuentra en estado deshabilitado, configúrelo para que se inicie manualmente. Inicie el servicio en el servidor. A continuación, vuelva a la consola de DPM o Microsoft Azure Backup Server y comience la sincronización con el trabajo de comprobación de coherencia.</li></ol>|

## <a name="online-recovery-point-creation-failed"></a>No se pudo crear el punto de recuperación en línea

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Copia de seguridad | No se pudo crear el punto de recuperación en línea | **Mensaje de error**: Windows Azure Backup Agent no pudo crear una instantánea del volumen seleccionado. <br> **Solución alternativa**: Intente aumentar el espacio en la réplica y el volumen del punto de recuperación.<br> <br> **Mensaje de error**: Windows Azure Backup Agent no se puede conectar al servicio OBEngine. <br> **Solución alternativa**: Compruebe que el servicio OBEngine figura en la lista de servicios en ejecución en el equipo. Si el servicio OBEngine no se está ejecutando, use el comando "net start OBEngine" para iniciarlo. <br> <br> **Mensaje de error**: No se ha establecido la frase de contraseña de cifrado para este servidor. Configure una frase de contraseña. <br> **Solución alternativa**: Intente configurar una frase de contraseña. Si hay algún problema, realice estos pasos: <br> <ol><li>Compruebe que existe la ubicación temporal. La ubicación mencionada en el Registro **HKEY_LOCAL_MACHINE\Software\Microsoft\Windows Azure Backup\Config** debe existir con el nombre **ScratchLocation**.</li><li> Si no existe la ubicación temporal, intente volver a registrar con la frase de contraseña de cifrado anterior. *Siempre que configure una frase de contraseña de cifrado, guárdela en una ubicación segura.*</li><ol>|

## <a name="the-original-and-external-dpm-servers-must-be-registered-to-the-same-vault"></a>Los servidores DPM originales y externos deben estar registrados en el mismo almacén

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Restauración | **Código de error**: CBPServerRegisteredVaultDontMatchWithCurrent/Error en las credenciales del almacén: 100110 <br/> <br/>**Mensaje de error**: Los servidores DPM originales y externos deben estar registrados en el mismo almacén | **Causa**: Este problema se produce al intentar restaurar archivos en un servidor alternativo desde el servidor original mediante la opción de recuperación del DPM externo, y si el servidor que se va a recuperar y el servidor original desde donde se realiza la copia de seguridad de los datos no están asociados al mismo almacén de Recovery Services.<br/> <br/>**Solución alternativa**: Para resolver este problema, asegúrese de que tanto el servidor original como el servidor alternativo están registrados en el mismo almacén.|

## <a name="online-recovery-point-creation-jobs-for-vmware-vm-fail"></a>Hubo un error en los trabajos de creación de puntos de recuperación en línea para la máquina virtual de VMware.

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Copia de seguridad | Hubo un error en los trabajos de creación de puntos de recuperación en línea para la máquina virtual de VMware. DPM detectó un error de VMware al intentar obtener información de ChangeTracking. ErrorCode - FileFaultFault (ID 33621) |  <ol><li> Restablezca el CTK en VMware, para las máquinas virtuales afectadas.</li> <li>Compruebe que los discos independientes no están en VMware.</li> <li>Detenga la protección de las máquinas virtuales afectadas y vuelva a protegerlas con el botón **Actualizar**. </li><li>Ejecute CC para las máquinas virtuales afectadas.</li></ol>|

## <a name="the-agent-operation-failed-because-of-a-communication-error-with-the-dpm-agent-coordinator-service-on-the-server"></a>Error en la operación del agente debido a un error de comunicación con el servicio Coordinador de agentes DPM en el servidor

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Insertar agentes en servidores protegidos | Error en la operación del agente debido a un error de comunicación con el servicio Coordinador de agentes DPM en \<ServerName>. | **Si no funciona la acción recomendada que se muestra en el producto, siga los pasos siguientes**: <ul><li> Si va a conectar un equipo desde un dominio que no es de confianza, siga [estos pasos](/system-center/dpm/back-up-machines-in-workgroups-and-untrusted-domains?view=sc-dpm-2019). <br> O BIEN </li><li> Si va a conectar un equipo desde un dominio de confianza, solucione el problema mediante los pasos descritos en [este blog](https://techcommunity.microsoft.com/t5/system-center-blog/data-protection-manager-agent-network-troubleshooting/ba-p/344726). <br>O BIEN</li><li> Pruebe a deshabilitar el antivirus como un paso para la solución de problemas. Si resuelve el problema, modifique la configuración del antivirus como se sugiere en [este artículo](/system-center/dpm/run-antivirus-server?view=sc-dpm-2019).</li></ul> |

## <a name="setup-could-not-update-registry-metadata"></a>El programa de instalación no pudo actualizar los metadatos del Registro

| Operación | Detalles del error | Solución alternativa |
|-----------|---------------|------------|
|Instalación | El programa de instalación no pudo actualizar los metadatos del Registro. Este error de actualización podría provocar un uso excesivo de almacenamiento. Para evitar esto, actualice la entrada del Registro del recorte de ReFS. | Ajustar la clave del Registro **SYSTEM\CurrentControlSet\Control\FileSystem\RefsEnableInlineTrim**. Establezca el valor Dword en 1. |
|Instalación | El programa de instalación no pudo actualizar los metadatos del Registro. Este error de actualización podría provocar un uso excesivo de almacenamiento. Para evitar esto, actualice la entrada del Registro de SnapOptimization del volumen. | Cree la clave del Registro **SOFTWARE\Microsoft Data Protection Manager\Configuration\VolSnapOptimization\WriteIds** con un valor de cadena vacía. |

## <a name="registration-and-agent-related-issues"></a>Problemas relacionados con el registro y el agente

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Insertar agentes en servidores protegidos | Las credenciales especificadas para el servidor no son válidas. | **Si no funciona la acción recomendada que se muestra en el producto, siga los pasos siguientes**: <br> Pruebe a instalar de forma manual el agente de protección en el servidor de producción, como se especifica en [este artículo](/system-center/dpm/deploy-dpm-protection-agent?view=sc-dpm-2019).|
| El agente de Azure Backup no pudo conectar con el servicio Azure Backup (Id.: 100050) | El agente de Azure Backup no pudo conectar con el servicio Azure Backup. | **Si no funciona la acción recomendada que se muestra en el producto, siga los pasos siguientes**: <br>1. Desde el símbolo del sistema con privilegios elevados, ejecute el siguiente comando: **psexec -i -s "c:\Program Files\Internet Explorer\iexplore.exe**. Se abrirá la ventana de Internet Explorer. <br/> 2. Vaya a **Herramientas** > **Opciones de Internet** > **Conexiones** > **Configuración de LAN**. <br/> 3. Cambie la configuración para usar un servidor proxy. A continuación, proporcione los detalles del servidor proxy.<br/> 4. Si la máquina tiene limitado el acceso a Internet, asegúrese de que su configuración de firewall está establecida para permitir estas direcciones [URL](install-mars-agent.md#verify-internet-access) e [IP](install-mars-agent.md#verify-internet-access).|
| Error en la instalación del agente de Azure Backup | Error en la instalación de Microsoft Azure Recovery Services. Se revirtieron todos los cambios que la instalación de Microsoft Azure Recovery Services realizó en el sistema. (Id.: 4024) | Instalación manual del agente de Azure.

## <a name="configuring-protection-group"></a>Configurar grupo de protección

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Configurar grupos de protección | DPM no pudo enumerar el componente de la aplicación en el equipo protegido (nombre del equipo protegido). | Seleccione **Actualizar** en la pantalla de interfaz de usuario para configurar el grupo de protección en el nivel de origen de datos o el componente correspondiente. |
| Configurar grupos de protección | No se puede configurar la protección. | Si el servidor protegido es un servidor SQL Server, compruebe si se han proporcionado permisos de rol sysadmin para la cuenta del sistema (NTAuthority\System) en el equipo protegido, tal como se describe en [este artículo](/system-center/dpm/back-up-sql-server?view=sc-dpm-2019).
| Configurar grupos de protección | No hay suficiente espacio disponible en el grupo de almacenamiento para este grupo de protección. | Los discos que se agregan al grupo de almacenamiento [no deben contener ninguna partición](/system-center/dpm/create-dpm-protection-groups?view=sc-dpm-2019). Elimine los volúmenes existentes en los discos. A continuación, agréguelos al grupo de almacenamiento.|
| Cambio de directiva |No se ha podido modificar la directiva de copia de seguridad. Error: No se pudo realizar la operación actual debido a un error de servicio interno [0x29834]. Vuelva a intentar la operación más tarde. Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Microsoft. | **Causa:**<br/>Este error se produce bajo tres condiciones: cuando está habilitada la configuración de seguridad, cuando se intenta reducir el período de retención por debajo de los valores mínimos especificados anteriormente y cuando se utiliza una versión no admitida. Las versiones no admitidas son las anteriores a la versión 2.0.9052 de Microsoft Azure Backup Server y a la actualización 1 de Azure Backup Server. <br/>**Acción recomendada:**<br/> Para continuar con las actualizaciones relacionadas con la directiva, establezca el período de retención por encima del período mínimo especificado. (El período de retención mínimo es de siete días para diario, cuatro semanas para semanal, tres semanas para mensual o un año para anual.) <br><br>Si lo desea, otro enfoque preferido sería actualizar el agente de copia de seguridad y Azure Backup Server para aprovechar todas las actualizaciones de seguridad. |

## <a name="backup"></a>Copia de seguridad

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Copia de seguridad | Se produjo un error inesperado mientras se ejecutaba el trabajo. El dispositivo no está listo. | **Si no funciona la acción recomendada que se muestra en el producto, siga los pasos siguientes:** <br> <ul><li>Configure el espacio de almacenamiento de instantáneas como ilimitado para los elementos del grupo de protección y, después, ejecute la comprobación de coherencia.<br></li> O BIEN <li>Intente eliminar el grupo de protección existente y la creación de varios grupos nuevos. Cada nuevo grupo de protección debe tener un elemento individual.</li></ul> |
| Copia de seguridad | Si está haciendo una copia de seguridad solo del Estado del sistema, compruebe si hay suficiente espacio disponible en el equipo protegido para almacenarla. | <ol><li>Compruebe que Windows Server Backup está instalado en la máquina protegida.</li><li>Compruebe que hay suficiente espacio en el equipo protegido para el estado del sistema. La manera más fácil de comprobar esto consiste en ir al equipo protegido, abrir Windows Server Backup, hacer clic en las diferentes selecciones y, después, seleccionar BMR. A continuación, la interfaz de usuario indica cuánto espacio es necesario. Abra **WSB** > **Copia de seguridad local** > **Programación de copia de seguridad** > **Seleccionar configuración de copia de seguridad** > **Servidor completo** (se muestra el tamaño). Use este tamaño para la comprobación.</li></ol>
| Copia de seguridad | Error en la copia de seguridad de BMR | Si el tamaño de la reconstrucción completa es grande, mueva algunos archivos de aplicación a la unidad de sistema operativo y vuelva a intentarlo. |
| Copia de seguridad | La opción para volver a proteger una máquina virtual de VMware en una instancia nueva de Microsoft Azure Backup Server no se muestra como disponible para agregarla. | Las propiedades de VMware señalan a una instancia anterior y retirada de Microsoft Azure Backup Server. Para solucionar este problema:<br><ol><li>En VCenter (equivalente a SC-VMM), vaya a la pestaña **Resumen** y, después, a **Atributos personalizados**.</li>  <li>Elimine el nombre antiguo de Microsoft Azure Backup Server del valor de **DPMServer**.</li>  <li>Vuelva a la nueva instancia de Microsoft Azure Backup Server y modifique el PG.  Después de usar el botón **Actualizar**, se presentará la máquina virtual con una casilla disponible para agregar a la protección.</li></ol> |
| Copia de seguridad | Error al acceder a carpetas o archivos compartidos | Intente modificar la configuración del antivirus como se sugiere en el artículo [Ejecución de software antivirus en el servidor DPM](/system-center/dpm/run-antivirus-server?view=sc-dpm-2019).|

## <a name="change-passphrase"></a>Cambiar la frase de contraseña

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Cambiar la frase de contraseña |El PIN de seguridad escrito no es correcto. Proporcione el PIN de seguridad correcto para completar esta operación. |**Causa:**<br/> Este error se produce al especificar un PIN de seguridad que no es válido o que ha expirado mientras se realiza una operación crítica (por ejemplo, modificar una frase de contraseña). <br/>**Acción recomendada:**<br/> Para completar la operación, debe escribir un PIN de seguridad válido. Para obtener el PIN, inicie sesión en Azure Portal y vaya al almacén de Recovery Services. A continuación, vaya a **Configuración** > **Propiedades** > **Generar PIN de seguridad**. Use este código PIN para cambiar la frase de contraseña. |
| Cambiar la frase de contraseña |No se pudo realizar la operación. Id.: 120002 |**Causa:**<br/>Este error se produce cuando se habilita la configuración de seguridad, o al intentar cambiar la frase de contraseña usando una versión no compatible.<br/>**Acción recomendada:**<br/> Para cambiar la frase de contraseña, primero debe actualizar el agente de copia de seguridad a la versión mínima, que es 2.0.9052. También debe actualizar Azure Backup Server a la actualización 1 como mínimo y, después, escribir un PIN de seguridad válido. Para obtener el PIN, inicie sesión en Azure Portal y vaya al almacén de Recovery Services. A continuación, vaya a **Configuración** > **Propiedades** > **Generar PIN de seguridad**. Use este código PIN para cambiar la frase de contraseña. |

## <a name="configure-email-notifications"></a>Configuración de notificaciones de correo electrónico

| Operación | Detalles del error | Solución alternativa |
| --- | --- | --- |
| Configurar notificaciones por correo electrónico con una cuenta de Office 365 |Id. del error: 2013| **Causa:**<br> Intentar usar la cuenta de Office 365 <br>**Acción recomendada:**<ol><li> En primer lugar, hay que asegurarse de que la opción para permitir la retransmisión anónima en un conector de recepción del servidor DPM esté establecida en Exchange. Para más información sobre cómo configurarla, consulte [Permitir la retransmisión anónima en un conector de recepción](/exchange/mail-flow/connectors/allow-anonymous-relay?view=exchserver-2019).</li> <li> Si no puede usar una retransmisión SMTP interna y necesita configurarla con un servidor Office 365, puede configurar IIS para que actúe como retransmisión. Configure el servidor DPM para [retransmitir el protocolo SMTP a Office 365 con IIS](/exchange/mail-flow/test-smtp-with-telnet?view=exchserver-2019).<br><br>  Asegúrese de usar el formato user\@domain.com y *no* domain\user.<br><br><li>Apunte a DMP para usar el nombre del servidor local como servidor SMTP, puerto 587. A continuación, apunte al correo electrónico del usuario del que deberían provenir los correos electrónicos.<li> El nombre de usuario y la contraseña en la página de configuración SMTP de DPM deben ser para una cuenta de dominio en la que el dominio DPM está habilitado. </li><br> Si va a modificar la dirección del servidor SMTP, realice el cambio en la nueva configuración, cierre el cuadro de configuración y, a continuación, vuelva a abrirlo para asegurarse de que refleja el nuevo valor.  No basta con cambiar y probar para que la nueva configuración entre en vigor, de modo que se recomienda realizar esta prueba.<br><br>En cualquier momento durante este proceso, puede borrar esta configuración cerrando la consola de DPM y modificando las siguientes claves del Registro: **HKLM\SOFTWARE\Microsoft\Microsoft Data Protection Manager\Notification\ <br/> Elimine las claves SMTPPassword y SMTPUserName**. Puede agregarlas de nuevo en la interfaz de usuario cuando la inicie otra vez.

## <a name="common-issues"></a>Problemas comunes

En esta sección se tratan los errores que se pueden producir al usar Azure Backup Server.

### <a name="cbpsourcesnapshotfailedreplicamissingorinvalid"></a>CBPSourceSnapshotFailedReplicaMissingOrInvalid

Mensaje de error | Acción recomendada |
-- | --
No se pudo realizar la copia de seguridad porque la réplica de copia de seguridad de disco no se encuentra o no es válida. | Para resolver este problema, compruebe lo siguiente y vuelva a intentar la operación: <br/> 1. Cree un punto de recuperación de disco.<br/> 2. Ejecute una comprobación de coherencia en el origen de datos. <br/> 3. Detenga la protección del origen de datos y vuelva a configurar la protección de este origen de datos.

### <a name="cbpsourcesnapshotfailedreplicametadatainvalid"></a>CBPSourceSnapshotFailedReplicaMetadataInvalid

Mensaje de error | Acción recomendada |
-- | --
Se produjo un error en la instantánea de volumen del origen debido a que los metadatos de la réplica no son válidos. | Cree un punto de recuperación de disco de este origen de datos y vuelva a intentar la copia de seguridad en línea.

### <a name="cbpsourcesnapshotfailedreplicainconsistent"></a>CBPSourceSnapshotFailedReplicaInconsistent

Mensaje de error | Acción recomendada |
-- | --
Se produjo un error en la instantánea de volumen del origen debido a una réplica de origen de datos incoherente. | Ejecute una comprobación de coherencia en este origen de datos y vuelva a intentarlo.

### <a name="cbpsourcesnapshotfailedreplicacloningissue"></a>CBPSourceSnapshotFailedReplicaCloningIssue

Mensaje de error | Acción recomendada |
-- | --
No se pudo realizar la copia de seguridad porque no se pudo clonar la réplica de copia de seguridad de disco.| Asegúrese de que todos los archivos de réplica de copia de seguridad de disco (.vhdx) anteriores estén desmontados y no haya ninguna copia de seguridad de disco a disco en curso durante las copias de seguridad en línea.
