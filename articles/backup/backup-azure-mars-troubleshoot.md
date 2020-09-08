---
title: Solución de problemas del agente de Azure Backup
description: En este artículo se explica cómo solucionar problemas de instalación y registro del agente de Azure Backup.
ms.topic: troubleshooting
ms.date: 07/15/2019
ms.openlocfilehash: 64996737a18add8ca1bee25e32929f1d602f9018
ms.sourcegitcommit: e2b36c60a53904ecf3b99b3f1d36be00fbde24fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88763514"
---
# <a name="troubleshoot-the-microsoft-azure-recovery-services-mars-agent"></a>Solución de problemas del agente de Microsoft Azure Recovery Services (MARS)

En este artículo se explica cómo resolver errores que pueden aparecer durante operaciones de configuración, registro, copia de seguridad y restauración.

## <a name="basic-troubleshooting"></a>Pasos básicos para solucionar problemas

Se recomienda confirmar lo siguiente antes de empezar a solucionar problemas del agente de Microsoft Azure Recovery Services (MARS):

- [Asegúrese de que el agente de MARS está actualizado](https://go.microsoft.com/fwlink/?linkid=229525&clcid=0x409).
- [Asegúrese de que hay conectividad de red entre el agente de MARS y Azure](#the-microsoft-azure-recovery-service-agent-was-unable-to-connect-to-microsoft-azure-backup).
- Asegúrese de que MARS se está ejecutando (en la consola de servicio). Si lo precisa, reinicie y vuelva a intentar la operación.
- [Asegúrese de que hay disponible entre un 5 % y un 10 % de espacio de volumen en la ubicación de la carpeta temporal](./backup-azure-file-folder-backup-faq.md#whats-the-minimum-size-requirement-for-the-cache-folder).
- [Compruebe si otro proceso o software antivirus interfiere con Azure Backup](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-another-process-or-antivirus-software-interfering-with-azure-backup).
- Si el trabajo de copia de seguridad se completó con advertencias, vea [Trabajos de copia de seguridad completados con advertencias](#backup-jobs-completed-with-warning).
- Si la copia de seguridad programada genera errores, pero no la copia de seguridad manual, vea [Las copias de seguridad no se ejecutan según la programación](#backups-dont-run-according-to-schedule).
- Asegúrese de que el sistema operativo tiene las actualizaciones más recientes.
- [Asegúrese de excluir de la copia de seguridad las unidades no compatibles y los archivos con atributos no compatibles](backup-support-matrix-mars-agent.md#supported-drives-or-volumes-for-backup).
- Asegúrese de que el reloj del sistema protegido está configurado con la zona horaria correcta.
- [Asegúrese de que .NET Framework 4.5.2 o posterior está instalado en el servidor](https://www.microsoft.com/download/details.aspx?id=30653).
- Si está intentando volver a registrar el servidor en un almacén:
  - Asegúrese de que el agente no está instalado en el servidor y de que se ha eliminado de Portal.
  - Utilice la misma frase de contraseña que se usó inicialmente para registrar el servidor.
- Antes de iniciar una copia de seguridad sin conexión, asegúrese de que Azure PowerShell 3.7.0 está instalado tanto en el equipo de origen como en el equipo donde se realizará la copia.
- Si el agente de Backup se ejecuta en una máquina virtual de Azure, vea [este artículo](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-backup-agent-running-on-an-azure-virtual-machine).

## <a name="invalid-vault-credentials-provided"></a>Se han proporcionado credenciales de almacén no válidas

**Mensaje de error**: Se han proporcionado credenciales de almacén no válidas. El archivo está dañado o no tiene asociadas las credenciales más recientes para el servicio de recuperación. (Id.: 34513)

| Causa | Acciones recomendadas |
| ---     | ---    |
| **Las credenciales de almacén no son válidas** <br/> <br/> Los archivos de credenciales de almacén podrían estar dañados o es posible que hayan expirado (por ejemplo, podrían haberse descargado más de 48 horas antes del momento del registro).| [Descargue nuevas credenciales](backup-azure-file-folder-backup-faq.md#where-can-i-download-the-vault-credentials-file) del almacén de Recovery Services en Azure Portal. Luego, realice estos pasos, según corresponda: <ul><li> Si ya ha instalado y registrado MARS, abra la consola MMC del agente de Microsoft Azure Backup. A continuación, seleccione **Registrar servidor** en el panel **Acciones** para completar el registro con las nuevas credenciales. <br/> <li> Si se produce un error en la nueva instalación, intente realizarla otra vez con las nuevas credenciales.</ul> **Nota**: Si se han descargado varios archivos de credenciales de almacén, solo el último archivo será válido las próximas 48 horas. Le recomendamos que descargue un nuevo archivo de credenciales de almacén.
| **El servidor proxy o el firewall están bloqueando el registro** <br/>or <br/>**No hay conectividad de Internet** <br/><br/> Si la máquina o servidor proxy tiene una conectividad de Internet limitada y el acceso a las direcciones URL necesarias no está garantizado, se producirá un error en el registro.| Siga estos pasos:<br/> <ul><li> Acuda al equipo de TI para asegurarse de que el sistema tiene conectividad de Internet.<li> Si no tiene un servidor proxy, asegúrese de que la opción de proxy no está seleccionada al registrar el agente. [Compruebe la configuración de proxy](#verifying-proxy-settings-for-windows).<li> Si tiene un servidor proxy o un firewall, acuda al equipo de red para asegurarse de que estas direcciones URL y direcciones IP tienen acceso:<br/> <br> **URLs**<br> `www.msftncsi.com` <br> .Microsoft.com <br> .WindowsAzure.com <br> .microsoftonline.com <br> .windows.net <br>**Direcciones IP**<br>  20.190.128.0/18 <br>  40.126.0.0/18 <br/></ul></ul>Intente realizar el registro de nuevo después de completar los pasos de solución de problemas anteriores.<br></br> Si la conexión se realiza a través de Azure ExpressRoute, asegúrese de que la configuración esté definida como se describe en [Compatibilidad con Azure ExpressRoute](backup-support-matrix-mars-agent.md#azure-expressroute-support).
| **El software antivirus está bloqueando el registro** | Si tiene un software antivirus instalado en el servidor, agregue al examen antivirus las reglas de exclusión necesarias correspondientes a los siguientes archivos y carpetas: <br/><ul> <li> CBengine.exe <li> CSC.exe<li> La carpeta temporal. La ubicación predeterminada es: C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Scratch. <li> La carpeta Bin en C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Bin.

### <a name="additional-recommendations"></a>Recomendaciones adicionales

- Vaya a C:/Windows/Temp para comprobar si hay más de 60 000 o 65 000 archivos con la extensión .tmp. Si es el caso, elimínelos.
- Asegúrese de que la fecha y la hora de la máquina coinciden con la zona horaria local.
- Asegúrese de que [estos sitios](install-mars-agent.md#verify-internet-access) están incluidos como sitios de confianza en Internet Explorer.

### <a name="verifying-proxy-settings-for-windows"></a>Comprobación de la configuración de proxy en Windows

1. Descargue PsExec de la página [Sysinternals](/sysinternals/downloads/psexec).
1. Ejecute `psexec -i -s "c:\Program Files\Internet Explorer\iexplore.exe"` en un símbolo del sistema con privilegios elevados.

   Este comando abrirá Internet Explorer.
1. Vaya a **Herramientas** > **Opciones de Internet** > **Conexiones** > **Configuración de LAN**.
1. Compruebe la configuración del proxy de la cuenta del sistema.
1. Si no se ha configurado ningún proxy, pero se han proporcionado detalles del proxy, quite esos detalles.
1. Si hay un proxy configurado y los detalles del proxy son incorrectos, asegúrese de que los datos de **IP del servidor proxy** y **Puerto** son correctos.
1. Cierre Internet Explorer.

## <a name="unable-to-download-vault-credential-file"></a>No se puede descargar el archivo de credenciales de almacén

| Error   | Acciones recomendadas |
| ---     | ---    |
|No se pudo descargar el archivo de credenciales de almacén. (Id.: 403) | <ul><li> Pruebe a descargar las credenciales del almacén con otro explorador o siga estos pasos: <ul><li> Inicie Internet Explorer. Seleccione F12. </li><li> Vaya a la pestaña **Red** y borre la memoria caché y las cookies. </li> <li> Actualice la página.<br></li></ul> <li> Compruebe si la suscripción está deshabilitada o ha expirado.<br></li> <li> Compruebe si hay alguna regla de firewall que esté bloqueando la descarga. <br></li> <li> Asegúrese de que no ha agotado el límite en el almacén (50 máquinas por almacén).<br></li>  <li> Asegúrese de que el usuario tiene los permisos de Azure Backup necesarios para descargar credenciales de almacén y registrar un servidor con el almacén. Vea [Uso del control de acceso basado en roles para administrar puntos de recuperación de Azure Backup](backup-rbac-rs-vault.md).</li></ul> |

## <a name="the-microsoft-azure-recovery-service-agent-was-unable-to-connect-to-microsoft-azure-backup"></a>El Agente de Microsoft Azure Recovery Service no se pudo conectar a Microsoft Azure Backup

| Error  | Causa posible | Acciones recomendadas |
| ---     | ---     | ---    |
| <br /><ul><li>El Agente de servicios Microsoft Azure Recovery no se pudo conectar al servicio Microsoft Azure Backup. (Id.: 100050) Compruebe la configuración de red y asegúrese de que tiene conexión a Internet.<li>(407) Se requiere autenticación del proxy. |El proxy bloquea la conexión. |  <ul><li>En Internet Explorer, vaya a **Herramientas** > **Opciones de Internet** > **Seguridad** > **Internet**. Seleccione **Nivel personalizado** y desplácese hasta que vea la sección de **descarga de archivos**. Seleccione **Habilitar**.<p>También es posible que deba agregar [direcciones URL y direcciones IP](install-mars-agent.md#verify-internet-access) a los sitios de confianza en Internet Explorer.<li>Cambie la configuración para usar un servidor proxy. A continuación, proporcione los detalles del servidor proxy.<li> Si la máquina tiene limitado el acceso a Internet, asegúrese de que su configuración de firewall está establecida para permitir estas [direcciones URL y direcciones IP](install-mars-agent.md#verify-internet-access). <li>Si tiene un software antivirus instalado en el servidor, excluya estos archivos del examen: <ul><li>CBEngine.exe (en lugar de dpmra.exe).<li>CSC.exe (relacionado con .NET Framework). Hay un archivo CSC.exe por cada versión de .NET Framework instalada en el servidor. Excluya todos los archivos CSC.exe de todas las versiones de .NET Framework en el servidor afectado. <li>La ubicación de la memoria caché o la carpeta temporal. <br>La ubicación predeterminada de la carpeta temporal o la ruta de acceso a la memoria caché es C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Scratch.<li>La carpeta Bin en C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Bin.

## <a name="the-specified-vault-credential-file-cannot-be-used-as-it-is-not-downloaded-from-the-vault-associated-with-this-server"></a>El archivo de credenciales del almacén especificado no se puede usar, ya que no se descargó desde el almacén asociado a este servidor.

| Error  | Causa posible | Acciones recomendadas |
| ---     | ---     | ---    |
| El archivo de credenciales del almacén especificado no se puede usar, ya que no se descargó desde el almacén asociado a este servidor. (Id.: 100110) Use las credenciales del almacén adecuado. | El archivo de credenciales del almacén es de un almacén diferente del que ya está registrado en este servidor. | Asegúrese de que tanto la máquina de destino como la máquina de origen están registradas en el mismo almacén de Recovery Services. Si el servidor de destino ya se ha registrado en otro almacén, use la opción **Registrar servidor** para registrarlo en el almacén correcto.  

## <a name="backup-jobs-completed-with-warning"></a>Trabajos de copia de seguridad completados con advertencias

- Cuando el agente de MARS recorre en iteración los archivos y las carpetas durante la copia de seguridad, puede encontrar varias condiciones que pueden provocar que la copia de seguridad se marque como completada con advertencias. Durante estas condiciones, un trabajo se muestra como completado con advertencias. No se trata de ningún error, pero significa que no se pudo realizar la copia de seguridad de, al menos, un archivo. Por lo tanto, el trabajo omitió ese archivo, pero realizó una copia de seguridad de los demás en el origen de datos.

  ![Trabajo de copia de seguridad completada con advertencias](./media/backup-azure-mars-troubleshoot/backup-completed-with-warning.png)

- Entre las condiciones que pueden provocar que las copias de seguridad omitan archivos se encuentran las siguientes:
  - Atributos de archivo no admitidos (por ejemplo: en una carpeta de OneDrive, un flujo comprimido, puntos de análisis, etc.). Para ver la lista completa, consulte la [matriz de compatibilidad](./backup-support-matrix-mars-agent.md#supported-file-types-for-backup).
  - Un problema del sistema de archivos.
  - Otro proceso interfiere (por ejemplo: el software antivirus que contiene identificadores en archivos puede impedir que el agente de MARS tenga acceso a los archivos).
  - Archivos bloqueados por una aplicación.  

- El servicio de copia de seguridad marcará estos archivos como erróneos en el archivo de registro, con la siguiente convención de nomenclatura: *LastBackupFailedFilesxxxx.txt* en la carpeta *C:\Archivos de programa\Microsoft Azure Recovery Service Agent\temp*.
- Para resolver el problema, revise el archivo de registro para comprender la naturaleza del problema:

  | Código de error             | Motivos                                             | Recomendaciones                                              |
  | ---------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
  | 0x80070570             | El archivo o directorio está dañado o es ilegible. | Ejecute **chkdsk** en el volumen de origen.                             |
  | 0x80070002, 0x80070003 | El sistema no encuentra el archivo especificado.         | [Asegúrese de que la carpeta temporal no está llena.](./backup-azure-file-folder-backup-faq.md#manage-the-backup-cache-folder)  <br><br>  Compruebe si existe el volumen donde se ha configurado el espacio de desecho (no se ha eliminado).  <br><br>   [Asegúrese de que el agente de MARS se ha excluido del antivirus instalado en la máquina.](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-another-process-or-antivirus-software-interfering-with-azure-backup)  |
  | 0x80070005             | Acceso denegado                                    | [Compruebe si el antivirus u otro software de terceros está bloqueando el acceso](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-another-process-or-antivirus-software-interfering-with-azure-backup).     |
  | 0x8007018b             | Se ha denegado el acceso al archivo de nube.                | Archivos de OneDrive, archivos Git o cualquier otro archivo que puede tener el estado sin conexión en la máquina |

- Puede seguir las instrucciones de la sección [Adición de reglas de exclusión a la directiva existente](./backup-azure-manage-mars.md#add-exclusion-rules-to-existing-policy) para excluir los archivos no admitidos, que faltan o que se han eliminado de la directiva de copia de seguridad para asegurarse de que las copias de seguridad se realizan correctamente.

- Evite eliminar carpetas protegidas con los mismos nombres en la carpeta de nivel superior. Tampoco vuelva a crearlas. Si lo hace, la copia de seguridad podría completase con advertencias con el error *Se detectó una incoherencia crítica, por lo que no se pueden replicar los cambios.*  Si tiene que eliminar carpetas y volver a crearlas, hágalo en las subcarpetas de la carpeta de nivel superior protegida.

## <a name="failed-to-set-the-encryption-key-for-secure-backups"></a>No se pudo establecer la clave de cifrado para proteger las copias de seguridad

| Error | Causas posibles | Acciones recomendadas |
| ---     | ---     | ---    |
| <br />No se pudo establecer la clave de cifrado para proteger las copias de seguridad. La activación no se realizó completamente, pero la frase de contraseña de cifrado se guardó en el siguiente archivo. |<li>El servidor ya está registrado con otro almacén.<li>Durante la configuración, se ha dañado la frase de contraseña.| Anule el registro del servidor del almacén y vuelva a registrarlo con una nueva frase de contraseña.

## <a name="the-activation-did-not-complete-successfully"></a>La activación no se completó correctamente.

| Error  | Causas posibles | Acciones recomendadas |
|---------|---------|---------|
|<br />La activación no se completó correctamente. No se pudo realizar la operación actual debido a un error de servicio interno [0x1FC07]. Vuelva a intentar la operación más tarde. Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Microsoft.     | <li> La carpeta temporal se encuentra en un volumen sin espacio suficiente. <li> La carpeta temporal se ha trasladado incorrectamente. <li> Falta el archivo OnlineBackup.KEK.         | <li>Actualice a la [versión más reciente](https://aka.ms/azurebackup_agent) del agente de MARS.<li>Mueva la carpeta temporal o la ubicación de caché a un volumen con espacio libre entre el 5 % y el 10 % del tamaño total de los datos de copia de seguridad. Para mover correctamente la ubicación de caché, vea los pasos de [Preguntas comunes acerca de la realización de copias de seguridad de archivos y carpetas](./backup-azure-file-folder-backup-faq.md#manage-the-backup-cache-folder).<li> Asegúrese de que exista el archivo OnlineBackup.KEK. <br>*La ubicación predeterminada de la carpeta temporal o la ruta de acceso a la memoria caché es C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Scratch*.        |

## <a name="encryption-passphrase-not-correctly-configured"></a>La frase de contraseña de cifrado no está configurada correctamente.

| Error  | Causas posibles | Acciones recomendadas |
|---------|---------|---------|
| <br />Error 34506. La frase de contraseña de cifrado almacenada en este equipo no está configurada correctamente.    | <li> La carpeta temporal se encuentra en un volumen sin espacio suficiente. <li> La carpeta temporal se ha trasladado incorrectamente. <li> Falta el archivo OnlineBackup.KEK.        | <li>Actualice a la [versión más reciente](https://aka.ms/azurebackup_agent) del agente de MARS.<li>Mueva la carpeta temporal o la ubicación de caché a un volumen con espacio libre entre el 5 % y el 10 % del tamaño total de los datos de copia de seguridad. Para mover correctamente la ubicación de caché, vea los pasos de [Preguntas comunes acerca de la realización de copias de seguridad de archivos y carpetas](./backup-azure-file-folder-backup-faq.md#manage-the-backup-cache-folder).<li> Asegúrese de que exista el archivo OnlineBackup.KEK. <br>*La ubicación predeterminada de la carpeta temporal o la ruta de acceso a la memoria caché es C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Scratch*.         |

## <a name="backups-dont-run-according-to-schedule"></a>Las copias de seguridad no se ejecutan según la programación

Si las copias de seguridad programadas no se desencadenan automáticamente, pero las copias de seguridad manuales funcionan bien, pruebe las acciones siguientes:

- Asegúrese de que la programación de copia de seguridad de Windows Server no entra en conflicto con la programación de copia de seguridad de archivos y carpetas de Azure.

- Asegúrese de que el estado de copia de seguridad en línea está establecido en **Habilitar**. Haga lo siguiente para comprobar el estado:

  1. En el Programador de tareas, expanda **Microsoft** y seleccione **Copia de seguridad en línea**.
  1. Haga doble clic en **Microsoft-OnlineBackup** y vaya a la pestaña **Desencadenadores**.
  1. Compruebe si el estado de la tarea está establecido en **Habilitado**. Si no lo está, seleccione **Editar**, luego **Habilitado** y, por último, **Aceptar**.

- Asegúrese de que la cuenta de usuario que está seleccionada para la ejecución de la tarea es la de **sistema** o la del **grupo de administradores locales** del servidor. Para comprobar la cuenta de usuario, vaya a la pestaña **General** y consulte las opciones en **Seguridad**.

- Asegúrese de que PowerShell 3.0 o posterior está instalado en el servidor. Para comprobar la versión de PowerShell, ejecute este comando y confirme que el número de versión reflejado en `Major` es 3 o posterior:

  `$PSVersionTable.PSVersion`

- Asegúrese de que esta ruta de acceso forma parte de la variable de entorno `PSMODULEPATH`:

  `<MARS agent installation path>\Microsoft Azure Recovery Services Agent\bin\Modules\MSOnlineBackup`

- Si la directiva de ejecución de PowerShell para `LocalMachine` está establecida en `restricted`, el cmdlet de PowerShell que desencadena la tarea de copia de seguridad puede producir errores. Ejecute estos comandos con permisos elevados para comprobar y establecer la directiva de ejecución en `Unrestricted` o `RemoteSigned`:

 ```PowerShell
 Get-ExecutionPolicy -List

Set-ExecutionPolicy Unrestricted
 ```

- Asegúrese de que no falta ningún archivo de MSOnlineBackup del módulo de PowerShell ni está dañado. Haga lo siguiente si falta algún archivo o está dañado:

  1. Desde cualquier máquina que tenga un agente de MARS que funcione correctamente, copie la carpeta MSOnlineBackup de C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules.
  1. En la máquina con el error, pegue los archivos copiados en la misma ubicación de carpeta (C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules).

     Si ya hay una carpeta MSOnlineBackup en la máquina, pegue los archivos en ella o reemplace los archivos existentes.

> [!TIP]
> Para garantizar que los cambios realizados se aplican de forma coherente, reinicie el servidor después de realizar los pasos anteriores.

## <a name="resource-not-provisioned-in-service-stamp"></a>Recurso no aprovisionado en la marca de servicio

Error | Causas posibles | Acciones recomendadas
--- | --- | ---
Error en la operación actual debido a un error interno del servicio de tipo "Recurso no aprovisionado en marca de servicio". Vuelva a intentar la operación más tarde. (Id.: 230006) | Se cambió el nombre del servidor protegido. | <li> Revierta el nombre del servidor al original, tal como está registrado en el almacén. <br> <li> Vuelva a registrar el servidor en el almacén con el nuevo nombre.

## <a name="job-could-not-be-started-as-another-job-was-in-progress"></a>No se pudo iniciar el trabajo porque había otro trabajo en curso

Si aparece un mensaje de advertencia en la **consola de MARS** > **Historial de trabajos**, que dice: "El trabajo no puede iniciarse mientras otro trabajo está en curso", podría deberse a una duplicación del trabajo desencadenada por el Programador de tareas.

![No se pudo iniciar el trabajo porque había otro trabajo en curso](./media/backup-azure-mars-troubleshoot/job-could-not-be-started.png)

Para solucionar este problema:

1. Inicie el complemento del Programador de tareas, en la ventana de ejecución. Para hacerlo, escriba *taskschd.msc*
1. En el panel izquierdo, vaya a **biblioteca del Programador de tareas** -> **Microsoft** -> **OnlineBackup**.
1. Para cada tarea de esta biblioteca, haga doble clic en la tarea para abrir propiedades y realice los pasos siguientes:
    1. Cambie a la pestaña **Configuración** .

         ![Pestaña Settings](./media/backup-azure-mars-troubleshoot/settings-tab.png)

    1. Cambie la opción  **si la tarea ya está en ejecución, entonces se aplica la siguiente regla**. Elija **No iniciar una instancia nueva**.

         ![Cambie la regla a no iniciar nueva instancia](./media/backup-azure-mars-troubleshoot/change-rule.png)

## <a name="troubleshoot-restore-problems"></a>Solución de problemas de restauración

Es posible que Azure Backup no monte correctamente el volumen de recuperación, incluso tras varios minutos. También se podrían producir mensajes de error durante el proceso. Para empezar la recuperación normalmente, siga estos pasos:

1. Cancele el proceso de montaje si se ha estado ejecutando durante varios minutos.

2. Compruebe si tiene la versión más reciente del agente de Backup. Para averiguar la versión, en el panel **Acciones** de la consola de MARS, seleccione **Acerca del Agente de Microsoft Azure Recovery Services**. Confirme que el número de **versión** es igual o mayor que la versión mencionada en [este artículo](https://go.microsoft.com/fwlink/?linkid=229525). Seleccione este vínculo para [descargar la versión más reciente](https://go.microsoft.com/fwLink/?LinkID=288905).

3. Vaya a **Administrador de dispositivos** > **Controladores de almacenamiento** y busque el **iniciador iSCSI de Microsoft**. Si lo encuentra, vaya directamente al paso 7.

4. Si no encuentra el servicio del iniciador iSCSI de Microsoft, intente encontrar una entrada en **Administrador de dispositivos** > **Controladores de almacenamiento** denominada **Dispositivo desconocido** con el identificador de hardware **ROOT\ISCSIPRT**.

5. Haga clic con el botón derecho en **Dispositivo desconocido** y seleccione **Actualizar software de controlador**.

6. Actualice el controlador. Para ello, seleccione la opción **Buscar software de controlador actualizado automáticamente**. Esta actualización debe cambiar **Dispositivo desconocido** por **Iniciador iSCSI de Microsoft**:

    ![Captura de pantalla del Administrador de dispositivos de Azure Backup con la opción Controladores de almacenamiento resaltada](./media/backup-azure-restore-windows-server/UnknowniSCSIDevice.png)

7. Vaya a **Administrador de tareas** > **Servicios (local)**  > **Servicio del iniciador iSCSI de Microsoft**:

    ![Captura de pantalla del Administrador de tareas de Azure Backup con la opción Servicios (local) resaltada](./media/backup-azure-restore-windows-server/MicrosoftInitiatorServiceRunning.png)

8. Reinicie el servicio del iniciador iSCSI de Microsoft. Para ello, haga clic con el botón derecho en el servicio y, después, seleccione **Detener**. Luego, haga clic con el botón derecho otra vez y seleccione **Iniciar**.

9. Vuelva a intentar la recuperación mediante la [restauración instantánea](backup-instant-restore-capability.md).

Si la recuperación sigue sin funcionar, reinicie el cliente o el servidor. Si no quiere reiniciar o la recuperación sigue sin funcionar incluso después del reinicio del servidor, pruebe a [recuperar desde otra máquina](backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine).

## <a name="troubleshoot-cache-problems"></a>Solución de problemas de caché

Se puede producir un error en la operación de copia de seguridad si la carpeta de caché (también denominada carpeta temporal) no está configurada correctamente, tiene acceso restringido o faltan requisitos previos.

### <a name="prerequisites"></a>Prerrequisitos

Para que las operaciones del agente de MARS se realicen correctamente, la carpeta de caché debe cumplir los siguientes requisitos:

- [Asegúrese de que hay disponible entre un 5 % y un 10 % de espacio de volumen en la ubicación de la carpeta temporal](backup-azure-file-folder-backup-faq.md#whats-the-minimum-size-requirement-for-the-cache-folder).
- [Asegúrese de que la ubicación de la carpeta temporal es válida y accesible](backup-azure-file-folder-backup-faq.md#how-to-check-if-scratch-folder-is-valid-and-accessible).
- [Asegúrese de que se admiten los atributos de archivo en la carpeta de caché](backup-azure-file-folder-backup-faq.md#are-there-any-attributes-of-the-cache-folder-that-arent-supported).
- [Asegúrese de que el espacio de almacenamiento de instantáneas asignado sea suficiente para el proceso de copia de seguridad](#increase-shadow-copy-storage).
- [Asegúrese de que no haya otros procesos (por ejemplo, software antivirus) que restrinjan el acceso a la carpeta de caché](#another-process-or-antivirus-software-blocking-access-to-cache-folder).

### <a name="increase-shadow-copy-storage"></a>Aumento del almacenamiento de instantáneas

Se podrían producir errores en las operaciones de copia de seguridad si no hay suficiente espacio de almacenamiento de instantáneas para proteger el origen de datos. Para resolver este problema, aumente el espacio de almacenamiento de instantáneas en el volumen protegido mediante **vssadmin**, como se muestra a continuación:

- Compruebe el espacio de almacenamiento de instantáneas actual desde el símbolo del sistema con privilegios elevados:<br/>
  `vssadmin List ShadowStorage /For=[Volume letter]:`
- Aumente el espacio de almacenamiento de instantáneas mediante el siguiente comando:<br/>
  `vssadmin Resize ShadowStorage /On=[Volume letter]: /For=[Volume letter]: /Maxsize=[size]`

### <a name="another-process-or-antivirus-software-blocking-access-to-cache-folder"></a>Otro proceso o software antivirus bloquea el acceso a la carpeta de caché

Si tiene un software antivirus instalado en el servidor, agregue al examen antivirus las reglas de exclusión necesarias correspondientes a los siguientes archivos y carpetas:  

- La carpeta temporal. Su ubicación predeterminada es `C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch`.
- La carpeta Bin se encuentra en `C:\Program Files\Microsoft Azure Recovery Services Agent\Bin`.
- CBengine.exe
- CSC.exe

## <a name="common-issues"></a>Problemas comunes

En esta sección se tratan los errores que se suelen producir al usar el agente de MARS.

### <a name="salchecksumstoreinitializationfailed"></a>SalChecksumStoreInitializationFailed

Mensaje de error | Acción recomendada
--|--
El agente de Microsoft Azure Recovery Services no pudo acceder a la suma de comprobación de la copia de seguridad almacenada en la ubicación temporal | Para resolver este problema, siga estos pasos y reinicie el servidor: <br/> - [Compruebe si hay un antivirus u otros procesos que bloqueen los archivos de ubicación temporal.](#another-process-or-antivirus-software-blocking-access-to-cache-folder)<br/> - [Compruebe si la ubicación temporal es válida y accesible para el agente de MARS.](backup-azure-file-folder-backup-faq.md#how-to-check-if-scratch-folder-is-valid-and-accessible)

### <a name="salvhdinitializationerror"></a>SalVhdInitializationError

Mensaje de error | Acción recomendada
--|--
El agente de Microsoft Azure Recovery Services no pudo acceder a la ubicación temporal para inicializar el VHD | Para resolver este problema, siga estos pasos y reinicie el servidor: <br/> - [Compruebe si hay un antivirus u otros procesos que bloqueen los archivos de ubicación temporal.](#another-process-or-antivirus-software-blocking-access-to-cache-folder)<br/> - [Compruebe si la ubicación temporal es válida y accesible para el agente de MARS.](backup-azure-file-folder-backup-faq.md#how-to-check-if-scratch-folder-is-valid-and-accessible)

### <a name="sallowdiskspace"></a>SalLowDiskSpace

Mensaje de error | Acción recomendada
--|--
No se pudo realizar la copia de seguridad debido a almacenamiento insuficiente en el volumen donde se encuentra la carpeta temporal | Para resolver este problema, haga las siguientes comprobaciones y vuelva a intentar la operación:<br/>- [Asegúrese de que el agente de MARS es el más reciente.](https://go.microsoft.com/fwlink/?linkid=229525&clcid=0x409)<br/> - [Compruebe y resuelva los problemas de almacenamiento que afecten al espacio de almacenamiento temporal de las copias de seguridad.](#prerequisites)

### <a name="salbitmaperror"></a>SalBitmapError

Mensaje de error | Acción recomendada
--|--
No se puede buscar cambios en un archivo. Esto podría deberse a diversos motivos. Vuelva a intentar la operación y, | Para resolver este problema, haga las siguientes comprobaciones y vuelva a intentar la operación:<br/> - [Asegúrese de que el agente de MARS es el más reciente.](https://go.microsoft.com/fwlink/?linkid=229525&clcid=0x409) <br/> - [Compruebe y resuelva los problemas de almacenamiento que afecten al espacio de almacenamiento temporal de las copias de seguridad.](#prerequisites)

## <a name="next-steps"></a>Pasos siguientes

- Más detalles sobre la [creación de copias de seguridad de Windows Server con el agente de Azure Backup](tutorial-backup-windows-server-to-azure.md).
- Si necesita restaurar una copia de seguridad, vea [Restauración de archivos en Windows](backup-azure-restore-windows-server.md).
