---
title: 'Publicación de aplicaciones cliente nativas: Azure AD | Microsoft Docs'
description: Explica cómo habilitar las aplicaciones cliente nativas para comunicarse con el conector del proxy de la aplicación de Azure AD para proporcionar acceso remoto seguro a las aplicaciones locales.
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
ms.date: 04/15/2019
ms.author: mimart
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: cb36d6a03da07681db468184a489a79f7f0deab7
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825484"
---
# <a name="how-to-enable-native-client-applications-to-interact-with-proxy-applications"></a>Cómo habilitar las aplicaciones cliente nativas interactuar con aplicaciones de proxy

Puede usar el Proxy de aplicación de Azure Active Directory (Azure AD) para publicar aplicaciones web, pero también se puede usar para publicar las aplicaciones cliente nativas que están configuradas con la biblioteca de autenticación de Azure AD (ADAL). Las aplicaciones cliente nativas son distintas de las aplicaciones web porque están instalados en un dispositivo, mientras se tiene acceso a las aplicaciones web a través de un explorador.

Para admitir las aplicaciones cliente nativas, el Proxy de aplicación acepta tokens de Azure AD emitidos que se envían en el encabezado. El servicio de Proxy de aplicación realiza la autenticación para los usuarios. Esta solución no utiliza tokens de aplicación para la autenticación.

![Relación entre los usuarios finales, Azure Active Directory y las aplicaciones publicadas](./media/application-proxy-configure-native-client-application/richclientflow.png)

Para publicar aplicaciones nativas, utilice la biblioteca de autenticación de Azure AD, que se encarga de autenticación y admite muchos entornos de cliente. El proxy de la aplicación se adapta a la [aplicación nativa para el escenario de Web API](../develop/native-app.md).

Este artículo le guiará por los cuatro pasos para publicar una aplicación nativa con el proxy de aplicación y la biblioteca de Autenticación de Azure AD.

## <a name="step-1-publish-your-proxy-application"></a>Paso 1: Publique su aplicación proxy

Publique su aplicación de proxy al igual que haría con cualquier otra aplicación y asigne a los usuarios el acceso a la aplicación. Para más información, consulte [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](application-proxy-add-on-premises-application.md).

## <a name="step-2-register-your-native-application"></a>Paso 2: Registrar la aplicación nativa

Ahora debe registrar la aplicación en Azure AD, como sigue:

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/). El **panel** para el **centro de administración de Azure Active Directory** aparece.
2. En la barra lateral, seleccione **Azure Active Directory**. El **Azure Active Directory** aparece la página de información general.
3. En la barra lateral información general de Azure AD, seleccione **registros de aplicaciones**. Aparece la lista de todos los registros de aplicaciones.
4. Seleccione **Nuevo registro**. El **registrar una aplicación** aparece la página.

   ![Creación de un registro de aplicaciones](./media/application-proxy-configure-native-client-application/create.png)
5. En el **nombre** encabezado, especifique un nombre para mostrar de cara al usuario para la aplicación.
6. En el **admite tipos de cuenta** encabezado, seleccione un nivel de acceso mediante estas instrucciones:
   - Para tener como destino únicamente las cuentas que son internas a su organización, seleccione **cuentas en este directorio organizativa solo**.
   - Para dirigirse solo empresa o educativos a los clientes, seleccione **cuentas en el directorio de cualquier organización**.
   - Para identificar el conjunto más amplio de identidades de Microsoft, seleccione **cuentas en cualquier directorio de organización y las cuentas personales de Microsoft**.
7. En el **URI de redireccionamiento** encabezado, seleccione **cliente público (móvil y de escritorio)**, y, a continuación, escriba el URI de redireccionamiento para su aplicación.
8. Seleccione y lea la **directivas de la plataforma de Microsoft**y, a continuación, seleccione **registrar**. Se crea y se muestra una página de introducción para el nuevo registro de aplicaciones.

Para obtener más información sobre cómo crear un nuevo registro de aplicaciones, consulte [integración de aplicaciones con Azure Active Directory](../develop/quickstart-v1-integrate-apps-with-azure-ad.md).

