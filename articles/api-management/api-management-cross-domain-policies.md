---
title: Directivas entre dominios de Azure API Management | Microsoft Docs
description: Aprenda sobre las directivas entre dominios disponibles para su uso en Azure API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 7689d277-8abe-472a-a78c-e6d4bd43455d
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/14/2020
ms.author: apimpm
ms.openlocfilehash: 99784e43130b70554c05ff79a10993f2b6eebbde
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86499620"
---
# <a name="api-management-cross-domain-policies"></a>Directivas entre dominios de API Management
En este tema se proporciona una referencia para las siguientes directivas de API Management. Para obtener más información sobre cómo agregar y configurar directivas, consulte [Directivas en Administración de API](https://go.microsoft.com/fwlink/?LinkID=398186).

## <a name="cross-domain-policies"></a><a name="CrossDomainPolicies"></a> Directivas entre dominios

- [Permitir llamadas entre dominios](api-management-cross-domain-policies.md#AllowCrossDomainCalls) : permite que la API sea accesible desde los clientes basados en explorador de Adobe Flash y Microsoft Silverlight.
- [CORS](api-management-cross-domain-policies.md#CORS) : agrega soporte de uso compartido de recursos entre orígenes (CORS) a una operación o a una API para permitir llamadas entre dominios desde clientes basados en explorador.
- [JSONP](api-management-cross-domain-policies.md#JSONP) : agrega JSON con soporte de relleno (JSONP) a una operación o a una API para permitir llamadas entre dominios desde clientes basados en explorador de JavaScript.

## <a name="allow-cross-domain-calls"></a><a name="AllowCrossDomainCalls"></a> Allow cross-domain calls (Permitir llamadas entre dominios)
Use la directiva `cross-domain` para que la API sea accesible desde Adobe Flash y clientes basados en explorador de Microsoft Silverlight.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<cross-domain>
    <!-Policy configuration is in the Adobe cross-domain policy file format,
        see https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html-->
</cross-domain>
```

### <a name="example"></a>Ejemplo

```xml
<cross-domain>
    <cross-domain>
        <allow-http-request-headers-from domain='*' headers='*' />
    </cross-domain>
</cross-domain>
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|entre dominios|Elemento raíz. Los elementos secundarios deben ajustarse a la [especificación de archivos de directivas entre dominios de Adobe](https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html).|Sí|

### <a name="usage"></a>Uso
Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

- **Secciones de la directiva:** inbound (entrada)
- **Ámbitos de la directiva:** todos los ámbitos

## <a name="cors"></a><a name="CORS"></a> CORS
La directiva `cors` agrega compatibilidad con el uso compartido de recursos entre orígenes (CORS) a una operación o a una API para permitir llamadas entre dominios desde clientes basados en explorador.

CORS permite a un explorador y a un servidor interactuar y determinar si se permiten o no solicitudes específicas entre orígenes (por ejemplo, llamadas XMLHttpRequests realizadas desde JavaScript en una página web a otros dominios). Esto permite más flexibilidad que si solo se permiten solicitudes del mismo origen, pero es más seguro que permitir todas las solicitudes entre orígenes.

Debe aplicar la directiva CORS para habilitar la consola interactiva en el portal para desarrolladores. Consulte la [documentación del portal para desarrolladores](./api-management-howto-developer-portal.md#cors) para obtener detalles.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<cors allow-credentials="false|true">
    <allowed-origins>
        <origin>origin uri</origin>
    </allowed-origins>
    <allowed-methods preflight-result-max-age="number of seconds">
        <method>http verb</method>
    </allowed-methods>
    <allowed-headers>
        <header>header name</header>
    </allowed-headers>
    <expose-headers>
        <header>header name</header>
    </expose-headers>
</cors>
```

### <a name="example"></a>Ejemplo
En este ejemplo se muestra cómo admitir solicitudes preparatorias, como aquellas con encabezados o métodos personalizados distintos de GET y POST. Para admitir encabezados personalizados y verbos HTTP adicionales, use las secciones `allowed-methods` y `allowed-headers` tal como se muestra en el ejemplo siguiente.

```xml
<cors allow-credentials="true">
    <allowed-origins>
        <!-- Localhost useful for development -->
        <origin>http://localhost:8080/</origin>
        <origin>http://example.com/</origin>
    </allowed-origins>
    <allowed-methods preflight-result-max-age="300">
        <method>GET</method>
        <method>POST</method>
        <method>PATCH</method>
        <method>DELETE</method>
    </allowed-methods>
    <allowed-headers>
        <!-- Examples below show Azure Mobile Services headers -->
        <header>x-zumo-installation-id</header>
        <header>x-zumo-application</header>
        <header>x-zumo-version</header>
        <header>x-zumo-auth</header>
        <header>content-type</header>
        <header>accept</header>
    </allowed-headers>
    <expose-headers>
        <!-- Examples below show Azure Mobile Services headers -->
        <header>x-zumo-installation-id</header>
        <header>x-zumo-application</header>
    </expose-headers>
</cors>
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|cors|Elemento raíz.|Sí|No aplicable|
|allowed-origins|Contiene elementos `origin` que describen los orígenes permitidos para las solicitudes entre dominios. `allowed-origins` puede contener un único elemento `origin` que especifica `*` para permitir cualquier origen, o uno o varios elementos `origin` que contienen un identificador URI.|Sí|No aplicable|
|origin|El valor puede ser `*` para permitir todos los orígenes o un identificador URI que especifica un único origen. El URI debe incluir un esquema, un host y un puerto.|Sí|Si se omite el puerto en un identificador URI, se usa el puerto 80 para HTTP y el puerto 443 para HTTPS.|
|allowed-methods|Este elemento es necesario si se permiten métodos distintos de GET o POST. Contiene elementos `method` que especifican los verbos HTTP admitidos. El valor `*` indica todos los métodos.|No|Si esta sección no está presente, se admiten GET y POST.|
|method|Especifica un verbo HTTP.|Al menos un elemento `method` es necesario si la sección `allowed-methods` está presente.|N/D|
|allowed-headers|Este elemento contiene elementos `header` que especifican los nombres de los encabezados que pueden incluirse en la solicitud.|No|N/D|
|expose-headers|Este elemento contiene elementos `header` que especifican los nombres de los encabezados a los que tendrá acceso el cliente.|No|N/D|
|header|Especifica un nombre de encabezado.|Al menos un elemento `header` es necesario en `allowed-headers` o `expose-headers` si está presente la sección.|N/D|

### <a name="attributes"></a>Atributos

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|allow-credentials|El encabezado `Access-Control-Allow-Credentials` de la respuesta preparatoria se establecerá en el valor de este atributo e influirá en la capacidad del cliente de enviar credenciales en solicitudes entre dominios.|No|false|
|preflight-result-max-age|El encabezado `Access-Control-Max-Age` de la respuesta preparatoria se establecerá en el valor de este atributo e influirá en la capacidad del agente del usuario de almacenar en caché la respuesta preparatoria.|No|0|

### <a name="usage"></a>Uso
Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

- **Secciones de la directiva:** inbound (entrada)
- **Ámbitos de la directiva:** todos los ámbitos

## <a name="jsonp"></a><a name="JSONP"></a> JSONP
La directiva `jsonp` agrega JSON con compatibilidad con relleno (JSONP) a una operación o a una API para permitir llamadas entre dominios desde clientes basados en explorador de JavaScript. JSONP es un método utilizado en los programas JavaScript para solicitar datos desde un servidor en un dominio diferente. JSONP sortea la limitación exigida por la mayoría de los exploradores web donde el acceso a las páginas web debe estar en el mismo dominio.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<jsonp callback-parameter-name="callback function name" />
```

### <a name="example"></a>Ejemplo

```xml
<jsonp callback-parameter-name="cb" />
```

Si llama al método sin el parámetro de devolución de llamada ?cb=XXX, devolverá JSON sin formato (sin un envoltorio de llamada de función).

Si agrega el parámetro de devolución de llamada `?cb=XXX`, devolverá un resultado JSONP, envolviendo los resultados JSON originales en torno a la función de devolución de llamada como `XYZ('<json result goes here>');`.

### <a name="elements"></a>Elementos

|Nombre|Descripción|Requerido|
|----------|-----------------|--------------|
|jsonp|Elemento raíz.|Sí|

### <a name="attributes"></a>Atributos

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|callback-parameter-name|La llamada de función de JavaScript entre dominios prefijada con el nombre de dominio completo en donde reside la función.|Sí|No aplicable|

### <a name="usage"></a>Uso
Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

- **Secciones de la directiva:** outbound
- **Ámbitos de la directiva:** todos los ámbitos

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo trabajar con directivas, consulte:

+ [Directivas de Azure API Management](api-management-howto-policies.md)
+ [API de transformación](transform-api.md)
+ En la [Referencia de directivas](./api-management-policies.md) se muestra una lista completa de declaraciones de directivas y su configuración
+ [Ejemplos de directivas](policy-samples.md)
