---
title: 'Tutorial: Adición de Azure CDN a una aplicación web de Azure App Service | Microsoft Docs'
description: En este tutorial, se agrega una red de Content Delivery Network (CDN) a una aplicación web de Azure App Service para almacenar en memoria caché y entregar archivos estáticos desde servidores próximos a sus clientes en todo el mundo.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/14/2018
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 470324799cd157c8b33311e1cae8b5b698433e1f
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079916"
---
# <a name="tutorial-add-azure-cdn-to-an-azure-app-service-web-app"></a>Tutorial: Adición de Azure CDN a una aplicación web de Azure App Service

Este tutorial muestra cómo agregar [Azure Content Delivery Network (CDN)](cdn-overview.md) a una [aplicación web de Azure App Service](../app-service/overview.md). Web Apps es un servicio para hospedar aplicaciones web, API de REST y back-ends para dispositivos móviles. 

Esta es la página de inicio del ejemplo del sitio HTML estático que se utilizará:

![Página de inicio de la aplicación de ejemplo](media/cdn-add-to-web-app/sample-app-home-page.png)

Temas que se abordarán:

> [!div class="checklist"]
> * Crear un punto de conexión de CDN.
> * Actualizar los recursos en caché.
> * Utilizar cadenas de consulta para controlar las versiones en caché.


## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

