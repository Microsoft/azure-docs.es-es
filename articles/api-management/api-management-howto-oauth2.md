---
title: Autorización de cuentas de desarrollador mediante OAuth 2.0 en API Management
titleSuffix: Azure API Management
description: Obtenga información acerca de cómo autorizar a los usuarios para que utilicen OAuth 2.0 en Administración de API. OAuth 2.0 protege la API para que los usuarios solo puedan acceder a los recursos a los que están autorizados.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 4a0e8c6eb87b053c0e3ee524c60435dc45d81ff3
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87902265"
---
# <a name="how-to-authorize-developer-accounts-using-oauth-20-in-azure-api-management"></a>Procedimiento para autorizar a las cuentas de desarrollador para que usen OAuth 2.0 en Azure API Management

Muchas API admiten [OAuth 2.0](https://oauth.net/2/) para proteger la API y garantizar que solo usuarios válidos obtengan acceso y que, además, solo puedan tener acceso a los recursos para los que estén autorizados. Para usar la consola interactiva para desarrolladores de la Azure API Management, el servicio permite configurar la instancia del servicio para que funcione con la API habilitada para OAuth 2.0.

> [!IMPORTANT]
> La autorización de OAuth 2.0 todavía no está disponible en la consola interactiva del nuevo portal para desarrolladores.

## <a name="prerequisites"></a><a name="prerequisites"> </a>Requisitos previos

En esta guía se explica cómo configurar la instancia del servicio Administración de API para que use la autorización OAuth 2.0 con las cuentas de desarrollador, pero no se explica cómo configurar un proveedor de OAuth 2.0. Aunque los proveedores de OAuth 2.0 tienen configuraciones diferentes, los pasos son similares y se precisa la misma información para configurar OAuth 2.0 en la instancia del servicio Administración de API. Este tema muestra ejemplos donde Azure Active Directory actúa como proveedor de OAuth 2.0.

> [!NOTE]
> Para obtener más información sobre cómo configurar OAuth 2.0 con Azure Active Directory, consulte el ejemplo de [WebApp-GraphAPI-DotNet][WebApp-GraphAPI-DotNet] .

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="configure-an-oauth-20-authorization-server-in-api-management"></a><a name="step1"> </a>Configuración de un servidor de autorización OAuth 2.0 en API Management

> [!NOTE]
> Si todavía no ha creado una instancia del servicio API Management, consulte [Creación de una instancia del servicio API Management][Create an API Management service instance].

1. Haga clic en la pestaña OAuth 2.0 en el menú de la izquierda y haga clic en **+Agregar**.

    ![Menú OAuth 2.0](./media/api-management-howto-oauth2/oauth-01.png)

2. Escriba un nombre y una descripción opcional en los campos **Nombre** y **Descripción**.

    > [!NOTE]
    > Estos campos sirven para identificar el servidor de autorización OAuth 2.0 en la instancia del servicio Administración de API y los valores que contienen no proceden del servidor OAuth 2.0.

3. Escriba la **URL de la página de registro de cliente**. En esta página, los usuarios pueden crear y administrar sus cuentas. Puede variar en función del proveedor de OAuth 2.0. La **URL de la página de registro de cliente** señala a la página que los usuarios pueden utilizar para crear y configurar sus propias cuentas para proveedores de OAuth 2.0 que admiten la administración de usuarios de las cuentas (por ejemplo, `https://contoso.com/login`). Algunas organizaciones no configuran ni usan esta funcionalidad, aunque la admita el proveedor de OAuth 2.0. Si el proveedor de OAuth 2.0 no tiene configurada la administración de usuarios de las cuentas, especifique aquí una URL de marcador de posición, por ejemplo, la URL de su empresa o una URL como `https://placeholder.contoso.com`.

    ![Nuevo servidor de OAuth 2.0](./media/api-management-howto-oauth2/oauth-02.png)

4. La siguiente sección del formulario contiene la configuración de **Authorization grant types** (Tipos de concesión de autorización), **Authorization endpoint URL** (URL del punto de conexión de autorización) y **Authorization request method** (Método de solicitud de autorización).

    Active las casillas de los **tipos de concesión de autorización** que desee. **Código de autorización** se especifica de forma predeterminada.

    Escriba la **URL del extremo de autorización**. En Azure Active Directory, esta dirección URL será similar a la siguiente, donde `<tenant_id>` se reemplaza por el identificador de su inquilino de Azure AD.

    `https://login.microsoftonline.com/<tenant_id>/oauth2/authorize`

    El **Método de solicitud de autorización** especifica cómo se envía la solicitud de autorización al servidor OAuth 2.0. El valor predeterminado es **GET** .

5. A continuación, debe especificar **URL del punto de conexión de token**, **Métodos de autenticación de cliente**, **Método de envío de tokens de acceso** y **Ámbito predeterminado**.

    ![Nuevo servidor de OAuth 2.0](./media/api-management-howto-oauth2/oauth-03.png)

    En un servidor OAuth 2.0 de Azure Active Directory, la **URL del punto de conexión de token** tendrá el formato siguiente, donde `<TenantID>` tiene el formato de `yourapp.onmicrosoft.com`.

    `https://login.microsoftonline.com/<TenantID>/oauth2/token`

    Los valores predeterminados para **Métodos de autenticación de cliente** y **Método de envío de tokens de acceso** son **Básico** y **Encabezado de autorización** respectivamente. Estos valores se configuran en esta sección del formulario, junto con el **Ámbito predeterminado**.

6. La sección **Credenciales de cliente** contiene el **Id. de cliente** y el **Secreto del cliente** que se obtienen durante los procesos de creación y configuración del servidor OAuth 2.0. Al especificar el **Id. de cliente** y el **Secreto del cliente**, se genera el **uri de redirección** para el **código de autorización**. Con este URI se configura la URL de respuesta en la configuración del servidor OAuth 2.0.

    En el nuevo portal para desarrolladores, el sufijo del URI tiene el formato siguiente:

    - `/signin-oauth/code/callback/{authServerName}` para el flujo de concesión de código de autorización
    - `/signin-oauth/implicit/callback` para el flujo de concesión implícita

    ![Nuevo servidor de OAuth 2.0](./media/api-management-howto-oauth2/oauth-04.png)

    Si **Authorization grant types**  (Tipos de concesión de autorización) se establece en **Resource owner password** (Contraseña de propietario de recursos), las credenciales se especifican con la sección **Resource owner password credentials** (Credenciales de contraseña de propietario de recursos). De no ser así, puede dejarse en blanco.

    Cuando complete el formulario, haga clic en **Crear** para guardar la configuración del servidor de autorización OAuth 2.0 de API Management. Cuando se guarda la configuración del servidor, pueden configurarse las API para que usen estos valores, conforme a la sección siguiente.

## <a name="configure-an-api-to-use-oauth-20-user-authorization"></a><a name="step2"> </a>Configuración de una API para usar la autorización de usuarios OAuth 2.0

1. Haga clic en **API** en el menú **API Management** de la izquierda.

    ![API de OAuth 2.0](./media/api-management-howto-oauth2/oauth-05.png)

2. Haga clic en el nombre de la API deseada y haga clic en **Configuración**. Desplácese hasta la sección **Seguridad** y, a continuación, active la casilla de **OAuth 2.0**.

    ![Configuración de OAuth 2.0](./media/api-management-howto-oauth2/oauth-06.png)

3. Seleccione el **Servidor de autorización** que quiera en la lista desplegable y haga clic en **Guardar**.

    ![Configuración de OAuth 2.0](./media/api-management-howto-oauth2/oauth-07.png)

## <a name="legacy-developer-portal---test-the-oauth-20-user-authorization"></a><a name="step3"> </a>Prueba de la autorización de usuarios OAuth 2.0 en el portal para desarrolladores (heredado)

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

Tras configurar el servidor de autorización OAuth 2.0 y las API para que usen dicho servidor, puede probarlo. Para ello, vaya al portal para desarrolladores y llame a una API. Haga clic en **Portal para desarrolladores (heredado)** en el menú superior de la página **Introducción** de la instancia de Azure API Management.

Haga clic en **API** en el menú superior y seleccione **API eco**.

![API eco][api-management-apis-echo-api]

> [!NOTE]
> Si solamente tiene una API configurada o visible en su cuenta, al hacer clic en API irá directamente a las operaciones de dicha API.

Seleccione la operación **Recurso GET**, haga clic en **Abrir consola** y seleccione **Código de autorización** en la lista desplegable.

![Open console][api-management-open-console]

Al seleccionar **Código de autorización** , se abre una ventana emergente con el formulario de suscripción del proveedor de OAuth 2.0. En este ejemplo, Azure Active Directory ha proporcionado el formulario de suscripción.

> [!NOTE]
> Si ha deshabilitado los elementos emergentes, el explorador le solicitará que los habilite. Cuando los habilite, seleccione **Código de autorización** de nuevo y se mostrará el formulario de suscripción.

![Iniciar sesión][api-management-oauth2-signin]

Tras iniciar sesión, los **Encabezados de solicitud** se rellenan con el encabezado `Authorization : Bearer` que autoriza la solicitud.

![Token de encabezado de solicitud][api-management-request-header-token]

Llegado a este punto, puede configurar los valores que desea para los demás parámetros y enviar la solicitud.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca del uso de OAuth 2.0 y Administración de API, vea el siguiente vídeo y el [artículo](api-management-howto-protect-backend-with-aad.md)que lo acompaña.

[api-management-oauth2-signin]: ./media/api-management-howto-oauth2/api-management-oauth2-signin.png
[api-management-request-header-token]: ./media/api-management-howto-oauth2/api-management-request-header-token.png
[api-management-open-console]: ./media/api-management-howto-oauth2/api-management-open-console.png
[api-management-apis-echo-api]: ./media/api-management-howto-oauth2/api-management-apis-echo-api.png

[How to add operations to an API]: ./mock-api-responses.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-policies.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[https://oauth.net/2/]: https://oauth.net/2/
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps
