---
title: Instalación de la extensión del explorador del panel de acceso de la aplicación - Azure| Microsoft Docs
description: Corrección de los errores comunes encontrados al instalar la extensión del explorador del panel de acceso.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 5/4/2018
ms.author: mimart
ms.reviewer: japere,asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1973bc647b0e3c7fc453009f638b770e171132fc
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784257"
---
# <a name="install-the-access-panel-browser-extension"></a>Instalación de la extensión del explorador del panel de acceso

El panel de acceso es un portal web. Si tiene una cuenta profesional o educativa en Azure Active Directory (Azure AD), puede utilizar el panel de acceso para ver e iniciar las aplicaciones basadas en la nube a las que el administrador de Azure AD le ha concedido acceso. 

Si utiliza ediciones de Azure AD, también puede usar las funcionalidades de administración de grupos de autoservicio y aplicaciones mediante el panel de acceso. 

El panel de acceso es independiente de Azure Portal. No es necesario tener una suscripción a Azure.

## <a name="web-browser-requirements"></a>Requisitos del explorador web

Como mínimo, el panel de acceso requiere un explorador que admita JavaScript y que tenga CSS habilitado. Para iniciar sesión en las aplicaciones mediante el inicio de sesión único (SSO) basado en contraseña en el panel de acceso, debe instalar una extensión del panel de acceso en el explorador. Esta extensión se descarga automáticamente cuando un usuario selecciona una aplicación que está configurada para el inicio de sesión único basado en contraseña.

Para el inicio de sesión único basado en contraseña, puede usar cualquiera de los siguientes exploradores:

- **Microsoft Edge**: en Windows 10 Anniversary Edition o versión posterior. 
- **Chrome**: en Windows 7 o versiones posteriores y en MacOS X o versiones posteriores.
- **Firefox 26.0 o versiones posteriores**: en Windows XP SP2 o versiones posteriores y en Mac OS X 10.6 o versiones posteriores.

## <a name="install-the-access-panel-browser-extension"></a>Instalación de la extensión del explorador del panel de acceso

Para instalar la extensión del explorador del panel de acceso, haga lo siguiente:

