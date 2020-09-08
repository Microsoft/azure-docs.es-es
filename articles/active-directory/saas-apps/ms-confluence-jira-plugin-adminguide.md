---
title: 'Guía del administrador de Jira/Confluence de Atlassian: Azure Active Directory| Microsoft Docs'
description: Guía del administrador para usar Jira y Confluence de Atlassian con Azure Active Directory (Azure AD)
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 11/19/2018
ms.author: jeedes
ms.openlocfilehash: 07bd9353f08e9440e495dfa92c608b4d6f70f5cf
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543989"
---
# <a name="atlassian-jira-and-confluence-admin-guide-for-azure-active-directory"></a>Guía del administrador de Jira y Confluence de Atlassian para Azure Active Directory

## <a name="overview"></a>Información general

El complemento de inicio de sesión único (SSO) de Azure Active Directory (Azure AD) permite a los clientes de Microsoft Azure AD usar su cuenta profesional o educativa para iniciar sesión en Atlassian Jira y en los productos basados en Confluence Server. Implementa el SSO basado en SAML 2.0.

## <a name="how-it-works"></a>Funcionamiento

Cuando los usuarios deseen iniciar sesión en la aplicación Confluence o Atlassian Jira, verán el botón **Login with Azure AD** (Iniciar sesión con Azure AD) en la página de inicio de sesión. Cuando se selecciona, deben iniciar sesión mediante la página de inicio de sesión de la organización de Azure AD (es decir, su cuenta profesional o educativa).

Una vez que los usuarios se autentican, deberían poder iniciar sesión en la aplicación. Si ya se han autenticado con el identificador y la contraseña de su cuenta profesional o educativa, inician sesión directamente en la aplicación. 

El inicio de sesión funciona en Jira y en Confluence. Si los usuarios iniciaron sesión en la aplicación Jira y Confluence también está abierto en la misma ventana del explorador, no tienen que proporcionar las credenciales para la otra aplicación. 

Los usuarios también pueden obtener los productos de Atlassian a través de Mis aplicaciones en la cuenta profesional o educativa. Deberían iniciar sesión sin que se les pidan las credenciales.

> [!NOTE]
> El aprovisionamiento de usuarios no se realiza con este complemento.

## <a name="audience"></a>Público

Los administradores de Jira y Confluence pueden usar el complemento para habilitar SSO con Azure AD.

## <a name="assumptions"></a>Supuestos

* Las instancias de Jira y Confluence están habilitadas para HTTPS.
* Los usuarios ya se crearon en Jira o Confluence.
* Los usuarios tienen roles asignados en Jira o Confluence.
* Los administradores tienen acceso a la información necesaria para configurar el complemento.
* Jira o Confluence también están disponibles fuera de la red de la empresa.
* El complemento funciona solo con las versiones locales de Jira y Confluence.

## <a name="prerequisites"></a>Requisitos previos

Antes de instalar el complemento, tenga en cuenta la siguiente información:

* Jira y Confluence están instalados en una versión de Windows de 64 bits.
* Las versiones de Jira y Confluence están habilitadas para HTTPS.
* Jira y Confluence están disponibles en Internet.
* Las credenciales de administrador están en vigor para Jira y Confluence.
* Las credenciales de administrador están en vigor para Azure AD.
* WebSudo está deshabilitado en Jira y Confluence.

## <a name="supported-versions-of-jira-and-confluence"></a>Versiones admitidas de Jira y Confluence

El complemento admite las versiones siguientes en Jira y Confluence:

