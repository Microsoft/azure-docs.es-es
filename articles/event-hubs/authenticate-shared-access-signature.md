---
title: Autenticación del acceso a Azure Event Hubs con firmas de acceso compartido
description: En este artículo se muestra cómo autenticar el acceso a recursos de Event Hubs mediante firmas de acceso compartido.
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 5397cd746f33fc82ee63cc7d2a1f396ba704f334
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423106"
---
# <a name="authenticate-access-to-event-hubs-resources-using-shared-access-signatures-sas"></a>Autenticación del acceso a recursos de Event Hubs mediante firmas de acceso compartido (SAS)
La firma de acceso compartido (SAS) le ofrece un control pormenorizado sobre el tipo de acceso que se concede a los clientes que la tienen. Estos son algunos de los controles que puede establecer en una SAS: 

- El intervalo durante el que la SAS es válida, incluida la hora de inicio y la hora de expiración.
- Los permisos concedidos por la SAS. Por ejemplo, una SAS para un espacio de nombres de Event Hubs podría conceder el permiso de escucha, pero no el permiso de envío.
- Solo los clientes que tengan credenciales válidas pueden enviar datos a un centro de eventos.
- Un cliente no puede suplantar a otro cliente.
- Se puede bloquear el envío de datos por parte de un cliente no autorizado a un centro de eventos.

En este artículo se explica cómo autenticar el acceso a los recursos de Event Hubs mediante SAS. Para más información sobre la **autorización** del acceso a recursos de Event Hubs mediante SAS, consulte [este artículo](authorize-access-shared-access-signature.md). 

> [!NOTE]
> Microsoft recomienda usar credenciales de Azure AD cuando sea posible como procedimiento recomendado de seguridad, en lugar de usar las firmas de acceso compartido, que se pueden exponer más fácilmente a riesgos. Aunque puede seguir usando firmas de acceso compartido (SAS) para conceder acceso específico a los recursos en su instancia de Event Hubs, Azure AD ofrece funcionalidades similares sin necesidad de administrar tokens de SAS ni preocuparse sobre cómo revocar una SAS en peligro.
> 
> Para más información sobre la integración de Azure AD en Azure Event Hubs, consulte [Autorización del acceso a Event Hubs con Azure AD](authorize-access-azure-active-directory.md). 


## <a name="configuring-for-sas-authentication"></a>Configuración de la autenticación de SAS
Puede configurar la regla de autorización de acceso compartido de EventHubs en un espacio de nombres de Event Hubs o una entidad (instancia de centro de eventos o tema de Kafka en un centro de eventos). Actualmente no se admite la configuración de una regla de autorización de acceso compartido en un grupo de consumidores, pero puede usar reglas configuradas en un espacio de nombres o una entidad para proteger el acceso a dicho grupo. 

En la imagen siguiente se muestra cómo se aplican las reglas de autorización en las entidades de ejemplo. 

![Configuración de la regla de autorización](./media/authenticate-shared-access-signature/configure-sas-authorization-rule.png)

En este ejemplo, el espacio de nombres de Event Hubs (ExampleNamespace) tiene dos entidades: eh1 y topic1. Las reglas de autorización se definen en el nivel de entidad y también en el nivel de espacio de nombres.  

Las reglas de autorización manageRuleNS, sendRuleNS y listenRuleNS se aplican al centro de eventos eh1 y al tema t1. Las reglas de autorización listenRule-eh y sendRule-eh se aplican únicamente a la instancia del centro de eventos eh1, y la regla de autorización sendRuleT se aplica solo al tema topic1. 

Al usar una regla de autorización sendRuleNS, las aplicaciones cliente pueden enviar a eh1 y topic1. Cuando se usa la regla de autorización sendRuleT, solo se aplica el acceso granular a topic1 y, por lo tanto, las aplicaciones cliente que usan esta regla para el acceso ahora no pueden enviar a eh1, sino solo a topic1.

## <a name="generate-a-shared-access-signature-token"></a>Generación de un token de Firmas de acceso compartido 
Cualquier cliente que tenga acceso al nombre de una regla de autorización y una de sus claves de firma puede generar un token de SAS. El token se genera diseñando una cadena con el siguiente formato:

- `se`: instante de expiración del token. Entero que refleja los segundos transcurridos desde la época 00:00:00 UTC el 1 de enero de 1970 (época de UNIX) cuando expira el token.
- `skn`: nombre de la regla de autorización, que es el nombre de la clave SAS.
- `sr`: URI del recurso al que se accede.
- `sig`: firma.

La cadena de firma es el hash SHA-256 calculado sobre el URI del recurso (ámbito tal y como se describe en la sección anterior) y la representación de cadenas del instante de expiración del token, separados por CRLF.

El cálculo del hash es similar al siguiente pseudocódigo y devuelve un valor de hash de 256 bits/32 bytes. 

```
SHA-256('https://<yournamespace>.servicebus.windows.net/'+'\n'+ 1438205742)
```

