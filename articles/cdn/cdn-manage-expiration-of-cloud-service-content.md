---
title: Administración de la expiración del contenido web en Azure CDN | Microsoft Docs
description: Aprenda a administrar la caducidad del contenido de Azure Web Apps/Cloud Services, ASP.NET o IIS en Azure CDN.
services: cdn
documentationcenter: .NET
author: asudbring
manager: danielgi
editor: ''
ms.assetid: bef53fcc-bb13-4002-9324-9edee9da8288
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: how-to
ms.date: 02/15/2018
ms.author: allensu
ms.openlocfilehash: 562d5010458fc938d9d62fed5d0d2c8284f2055d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936952"
---
# <a name="manage-expiration-of-web-content-in-azure-cdn"></a>Administración de la expiración del contenido web en Azure CDN
> [!div class="op_single_selector"]
> * [Contenido web de Azure](cdn-manage-expiration-of-cloud-service-content.md)
> * [Almacenamiento de blobs de Azure](cdn-manage-expiration-of-blob-content.md)
> 

Los archivos de servidores web de origen accesible públicamente se pueden almacenar en caché en Azure Content Delivery Network (CDN) hasta que transcurra su tiempo de vida (TTL). El período de vida viene determinado por el encabezado `Cache-Control` en la respuesta HTTP del servidor de origen. En este artículo se describe cómo establecer los encabezados `Cache-Control` para la característica Web Apps de Microsoft Azure App Service, Azure Cloud Services, las aplicaciones de ASP.NET y los sitios de Internet Information Services (IIS), ya que todos ellos se configuran de forma parecida. Puede establecer el encabezado `Cache-Control` mediante el uso de archivos de configuración o mediante programación. 

También puede controlar la configuración de caché desde Azure Portal estableciendo [reglas de almacenamiento en caché de la red CDN](cdn-caching-rules.md). Si crea una o más reglas de almacenamiento en caché y establece el comportamiento de dicho almacenamiento en **Invalidar** u **Omitir caché**, se omite la configuración de almacenamiento en caché proporcionada por el origen que se trata en este artículo. Para información sobre conceptos generales de almacenamiento en caché, consulte [Funcionamiento del almacenamiento en caché](cdn-how-caching-works.md).

> [!TIP]
> Puede optar por no configurar ningún TTL en un archivo. En este caso, Azure CDN aplica automáticamente un TTL predeterminado de siete días, a menos que haya configurado reglas de almacenamiento en caché en Azure Portal. Este TTL predeterminado solo se aplica a las optimizaciones de entrega web general. Para las optimizaciones de archivos de gran tamaño, el TTL predeterminado es un día, y para las optimizaciones de streaming multimedia, el TTL predeterminado es un año.
> 
> Para obtener más información sobre el funcionamiento de Azure CDN para acelerar el acceso a los archivos y a otros recursos, consulte [Información general de Azure Content Delivery Network](cdn-overview.md).
> 

## <a name="setting-cache-control-headers-by-using-cdn-caching-rules"></a>Establecimiento de los encabezados Cache-Control mediante reglas de almacenamiento en caché de CDN
El método preferido para establecer el encabezado `Cache-Control` de un servidor web consiste en usar las reglas de almacenamiento en caché en Azure Portal. Para más información sobre las reglas de almacenamiento en caché de CDN, consulte [Control del comportamiento del almacenamiento en caché de Azure CDN con reglas de almacenamiento en caché](cdn-caching-rules.md).

> [!NOTE] 
> Las reglas de almacenamiento en caché solo están disponibles para los perfiles **Azure CDN estándar de Verizon** y **Azure CDN estándar de Akamai**. Para perfiles **Azure CDN premium de Verizon**, debe usar el [motor de reglas de Azure CDN](cdn-rules-engine.md) en el portal **Administrar** para una funcionalidad similar.

**Para navegar a la página de reglas de almacenamiento en caché de CDN**:

1. En Azure Portal, seleccione un perfil de CDN y luego seleccione un punto de conexión para el servidor web.

