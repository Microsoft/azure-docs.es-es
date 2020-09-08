---
title: Privacidad del usuario y autenticación de paso a través de Azure Active Directory | Microsoft Docs
description: Este artículo trata sobre la autenticación de paso a través de Azure Active Directory (Azure AD) y el cumplimiento del RGPD.
services: active-directory
keywords: Autenticación de paso a través de Azure AD Connect, RGPD, componentes necesarios para Azure AD, SSO, inicio de sesión único
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/23/2018
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 396344ba90aa3850d7d23dc40d6df95f6d1f6c3f
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279540"
---
# <a name="user-privacy-and-azure-active-directory-pass-through-authentication"></a>Privacidad del usuario y autenticación de paso a través de Azure Active Directory


[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

## <a name="overview"></a>Información general

La autenticación paso a través de Azure AD crea el tipo de registro siguiente, que puede contener datos personales:

- Archivos de registro de seguimiento de Azure AD Connect.
- Archivos de registro de seguimiento del agente de autenticación.
- Archivos de registro de eventos de Windows.

Puede mejorar la privacidad del usuario para la autenticación de paso de dos maneras:

1.  Previa solicitud, extraer los datos de una persona y quitarlos de las instalaciones.
2.  Asegurarse de que ningún dato se conserva más de 48 horas.

Se recomienda encarecidamente la segunda opción, porque es más simple de implementar y mantener. A continuación, aparecen las instrucciones para cada tipo de registro:

### <a name="delete-azure-ad-connect-trace-log-files"></a>Eliminación de archivos de registro de seguimiento de Azure AD Connect

Revise el contenido de la carpeta **%ProgramData%\AADConnect** y elimine el contenido de registro de seguimiento (archivos **trace-\*.log**) de esta carpeta en un plazo de 48 horas a contar de la instalación o la instalación de Azure AD Connect o de la modificación de la configuración de la autenticación de paso a través, porque esta acción podría crear los datos que cubre el RGPD.

>[!IMPORTANT]
>No elimine el archivo **PersistedState.xml** de esta carpeta, porque este archivo se usa para mantener el estado de la instalación anterior de Azure AD Connect y se usa cuando se realiza una instalación de actualización. Este archivo nunca contendrá ningún dato sobre una persona y nunca deberá eliminarse.

Puede revisar y eliminar estos archivos de registro de seguimiento con el Explorador de Windows, o bien puede usar el script de PowerShell siguiente para realizar las acciones necesarias:

```
$Files = ((Get-Item -Path "$env:programdata\aadconnect\trace-*.log").VersionInfo).FileName 
 
Foreach ($file in $Files) { 
    {Remove-Item -Path $File -Force} 
}
```

Guarde el script en un archivo con la extensión ".PS1". Ejecute este script según sea necesario.

Para más información sobre los requisitos relacionados del RGPD de Azure AD Connect, consulte [este artículo](reference-connect-user-privacy.md).

### <a name="delete-authentication-agent-event-logs"></a>Eliminación de registros de eventos del agente de autenticación

Este producto también puede crear **registros de eventos de Windows**. Para más información, lea [este artículo](/windows/win32/wes/windows-event-log).

Para ver los registros relacionados con el agente de autenticación de paso a través, abra la aplicación **Visor de eventos** en el servidor y consulte **Application and Service Logs\Microsoft\AzureAdConnect\AuthenticationAgent\Admin**.

### <a name="delete-authentication-agent-trace-log-files"></a>Eliminación de archivos de registro de seguimiento del agente de autenticación.

Debe comprobar con regularidad el contenido de **%ProgramData%\Microsoft\Azure AD Connect Authentication Agent\Trace** y eliminar el contenido de esta carpeta cada 48 horas. 

>[!IMPORTANT]
>Si el servicio del agente de autenticación está en ejecución, no podrá eliminar el archivo de registro actual en la carpeta. Detenga el servicio antes de volver a intentarlo. Para evitar errores de inicio de sesión de usuario, ya debe haber configurado la autenticación de paso a través para una [alta disponibilidad](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability).

Estos archivos se pueden revisar y eliminar mediante el Explorador de Windows, o bien puede usar el script siguiente para realizar las acciones necesarias:

```
$Files = ((Get-childitem -Path "$env:programdata\microsoft\azure ad connect authentication agent\trace" -Recurse).VersionInfo).FileName 
 
Foreach ($file in $files) { 
    {Remove-Item -Path $File -Force} 
}
```

Para programar que este script se ejecute cada 48 horas, siga estos pasos:

1.  Guarde el script en un archivo con la extensión ".PS1".
2.  Abra el **panel de control** y haga clic en **Sistema y seguridad**.
3.  En el encabezado **Herramientas administrativas**, haga clic en "**Programar tareas**".
4.  En el **Programador de tareas**, haga clic con el botón derecho en "**Biblioteca del Programador de tareas**" y haga clic en "**Crear tarea básica…** ".
5.  Escriba el nombre de la nueva tarea y haga clic en **Siguiente**.
6.  Seleccione "**Diariamente**" para el **desencadenador de tareas** y haga clic en **Siguiente**.
7.  Establezca la repetición en dos días y haga clic en **Siguiente**.
8.  Seleccione "**Iniciar un programa**" como la acción y haga clic en **Siguiente**.
9.  Escriba "**PowerShell**" en el cuadro Programa o script y, en el cuadro "**Agregar argumentos (opcional)** ", escriba la ruta de acceso completa al script que creó anteriormente y, luego, haga clic en **Siguiente**.
10. En la siguiente pantalla se muestra un resumen de la tarea que se dispone a crear. Compruebe los valores y haga clic en **Finalizar** para crear la tarea:
 
### <a name="note-about-domain-controller-logs"></a>Nota sobre los registros de controlador de dominio

Si el registro de auditoría está habilitado, este producto puede generar registros de seguridad para los controladores de dominio. Para más información sobre cómo configurar las directivas de auditoría, lea este [artículo](/previous-versions/tn-archive/dd277403(v=technet.10)).

## <a name="next-steps"></a>Pasos siguientes
* [Revisar la directiva de privacidad de Microsoft en Trust Center](https://www.microsoft.com/trustcenter)
* [**Solución de problemas**](tshoot-connect-pass-through-authentication.md): información para resolver problemas habituales de esta característica.