1.  En uno de los exploradores admitidos, abra el [panel de acceso](https://myapps.microsoft.com) e inicie sesión como usuario en su cuenta de Azure AD.

2.  Seleccione una aplicación con inicio de sesión único basado en contraseña.

3.  Cuando se le pida, seleccione **Instalar ahora**.  
    Se le dirigirá al vínculo de descarga para el explorador seleccionado. 
    
4.  Seleccione **Agregar**.

5.  Si se le solicita, puede **Habilitar** o **Permitir** la extensión.

6.  Una vez completada la instalación, reinicie el explorador.

7.  Inicie sesión en el panel de acceso y compruebe si puede iniciar las aplicaciones con inicio de sesión único basado en contraseña.

También puede descargar la extensión para Chrome y Microsoft Edge directamente desde los siguientes sitios:

- [Extensión para Chrome](https://chrome.google.com/webstore/detail/access-panel-extension/ggjhpefgjjfobnfoldnjipclpcfbgbhl)
- [Extensión para Microsoft Edge](https://www.microsoft.com/store/apps/9pc9sckkzk84)
- [Extensión de Firefox](https://addons.mozilla.org/en-US/firefox/addon/access-panel-extension/)

## <a name="use-the-my-apps-secure-sign-in-extension"></a>Uso de la extensión de inicio de sesión seguro de Mis aplicaciones
* Si va utilizar una dirección URL de Mis aplicaciones que no sea `https://myapps.microsoft.com`, configure la dirección URL predeterminada siguiendo estos pasos:
   1. *Sin* iniciar sesión en la extensión, haga clic con el botón derecho en el icono de la extensión.
   2. En el menú, seleccione **My Apps URL** (URL de Mis aplicaciones).
   3. Seleccione la dirección URL predeterminada.
   4. Seleccione el icono de la extensión.
   5. Para iniciar sesión en la extensión, seleccione **Iniciar sesión para comenzar**.

* Para iniciar sesión directamente en una aplicación desde el explorador, haga lo siguiente:
   1. Después de instalar la extensión, seleccione **Iniciar sesión para comenzar** para iniciar sesión.
   2. Inicie sesión en la aplicación con la dirección URL de inicio de sesión.  
       La dirección URL de inicio de sesión suele ser la dirección URL de la aplicación que muestra el formulario de inicio de sesión.
      La extensión debe cambiar el estado e indicar que hay una contraseña disponible.
   3. Para iniciar sesión, seleccione el icono de la extensión.

* Para iniciar una aplicación desde la extensión, haga lo siguiente:
   1. Después de instalar la extensión, seleccione **Iniciar sesión para comenzar** para iniciar sesión.
   2. Seleccione el icono de extensión para abrir su menú.
   3. Busque una aplicación que esté disponible en el portal Mis aplicaciones.
   4. En la lista de resultados de la búsqueda, seleccione la aplicación.  
       Las tres últimas aplicaciones que ha utilizado se muestran en la lista de accesos directos **Usados recientemente**.
       
* Para usar direcciones URL internas de la empresa de forma remota, haga lo siguiente:
    1. [Configure el proxy de aplicación](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-enable) en el inquilino
    2. [Publique la aplicación](https://docs.microsoft.com/azure/active-directory/application-proxy-publish-azure-portal) y la dirección URL a través del proxy de aplicación
    3. Instale la extensión y seleccione Iniciar sesión para comenzar para iniciar sesión
    4. Ahora podrá examinar la dirección URL interna de la empresa de manera remota

> [!NOTE]
> Las opciones anteriores están disponibles solo para Microsoft Edge, Chrome y Firefox.

## <a name="set-up-a-group-policy-for-internet-explorer"></a>Configuración de una directiva de grupo para Internet Explorer

Puede configurar una directiva de grupo que le permita instalar de forma remota la extensión del panel de acceso para Internet Explorer en las máquinas de los usuarios.

Antes de configurar una directiva de grupo, asegúrese de que:

-   Configuró [Active Directory Domain Services](https://msdn.microsoft.com/library/aa362244%28v=vs.85%29.aspx)y unió los equipos de los usuarios a su dominio.

-   Para editar el objeto de directiva de grupo (GPO), debe tener los permisos de *Editar configuración*. De forma predeterminada, este permiso se proporciona a los miembros de los siguientes grupos de seguridad: administradores de dominio, administradores de organización y propietarios del creador de directivas de grupo.

Para instrucciones paso a paso sobre cómo configurar la directiva de grupo e implementarla en los usuarios, consulte [Implementación de la extensión de panel de acceso para Internet Explorer mediante la directiva de grupo](deploy-access-panel-browser-extension.md).

## <a name="troubleshoot-the-access-panel-extension-in-internet-explorer"></a>Solución de problemas de la extensión del panel de acceso en Internet Explorer

Para acceder a una herramienta de diagnóstico y a información sobre la configuración de la extensión para Internet Explorer, consulte [Solución de problemas de la extensión del Panel de acceso para Internet Explorer](manage-access-panel-browser-extension.md).

> [!NOTE]
> Internet Explorer se encuentra en un periodo de compatibilidad limitada y ya no recibe nuevas actualizaciones de software. Microsoft Edge es el explorador recomendado.

## <a name="if-the-preceding-steps-do-not-resolve-the-issue"></a>Si los pasos anteriores no resuelven el problema

Abra una incidencia de soporte técnico con la información siguiente si está disponible:

-   Id. de error de correlación
-   UPN (dirección de correo electrónico del usuario)
-   TenantID
-   Tipo de explorador
-   Zona horaria y la hora o el período de tiempo cuando se produjo el error
-   Seguimientos de Fiddler

## <a name="next-steps"></a>Pasos siguientes
[¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](what-is-single-sign-on.md)
