---
title: 'Administración de excepciones: Microsoft Threat Modeling Tool (Azure) | Microsoft Docs'
description: Obtenga información sobre la administración de excepciones en Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
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
ms.openlocfilehash: 8f68bc5e4604f35f9c4c45cd3e38ddaf8d24cd03
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004466"
---
# <a name="security-frame-exception-management--mitigations"></a>Marco de seguridad: Administración de excepciones: mitigaciones 
| Producto o servicio | Artículo |
| --------------- | ------- |
| **WCF** | <ul><li>[WCF: no incluya el nodo serviceDebug en el archivo de configuración](#servicedebug)</li><li>[WCF: no incluya el nodo serviceMetadata en el archivo de configuración](#servicemetadata)</li></ul> |
| **API web** | <ul><li>[Asegúrese de que se realiza un control adecuado de las excepciones en ASP.NET Web API](#exception)</li></ul> |
| **Aplicación web** | <ul><li>[No exponga detalles de seguridad en los mensajes de error](#messages)</li><li>[Implemente una página de control de errores predeterminado](#default)</li><li>[Establezca el método de implementación como deployment retail en IIS](#deployment)</li><li>[Las excepciones deben generar un error con seguridad](#fail)</li></ul> |

## <a name="wcf--do-not-include-servicedebug-node-in-configuration-file"></a><a id="servicedebug"></a>WCF: no incluya el nodo serviceDebug en el archivo de configuración

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_debug_information) |
| **Pasos** | Los servicios de Windows Communication Framework (WCF) pueden configurarse para exponer información de depuración. La información de depuración no debe usarse en entornos de producción. La etiqueta `<serviceDebug>` define si la característica de información de depuración está habilitada para un servicio WCF. Si el atributo includeExceptionDetailInFaults se establece en True, la información de excepción de la aplicación se devolverá a los clientes. Los atacantes pueden aprovechar la información adicional que obtengan del resultado de la depuración para montar los ataques dirigidos en el marco de trabajo, la base de datos u otros recursos usados por la aplicación. |

### <a name="example"></a>Ejemplo
El siguiente archivo de configuración incluye la etiqueta `<serviceDebug>`: 
```
<configuration> 
<system.serviceModel> 
<behaviors> 
<serviceBehaviors> 
<behavior name=""MyServiceBehavior""> 
<serviceDebug includeExceptionDetailInFaults=""True"" httpHelpPageEnabled=""True""/> 
... 
```
Deshabilite la información de depuración en el servicio. Puede hacerlo eliminando la etiqueta `<serviceDebug>` del archivo de configuración de la aplicación. 

## <a name="wcf--do-not-include-servicemetadata-node-in-configuration-file"></a><a id="servicemetadata"></a>WCF: no incluya el nodo serviceMetadata en el archivo de configuración

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Genérico, .NET Framework 3 |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_service_enumeration) |
| **Pasos** | Exponer públicamente información acerca de un servicio puede proporcionar a los atacantes valiosos detalles sobre cómo podrían aprovechar el servicio. La etiqueta `<serviceMetadata>` habilita la característica de publicación de metadatos. Los metadatos del servicio pueden contener información confidencial que no debe estar accesible públicamente. Como mínimo, permita solo a los usuarios de confianza acceder a los metadatos, y asegúrese de que no se expone información innecesaria. Mejor aún, deshabilite completamente la capacidad de publicar los metadatos. Una configuración segura de WCF no contendrá la etiqueta `<serviceMetadata>`. |

## <a name="ensure-that-proper-exception-handling-is-done-in-aspnet-web-api"></a><a id="exception"></a>Asegúrese de que se realiza un tratamiento adecuado de las excepciones en ASP.NET Web API

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC 5, MVC 6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Exception Handling in ASP.NET Web API](https://www.asp.net/web-api/overview/error-handling/exception-handling) (Control de excepciones en ASP.NET Web API), [Model Validation in ASP.NET Web API](https://www.asp.net/web-api/overview/formats-and-model-binding/model-validation-in-aspnet-web-api) (Validación de modelos en ASP.NET Web API) |
| **Pasos** | De forma predeterminada, la mayoría de las excepciones no detectadas en ASP.NET Web API se traducen en una respuesta HTTP con código de estado `500, Internal Server Error`|

### <a name="example"></a>Ejemplo
Para controlar el código de estado devuelto por la API, puede utilizarse `HttpResponseException`, tal y como se muestra a continuación: 
```csharp
public Product GetProduct(int id)
{
    Product item = repository.Get(id);
    if (item == null)
    {
        throw new HttpResponseException(HttpStatusCode.NotFound);
    }
    return item;
}
```

### <a name="example"></a>Ejemplo
Para controlar aún más la respuesta de la excepción, se puede utilizar la clase `HttpResponseMessage`, tal y como se muestra a continuación: 
```csharp
public Product GetProduct(int id)
{
    Product item = repository.Get(id);
    if (item == null)
    {
        var resp = new HttpResponseMessage(HttpStatusCode.NotFound)
        {
            Content = new StringContent(string.Format("No product with ID = {0}", id)),
            ReasonPhrase = "Product ID Not Found"
        }
        throw new HttpResponseException(resp);
    }
    return item;
}
```
Para detectar excepciones no controladas que no son del tipo `HttpResponseException`, se pueden usar filtros de excepción. Los filtros de excepción implementan la interfaz `System.Web.Http.Filters.IExceptionFilter`. La manera más sencilla de crear un filtro de excepción es derivar desde la clase `System.Web.Http.Filters.ExceptionFilterAttribute` e invalidar el método OnException. 

### <a name="example"></a>Ejemplo
A continuación puede ver un filtro que convierte excepciones `NotImplementedException` en código de estado HTTP `501, Not Implemented`: 
```csharp
namespace ProductStore.Filters
{
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Web.Http.Filters;

    public class NotImplExceptionFilterAttribute : ExceptionFilterAttribute 
    {
        public override void OnException(HttpActionExecutedContext context)
        {
            if (context.Exception is NotImplementedException)
            {
                context.Response = new HttpResponseMessage(HttpStatusCode.NotImplemented);
            }
        }
    }
}
```

Hay varias formas de registrar un filtro de excepción API web:
- Por acción
- Por controlador
- Globalmente

### <a name="example"></a>Ejemplo
Para aplicar el filtro a una acción concreta, agregue el filtro como un atributo a la acción: 
```csharp
public class ProductsController : ApiController
{
    [NotImplExceptionFilter]
    public Contact GetContact(int id)
    {
        throw new NotImplementedException("This method is not implemented");
    }
}
```
### <a name="example"></a>Ejemplo
Para aplicar el filtro a todas las acciones en un `controller`, agregue el filtro como un atributo a la clase `controller`: 

```csharp
[NotImplExceptionFilter]
public class ProductsController : ApiController
{
    // ...
}
```

### <a name="example"></a>Ejemplo
Para aplicar el filtro globalmente a todos los controladores de API web, agregue una instancia del filtro a la colección `GlobalConfiguration.Configuration.Filters`. Los filtros de excepción de esta colección se aplican a cualquier acción de controlador de API web. 
```csharp
GlobalConfiguration.Configuration.Filters.Add(
    new ProductStore.NotImplExceptionFilterAttribute());
```

### <a name="example"></a>Ejemplo
Para la validación del modelo, el estado del modelo se puede pasar al método CreateErrorResponse, tal y como se muestra a continuación: 
```csharp
public HttpResponseMessage PostProduct(Product item)
{
    if (!ModelState.IsValid)
    {
        return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
    }
    // Implementation not shown...
}
```

Compruebe los vínculos en la sección de referencias para obtener información más detallada acerca del control de excepciones y la validación del modelo en ASP.NET Web API. 

## <a name="do-not-expose-security-details-in-error-messages"></a><a id="messages"></a>No exponga detalles de seguridad en los mensajes de error

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Los mensajes de error genéricos se proporcionan directamente al usuario sin necesidad de incluir datos confidenciales de la aplicación. Ejemplos de datos confidenciales incluyen:</p><ul><li>Nombres de servidor</li><li>Cadenas de conexión</li><li>Nombres de usuario</li><li>Contraseñas</li><li>Procedimientos SQL</li><li>Detalles de errores SQL dinámicos</li><li>Seguimiento de la pila y líneas de código</li><li>Variables almacenadas en memoria</li><li>Ubicaciones de unidad y carpeta</li><li>Puntos de instalación de aplicación</li><li>Valores de la configuración de host</li><li>Otros detalles de la aplicación interna</li></ul><p>La captura de todos los errores dentro de una aplicación y el envío de mensajes de error genéricos, así como la habilitación de errores personalizados dentro de IIS, le ayudará a evitar la divulgación de información. La base de datos de SQL Server y el control de excepciones de .NET, entre otras arquitecturas de control de errores, son especialmente detallados y muy útiles para la evaluación del perfil de una aplicación por parte de un usuario malintencionado. No muestre directamente el contenido de una clase derivada de la clase de excepción de .NET, y asegúrese de que tiene un control de excepciones adecuado para que una excepción inesperada no se presente directamente al usuario.</p><ul><li>Proporcione directamente al usuario mensajes de error genéricos que condensen y alejen los detalles específicos que se encuentra directamente en el mensaje de excepción o error</li><li>No muestre el contenido de una clase de excepción de .NET directamente al usuario</li><li>Capture todos los mensajes de error, y si es necesario informe al usuario a través de un mensaje de error genérico enviado a la aplicación cliente</li><li>No exponga el contenido de la clase Exception directamente al usuario, especialmente el valor devuelto de `.ToString()`, o los valores de las propiedades del mensaje o el seguimiento de la pila. Registre esta información de forma segura y muestre un mensaje más inocuo al usuario</li></ul>|

## <a name="implement-default-error-handling-page"></a><a id="default"></a>Implemente una página de control de errores predeterminado

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Modificar configuración de páginas de errores ASP.NET (Cuadro de diálogo)](https://technet.microsoft.com/library/dd569096(WS.10).aspx) |
| **Pasos** | <p>Cuando se produce un error en una aplicación ASP.NET que provoca un error 1.x 500 o HTTP 500 Error interno del servidor, o la configuración de una característica (por ejemplo, el filtrado de solicitudes) impide que se muestre una página, se generará un mensaje de error. Los administradores pueden elegir si la aplicación debe mostrar un mensaje descriptivo al cliente, un mensaje de error detallado al cliente o un mensaje de error detallado solamente al host local. La etiqueta `<customErrors>` en el archivo web.config tiene tres modos:</p><ul><li>**On:** especifica que los errores personalizados están habilitados. Si no se especifica ningún atributo defaultRedirect, los usuarios verán un error genérico. Los errores personalizados se muestran a los clientes remotos y al host local</li><li>**Off:** especifica que los errores personalizados están deshabilitados. Los errores detallados de ASP.NET se muestran a los clientes remotos y al host local</li><li>**RemoteOnly:** especifica que los errores personalizados se muestran solo a los clientes remotos y que los errores de ASP.NET se muestran al host local. Este es el valor predeterminado</li></ul><p>Abra el archivo `web.config` de la aplicación o el sitio y asegúrese de que la etiqueta tiene definido `<customErrors mode="RemoteOnly" />` o `<customErrors mode="On" />`.</p>|

## <a name="set-deployment-method-to-retail-in-iis"></a><a id="deployment"></a>Establezca el método de implementación como deployment retail en IIS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [deployment (Elemento, Esquema de configuración de ASP.NET)](https://msdn.microsoft.com/library/ms228298(VS.80).aspx) |
| **Pasos** | <p>El conmutador `<deployment retail>` está diseñado para su uso en servidores IIS de producción. Este conmutador se usa para ayudar a las aplicaciones a ejecutarse con el mejor rendimiento y las mínimas fugas de información de seguridad posibles, esto se consigue deshabilitando la capacidad de la aplicación para generar el resultado de un seguimiento en una página y para mostrar mensajes de error detallados a los usuarios finales, y deshabilitando también el conmutador de depuración.</p><p>A menudo, los conmutadores y opciones que están enfocadas hacia en el desarrollador, como los errores de solicitud de seguimiento y depuración, se habilitan durante el desarrollo activo. Se recomienda que el método de implementación en cualquier servidor de producción se establezca como deployment retail. Abra el archivo machine.config y asegúrese de que `<deployment retail="true" />` permanece establecido en True.</p>|

## <a name="exceptions-should-fail-safely"></a><a id="fail"></a>Las excepciones deben generar un error con seguridad

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Fail securely](https://owasp.org/www-community/Fail_securely) (Control seguro de los errores) |
| **Pasos** | La aplicación debe generar un error con seguridad. Cualquier método que devuelve un valor booleano en el que se va a basar la toma de una decisión específica, debe tener el bloque de excepción creado con cuidado. Hay muchos errores lógicos debido a los que se pueden infiltrar problemas de seguridad cuando el bloque de excepción se escribe sin el cuidado necesario.|

### <a name="example"></a>Ejemplo
```csharp
        public static bool ValidateDomain(string pathToValidate, Uri currentUrl)
        {
            try
            {
                if (!string.IsNullOrWhiteSpace(pathToValidate))
                {
                    var domain = RetrieveDomain(currentUrl);
                    var replyPath = new Uri(pathToValidate);
                    var replyDomain = RetrieveDomain(replyPath);

                    if (string.Compare(domain, replyDomain, StringComparison.OrdinalIgnoreCase) != 0)
                    {
                        //// Adding additional check to enable CMS urls if they are not hosted on same domain.
                        if (!string.IsNullOrWhiteSpace(Utilities.CmsBase))
                        {
                            var cmsDomain = RetrieveDomain(new Uri(Utilities.Base.Trim()));
                            if (string.Compare(cmDomain, replyDomain, StringComparison.OrdinalIgnoreCase) != 0)
                            {
                                return false;
                            }
                            else
                            {
                                return true;
                            }
                        }

                        return false;
                    }
                }

                return true;
            }
            catch (UriFormatException ex)
            {
                LogHelper.LogException("Utilities:ValidateDomain", ex);
                return true;
            }
        }
```
El método anterior siempre devolverá True, si se produce alguna excepción. Si el usuario final proporciona una dirección URL con formato incorrecto que el explorador respeta, pero el constructor `Uri()` no, se producirá una excepción y se dirigirá a la víctima a la dirección URL válida pero con formato incorrecto. 
