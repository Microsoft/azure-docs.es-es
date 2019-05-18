---
title: Solución de problemas del proxy de aplicación | Microsoft Docs
description: Explica cómo solucionar errores en Proxy de aplicación de Azure AD.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/26/2018
ms.author: mimart
ms.reviewer: harshja
ms.custom: H1Hack27Feb2017; it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: dbdccf3b7a3ba1b8e55befa0fdc24eeff3e403da
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65782930"
---
# <a name="troubleshoot-application-proxy-problems-and-error-messages"></a>Solución de problemas y mensajes de error de Proxy de aplicación
Si se producen errores al obtener acceso a una aplicación publicada o al publicar aplicaciones, compruebe las siguientes opciones para ver si Proxy de aplicación de Microsoft Azure AD funciona correctamente:

* Abra la consola de Servicios de Windows y compruebe que el servicio **Microsoft AAD Application Proxy Connector** (Conector del proxy de aplicación de Microsoft AAD) está habilitado y en ejecución. Puede que también quiera consultar la página de propiedades del servicio Proxy de aplicación, como se muestra en la imagen siguiente:   
  ![Captura de pantalla de la ventana de propiedades del conector del proxy de aplicación de Microsoft AAD](./media/application-proxy-troubleshoot/connectorproperties.png)