El token contiene los valores sin el hash para que el destinatario pueda volver a calcular el hash con los mismos parámetros y verificar que el emisor posee una clave de firma válida.

El URI de recurso es el URI completo del recurso de Service Bus al que se solicita el acceso. Por ejemplo, http://<namespace>.servicebus.windows.net/<entityPath> o `sb://<namespace>.servicebus.windows.net/<entityPath>;`, es decir, `http://contoso.servicebus.windows.net/eventhubs/eh1`.

El URI debe estar codificado por porcentaje.

La regla de autorización de acceso compartido usada para firmar debe configurarse en la entidad especificada por este URI, o uno de sus primarios jerárquicos. Por ejemplo, `http://contoso.servicebus.windows.net/eventhubs/eh1` o `http://contoso.servicebus.windows.net` en el ejemplo anterior.

Un token SAS es válido en todos los recursos con el prefijo <resourceURI> usado en la cadena de firma.

> [!NOTE]
> El token de acceso para Event Hubs se genera mediante la directiva de acceso compartido. Para más información, consulte [Directiva de autorización de acceso compartido](authorize-access-shared-access-signature.md#shared-access-authorization-policies).

### <a name="generating-a-signaturetoken-from-a-policy"></a>Generación de una firma (token) a partir de una directiva 
En la sección siguiente se muestra cómo generar un token de SAS mediante directivas de firma de acceso compartido.

#### <a name="nodejs"></a>NodeJS

```javascript
function createSharedAccessToken(uri, saName, saKey) { 
    if (!uri || !saName || !saKey) { 
            throw "Missing required parameter"; 
        } 
    var encoded = encodeURIComponent(uri); 
    var now = new Date(); 
    var week = 60*60*24*7;
    var ttl = Math.round(now.getTime() / 1000) + week;
    var signature = encoded + '\n' + ttl; 
    var signatureUTF8 = utf8.encode(signature); 
    var hash = crypto.createHmac('sha256', saKey).update(signatureUTF8).digest('base64'); 
    return 'SharedAccessSignature sr=' + encoded + '&sig=' +  
        encodeURIComponent(hash) + '&se=' + ttl + '&skn=' + saName; 
```

#### <a name="java"></a>Java

```java
private static String GetSASToken(String resourceUri, String keyName, String key)
  {
      long epoch = System.currentTimeMillis()/1000L;
      int week = 60*60*24*7;
      String expiry = Long.toString(epoch + week);

      String sasToken = null;
      try {
          String stringToSign = URLEncoder.encode(resourceUri, "UTF-8") + "\n" + expiry;
          String signature = getHMAC256(key, stringToSign);
          sasToken = "SharedAccessSignature sr=" + URLEncoder.encode(resourceUri, "UTF-8") +"&sig=" +
                  URLEncoder.encode(signature, "UTF-8") + "&se=" + expiry + "&skn=" + keyName;
      } catch (UnsupportedEncodingException e) {

          e.printStackTrace();
      }

      return sasToken;
  }


public static String getHMAC256(String key, String input) {
    Mac sha256_HMAC = null;
    String hash = null;
    try {
        sha256_HMAC = Mac.getInstance("HmacSHA256");
        SecretKeySpec secret_key = new SecretKeySpec(key.getBytes(), "HmacSHA256");
        sha256_HMAC.init(secret_key);
        Encoder encoder = Base64.getEncoder();

        hash = new String(encoder.encode(sha256_HMAC.doFinal(input.getBytes("UTF-8"))));

    } catch (InvalidKeyException e) {
        e.printStackTrace();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
   } catch (IllegalStateException e) {
        e.printStackTrace();
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    }

    return hash;
}
```
#### <a name="php"></a>PHP

```php
function generateSasToken($uri, $sasKeyName, $sasKeyValue) 
{ 
    $targetUri = strtolower(rawurlencode(strtolower($uri))); 
    $expires = time();  
    $expiresInMins = 60; 
    $week = 60*60*24*7;
    $expires = $expires + $week; 
    $toSign = $targetUri . "\n" . $expires; 
    $signature = rawurlencode(base64_encode(hash_hmac('sha256',             
     $toSign, $sasKeyValue, TRUE))); 
    
    $token = "SharedAccessSignature sr=" . $targetUri . "&sig=" . $signature . "&se=" . $expires .      "&skn=" . $sasKeyName; 
    return $token; 
}
```

#### <a name="c"></a>C#

```csharp
private static string createToken(string resourceUri, string keyName, string key)
{
    TimeSpan sinceEpoch = DateTime.UtcNow - new DateTime(1970, 1, 1);
    var week = 60 * 60 * 24 * 7;
    var expiry = Convert.ToString((int)sinceEpoch.TotalSeconds + week);
    string stringToSign = HttpUtility.UrlEncode(resourceUri) + "\n" + expiry;
    HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key));
    var signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));
    var sasToken = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}&skn={3}", HttpUtility.UrlEncode(resourceUri), HttpUtility.UrlEncode(signature), expiry, keyName);
    return sasToken;
}
```

## <a name="authenticating-event-hubs-publishers-with-sas"></a>Autenticación de los publicadores de Event Hubs con SAS 
Un publicador de eventos define un punto de conexión virtual para un centro de eventos. El publicador solo puede usarse para enviar mensajes a un centro de eventos, no para recibirlos.

Normalmente, un centro de eventos emplea a un publicador por cliente. Todos los mensajes que se envíen a cualquiera de los publicadores de un centro de eventos se ponen en cola dentro de ese centro de eventos. Los publicadores permiten control de acceso pormenorizado.

A cada cliente de Event Hubs se le asigna un token único, que se carga en el cliente. Los tokens se generan de forma que cada token único concede acceso a un publicador único diferente. Un cliente que posea un token solo puede enviar a un publicador y a ningún otro. Si varios clientes comparten el mismo token, cada uno de estos clientes comparte el publicador.

Todos los tokens se asignan con claves SAS. Normalmente, todos los tokens se firman con la misma clave. Los clientes no conocen la clave, lo que evita que los clientes produzcan tokens. Los clientes operan en los mismos tokens hasta que expiran.

Por ejemplo, para definir reglas de autorización cuyo ámbito sea solo enviar o publicar en Event Hubs, debe definir una regla de autorización de envío. Esto se puede hacer en un nivel de espacio de nombres o proporcionar un ámbito más específico a una entidad determinada (una instancia de Event Hubs o un tema). A un cliente o una aplicación que estén limitados con este acceso específico se les conoce como publicador de Event Hubs. Para hacerlo, siga estos pasos:

1. Cree una clave SAS en la entidad que quiera publicar para asignarle el ámbito de **envío**. Para más información, consulte [Directivas de autorización de acceso compartido](authorize-access-shared-access-signature.md#shared-access-authorization-policies).
2. Genere un token de SAS con una hora de expiración para un publicador específico mediante la clave generada en paso 1.

    ```csharp
    var sasToken = SharedAccessSignatureTokenProvider.GetPublisherSharedAccessSignature(
                new Uri("Service-Bus-URI"),
                "eventub-name",
                "publisher-name",
                "sas-key-name",
                "sas-key",
                TimeSpan.FromMinutes(30));
    ```
3. Proporcione el token al cliente del publicador, que solo puede enviar a la entidad y al publicador a los que el token concede acceso.

    Una vez que el token expira, el cliente pierde su acceso para enviar o publicar en la entidad. 


> [!NOTE]
> Aunque no se recomienda, es posible equipar a los dispositivos con tokens que concedan acceso directo a un centro de eventos o espacio de nombres. Cualquier dispositivo que contenga un token de ese tipo puede enviar mensajes directamente a ese centro de eventos. Además, el dispositivo no puede estar en la lista negra que impide el envío a ese centro de eventos.
> 
> Siempre se recomienda proporcionar ámbitos específicos y pormenorizados.

> [!IMPORTANT]
> Cuando se han creado los tokens, cada cliente se aprovisiona con su propio token único.
>
> Cuando el cliente envía datos a un centro de eventos, la solicitud se etiqueta con el token. Para evitar que un atacante use la técnica de eavesdropping y robe el token, la comunicación entre el cliente y el centro de eventos debe realizarse a través de un canal cifrado.
> 
> Si un atacante roba un token, el atacante puede suplantar el cliente al que se ha robado el token. Al incorporar un publicador a la lista de no permitidos, ese cliente se representa como inutilizable hasta que recibe un token nuevo que usa un publicador diferente.


## <a name="authenticating-event-hubs-consumers-with-sas"></a>Autenticación de consumidores de Event Hubs con SAS 
Para autenticar las aplicaciones back-end que consumen los datos generados por los productores de Event Hubs, la autenticación de tokens de Event Hubs requiere que sus clientes tengan asignados los derechos de **administración** o los privilegios de **escucha** a su espacio de nombres de Event Hubs o instancia o tema del centro de eventos. Los datos se consumen de Event Hubs mediante grupos de consumidores. Aunque la directiva SAS proporciona un ámbito granular, este ámbito solo se define en el nivel de entidad, no en el nivel de consumidor. Significa que los privilegios definidos en el nivel de espacio de nombres o en el nivel de instancia o tema de centro de eventos se aplicarán a los grupos de consumidores de esa entidad.

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:

- [Autorización del uso de SAS](authenticate-shared-access-signature.md)
- [Autorización mediante el control de acceso basado en rol (RBAC)](authenticate-shared-access-signature.md)
- [Más información sobre Event Hubs](event-hubs-about.md)

Consulte los artículos relacionados siguientes:

- [Autenticación de solicitudes a Azure Event Hubs desde una aplicación mediante Azure Active Directory](authenticate-application.md)
- [Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Event Hubs](authenticate-managed-identity.md)
- [Autorización del acceso a recursos de Event Hubs mediante Azure Active Directory](authorize-access-azure-active-directory.md)
- [Autorización del acceso a recursos de Event Hubs mediante firmas de acceso compartido](authorize-access-shared-access-signature.md)
