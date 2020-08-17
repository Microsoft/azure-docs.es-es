---
title: Plataforma de identidad de Microsoft y flujo de aserción de portador SAML | Azure
description: Obtenga información acerca de cómo capturar datos de Microsoft Graph sin pedir al usuario las credenciales mediante el flujo de aserción de portador SAML.
services: active-directory
author: umeshbarapatre
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/05/2019
ms.author: kenwith
ms.reviewer: paulgarn
ms.custom: aaddev
ms.openlocfilehash: e0db5bec00ce864536b3559eda160acdada5e157
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88114700"
---
# <a name="microsoft-identity-platform-and-oauth-20-saml-bearer-assertion-flow"></a>Plataforma de identidad de Microsoft y flujo de aserción de portador SAML de OAuth 2.0
El flujo de aserción de portador SAML de OAuth 2.0 le permite solicitar un token de acceso de OAuth mediante una aserción SAML cuando un cliente tiene que usar una relación de confianza existente. La firma aplicada a la aserción SAML proporciona la autenticación de la aplicación autorizada. Una aserción SAML es un token de seguridad XML emitido por un proveedor de identidades y consumido por un proveedor de servicios. El proveedor de servicios se basa en su contenido para identificar el asunto de la aserción con fines relacionados con la seguridad.

La aserción SAML se envía al punto de conexión del token de OAuth.  El punto de conexión procesa la aserción y emite un token de acceso basado en la aprobación anterior de la aplicación. No es necesario que el cliente tenga o almacene un token de actualización, ni que el secreto de cliente deba pasarse al punto de conexión del token.

El flujo de aserción de portador SAML es útil cuando se recuperan datos de Microsoft Graph API (que solo admiten permisos delegados) sin pedir las credenciales al usuario. En este escenario, la concesión de credenciales de cliente, que es preferible para los procesos en segundo plano, no funciona.

En el caso de las aplicaciones con un inicio de sesión interactivo basado en explorador para obtener una aserción SAML y luego quieren agregar acceso a una API protegida de OAuth (como Microsoft Graph), puede hacer una solicitud de OAuth para obtener un token de acceso para la API. Cuando el explorador se redirige a Azure AD para autenticar al usuario, el explorador seleccionará la sesión desde el inicio de sesión de SAML, y el usuario no tendrá que escribir sus credenciales.

También se admite el flujo de aserción de portador SAML de OAuth para los usuarios que se autentican con proveedores de identidades, como Servicios de federación de Active Directory (ADFS), federados para Azure Active Directory.  La aserción SAML obtenida de ADFS puede usarse en un flujo de OAuth para autenticar al usuario.

![Flujo de OAuth](./media/v2-saml-bearer-assertion/1.png)

## <a name="call-graph-using-saml-bearer-assertion"></a>Gráfico de llamadas con aserción de portador SAML
Ahora, vamos a comprender cómo podemos capturar realmente aserciones SAML mediante programación. Este enfoque se prueba con ADFS. Sin embargo, funciona con cualquier proveedor de identidades que admita el retorno de la aserción SAML mediante programación. El proceso básico es: obtener una aserción SAML, obtener un token de acceso y acceder a Microsoft Graph.

### <a name="prerequisites"></a>Requisitos previos

Establezca una relación de confianza entre el servidor o entorno de autorización (Microsoft 365) y el proveedor de identidades o el emisor de la aserción de portador SAML 2.0 (ADFS). Para configurar ADFS para el inicio de sesión único y como proveedor de identidades, puede consultar [este artículo](/archive/blogs/canitpro/step-by-step-setting-up-ad-fs-and-enabling-single-sign-on-to-office-365).

Registre la aplicación en el [portal](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade):
1. Inicie sesión en la [hoja de registro de aplicaciones del portal](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) (tenga en cuenta que usamos los puntos de conexión v2.0 para Graph API y, por tanto, es necesario registrar la aplicación en este portal. De lo contrario, podríamos haber usado los registros en Azure Active Directory). 
1. Seleccione **Nuevo registro**.
1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación: 
    1. **Nombre**: escriba un nombre significativo para la aplicación, que se mostrará a los usuarios de la aplicación.
    1. **Tipos de cuentas compatibles**: seleccione qué cuentas desea que admita la aplicación.
    1. **URI de redirección (opcional)** : Seleccione el tipo de aplicación que se va a crear, Web o Cliente público (móvil y escritorio) y, luego, escriba el URI de redireccionamiento (o la dirección URL de respuesta) para la aplicación.
    1. Cuando termine, seleccione **Registrar**.
1. Tome nota del id. de aplicación (cliente).
1. En el panel izquierdo, seleccione **Certificados y secretos**. Haga clic en **Nuevo secreto de cliente** en la sección **Secretos de cliente**. Copie el nuevo secreto de cliente, no podrá recuperarlo cuando salga de la hoja.
1. En el panel izquierdo, seleccione **Permisos de API** y, luego, **Agregar un permiso**. Seleccione **Microsoft Graph**, luego, **permisos delegados** y, luego, seleccione **Tasks.read**, ya que tenemos previsto usar la Graph API de Outlook. 

Instale [Postman](https://www.getpostman.com/), una herramienta necesaria para probar las solicitudes de ejemplo.  Posteriormente, puede convertir las solicitudes en código.

### <a name="get-the-saml-assertion-from-adfs"></a>Obtención de la aserción SAML desde ADFS
Cree una solicitud POST al punto de conexión de ADFS mediante un sobre SOAP para capturar la aserción SAML:

![Obtención de la aserción SAML](./media/v2-saml-bearer-assertion/2.png)

Valores de encabezado:

![Valores de encabezado](./media/v2-saml-bearer-assertion/3.png)

Cuerpo de la solicitud de ADFS:

![Cuerpo de la solicitud de ADFS](./media/v2-saml-bearer-assertion/4.png)

Una vez que esta solicitud se publique correctamente, debería recibir una aserción SAML de ADFS. Solo se requieren los datos de la etiqueta **SAML:Assertion**; conviértalos a la codificación en base 64 para usarlos en solicitudes posteriores.

### <a name="get-the-oauth2-token-using-the-saml-assertion"></a>Obtención del token de OAuth2 mediante la aserción SAML 
En este paso, capture un token de OAuth2 mediante la respuesta de aserción de ADFS.

1. Cree una solicitud POST como se muestra a continuación con los valores de encabezado:

    ![Solicitud POST](./media/v2-saml-bearer-assertion/5.png)
1. En el cuerpo de la solicitud, reemplace **client_id**, **client_secret**y **assertion** (la aserción SAML codificada en base 64 que obtuvo en el paso anterior):

    ![Cuerpo de la solicitud](./media/v2-saml-bearer-assertion/6.png)
1. Tras una solicitud correcta, recibirá un token de acceso de Azure Active Directory.

### <a name="get-the-data-with-the-oauth-token"></a>Obtención de los datos con el token de OAuth

Después de recibir el token de acceso, llame a las Graph API (tareas de Outlook en este ejemplo). 

1. Cree una solicitud GET con el token de acceso capturado en el paso anterior:

    ![Solicitud GET](./media/v2-saml-bearer-assertion/7.png)

1. Tras una solicitud correcta, recibirá una respuesta JSON.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los distintos [flujos de autenticación y escenarios de aplicaciones](authentication-flows-app-scenarios.md).