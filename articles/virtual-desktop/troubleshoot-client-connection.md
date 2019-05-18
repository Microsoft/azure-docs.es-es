---
title: Conexiones de cliente de escritorio remoto en el escritorio Virtual de Windows - Azure
description: Cómo resolver problemas al configurar las conexiones de cliente en un entorno de inquilinos de Escritorio Virtual de Windows.
services: virtual-desktop
author: ChJenk
ms.service: virtual-desktop
ms.topic: troubleshoot
ms.date: 04/08/2019
ms.author: v-chjenk
ms.openlocfilehash: 99295fd4581cd81751f7d64b694c853efe51a106
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522940"
---
# <a name="remote-desktop-client-connections"></a>Conexiones de cliente de Escritorio remoto

Use este artículo para resolver problemas con conexiones de cliente de Escritorio Virtual de Windows.

## <a name="provide-feedback"></a>Proporcionar comentarios

En este momento no se aceptan casos de soporte técnico mientras Windows Virtual Desktop se encuentre en versión preliminar. Visite la [Comunidad técnica de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) para hablar sobre Windows Virtual Desktop con el equipo de producto y los miembros activos de la comunidad.

## <a name="you-cant-open-a-web-client"></a>No se puede abrir a un cliente web

Confirme que hay conectividad a internet, abra otro sitio web. Por ejemplo, [www.Bing.com](https://www.bing.com).

Use **nslookup** para confirmar que DNS pueda resolver el FQDN:

    ```cmd
    nslookup rdweb.wvd.microsoft.com
    ```

Intente conectarse con otro cliente, al igual que el cliente de escritorio remoto para Windows 7 o Windows 10 y comprobación ver si puede abrir el cliente web.

### <a name="error-opening-another-site-fails"></a>Error: Abrir otro se produce un error de sitio

**Causa:** Problemas de red o interrupciones.

**Fix:** Póngase en contacto con soporte técnico de la red.

### <a name="error-nslookup-cannot-resolve-the-name"></a>Error: Nslookup no puede resolver el nombre

**Causa:** Problemas de red o interrupciones.

**Fix:** Póngase en contacto con soporte técnico de red

### <a name="error-you-cant-connect-but-other-clients-can-connect"></a>Error: No puede conectarse, pero pueden conectarse a otros clientes

**Causa:** El explorador no se comporta como trabajo esperado y detenido.

**Fix:** Siga estas instrucciones para solucionar problemas del explorador.

1. Reinicie el explorador.
2. Cookies del explorador no cifrado. Consulte [cómo eliminar archivos de cookies en Internet Explorer](https://support.microsoft.com/help/278835/how-to-delete-cookie-files-in-internet-explorer).
3. Caché del explorador no cifrado. Consulte [borrar la memoria caché del explorador para su explorador](https://binged.it/2RKyfdU).
4. Explorador abierto en modo privado.

## <a name="web-client-stops-responding-or-disconnects"></a>Cliente Web deja de responder o se desconecta

Intente conectarse con otro explorador o cliente.

### <a name="error-other-browsers-and-clients-also-malfunction-or-fail-to-open"></a>Error: Otros exploradores y los clientes también funcione incorrectamente o no abrirse

**Causa:** Problemas de red o la operación del sistema o las interrupciones

**Fix:** Póngase en contacto con soporte técnico de equipos.

## <a name="web-client-keeps-prompting-for-credentials"></a>Cliente Web mantiene solicitar las credenciales

Si el cliente Web mantiene solicitar credenciales, siga estas instrucciones.

1. Confirme la que dirección URL del cliente web es correcta.
2. Confirme que las credenciales son para el entorno de Escritorio Virtual de Windows asociado a la dirección URL.
3. Cookies del explorador no cifrado. Consulte [cómo eliminar archivos de cookies en Internet Explorer](https://support.microsoft.com/help/278835/how-to-delete-cookie-files-in-internet-explorer).
4. Caché del explorador no cifrado. Consulte [borrar la memoria caché del explorador para su explorador](https://binged.it/2RKyfdU).
5. Explorador abierto en modo privado.

## <a name="remote-desktop-client-for-windows-7-or-windows-10-stops-responding-or-cannot-be-opened"></a>Cliente de escritorio remoto para Windows 7 o Windows 10 deja de responder o no se puede abrir

Use los siguientes cmdlets de PowerShell para limpiar los registros de cliente de fuera de banda (OOB).

```PowerShell
Remove-ItemProperty 'HKCU:\Software\Microsoft\Terminal Server Client\Default' - Name FeedURLs

#Remove RdClientRadc registry key
Remove-Item 'HKCU:\Software\Microsoft\RdClientRadc' -Recurse

#Remove all files under %appdata%\RdClientRadc
Remove-Item C:\Users\pavithir\AppData\Roaming\RdClientRadc\* -Recurse
```

Vaya a **%AppData%\RdClientRadc** y elimine todo el contenido.

Desinstale y reinstale el cliente de escritorio remoto para Windows 7 y Windows 10.

## <a name="troubleshooting-end-user-connectivity"></a>Solución de problemas de conectividad del usuario final

A veces, los usuarios pueden tener acceso a sus recursos locales y fuente, pero todavía tiene problemas de rendimiento que les impiden tener acceso a recursos remotos, la disponibilidad o la configuración. En estos casos, el usuario obtiene mensajes similares a los siguientes:

![Mensaje de error de conexión a Escritorio remoto.](media/eb76b666808bddb611448dfb621152ce.png)

![No se puede conectar al mensaje de error de puerta de enlace.](media/a8fbb9910d4672147335550affe58481.png)

Siga estas instrucciones para solucionar problemas generales para los códigos de error de conexión de cliente.

1. Confirme el nombre de usuario y la hora cuando se produjo el problema.
2. Abra **PowerShell** y establecer conexión con el inquilino de Escritorio Virtual de Windows donde se notificó el problema.
3. Confirmar la conexión al inquilino correcto con **Get RdsTenant.**
4. Uso de **Get RdsHostPool** y **Get RdsSessionHost** cmdlets, confirme que está realizando la solución de problemas en el grupo host correcto.
5. Ejecute el comando siguiente para obtener una lista de todas las actividades con errores de conexión de tipo para el período de tiempo especificado:

    ```cmd
     Get-RdsDiagnosticActivities -TenantName <TenantName> -username <UPN> -StartTime
     "11/21/2018 1:07:03 PM" -EndTime "11/21/2018 1:27:03 PM" -Outcome Failure -ActivityType Connection
    ```

6. Mediante el **ActivityId** desde la salida del cmdlet anterior, ejecute el siguiente comando:

    ```
    (Get-RdsDiagnosticActivities -TenantName $tenant -ActivityId <ActivityId> -Detailed).Errors
    ```

7. El comando genera una salida similar a la salida que se muestra a continuación. Use **ErrorCodeSymbolic** y **ErrorMessage** para solucionar la causa raíz.

    ```
    ErrorSource       : <Source>
    ErrorOperation    : <Operation>
    ErrorCode         : <Error code>
    ErrorCodeSymbolic : <Error code string>
    ErrorMessage      : <Error code message>
    ErrorInternal     : <Internal for the OS>
    ReportedBy        : <Reported by component>
    Time              : <Timestampt>
    ```

### <a name="error-oaddusertogroupfailed--failed-to-add-user--username-to-group--remote-desktop-users-reason-win32errornosuchmember"></a>Error: O_ADD_USER_TO_GROUP_FAILED / no se pudo agregar el usuario = ≤username≥ al grupo = usuarios de escritorio remoto. Motivo: Win32.ERROR_NO_SUCH_MEMBER

**Causa:** Máquina virtual no se ha unido al dominio donde está el objeto de usuario.

**Fix:** Agregar máquina virtual al dominio correcto. Consulte [unir una máquina virtual Windows Server a un dominio administrado](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-admin-guide-join-windows-vm-portal).

### <a name="error-nslookup-cannot-resolve-the-name"></a>Error: Nslookup no puede resolver el nombre

**Causa:** Problemas de red o interrupciones.

**Fix:** Póngase en contacto con soporte técnico de red

### <a name="error-connectionfailedclientprotocolerror"></a>Error: ConnectionFailedClientProtocolError

**Causa:** Las máquinas virtuales que el usuario está intentando conectarse a no están unidos a un dominio.

**Fix:** Únase a todas las máquinas virtuales que forman parte de un grupo host para el controlador de dominio.

## <a name="user-connects-but-nothing-is-displayed-no-feed"></a>Se conecta el usuario pero no se muestra nada (ninguna fuente de distribución)

Un usuario puede iniciar a clientes de escritorio remoto y es capaz de realizar la autenticación, pero el usuario no ve ningún icono en la fuente de detección de web.

Confirme que el usuario los problemas de informes se ha asignado a grupos de aplicaciones mediante el uso de esta línea de comandos:

```cmd
Get-RdsAppGroupUser <tenantname> <hostpoolname> <appgroupname>
```

Confirme que el usuario inicia sesión con las credenciales correctas.

Si se utiliza el cliente web, confirme que no hay ningún problema de credenciales almacenadas en caché.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo solucionar problemas de Escritorio Virtual de Windows y las pistas de escalado, consulte [solución de problemas de introducción, comentarios y soporte técnico](troubleshoot-set-up-overview.md).
- Para solucionar problemas durante la creación de un grupo de inquilinos y host en un entorno de Escritorio Virtual de Windows, consulte [creación del grupo de inquilinos y host](troubleshoot-set-up-issues.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en el escritorio Virtual de Windows, consulte [configuración de máquina virtual del host de sesión](troubleshoot-vm-configuration.md).
- Para solucionar problemas al usar PowerShell con el escritorio Virtual de Windows, consulte [Windows PowerShell de Escritorio Virtual](troubleshoot-powershell.md).
- Para obtener más información sobre el servicio en versión preliminar, consulte [entorno de versión preliminar de Windows Desktop](https://docs.microsoft.com/azure/virtual-desktop/environment-setup?).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de implementaciones de plantilla de Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-tutorial-troubleshoot).
- Para más información sobre las acciones de auditoría, consulte [Operaciones de auditoría con Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit).
- Si desea conocer más detalles sobre las acciones que permiten determinar los errores durante la implementación, consulte [Visualización de operaciones de implementación con el Portal de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-deployment-operations).