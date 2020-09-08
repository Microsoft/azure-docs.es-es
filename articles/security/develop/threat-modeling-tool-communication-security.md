---
title: Seguridad de la comunicación para Microsoft Threat Modeling Tool
titleSuffix: Azure
description: Más información sobre la mitigación de las amenazas a la seguridad de las comunicaciones expuestas en Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: devx-track-csharp
ms.openlocfilehash: 3d795d30e3ad420e0fed002baddf37469ddcf995
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004569"
---
# <a name="security-frame-communication-security--mitigations"></a>Marco de seguridad: seguridad en las comunicaciones | Mitigaciones 
| Producto o servicio | Artículo |
| --------------- | ------- |
| **Centro de eventos de Azure** | <ul><li>[Protección de las comunicaciones con el centro de eventos mediante SSL/TLS](#comm-ssltls)</li></ul> |
| **Dynamics CRM** | <ul><li>[Comprobación de los privilegios de cuenta de servicio y de que las páginas de ASP.NET o los servicios personalizados respetan la seguridad de CRM](#priv-aspnet)</li></ul> |
| **Azure Data Factory** | <ul><li>[Uso de Data Management Gateway durante la conexión del servidor SQL Server local a Azure Data Factory](#sqlserver-factory)</li></ul> |
| **Identity Server** | <ul><li>[Comprobación de que todo el tráfico a Identity Server se transmite a través de una conexión HTTPS](#identity-https)</li></ul> |
| **Aplicación web** | <ul><li>[Comprobación de certificados X.509 utilizados para autenticar las conexiones SSL, TLS y DTLS](#x509-ssltls)</li><li>[Configuración de un certificado TLS/SSL para un dominio personalizado en Azure App Service](#ssl-appservice)</li><li>[Direccionamiento forzoso de todo el tráfico a Azure App Service a través de una conexión HTTPS](#appservice-https)</li><li>[Habilitación de seguridad de transporte estricto HTTP (HSTS)](#http-hsts)</li></ul> |
| **Base de datos** | <ul><li>[Comprobación de cifrado de la conexión de SQL Server y validación de certificados](#sqlserver-validation)</li><li>[Aplicación forzosa de comunicación cifrada a SQL Server](#encrypted-sqlserver)</li></ul> |
| **Almacenamiento de Azure** | <ul><li>[Comprobación de que la comunicación a Azure Storage se realiza a través de HTTPS](#comm-storage)</li><li>[Validación de hash MD5 después de descargar blob si no se puede habilitar HTTPS](#md5-https)</li><li>[Uso de un cliente compatible con SMB 3.0 para garantizar el cifrado de datos en tránsito en recursos compartidos de archivos de Azure](#smb-shares)</li></ul> |
| **Cliente para dispositivos móviles** | <ul><li>[Implementación de asignación de certificados](#cert-pinning)</li></ul> |
| **WCF** | <ul><li>[Habilitación de HTTPS: canal de transporte seguro](#https-transport)</li><li>[WCF: establecimiento del nivel de protección de seguridad de mensajes en EncryptAndSign](#message-protection)</li><li>[WCF: uso de una cuenta con privilegios mínimos para ejecutar el servicio WCF](#least-account-wcf)</li></ul> |
| **API web** | <ul><li>[Direccionamiento forzoso de todo el tráfico a API web a través de una conexión HTTPS](#webapi-https)</li></ul> |
| **Azure Cache for Redis** | <ul><li>[Comprobación de que la comunicación a Azure Cache for Redis se realiza a través de TLS](#redis-ssl)</li></ul> |
| **Puerta de enlace de campo de IoT** | <ul><li>[Protección de la comunicación entre el dispositivo y la puerta de enlace de campo](#device-field)</li></ul> |
| **Puerta de enlace de nube de IoT** | <ul><li>[Protección de la comunicación entre el dispositivo y la puerta de enlace de nube mediante SSL/TLS](#device-cloud)</li></ul> |

## <a name="secure-communication-to-event-hub-using-ssltls"></a><a id="comm-ssltls"></a>Protección de las comunicaciones con el centro de eventos mediante SSL/TLS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Centro de eventos de Azure | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Introducción al modelo de autenticación y seguridad de Event Hubs](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **Pasos** | Proteja conexiones AMQP o HTTP al centro de eventos mediante SSL/TLS. |

## <a name="check-service-account-privileges-and-check-that-the-custom-services-or-aspnet-pages-respect-crms-security"></a><a id="priv-aspnet"></a>Comprobación de los privilegios de cuenta de servicio y de que las páginas de ASP.NET o los servicios personalizados respetan la seguridad de CRM

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dynamics CRM | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Compruebe los privilegios de cuenta de servicio y que las páginas de ASP.NET o los servicios personalizados respetan la seguridad de CRM. |

## <a name="use-data-management-gateway-while-connecting-on-premises-sql-server-to-azure-data-factory"></a><a id="sqlserver-factory"></a>Uso de Data Management Gateway durante la conexión del servidor SQL Server local a Azure Data Factory

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Data Factory | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Tipos de servicios vinculados: Azure y local |
| **Referencias**              |[Movimiento de datos entre el entorno local y Azure Data Factory](https://azure.microsoft.com/documentation/articles/data-factory-move-data-between-onprem-and-cloud/#create-gateway), [Data Management Gateway](https://azure.microsoft.com/documentation/articles/data-factory-data-management-gateway/) |
| **Pasos** | <p>La herramienta Data Management Gateway (DMG) es necesaria para conectarse a orígenes de datos que están protegidos en una red corporativa o mediante firewall.</p><ol><li>Al bloquearse la máquina, se aísla la herramienta DMG, lo que impide que los programas que funcionan incorrectamente dañen o accedan a la máquina del origen de datos (por ejemplo, deben instalarse las actualizaciones más recientes, habilitar los puertos mínimos necesarios, aprovisionar cuentas controladas, habilitar la auditoría, habilitar el cifrado de disco, etc.).</li><li>La clave de puerta de enlace de datos se debe alternar a intervalos frecuentes o siempre que se renueva la contraseña de cuenta del servicio DMG.</li><li>El tránsito de datos a través del servicio de vínculos debe cifrarse.</li></ol> |

## <a name="ensure-that-all-traffic-to-identity-server-is-over-https-connection"></a><a id="identity-https"></a>Comprobación de que todo el tráfico a Identity Server se transmite a través de una conexión HTTPS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Identity Server | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [IdentityServer3: claves, firmas y criptografía](https://identityserver.github.io/Documentation/docsv2/configuration/crypto.html), [IdentityServer3: implementación](https://identityserver.github.io/Documentation/docsv2/advanced/deployment.html) |
| **Pasos** | De forma predeterminada, IdentityServer requiere que todas las conexiones entrantes se realicen a través de HTTPS. Es absolutamente obligatorio que la comunicación con IdentityServer se realice exclusivamente a través de transportes seguros. En ciertos escenarios de implementación, como la descarga de TLS, este requisito puede ser más flexible. Consulte la página de implementación de Identity Server en las referencias para más información. |

## <a name="verify-x509-certificates-used-to-authenticate-ssl-tls-and-dtls-connections"></a><a id="x509-ssltls"></a>Comprobación de certificados X.509 utilizados para autenticar las conexiones SSL, TLS y DTLS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Las aplicaciones que utilizan SSL, TLS y DTLS deben comprobar completamente los certificados X.509 de las entidades a las que se conectan. Esto incluye la comprobación de los certificados de:</p><ul><li>Nombre de dominio</li><li>Fechas de validez (fechas de inicio y caducidad)</li><li>Estado de revocación</li><li>Uso (por ejemplo, autenticación de servidor para servidores, autenticación de cliente para clientes)</li><li>Cadena de confianza: los certificados deben estar encadenados a una entidad de certificación raíz (CA) que sea de confianza para la plataforma o que haya configurado explícitamente el administrador</li><li>Longitud de la clave pública del certificado > 2048 bits</li><li>Algoritmo hash: SHA256 y versiones posteriores |

## <a name="configure-tlsssl-certificate-for-custom-domain-in-azure-app-service"></a><a id="ssl-appservice"></a>Configuración de un certificado TLS/SSL para un dominio personalizado en Azure App Service

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | EnvironmentType: Azure |
| **Referencias**              | [Habilitación de HTTPS para una aplicación en Azure App Service](../../app-service/configure-ssl-bindings.md) |
| **Pasos** | De forma predeterminada, Azure ya habilita HTTPS para cada aplicación con un certificado comodín para el dominio *.azurewebsites.net. Sin embargo, al igual que todos los dominios comodín, no es tan seguro como usar un dominio personalizado con su propio certificado. [Más información](https://casecurity.org/2014/02/26/pros-and-cons-of-single-domain-multi-domain-and-wildcard-certificates/). Se recomienda habilitar TLS para el dominio personalizado a través del que se accederá a la aplicación implementada.|

## <a name="force-all-traffic-to-azure-app-service-over-https-connection"></a><a id="appservice-https"></a>Direccionamiento forzoso de todo el tráfico a Azure App Service a través de una conexión HTTPS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | EnvironmentType: Azure |
| **Referencias**              | [Aplicación de HTTPS en Azure App Service](../../app-service/configure-ssl-bindings.md#enforce-https) |
| **Pasos** | <p>Aunque Azure ya habilita HTTPS para Azure App Services con un certificado comodín para el dominio *.azurewebsites.net, no lo exige. Los visitantes pueden seguir accediendo a la aplicación mediante HTTP, lo que puede comprometer la seguridad de la aplicación, por lo que debe exigirse el uso de HTTPS explícitamente. Las aplicaciones de ASP.NET MVC deben utilizar el [filtro RequireHttps](https://msdn.microsoft.com/library/system.web.mvc.requirehttpsattribute.aspx) que obliga a que una solicitud HTTP no segura se vuelva a enviar a través de HTTPS.</p><p>También es posible utilizar el módulo URL Rewrite, incluido con Azure App Service, para exigir el uso de HTTPS. El módulo URL Rewrite permite a los desarrolladores definir reglas que se aplican a las solicitudes entrantes antes de que las solicitudes lleguen a su aplicación. Las reglas de URL Rewrite se definen en el archivo web.config, que se almacena en la raíz de la aplicación.</p>|

### <a name="example"></a>Ejemplo
El ejemplo siguiente contiene una regla básica de URL Rewrite que impone el uso de HTTPS a todo el tráfico entrante.
```XML
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Force HTTPS" enabled="true">
          <match url="(.*)" ignoreCase="false" />
          <conditions>
            <add input="{HTTPS}" pattern="off" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" appendQueryString="true" redirectType="Permanent" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```
Esta regla funciona devolviendo un código de estado HTTP de 301 (redirección permanente) cuando el usuario solicita una página mediante HTTP. El 301 redirige la solicitud a la misma URL que el visitante solicitó, pero sustituye la parte de HTTP de la solicitud por HTTPS. Por ejemplo, `HTTP://contoso.com` se redirigirá a `HTTPS://contoso.com`. 

## <a name="enable-http-strict-transport-security-hsts"></a><a id="http-hsts"></a>Habilitación de seguridad de transporte estricto HTTP (HSTS)

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Hoja de referencia rápida de seguridad de transporte estricto HTTP del Proyecto de seguridad de aplicación web abierta](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html) |
| **Pasos** | <p>La seguridad de transporto estricta HTTP (HSTS) es una mejora de seguridad opcional que se especifica con una aplicación web mediante el uso de un encabezado de respuesta especial. Una vez que un explorador compatible recibe este encabezado, ese explorador impedirá que se envíen comunicaciones a través de HTTP al dominio especificado y, en su lugar, enviará todas las comunicaciones a través de HTTPS. También evita que aparezcan en los exploradores elementos click-through HTTPS.</p><p>Para implementar HSTS, se debe configurar el siguiente encabezado de respuesta para un sitio web a nivel global, ya sea en el código o en la configuración. Strict-Transport-Security: max-age=300; includeSubDomains. HSTS resuelve las amenazas siguientes:</p><ul><li>Un usuario guarda en sus marcadores la dirección `https://example.com` o la introduce manualmente y sufre un ataque de tipo "Man in the middle": HSTS redirige automáticamente las solicitudes HTTP a HTTPS para el dominio de destino.</li><li>Una aplicación web diseñada para ser exclusivamente HTTPS contiene accidentalmente vínculos HTTP o sirve contenido a través de HTTP: HSTS redirige automáticamente las solicitudes HTTP a HTTPS para el dominio de destino.</li><li>Un ataque de tipo "Man in the middle" intenta interceptar el tráfico de un usuario mediante un certificado no válido y espera que este usuario acepte dicho certificado: HSTS no permite que un usuario omita el mensaje de certificado no válido.</li></ul>|

## <a name="ensure-sql-server-connection-encryption-and-certificate-validation"></a><a id="sqlserver-validation"></a>Comprobación de cifrado de la conexión de SQL Server y validación de certificados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | SQL Azure  |
| **Atributos**              | Versión de SQL: V12 |
| **Referencias**              | [Procedimientos recomendados sobre cómo escribir cadenas de conexión seguras para SQL Database](https://social.technet.microsoft.com/wiki/contents/articles/2951.windows-azure-sql-database-connection-security.aspx#best) |
| **Pasos** | <p>Todas las comunicaciones entre SQL Database y una aplicación cliente se cifran mediante la Seguridad de la capa de transporte (TLS), anteriormente conocida como Capa de sockets seguros (SSL), en todo momento. SQL Database no admite conexiones no cifradas. Para validar certificados con código de aplicación o herramientas, solicite explícitamente una conexión cifrada y no confíe en los certificados de servidor. Si el código de su aplicación o las herramientas no solicitan una conexión cifrada, seguirán recibiendo conexiones cifradas.</p><p>Sin embargo, es posible que no validen los certificados de servidor, por lo que serán susceptibles de recibir ataques de tipo "man in the middle". Para validar certificados con código de aplicación ADO.NET, establezca `Encrypt=True` y `TrustServerCertificate=False` en la cadena de conexión de la base de datos. Para validar certificados mediante SQL Server Management Studio, abra el cuadro de diálogo Conectar con el servidor. Haga clic en Cifrar conexión en la pestaña Propiedades de conexión.</p>|

## <a name="force-encrypted-communication-to-sql-server"></a><a id="encrypted-sqlserver"></a>Aplicación forzosa de comunicación cifrada a SQL Server

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | OnPrem |
| **Atributos**              | Versión de SQL: MsSQL2016, Versión de SQL: MsSQL2012, Versión de SQL: MsSQL2014 |
| **Referencias**              | [Habilitación de conexiones cifradas en el motor de base de datos](https://msdn.microsoft.com/library/ms191192)  |
| **Pasos** | Al habilitar el cifrado TLS, aumenta la seguridad de los datos transmitidos a través de redes entre instancias de SQL Server y las aplicaciones. |

## <a name="ensure-that-communication-to-azure-storage-is-over-https"></a><a id="comm-storage"></a>Comprobación de que la comunicación a Azure Storage se realiza a través de HTTPS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Azure Storage: Cifrado de nivel de transporte – Uso de HTTPS](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_encryption-in-transit) |
| **Pasos** | Para garantizar la seguridad de los datos de Azure Storage en tránsito, utilice siempre el protocolo HTTPS al llamar a las API de REST o al acceder a objetos en el almacenamiento. Además, las Firmas de acceso compartido, que pueden utilizarse para delegar el acceso a objetos de Azure Storage, incluyen una opción para especificar que se utilice solo el protocolo HTTPS con las Firmas de acceso compartido, lo que garantiza que cualquiera que envíe vínculos con tokens de SAS utilice el protocolo adecuado.|

## <a name="validate-md5-hash-after-downloading-blob-if-https-cannot-be-enabled"></a><a id="md5-https"></a>Validación de hash MD5 después de descargar blob si no se puede habilitar HTTPS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | StorageType: blob |
| **Referencias**              | [Información general de MD5 de Windows Azure Blob](https://blogs.msdn.microsoft.com/windowsazurestorage/2011/02/17/windows-azure-blob-md5-overview/) |
| **Pasos** | <p>Azure Blob service proporciona mecanismos para garantizar la integridad de datos tanto en el nivel de la aplicación como en el de transporte. Si por algún motivo debe utilizar HTTP en lugar de HTTPS y está trabajando con blobs en bloques, puede usar la comprobación de MD5 para ayudar a comprobar la integridad de los blobs que se transfieren.</p><p>Esto le ayudará con la protección frente a errores de red o de la capa de transporte, pero no necesariamente con ataques de intermediarios. Si puede usar HTTPS, que proporciona seguridad de nivel de transporte, el uso de la comprobación de MD5 es redundante e innecesario.</p>|

## <a name="use-smb-30-compatible-client-to-ensure-in-transit-data-encryption-to-azure-file-shares"></a><a id="smb-shares"></a>Uso de un cliente compatible con SMB 3.0 para garantizar el cifrado de datos en tránsito en recursos compartidos de Azure File

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Cliente móvil | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | StorageType: archivo |
| **Referencias**              | [Azure File Storage](https://azure.microsoft.com/blog/azure-file-storage-now-generally-available/#comment-2529238931), [Compatibilidad con SMB de Azure File Storage para clientes de Windows](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-files/#_mount-the-file-share) |
| **Pasos** | Azure File Storage admite HTTPS cuando se usa la API de REST, pero se usa con más frecuencia como un recurso compartido de archivos de SMB asociado a una máquina virtual. SMB 2.1 no admite el cifrado, por lo que solo se permiten las conexiones dentro de la misma región de Azure. Sin embargo, SMB 3.0 admite el cifrado, y se puede utilizar con Windows Server 2012 R2, Windows 8, Windows 8.1 y Windows 10, lo que permite el acceso entre regiones e incluso el acceso en el escritorio. |

## <a name="implement-certificate-pinning"></a><a id="cert-pinning"></a>Implementación de asignación de certificados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, Windows Phone |
| **Atributos**              | N/D  |
| **Referencias**              | [Asignación de certificados y claves públicas](https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning) |
| **Pasos** | <p>La asignación de certificados protege frente a ataques de tipo "man in the middle". La asignación es el proceso de asociar un host a su certificado X509 o clave pública esperados. Tras determinar el certificado o clave pública de un host, dicho certificado o clave pública se asocia o asigna al host. </p><p>De este modo, cuando un atacante intenta realizar un ataque de tipo "man in the middle" TLS, durante el protocolo de enlace TLS, la clave del servidor del atacante será distinta a la clave del certificado asignado, por lo que la solicitud se descartará, lo que evita que pueda realizarse una asignación de certificados mediante ataque de tipo "man in the middle" al implementar el delegado `ServerCertificateValidationCallback` de ServicePointManager.</p>|

### <a name="example"></a>Ejemplo
```csharp
using System;
using System.Net;
using System.Net.Security;
using System.Security.Cryptography;

namespace CertificatePinningExample
{
    class CertificatePinningExample
    {
        /* Note: In this example, we're hardcoding a the certificate's public key and algorithm for 
           demonstration purposes. In a real-world application, this should be stored in a secure
           configuration area that can be updated as needed. */

        private static readonly string PINNED_ALGORITHM = "RSA";

        private static readonly string PINNED_PUBLIC_KEY = "3082010A0282010100B0E75B7CBE56D31658EF79B3A1" +
            "294D506A88DFCDD603F6EF15E7F5BCBDF32291EC50B2B82BA158E905FE6A83EE044A48258B07FAC3D6356AF09B2" +
            "3EDAB15D00507B70DB08DB9A20C7D1201417B3071A346D663A241061C151B6EC5B5B4ECCCDCDBEA24F051962809" +
            "FEC499BF2D093C06E3BDA7D0BB83CDC1C2C6660B8ECB2EA30A685ADE2DC83C88314010FFC7F4F0F895EDDBE5C02" +
            "ABF78E50B708E0A0EB984A9AA536BCE61A0C31DB95425C6FEE5A564B158EE7C4F0693C439AE010EF83CA8155750" +
            "09B17537C29F86071E5DD8CA50EBD8A409494F479B07574D83EDCE6F68A8F7D40447471D05BC3F5EAD7862FA748" +
            "EA3C92A60A128344B1CEF7A0B0D94E50203010001";


        public static void Main(string[] args)
        {
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("https://azure.microsoft.com");
            request.ServerCertificateValidationCallback = (sender, certificate, chain, sslPolicyErrors) =>
            {
                if (certificate == null || sslPolicyErrors != SslPolicyErrors.None)
                {
                    // Error getting certificate or the certificate failed basic validation
                    return false;
                }

                var targetKeyAlgorithm = new Oid(certificate.GetKeyAlgorithm()).FriendlyName;
                var targetPublicKey = certificate.GetPublicKeyString();
                
                if (targetKeyAlgorithm == PINNED_ALGORITHM &&
                    targetPublicKey == PINNED_PUBLIC_KEY)
                {
                    // Success, the certificate matches the pinned value.
                    return true;
                }
                // Reject, either the key or the algorithm does not match the expected value.
                return false;
            };

            try
            {
                var response = (HttpWebResponse)request.GetResponse();
                Console.WriteLine($"Success, HTTP status code: {response.StatusCode}");
            }
            catch(Exception ex)
            {
                Console.WriteLine($"Failure, {ex.Message}");
            }
            Console.WriteLine("Press any key to end.");
            Console.ReadKey();
        }
    }
}
```

## <a name="enable-https---secure-transport-channel"></a><a id="https-transport"></a>Habilitación de HTTPS: canal de transporte seguro

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_transport_security_enabled) |
| **Pasos** | La configuración de la aplicación debe garantizar que se utilice HTTPS siempre que se acceda a información confidencial.<ul><li>**EXPLICACIÓN:** si una aplicación administra información confidencial y no usa el cifrado de nivel de mensaje, solo debe poder comunicarse a través de un canal de transporte cifrado.</li><li>**RECOMENDACIONES:** asegúrese de que el transporte HTTP está deshabilitado y habilite el transporte HTTPS en su lugar. Por ejemplo, reemplace `<httpTransport/>` por la etiqueta `<httpsTransport/>`. No utilice una configuración de red (firewall) para asegurarse de que solo se pueda acceder a la aplicación a través de un canal seguro. Desde un enfoque filosófico, la seguridad de la aplicación no debe depender de la red.</li></ul><p>Desde un punto de vista práctico, las personas responsables de proteger la red no siempre llevan al día los requisitos de seguridad de la aplicación a medida que estos evolucionan.</p>|

## <a name="wcf-set-message-security-protection-level-to-encryptandsign"></a><a id="message-protection"></a>WCF: establecimiento del nivel de protección de seguridad de mensajes en EncryptAndSign

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff650862.aspx) |
| **Pasos** | <ul><li>**EXPLICACIÓN:** cuando el nivel de protección está establecido en "none", se deshabilitará la protección de mensajes. La confidencialidad e integridad requieren definir un nivel adecuado.</li><li>**RECOMENDACIONES:**<ul><li>si `Mode=None`: deshabilita la protección del mensaje.</li><li>si `Mode=Sign`: firma el mensaje, pero no lo cifra. Debe utilizarse cuando importa la integridad de los datos.</li><li>si `Mode=EncryptAndSign`: firma y cifra el mensaje.</li></ul></li></ul><p>Considere la posibilidad de desactivar el cifrado y solo firmar el mensaje cuando únicamente necesite validar la integridad de la información sin necesidad de preocuparse por la confidencialidad. Esto puede resultar útil para operaciones o contratos de servicio en los que se debe validar el remitente original pero no se transmiten datos confidenciales. Al reducir el nivel de protección, asegúrese de que el mensaje no contenga datos personales.</p>|

### <a name="example"></a>Ejemplo
En los siguientes ejemplos, se muestra cómo configurar el servicio y la operación para que solo se firme el mensaje. Ejemplo de contrato de servicio de `ProtectionLevel.Sign`: el siguiente es un ejemplo del uso de ProtectionLevel.Sign en el nivel de contrato de servicio: 
```
[ServiceContract(Protection Level=ProtectionLevel.Sign] 
public interface IService 
  { 
  string GetData(int value); 
  } 
```

### <a name="example"></a>Ejemplo
Ejemplo de contrato de operación de `ProtectionLevel.Sign` (para un control pormenorizado): El siguiente es un ejemplo del uso de `ProtectionLevel.Sign` en el nivel OperationContract:

```
[OperationContract(ProtectionLevel=ProtectionLevel.Sign] 
string GetData(int value);
``` 

## <a name="wcf-use-a-least-privileged-account-to-run-your-wcf-service"></a><a id="least-account-wcf"></a>WCF: uso de una cuenta con privilegios mínimos para ejecutar el servicio WCF

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648826.aspx ) |
| **Pasos** | <ul><li>**EXPLICACIÓN:** no ejecute servicios WCF en una cuenta de administrador o con privilegios elevados. Esto tendría una gran incidencia en caso de que los servicios se vieran comprometidos.</li><li>**RECOMENDACIONES:** utilice una cuenta con privilegios mínimos para hospedar el servicio WCF, ya que así se reducirá la superficie expuesta a ataques de su aplicación, así como los posibles daños en caso de ataque. Si la cuenta de servicio requiere derechos de acceso adicionales en los recursos de infraestructura, como MSMQ, el registro de eventos, los contadores de rendimiento y el sistema de archivos, deben otorgarse los permisos adecuados a estos recursos para que el servicio WCF se pueda ejecutar correctamente.</li></ul><p>Si el servicio necesita acceder a recursos específicos en nombre del llamador original, utilice la suplantación y la delegación para transmitir la identidad del llamador para una comprobación de autorización indirecta. En un escenario de desarrollo, use la cuenta de servicio de red local, que es una cuenta especial integrada que tiene privilegios reducidos. En un escenario de producción, cree una cuenta de servicio de dominio personalizado con privilegios mínimos.</p>|

## <a name="force-all-traffic-to-web-apis-over-https-connection"></a><a id="webapi-https"></a>Direccionamiento forzoso de todo el tráfico a API web a través de una conexión HTTPS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Aplicación de SSL en un controlador de API web](https://www.asp.net/web-api/overview/security/working-with-ssl-in-web-api) |
| **Pasos** | Si una aplicación tiene tanto un enlace HTTP como HTTPS, los clientes aún pueden usar HTTP para acceder al sitio. Para evitar esto, utilice un filtro de acción para asegurarse de que las solicitudes de API protegidas se realizan siempre a través de HTTPS.|

### <a name="example"></a>Ejemplo 
El código siguiente muestra un filtro de autenticación de API web que comprueba el estado de TLS: 
```csharp
public class RequireHttpsAttribute : AuthorizationFilterAttribute
{
    public override void OnAuthorization(HttpActionContext actionContext)
    {
        if (actionContext.Request.RequestUri.Scheme != Uri.UriSchemeHttps)
        {
            actionContext.Response = new HttpResponseMessage(System.Net.HttpStatusCode.Forbidden)
            {
                ReasonPhrase = "HTTPS Required"
            };
        }
        else
        {
            base.OnAuthorization(actionContext);
        }
    }
}
```
Agregue este filtro a todas las acciones de API web que requieran TLS: 
```csharp
public class ValuesController : ApiController
{
    [RequireHttps]
    public HttpResponseMessage Get() { ... }
}
```
 
## <a name="ensure-that-communication-to-azure-cache-for-redis-is-over-tls"></a><a id="redis-ssl"></a>Comprobación de que la comunicación a Azure Cache for Redis se realiza a través de TLS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Cache for Redis | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Soporte técnico de Azure Redis TLS](https://azure.microsoft.com/documentation/articles/cache-faq/#when-should-i-enable-the-non-ssl-port-for-connecting-to-redis) |
| **Pasos** | El servidor de Redis no admite TLS desde el principio, pero Azure Cache for Redis sí. Si se conecta a Azure Cache for Redis y el cliente admite TLS, como StackExchange.Redis, deberá utilizar TLS. De forma predeterminada, el puerto no TLS está deshabilitado para instancias nuevas de Azure Cache for Redis. Asegúrese de que no se modifiquen los valores predeterminados seguros a menos que exista una dependencia de compatibilidad con TLS para los clientes de Redis. |

Tenga en cuenta que Redis está diseñado para que puedan acceder clientes de confianza dentro de entornos de confianza. Por ello, normalmente no es recomendable exponer la instancia de Redis directamente a Internet o, en general, a un entorno desde el que clientes que no sean de confianza puedan acceder directamente el puerto TCP de Redis o a un socket de UNIX. 

## <a name="secure-device-to-field-gateway-communication"></a><a id="device-field"></a>Protección de la comunicación entre el dispositivo y la puerta de enlace de campo

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de campo de IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | En el caso de dispositivos basados en IP, normalmente es posible encapsular el protocolo de comunicación en un canal SSL/TLS para proteger los datos en tránsito. Para otros protocolos que no admitan SSL/TLS, investigue si existen versiones seguras de esos protocolos que proporcionen seguridad en el nivel de transporte o de mensaje. |

## <a name="secure-device-to-cloud-gateway-communication-using-ssltls"></a><a id="device-cloud"></a>Protección de la comunicación entre el dispositivo y la puerta de enlace de nube mediante SSL/TLS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de la nube de IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Elección del protocolo de comunicación](https://azure.microsoft.com/documentation/articles/iot-hub-devguide/#messaging) |
| **Pasos** | Proteja los protocolos HTTP/AMQP o MQTT mediante SSL/TLS. |
