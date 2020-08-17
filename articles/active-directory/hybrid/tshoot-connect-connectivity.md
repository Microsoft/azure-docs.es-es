---
title: 'Azure AD Connect: solución de problemas de conectividad de Azure AD | Microsoft Docs'
description: Explica cómo solucionar problemas de conectividad con Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 3aa41bb5-6fcb-49da-9747-e7a3bd780e64
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 04/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 7bc39e409d0ac10e41fae58c5e5216f386427e30
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541743"
---
# <a name="troubleshoot-azure-ad-connectivity"></a>Solución de problemas de conectividad de Azure AD
Este artículo explica cómo funciona la conectividad entre Azure AD Connect y Azure AD y cómo solucionar los problemas de conectividad. Estos problemas suelen aparecer en un entorno con un servidor proxy.

## <a name="troubleshoot-connectivity-issues-in-the-installation-wizard"></a>Solución de problemas de conectividad en el asistente para la instalación
Azure AD Connect está utilizando autenticación moderna (mediante la biblioteca ADAL) para la autenticación. El Asistente para instalación y el motor de sincronización correspondiente requieren machine.config para configurarse correctamente ya que se trata de dos aplicaciones. NET.

En este artículo se explica cómo Fabrikam se conecta a Azure AD a través de su servidor proxy. El servidor proxy se llama fabrikamproxy y está usando el puerto 8080.

En primer lugar, debemos asegurarnos de que [**machine.config**](how-to-connect-install-prerequisites.md#connectivity) esté configurado correctamente y de que el **servicio de Microsoft Azure AD Sync** se haya reiniciado después de la actualización del archivo machine.config.
![machineconfig](./media/tshoot-connect-connectivity/machineconfig.png)

> [!NOTE]
> En algunos blogs que no son de Microsoft se indica que se deben realizar cambios en miiserver.exe.config. Sin embargo, este archivo se sobrescribe en cada actualización por lo que, aunque funcione durante la instalación inicial, el sistema deja de funcionar en la primera actualización. Por ese motivo, se recomienda actualizar el archivo machine.config en su lugar.
>
>

El servidor proxy también debe tener abiertas las direcciones URL necesarias. La lista oficial está documentada en direcciones [URL de Office 365 e intervalos de direcciones IP](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2).

De ellas, la tabla siguiente es el mínimo necesario para poder conectarse a Azure AD. Esta lista no incluye características opcionales, como la escritura diferida de contraseñas ni Azure AD Connect Health. Se documentan aquí para ayudar a solucionar problemas de la configuración inicial.

| URL | Port | Descripción |
| --- | --- | --- |
| mscrl.microsoft.com |HTTP/80 |Se usa para descargar listas CRL. |
| \*.verisign.com |HTTP/80 |Se usa para descargar listas CRL. |
| \*.entrust.net |HTTP/80 |Se utiliza para descargar listas CRL de MFA. |
| \*.windows.net |HTTPS/443 |Se utiliza para iniciar sesión en Azure AD. |
| secure.aadcdn.microsoftonline-p.com |HTTPS/443 |Se utiliza para MFA. |
| \*.microsoftonline.com |HTTPS/443 |Se utiliza para configurar el directorio de Azure AD, así como para importar y exportar datos. |

## <a name="errors-in-the-wizard"></a>Errores en el asistente
El asistente para la instalación usa dos contextos de seguridad diferentes. En la página **Conectar con Azure AD**, se utiliza el usuario que ha iniciado sesión actualmente. En la página **Configurar**, se cambia a la [cuenta que está ejecutando el servicio del motor de sincronización](reference-connect-accounts-permissions.md#adsync-service-account). Si hay algún error, lo más probable es que ya aparezca en la página **Conectar con Azure AD** del asistente, ya que la configuración del proxy es global.

Los siguientes errores son los más comunes que se pueden encontrar en el Asistente para instalación.

### <a name="the-installation-wizard-has-not-been-correctly-configured"></a>El asistente para la instalación no se ha configurado correctamente
Este error aparece cuando al propio asistente no puede establecer conexión con el proxy.
![nomachineconfig](./media/tshoot-connect-connectivity/nomachineconfig.png)

* Si aparece este error, verifique si [machine.config](how-to-connect-install-prerequisites.md#connectivity) se ha configurado correctamente.
* Si es correcta, siga los pasos descritos en [Comprobación de la conectividad de proxy](#verify-proxy-connectivity) para ver si el problema está presente también fuera del asistente.

### <a name="a-microsoft-account-is-used"></a>Se usa una cuenta Microsoft
Si utiliza una **cuenta Microsoft** en lugar de una cuenta **educativa o profesional**, aparece un error genérico.
![Se usa una cuenta Microsoft](./media/tshoot-connect-connectivity/unknownerror.png)

### <a name="the-mfa-endpoint-cannot-be-reached"></a>No se puede alcanzar el punto de conexión de MFA
Aparece este error si no se puede establecer comunicación con el punto de conexión **https://secure.aadcdn.microsoftonline-p.com** y el administrador global tiene MFA habilitado.
![nomachineconfig](./media/tshoot-connect-connectivity/nomicrosoftonlinep.png)

* Si ve este error, verifique si el punto de conexión **secure.aadcdn.microsoftonline-p.com** se ha agregado al proxy.

### <a name="the-password-cannot-be-verified"></a>No se puede comprobar la contraseña
Si el Asistente para instalación se conecta correctamente con Azure AD, pero la contraseña no se puede verificar, aparece lo siguiente: ![Contraseña incorrecta.](./media/tshoot-connect-connectivity/badpassword.png)

* ¿La contraseña es temporal y se debe cambiar? ¿Es realmente la contraseña correcta? Trate de iniciar sesión en `https://login.microsoftonline.com` (en un equipo distinto del servidor de Azure AD Connect) y compruebe que la cuenta se puede utilizar.

### <a name="verify-proxy-connectivity"></a>Comprobación de la conectividad de proxy
Para verificar si el servidor de Azure AD Connect tiene conectividad real con el servidor proxy e Internet, use PowerShell para ver si el servidor proxy permite solicitudes web o no. En un símbolo del sistema de PowerShell, ejecute `Invoke-WebRequest -Uri https://adminwebservice.microsoftonline.com/ProvisioningService.svc` (Técnicamente, la primera llamada es a `https://login.microsoftonline.com` y este identificador URI funciona también, pero el otro identificador URI responde más rápido).

PowerShell usa la configuración de machine.config para establecer conexión con el servidor proxy. La configuración de winhttp/netsh no debería afectar a estos cmdlets.

Si el servidor proxy está configurado correctamente, debe obtener un estado correcto: ![proxy200](./media/tshoot-connect-connectivity/invokewebrequest200.png)

Si recibe **No es posible conectar con el servidor remoto**, PowerShell está intentando realizar una llamada directa sin utilizar el servidor proxy o DNS no está configurado correctamente. Asegúrese de que el archivo **machine.config** está configurado correctamente.
![unabletoconnect](./media/tshoot-connect-connectivity/invokewebrequestunable.png)

Si el proxy no está configurado correctamente, se producirá un error: ![proxy200](./media/tshoot-connect-connectivity/invokewebrequest403.png)
![proxy407](./media/tshoot-connect-connectivity/invokewebrequest407.png)

| Error | Texto del error | Comentario |
| --- | --- | --- |
| 403 |Prohibido |No se abrió el servidor proxy para la dirección URL solicitada. Revise la configuración del servidor proxy y asegúrese de que las direcciones [URL](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) estén abiertas. |
| 407 |Se requiere autenticación del proxy |El servidor proxy requiere un inicio de sesión y no se ha proporcionado. Si el servidor proxy requiere autenticación, asegúrese de que esta opción está configurada en el archivo machine.config. Asegúrese también de que usa cuentas de dominio para el usuario que ejecuta el asistente y para la cuenta de servicio. |

### <a name="proxy-idle-timeout-setting"></a>Configuración del tiempo de espera de inactividad del proxy
Cuando Azure AD Connect envía una solicitud de exportación a Azure AD, Azure AD puede tardar hasta 5 minutos en procesar la solicitud antes de generar una respuesta. Esto puede ocurrir especialmente si hay un número de objetos de grupo con una gran pertenencia a grupos en la misma solicitud de exportación. Asegúrese de que el tiempo de espera de inactividad del proxy está configurado para que sean más de 5 minutos. En caso contrario,es posible que se observen problemas de conectividad intermitente con Azure AD en el servidor de Azure AD Connect.

## <a name="the-communication-pattern-between-azure-ad-connect-and-azure-ad"></a>Patrón de comunicación entre Azure AD Connect y Azure AD
Si ha seguido todos los pasos anteriores y aún no se puede conectar, en este momento puede comenzar a examinar los registros de red. En esta sección se documenta un patrón de conectividad normal y correcta. También se muestra una lista de pistas falsas habituales que se pueden omitir si se están leyendo los registros de red.

* Hay llamadas a `https://dc.services.visualstudio.com`. No es necesario tener abierta esta dirección URL en el proxy para que la instalación se realice correctamente, y estas llamadas pueden ignorarse.
* Observe que la resolución DNS enumera los hosts reales que están en el espacio de nombres DNS nsatc.net y otros espacios de nombres que no están en microsoftonline.com. Sin embargo, no hay solicitudes de servicio web en los nombres de servidor reales y no es necesario agregar estas direcciones URL al proxy.
* Los puntos de conexión adminwebservice y provisioningapi son los puntos de conexión de detección y se usan para buscar el punto de conexión real que se usará. Estos puntos de conexión difieren en función de la región.

### <a name="reference-proxy-logs"></a>Registros de proxy de referencia
Este es un volcado de un registro de proxy real y la página del asistente para instalación desde la que se tomó (se quitaron las entradas duplicadas al mismo punto de conexión). Esta sección se puede usar como referencia para su propio proxy y los registros de red. Los puntos de conexión reales pueden variar en su entorno (en concreto, las direcciones URL que están en *cursiva*).

**Conectarse a Azure AD**

| Time | URL |
| --- | --- |
| 11/1/2016 8:31 |connect://login.microsoftonline.com:443 |
| 11/1/2016 8:31 |connect://adminwebservice.microsoftonline.com:443 |
| 11/1/2016 8:32 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 11/1/2016 8:32 |connect://login.microsoftonline.com:443 |
| 11/1/2016 8:33 |connect://provisioningapi.microsoftonline.com:443 |
| 11/1/2016 8:33 |connect://*bwsc02-relay*.microsoftonline.com:443 |

**Configuración**

| Time | URL |
| --- | --- |
| 11/1/2016 8:43 |connect://login.microsoftonline.com:443 |
| 11/1/2016 8:43 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 11/1/2016 8:43 |connect://login.microsoftonline.com:443 |
| 11/1/2016 8:44 |connect://adminwebservice.microsoftonline.com:443 |
| 11/1/2016 8:44 |connect://*bba900-anchor*.microsoftonline.com:443 |
| 11/1/2016 8:44 |connect://login.microsoftonline.com:443 |
| 11/1/2016 8:44 |connect://adminwebservice.microsoftonline.com:443 |
| 11/1/2016 8:44 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 11/1/2016 8:44 |connect://login.microsoftonline.com:443 |
| 11/1/2016 8:46 |connect://provisioningapi.microsoftonline.com:443 |
| 11/1/2016 8:46 |connect://*bwsc02-relay*.microsoftonline.com:443 |

**Sincronización inicial**

| Time | URL |
| --- | --- |
| 11/1/2016 8:48 |connect://login.windows.net:443 |
| 11/1/2016 8:49 |connect://adminwebservice.microsoftonline.com:443 |
| 11/1/2016 8:49 |connect://*bba900-anchor*.microsoftonline.com:443 |
| 11/1/2016 8:49 |connect://*bba800-anchor*.microsoftonline.com:443 |

## <a name="authentication-errors"></a>Errores de autenticación
Esta sección trata los errores que pueden devolverse desde ADAL (la biblioteca de autenticación usada por Azure AD Connect) y PowerShell. El error que se explica debe ayudarle a comprender los pasos siguientes a seguir.

### <a name="invalid-grant"></a>Concesión no válida
Nombre de usuario o contraseña no válidos. Para más información, vea [No se puede verificar la contraseña](#the-password-cannot-be-verified).

### <a name="unknown-user-type"></a>Tipo de usuario desconocido
El directorio de Azure AD no se encuentra o no se ha resuelto. ¿Puede ser que haya intentado iniciar sesión con un nombre de usuario en un dominio sin comprobar?

### <a name="user-realm-discovery-failed"></a>Error de detección de dominio de usuario
Problemas de configuración de red o proxy. No se puede establecer conexión con la red. Vea [Solución de problemas de conectividad en el Asistente para instalación](#troubleshoot-connectivity-issues-in-the-installation-wizard).

### <a name="user-password-expired"></a>Contraseña de usuario expirada
Las credenciales han expirado. Cambie la contraseña.

### <a name="authorization-failure"></a>Error de autorización
No se pudo autorizar el usuario para realizar la acción en Azure AD.

### <a name="authentication-canceled"></a>Autenticación cancelada
Se canceló el desafío de la autenticación multifactor (MFA).

<div id="connect-msolservice-failed">
<!--
  Empty div just to act as an alias for the "Connect To MS Online Failed" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="connect-to-ms-online-failed"></a>Se produjo un error en la conexión a MS Online
La autenticación fue correcta, pero Azure AD PowerShell tiene un problema de autenticación.

<div id="get-msoluserrole-failed">
<!--
  Empty div just to act as an alias for the "Azure AD Global Admin Role Needed" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="azure-ad-global-admin-role-needed"></a>Se necesita el rol de administrador global de Azure AD
El usuario se autenticó correctamente. Sin embargo, el usuario no tiene asignado el rol de administrador global. Aquí se indica [cómo se puede asignar el rol de administrador global](../users-groups-roles/directory-assign-admin-roles.md) al usuario.

<div id="privileged-identity-management">
<!--
  Empty div just to act as an alias for the "Privileged Identity Management Enabled" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="privileged-identity-management-enabled"></a>Privileged Identity Management habilitado
Autenticación realizada correctamente. Se ha habilitado la administración de identidades con privilegios y en la actualidad usted no es un administrador global. Para más información, vea [Introducción a Privileged Identity Management](../privileged-identity-management/pim-getting-started.md).

<div id="get-msolcompanyinformation-failed">
<!--
  Empty div just to act as an alias for the "Company Information Unavailable" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="company-information-unavailable"></a>Información de la empresa no disponible
Autenticación realizada correctamente. No se pudo recuperar la información de la empresa de Azure AD.

<div id="get-msoldomain-failed">
<!--
  Empty div just to act as an alias for the "Domain Information Unavailable" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="domain-information-unavailable"></a>Información del dominio no disponible
Autenticación realizada correctamente. No se pudo recuperar la información de domino de Azure AD.

### <a name="unspecified-authentication-failure"></a>Error de autenticación no especificado
Se muestra como un error inesperado en el Asistente para instalación. Puede ocurrir si intenta utilizar una **cuenta Microsoft** en lugar de una cuenta **educativa o profesional**.

## <a name="troubleshooting-steps-for-previous-releases"></a>Pasos para solucionar problemas de versiones anteriores.
Con las versiones a partir del número de compilación 1.1.105.0 (publicado en febrero de 2016) se ha retirado el Ayudante para el inicio de sesión. Esta sección y la configuración no son ya necesarios, pero se conservan como referencia.

Para que el Ayudante para el inicio de sesión funcione, debe configurarse winhttp. Esta configuración puede realizarse con [**netsh**](how-to-connect-install-prerequisites.md#connectivity).
![netsh](./media/tshoot-connect-connectivity/netsh.png)

### <a name="the-sign-in-assistant-has-not-been-correctly-configured"></a>El Asistente para el inicio de sesión no se ha configurado correctamente
Este error aparece cuando el Ayudante para el inicio de sesión no puede conectar con el proxy o este no permite la solicitud.
![nonetsh](./media/tshoot-connect-connectivity/nonetsh.png)

* Si ve este error, examine la configuración de proxy en [netsh](how-to-connect-install-prerequisites.md#connectivity) y verifique si es correcta.
  ![netshshow](./media/tshoot-connect-connectivity/netshshow.png)
* Si es correcta, siga los pasos descritos en [Comprobación de la conectividad de proxy](#verify-proxy-connectivity) para ver si el problema está presente también fuera del asistente.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).
