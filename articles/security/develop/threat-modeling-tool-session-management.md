---
title: 'Administración de sesiones: Microsoft Threat Modeling Tool (Azure) | Microsoft Docs'
description: Obtenga información sobre la mitigación de la administración de sesiones para las amenazas expuestas en Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
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
ms.custom: has-adal-ref, devx-track-javascript
ms.openlocfilehash: 6a40d89db3e81721ec8a35973b79a558c17caee4
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543697"
---
# <a name="security-frame-session-management"></a>Marco de seguridad: Administración de sesiones
| Producto o servicio | Artículo |
| --------------- | ------- |
| **Azure AD**    | <ul><li>[Implemente el cierre de sesión correcto mediante métodos ADAL cuando use Azure AD](#logout-adal)</li></ul> |
| **Dispositivo IoT** | <ul><li>[Use duraciones finitas para los tokens de SaS generados](#finite-tokens)</li></ul> |
| **Azure Document DB** | <ul><li>[Use duraciones mínimas de token para los tokens de recursos generados](#resource-tokens)</li></ul> |
| **ADFS** | <ul><li>[Implemente el cierre de sesión correcto mediante métodos WsFederation cuando use ADFS](#wsfederation-logout)</li></ul> |
| **Identity Server** | <ul><li>[Implemente el cierre de sesión correcto cuando use Identity Server](#proper-logout)</li></ul> |
| **Aplicación web** | <ul><li>[Las aplicaciones disponibles a través de HTTPS deben usar cookies seguras](#https-secure-cookies)</li><li>[Todas las aplicaciones basadas en HTTP deben especificar HTTP solo para la definición de cookies](#cookie-definition)</li><li>[Mitigue el riesgo de ataques de falsificación de solicitud entre sitios (CSRF) en páginas web ASP.NET](#csrf-asp)</li><li>[Configure la sesión para la duración de la inactividad](#inactivity-lifetime)</li><li>[Implemente el cierre de sesión correcto desde la aplicación](#proper-app-logout)</li></ul> |
| **API web** | <ul><li>[Mitigue el riesgo de ataques de falsificación de solicitud entre sitios (CSRF) en las API web de ASP.NET](#csrf-api)</li></ul> |

## <a name="implement-proper-logout-using-adal-methods-when-using-azure-ad"></a><a id="logout-adal"></a>Implemente el cierre de sesión correcto mediante métodos ADAL cuando use Azure AD

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure AD | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Si la aplicación se basa en el token de acceso emitido por Azure AD, el controlador de eventos de cierre de sesión debe realizar una llamada. |

### <a name="example"></a>Ejemplo
```csharp
HttpContext.GetOwinContext().Authentication.SignOut(OpenIdConnectAuthenticationDefaults.AuthenticationType, CookieAuthenticationDefaults.AuthenticationType)
```

### <a name="example"></a>Ejemplo
También debe destruir la sesión del usuario llamando al método Session.Abandon(). En el siguiente método, se muestra una implementación segura del cierre de sesión del usuario:
```csharp
    [HttpPost]
        [ValidateAntiForgeryToken]
        public void LogOff()
        {
            string userObjectID = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
            AuthenticationContext authContext = new AuthenticationContext(Authority + TenantId, new NaiveSessionCache(userObjectID));
            authContext.TokenCache.Clear();
            Session.Clear();
            Session.Abandon();
            Response.SetCookie(new HttpCookie("ASP.NET_SessionId", string.Empty));
            HttpContext.GetOwinContext().Authentication.SignOut(
                OpenIdConnectAuthenticationDefaults.AuthenticationType,
                CookieAuthenticationDefaults.AuthenticationType);
        } 
```

## <a name="use-finite-lifetimes-for-generated-sas-tokens"></a><a id="finite-tokens"></a>Use duraciones finitas para los tokens de SaS generados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dispositivo IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Los tokens de SaS generados para la autenticación en Azure IoT Hub deben tener un período finito de expiración. Mantenga las duraciones de los tokens de SaS en un mínimo para limitar el período de tiempo que se pueden reproducir en caso de que los tokens se vean en peligro.|

## <a name="use-minimum-token-lifetimes-for-generated-resource-tokens"></a><a id="resource-tokens"></a>Use duraciones mínimas de token para los tokens de recursos generados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure DocumentDB | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Reduzca el intervalo de tiempo del token de recurso a un valor mínimo necesario. Los tokens de recursos tienen un intervalo de tiempo válido predeterminado de 1 hora.|

## <a name="implement-proper-logout-using-wsfederation-methods-when-using-adfs"></a><a id="wsfederation-logout"></a>Implemente el cierre de sesión correcto mediante métodos WsFederation cuando use ADFS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | ADFS | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Si la aplicación se basa en un token de STS emitido por ADFS, el controlador de eventos de cierre de sesión debe llamar al método WSFederationAuthenticationModule.FederatedSignOut() para cerrar la sesión del usuario. Además, se debe destruir la sesión actual, y se debe restablecer y anular el valor del token de sesión.|

### <a name="example"></a>Ejemplo
```csharp
        [HttpPost, ValidateAntiForgeryToken]
        [Authorization]
        public ActionResult SignOut(string redirectUrl)
        {
            if (!this.User.Identity.IsAuthenticated)
            {
                return this.View("LogOff", null);
            }

            // Removes the user profile.
            this.Session.Clear();
            this.Session.Abandon();
            HttpContext.Current.Response.Cookies.Add(new System.Web.HttpCookie("ASP.NET_SessionId", string.Empty)
                {
                    Expires = DateTime.Now.AddDays(-1D),
                    Secure = true,
                    HttpOnly = true
                });

            // Signs out at the specified security token service (STS) by using the WS-Federation protocol.
            Uri signOutUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Issuer);
            Uri replyUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Realm);
            if (!string.IsNullOrEmpty(redirectUrl))
            {
                replyUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Realm + redirectUrl);
            }
           //     Signs out of the current session and raises the appropriate events.
            var authModule = FederatedAuthentication.WSFederationAuthenticationModule;
            authModule.SignOut(false);
        //     Signs out at the specified security token service (STS) by using the WS-Federation
        //     protocol.            
            WSFederationAuthenticationModule.FederatedSignOut(signOutUrl, replyUrl);
            return new RedirectResult(redirectUrl);
        }
```

## <a name="implement-proper-logout-when-using-identity-server"></a><a id="proper-logout"></a>Implemente el cierre de sesión correcto cuando use Identity Server

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Identity Server | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [IdentityServer3-Federated Signout](https://identityserver.github.io/Documentation/docsv2/advanced/federated-signout.html) (Cierre de sesión federado en IdentityServer3) |
| **Pasos** | IdentityServer admite la capacidad de federación con proveedores de identidades externos. Cuando un usuario cierra sesión en un proveedor de identidades precedente, según el protocolo que se use, podría ser posible recibir una notificación cuando el usuario cierre la sesión. Esto permite, a su vez, que IdentityServer notifique a sus clientes para que también cierren la sesión del usuario. Consulte la documentación en la sección Referencias para ver los detalles de la implementación.|

## <a name="applications-available-over-https-must-use-secure-cookies"></a><a id="https-secure-cookies"></a>Las aplicaciones disponibles a través de HTTPS deben usar cookies seguras

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | EnvironmentType - OnPrem |
| **Referencias**              | [httpCookies (Elemento, Esquema de configuración de ASP.NET)](https://msdn.microsoft.com/library/ms228262(v=vs.100).aspx), [Propiedad HttpCookie.Secure](https://msdn.microsoft.com/library/system.web.httpcookie.secure.aspx) |
| **Pasos** | Normalmente, las cookies solo son accesibles para el dominio a cuyo ámbito se limitaron. Lamentablemente, la definición de "dominio" no incluye el protocolo, así que las cookies que se crean a través de HTTPS son accesibles a través de HTTP. El atributo "secure" indica al explorador que la cookie solo debe estar disponible a través de HTTPS. Asegúrese de que todas las cookies establecidas a través de HTTPS usen el atributo **secure**. Para aplicar el requisito en el archivo Web.config, establezca el atributo requireSSL en true. Se trata del método preferido porque aplicará el atributo **secure** para todas las cookies actuales y futuras sin necesidad de que se realice ningún otro cambio en el código.|

### <a name="example"></a>Ejemplo
```csharp
<configuration>
  <system.web>
    <httpCookies requireSSL="true"/>
  </system.web>
</configuration>
```
La configuración se aplica incluso si se usa HTTP para tener acceso a la aplicación. Si se utiliza HTTP para tener acceso a la aplicación, la configuración la interrumpe porque las cookies están establecidas con el atributo secure y el explorador no las enviará de vuelta a la aplicación.

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Formularios Web Forms, MVC5 |
| **Atributos**              | EnvironmentType - OnPrem |
| **Referencias**              | N/D  |
| **Pasos** | Cuando la aplicación web es el usuario de confianza y el proveedor de identidades es el servidor AD FS, se puede configurar el atributo secure del token FedAuth estableciendo requireSSL en True en la sección `system.identityModel.services` del archivo Web.config:|

### <a name="example"></a>Ejemplo
```csharp
  <system.identityModel.services>
    <federationConfiguration>
      <!-- Set requireSsl=true; domain=application domain name used by FedAuth cookies (Ex: .gdinfra.com); -->
      <cookieHandler requireSsl="true" persistentSessionLifetime="0.0:20:0" />
    ....  
    </federationConfiguration>
  </system.identityModel.services>
```

## <a name="all-http-based-application-should-specify-http-only-for-cookie-definition"></a><a id="cookie-definition"></a>Todas las aplicaciones basadas en HTTP deben especificar HTTP solo para la definición de cookies

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Atributo de cookie segura](https://en.wikipedia.org/wiki/HTTP_cookie#Secure_cookie) |
| **Pasos** | Para mitigar el riesgo de divulgación de información por un ataque de scripting entre sitios (XSS), se incorporó un nuevo atributo (httpOnly) a las cookies, el cual es compatible con todos los exploradores principales. El atributo especifica que una cookie no es accesible a través de un script. Al usar cookies HttpOnly, una aplicación web reduce la posibilidad de que se robe mediante un script la información confidencial contenida en la cookie y se envíe al sitio web de un atacante. |

### <a name="example"></a>Ejemplo
Todas las aplicaciones basadas en HTTP que usen cookies deben especificar HttpOnly en la definición de la cookie; para ello, se debe implementar la siguiente configuración en el archivo Web.config:
```XML
<system.web>
.
.
   <httpCookies requireSSL="false" httpOnlyCookies="true"/>
.
.
</system.web>
```

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Formularios Web Forms |
| **Atributos**              | N/D  |
| **Referencias**              | [Propiedad FormsAuthentication.RequireSSL](https://msdn.microsoft.com/library/system.web.security.formsauthentication.requiressl.aspx) |
| **Pasos** | El valor de la propiedad RequireSSL se establece en el archivo de configuración para una aplicación ASP.NET mediante el atributo requireSSL del elemento de configuración. Puede especificar en el archivo Web.config de la aplicación ASP.NET si se requiere Seguridad de la capa de transporte (TLS), conocido anteriormente como SSL (Capa de sockets seguros), para devolver la cookie de autenticación de formularios al servidor estableciendo el atributo requireSSL.|

### <a name="example"></a>Ejemplo 
En el ejemplo de código siguiente, se establece el atributo requireSSL en el archivo Web.config.
```XML
<authentication mode="Forms">
  <forms loginUrl="member_login.aspx" cookieless="UseCookies" requireSSL="true"/>
</authentication>
```

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC5 |
| **Atributos**              | EnvironmentType - OnPrem |
| **Referencias**              | [Windows Identity Foundation (WIF) Configuration – Part II](https://blogs.msdn.microsoft.com/alikl/2011/02/01/windows-identity-foundation-wif-configuration-part-ii-cookiehandler-chunkedcookiehandler-customcookiehandler/) (Configuración de Windows Identity Foundation: parte II) |
| **Pasos** | Para establecer el atributo httpOnly para las cookies FedAuth, el valor del atributo hideFromScript se debe establecer en True. |

### <a name="example"></a>Ejemplo
En el siguiente ejemplo se muestra la configuración correcta:
```XML
<federatedAuthentication>
<cookieHandler mode="Custom"
                       hideFromScript="true"
                       name="FedAuth"
                       path="/"
                       requireSsl="true"
                       persistentSessionLifetime="25">
</cookieHandler>
</federatedAuthentication>
```

## <a name="mitigate-against-cross-site-request-forgery-csrf-attacks-on-aspnet-web-pages"></a><a id="csrf-asp"></a>Mitigue el riesgo de ataques de falsificación de solicitud entre sitios (CSRF) en páginas web ASP.NET

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | La falsificación de solicitud entre sitios (CSRF o XSRF) es un tipo de ataque en el que un atacante puede llevar a cabo acciones en el contexto de seguridad de una sesión establecida de un usuario diferente en un sitio web. El objetivo es modificar o eliminar contenido, si el sitio web de destino confía exclusivamente en cookies de sesión para autenticar la solicitud recibida. Un atacante podría aprovechar esta vulnerabilidad y hacer que el explorador de un usuario diferente cargue una dirección URL con un comando procedente de un sitio vulnerable en el que el usuario ya haya iniciado sesión. Hay muchas maneras de hacerlo, como hospedar un sitio web diferente que carga un recurso desde el servidor vulnerable o conseguir que el usuario haga clic en un vínculo. El ataque se puede evitar si el servidor envía un token adicional al cliente, requiere que el cliente lo incluya en todas las solicitudes futuras y comprueba que todas las solicitudes futuras incluyan un token asociado a la sesión actual, como con los métodos AntiForgeryToken o ViewState de ASP.NET. |

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | [XSRF/CSRF Prevention in ASP.NET MVC and Web Pages](https://www.asp.net/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages) (Prevención de XSRF y CSRF en ASP.NET MVC y Web Pages) |
| **Pasos** | En formularios anti-CSRF y de ASP.NET MVC: use el método del asistente `AntiForgeryToken` en vistas; incluya un elemento `Html.AntiForgeryToken()` en el formulario, como en el siguiente ejemplo.|

### <a name="example"></a>Ejemplo
```csharp
@using (Html.BeginForm("UserProfile", "SubmitUpdate")) { 
    @Html.ValidationSummary(true) 
    @Html.AntiForgeryToken()
    <fieldset> 
```

### <a name="example"></a>Ejemplo
```csharp
<form action="/UserProfile/SubmitUpdate" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="saTFWpkKN0BYazFtN6c4YbZAmsEwG0srqlUqqloi/fVgeV2ciIFVmelvzwRZpArs" />
    <!-- rest of form goes here -->
</form>
```

### <a name="example"></a>Ejemplo
Al mismo tiempo, Html.AntiForgeryToken() proporciona al visitante una cookie denominada __RequestVerificationToken, con el mismo valor que el valor aleatorio de hidden mostrado arriba. A continuación, para validar un envío de formulario entrante, agregue el filtro [ValidateAntiForgeryToken] al método de acción de destino. Por ejemplo:
```
[ValidateAntiForgeryToken]
public ViewResult SubmitUpdate()
{
// ... etc.
}
```
Filtro de autorización que comprueba que:
* La solicitud entrante tenga una cookie denominada __RequestVerificationToken;
* La solicitud entrante tenga una entrada `Request.Form` denominada __RequestVerificationToken;
* Estos valores de cookie y `Request.Form` coincidan. Suponiendo que todo sea correcto, la solicitud entra de la forma normal. De lo contrario, aparece un error de autorización con el mensaje "No se especificó un token antifalsificación o el que se especificó no era válido". 

### <a name="example"></a>Ejemplo
Anti-CSRF y AJAX: el token de formulario puede ser un problema para las solicitudes AJAX, porque estas podrían enviar datos JSON en lugar de datos de formulario HTML. Una solución consiste en enviar los tokens en un encabezado HTTP personalizado. En el código siguiente, se utiliza sintaxis de Razor para generar los tokens, que después se agregan a una solicitud AJAX. 
```csharp
<script>
    @functions{
        public string TokenHeaderValue()
        {
            string cookieToken, formToken;
            AntiForgery.GetTokens(null, out cookieToken, out formToken);
            return cookieToken + ":" + formToken;                
        }
    }

    $.ajax("api/values", {
        type: "post",
        contentType: "application/json",
        data: {  }, // JSON data goes here
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
</script>
```

### <a name="example"></a>Ejemplo
Cuando procese la solicitud, extraiga los tokens del encabezado de la solicitud. Después, llame al método AntiForgery.Validate para validar los tokens. El método Validate produce una excepción si los tokens no son válidos.
```csharp
void ValidateRequestHeader(HttpRequestMessage request)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
```

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Formularios Web Forms |
| **Atributos**              | N/D  |
| **Referencias**              | [Cómo aprovechar las ventajas de las características integradas de ASP.NET para rechazar los ataques a través de Internet](https://msdn.microsoft.com/library/ms972969.aspx#securitybarriers_topic2) |
| **Pasos** | Los ataques CSRF en aplicaciones basadas en formularios Web Forms se pueden mitigar si se establece ViewStateUserKey en una cadena aleatoria que varíe para cada usuario, como el id. de usuario o, mejor aún, el identificador de sesión. Por diversas razones técnicas y sociales, el id. de sesión es mucho más adecuado porque es imprevisible, agota el tiempo de espera y varía para cada usuario.|

### <a name="example"></a>Ejemplo
Este es el código que debe tener en todas las páginas:
```csharp
void Page_Init (object sender, EventArgs e) {
   ViewStateUserKey = Session.SessionID;
   :
}
```

## <a name="set-up-session-for-inactivity-lifetime"></a><a id="inactivity-lifetime"></a>Configure la sesión para la duración de la inactividad

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Propiedad HttpSessionState.Timeout](https://msdn.microsoft.com/library/system.web.sessionstate.httpsessionstate.timeout(v=vs.110).aspx) |
| **Pasos** | El tiempo de espera de la sesión representa el evento que se produce cuando un usuario no realiza ninguna acción en un sitio web durante un intervalo (definido por el servidor web). El evento, en el lado servidor, cambia el estado de la sesión del usuario a "invalid" (por ejemplo, "ya no se usa") e indica al servidor web que lo destruirá (y eliminará todos los datos que contiene). En el ejemplo de código siguiente, se establece el atributo de sesión de tiempo de espera en 15 minutos en el archivo Web.config.|

### <a name="example"></a>Ejemplo
```XML 
<configuration>
  <system.web>
    <sessionState mode="InProc" cookieless="true" timeout="15" />
  </system.web>
</configuration>
```

## <a name="enable-threat-detection-on-azure-sql"></a><a id="threat-detection"></a>Habilitamiento de la detección de amenazas en Azure SQL

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Formularios Web Forms |
| **Atributos**              | N/D  |
| **Referencias**              | [Elemento forms para authentication (Esquema de configuración de ASP.NET)](https://msdn.microsoft.com/library/1d3t3c61(v=vs.100).aspx) |
| **Pasos** | Establezca el tiempo de espera de la cookie para el vale de autenticación de formularios en 15 minutos.|

### <a name="example"></a>Ejemplo
```XML
<forms  name=".ASPXAUTH" loginUrl="login.aspx"  defaultUrl="default.aspx" protection="All" timeout="15" path="/" requireSSL="true" slidingExpiration="true"/>
</forms>
```

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Formularios Web Forms, MVC5 |
| **Atributos**              | EnvironmentType - OnPrem |
| **Referencias**              | [asdeqa](https://skf.azurewebsites.net/Mitigations/Details/wefr) |
| **Pasos** | Cuando la aplicación web es Usuario de confianza y ADFS es el STS, la duración de las cookies de autenticación (tokens de FedAuth) se puede establecer mediante la siguiente configuración en web.config:|

### <a name="example"></a>Ejemplo
```XML
  <system.identityModel.services>
    <federationConfiguration>
      <!-- Set requireSsl=true; domain=application domain name used by FedAuth cookies (Ex: .gdinfra.com); -->
      <cookieHandler requireSsl="true" persistentSessionLifetime="0.0:15:0" />
      <!-- Set requireHttps=true; -->
      <wsFederation passiveRedirectEnabled="true" issuer="http://localhost:39529/" realm="https://localhost:44302/" reply="https://localhost:44302/" requireHttps="true"/>
      <!--
      Use the code below to enable encryption-decryption of claims received from ADFS. Thumbprint value varies based on the certificate being used.
      <serviceCertificate>
        <certificateReference findValue="4FBBBA33A1D11A9022A5BF3492FF83320007686A" storeLocation="LocalMachine" storeName="My" x509FindType="FindByThumbprint" />
      </serviceCertificate>
      -->
    </federationConfiguration>
  </system.identityModel.services>
```

### <a name="example"></a>Ejemplo
Además, la duración del token de notificación SAML emitido por ADFS se debe establecer en 15 minutos ejecutando el siguiente comando de PowerShell en el servidor de AD FS:
```csharp
Set-ADFSRelyingPartyTrust -TargetName "<RelyingPartyWebApp>" -ClaimsProviderName @("Active Directory") -TokenLifetime 15 -AlwaysRequireAuthentication $true
```

## <a name="implement-proper-logout-from-the-application"></a><a id="proper-app-logout"></a>Implemente el cierre de sesión correcto desde la aplicación

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Realice el cierre de sesión correctamente desde la aplicación, cuando el usuario presione el botón de cierre de sesión. Al cerrar la sesión, la aplicación debe destruir la sesión del usuario y también restablecer y anular el valor de la cookie de la sesión, así como restablecer y anular el valor de la cookie de autenticación. Además, cuando hay varias sesiones asociadas a una sola identidad de usuario, se deben finalizar colectivamente en el lado servidor cuando se agote el tiempo de espera o se produzca el cierre de sesión. Por último, asegúrese de que la funcionalidad de cierre de sesión esté disponible en todas las páginas. |

## <a name="mitigate-against-cross-site-request-forgery-csrf-attacks-on-aspnet-web-apis"></a><a id="csrf-api"></a>Mitigue el riesgo de ataques de falsificación de solicitud entre sitios (CSRF) en las API web de ASP.NET

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | La falsificación de solicitud entre sitios (CSRF o XSRF) es un tipo de ataque en el que un atacante puede llevar a cabo acciones en el contexto de seguridad de una sesión establecida de un usuario diferente en un sitio web. El objetivo es modificar o eliminar contenido, si el sitio web de destino confía exclusivamente en cookies de sesión para autenticar la solicitud recibida. Un atacante podría aprovechar esta vulnerabilidad y hacer que el explorador de un usuario diferente cargue una dirección URL con un comando procedente de un sitio vulnerable en el que el usuario ya haya iniciado sesión. Hay muchas maneras de hacerlo, como hospedar un sitio web diferente que carga un recurso desde el servidor vulnerable o conseguir que el usuario haga clic en un vínculo. El ataque se puede evitar si el servidor envía un token adicional al cliente, requiere que el cliente lo incluya en todas las solicitudes futuras y comprueba que todas las solicitudes futuras incluyan un token asociado a la sesión actual, como con los métodos AntiForgeryToken o ViewState de ASP.NET. |

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Preventing Cross-Site Request Forgery (CSRF) Attacks in ASP.NET Web API](https://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks) (Prevención de ataques de falsificación de solicitud entre sitios [CSRF] en ASP.NET Web API) |
| **Pasos** | Anti-CSRF y AJAX: el token de formulario puede ser un problema para las solicitudes AJAX, porque estas podrían enviar datos JSON en lugar de datos de formulario HTML. Una solución consiste en enviar los tokens en un encabezado HTTP personalizado. En el código siguiente, se utiliza sintaxis de Razor para generar los tokens, que después se agregan a una solicitud AJAX. |

### <a name="example"></a>Ejemplo
```Javascript
<script>
    @functions{
        public string TokenHeaderValue()
        {
            string cookieToken, formToken;
            AntiForgery.GetTokens(null, out cookieToken, out formToken);
            return cookieToken + ":" + formToken;                
        }
    }
    $.ajax("api/values", {
        type: "post",
        contentType: "application/json",
        data: {  }, // JSON data goes here
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
</script>
```

### <a name="example"></a>Ejemplo
Cuando procese la solicitud, extraiga los tokens del encabezado de la solicitud. Después, llame al método AntiForgery.Validate para validar los tokens. El método Validate produce una excepción si los tokens no son válidos.
```csharp
void ValidateRequestHeader(HttpRequestMessage request)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
```

### <a name="example"></a>Ejemplo
En formularios anti-CSRF y de ASP.NET MVC: use método del asistente AntiForgeryToken en vistas; incluya un elemento Html.AntiForgeryToken() en el formulario, como en el siguiente ejemplo.
```csharp
@using (Html.BeginForm("UserProfile", "SubmitUpdate")) { 
    @Html.ValidationSummary(true) 
    @Html.AntiForgeryToken()
    <fieldset> 
}
```

### <a name="example"></a>Ejemplo
La salida se parecerá a lo siguiente:
```csharp
<form action="/UserProfile/SubmitUpdate" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="saTFWpkKN0BYazFtN6c4YbZAmsEwG0srqlUqqloi/fVgeV2ciIFVmelvzwRZpArs" />
    <!-- rest of form goes here -->
</form>
```

### <a name="example"></a>Ejemplo
Al mismo tiempo, Html.AntiForgeryToken() proporciona al visitante una cookie denominada __RequestVerificationToken, con el mismo valor que el valor aleatorio de hidden mostrado arriba. A continuación, para validar un envío de formulario entrante, agregue el filtro [ValidateAntiForgeryToken] al método de acción de destino. Por ejemplo:
```
[ValidateAntiForgeryToken]
public ViewResult SubmitUpdate()
{
// ... etc.
}
```
Filtro de autorización que comprueba que:
* La solicitud entrante tenga una cookie denominada __RequestVerificationToken;
* La solicitud entrante tenga una entrada `Request.Form` denominada __RequestVerificationToken;
* Estos valores de cookie y `Request.Form` coincidan. Suponiendo que todo sea correcto, la solicitud entra de la forma normal. De lo contrario, aparece un error de autorización con el mensaje "No se especificó un token antifalsificación o el que se especificó no era válido".

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC5, MVC6 |
| **Atributos**              | Proveedor de identidades; ADFS, Proveedor de identidades; Azure AD |
| **Referencias**              | [Secure a Web API with Individual Accounts and Local Login in ASP.NET Web API 2.2](https://www.asp.net/web-api/overview/security/individual-accounts-in-web-api) (Protección de una API web con cuentas individuales e inicio de sesión local en ASP.NET Web API 2.2) |
| **Pasos** | Si la API web se protege con OAuth 2.0, espera un token de portador en el encabezado de solicitud de autorización y concede acceso a la solicitud solo si el token es válido. A diferencia de la autenticación basada en cookies, los exploradores no adjuntan los tokens de portador a las solicitudes. El cliente solicitante debe adjuntar explícitamente el token de portador al encabezado de solicitud. Por lo tanto, para las API web de ASP.NET protegidas con OAuth 2.0, los tokens de portador se consideran una defensa contra los ataques CSRF. Tenga en cuenta que, si la parte MVC de la aplicación utiliza la autenticación de formularios (es decir, usa cookies), la aplicación web de MVC debe usar tokens antifalsificación. |

### <a name="example"></a>Ejemplo
Se ha de informar a la API web de que solo debe confiar en tokens de portador y no en cookies. Se puede realizar mediante la siguiente configuración en el método `WebApiConfig.Register`:

```csharp
config.SuppressDefaultHostAuthentication();
config.Filters.Add(new HostAuthenticationFilter(OAuthDefaults.AuthenticationType));
```

El método SuppressDefaultHostAuthentication indica a API Web que ignore toda autenticación que se produzca antes de que la solicitud llegue a la canalización de API Web, ya sea mediante IIS o middleware de OWIN. De este modo, se puede restringir API Web para la autenticación solo con tokens de portador.
