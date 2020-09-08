---
title: Expresiones de las directivas de Azure API Management | Microsoft Docs
description: Obtenga información acerca de las expresiones de las directivas de Azure API Management. Vea ejemplos y examine los recursos adicionales disponibles.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: ea160028-fc04-4782-aa26-4b8329df3448
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/22/2019
ms.author: apimpm
ms.openlocfilehash: 7117ffcbaf4eba8d83a6e968f4fed7422673610b
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844247"
---
# <a name="api-management-policy-expressions"></a>Expresiones de las directivas de API Management
En este artículo se describe la sintaxis de expresiones de directiva en C# 7. Cada expresión tiene acceso a la variable de [contexto](api-management-policy-expressions.md#ContextVariables) proporcionada de forma implícita y a un [subconjunto](api-management-policy-expressions.md#CLRTypes) permitido de tipos de .NET Framework.

Para obtener más información:

- Vea cómo proporcionar información de contexto al servicio back-end. Use las directivas [Establecer el parámetro de cadena de consulta](api-management-transformation-policies.md#SetQueryStringParameter) y [Establecer encabezado HTTP](api-management-transformation-policies.md#SetHTTPheader) para proporcionar esta información.
- Vea cómo utilizar la directiva de [validación de JWT](api-management-access-restriction-policies.md#ValidateJWT) para preautorizar el acceso a operaciones según notificaciones de token.
- Vea cómo se utiliza un seguimiento de [API Inspector](./api-management-howto-api-inspector.md) para ver cómo se evalúan las directivas y los resultados de las evaluaciones.
- Vea cómo usar expresiones con las directivas [Obtener de caché](api-management-caching-policies.md#GetFromCache) y [Almacenar en caché](api-management-caching-policies.md#StoreToCache) para configurar el almacenamiento en caché de la respuesta de API Management. Defina una duración que coincida con el almacenamiento en caché de la respuesta del servicio back-end especificado por la directiva `Cache-Control` del servicio back-end.
- Vea cómo realizar el filtrado de contenido. Quite elementos de datos de la respuesta recibida desde el servicio back-end mediante las directivas [Flujo de control](api-management-advanced-policies.md#choose) y [Establecer cuerpo](api-management-transformation-policies.md#SetBody).
- Para descargar las declaraciones de directiva, vaya a [api-management-samples/policies](https://github.com/Azure/api-management-samples/tree/master/policies) en el repositorio de GitHub.


## <a name="syntax"></a><a name="Syntax"></a> Sintaxis
Las expresiones de declaración única se incluyen en `@(expression)`, donde `expression` es una instrucción de expresión bien formada de C#.

Las expresiones de múltiples declaraciones se incluyen en `@{expression}`. Todas las rutas de código de las expresiones de múltiples declaraciones deben terminar con una declaración `return`.

## <a name="examples"></a><a name="PolicyExpressionsExamples"></a> Ejemplos

```
@(true)

@((1+1).ToString())

@("Hi There".Length)

@(Regex.Match(context.Response.Headers.GetValueOrDefault("Cache-Control",""), @"max-age=(?<maxAge>\d+)").Groups["maxAge"]?.Value)

@(context.Variables.ContainsKey("maxAge") ? int.Parse((string)context.Variables["maxAge"]) : 3600)

@{
  string[] value;
  if (context.Request.Headers.TryGetValue("Authorization", out value))
  {
      if(value != null && value.Length > 0)
      {
          return Encoding.UTF8.GetString(Convert.FromBase64String(value[0]));
      }
  }
  return null;

}
```

## <a name="usage"></a><a name="PolicyExpressionsUsage"></a>Uso
Las expresiones pueden utilizarse como valores de atributos o valores de texto en cualquier [directiva](api-management-policies.md) de API Management, a menos que la referencia de la directiva especifique lo contrario.

> [!IMPORTANT]
> Al utilizar expresiones de directiva, solo hay una verificación limitada de estas cuando se define la directiva. La puerta de enlace ejecuta las expresiones en tiempo de ejecución, y las excepciones generadas por las expresiones de directiva generan un error en tiempo de ejecución.

## <a name="net-framework-types-allowed-in-policy-expressions"></a><a name="CLRTypes"></a>Tipos de .NET framework que se permiten en expresiones de directiva
En la tabla siguiente se enumeran los tipos de .NET Framework que se permiten en las expresiones de directiva y sus miembros.

|Tipo|Miembros compatibles|
|--------------|-----------------------|
|Newtonsoft.Json.Formatting|All|
|Newtonsoft.Json.JsonConvert|SerializeObject, DeserializeObject|
|Newtonsoft.Json.Linq.Extensions|All|
|Newtonsoft.Json.Linq.JArray|All|
|Newtonsoft.Json.Linq.JConstructor|All|
|Newtonsoft.Json.Linq.JContainer|All|
|Newtonsoft.Json.Linq.JObject|All|
|Newtonsoft.Json.Linq.JProperty|All|
|Newtonsoft.Json.Linq.JRaw|All|
|Newtonsoft.Json.Linq.JToken|All|
|Newtonsoft.Json.Linq.JTokenType|All|
|Newtonsoft.Json.Linq.JValue|All|
|System.Array|All|
|System.BitConverter|All|
|System.Boolean|All|
|System.Byte|All|
|System.Char|All|
|System.Collections.Generic.Dictionary<TKey, TValue>|All|
|System.Collections.Generic.HashSet\<T>|All|
|System.Collections.Generic.ICollection\<T>|All|
|System.Collections.Generic.IDictionary<TKey, TValue>|All|
|System.Collections.Generic.IEnumerable\<T>|All|
|System.Collections.Generic.IEnumerator\<T>|All|
|System.Collections.Generic.IList\<T>|All|
|System.Collections.Generic.IReadOnlyCollection\<T>|All|
|System.Collections.Generic.IReadOnlyDictionary<TKey, TValue>|All|
|System.Collections.Generic.ISet\<T>|All|
|System.Collections.Generic.KeyValuePair<TKey, TValue>|All|
|System.Collections.Generic.List\<T>|All|
|System.Collections.Generic.Queue\<T>|All|
|System.Collections.Generic.Stack\<T>|All|
|System.Convert|All|
|System.DateTime|(Constructor), Add, AddDays, AddHours, AddMilliseconds, AddMinutes, AddMonths, AddSeconds, AddTicks, AddYears, Date, Day, DayOfWeek, DayOfYear, DaysInMonth, Hour, IsDaylightSavingTime, IsLeapYear, MaxValue, Millisecond, Minute, MinValue, Month, Now, Parse, Second, Subtract, Ticks, TimeOfDay, Today, ToString, UtcNow, Year|
|System.DateTimeKind|UTC|
|System.DateTimeOffset|All|
|System.Decimal|All|
|System.Double|All|
|System.Exception|All|
|System.Guid|All|
|System.Int16|All|
|System.Int32|All|
|System.Int64|All|
|System.IO.StringReader|All|
|System.IO.StringWriter|All|
|System.Linq.Enumerable|All|
|System.Math|All|
|System.MidpointRounding|All|
|System.Net.WebUtility|All|
|System.Nullable|All|
|System.Random|All|
|System.SByte|All|
|System.Security.Cryptography.AsymmetricAlgorithm|All|
|System.Security.Cryptography.CipherMode|All|
|System.Security.Cryptography.HashAlgorithm|All|
|System.Security.Cryptography.HashAlgorithmName|All|
|System.Security.Cryptography.HMAC|All|
|System.Security.Cryptography.HMACMD5|All|
|System.Security.Cryptography.HMACSHA1|All|
|System.Security.Cryptography.HMACSHA256|All|
|System.Security.Cryptography.HMACSHA384|All|
|System.Security.Cryptography.HMACSHA512|All|
|System.Security.Cryptography.KeyedHashAlgorithm|All|
|System.Security.Cryptography.MD5|All|
|System.Security.Cryptography.Oid|All|
|System.Security.Cryptography.PaddingMode|All|
|System.Security.Cryptography.RNGCryptoServiceProvider|All|
|System.Security.Cryptography.RSA|All|
|System.Security.Cryptography.RSAEncryptionPadding|All|
|System.Security.Cryptography.RSASignaturePadding|All|
|System.Security.Cryptography.SHA1|All|
|System.Security.Cryptography.SHA1Managed|All|
|System.Security.Cryptography.SHA256|All|
|System.Security.Cryptography.SHA256Managed|All|
|System.Security.Cryptography.SHA384|All|
|System.Security.Cryptography.SHA384Managed|All|
|System.Security.Cryptography.SHA512|All|
|System.Security.Cryptography.SHA512Managed|All|
|System.Security.Cryptography.SymmetricAlgorithm|All|
|System.Security.Cryptography.X509Certificates.PublicKey|All|
|System.Security.Cryptography.X509Certificates.RSACertificateExtensions|All|
|System.Security.Cryptography.X509Certificates.X500DistinguishedName|Nombre|
|System.Security.Cryptography.X509Certificates.X509Certificate|All|
|System.Security.Cryptography.X509Certificates.X509Certificate2|All|
|System.Security.Cryptography.X509Certificates.X509ContentType|All|
|System.Security.Cryptography.X509Certificates.X509NameType|All|
|System.Single|All|
|System.String|All|
|System.StringComparer|All|
|System.StringComparison|All|
|System.StringSplitOptions|All|
|System.Text.Encoding|All|
|System.Text.RegularExpressions.Capture|Index, Length, Value.|
|System.Text.RegularExpressions.CaptureCollection|Count, Item.|
|System.Text.RegularExpressions.Group|Captures, Success.|
|System.Text.RegularExpressions.GroupCollection|Count, Item.|
|System.Text.RegularExpressions.Match|Empty, Groups, Result.|
|System.Text.RegularExpressions.Regex|(Constructor), IsMatch, Match, Matches, Replace, Unescape, Split|
|System.Text.RegularExpressions.RegexOptions|All|
|System.Text.StringBuilder|All|
|System.TimeSpan|All|
|System.TimeZone|All|
|System.TimeZoneInfo.AdjustmentRule|All|
|System.TimeZoneInfo.TransitionTime|All|
|System.TimeZoneInfo|All|
|System.Tuple|All|
|System.UInt16|All|
|System.UInt32|All|
|System.UInt64|All|
|System.Uri|All|
|System.UriPartial|All|
|System.Xml.Linq.Extensions|All|
|System.Xml.Linq.XAttribute|All|
|System.Xml.Linq.XCData|All|
|System.Xml.Linq.XComment|All|
|System.Xml.Linq.XContainer|All|
|System.Xml.Linq.XDeclaration|All|
|System.Xml.Linq.XDocument|Todos, excepto: Cargar|
|System.Xml.Linq.XDocumentType|All|
|System.Xml.Linq.XElement|All|
|System.Xml.Linq.XName|All|
|System.Xml.Linq.XNamespace|All|
|System.Xml.Linq.XNode|All|
|System.Xml.Linq.XNodeDocumentOrderComparer|All|
|System.Xml.Linq.XNodeEqualityComparer|All|
|System.Xml.Linq.XObject|All|
|System.Xml.Linq.XProcessingInstruction|All|
|System.Xml.Linq.XText|All|
|System.Xml.XmlNodeType|All|

## <a name="context-variable"></a><a name="ContextVariables"></a> Variable de contexto
Una variable denominada `context` está disponible implícitamente en todas las [expresiones](api-management-policy-expressions.md#Syntax) de directiva. Sus miembros proporcionan información relativa a `\request`. Todos los miembros de `context` son de solo lectura.

|Variable de contexto|Métodos, propiedades y valores de parámetro admitidos|
|----------------------|-------------------------------------------------------|
|context|[API](#ref-context-api): [IApi](#ref-iapi)<br /><br /> [Implementación](#ref-context-deployment)<br /><br /> Transcurrido: TimeSpan: intervalo de tiempo entre el valor de marca de tiempo y la hora actual<br /><br /> [LastError](#ref-context-lasterror)<br /><br /> [operación](#ref-context-operation)<br /><br /> [Producto](#ref-context-product)<br /><br /> [Solicitud](#ref-context-request)<br /><br /> RequestId: Guid: identificador único de la solicitud<br /><br /> [Respuesta](#ref-context-response)<br /><br /> [Suscripción](#ref-context-subscription)<br /><br /> Marca de tiempo: DateTime: punto en el tiempo en que se recibió la solicitud<br /><br /> Tracing: bool - indica si el seguimiento está activado o desactivado <br /><br /> [User](#ref-context-user)<br /><br /> [Variables](#ref-context-variables): IReadOnlyDictionary<cadena, objeto><br /><br /> void Trace(message: cadena)|
|<a id="ref-context-api"></a>context.Api|Id: cadena<br /><br /> IsCurrentRevision: bool<br /><br />  Name: cadena<br /><br /> Path: cadena<br /><br /> Revision: cadena<br /><br /> ServiceUrl: [IUrl](#ref-iurl)<br /><br /> Version: cadena |
|<a id="ref-context-deployment"></a>context.Deployment|Region: cadena<br /><br /> ServiceName: cadena<br /><br /> Certificates: IReadOnlyDictionary<cadena, X509Certificate2>|
|<a id="ref-context-lasterror"></a>context.LastError|Source: cadena<br /><br /> Reason: cadena<br /><br /> Message: cadena<br /><br /> Scope: cadena<br /><br /> Section: cadena<br /><br /> Path: cadena<br /><br /> PolicyId: cadena<br /><br /> Para obtener más información sobre context.LastError, consulte [Error handling in API Management policies](api-management-error-handling-policies.md) (Control de errores en directivas de API Management).|
|<a id="ref-context-operation"></a>context.Operation|Id: cadena<br /><br /> Method: cadena<br /><br /> Name: cadena<br /><br /> UrlTemplate: cadena|
|<a id="ref-context-product"></a>context.Product|Apis: IEnumerable<[IApi](#ref-iapi)\><br /><br /> ApprovalRequired: bool<br /><br /> Groups: IEnumerable<[IGroup](#ref-igroup)\><br /><br /> Id: cadena<br /><br /> Name: cadena<br /><br /> State: enum ProductState {NotPublished, Published}<br /><br /> SubscriptionLimit: int?<br /><br /> SubscriptionRequired: bool|
|<a id="ref-context-request"></a>context.Request|Cuerpo: [IMessageBody](#ref-imessagebody) o `null` si la solicitud no tiene un cuerpo.<br /><br /> Certificate: System.Security.Cryptography.X509Certificates.X509Certificate2<br /><br /> [Encabezados](#ref-context-request-headers): IReadOnlyDictionary<cadena, cadena[]><br /><br /> IpAddress: cadena<br /><br /> MatchedParameters: IReadOnlyDictionary<cadena, cadena><br /><br /> Method: cadena<br /><br /> OriginalUrl: [IUrl](#ref-iurl)<br /><br /> Url: [IUrl](#ref-iurl)|
|<a id="ref-context-request-headers"></a>string context.Request.Headers.GetValueOrDefault(headerName: string, defaultValue: string)|headerName: cadena<br /><br /> defaultValue: cadena<br /><br /> Devuelve valores de encabezado de solicitud separados por comas o `defaultValue`, si no se encuentra el encabezado.|
|<a id="ref-context-response"></a>context.Response|Cuerpo: [IMessageBody](#ref-imessagebody)<br /><br /> [Encabezados](#ref-context-response-headers): IReadOnlyDictionary<cadena, cadena[]><br /><br /> StatusCode: int<br /><br /> StatusReason: cadena|
|<a id="ref-context-response-headers"></a>string context.Response.Headers.GetValueOrDefault(headerName: string, defaultValue: string)|headerName: cadena<br /><br /> defaultValue: cadena<br /><br /> Devuelve valores de encabezado de respuesta separados por comas o `defaultValue`, si no se encuentra el encabezado.|
|<a id="ref-context-subscription"></a>context.Subscription|CreatedTime: DateTime<br /><br /> EndDate: DateTime?<br /><br /> Id: cadena<br /><br /> Key: cadena<br /><br /> Name: cadena<br /><br /> PrimaryKey: cadena<br /><br /> SecondaryKey: cadena<br /><br /> StartDate: DateTime?|
|<a id="ref-context-user"></a>context.User|Email: cadena<br /><br /> FirstName: cadena<br /><br /> Groups: IEnumerable<[IGroup](#ref-igroup)\><br /><br /> Id: cadena<br /><br /> Identities: IEnumerable<[IUserIdentity](#ref-iuseridentity)\><br /><br /> LastName: cadena<br /><br /> Note: cadena<br /><br /> RegistrationDate: DateTime|
|<a id="ref-iapi"></a>IApi|Id: cadena<br /><br /> Name: cadena<br /><br /> Path: cadena<br /><br /> Protocols: IEnumerable<cadena\><br /><br /> ServiceUrl: [IUrl](#ref-iurl)<br /><br /> SubscriptionKeyParameterNames: [ISubscriptionKeyParameterNames](#ref-isubscriptionkeyparameternames)|
|<a id="ref-igroup"></a>IGroup|Id: cadena<br /><br /> Name: cadena|
|<a id="ref-imessagebody"></a>IMessageBody|As<T\>(preserveContent: bool = false): Donde T: string, byte[],JObject, JToken, JArray, XNode, XElement, XDocument<br /><br /> Los métodos `context.Request.Body.As<T>` y `context.Response.Body.As<T>` se usan para leer los cuerpos de un mensaje de respuesta y solicitud en un tipo `T` especificado. De forma predeterminada, el método usa la secuencia de cuerpo del mensaje original y lo presenta como no disponible tras la devolución. Para evitar este resultado haciendo que el método procese una copia de la secuencia del cuerpo, establezca el parámetro `preserveContent` en `true`. Vaya [aquí](api-management-transformation-policies.md#SetBody) para ver un ejemplo.|
|<a id="ref-iurl"></a>IUrl|Host: cadena<br /><br /> Path: cadena<br /><br /> Port: int<br /><br /> [Consultar](#ref-iurl-query) IReadOnlyDictionary<cadena, cadena[]><br /><br /> QueryString: cadena<br /><br /> Scheme: cadena|
|<a id="ref-iuseridentity"></a>IUserIdentity|Id: cadena<br /><br /> Provider: cadena|
|<a id="ref-isubscriptionkeyparameternames"></a>ISubscriptionKeyParameterNames|Header: cadena<br /><br /> Query: cadena|
|<a id="ref-iurl-query"></a>string IUrl.Query.GetValueOrDefault(queryParameterName: string, defaultValue: string)|queryParameterName: cadena<br /><br /> defaultValue: cadena<br /><br /> Devuelve valores de parámetro de consulta separados por comas o `defaultValue`, si no se encuentra el parámetro.|
|<a id="ref-context-variables"></a>T context.Variables.GetValueOrDefault<T\>(variableName: string, defaultValue: T)|variableName: cadena<br /><br /> defaultValue: T<br /><br /> Devuelve el valor de la variable convertido al tipo `T` o `defaultValue` si no se encuentra la variable.<br /><br /> Este método produce una excepción si el tipo especificado no coincide con el tipo real de la variable devuelta.|
|BasicAuthCredentials AsBasic(input: esta cadena)|input: cadena<br /><br /> Si el parámetro de entrada contiene un valor válido de encabezado de solicitud de autorización de autenticación básica HTTP, el método devuelve un objeto de tipo `BasicAuthCredentials`; en caso contrario, el método devuelve nulo.|
|bool TryParseBasic(input: esta cadena, result: BasicAuthCredentials de salida)|input: cadena<br /><br /> resultado: out BasicAuthCredentials<br /><br /> Si el parámetro de entrada contiene un valor válido de autorización de autenticación básica HTTP en el encabezado de solicitud, el método devuelve `true` y el parámetro de resultado contiene un valor del tipo `BasicAuthCredentials`; en caso contrario, el método devuelve `false`.|
|BasicAuthCredentials|Password: cadena<br /><br /> UserId: cadena|
|Jwt AsJwt(input: esta cadena)|input: cadena<br /><br /> Si el parámetro de entrada contiene un valor de token JWT válido, el método devuelve un objeto de tipo `Jwt`; en caso contrario, el método devuelve `null`.|
|bool TryParseJwt(input: esta cadena, result: Jwt de salida)|input: cadena<br /><br /> resultado: JWT de salida<br /><br /> Si el parámetro de entrada contiene un valor de token JWT válido, el método devuelve `true` y el parámetro de resultado contiene un valor de tipo `Jwt`; en caso contrario, el método devuelve `false`.|
|Jwt|Algorithm: cadena<br /><br /> Audiences: IEnumerable<cadena\><br /><br /> Claims: IReadOnlyDictionary<cadena, cadena[]><br /><br /> ExpirationTime: DateTime?<br /><br /> Id: cadena<br /><br /> Issuer: cadena<br /><br /> IssuedAt: DateTime?<br /><br /> NotBefore: DateTime?<br /><br /> Subject: cadena<br /><br /> Type: cadena|
|cadena Jwt.Claims.GetValueOrDefault(claimName: cadena, defaultValue: cadena)|claimName: cadena<br /><br /> defaultValue: cadena<br /><br /> Devuelve valores de notificación separados por comas o `defaultValue`, si no se encuentra el encabezado.|
|byte[] Encrypt(input: this byte[], alg: string, key:byte[], iv:byte[])|input: texto no cifrado que se va a cifrar<br /><br />alg: nombre de un algoritmo de cifrado simétrico<br /><br />key: clave de cifrado<br /><br />iv: vector de inicialización<br /><br />Devuelve cifrado el texto no cifrado.|
|byte[] Encrypt(entrada: este byte[], alg: System.Security.Cryptography.SymmetricAlgorithm)|input: texto no cifrado que se va a cifrar<br /><br />alg: algoritmo de cifrado<br /><br />Devuelve cifrado el texto no cifrado.|
|byte[] Encrypt(entrada: este byte[], alg: System.Security.Cryptography.SymmetricAlgorithm, key:byte[], iv:byte[])|input: texto no cifrado que se va a cifrar<br /><br />alg: algoritmo de cifrado<br /><br />key: clave de cifrado<br /><br />iv: vector de inicialización<br /><br />Devuelve cifrado el texto no cifrado.|
|byte[] Decrypt(input: this byte[], alg: string, key:byte[], iv:byte[])|input: texto cifrado que se va a descifrar<br /><br />alg: nombre de un algoritmo de cifrado simétrico<br /><br />key: clave de cifrado<br /><br />iv: vector de inicialización<br /><br />Devuelve texto no cifrado.|
|byte[] Decrypt(entrada: este byte[], alg: System.Security.Cryptography.SymmetricAlgorithm)|input: texto cifrado que se va a descifrar<br /><br />alg: algoritmo de cifrado<br /><br />Devuelve texto no cifrado.|
|byte[] Decrypt(entrada: este byte[], alg: System.Security.Cryptography.SymmetricAlgorithm, key:byte[], iv:byte[])|input: texto cifrado que se va a descifrar<br /><br />alg: algoritmo de cifrado<br /><br />key: clave de cifrado<br /><br />iv: vector de inicialización<br /><br />Devuelve texto no cifrado.|
|bool VerifyNoRevocation(input: this System.Security.Cryptography.X509Certificates.X509Certificate2)|Realiza una validación de la cadena X.509 sin comprobar el estado de revocación de los certificados.<br /><br />input - certificate object<br /><br />Devuelve `true` si la validación es correcta; `false` si hay un error.|


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo trabajar con directivas, consulte:

+ [Directivas de Azure API Management](api-management-howto-policies.md)
+ [API de transformación](transform-api.md)
+ En la [Referencia de directivas](./api-management-policies.md) se muestra una lista completa de declaraciones de directivas y su configuración
+ [Ejemplos de directivas](policy-samples.md)
