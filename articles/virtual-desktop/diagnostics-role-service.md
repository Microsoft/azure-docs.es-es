---
title: 'Problemas de diagnóstico de Windows Virtual Desktop: Azure'
description: Procedimientos para usar la característica de diagnóstico de Windows Virtual Desktop para diagnosticar problemas.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: troubleshooting
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 6ff1e3d8eb9cb40d46ae0624be9d37fd199accd2
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288755"
---
# <a name="identify-and-diagnose-windows-virtual-desktop-issues"></a>Identificación y diagnóstico de problemas de Windows Virtual Desktop

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/diagnostics-role-service-2019.md).

Windows Virtual Desktop ofrece una característica de diagnóstico que permite al administrador detectar problemas a través de una única interfaz. Para obtener más información sobre las funcionalidades de diagnóstico de Windows Virtual Desktop, consulte [Uso de Log Analytics para la característica de diagnóstico](diagnostics-log-analytics.md).

Las conexiones que no lleguen a Windows Virtual Desktop no aparecerán en los resultados de diagnóstico, ya que el propio servicio de rol de diagnóstico forma parte de Windows Virtual Desktop. Se pueden producir problemas de conexión a Windows Virtual Desktop si el usuario final está experimentando problemas de conectividad de red.

## <a name="common-error-scenarios"></a>Escenarios de error habituales

Los escenarios de error se clasifican como internos en el servicio y como externos en Windows Virtual Desktop.

* Problema interno: especifica escenarios que el cliente no puede mitigar y que deben resolverse como un problema de soporte técnico. Cuando aporte sus comentarios mediante la [comunidad de técnicos de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop), incluya el id. de correlación y el período de tiempo aproximado en que se produjo el problema.
* Problema externo: hace referencia a escenarios que el cliente puede mitigar. Son externos en Windows Virtual Desktop.

En la siguiente tabla se enumeran los errores comunes que los administradores pueden encontrarse.

>[!NOTE]
>Esta lista incluye los errores más comunes y se actualiza de manera periódica. Para estar seguro de que dispone de la información más actualizada, procure consultar en este artículo al menos una vez al mes.

## <a name="management-errors"></a>Errores de administración

|Mensaje de error|Solución propuesta|
|---|---|
|Failed to create registration key. (No se pudo crear la clave de registro). |No se pudo crear el token de registro. Pruebe a crearlo de nuevo con un tiempo de expiración más breve (entre 1 hora y 1 mes). |
|Failed to delete registration key. (No se pudo eliminar la clave de registro).|No se pudo eliminar el token de registro. Vuelva a intentar eliminarlo. Si sigue sin funcionar, use PowerShell para comprobar si el token sigue ahí. Si es así, elimínelo con PowerShell.|
|Failed to change session host drain mode. (No se pudo cambiar el modo de purga del host de sesión). |No se pudo cambiar el modo de purga en la VM. Compruebe el estado de la VM. Si no está disponible, no se puede cambiar el modo de purga.|
|Failed to disconnect user sessions. (No se pudieron desconectar las sesiones del usuario). |No se pudo desconectar el usuario de la VM. Compruebe el estado de la VM. Si no está disponible, la sesión de usuario no se puede desconectar. Si está disponible, compruebe el estado de la sesión de usuario para saber si está desconectado. |
|Failed to log off all user(s) within the session host. (No se pudo cerrar la sesión de todos los usuarios en el host de sesión). |No se pudo cerrar la sesión de los usuarios en la VM. Compruebe el estado de la VM. Si no está disponible, no se puede cerrar la sesión de los usuarios. Compruebe el estado de la sesión del usuario para averiguar si ya ha cerrado la sesión. Puede forzar el cierre de sesión con PowerShell. |
|Failed to unassign user from application group. (No se pudo cancelar la asignación del usuario del grupo de aplicaciones).|No se pudo anular la publicación de un grupo de aplicaciones para un usuario. Compruebe si el usuario está disponible en Azure AD. Consulte si el usuario forma parte de un grupo de usuarios en que está publicado el grupo de aplicaciones. |
|There was an error retrieving the available locations. (Error al recuperar las ubicaciones disponibles). |Compruebe la ubicación de la VM usada en el asistente para crear grupos de hosts. Si la imagen no está disponible en esa ubicación, agregue la imagen en la ubicación o elija otra ubicación de VM. |

### <a name="external-connection-error-codes"></a>Códigos de error de conexión externa

|Código numérico|Código de error|Solución propuesta|
|---|---|---|
|-2147467259|ConnectionFailedAdTrustedRelationshipFailure|El host de sesión no está unido correctamente a Active Directory.|
|-2146233088|ConnectionFailedUserHasValidSessionButRdshIsUnhealthy|Las conexiones no se pudieron establecer porque el host de sesión no está disponible. Compruebe el estado del host de sesión.|
|-2146233088|ConnectionFailedClientDisconnect|Si ve este error con frecuencia, asegúrese de que el equipo del usuario está conectado a la red.|
|-2146233088|ConnectionFailedNoHealthyRdshAvailable|La sesión a la que el usuario de host ha intentado conectarse no está en un estado adecuado. Depure la máquina virtual.|
|-2146233088|ConnectionFailedUserNotAuthorized|El usuario no tiene permiso para acceder al escritorio o la aplicación publicada. Este error puede aparecer después de que el administrador quite recursos publicados. Pida al usuario que actualice la fuente en la aplicación de Escritorio remoto.|
|2|FileNotFound|La aplicación a la que el usuario intentó tener acceso incorrectamente está instalada o establecida en una ruta de acceso incorrecta.<br><br>Al publicar nuevas aplicaciones mientras el usuario tiene una sesión activa, este no podrá acceder a esta aplicación. La sesión debe cerrarse y reiniciarse para que el usuario pueda acceder a la aplicación. |
|3|InvalidCredentials|El nombre de usuario o contraseña especificados por el usuario no coincide con el nombre de usuario o contraseña existentes. Revise las credenciales para ver si hay errores tipográficos e inténtelo de nuevo.|
|8|ConnectionBroken|La conexión entre el cliente y la puerta de enlace o servidor se ha interrumpido. No es necesario hacer nada a menos que esto ocurra de forma inesperada.|
|14|UnexpectedNetworkDisconnect|La conexión a la red se ha interrumpido. Pida al usuario que vuelva a conectarse.|
|24|ReverseConnectFailed|La máquina virtual host no tiene línea de visión directa a la puerta de enlace de Escritorio remoto. Asegúrese de que la dirección IP de la puerta de enlace se puede resolver.|

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los roles de Windows Virtual Desktop, consulte [Entorno de Windows Virtual Desktop](environment-setup.md).

Para ver una lista de cmdlets de PowerShell disponibles para Windows Virtual Desktop, vea la [referencia de PowerShell](/powershell/windows-virtual-desktop/overview).
