---
title: Protección de una API de Azure API Management mediante Azure Active Directory B2C
description: Obtenga información sobre el uso de tokens de acceso emitidos por Azure Active Directory B2C para proteger un punto de conexión de API de Azure API Management.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 0e8802d612f2497cc58c90856e9a5a5572a142f1
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87482845"
---
# <a name="secure-an-azure-api-management-api-with-azure-ad-b2c"></a>Protección de una API de Azure API Management con Azure AD B2C

Obtenga información sobre cómo restringir el acceso a la API de Azure API Management (APIM) a los clientes que se han autenticado con Azure Active Directory B2C (Azure AD B2C). Siga los pasos de este artículo para crear y probar una directiva de entrada en APIM que restrinja el acceso solo a aquellas solicitudes que incluyan un token de acceso válido emitido por Azure AD B2C.

## <a name="prerequisites"></a>Requisitos previos

Necesitará los siguientes recursos antes de continuar con los pasos de este artículo:

* [Inquilino de Azure AD B2C](tutorial-create-tenant.md)
* [Aplicación registrada](tutorial-register-applications.md) en su inquilino
* [Flujos de usuario creados](tutorial-create-user-flows.md) en su inquilino
* [API publicada](../api-management/import-and-publish.md) en Azure API Management
* [Postman](https://www.getpostman.com/) para probar el acceso protegido (opcional)

## <a name="get-azure-ad-b2c-application-id"></a>Obtención del identificador de aplicación de Azure AD B2C

Cuando se protege una API en Azure API Management con Azure AD B2C, se necesitan varios valores para la [directiva de entrada](../api-management/api-management-howto-policies.md) que se crea en APIM. En primer lugar, anote el identificador de una aplicación que haya creado anteriormente en el inquilino de Azure AD B2C. Si usa la aplicación que creó en los requisitos previos, utilice el identificador de aplicación de *webbapp1*.

Para registrar una aplicación en su inquilino de Azure AD B2C, puede usar la nueva experiencia unificada **Registros de aplicaciones**, o bien la experiencia anterior **Aplicaciones (heredado)** . [Más información acerca de la nueva experiencia](https://aka.ms/b2cappregtraining).

#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones** y, a continuación, la pestaña **Aplicaciones propias**.
1. Anote el valor de la columna **Id. de aplicación (cliente)** de *webapp1* u otra aplicación que haya creado previamente.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. En **Administrar**, seleccione **Aplicaciones (heredado)** .
1. Anote el valor de la columna **Identificador de aplicación** de *webapp1* u otra aplicación que haya creado previamente.

* * *

## <a name="get-token-issuer-endpoint"></a>Obtención del punto de conexión del emisor del token

A continuación, obtenga la conocida dirección URL de configuración de uno de los flujos de usuario de Azure AD B2C. También necesita el identificador URI del punto de conexión del emisor del token que desea admitir en Azure API Management.

1. Vaya al inquilino de Azure AD B2C en [Azure Portal](https://portal.azure.com).
1. En **Directivas**, seleccione**Flujos de usuario**.
1. Seleccione una directiva existente, por ejemplo *B2C_1_signupsignin1* y, luego, seleccione **Ejecutar flujo de usuario**.
1. Anote la dirección URL del hipervínculo que aparece en el encabezado **Ejecutar flujo de usuario** situado cerca de la parte superior de la página. Esta dirección URL es el conocido punto de conexión de detección de OpenID Connect del flujo de usuario y se usa en la sección siguiente cuando se configura la directiva de entrada en Azure API Management.

    ![Hipervínculo de URI conocido en la página Ejecutar ahora de Azure Portal](media/secure-apim-with-b2c-token/portal-01-policy-link.png)

1. Seleccione el hipervínculo para ir a la conocida página de configuración de OpenID Connect.
1. En la página que se abre en el explorador, registre el valor `issuer`, por ejemplo:

    `https://<tenant-name>.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/`

    Este valor se usa en la sección siguiente al configurar la API en Azure API Management.

Ahora debería tener dos direcciones URL anotadas para su uso en la sección siguiente: la dirección URL del punto de conexión de configuración de OpenID Connect y el identificador URI del emisor. Por ejemplo:

```
https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration
https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/
```

## <a name="configure-inbound-policy-in-azure-api-management"></a>Configuración de la directiva de entrada en Azure API Management

Ahora está listo para agregar la directiva de entrada en Azure API Management que valida las llamadas a la API. Puede asegurarse de que solo se aceptan llamadas a la API con un token válido mediante la adición de una directiva de [validación de JWT](../api-management/api-management-access-restriction-policies.md#ValidateJWT) que comprueba la audiencia y el emisor de un token de acceso.

1. Navegue a la instancia de Azure API Management en [Azure Portal](https://portal.azure.com).
1. Seleccione **API**.
1. Seleccione la API que desea proteger con Azure AD B2C.
1. Seleccione la pestaña **Diseño**.
1. En **Procesamiento de entrada**, seleccione **\</\>** para abrir el editor de código de directivas.
1. Coloque la siguiente etiqueta `<validate-jwt>` dentro de la directiva `<inbound>`.

    1. Actualice el valor de `url` del elemento `<openid-config>` con la conocida dirección URL de configuración de la directiva.
    1. Actualice el elemento `<audience>` con el identificador de aplicación de la aplicación que creó anteriormente en el inquilino de B2C (por ejemplo, *webapp1*).
    1. Actualice el elemento `<issuer>` con el punto de conexión del emisor del token que anotó anteriormente.

    ```xml
    <policies>
        <inbound>
            <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
                <openid-config url="https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration" />
                <audiences>
                    <audience>44444444-0000-0000-0000-444444444444</audience>
                </audiences>
                <issuers>
                    <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
                </issuers>
            </validate-jwt>
            <base />
        </inbound>
        <backend> <base /> </backend>
        <outbound> <base /> </outbound>
        <on-error> <base /> </on-error>
    </policies>
    ```

## <a name="validate-secure-api-access"></a>Validación del acceso protegido a la API

Para asegurarse de que solo los llamadores autenticados pueden acceder a la API, puede validar la configuración de Azure API Management mediante una llamada a la API con [Postman](https://www.getpostman.com/).

Para llamar a la API, necesita un token de acceso emitido por Azure AD B2C y una clave de suscripción de APIM.

### <a name="get-an-access-token"></a>Obtención de un token de acceso

En primer lugar, necesita un token emitido por Azure AD B2C para usarlo en el encabezado `Authorization` de Postman. Puede obtener uno mediante la característica **Ejecutar ahora** del flujo de usuario de registro o de inicio de sesión que debe haber creado como uno de los requisitos previos.

1. Vaya al inquilino de Azure AD B2C en [Azure Portal](https://portal.azure.com).
1. En **Directivas**, seleccione**Flujos de usuario**.
1. Seleccione un flujo de usuario de registro o de inicio de sesión existente, por ejemplo *B2C_1_signupsignin1*.
1. En **Aplicación**, seleccione *webapp1*.
1. Para **Dirección URL de respuesta**, elija `https://jwt.ms`.
1. Seleccione **Ejecutar flujo de usuario**.

    ![Página Ejecutar flujo de usuario del flujo de usuario de registro o inicio de sesión en Azure Portal](media/secure-apim-with-b2c-token/portal-03-user-flow.png)

1. Complete el proceso de inicio de sesión. Debería redirigirse a `https://jwt.ms`.
1. Anote el valor del token codificado que se muestra en el explorador. Use este valor de token para el encabezado Authorization de Postman.

    ![Valor del token codificado que se muestra en jwt.ms](media/secure-apim-with-b2c-token/jwt-ms-01-token.png)

### <a name="get-api-subscription-key"></a>Obtención de la clave de suscripción de la API

Una aplicación cliente (en este caso, Postman) que llama a una API publicada debe incluir una clave de suscripción de API Management válida en sus solicitudes HTTP a la API. Para obtener una clave de suscripción para su inclusión en la solicitud HTTP de Postman:

1. Vaya a la instancia del servicio Azure API Management en [Azure Portal](https://portal.azure.com).
1. Seleccione **Suscripciones**.
1. Seleccione los puntos suspensivos de **Producto: ilimitado** y, a continuación, seleccione **Mostrar u ocultar claves**.
1. Anote la **clave principal** del producto. Esta clave se usa para el encabezado `Ocp-Apim-Subscription-Key` de la solicitud HTTP en Postman.

![Página Clave de suscripción con el botón Mostrar u ocultar claves seleccionado en Azure Portal](media/secure-apim-with-b2c-token/portal-04-api-subscription-key.png)

### <a name="test-a-secure-api-call"></a>Prueba de una llamada a la API protegida

Con el token de acceso y la clave de suscripción de APIM anotadas, ya está listo para probar si ha configurado correctamente el acceso seguro a la API.

1. Cree una nueva solicitud `GET` en [Postman](https://www.getpostman.com/). Para la dirección URL de la solicitud, especifique el punto de conexión de la lista de oradores de la API que publicó como uno de los requisitos previos. Por ejemplo:

    `https://contosoapim.azure-api.net/conference/speakers`

1. A continuación, agregue los encabezados siguientes:

    | Clave | Value |
    | --- | ----- |
    | `Authorization` | Valor del token codificado que anotó anteriormente, con el prefijo `Bearer ` (incluya el espacio después de "Bearer") |
    | `Ocp-Apim-Subscription-Key` | Clave de suscripción de APIM que anotó anteriormente |

    La dirección URL de la solicitud **GET** y los **Encabezados** deben ser similares a:

    ![Interfaz de usuario de Postman que muestra la dirección URL de la solicitud GET y los encabezados](media/secure-apim-with-b2c-token/postman-01-headers.png)

1. Seleccione el botón **Enviar** de Postman para ejecutar la solicitud. Si ha configurado todo correctamente, debe aparecer una respuesta JSON con una colección de oradores de la conferencia (aquí se muestra truncada):

    ```json
    {
      "collection": {
        "version": "1.0",
        "href": "https://conferenceapi.azurewebsites.net:443/speakers",
        "links": [],
        "items": [
          {
            "href": "https://conferenceapi.azurewebsites.net/speaker/1",
            "data": [
              {
                "name": "Name",
                "value": "Scott Guthrie"
              }
            ],
            "links": [
              {
                "rel": "http://tavis.net/rels/sessions",
                "href": "https://conferenceapi.azurewebsites.net/speaker/1/sessions"
              }
            ]
          },
    [...]
    ```

### <a name="test-an-insecure-api-call"></a>Prueba de una llamada a una API no protegida

Ahora que ha realizado una solicitud correcta, pruebe el caso de error para asegurarse de que las llamadas a la API con un token *no válido* se rechazan según lo previsto. Una manera de realizar la prueba es agregar o cambiar algunos caracteres en el valor del token y, a continuación, ejecutar la misma solicitud `GET` que antes.

1. Agregue varios caracteres al valor del token para simular un token no válido. Por ejemplo, agregue "INVALID" al valor del token:

    ![Sección Encabezados de la interfaz de usuario de Postman que muestra INVALID agregado al token](media/secure-apim-with-b2c-token/postman-02-invalid-token.png)

1. Seleccione el botón **Enviar** para ejecutar la solicitud. Con un token no válido, el resultado esperado es un código de estado no autorizado `401`:

    ```json
    {
        "statusCode": 401,
        "message": "Unauthorized. Access token is missing or invalid."
    }
    ```

Si ve el código de estado `401`, habrá comprobado que solo los autores de llamadas con un token de acceso válido emitido por Azure AD B2C pueden realizar solicitudes correctas a la API de Azure API Management.

## <a name="support-multiple-applications-and-issuers"></a>Compatibilidad con varias aplicaciones y emisores

Normalmente, interactúan varias aplicaciones con una sola API REST. Para permitir que la API acepte los tokens destinados a varias aplicaciones, agregue sus identificadores de aplicación al elemento `<audiences>` de la directiva de entrada de APIM.

```xml
<!-- Accept tokens intended for these recipient applications -->
<audiences>
    <audience>44444444-0000-0000-0000-444444444444</audience>
    <audience>66666666-0000-0000-0000-666666666666</audience>
</audiences>
```

Del mismo modo, para admitir varios emisores de tokens, agregue los identificadores URI del punto de conexión al elemento `<issuers>` de la directiva de entrada de APIM.

```xml
<!-- Accept tokens from multiple issuers -->
<issuers>
    <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
    <issuer>https://login.microsoftonline.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
</issuers>
```

## <a name="migrate-to-b2clogincom"></a>Migración a b2clogin.com

Si tiene una API de APIM que valida los tokens emitidos por el punto de conexión heredado `login.microsoftonline.com`, debe migrar la API y las aplicaciones que lo llaman para que usen tokens emitidos por [b2clogin.com](b2clogin.md).

Puede seguir este proceso general para realizar una migración por fases:

1. Agregue compatibilidad en la directiva de entrada de APIM para los tokens emitidos por b2clogin.com y login.microsoftonline.com.
1. Actualice las aplicaciones de una en una para obtener los tokens del punto de conexión b2clogin.com.
1. Una vez que todas las aplicaciones obtengan correctamente tokens de b2clogin.com, elimine de la API la compatibilidad con los tokens emitidos por login.microsoftonline.com.

En el siguiente ejemplo de directiva de entrada de APIM se muestra cómo aceptar tokens emitidos por b2clogin.com y login.microsoftonline.com. Además, admite solicitudes de API desde dos aplicaciones.

```xml
<policies>
    <inbound>
        <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
            <openid-config url="https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration" />
            <audiences>
                <audience>44444444-0000-0000-0000-444444444444</audience>
                <audience>66666666-0000-0000-0000-666666666666</audience>
            </audiences>
            <issuers>
                <issuer>https://login.microsoftonline.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
                <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
            </issuers>
        </validate-jwt>
        <base />
    </inbound>
    <backend> <base /> </backend>
    <outbound> <base /> </outbound>
    <on-error> <base /> </on-error>
</policies>
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las directivas de Azure API Management, consulte [Índice de referencia de directivas de APIM](../api-management/api-management-policies.md).

Puede encontrar información sobre cómo migrar API web basadas en OWIN y sus aplicaciones a b2clogin.com en [Migración de una API web basada en OWIN a b2clogin.com](multiple-token-endpoints.md).
