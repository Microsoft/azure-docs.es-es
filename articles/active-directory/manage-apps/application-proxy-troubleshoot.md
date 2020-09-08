---
title: Solución de problemas del proxy de aplicación | Microsoft Docs
description: Explica cómo solucionar errores en Proxy de aplicación de Azure AD.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 06/24/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 413cfe4f3aed446ad26a210b4faa452c4f624685
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88640861"
---
# <a name="troubleshoot-application-proxy-problems-and-error-messages"></a>Solución de problemas y mensajes de error de Proxy de aplicación

Al solucionar problemas de Application Proxy, se recomienda comenzar por revisar el flujo de solución de problemas, [Depuración de problemas del conector de proxy de aplicación](application-proxy-debug-connectors.md), para determinar si los conectores de Application Proxy están configurados correctamente. Si sigue teniendo problemas para conectarse a la aplicación, siga el flujo de solución de problemas en el artículo sobre la [depuración de problemas de aplicación de Application Proxy](application-proxy-debug-apps.md).

Si se producen errores al obtener acceso a una aplicación publicada o al publicar aplicaciones, compruebe las siguientes opciones para ver si Proxy de aplicación de Microsoft Azure AD funciona correctamente:

* Abra la consola de Servicios de Windows. Compruebe que el servicio **Microsoft AAD Application Proxy Connector** está habilitado y en ejecución. Puede que también quiera consultar la página de propiedades del servicio Proxy de aplicación, como se muestra en la imagen siguiente:  
  ![Captura de pantalla de la ventana de propiedades del conector del proxy de aplicación de Microsoft AAD](./media/application-proxy-troubleshoot/connectorproperties.png)
* Abra el Visor de eventos y busque eventos del conector del proxy de aplicación en **Registros de aplicaciones y servicios** > **Microsoft** >  **AadApplicationProxy** > **Conector** > **Administrador**.
* Si los necesita, encontrará registros más detallados activando [los registros de sesiones del conector del proxy de aplicación](application-proxy-connectors.md#under-the-hood).

## <a name="the-page-is-not-rendered-correctly"></a>La página no se representa correctamente.
Es posible que tenga problemas con la representación o el funcionamiento incorrecto de la aplicación, aunque no reciba ningún mensaje de error específico. Esto puede ocurrir si se publica la ruta de acceso del artículo, pero la aplicación requiere contenido que existe fuera de dicha ruta de acceso.

Por ejemplo, si publica la ruta de acceso `https://yourapp/app`, pero la aplicación llama a imágenes de `https://yourapp/media`, no se representarán. Asegúrese de publicar la aplicación con la ruta de nivel superior que debe incluir todo el contenido relevante. En este ejemplo, sería `http://yourapp/`.

## <a name="connector-errors"></a>Errores del conector

Si se produce un error en el registro durante la instalación del Asistente para el conector, hay dos maneras de ver el motivo de dicho error. Se puede examinar el registro de eventos en **Registros de aplicaciones y servicios/\Microsoft\AadApplicationProxy\Connector\Admin**, o bien se puede ejecutar el siguiente comando de Windows PowerShell:

```powershell
Get-EventLog application –source "Microsoft AAD Application Proxy Connector" –EntryType "Error" –Newest 1
```

Una vez que encuentre el error del conector en el registro de eventos, use esta lista de errores comunes para resolver el problema:

| Error | Pasos recomendados |
| ----- | ----------------- |
| Error de registro del conector: asegúrese de haber habilitado Proxy de aplicación en el Portal de administración de Azure, así como de haber escrito correctamente su nombre de usuario y contraseña de Active Directory. Error: "Se han producido uno o varios errores". | Si cierra la ventana de registro sin iniciar sesión en Azure AD, vuelva a ejecutar el Asistente para conector y registre el conector. <br><br> Si se abre la ventana de registro y se cierra inmediatamente sin permitirle iniciar sesión, es probable que reciba este error. Este error se produce cuando hay un error de red en el sistema. Asegúrese de que es posible conectarse desde un explorador a un sitio web público y de que los puertos están abiertos como se especifica en [Requisitos previos del proxy de aplicación](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment). |
| Aparecerá un error de borrado en la ventana de registro. No se puede continuar | Si ve este error y luego se cierra la ventana, significa que escribió el nombre de usuario o contraseña incorrectos. Inténtelo de nuevo. |
| Error de registro del conector: asegúrese de haber habilitado Proxy de aplicación en el Portal de administración de Azure, así como de haber escrito correctamente su nombre de usuario y contraseña de Active Directory. Error: "AADSTS50059: No se ha producido ningún error en la información de identificación del inquilino en la solicitud o implícita en ninguna credencial proporcionada y búsqueda por identificador URI de la entidad de servicio. | Intenta iniciar sesión con una cuenta Microsoft, no un dominio que forma parte del Id. de organización del directorio al que intenta acceder. Asegúrese de que el administrador forma parte del mismo nombre de dominio que el dominio del inquilino, por ejemplo, si el dominio de Azure AD es contoso.com, el administrador debe ser admin@contoso.com. |
| Error al recuperar la directiva de ejecución actual para ejecutar scripts de PowerShell. | Si se produce un error en la instalación del conector, compruebe que la directiva de ejecución de PowerShell no está deshabilitada. <br><br>1. Abra el Editor de directivas de grupo.<br>2. Vaya a **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows** > **Windows PowerShell** y haga doble clic en **Activar la ejecución de scripts**.<br>3. La directiva de ejecución puede definirse como **No configurado** o **Habilitado**. Si elige **Habilitado**, asegúrese de que, en Opciones, en Directiva de ejecución se selecciona en **Permitir scripts locales y scripts remotos firmados** o en **Permitir todos los scripts**. |
| El conector no pudo descargar la configuración. | El certificado de cliente del conector, que se utiliza para la autenticación, ha expirado. Esto también puede ocurrir si tiene instalado el conector detrás de un proxy. En este caso, el conector no puede acceder a Internet ni tampoco podrá proporcionar aplicaciones a usuarios remotos. Renueve la confianza manualmente mediante el cmdlet `Register-AppProxyConnector` en Windows PowerShell. Si el conector está detrás de un proxy, es preciso otorgar acceso a Internet a las cuentas del conector "servicios de red" y "sistema local". Esto puede realizarse concediéndoles acceso al proxy o estableciéndoles para omitir el proxy. |
| Error de registro del conector: asegúrese de que es administrador de aplicaciones de Active Directory para registrar el conector. Error: "Se ha denegado la solicitud de registro". | El alias con el que intenta iniciar sesión no es administrador en este dominio. Su conector siempre está instalado para el directorio que posee el dominio del usuario. Asegúrese de que la cuenta de administrador con la que intenta iniciar sesión tiene, al menos, permisos de administrador de aplicaciones en el inquilino de Azure AD. |
| El conector no pudo conectarse al servicio debido a problemas de red. El conector intentó acceder a la siguiente dirección URL. | El conector no se puede conectar al servicio de nube de Application Proxy. Esto puede ocurrir si hay una regla de firewall que bloquea la conexión. Asegúrese de haber permitido el acceso a los puertos y direcciones URL correctos que aparecen en los [requisitos previos de Application Proxy](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment). |

## <a name="kerberos-errors"></a>Errores de Kerberos

En esta tabla se incluyen los errores más comunes de instalación y configuración de Kerberos y se realizan sugerencias para la resolución.

| Error | Pasos recomendados |
| ----- | ----------------- |
| Error al recuperar la directiva de ejecución actual para ejecutar scripts de PowerShell. | Si se produce un error en la instalación del conector, compruebe que la directiva de ejecución de PowerShell no está deshabilitada.<br><br>1. Abra el Editor de directivas de grupo.<br>2. Vaya a **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows** > **Windows PowerShell** y haga doble clic en **Activar la ejecución de scripts**.<br>3. La directiva de ejecución puede definirse como **No configurado** o **Habilitado**. Si elige **Habilitado**, asegúrese de que, en Opciones, en Directiva de ejecución se selecciona en **Permitir scripts locales y scripts remotos firmados** o en **Permitir todos los scripts**. |
| 12008: Azure AD superó el número máximo de intentos de autenticación Kerberos permitidos en el servidor backend. | Este error puede indicar una configuración incorrecta entre Azure AD y el servidor de aplicaciones backend o un problema en la configuración de fecha y hora de ambos equipos. El servidor backend rechazó el vale Kerberos creado por Azure AD. Compruebe que tanto Azure AD como el servidor de aplicaciones back-end están configurados correctamente. Asegúrese de que la configuración de fecha y hora de Azure AD y el servidor de aplicaciones backend está sincronizada. |
| 13016: Azure AD no puede recuperar un vale Kerberos en nombre del usuario porque no hay ningún UPN en el token perimetral o en la cookie de acceso. | Hay un problema con la configuración de STS. Corrija la configuración de notificación de UPN en STS. |
| 13019: Azure AD no puede recuperar un vale Kerberos en nombre del usuario debido al siguiente error general API. | Este evento puede indicar una configuración incorrecta entre Azure AD y el servidor controlador de dominios o un problema en la configuración de fecha y hora de ambos equipos. El controlador de dominio rechazó el vale Kerberos creado por Azure AD. Compruebe que tanto Azure AD como el servidor de aplicaciones back-end están configurados correctamente, especialmente la configuración de SPN. Asegúrese de que Azure AD está unido mediante dominio al mismo dominio que el controlador de dominio para garantizar que este establezca la confianza con Azure AD. Asegúrese de que la configuración de fecha y hora de Azure AD y el controlador de dominio está sincronizada. |
| 13020: Azure AD no puede recuperar un vale Kerberos en nombre del usuario al no estar definido el SPN del servidor backend. | Este evento puede indicar una configuración incorrecta entre Azure AD y el servidor controlador de dominios o un problema en la configuración de fecha y hora de ambos equipos. El controlador de dominio rechazó el vale Kerberos creado por Azure AD. Compruebe que tanto Azure AD como el servidor de aplicaciones back-end están configurados correctamente, especialmente la configuración de SPN. Asegúrese de que Azure AD está unido mediante dominio al mismo dominio que el controlador de dominio para garantizar que este establezca la confianza con Azure AD. Asegúrese de que la configuración de fecha y hora de Azure AD y el controlador de dominio está sincronizada. |
| 13022: Azure AD no puede autenticar el usuario porque el servidor backend responde a los intentos de autenticación Kerberos con un error HTTP 401. | Este evento puede indicar una configuración incorrecta entre Azure AD y el servidor de aplicaciones backend o un problema en la configuración de fecha y hora de ambos equipos. El servidor backend rechazó el vale Kerberos creado por Azure AD. Compruebe que tanto Azure AD como el servidor de aplicaciones back-end están configurados correctamente. Asegúrese de que la configuración de fecha y hora de Azure AD y el servidor de aplicaciones backend está sincronizada. Para obtener más información, consulte [Solucionar problemas de las configuraciones de delegación limitada de Kerberos para el proxy de aplicación](application-proxy-back-end-kerberos-constrained-delegation-how-to.md).  |

## <a name="end-user-errors"></a>Errores del usuario final

En esta lista se muestran los errores que los usuarios finales pueden encontrar al intentar acceder a la aplicación. 

| Error | Pasos recomendados |
| ----- | ----------------- |
| El sitio web no puede mostrar la página. | Es posible que el usuario reciba este error al intentar acceder a la aplicación que publicó esta es una aplicación IWA. Es posible que el SPN definido para esta aplicación no sea correcto. Para las aplicaciones IWA: asegúrese de que el SPN configurado para esta aplicación es correcto. |
| El sitio web no puede mostrar la página. | Es posible que el usuario reciba este error al intentar acceder a la aplicación que publicó esta es una aplicación OWA. Esto puede deberse a uno de los motivos siguientes:<br><li>El SPN definido para esta aplicación es incorrecto. Asegúrese de que el SPN configurado para esta aplicación es correcto.</li><li>El usuario que intentó obtener acceso a la aplicación usa una cuenta Microsoft en lugar de la cuenta corporativa adecuada para iniciar sesión, o bien el usuario es un usuario invitado. Asegúrese de que el usuario inicia sesión con la cuenta corporativa que coincide con el dominio de la aplicación publicada. Los usuarios de cuenta Microsoft y los invitados no pueden tener acceso a aplicaciones IWA.</li><li>El usuario que intentó acceder a la aplicación no está correctamente definido para esta aplicación a nivel local. Asegúrese de que este usuario tiene los permisos adecuados como se define para esta aplicación back-end en el equipo local. |
| No se puede tener acceso a esta aplicación corporativa. No está autorizado para tener acceso a esta aplicación. Error de autorización. Asegúrese de asignar el usuario con acceso a esta aplicación. | El usuario puede recibir este error al intentar acceder a la aplicación que publicó si para iniciar sesión usa cuentas de Microsoft, en lugar de su cuenta corporativa. Los usuarios invitados también pueden recibir este error. Los usuarios y los invitados de la Cuenta Microsoft no pueden tener acceso a aplicaciones IWA. Asegúrese de que el usuario inicia sesión con la cuenta corporativa que coincide con el dominio de la aplicación publicada.<br><br>Puede que no haya asignado el usuario para esta aplicación. Vaya a la pestaña **Aplicación** y, en **Usuarios y grupos**, asigne este usuario o grupo de usuarios a esta aplicación. |
| No se puede tener acceso a esta aplicación corporativa en este momento. Inténtelo de nuevo más tarde... Se agotó el tiempo de espera del conector. | El usuario puede recibir este error al intentar acceder a la aplicación que publicó si no está definido correctamente para esta aplicación a nivel local. Asegúrese de que los usuarios tengan los permisos adecuados definidos para esta aplicación back-end en el equipo local. |
| No se puede tener acceso a esta aplicación corporativa. No está autorizado para tener acceso a esta aplicación. Error de autorización. Asegúrese de que el usuario tiene una licencia de Azure Active Directory Premium. | Es posible que el usuario reciba este error al intentar acceder a la aplicación que publicó si el administrador del suscriptor no les asignó explícitamente una licencia Premium. Vaya a la pestaña **Licencias** de Active Directory del suscriptor y asegúrese de que se asigne a este usuario o grupo de usuarios una licencia Premium. |
| No se encontró un servidor con el nombre de host especificado. | Es posible que el usuario reciba este error al intentar acceder a la aplicación que publicó si el dominio personalizado de la aplicación no está configurado correctamente. Asegúrese de haber cargado un certificado para el dominio y haber configurado correctamente el registro DNS siguiendo los pasos descritos en [Uso de dominios personalizados en el proxy de la aplicación de Azure AD](application-proxy-configure-custom-domain.md) |
|Prohibido: This corporate app can't be accessed OR The user could not be authorized. Make sure the user is defined in your on-premises AD and that the user has access to the app in your on-premises AD (Prohibido: No se puede acceder a esta aplicación corporativa o no se pudo autorizar al usuario. Asegúrese de que el usuario está definido en la instancia de AD local y de que tenga acceso a la aplicación en dicha instancia). | Podría haber un problema con el acceso a la información de autorización. Consulte [Algunas aplicaciones y API requieren acceso a información de autorización en los objetos de cuenta]( https://support.microsoft.com/help/331951/some-applications-and-apis-require-access-to-authorization-information). En resumen, agregue la cuenta de máquina del conector del proxy de aplicación al grupo de dominio integrado "Grupo de acceso de autorización de Windows" para resolverlo. |

## <a name="my-error-wasnt-listed-here"></a>Mi error no aparece aquí.

Si se produce un error o un problema con el Proxy de aplicación de Azure AD que no aparece en esta guía de solución de problemas, es conveniente obtener información al respecto. Envíe un correo electrónico al [equipo de comentarios](mailto:aadapfeedback@microsoft.com) con los detalles del error que se ha producido.

## <a name="see-also"></a>Consulte también
* [Habilitación del proxy de aplicación de Azure Active Directory](application-proxy-add-on-premises-application.md)
* [Publicar aplicaciones con Proxy de aplicación](application-proxy-add-on-premises-application.md)
* [Habilitar el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md)
* [Habilitar el acceso condicional](application-proxy-integrate-with-sharepoint-server.md)


<!--Image references-->
[1]: ./media/application-proxy-troubleshoot/connectorproperties.png
[2]: ./media/active-directory-application-proxy-troubleshoot/sessionlog.png