1. En el panel izquierdo, debajo de Configuración, haga clic en **Reglas de caché**.

   ![Botón Reglas de caché de CDN](./media/cdn-manage-expiration-of-cloud-service-content/cdn-caching-rules-btn.png)

   Aparece la página **Reglas de caché**.

   ![Página de caché de CDN](./media/cdn-manage-expiration-of-cloud-service-content/cdn-caching-page.png)


**Para establecer encabezados Cache-Control de un servidor web con reglas de almacenamiento en caché globales:**

1. En **Reglas de almacenamiento en caché globales**, establezca **Comportamiento del almacenamiento en caché de cadenas de consulta** en **Ignorar cadenas de consulta**, y establezca **Comportamiento de almacenamiento en caché** en **Invalidar**.
      
1. Para **Duración de expiración de caché**, escriba 3600 en el cuadro **Segundos** o 1 en el cuadro **Horas**. 

   ![Ejemplo de reglas de almacenamiento en caché globales de CDN](./media/cdn-manage-expiration-of-cloud-service-content/cdn-global-caching-rules-example.png)

   Esta regla de almacenamiento en caché global establece una duración de caché de una hora y afecta a todas las solicitudes para el punto de conexión. Invalida todos los encabezados HTTP `Cache-Control` o `Expires` que envía el servidor de origen especificado por el punto de conexión.   

1. Seleccione **Guardar**.

**Para establecer encabezados Cache-Control del archivo de servidor web con reglas de almacenamiento en caché personalizadas:**

1. En **Reglas de almacenamiento en caché personalizadas**, cree dos condiciones de coincidencia:

     a. Para la primera condición de coincidencia, establezca **Condición de coincidencia** en **Ruta de acceso** y escriba `/webfolder1/*` en **Match value** (Valor de coincidencia). Establezca **Comportamiento de almacenamiento en caché** en **Invalidar** y escriba 4 en el cuadro **Horas**.

     b. Para la segunda condición de coincidencia, establezca **Condición de coincidencia** en **Ruta de acceso** y escriba `/webfolder1/file1.txt` en **Match value** (Valor de coincidencia). Establezca **Comportamiento de almacenamiento en caché** en **Invalidar** y escriba 2 en el cuadro **Horas**.

    ![Ejemplo de reglas de almacenamiento en caché personalizadas de CDN](./media/cdn-manage-expiration-of-cloud-service-content/cdn-custom-caching-rules-example.png)

    La primera regla de almacenamiento en caché personalizada establece una duración de caché de cuatro horas para los archivos en la carpeta `/webfolder1` en el servidor de origen especificado por el punto de conexión. La segunda regla invalida la primera regla para el archivo `file1.txt` únicamente y establece una duración de caché de dos horas para él.

1. Seleccione **Guardar**.


## <a name="setting-cache-control-headers-by-using-configuration-files"></a>Establecimiento de los encabezados Cache-Control mediante archivos de configuración
Para el contenido estático, como imágenes y hojas de estilos, puede controlar la frecuencia de actualización al modificar los archivos de configuración **applicationHost.config** o **Web.config** de la aplicación web. Para establecer el encabezado `Cache-Control` para su contenido, use el elemento `<system.webServer>/<staticContent>/<clientCache>` en cualquiera de los archivos.

### <a name="using-applicationhostconfig-files"></a>Uso de archivos ApplicationHost.config
El archivo **ApplicationHost.config** es el archivo raíz del sistema de configuración de IIS. El valor de la configuración del archivo **ApplicationHost.config** afecta a todas las aplicaciones del sitio, pero se reemplaza por la configuración de cualquier archivo **Web.config** existente en una aplicación web.

### <a name="using-webconfig-files"></a>Uso de archivos Web.config
Con un archivo **Web.config**, puede personalizar el comportamiento de toda la aplicación web o de un directorio específico en la aplicación web. Por lo general, tiene al menos un archivo **Web.config** en la carpeta raíz de la aplicación web. Para cada archivo **Web.config** de una carpeta específica, el valor de la configuración afecta a todo lo que haya en dicha carpeta y en sus subcarpetas, a menos que se invaliden en el nivel de subcarpeta por otro archivo **Web.config**. 