- [Instalación de Git](https://git-scm.com/)
- [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-the-web-app"></a>Creación de la aplicación web

Para crear la aplicación web con la que va a trabajar, siga el [tutorial de inicio rápido de HTML estático](../app-service/quickstart-html.md) mediante el paso de **búsqueda de la aplicación**.

## <a name="log-in-to-the-azure-portal"></a>Iniciar sesión en Azure Portal

Abra un explorador y vaya a [Azure Portal](https://portal.azure.com).

### <a name="dynamic-site-acceleration-optimization"></a>Optimización de la aceleración de sitios dinámicos
Si desea optimizar el punto de conexión de CDN para la aceleración de sitios dinámicos (DSA), debe usar el [portal CDN](cdn-create-new-endpoint.md) para crear su perfil y el punto de conexión. Con la [optimización de la aceleración de sitios dinámicos](cdn-dynamic-site-acceleration.md), el rendimiento de las páginas web con contenido dinámico se ha mejorado de un modo contrastable. Para obtener instrucciones acerca de cómo optimizar un punto de conexión de CDN para DSA desde el portal de CDN, consulte [Configuración de punto de conexión de red CDN para acelerar la entrega de archivos dinámicos](cdn-dynamic-site-acceleration.md#cdn-endpoint-configuration-to-accelerate-delivery-of-dynamic-files). En caso contrario, si no desea optimizar el nuevo punto de conexión, puede usar el portal de aplicaciones web para crearlo siguiendo los pasos descritos en la siguiente sección. Tenga en cuenta que en los perfiles de **Azure CDN de Verizon** no se puede cambiar la optimización de un punto de conexión de CDN una vez que se ha creado.

## <a name="create-a-cdn-profile-and-endpoint"></a>Creación de un perfil y un punto de conexión de CDN

En el panel de navegación izquierdo, seleccione **App Services** y, a continuación, seleccione la aplicación que creó en el tutorial [Creación de una aplicación web HTML estática en Azure en cinco minutos](../app-service/quickstart-html.md).

![Selección de la aplicación de App Service en el portal](media/cdn-add-to-web-app/portal-select-app-services.png)

En la página **App Service**, en la sección **Configuración**, seleccione **Redes > Configurar Azure CDN para la aplicación**.

![Selección de la instancia de CDN en el portal](media/cdn-add-to-web-app/portal-select-cdn.png)

En la página **Azure Content Delivery Network**, proporcione la configuración del **Nuevo punto de conexión** tal y como se especifica en la tabla.

![Creación del perfil y el punto de conexión en el portal](media/cdn-add-to-web-app/portal-new-endpoint.png)

| Configuración | Valor sugerido | Descripción |
| ------- | --------------- | ----------- |
| **Perfil de CDN** | myCDNProfile | Un perfil de CDN es una colección de puntos de conexión de CDN con el mismo plan de tarifa. |
| **Plan de tarifa** | Estándar de Akamai | El [plan de tarifa](cdn-features.md) especifica el proveedor y las características disponibles. En este tutorial se usa *Standard Akamai*. |
| **Nombre del punto de conexión de CDN** | Cualquier nombre que sea único en el dominio azureedge.net | Accederá a los recursos en caché en el dominio *&lt;nombrePuntoConexión&gt;* .azureedge.net.

Seleccione **Crear** para crear un nuevo perfil de red CDN.

Azure crea el perfil y el punto de conexión. El nuevo punto de conexión aparece en la lista **Puntos de conexión** y, una vez aprovisionado, el estado es **En ejecución**.

![Nuevo punto de conexión en la lista](media/cdn-add-to-web-app/portal-new-endpoint-in-list.png)

### <a name="test-the-cdn-endpoint"></a>Probar el punto de conexión de CDN

 Dado que se tarda un tiempo en que el registro se propague, el punto de conexión no estará disponible para su uso de forma inmediata: 
   - En los perfiles de **Azure CDN Estándar de Microsoft**, la propagación se completa normalmente en 10 minutos. 
   - En los perfiles de **Azure CDN estándar**, la propagación normalmente se completa en un minuto. 
   - En los perfiles **Azure CDN de Verizon estándar** y **Azure CDN de Verizon premium**, la propagación se completa normalmente en 90 minutos. 

La aplicación de ejemplo tiene un archivo *index.html* y las carpetas *css*, *img* y *js* que contienen otros recursos estáticos. Las rutas de acceso de contenido para todos estos archivos son las mismas en el punto de conexión de CDN. Por ejemplo, las siguientes direcciones URL acceden ambas al archivo *bootstrap.css* en la carpeta *css*:

```
http://<appname>.azurewebsites.net/css/bootstrap.css
```

```
http://<endpointname>.azureedge.net/css/bootstrap.css
```

Mediante un explorador, vaya la dirección URL siguiente:

```
http://<endpointname>.azureedge.net/index.html
```

![Página de inicio de la aplicación de ejemplo servida desde la red CDN](media/cdn-add-to-web-app/sample-app-home-page-cdn.png)

 Verá la misma página que ejecutó anteriormente en una aplicación web de Azure. Azure CDN ha recuperado los recursos de la aplicación web original y los sirve desde el punto de conexión de CDN.

Para asegurarse de que esta página se almacena en caché en la red CDN, actualice la página. En ocasiones se necesitan dos solicitudes del mismo recurso para que la red CDN almacene en memoria caché el contenido solicitado.

Para más información acerca de cómo crear perfiles y puntos de conexión de Azure CDN, consulte [Introducción a Azure CDN](cdn-create-new-endpoint.md).

## <a name="purge-the-cdn"></a>Purga de la red CDN

La red CDN actualiza periódicamente los recursos desde la aplicación web original en función de la configuración del período de vida (TTL). El TTL predeterminado es siete días.

En ocasiones podría necesitar actualizar la red CDN antes de la expiración del TTL; por ejemplo, al implementar contenido actualizado en la aplicación web. Para desencadenar una actualización, purgue manualmente los recursos de la red CDN. 

En esta sección del tutorial, se implementará un cambio en la aplicación web y se purgará la red CDN para desencadenar la actualización de su caché.

### <a name="deploy-a-change-to-the-web-app"></a>Implementación de cambios en la aplicación web

Abra el archivo *index.html* y agregue *- V2* al encabezado H1, como se muestra en el ejemplo siguiente: 

```
<h1>Azure App Service - Sample Static HTML Site - V2</h1>
```

Confirme el cambio e impleméntelo en la aplicación web.

```bash
git commit -am "version 2"
git push azure master
```

Una vez finalizada la implementación, vaya a la dirección URL de la aplicación web para ver el cambio.

```
http://<appname>.azurewebsites.net/index.html
```

![V2 en el título de la aplicación web](media/cdn-add-to-web-app/v2-in-web-app-title.png)

Si va a la dirección URL de la página de inicio del punto de conexión de CDN, no verá el cambio porque la versión almacenada en caché en la red CDN no ha expirado todavía. 

```
http://<endpointname>.azureedge.net/index.html
```

![Sin título V2 en la red CDN](media/cdn-add-to-web-app/no-v2-in-cdn-title.png)

### <a name="purge-the-cdn-in-the-portal"></a>Purga de la red CDN en el portal

Para hacer que la red CDN actualice la versión almacenada en caché, purgue la red CDN.

En el área de navegación izquierda del portal, seleccione **Grupos de recursos** y, a continuación, seleccione el grupo de recursos que ha creado para la aplicación web (myResourceGroup).

![Selección de un grupo de recursos](media/cdn-add-to-web-app/portal-select-group.png)

En la lista de recursos, seleccione el punto de conexión de CDN.

![Selección del punto de conexión](media/cdn-add-to-web-app/portal-select-endpoint.png)

En la parte superior de la página **Punto de conexión**, seleccione **Purgar**.

![Seleccionar Purgar](media/cdn-add-to-web-app/portal-select-purge.png)

Escriba las rutas de acceso al contenido que desea purgar. Puede pasar una ruta de acceso de archivo completa para purgar un archivo individual, o un segmento de ruta de acceso para purgar y actualizar todo el contenido de una carpeta. Debido a que cambió *index.html*, asegúrese de que está en una de las rutas de acceso.

En la parte inferior de la página, seleccione **Purgar**.

![Página Purgar](media/cdn-add-to-web-app/app-service-web-purge-cdn.png)

### <a name="verify-that-the-cdn-is-updated"></a>Compruebe que se ha actualizado la red CDN

Espere hasta que la solicitud de purga finalice el procesamiento; por lo general, un par de minutos. Para ver el estado actual, seleccione el icono de campana en la parte superior de la página. 

![Notificación de purga](media/cdn-add-to-web-app/portal-purge-notification.png)

Cuando vaya a la dirección URL del punto de conexión de la red CDN para *index.html*, verá el *V2* que agregó en el título en la página de inicio, que indica que se ha actualizado la caché de la red CDN.

```
http://<endpointname>.azureedge.net/index.html
```

![V2 en el título en la red CDN](media/cdn-add-to-web-app/v2-in-cdn-title.png)

Para más información, consulte [Purgar un punto de conexión de Azure CDN](../cdn/cdn-purge-endpoint.md). 

## <a name="use-query-strings-to-version-content"></a>Utilizar cadenas de consulta para versiones del contenido

Azure CDN ofrece las siguientes opciones de comportamiento de almacenamiento en caché:

* Pasar por alto las cadenas de consulta
* Omitir el almacenamiento en caché de cadenas de consulta
* Almacenar en caché todas las URL únicas 

La primera opción es el comportamiento predeterminado, lo que significa que solo hay una versión almacenada en caché de un recurso, independientemente de la cadena de consulta utilizada en la dirección URL. 

En esta sección del tutorial, cambiará el comportamiento de almacenamiento en caché para almacenar en caché todas las direcciones URL únicas.

### <a name="change-the-cache-behavior"></a>Cambio del comportamiento de la memoria caché

En la página **Punto de conexión de CDN** de Azure Portal, seleccione **Caché**.

Seleccione **Almacenar en caché todas las URL únicas** en la lista desplegable **Comportamiento del almacenamiento en caché de cadenas de consulta**.

Seleccione **Guardar**.

![Selección del comportamiento de almacenamiento de cadenas de consulta en caché](media/cdn-add-to-web-app/portal-select-caching-behavior.png)

### <a name="verify-that-unique-urls-are-cached-separately"></a>Compruebe que las direcciones URL únicas se almacenan en caché por separado

En un explorador, vaya a la página de inicio del punto de conexión de CDN, e incluya una cadena de consulta: 

```
http://<endpointname>.azureedge.net/index.html?q=1
```

Azure CDN devuelve el contenido de la aplicación web actual, que incluye *V2* en el encabezado. 

Para asegurarse de que esta página se almacena en caché en la red CDN, actualice la página. 

Abra *index.html*, cambie *V2* a *V3* y, a continuación, implemente el cambio. 

```bash
git commit -am "version 3"
git push azure master
```

En un explorador, vaya a la dirección URL del punto de conexión CDN con una nueva cadena de consulta, como `q=2`. Azure CDN obtiene el archivo *index.html* actual y muestra *V3*. Sin embargo, si navega hasta el punto de conexión de CDN con la cadena de consulta `q=1`, verá *V2*.

```
http://<endpointname>.azureedge.net/index.html?q=2
```

![V3 en el título en la red CDN, cadena de consulta 2](media/cdn-add-to-web-app/v3-in-cdn-title-qs2.png)

```
http://<endpointname>.azureedge.net/index.html?q=1
```

![V2 en el título en la red CDN, cadena de consulta 1](media/cdn-add-to-web-app/v2-in-cdn-title-qs1.png)

Este resultado muestra que cada cadena de consulta se trata de manera diferente:

* q = 1 se usó antes, por lo que se devuelve contenido almacenado en caché (V2).
* q = 2 es nuevo, por lo que se recupera y se devuelve el contenido de aplicación web más reciente (V3).

Para más información, consulte [Control del comportamiento del almacenamiento en caché de Azure CDN con cadenas de consulta](../cdn/cdn-query-string.md).

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>Pasos siguientes

¿Qué ha aprendido?

> [!div class="checklist"]
> * Crear un punto de conexión de CDN.
> * Actualizar los recursos en caché.
> * Utilizar cadenas de consulta para controlar las versiones en caché.

Aprenda cómo optimizar el rendimiento de CDN en los siguientes artículos:

> [!div class="nextstepaction"]
> [Tutorial: Adición de un dominio personalizado a un punto de conexión de Azure CDN](cdn-map-content-to-custom-domain.md)


