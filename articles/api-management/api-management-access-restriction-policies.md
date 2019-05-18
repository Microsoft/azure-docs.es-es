---
title: Directivas de restricción de acceso de Azure API Management | Microsoft Docs
description: Aprenda sobre las directivas de restricción de acceso disponibles para su uso en Azure API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 034febe3-465f-4840-9fc6-c448ef520b0f
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/21/2019
ms.author: apimpm
ms.openlocfilehash: b8c564ef2de22555930f998ccd9918b252d35f17
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65541692"
---
# <a name="api-management-access-restriction-policies"></a>Directivas de restricción de acceso de API Management

En este tema se proporciona una referencia para las siguientes directivas de API Management. Para obtener más información sobre cómo agregar y configurar directivas, consulte [Directivas en Administración de API](https://go.microsoft.com/fwlink/?LinkID=398186).

## <a name="AccessRestrictionPolicies"></a> Directivas de restricción de acceso

-   [Activar encabezado HTTP](api-management-access-restriction-policies.md#CheckHTTPHeader) : aplica la existencia o el valor de un encabezado HTTP.
-   [Limitar la frecuencia de llamadas por suscripción](api-management-access-restriction-policies.md#LimitCallRate) : evita los picos de uso de la API limitando la frecuencia de llamadas, por suscripción.
-   [Limitar la frecuencia de llamadas por clave](#LimitCallRateByKey) : evita los picos de uso de la API limitando la frecuencia de llamadas, por clave.
-   [Restringir IP de autor de llamada](api-management-access-restriction-policies.md#RestrictCallerIPs) : filtra (permite/deniega) las llamadas de direcciones IP específicas o de intervalos de direcciones.
-   [Establecer cuota de uso por suscripción](api-management-access-restriction-policies.md#SetUsageQuota) : le permite aplicar un volumen de llamadas renovables o permanentes o una cuota de ancho de banda por suscripción.
-   [Establecer cuota de uso por clave](#SetUsageQuotaByKey) : le permite aplicar un volumen de llamadas renovables o permanentes o una cuota de ancho de banda por clave.
-   [Validar JWT](api-management-access-restriction-policies.md#ValidateJWT) : aplica la existencia y la validez de un JWT extraído de un encabezado HTTP especificado o un parámetro de consulta especificado.

## <a name="CheckHTTPHeader"></a> Activar encabezado HTTP

Usa la directiva `check-header` para exigir que una solicitud tenga un encabezado HTTP especificado. Si lo desea, puede comprobar si el encabezado tiene un valor específico o un intervalo de valores permitidos. Si el resultado de la comprobación es negativo, la directiva finaliza el procesamiento de la solicitud y devuelve el mensaje de error y el código de estado HTTP que especifica.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<check-header name="header name" failed-check-httpcode="code" failed-check-error-message="message" ignore-case="true">
    <value>Value1</value>
    <value>Value2</value>
</check-header>
```

### <a name="example"></a>Ejemplo

```xml
<check-header name="Authorization" failed-check-httpcode="401" failed-check-error-message="Not authorized" ignore-case="false">
    <value>f6dc69a089844cf6b2019bae6d36fac8</value>
</check-header>
```

### <a name="elements"></a>Elementos

| NOMBRE         | DESCRIPCIÓN                                                                                                                                   | Obligatorio |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| check-header | Elemento raíz.                                                                                                                                 | Sí      |
| value        | Valor del encabezado HTTP permitido. Cuando se especifican varios elementos de valor, el resultado de la comprobación se considera positivo en caso de coincidencia con cualquiera de los valores. | No       |

### <a name="attributes"></a>Atributos

| NOMBRE                       | DESCRIPCIÓN                                                                                                                                                            | Obligatorio | Valor predeterminado |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| failed-check-error-message | Mensaje de error que se devuelve en el cuerpo de la respuesta HTTP si el encabezado no existe o tiene un valor no válido. Los caracteres especiales de este mensaje deben incluir los caracteres de escape correctos. | Sí      | N/D     |
| failed-check-httpcode      | Código de estado HTTP que se devuelve si el encabezado no existe o tiene un valor no válido.                                                                                        | Sí      | N/D     |
| header-name                | Nombre del encabezado HTTP que hay que comprobar.                                                                                                                                  | Sí      | N/D     |
| ignore-case                | Puede establecerse en True o False. Si se establece en True, se omite la presencia de mayúsculas/minúsculas cuando el valor del encabezado se compara con el conjunto de valores aceptables.                                    | Sí      | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections) y [ámbitos](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes) de directiva.

-   **Secciones de la directiva:** entrante y saliente

-   **Ámbitos de la directiva:** global, producto, API, operación

## <a name="LimitCallRate"></a> Limitar la tasa de llamadas por suscripción

La directiva `rate-limit` evita los picos de uso de la API según suscripción limitando la tasa de llamadas a un número especificado por un período de tiempo establecido. Cuando se desencadena esta directiva, el autor de la llamada recibe un código de estado de respuesta `429 Too Many Requests`.

> [!IMPORTANT]
> Esta directiva se puede usar una sola vez por documento de directiva.
>
> Las [expresiones de directiva](api-management-policy-expressions.md) no se pueden usar en ninguno de los atributos de esta directiva.

> [!CAUTION]
> Debido a la naturaleza distribuida de la arquitectura de limitación, la limitación de velocidad nunca es completamente exacta. La diferencia entre configurado y el número real de permite varían las solicitudes en función de volumen de solicitudes y velocidad, latencia de back-end y otros factores.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<rate-limit calls="number" renewal-period="seconds">
    <api name="API name" id="API id" calls="number" renewal-period="seconds" />
        <operation name="operation name" id="operation id" calls="number" renewal-period="seconds" />
    </api>
</rate-limit>
```

### <a name="example"></a>Ejemplo

```xml
<policies>
    <inbound>
        <base />
        <rate-limit calls="20" renewal-period="90" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

| NOMBRE      | DESCRIPCIÓN                                                                                                                                                                                                                                                                                              | Obligatorio |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| set-limit | Elemento raíz.                                                                                                                                                                                                                                                                                            | Sí      |
| api       | Agregue uno o varios de estos elementos para imponer un límite de tasa de llamadas de API dentro del producto. Los límites de tasa de llamadas a la API y al producto se aplican de forma independiente. Se puede hacer referencia a la API a través de `name` o `id`. Si se proporcionan ambos atributos, `id` se usará y `name` se omitirá.                    | No       |
| operación | Agregue uno o varios de estos elementos para imponer un límite de tasa de llamadas de las operaciones dentro de una API. Los límites de tasa de llamadas se aplican de forma independiente a la API, a la operación y al producto. Se puede hacer referencia a la operación a través de `name` o `id`. Si se proporcionan ambos atributos, `id` se usará y `name` se omitirá. | No       |

### <a name="attributes"></a>Atributos

| NOMBRE           | DESCRIPCIÓN                                                                                           | Obligatorio | Valor predeterminado |
| -------------- | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| Nombre           | Nombre de la API a la que se va a aplicar un límite de tasa.                                                | Sí      | N/D     |
| calls          | Número total máximo de llamadas permitidas durante el intervalo de tiempo especificado en `renewal-period`. | Sí      | N/D     |
| renewal-period | Período de tiempo en segundos tras el cual se restablece la cuota.                                              | Sí      | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections) y [ámbitos](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)

-   **Ámbitos de la directiva:** producto

## <a name="LimitCallRateByKey"></a> Limitar la tasa de llamadas por clave

> [!IMPORTANT]
> Esta característica no está disponible en el nivel **Consumo** de API Management.

La directiva `rate-limit-by-key` evita los picos de uso de la API según clave limitando la tasa de llamadas a un número especificado por un período de tiempo establecido. La clave puede tener un valor de cadena arbitrario y normalmente se proporciona mediante una expresión de directiva. Puede agregarse una condición de incremento opcional para especificar qué solicitudes se deben contar para este límite. Cuando se desencadena esta directiva, el autor de la llamada recibe un código de estado de respuesta `429 Too Many Requests`.

Para obtener más información y ver ejemplos de esta directiva, consulte [Limitación avanzada de solicitudes con Azure API Management](https://azure.microsoft.com/documentation/articles/api-management-sample-flexible-throttling/).

> [!CAUTION]
> Debido a la naturaleza distribuida de la arquitectura de limitación, la limitación de velocidad nunca es completamente exacta. La diferencia entre configurado y el número real de permite varían las solicitudes en función de volumen de solicitudes y velocidad, latencia de back-end y otros factores.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<rate-limit-by-key calls="number"
                   renewal-period="seconds"
                   increment-condition="condition"
                   counter-key="key value" />

```

### <a name="example"></a>Ejemplo

En el ejemplo siguiente, la clave del límite de velocidad se establece según la dirección IP del autor de la llamada.

```xml
<policies>
    <inbound>
        <base />
        <rate-limit-by-key  calls="10"
              renewal-period="60"
              increment-condition="@(context.Response.StatusCode == 200)"
              counter-key="@(context.Request.IpAddress)"/>
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

| NOMBRE      | DESCRIPCIÓN   | Obligatorio |
| --------- | ------------- | -------- |
| set-limit | Elemento raíz. | Sí      |

### <a name="attributes"></a>Atributos

| NOMBRE                | DESCRIPCIÓN                                                                                           | Obligatorio | Valor predeterminado |
| ------------------- | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| llamadas               | Número total máximo de llamadas permitidas durante el intervalo de tiempo especificado en `renewal-period`. | Sí      | N/D     |
| counter-key         | Clave que se usa para la directiva de límite de tasa.                                                             | Sí      | N/D     |
| increment-condition | Expresión booleana que especifica si la solicitud se debe contar para la cuota (`true`).        | No       | N/D     |
| renewal-period      | Período de tiempo en segundos tras el cual se restablece la cuota.                                              | Sí      | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections) y [ámbitos](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes) de directiva.

-   **Secciones de la directiva:** entrante

-   **Ámbitos de la directiva:** global, producto, API, operación

## <a name="RestrictCallerIPs"></a> Restringir IP de autor de llamada

La directiva `ip-filter` filtra (permite/deniega) llamadas de direcciones IP específicas o de intervalos de direcciones.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<ip-filter action="allow | forbid">
    <address>address</address>
    <address-range from="address" to="address" />
</ip-filter>
```

### <a name="example"></a>Ejemplo

En el ejemplo siguiente, la directiva solo permite solicitudes que entran desde la única dirección IP o intervalo de direcciones IP especificadas

```xml
<ip-filter action="allow">
    <address>13.66.201.169</address>
    <address-range from="13.66.140.128" to="13.66.140.143" />
</ip-filter>
```

### <a name="elements"></a>Elementos

| NOMBRE                                      | DESCRIPCIÓN                                         | Obligatorio                                                       |
| ----------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------- |
| ip-filter                                 | Elemento raíz.                                       | Sí                                                            |
| dirección                                   | Especifica la dirección IP única por la que filtrar.   | Es necesario al menos un elemento `address` o `address-range`. |
| address-range from="address" to="address" | Especifica un intervalo de direcciones IP por el que filtrar. | Es necesario al menos un elemento `address` o `address-range`. |

### <a name="attributes"></a>Atributos

| NOMBRE                                      | DESCRIPCIÓN                                                                                 | Obligatorio                                           | Valor predeterminado |
| ----------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------- | ------- |
| address-range from="address" to="address" | Un intervalo de direcciones IP a las que permitir o denegar el acceso.                                        | Obligatorio cuando se utiliza el elemento `address-range`. | N/D     |
| ip-filter action="allow &#124; forbid"    | Especifica si se deben permitir o no las llamadas para los intervalos y direcciones IP especificados. | Sí                                                | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections) y [ámbitos](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes) de directiva.

-   **Secciones de la directiva:** entrante
-   **Ámbitos de la directiva:** global, producto, API, operación

## <a name="SetUsageQuota"></a> Establecer cuota de uso por suscripción

La directiva `quota` aplica un volumen de llamadas o una cuota de ancho de banda renovables o permanentes por suscripción.

> [!IMPORTANT]
> Esta directiva se puede usar una sola vez por documento de directiva.
>
> Las [expresiones de directiva](api-management-policy-expressions.md) no se pueden usar en ninguno de los atributos de esta directiva.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
    <api name="API name" id="API id" calls="number" renewal-period="seconds" />
        <operation name="operation name" id="operation id" calls="number" renewal-period="seconds" />
    </api>
</quota>
```

### <a name="example"></a>Ejemplo

```xml
<policies>
    <inbound>
        <base />
        <quota calls="10000" bandwidth="40000" renewal-period="3600" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

| NOMBRE      | DESCRIPCIÓN                                                                                                                                                                                                                                                                                  | Obligatorio |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| quota     | Elemento raíz.                                                                                                                                                                                                                                                                                | Sí      |
| api       | Agregue uno o varios de estos elementos para imponer una cuota de llamadas de API dentro del producto. Las cuotas de llamada de API y de producto se aplican de forma independiente. Se puede hacer referencia a la API a través de `name` o `id`. Si se proporcionan ambos atributos, `id` se usará y `name` se omitirá.                    | No       |
| operación | Agregue uno o varios de estos elementos para imponer una cuota de llamadas de las operaciones dentro de una API. Las cuotas de llamadas de API, operación y producto se aplican de forma independiente. Se puede hacer referencia a la operación a través de `name` o `id`. Si se proporcionan ambos atributos, `id` se usará y `name` se omitirá. | No       |

### <a name="attributes"></a>Atributos

| NOMBRE           | DESCRIPCIÓN                                                                                               | Obligatorio                                                         | Valor predeterminado |
| -------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------- |
| Nombre           | Nombre de la API u operación a la que se aplica la cuota.                                             | Sí                                                              | N/D     |
| bandwidth      | Número total máximo de kilobytes permitidos durante el intervalo de tiempo especificado en `renewal-period`. | Debe especificarse `calls`, `bandwidth` o ambos. | N/D     |
| calls          | Número total máximo de llamadas permitidas durante el intervalo de tiempo especificado en `renewal-period`.     | Debe especificarse `calls`, `bandwidth` o ambos. | N/D     |
| renewal-period | Período de tiempo en segundos tras el cual se restablece la cuota.                                                  | Sí                                                              | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections) y [ámbitos](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)
-   **Ámbitos de la directiva:** producto

## <a name="SetUsageQuotaByKey"></a>Establecer cuota de uso por clave

> [!IMPORTANT]
> Esta característica no está disponible en el nivel **Consumo** de API Management.

La directiva `quota-by-key` aplica un volumen de llamadas o una cuota de ancho de banda por clave renovables o permanentes. La clave puede tener un valor de cadena arbitrario y normalmente se proporciona mediante una expresión de directiva. Puede agregarse una condición de incremento opcional para especificar qué solicitudes se cuentan para esta cuota. Si varias directivas incrementan el mismo valor de clave, se incrementa solo una vez por solicitud. Cuando se alcanza el límite de llamadas, el autor de la llamada recibe un código de estado de respuesta `403 Forbidden`.

Para obtener más información y ver ejemplos de esta directiva, consulte [Limitación avanzada de solicitudes con Azure API Management](https://azure.microsoft.com/documentation/articles/api-management-sample-flexible-throttling/).

> Las [expresiones de directiva](api-management-policy-expressions.md) no se pueden usar en ninguno de los atributos de esta directiva.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<quota-by-key calls="number"
              bandwidth="kilobytes"
              renewal-period="seconds"
              increment-condition="condition"
              counter-key="key value" />

```

### <a name="example"></a>Ejemplo

En el ejemplo siguiente, la clave de la cuota se establece según la dirección IP del autor de la llamada.

```xml
<policies>
    <inbound>
        <base />
        <quota-by-key calls="10000" bandwidth="40000" renewal-period="3600"
                      increment-condition="@(context.Response.StatusCode >= 200 && context.Response.StatusCode < 400)"
                      counter-key="@(context.Request.IpAddress)" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

| NOMBRE  | DESCRIPCIÓN   | Obligatorio |
| ----- | ------------- | -------- |
| quota | Elemento raíz. | Sí      |

### <a name="attributes"></a>Atributos

| NOMBRE                | DESCRIPCIÓN                                                                                               | Obligatorio                                                         | Valor predeterminado |
| ------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------- |
| ancho de banda           | Número total máximo de kilobytes permitidos durante el intervalo de tiempo especificado en `renewal-period`. | Debe especificarse `calls`, `bandwidth` o ambos. | N/D     |
| calls               | Número total máximo de llamadas permitidas durante el intervalo de tiempo especificado en `renewal-period`.     | Debe especificarse `calls`, `bandwidth` o ambos. | N/D     |
| counter-key         | Clave que se usa para la directiva de cuota.                                                                      | Sí                                                              | N/D     |
| increment-condition | Expresión booleana que especifica si la solicitud se debe contar para la cuota (`true`).             | No                                                               | N/D     |
| renewal-period      | Período de tiempo en segundos tras el cual se restablece la cuota.                                                  | Sí                                                              | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections) y [ámbitos](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes) de directiva.

-   **Secciones de la directiva:** entrante
-   **Ámbitos de la directiva:** global, producto, API, operación

## <a name="ValidateJWT"></a> Validación de JWT

La directiva `validate-jwt` aplica la existencia y la validez de un JWT extraído de un encabezado HTTP o un parámetro de consulta especificados.

> [!IMPORTANT]
> La directiva `validate-jwt` requiere que la notificación registrada `exp` se incluya en el token de JWT, a menos que se especifique el atributo `require-expiration-time` y se establezca en `false`.
> La directiva `validate-jwt` es compatible con los algoritmos de firma HS256 y RS256. En el caso de HS256, la clave debe proporcionarse en línea dentro de la directiva con el formato de codificación Base64. En el caso de RS256, la clave debe proporcionarse a través de un punto de conexión de configuración OpenID.
> La directiva `validate-jwt` admite tokens cifrados con claves simétricas que utilicen los siguientes algoritmos de cifrado: A128CBC-HS256, A192CBC-HS384 y A256CBC-HS512.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<validate-jwt
    header-name="name of http header containing the token (use query-parameter-name attribute if the token is passed in the URL)"
    failed-validation-httpcode="http status code to return on failure"
    failed-validation-error-message="error message to return on failure"
    token-value="expression returning JWT token as a string"
    require-expiration-time="true|false"
    require-scheme="scheme"
    require-signed-tokens="true|false"
    clock-skew="allowed clock skew in seconds"
    output-token-variable-name="name of a variable to receive a JWT object representing successfully validated token">
  <issuer-signing-keys>
    <key>base64 encoded signing key</key>
    <!-- if there are multiple keys, then add additional key elements -->
  </issuer-signing-keys>
  <decryption-keys>
    <key>base64 encoded signing key</key>
    <!-- if there are multiple keys, then add additional key elements -->
  </decryption-keys>
  <audiences>
    <audience>audience string</audience>
    <!-- if there are multiple possible audiences, then add additional audience elements -->
  </audiences>
  <issuers>
    <issuer>issuer string</issuer>
    <!-- if there are multiple possible issuers, then add additional issuer elements -->
  </issuers>
  <required-claims>
    <claim name="name of the claim as it appears in the token" match="all|any" separator="separator character in a multi-valued claim">
      <value>claim value as it is expected to appear in the token</value>
      <!-- if there is more than one allowed values, then add additional value elements -->
    </claim>
    <!-- if there are multiple possible allowed values, then add additional value elements -->
  </required-claims>
  <openid-config url="full URL of the configuration endpoint, e.g. https://login.constoso.com/openid-configuration" />
  <zumo-master-key id="key identifier">key value</zumo-master-key>
</validate-jwt>

```

### <a name="examples"></a>Ejemplos

#### <a name="simple-token-validation"></a>Validación de token simple

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key>  <!-- signing key specified as a named value -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>  <!-- audience is set to API Management host name -->
    </audiences>
    <issuers>
        <issuer>http://contoso.com/</issuer>
    </issuers>
</validate-jwt>
```

#### <a name="azure-active-directory-token-validation"></a>Validación de tokens de Azure Active Directory

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/contoso.onmicrosoft.com/.well-known/openid-configuration" />
    <audiences>
        <audience>25eef6e4-c905-4a07-8eb4-0d08d5df8b3f</audience>
    </audiences>
    <required-claims>
        <claim name="id" match="all">
            <value>insert claim here</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### <a name="azure-active-directory-b2c-token-validation"></a>Validación de tokens de Azure Active Directory B2C

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/b2c_1_signin/v2.0/.well-known/openid-configuration" />
    <audiences>
        <audience>d313c4e4-de5f-4197-9470-e509a2f0b806</audience>
    </audiences>
    <required-claims>
        <claim name="id" match="all">
            <value>insert claim here</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### <a name="authorize-access-to-operations-based-on-token-claims"></a>Autorización de acceso a las operaciones según las notificaciones de tokens

En este ejemplo se muestra cómo usar el [validar JWT](api-management-access-restriction-policies.md#ValidateJWT) basada en directivas para autorizar el acceso a las operaciones en el valor de las notificaciones de tokens.

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer" output-token-variable-name="jwt">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key> <!-- signing key is stored in a named value -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>
    </audiences>
    <issuers>
        <issuer>contoso.com</issuer>
    </issuers>
    <required-claims>
        <claim name="group" match="any">
            <value>finance</value>
            <value>logistics</value>
        </claim>
    </required-claims>
</validate-jwt>
<choose>
    <when condition="@(context.Request.Method == "POST" && !((Jwt)context.Variables["jwt"]).Claims["group"].Contains("finance"))">
        <return-response>
            <set-status code="403" reason="Forbidden" />
        </return-response>
    </when>
</choose>
```

#### <a name="azure-mobile-services-token-validation"></a>Validación de tokens de Azure Mobile Services

```xml
<validate-jwt header-name="x-zumo-auth" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Supplied access token is invalid.">
    <issuers>
        <issuer>urn:microsoft:windows-azure:zumo</issuer>
    </issuers>
    <audiences>
        <audience>Facebook</audience>
    </audiences>
    <issuer-signing-keys>
        <zumo-master-key id="0">insert key here</zumo-master-key>
    </issuer-signing-keys>
</validate-jwt>
```

### <a name="elements"></a>Elementos

| Elemento             | DESCRIPCIÓN                                                                                                                                                                                                                                                                                                                                           | Obligatorio |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-jwt        | Elemento raíz.                                                                                                                                                                                                                                                                                                                                         | Sí      |
| audiences           | Contiene una lista de notificaciones de audiencia aceptables que pueden estar presentes en el token. Si existen varios valores de audiencia, se prueban los valores uno a uno hasta que se agoten todos (en cuyo caso no se superará la validación) o hasta que se obtenga un resultado positivo con alguno. Debe especificarse al menos una audiencia.                                                                     | No       |
| issuer-signing-keys | Lista de las claves de seguridad con codificación Base64 que se utilizan para validar los tokens firmados. Si existen varias claves de seguridad, se prueban las claves una a una hasta que se agoten todas (en cuyo caso no se superará la validación) o hasta que una sea correcta (lo que es útil para la sustitución de tokens). Los elementos de clave tienen un atributo `id` opcional que se utiliza para compararlo con la notificación `kid`.               | No       |
| decryption-keys     | Una lista de claves con codificación Base64 que se usan para descifrar los tokens. Si existen varias claves de seguridad, se prueba cada clave hasta que se agoten todas (en cuyo caso no se superará la validación) o hasta que una sea correcta. Los elementos de clave tienen un atributo `id` opcional que se utiliza para compararlo con la notificación `kid`.                                                 | No       |
| issuers             | Lista de entidades de seguridad aceptables que emitieron el token. Si existen varios valores de emisor, se prueban los valores uno a uno hasta que se agoten todos (en cuyo caso no se superará la validación) o hasta que se obtenga un resultado positivo con alguno.                                                                                                                                         | No       |
| openid-config       | Elemento que se usa para especificar un punto de conexión de configuración OpenID compatible desde el que se puedan obtener las claves y el emisor de la firma.                                                                                                                                                                                                                        | No       |
| required-claims     | Contiene una lista de las notificaciones que se espera que estén presentes en el token para que se considere válido. Cuando el atributo `match` está establecido en `all`, todos los valores de notificación de la directiva deben estar presentes en el token para que la validación se efectúe correctamente. Cuando el atributo `match` está establecido en `any`, debe haber al menos una notificación en el token para que la validación se efectúe correctamente. | No       |
| zumo-master-key     | Clave maestra para los tokens que emite Azure Mobile Services.                                                                                                                                                                                                                                                                                                 | No       |

### <a name="attributes"></a>Atributos

| NOMBRE                            | DESCRIPCIÓN                                                                                                                                                                                                                                                                                                                                                                                                                                            | Obligatorio                                                                         | Valor predeterminado                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| clock-skew                      | Intervalo de tiempo. Utilice esta opción para especificar la diferencia máxima de tiempo esperado entre los relojes del sistema del emisor del token y la instancia de API Management.                                                                                                                                                                                                                                                                                                               | No                                                                               | 0 segundos                                                                         |
| failed-validation-error-message | Mensaje de error que se devuelve en el cuerpo de respuesta HTTP si el elemento JWT no pasa la validación. Los caracteres especiales de este mensaje deben incluir los caracteres de escape correctos.                                                                                                                                                                                                                                                                                                 | No                                                                               | El mensaje de error predeterminado depende del problema de validación, por ejemplo, la notificación de la ausencia del elemento JWT. |
| failed-validation-httpcode      | Código de estado HTTP que se devuelve si el elemento JWT no pasa la validación.                                                                                                                                                                                                                                                                                                                                                                                         | No                                                                               | 401                                                                               |
| header-name                     | El nombre del encabezado HTTP que contiene el token.                                                                                                                                                                                                                                                                                                                                                                                                         | Uno de `header-name`, `query-parameter-name` o `token-value` debe especificarse. | N/D                                                                               |
| nombre del parámetro de consulta            | Nombre del parámetro de consulta que contiene el token.                                                                                                                                                                                                                                                                                                                                                                                                     | Uno de `header-name`, `query-parameter-name` o `token-value` debe especificarse. | N/D                                                                               |
| token-value                     | Expresión que devuelve una cadena que contiene el token de JWT                                                                                                                                                                                                                                                                                                                                                                                                     | Uno de `header-name`, `query-parameter-name` o `token-value` debe especificarse. | N/D                                                                               |
| id                              | El atributo `id` del elemento `key` le permite especificar la cadena que se comparará con la notificación `kid` del token (si existe) para averiguar qué clave debe usarse para validar la firma.                                                                                                                                                                                                                                           | No                                                                               | N/D                                                                               |
| match                           | El atributo `match` del elemento `claim` especifica si todos los valores de notificación de la directiva deben estar presentes en el token para que la validación se efectúe correctamente. Los valores posibles son:<br /><br /> - `all`: todos los valores de notificación de la directiva deben estar presentes en el token para que la validación se efectúe correctamente.<br /><br /> - `any`: al menos un valor de notificación debe estar presente en el token para que la validación se efectúe correctamente.                                                       | No                                                                               | todas                                                                               |
| require-expiration-time         | Booleano. Especifica si es necesaria una notificación de expiración en el token.                                                                                                                                                                                                                                                                                                                                                                               | No                                                                               | true                                                                              |
| require-scheme                  | El nombre del token de esquema; por ejemplo, "Bearer". Cuando se establece este atributo, la directiva se asegurará de que ese esquema especificado esté presente en el valor del encabezado de la autorización.                                                                                                                                                                                                                                                                                    | No                                                                               | N/D                                                                               |
| require-signed-tokens           | Booleano. Especifica si un token debe estar firmado.                                                                                                                                                                                                                                                                                                                                                                                           | No                                                                               | true                                                                              |
| separator                       | Cadena. Especifica el separador (por ejemplo: ",") que se va a usar para extraer un conjunto de valores de una notificación con varios valores.                                                                                                                                                                                                                                                                                                                                          | No                                                                               | N/D                                                                               |
| url                             | Dirección URL de punto de conexión de configuración de OpenID desde donde se pueden obtener los metadatos de configuración de OpenID. La respuesta debe ser acorde a las especificaciones definidas en la dirección URL:`https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata`. En Azure Active Directory, utilice la dirección URL `https://login.microsoftonline.com/{tenant-name}/.well-known/openid-configuration` y sustituya tenant-name por el nombre del inquilino de directorio, por ejemplo, `contoso.onmicrosoft.com`. | Sí                                                                              | N/D                                                                               |
output-token-variable-name|Cadena. Nombre de variable de contexto que recibirá el valor del token como un objeto de tipo [ `Jwt` ](api-management-policy-expressions.md) tras la correcta validación de tokens|No|N/D

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections) y [ámbitos](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes) de directiva.

-   **Secciones de la directiva:** entrante
-   **Ámbitos de la directiva:** global, producto, API, operación

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo trabajar con directivas, consulte:

-   [Directivas de Azure API Management](api-management-howto-policies.md)
-   [API de transformación](transform-api.md)
-   En la [Referencia de directivas](api-management-policy-reference.md) se muestra una lista completa de declaraciones de directivas y su configuración
-   [Ejemplos de directivas](policy-samples.md)