* Abra el Visor de eventos y busque eventos del conector del proxy de aplicación en **Registros de aplicaciones y servicios** > **Microsoft** >  **AadApplicationProxy** > **Conector** > **Administrador**.
* Si los necesita, encontrará registros más detallados activando [los registros de sesiones del conector del proxy de aplicación](application-proxy-connectors.md#under-the-hood).

Para más información sobre la herramienta de solución de problemas de Azure AD, consulte [Troubleshooting tool to validate connector networking prerequisites](https://blogs.technet.microsoft.com/applicationproxyblog/2015/09/03/troubleshooting-tool-to-validate-connector-networking-prerequisites) (Herramienta de solución de problemas para validar los requisitos previos de red del conector).

## <a name="the-page-is-not-rendered-correctly"></a>La página no se representa correctamente.
Es posible que tenga problemas con la representación o el funcionamiento incorrecto de la aplicación, aunque no reciba ningún mensaje de error específico. Esto puede ocurrir si se publica la ruta de acceso del artículo, pero la aplicación requiere contenido que existe fuera de dicha ruta de acceso.

Por ejemplo, si publica la ruta de acceso `https://yourapp/app`, pero la aplicación llama a imágenes de `https://yourapp/media`, no se representarán. Asegúrese de publicar la aplicación con la ruta de nivel superior que debe incluir todo el contenido relevante. En este ejemplo, sería `http://yourapp/`.

Si cambia la ruta de acceso para incluir el contenido al que se hace referencia, pero necesita que los usuarios lleguen a un vínculo más profundo de la ruta de acceso, consulte la entrada del blog [Setting the right link for Application Proxy applications in the Azure AD access panel and Office 365 app launcher](https://blogs.technet.microsoft.com/applicationproxyblog/2016/04/06/setting-the-right-link-for-application-proxy-applications-in-the-azure-ad-access-panel-and-office-365-app-launcher/)(Establecimiento del vínculo correcto para las aplicaciones del proxy de aplicación en el panel de acceso de Azure AD y en el selector de programas de Office 365).

## <a name="connector-errors"></a>Errores del conector

Si se produce un error en el registro durante la instalación del Asistente para el conector, hay dos maneras de ver el motivo de dicho error. Se puede examinar el registro de eventos en **Registros de aplicaciones y servicios/\Microsoft\AadApplicationProxy\Connector\Admin**, o bien se puede ejecutar el siguiente comando de Windows PowerShell:

    Get-EventLog application –source "Microsoft AAD Application Proxy Connector" –EntryType "Error" –Newest 1

Una vez que encuentre el error del conector en el registro de eventos, use esta lista de errores comunes para resolver el problema:

| Error | Pasos recomendados |
| ----- | ----------------- |
| Error de registro del conector: asegúrese de haber habilitado Proxy de aplicación en el Portal de administración de Azure, así como de haber escrito correctamente su nombre de usuario y contraseña de Active Directory. Error: "Se han producido uno o varios errores". | Si cierra la ventana de registro sin iniciar sesión en Azure AD, vuelva a ejecutar el Asistente para conector y registre el conector. <br><br> Si se abre la ventana de registro y, a continuación, se cierra inmediatamente sin permitirle iniciar sesión, es probable que reciba este error. Este error se produce cuando hay un error de red en el sistema. Asegúrese de que es posible conectarse desde un explorador a un sitio web público y de que los puertos están abiertos como se especifica en [Requisitos previos del proxy de aplicación](application-proxy-add-on-premises-application.md). |
| Aparecerá un error de borrado en la ventana de registro. No se puede continuar | Si ve este error y luego se cierra la ventana, significa que escribió el nombre de usuario o contraseña incorrectos. Vuelva a intentarlo. |
| Error de registro del conector: asegúrese de haber habilitado Proxy de aplicación en el Portal de administración de Azure, así como de haber escrito correctamente su nombre de usuario y contraseña de Active Directory. Error: "AADSTS50059: No se ha producido ningún error en la información de identificación del inquilino en la solicitud o implícita en ninguna credencial proporcionada y búsqueda por identificador URI de la entidad de servicio. | Intenta iniciar sesión con una cuenta Microsoft y no un dominio que forma parte del id. de organización del directorio al que intenta tener acceso. Asegúrese de que el administrador forma parte del mismo nombre de dominio que el dominio del inquilino, por ejemplo, si el dominio de Azure AD es contoso.com, el administrador debe ser admin@contoso.com. |
| Error al recuperar la directiva de ejecución actual para ejecutar scripts de PowerShell. | Si se produce un error en la instalación del conector, compruebe que la directiva de ejecución de PowerShell no está deshabilitada. <br><br>1. Abra el Editor de directivas de grupo.<br>2. Vaya a **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows** > **Windows PowerShell** y haga doble clic en **Activar la ejecución de scripts**.<br>3. La directiva de ejecución puede definirse como **No configurado** o **Habilitado**. Si elige **Habilitado**, asegúrese de que, en Opciones, en Directiva de ejecución se selecciona en **Permitir scripts locales y scripts remotos firmados** o en **Permitir todos los scripts**. |
| El conector no pudo descargar la configuración. | El certificado de cliente del conector, que se utiliza para la autenticación, ha expirado. Esto también puede ocurrir si tiene instalado el conector detrás de un proxy. En este caso, el conector no puede acceder a Internet ni tampoco podrá proporcionar aplicaciones a usuarios remotos. Renueve la confianza manualmente mediante el cmdlet `Register-AppProxyConnector` en Windows PowerShell. Si el conector está detrás de un proxy, es preciso otorgar acceso a Internet a las cuentas del conector "servicios de red" y "sistema local". Esto puede realizarse concediéndoles acceso al proxy o estableciéndoles para omitir el proxy. |
| Error de registro del conector: asegúrese de que es administrador de aplicaciones de Active Directory para registrar el conector. Error: "Se ha denegado la solicitud de registro". | El alias con el que intenta iniciar sesión no es administrador en este dominio. Su conector siempre está instalado para el directorio que posee el dominio del usuario. Asegúrese de que la cuenta de administrador con la que intenta iniciar sesión tiene, al menos, permisos de administrador de aplicaciones en el inquilino de Azure AD. |

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
| El sitio web no puede mostrar la página. | Es posible que el usuario reciba este error al intentar acceder a la aplicación que publicó esta es una aplicación OWA. Esto puede deberse a uno de los motivos siguientes: <br><li>El SPN definido para esta aplicación es incorrecto. Asegúrese de que el SPN configurado para esta aplicación es correcto.</li><li>El usuario que intentó obtener acceso a la aplicación usa una cuenta Microsoft en lugar de la cuenta corporativa adecuada para iniciar sesión, o bien el usuario es un usuario invitado. Asegúrese de que el usuario inicia sesión con la cuenta corporativa que coincide con el dominio de la aplicación publicada. Los usuarios de cuenta Microsoft y los invitados no pueden tener acceso a aplicaciones IWA.</li><li>El usuario que intentó obtener acceso a la aplicación no se define correctamente para esta aplicación en el lado local de. Asegúrese de que este usuario tiene los permisos adecuados, tal como se define para esta aplicación de back-end en la máquina local en. |
| No se puede tener acceso a esta aplicación corporativa. No está autorizado para tener acceso a esta aplicación. Error de autorización. Asegúrese de asignar el usuario con acceso a esta aplicación. | El usuario puede obtener este error al intentar acceder a la aplicación publicada si usan cuentas de Microsoft en lugar de su cuenta corporativa al iniciar sesión. Los usuarios invitados también pueden recibir este error. Los usuarios y los invitados de la Cuenta Microsoft no pueden tener acceso a aplicaciones IWA. Asegúrese de que el usuario inicia sesión con la cuenta corporativa que coincide con el dominio de la aplicación publicada.<br><br>Puede que no haya asignado el usuario para esta aplicación. Vaya a la pestaña **Aplicación** y, en **Usuarios y grupos**, asigne este usuario o grupo de usuarios a esta aplicación. |
| No se puede tener acceso a esta aplicación corporativa en este momento. Inténtelo de nuevo más tarde... Se agotó el tiempo de espera del conector. | El usuario puede obtener este error al intentar acceder a la aplicación que publicó si no están definidos correctamente para esta aplicación en el lado local. Asegúrese de que los usuarios tengan los permisos adecuados, tal como se define para esta aplicación de back-end en la máquina local en. |
| No se puede tener acceso a esta aplicación corporativa. No está autorizado para tener acceso a esta aplicación. Error de autorización. Asegúrese de que el usuario tiene una licencia de Azure Active Directory Premium o Basic. | El usuario puede obtener este error al intentar acceder a la aplicación que publicó si no les asignó explícitamente con una licencia Premium o Basic por el administrador del suscriptor. Vaya a la pestaña **Licencias** de Active Directory del suscriptor y asegúrese de que se asigne a este usuario o grupo de usuarios una licencia Premium o Básica. |
| No se encontró un servidor con el nombre de host especificado. | El usuario puede obtener este error al intentar acceder a la aplicación publicada si el dominio personalizado de la aplicación no está configurado correctamente. Asegúrese de que ha cargado un certificado para el dominio y la correcta el registro DNS siguiendo los pasos descritos en [trabajar con dominios personalizados en Azure AD Application Proxy](application-proxy-configure-custom-domain.md) |

## <a name="my-error-wasnt-listed-here"></a>Mi error no aparece aquí.

Si se produce un error o un problema con el Proxy de aplicación de Azure AD que no aparece en esta guía de solución de problemas, es conveniente obtener información al respecto. Envíe un correo electrónico al [equipo de comentarios](mailto:aadapfeedback@microsoft.com) con los detalles del error que se ha producido.

## <a name="see-also"></a>Vea también
* [Habilitación del proxy de aplicación de Azure Active Directory](application-proxy-add-on-premises-application.md)
* [Publicar aplicaciones con Proxy de aplicación](application-proxy-add-on-premises-application.md)
* [Habilitar el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md)
* [Habilitar el acceso condicional](application-proxy-integrate-with-sharepoint-server.md)


<!--Image references-->
[1]: ./media/application-proxy-troubleshoot/connectorproperties.png
[2]: ./media/active-directory-application-proxy-troubleshoot/sessionlog.png