* Jira Core y Software: de la versión 6.0 a la 7.12
* Jira Service Desk: 3.0.0 a 3.5.0
* JIRA también admite 5.2. Para obtener más detalles, haga clic en [Microsoft Azure Active Directory single sign-on for JIRA 5.2](https://docs.microsoft.com/azure/active-directory/saas-apps/jira52microsoft-tutorial) (Inicio de sesión único de Microsoft Azure Active Directory para JIRA 5.2).
* Confluence: 5.0 a 5.10
* Confluence: 6.0.1
* Confluence: 6.1.1
* Confluence: 6.2.1
* Confluence: 6.3.4
* Confluence: 6.4.0
* Confluence: 6.5.0
* Confluence: 6.6.2
* Confluence: 6.7.0
* Confluence: 6.8.1
* Confluence: 6.9.0
* Confluence: 6.10.0
* Confluence: 6.11.0
* Confluence: 6.12.0

## <a name="installation"></a>Instalación

Para instalar el complemento, siga estos pasos:

1. Inicie sesión en la instancia de Jira o Confluence como administrador.

2. Vaya a la consola de administración de Jira o Confluence y seleccione **Add-ons** (Complementos).

3. Desde el Centro de descarga de Microsoft, descargue el [complemento de SSO de Microsoft SAML para Jira](https://www.microsoft.com/download/details.aspx?id=56506)/ [Complemento de SSO de Microsoft SAML para Confluence](https://www.microsoft.com/download/details.aspx?id=56503).

   La versión adecuada del complemento aparece en los resultados de búsqueda.

4. Seleccione el complemento y el Administrador de complementos universal (UPM) lo instala.

Una vez instalado el complemento, aparece en la sección **Complementos instalados por el usuario** de **Administrar complemento**.

## <a name="plug-in-configuration"></a>Configuración de complementos

Para empezar a usar el complemento, debe configurarlo. Seleccione el complemento, seleccione el botón **Configurar** y proporcione los detalles de configuración.

En la imagen siguiente se muestra la pantalla de configuración tanto en Jira como en Confluence:

![Pantalla de configuración de complementos](./media/ms-confluence-jira-plugin-adminguide/jira.png)

* **Metadata URL** (Dirección URL de metadatos): Dirección URL para obtener los metadatos de federación de Azure AD.

* **Identifiers** (Identificadores): Dirección URL que Azure AD usa para validar el origen de la solicitud. Se asigna al elemento **Identificador** de Azure AD. El complemento obtiene automáticamente esta dirección URL como https:// *\<domain:port>* /.

* **Reply URL** (URL de respuesta): Dirección URL de respuesta en el proveedor de identidades (IdP) que inicia el inicio de sesión de SAML. Se asigna al elemento **Dirección URL de respuesta** de Azure AD. El complemento deduce automáticamente esta dirección URL como https:// *\<domain:port>* /plugins/servlet/saml/auth.

* **Sign On URL** (Dirección URL de inicio de sesión): Dirección URL de inicio de sesión en el IdP que inicia el inicio de sesión de SAML. Se asigna al elemento **Inicio de sesión** de Azure AD. El complemento deduce automáticamente esta dirección URL como https:// *\<domain:port>* /plugins/servlet/saml/auth.

* **IdP Entity ID** (Id. de entidad de IdP): Identificador de entidad que el IdP usa. Este campo se rellena cuando se resuelve la dirección URL de metadatos.

* **Login URL** (Dirección URL de inicio de sesión): Dirección URL de inicio de sesión del IdP. Este cuadro se rellena desde Azure AD cuando se resuelve la dirección URL de metadatos.

* **Logout URL** (Dirección URL de cierre de sesión): Dirección URL de cierre de sesión del IdP. Este cuadro se rellena desde Azure AD cuando se resuelve la dirección URL de metadatos.

* **X.509 Certificate** (Certificado X.509): Certificado X.509 del IdP. Este cuadro se rellena desde Azure AD cuando se resuelve la dirección URL de metadatos.

* **Login Button Name** (Nombre de botón de inicio de sesión): Nombre del botón de inicio de sesión que la organización quiere que los usuarios vean en la página de inicio de sesión.

* **SAML User ID Locations** (Ubicaciones de Id. de usuario SAML): Ubicación donde se espera el identificador de usuario de Jira o Confluence en la respuesta de SAML. Puede estar en **NameID** o en un nombre de atributo personalizado.

* **Attribute Name** (Nombre del atributo): Nombre del atributo donde se espera el identificador del usuario.

* **Enable Home Realm Discovery** (Habilitar detección del dominio de inicio): Selección que hay que hacer si la compañía usa el inicio de sesión basado en Servicios de federación de Active Directory (AD FS).

* **Domain Name** (Nombre de dominio): Nombre del dominio si el inicio de sesión se basa en AD FS.

* **Enable Single Signout** (Habilitar cierre de sesión único): Selección que hay que hacer si quiere cerrar la sesión de Azure AD cuando un usuario cierra la sesión de Jira o Confluence.

## <a name="troubleshooting"></a>Solución de problemas

* **Recibe varios errores de certificados**: Inicie sesión en Azure AD y quite los distintos certificados disponibles en la aplicación. Asegúrese de que haya solo un certificado.

* **Un certificado está a punto de expirar en Azure AD**: Los complementos se encargan de sustituir automáticamente el certificado. Cuando hay un certificado a punto de expirar, se debe marcar como activo un certificado nuevo y se deben eliminar los certificados sin usar. Cuando un usuario intenta iniciar sesión en Jira en este escenario, el complemento captura el certificado nuevo y lo guarda.

* **Desea deshabilitar WebSudo (deshabilitar la sesión segura del administrador)** :

  * En Jira, las sesiones seguras del administrador (es decir, con confirmación de contraseña antes de acceder a las funciones de administración) están habilitadas de manera predeterminada. Si desea quitar esta capacidad en la instancia de Jira, especifique la siguiente línea en el archivo jira-config.properties: `jira.websudo.is.disabled = true`

  * En Confluence, siga los pasos del [sitio de soporte técnico de Confluence](https://confluence.atlassian.com/doc/configuring-secure-administrator-sessions-218269595.html).

* **Los campos que se supone que deberían rellenarse con la dirección URL de metadatos no se están rellenando**:

  * Compruebe que la dirección URL sea la correcta. Revise si asignó el inquilino y el identificador de aplicación correctos.

  * Escriba la dirección URL en el explorador y vea si recibe el archivo XML de metadatos de federación.

* **Hay un error interno del servidor**: Revise los registros en el directorio de registro de la instalación. Si recibe el error cuando el usuario intenta iniciar sesión mediante el SSO de Azure AD, puede compartir los registros con el equipo de soporte técnico.

* **Hay un error "User ID not found" cuando el usuario intenta iniciar sesión**: Cree el identificador de usuario en Jira o Confluence.

* **Hay un error "App not found" en Azure AD**: Vea si la dirección URL correspondiente está asignada a la aplicación en Azure AD.

* **Necesita soporte técnico**: Escriba al [equipo de integración de SSO de Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>). El equipo responde en un plazo de entre 24 y 48 horas laborales.

  También puede generar una incidencia de soporte técnico con Microsoft a través del canal de Azure Portal.

## <a name="plug-in-faq"></a>Preguntas más frecuentes sobre complementos

Consulte a continuación las preguntas más frecuentes en caso de que tenga cualquier consulta con respecto a este complemento.

### <a name="what-does-the-plug-in-do"></a>¿Qué hace el complemento?

El complemento proporciona la funcionalidad de inicio de sesión único para el software local Jira (como Jira Core, Jira Software, Jira Service Desk) y Confluence de Atlassian. El complemento funciona con Azure Active Directory (Azure AD) como proveedor de identidades (IdP).

### <a name="which-atlassian-products-does-the-plug-in-work-with"></a>¿Con qué productos de Atlassian funciona el complemento?

El complemento funciona con las versiones locales de Jira y Confluence.

### <a name="does-the-plug-in-work-on-cloud-versions"></a>¿Este complemento funciona en las versiones en la nube?

No. El complemento admite solo las versiones locales de Jira y Confluence.

### <a name="which-versions-of-jira-and-confluence-does-the-plug-in-support"></a>¿Qué versiones de JIRA y Confluence admite el complemento?

El complemento admite estas versiones:

* Jira Core y Software: de la versión 6.0 a la 7.12
* Jira Service Desk: 3.0.0 a 3.5.0
* JIRA también admite 5.2. Para obtener más detalles, haga clic en [Microsoft Azure Active Directory single sign-on for JIRA 5.2](https://docs.microsoft.com/azure/active-directory/saas-apps/jira52microsoft-tutorial) (Inicio de sesión único de Microsoft Azure Active Directory para JIRA 5.2).
* Confluence: 5.0 a 5.10
* Confluence: 6.0.1
* Confluence: 6.1.1
* Confluence: 6.2.1
* Confluence: 6.3.4
* Confluence: 6.4.0
* Confluence: 6.5.0
* Confluence: 6.6.2
* Confluence: 6.7.0
* Confluence: 6.8.1
* Confluence: 6.9.0
* Confluence: 6.10.0
* Confluence: 6.11.0
* Confluence: 6.12.0

### <a name="is-the-plug-in-free-or-paid"></a>¿El complemento es gratuito o de pago?

Es un complemento gratuito.

### <a name="do-i-need-to-restart-jira-or-confluence-after-i-deploy-the-plug-in"></a>¿Tengo que reiniciar Kira o Confluence una vez que implemente el complemento?

No es necesario reiniciar. Puede empezar a usar el complemento de inmediato.

### <a name="how-do-i-get-support-for-the-plug-in"></a>¿Cómo puedo obtener soporte técnico para el complemento?

Puede ponerse en contacto con el [equipo de integración de Azure AD SSO](<mailto:SaaSApplicationIntegrations@service.microsoft.com>) si necesita soporte técnico con relación a este complemento. El equipo responde en un plazo de entre 24 y 48 horas laborales.

También puede generar una incidencia de soporte técnico con Microsoft a través del canal de Azure Portal.

### <a name="would-the-plug-in-work-on-a-mac-or-ubuntu-installation-of-jira-and-confluence"></a>¿El complemento funciona en una instalación Mac o Ubuntu de Jira y Confluence?

Hemos probado este complemento solo en instalaciones de Windows Server de 64 bits de Jira y Confluence.

### <a name="does-the-plug-in-work-with-idps-other-than-azure-ad"></a>¿El complemento funciona con IdP distintos de Azure AD?

No. Solo funciona con Azure AD.

### <a name="what-version-of-saml-does-the-plug-in-work-with"></a>¿Con qué versión de SAML funciona el complemento?

Funciona con SAML 2.0.

### <a name="does-the-plug-in-do-user-provisioning"></a>¿El complemento se ocupa del aprovisionamiento de usuarios?

No. El complemento solo proporciona SSO basado en SAML 2.0. El usuario tiene que estar aprovisionado en la aplicación antes del inicio de sesión SSO.

### <a name="does-the-plug-in-support-cluster-versions-of-jira-and-confluence"></a>¿El complemento admite versiones en clúster de Jira y Confluence?

No. El complemento funciona con las versiones locales de Jira y Confluence.

### <a name="does-the-plug-in-work-with-http-versions-of-jira-and-confluence"></a>¿El complemento admite versiones HTTP de Jira y Confluence?

No. El complemento solo funciona con instalaciones habilitadas de HTTPS.