Por ejemplo, puede establecer un elemento `<clientCache>` de un archivo **Web.config** en la carpeta raíz de la aplicación web para almacenar en caché todo el contenido estático en la aplicación web durante tres días. También puede agregar un archivo **Web.config** a una subcarpeta con contenido más variable (por ejemplo, `\frequent`) y establecer el elemento `<clientCache>` para almacenar en caché contenido de la subcarpeta durante seis horas. El resultado neto es que el contenido de todo el sitio web se almacenará en caché durante tres días, excepto el contenido del directorio `\frequent`, que se almacenará en caché solo durante seis horas.  

En el archivo de configuración XML de ejemplo siguiente se muestra cómo establecer el elemento `<clientCache>` para especificar una antigüedad máxima de tres días:  

```xml
<configuration>
    <system.webServer>
        <staticContent>
            <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="3.00:00:00" />
        </staticContent>
    </system.webServer>
</configuration>
```

Para usar el atributo **cacheControlMaxAge**, debe establecer el valor del atributo **cacheControlMode** en `UseMaxAge`. Esta configuración hace que el encabezado HTTP y la directiva, `Cache-Control: max-age=<nnn>`, se agreguen a la respuesta. El formato del valor del intervalo de tiempo para el atributo **cacheControlMaxAge** es `<days>.<hours>:<min>:<sec>`. Su valor se convierte en segundos y se utiliza como el valor de la directiva `Cache-Control` `max-age`. Para más información sobre el elemento `<clientCache>`, consulte el elemento [Client Cache\<clientCache>](https://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache).  

## <a name="setting-cache-control-headers-programmatically"></a>Establecimiento de los encabezados Cache-Control mediante programación
Para las aplicaciones de ASP.NET, puede controlar el comportamiento de almacenamiento en caché de CDN mediante programación al establecer la propiedad **HttpResponse.Cache** de la API de .NET. Para más información sobre la propiedad **HttpResponse.Cache**, consulte [Propiedad HttpResponse.Cache](/dotnet/api/system.web.httpresponse.cache#System_Web_HttpResponse_Cache) y [Clase HttpCachePolicy](/dotnet/api/system.web.httpcachepolicy).  

Para almacenar en caché mediante programación el contenido de la aplicación de ASP.NET, siga estos pasos:
   1. Compruebe que el contenido esté marcado como almacenable en caché. Para ello, establezca `HttpCacheability` en `Public`. 
   1. Establezca un validador de caché mediante una llamada a uno de los métodos siguientes `HttpCachePolicy`:
      - Llame a `SetLastModified` para establecer un valor de marca de tiempo para el encabezado `Last-Modified`.
      - Llame a `SetETag` para establecer un valor para el encabezado `ETag`.
   1. Si lo desea, especifique una hora de expiración de caché mediante una llamada a `SetExpires` para establecer un valor para el encabezado `Expires`. En caso contrario, se aplica la heurística de caché predeterminada descrita anteriormente en este documento.

Por ejemplo, para almacenar en caché el contenido durante una hora, agregue el siguiente código de C#:  

```csharp
// Set the caching parameters.
Response.Cache.SetExpires(DateTime.Now.AddHours(1));
Response.Cache.SetCacheability(HttpCacheability.Public);
Response.Cache.SetLastModified(DateTime.Now);
```

## <a name="testing-the-cache-control-header"></a>Prueba del encabezado Cache-Control
Puede comprobar fácilmente la configuración de TTL de su contenido web. Con las [herramientas para desarrollador](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/) del explorador, compruebe que el contenido web incluye el encabezado de respuesta `Cache-Control`. Asimismo, también puede usar una herramienta como **wget**, [Postman](https://www.getpostman.com/) o [Fiddler](https://www.telerik.com/fiddler) para examinar los encabezados de respuesta.

## <a name="next-steps"></a>Pasos siguientes
* [Obtener información sobre el elemento **clientCache**](https://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache)
* [Lea la documentación de la propiedad **HttpResponse.Cache**](/dotnet/api/system.web.httpresponse.cache#System_Web_HttpResponse_Cache) 
* [Consulte la documentación de la **clase HttpCachePolicy**](/dotnet/api/system.web.httpcachepolicy)  
* [Obtenga información sobre conceptos del almacenamiento en caché](cdn-how-caching-works.md)