## <a name="step-3-grant-access-to-your-proxy-application"></a>Paso 3: Conceder acceso a la aplicación de proxy

Ahora que ha registrado su aplicación nativa, puede darle acceso a otras aplicaciones en su directorio, en este caso para tener acceso a la aplicación de proxy. Para habilitar la aplicación nativa se expongan en el proxy de aplicación:

1. En la barra lateral de la nueva página de registro de aplicación, seleccione **permisos de API**. El **permisos de API** página aparece el nuevo registro de aplicaciones.
2. Seleccione **Agregar un permiso**. El **permisos de solicitud API** aparece la página.
3. En el **seleccionar una API** , seleccione **API que usa mi organización**. Aparece una lista, que contiene las aplicaciones en el directorio que exponen las API.
4. Escribir en el cuadro de búsqueda o desplácese para buscar el proxy de aplicación que publicó en [paso 1: Publique su aplicación proxy](#step-1-publish-your-proxy-application)y, a continuación, seleccione el proxy de aplicación.
5. En el **qué tipo de permisos requiere la aplicación?** encabezado, seleccione el tipo de permiso. Si su aplicación nativa necesita acceder a la API de proxy de aplicación como el usuario que inició sesión, elija **permisos delegados**. Si su aplicación nativa se ejecuta como un servicio en segundo plano o demonio sin un usuario con sesión iniciada, elija **permisos de la aplicación**.
6. En el **seleccionar permisos** encabezado, seleccione el permiso deseado y seleccione **agregar permisos**. El **permisos de API** página ahora su aplicación nativa mostrarán el proxy de aplicación y el permiso API que ha agregado.

## <a name="step-4-edit-the-active-directory-authentication-library"></a>Paso 4: Editar la biblioteca de autenticación de Active Directory

Edite el código de la aplicación nativa en el contexto de autenticación de Active Directory Authentication Library (ADAL) para incluir el siguiente texto:

```
// Acquire Access Token from AAD for Proxy Application
AuthenticationContext authContext = new AuthenticationContext("https://login.microsoftonline.com/<Tenant ID>");
AuthenticationResult result = await authContext.AcquireTokenAsync("< External Url of Proxy App >",
        "<App ID of the Native app>",
        new Uri("<Redirect Uri of the Native App>"),
        PromptBehavior.Never);

//Use the Access Token to access the Proxy Application
HttpClient httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);
HttpResponseMessage response = await httpClient.GetAsync("< Proxy App API Url >");
```

La información necesaria en el código de ejemplo puede encontrarse en el portal de Azure AD, como sigue:

| Se necesita información | Cómo encontrar en el portal de Azure AD |
| --- | --- |
| \<Id. de inquilino > | **Azure Active Directory** > **propiedades** > **Id. de directorio** |
| \<Dirección Url externa de App Proxy > | **Aplicaciones empresariales** > *su aplicación proxy* > **proxy de aplicación** > **dirección Url externa** |
| \<Id. de aplicación de la aplicación nativa > | **Aplicaciones empresariales** > *su aplicación nativa* > **propiedades** > **Id. de aplicación** |
| \<URI de la aplicación nativa de redirección > | **Azure Active Directory** > **registros de aplicaciones** > *su aplicación nativa* > **URI de redirección** |
| \<Dirección Url de API de App proxy > | **Azure Active Directory** > **registros de aplicaciones** > *su aplicación nativa* > **permisos de API**  >  **API / nombre de permisos** |

Después de editar la ADAL con estos parámetros, los usuarios pueden autenticar las aplicaciones cliente nativas incluso cuando están fuera de la red corporativa.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el flujo de la aplicación nativa, vea [aplicaciones nativas en Azure Active Directory](../develop/native-app.md).

Obtenga información acerca de cómo configurar [inicio de sesión único para aplicaciones de Azure Active Directory](what-is-single-sign-on.md#choosing-a-single-sign-on-method).
