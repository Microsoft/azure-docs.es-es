---
title: 'Solución de problemas de host de sesión de Windows Virtual Desktop (clásico): Azure'
description: Cómo resolver problemas al configurar máquinas virtuales de host de sesión en Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: troubleshooting
ms.date: 05/11/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 90b4c574a03d8dee50beff60304fb5c1f3b52945
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008634"
---
# <a name="windows-virtual-desktop-classic-session-host-virtual-machine-configuration"></a>Configuración de máquina virtual de host de sesión de Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../troubleshoot-vm-configuration.md).

Use este artículo para solucionar los problemas que tiene al configurar las máquinas de virtuales (VM) de host de sesión de Windows Virtual Desktop.

## <a name="provide-feedback"></a>Envío de comentarios

Visite la [Comunidad técnica de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) para hablar sobre Windows Virtual Desktop con el equipo de producto y los miembros activos de la comunidad.

## <a name="vms-are-not-joined-to-the-domain"></a>Las VM no se unen al dominio

Siga estas instrucciones si tiene problemas para unir las VM al dominio.

- Una la VM manualmente con el proceso de [Unir una máquina virtual de Windows Server a un dominio administrado](../../active-directory-domain-services/join-windows-vm.md) o con la [plantilla de unión a un dominio](https://azure.microsoft.com/resources/templates/201-vm-domain-join-existing/).
- Intente hacer ping en el nombre de dominio desde la línea de comandos en la VM.
- Revise la lista de mensajes de error de unión a un dominio en [Troubleshooting Domain Join Error Messages](https://social.technet.microsoft.com/wiki/contents/articles/1935.troubleshooting-domain-join-error-messages.aspx) (Solución de problemas con mensajes de error de unión a un dominio).

### <a name="error-incorrect-credentials"></a>Error: Credenciales incorrectas

**Causa:** Hubo un error ortográfico al escribir las credenciales en las correcciones de la interfaz de la plantilla de Azure Resource Manager.

**Solución:** realice una de las siguientes acciones para resolverlo.

- Agregue manualmente las VM a un dominio.
- Vuelva a implementar la plantilla una vez que las credenciales se hayan confirmado. Consulte [Creación de un grupo host con PowerShell](create-host-pools-powershell-2019.md).
- Una las VM a un dominio con una plantilla con [Joins an existing Windows VM to AD Domain](https://azure.microsoft.com/resources/templates/201-vm-domain-join-existing/) (Unir una VM Windows existente a un dominio de AD).

### <a name="error-timeout-waiting-for-user-input"></a>Error: Se agotó el tiempo de espera para la entrada del usuario

**Causa:** La cuenta usada para completar la unión a un dominio puede tener autenticación multifactor (MFA).

**Solución:** realice una de las siguientes acciones para resolverlo.

- Quite temporalmente la MFA para la cuenta.
- Use una cuenta de servicio.

### <a name="error-the-account-used-during-provisioning-doesnt-have-permissions-to-complete-the-operation"></a>Error: La cuenta usada durante el aprovisionamiento no tiene permisos para completar la operación

**Causa:** La cuenta usada no tiene permisos para unir las VM al dominio debido al cumplimiento de normas y reglamentos.

**Solución:** realice una de las siguientes acciones para resolverlo.

- Use una cuenta que sea miembro del grupo Administrador.
- Conceda los permisos necesarios para la cuenta usada.

### <a name="error-domain-name-doesnt-resolve"></a>Error: El nombre de dominio no se resuelve

**Causa 1:** las máquinas virtuales están en una red virtual que no está asociada con la red virtual (VNET) donde se encuentra el dominio.

**Corrección 1:** Cree el emparejamiento de VNET entre la red virtual donde se han aprovisionado las VM y la red virtual donde se ejecuta el controlador de dominio (DC). Consulte [Crear un emparejamiento de redes virtuales: Resource Manager, suscripciones diferentes](../../virtual-network/create-peering-different-subscriptions.md).

**Causa 2:** cuando se usa Azure Active Directory Domain Services (Azure AD DS), la red virtual no tiene la configuración del servidor DNS actualizada para apuntar a los controladores de dominio administrados.

**Corrección 2:** con el fin de actualizar la configuración DNS de la red virtual que contiene Azure AD DS, [consulte Actualización de la configuración DNS para la red virtual de Azure](../../active-directory-domain-services/tutorial-create-instance.md#update-dns-settings-for-the-azure-virtual-network).

**Causa 3:** la configuración del servidor DNS de la interfaz de red no apunta al servidor DNS adecuado en la red virtual.

**Corrección 3:** realice una de las siguientes acciones para resolverlo, siguiendo los pasos descritos en [Cambio de servidores DNS].
- Cambie la configuración del servidor DNS de la interfaz de red a **Personalizado** con los pasos que se indican en [Cambio de servidores DNS](../../virtual-network/virtual-network-network-interface.md#change-dns-servers) y especifique las direcciones IP privadas de los servidores DNS en la red virtual.
- Cambie la configuración del servidor DNS de la interfaz de red a **Herencia de la red virtual** con los pasos que se indican en [Cambio de servidores DNS](../../virtual-network/virtual-network-network-interface.md#change-dns-servers) y, después, cambie la configuración del servidor DNS de la red virtual con los pasos que se indican en [Cambio de servidores DNS](../../virtual-network/manage-virtual-network.md#change-dns-servers).

## <a name="windows-virtual-desktop-agent-and-windows-virtual-desktop-boot-loader-are-not-installed"></a>El agente de Windows Virtual Desktop y el cargador de arranque de Windows Virtual Desktop no están instalados

La manera recomendada para aprovisionar VM es mediante la plantilla **Create and provision Windows Virtual Desktop host pool** (Creación y aprovisionamiento del grupo de host de Windows Virtual Desktop) de Azure Resource Manager. La plantilla instala automáticamente el agente de Windows Virtual Desktop y el cargador de arranque de Windows Virtual Desktop.

Siga estas instrucciones para confirmar que los componentes se han instalado y para comprobar los mensajes de error.

1. Para confirmar que los dos componentes se han instalado, consulte **Panel de control** > **Programas** > **Programas y características**. Si no ve **Windows Virtual Desktop Agent** y **Windows Virtual Desktop Agent Boot Loader**, no están instalados en la VM.
2. Abra el **Explorador de archivos** y vaya a **C:\Windows\Temp\ScriptLog.log**. Si falta el archivo, es señal de que el DSC de PowerShell que instala los dos componentes no pudo ejecutarse en el contexto de seguridad proporcionado.
3. Si se encuentra el archivo **C:\Windows\Temp\ScriptLog.log**, ábralo y busque mensajes de error.

### <a name="error-windows-virtual-desktop-agent-and-windows-virtual-desktop-agent-boot-loader-are-missing-cwindowstempscriptloglog-is-also-missing"></a>Error: Faltan el agente de Windows Virtual Desktop y el cargador de arranque de Windows Virtual Desktop. C:\Windows\Temp\ScriptLog.log también falta

**Causa 1:** Las credenciales proporcionadas durante la entrada de la plantilla de Azure Resource Manager son incorrectas o los permisos eran insuficientes.

**Corrección 1:** Agregue manualmente los componentes que faltan para las VM con [Creación de un grupo host con PowerShell](create-host-pools-powershell-2019.md).

**Causa 2:** DSC de PowerShell se puedo iniciar y ejecutar, pero no se pudo completar, ya no se puede iniciar sesión en Windows Virtual Desktop y obtener la información necesaria.

**Corrección 2:** Confirme los elementos de la lista siguiente.

- Asegúrese de que la cuenta no tenga MFA.
- Confirme que el nombre del inquilino esté correcto y que el inquilino exista en Windows Virtual Desktop.
- Confirme que la cuenta tiene al menos permisos de colaborador de RDS.

### <a name="error-authentication-failed-error-in-cwindowstempscriptloglog"></a>Error: Error de autenticación, error en C:\Windows\Temp\ScriptLog.log

**Causa:** DSC de PowerShell pudo ejecutarse, pero no pudo conectarse a Windows Virtual Desktop.

**Solución:** Confirme los elementos de la lista siguiente.

- Registre manualmente las VM con el servicio de Windows Virtual Desktop.
- Confirme que la cuenta usada para conectarse a Windows Virtual Desktop tiene permisos en el inquilino para crear grupos host.
- Confirme que la cuenta no tenga MFA.

## <a name="windows-virtual-desktop-agent-is-not-registering-with-the-windows-virtual-desktop-service"></a>El agente de Windows Virtual Desktop no se registra en el servicio de Windows Virtual Desktop

Cuando el agente de Windows Virtual Desktop se instala por primera vez en las VM de host de sesión (ya sea manualmente o a través de la plantilla de Azure Resource Manager y DSC de PowerShell), proporciona un token de registro. En la siguiente sección se trata la solución de problemas aplicables al agente de Windows Virtual Desktop y el token.

### <a name="error-the-status-filed-in-get-rdssessionhost-cmdlet-shows-status-as-unavailable"></a>Error: El estado archivado en el cmdlet Get-RdsSessionHost muestra el estado como no disponible

> [!div class="mx-imgBorder"]
> ![El cmdlet Get-RdsSessionHost muestra el estado como no disponible.](../media/23b8e5f525bb4e24494ab7f159fa6b62.png)

**Causa:** El agente no puede actualizarse automáticamente a una nueva versión.

**Solución:** Siga estas instrucciones para actualizar manualmente el agente.

1. Descargue una nueva versión del agente en la VM del host de sesión.
2. Inicie el Administrador de tareas y, en la pestaña Servicio, detenga el servicio RDAgentBootLoader.
3. Ejecute al instalador para la nueva versión del agente de Windows Virtual Desktop.
4. Cuando se le pida el token de registro, quite la entrada INVALID_TOKEN y presione Siguiente (no es necesario un nuevo token).
5. Complete el Asistente para la instalación.
6. Abra el Administrador de tareas e inicie el servicio RDAgentBootLoader.

## <a name="error--windows-virtual-desktop-agent-registry-entry-isregistered-shows-a-value-of-0"></a>Error:  La entrada del registro IsRegistered del agente de Windows Virtual Desktop muestra un valor de 0

**Causa:** El token de registro ha expirado o se ha generado con el valor de expiración de 999999.

**Solución:** Siga estas instrucciones para corregir el error del registro del agente.

1. Si ya hay un token de registro, quítelo con Remove-RDSRegistrationInfo.
2. Genere un nuevo token con Rds-NewRegistrationInfo.
3. Confirme que el parámetro -ExpriationHours esté configurado en 72 (el valor máximo es 99999).

### <a name="error-windows-virtual-desktop-agent-isnt-reporting-a-heartbeat-when-running-get-rdssessionhost"></a>Error: El agente de Windows Virtual Desktop no notifica un latido al ejecutar Get-RdsSessionHost

**Causa 1:** Se ha detenido el servicio RDAgentBootLoader.

**Corrección 1:** Inicie el Administrador de tareas y, si la pestaña Servicio notifica un estado detenido para el servicio RDAgentBootLoader, inicie el servicio.

**Causa 2:** Puede que el puerto 443 esté cerrado.

**Corrección 2:** Siga estas instrucciones para abrir el puerto 443.

1. Para confirmar que el puerto 443 esté abierto, descargue la herramienta PSPing desde las [herramientas de Sysinternal](/sysinternals/downloads/psping/).
2. Instale PSPing en la VM de host de sesión donde se ejecuta el agente.
3. Abra el símbolo del sistema como administrador y emita el comando siguiente:

    ```cmd
    psping rdbroker.wvdselfhost.microsoft.com:443
    ```

4. Confirme que PSPing reciba la información desde RDBroker:

    ```
    PsPing v2.10 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2016 Mark Russinovich
    Sysinternals - www.sysinternals.com
    TCP connect to 13.77.160.237:443:
    5 iterations (warmup 1) ping test:
    Connecting to 13.77.160.237:443 (warmup): from 172.20.17.140:60649: 2.00ms
    Connecting to 13.77.160.237:443: from 172.20.17.140:60650: 3.83ms
    Connecting to 13.77.160.237:443: from 172.20.17.140:60652: 2.21ms
    Connecting to 13.77.160.237:443: from 172.20.17.140:60653: 2.14ms
    Connecting to 13.77.160.237:443: from 172.20.17.140:60654: 2.12ms
    TCP connect statistics for 13.77.160.237:443:
    Sent = 4, Received = 4, Lost = 0 (0% loss),
    Minimum = 2.12ms, Maximum = 3.83ms, Average = 2.58ms
    ```

## <a name="troubleshooting-issues-with-the-windows-virtual-desktop-side-by-side-stack"></a>Solución de problemas con la pila en paralelo de Windows Virtual Desktop

La pila en paralelo de Windows Virtual Desktop se instala automáticamente con Windows Server 2019. Use Microsoft Installer (MSI) para instalar la pila en paralelo en Microsoft Windows Server 2016 o Windows Server 2012 R2. Para Microsoft Windows 10, la pila en paralelo de Windows Virtual Desktop se habilita con **enablesxstackrs.ps1**.

Hay tres maneras principales de instalar la pila en paralelo o habilitarla en VM del grupo de host de sesión:

- Con la plantilla **Create and provision Windows Virtual Desktop host pool** (Creación y aprovisionamiento del grupo de host de Windows Virtual Desktop) de Azure Resource Manager.
- Al incluirla y habilitarla en la imagen maestra.
- Al instalarla o habilitarla manualmente en cada VM (con extensiones o PowerShell).

Si tiene problemas con la pila en paralelo de Windows Virtual Desktop, escriba el comando **qwinsta** desde el símbolo del sistema para confirmar que la pila en paralelo esté instalada o habilitada.

La salida de **qwinsta** enumerará **rdp-sxs** en el resultado si la pila en paralelo está instalada y habilitada.

> [!div class="mx-imgBorder"]
> ![La pila en paralelo instalada o habilitada, en la que qwinsta aparece como rdp-sxs en la salida.](../media/23b8e5f525bb4e24494ab7f159fa6b62.png)

Examine las entradas del Registro que se enumeran a continuación y confirme que coincidan con sus valores. Si faltan las claves del Registro o los valores no coinciden, siga las instrucciones de [Creación de un grupo host con PowerShell](create-host-pools-powershell-2019.md) para conocer cómo reinstalar la pila en paralelo.

```registry
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal
    Server\WinStations\rds-sxs\"fEnableWinstation":DWORD=1

    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal
    Server\ClusterSettings\"SessionDirectoryListener":rdp-sxs
```

### <a name="error-o_reverse_connect_stack_failure"></a>Error: O_REVERSE_CONNECT_STACK_FAILURE

> [!div class="mx-imgBorder"]
> ![Código de error de O_REVERSE_CONNECT_STACK_FAILURE.](../media/23b8e5f525bb4e24494ab7f159fa6b62.png)

**Causa:** La pila en paralelo no está instalada en la VM del host de sesión.

**Solución:** Siga estas instrucciones para instalar la pila en paralelo en la VM del host de sesión.

1. Use el Protocolo de escritorio remoto (RDP) para ir directamente a la VM del host de sesión como administrador local.
2. Descargue e importe [el módulo de PowerShell para Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) que se usará en la sesión de PowerShell, si aún no lo ha hecho y, a continuación, ejecute este cmdlet para iniciar sesión en la cuenta:

    ```powershell
    Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
    ```

3. Instale la pila en paralelo con [Creación de un grupo host con PowerShell](create-host-pools-powershell-2019.md).

## <a name="how-to-fix-a-windows-virtual-desktop-side-by-side-stack-that-malfunctions"></a>Cómo corregir una pila en paralelo de Windows Virtual Desktop con error de funcionamiento

Existen circunstancias conocidas que pueden provocar que la pila en paralelo no funcione correctamente:

- No seguir el orden correcto de los pasos para habilitar la pila en paralelo
- Actualizar automáticamente a Enhanced Versatile Disc (EVD) de Windows 10
- La falta del rol de host de sesión de Escritorio remoto (RDSH)
- Ejecutar enablesxsstackrc.ps1 varias veces
- Ejecutar enablesxsstackrc.ps1 en una cuenta que no tiene privilegios de administrador local

Las instrucciones de esta sección pueden ayudarle a desinstalar la pila en paralelo de Windows Virtual Desktop. Una vez que se desinstale la pila en paralelo, vaya a "Registro de la máquina virtual al grupo host de Windows Virtual Desktop" en [Creación de un grupo host con PowerShell](create-host-pools-powershell-2019.md) para reinstalar la pila en paralelo.

La VM usada para ejecutar la corrección debe estar en la misma subred y el mismo dominio que la VM con la pila en paralelo con error de funcionamiento.

Siga estas instrucciones para ejecutar la corrección desde la misma subred y dominio:

1. Conéctese con el Protocolo de escritorio remoto (RDP) a la VM desde donde se aplicará la revisión.
2. Descargue PsExec de https://docs.microsoft.com/sysinternals/downloads/psexec.
3. Descomprima el archivo descargado.
4. Inicie un símbolo del sistema como administrador local.
5. Navegue hasta la carpeta donde ha descomprimido PsExec.
6. En un símbolo del sistema, use el siguiente comando:

    ```cmd
            psexec.exe \\<VMname> cmd
    ```

    >[!NOTE]
    >VMname es el nombre de equipo de la VM con la pila en paralelo con error de funcionamiento.

7. Para aceptar el contrato de licencia de PsExec, haga clic en Aceptar.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla del contrato de licencia de software.](../media/SoftwareLicenseTerms.png)

    >[!NOTE]
    >Este cuadro de diálogo se muestra solo la primera vez que se ejecuta PsExec.

8. Una vez que se abre la sesión de símbolo del sistema en la VM con la pila en paralelo con error de funcionamiento, ejecute qwinsta y confirme que haya disponible una entrada denominada rdp-sxs. Si no es así, no hay una pila en paralelo en la VM, así que el problema no está vinculado a la pila en paralelo.

    > [!div class="mx-imgBorder"]
    > ![Símbolo del sistema de administrador](../media/AdministratorCommandPrompt.png)

9. Ejecute el comando siguiente, que mostrará una lista de componentes de Microsoft instalados en la VM con la pila en paralelo con error de funcionamiento.

    ```cmd
        wmic product get name
    ```

10. Ejecute el comando siguiente con los nombres de productos del paso anterior.

    ```cmd
        wmic product where name="<Remote Desktop Services Infrastructure Agent>" call uninstall
    ```

11. Desinstale todos los productos que empiecen por "Remote Desktop".

12. Una vez se hayan desinstalado todos los componentes de Windows Virtual Desktop, siga las instrucciones para su sistema operativo:

13. Si el sistema operativo es Windows Server, reinicie la VM que tenía la pila en paralelo con error de funcionamiento (ya sea con Azure Portal o desde la herramienta PsExec).

Si el sistema operativo es Microsoft Windows 10, continúe con las instrucciones siguientes:

14. Desde la VM que ejecuta PsExec, abra el Explorador de archivos y copie disablesxsstackrc.ps1 en la unidad del sistema de la VM con la pila en paralelo con error de funcionamiento.

    ```cmd
        \\<VMname>\c$\
    ```

    >[!NOTE]
    >VMname es el nombre de equipo de la VM con la pila en paralelo con error de funcionamiento.

15. Proceso recomendado: desde la herramienta PsExec, inicie PowerShell y navegue hasta la carpeta del paso anterior y ejecute disablesxsstackrc.ps1. Como alternativa, puede ejecutar los cmdlets siguientes:

    ```PowerShell
    Remove-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\ClusterSettings" -Name "SessionDirectoryListener" -Force
    Remove-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\rdp-sxs" -Recurse -Force
    Remove-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations" -Name "ReverseConnectionListener" -Force
    ```

16. Cuando se terminen de ejecutar los cmdlets, reinicie la VM con la pila en paralelo con error de funcionamiento.

## <a name="remote-desktop-licensing-mode-isnt-configured"></a>El modo de licencia de Escritorio remoto no está configurado

Si inicia sesión en Windows 10 Enterprise multisesión con una cuenta administrativa, puede recibir una notificación que diga que "el modo de administración de licencias de Escritorio remoto no está configurado, los servicios de Escritorio remoto dejarán de funcionar en X días. En el servidor de Agente de conexión, use el administrador del servidor para especificar el modo de administración de licencias de Escritorio remoto."

Si el límite de tiempo expira, aparece el mensaje de error "Se desconectó la sesión remota porque no hay licencias de acceso de cliente de Escritorio remoto disponibles para este equipo".

Si ve alguno de estos mensajes, la imagen no tiene instaladas las actualizaciones más recientes de Windows o está configurando el modo de licencia de Escritorio remoto a través de la directiva de grupo. Siga los pasos descritos en las secciones siguientes para comprobar la configuración de la directiva de grupo, identificar la versión de la sesión múltiple de Windows 10 Enterprise e instalar la actualización correspondiente.

>[!NOTE]
>Windows Virtual Desktop solo requiere una licencia de acceso de cliente (CAL) de RDS si el grupo de hosts contiene hosts de sesión de Windows Server. Para obtener información sobre cómo configurar una CAL de RDS, vea [Licencia para la implementación de RDS con licencias de acceso de cliente (CAL)](/windows-server/remote/remote-desktop-services/rds-client-access-license/).

### <a name="disable-the-remote-desktop-licensing-mode-group-policy-setting"></a>Deshabilitación de la configuración de directiva de grupo del modo de licencia de Escritorio remoto

Para comprobar la configuración de la directiva de grupo, abra el Editor de directivas de grupo en la VM y vaya a **Plantillas administrativas** > **Componentes de Windows** > **Servicios de Escritorio remoto**  > **Host de sesión de Escritorio remoto** > **Licencias** > **Establecer el modo de licencia de Escritorio remoto**. Si el valor de la directiva de grupo es **Habilitado**, cámbielo a **Deshabilitado**. Si ya se ha deshabilitado, déjelo tal cual.

>[!NOTE]
>Si establece la directiva de grupo a través de su dominio, deshabilite esta opción en las directivas que tienen como destino estas máquinas virtuales de sesión múltiple de Windows 10 Enterprise.

### <a name="identify-which-version-of-windows-10-enterprise-multi-session-youre-using"></a>Identificación de la versión de Windows 10 Enterprise multisesión que está usando

Para consultar la versión de Windows 10 Enterprise multisesión que tiene:

1. Inicie sesión con su cuenta de administrador.
2. Escriba "Acerca de" en la barra de búsqueda situada junto al menú Inicio.
3. Seleccione **Acerca de tu PC**.
4. Consulte el número que aparece junto a "Versión". El número debe ser "1809" o "1903", tal como se muestra en la siguiente imagen.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la ventana de especificaciones de Windows. El número de versión está resaltado en azul.](../media/windows-specifications.png)

Ahora que conoce el número de versión, vaya directamente a la sección correspondiente.

### <a name="version-1809"></a>Versión 1809

Si el número de versión indica "1809", instale [la actualización KB4516077 ](https://support.microsoft.com/help/4516077).

### <a name="version-1903"></a>Versión 1903

Vuelva a implementar el sistema operativo host con la versión más reciente de la imagen de Windows 10, versión 1903, de la galería de Azure.

## <a name="we-couldnt-connect-to-the-remote-pc-because-of-a-security-error"></a>No se pudo conectar al equipo remoto debido a un error de seguridad

Si los usuarios ven un error que dice "no se pudo conectar al equipo remoto debido a un error de seguridad. Si esto sigue ocurriendo, póngase en contacto con el administrador o el servicio de soporte técnico para obtener ayuda", valide las directivas existentes que cambian los permisos RDP predeterminados. Una directiva que podría provocar que este error aparezca es "Permitir el inicio de sesión a través de la directiva de seguridad de los Servicios de Escritorio remoto".

Para obtener más información acerca de esta directiva, consulte [Permitir el inicio de sesión a través de Servicios de Escritorio remoto](/windows/security/threat-protection/security-policy-settings/allow-log-on-through-remote-desktop-services).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo solucionar problemas de Windows Virtual Desktop y las pistas de escalación, consulte [Introducción, comentarios y soporte técnico para solucionar problemas](troubleshoot-set-up-overview-2019.md).
- Para solucionar problemas durante la creación de un grupo de inquilinos y de hosts en un entorno de Windows Virtual Desktop, consulte [Creación de los grupos de inquilinos y de host](troubleshoot-set-up-issues-2019.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en Windows Virtual Desktop, consulte [Configuración de la máquina virtual del host de sesión](troubleshoot-vm-configuration-2019.md).
- Para solucionar problemas con conexiones de cliente de Windows Virtual Desktop, consulte [Conexiones de servicios de Windows Virtual Desktop](troubleshoot-service-connection-2019.md).
- Para solucionar problemas con los clientes de Escritorio remoto, consulte [Solución de problemas del cliente de Escritorio remoto](../troubleshoot-client.md).
- Para solucionar problemas al usar PowerShell con Windows Virtual Desktop, consulte [PowerShell para Windows Virtual Desktop](troubleshoot-powershell-2019.md).
- Para más información sobre el servicio, consulte [Entorno de Windows Virtual Desktop](environment-setup-2019.md).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager](../../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Para más información sobre las acciones de auditoría, consulte [Operaciones de auditoría con Resource Manager](../../azure-resource-manager/management/view-activity-logs.md).
- Si desea conocer más detalles sobre las acciones que permiten determinar los errores durante la implementación, consulte [Visualización de operaciones de implementación con el Portal de Azure](../../azure-resource-manager/templates/deployment-history.md).
