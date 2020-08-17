---
title: Configuración de una interfaz de usuario hospedada para Bing Custom Search | Microsoft Docs
titleSuffix: Azure Cognitive Services
description: Use este artículo para configurar e integrar una interfaz de usuario hospedada para Bing Custom Search.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: conceptual
ms.date: 02/12/2019
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: 857195e0befb95035572b5b52a01d151b332518e
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406357"
---
# <a name="configure-your-hosted-ui-experience"></a>Configure su experiencia de interfaz de usuario hospedada.

Bing Custom Search proporciona una UI hospedada que puede integrar fácilmente en las aplicaciones web y páginas web como un fragmento de código de JavaScript. Mediante el portal de Bing Custom Search, puede configurar el diseño, el color y las opciones de búsqueda de la UI.



## <a name="configure-the-custom-hosted-ui"></a>Configuración de la interfaz de usuario hospedada personalizada

Para configurar una interfaz de usuario hospedada para las aplicaciones web, siga estos pasos: A medida que realice cambios, el panel de la derecha le proporcionará una vista previa de la interfaz de usuario. Los resultados de búsqueda mostrados no son los resultados reales de la instancia.

1. Inicie sesión en el [portal](https://customsearch.ai) de Bing Custom Search.  
  
2. Seleccione una instancia de Bing Custom Search.

3. Haga clic en la pestaña **Hosted UI** (UI hospedada).  
  
4. Seleccione un diseño.

    - Barra de búsqueda y resultados (valor predeterminado): Muestra un cuadro de búsqueda con los resultados de búsqueda debajo de él.
    - Solo los resultados: Muestra solo los resultados de búsqueda sin un cuadro de búsqueda. Al utilizar este diseño, debe proporcionar la consulta de búsqueda (`&q=<query string>`). Agregue el parámetro de consulta a la dirección URL de solicitud en el fragmento de código de JavaScript o el vínculo de punto de conexión HTML.
    - Ventana emergente: Proporciona un cuadro de búsqueda y muestra los resultados de búsqueda en una superposición deslizante.

5. Seleccione un tema de color. Puede personalizar los colores para ajustarse a la aplicación haciendo clic en **Personalizar tema**. Para cambiar un color, escriba el valor HEXADECIMAL RGB del color (por ejemplo, `#366eb8`) o haga clic en la vista previa del color.

   Puede obtener una vista previa de los cambios en el lado derecho del portal. Al hacer clic en **Restablecer valores predeterminados**, se revertirán los cambios para volver a los colores predeterminados del tema seleccionado.

   > [!NOTE]
   > Considere la posibilidad de accesibilidad al elegir los colores.

6. En **Configuraciones adicionales**, proporcione los valores según corresponda para la aplicación. Estos parámetros son opcionales. Para ver el efecto de aplicarlos o eliminarlos, consulte el panel de vista previa de la derecha. Las opciones de configuración disponibles son las siguientes:  

7. Escriba la clave de suscripción de búsqueda o elija una en la lista desplegable. La lista desplegable se completa con las claves de las suscripciones de su cuenta de Azure. Consulte [Creación de una cuenta de Cognitive Services APIs en Azure Portal](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account).  

8. Si habilita las sugerencias automáticas, escriba la clave de suscripción de sugerencias automáticas o elija una en la lista desplegable. La lista desplegable se completa con las claves de las suscripciones de su cuenta de Azure. Custom Autosuggest requiere un nivel de suscripción específico, consulte [precios](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/).

[!INCLUDE [publish or revert](./includes/publish-revert.md)]

## <a name="consume-custom-ui"></a>Uso de la IU personalizada

Para utilizar la IU hospedada, elija una de estas dos opciones: 

- Incluir el script en la página web  
  
  ```html
  <html>
      <body>
          <script type="text/javascript" 
              id="bcs_js_snippet"
              src="https://ui.customsearch.ai /api/ux/rendering-js?customConfig=<YOUR-CUSTOM-CONFIG-ID>&market=en-US&safeSearch=Moderate&version=latest&q=">
          </script>
      </body>    
  </html>
  ```

- O bien, puede usar la siguiente dirección URL en un explorador web.   
  
  `https://ui.customsearch.ai/hosted?customConfig=YOUR-CUSTOM-CONFIG-ID`  
  
  > [!NOTE]
  > Agregue los siguientes parámetros de consulta a la dirección URL según sea necesario. Para más información acerca de estos parámetros, consulte la referencia de [Custom Search API](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-custom-search-api-v7-reference#query-parameters).
  >
  > - q
  > - mkt
  > - safesearch
  > - setlang

  > [!IMPORTANT]
  > La página no puede mostrar su declaración de privacidad ni otros avisos y términos. La idoneidad de su uso puede variar.  

Para obtener más información, incluido el identificador de configuración personalizada, vaya a **Puntos de conexión** en la pestaña **Producción**.

## <a name="configuration-options"></a>Opciones de configuración

Para configurar el comportamiento de la interfaz de usuario hospedada, haga clic en **Configuraciones adicionales** y proporcione los valores. Estos parámetros son opcionales. Para ver el efecto de aplicarlos o eliminarlos, consulte el panel de vista previa de la derecha. 

### <a name="web-search-configurations"></a>Configuraciones de búsqueda web

- Resultados web habilitados: Determina si está habilitada la búsqueda web (verá una pestaña Web en la parte superior de la página).
- Habilitar las sugerencias automáticas: Determina si están habilitadas las sugerencias automáticas personalizadas (consulte [precios](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/) para conocer los posibles costos adicionales).
- Resultados web por página: Número de resultados de la búsqueda web para mostrar a la vez (el máximo son 50 resultados por página).
- Leyenda de imagen: Determina si las imágenes se muestran con resultados de búsqueda.

Las configuraciones siguientes se mostrarán si hace clic en **Mostrar configuraciones avanzadas**:

- Resaltar palabras: Determina si se muestran los resultados con los términos de búsqueda en negrita.
- Destino del vínculo: Determina si la página web se abre en una nueva pestaña del explorador (en blanco) o en la misma pestaña del explorador (suya propia) cuando el usuario hace clic en un resultado de búsqueda.

### <a name="image-search-configurations"></a>Configuraciones de búsqueda de imágenes

- Resultados de imagen habilitados: Determina si está habilitada la búsqueda de imágenes (verá una pestaña Imágenes en la parte superior de la página).
- Resultados de imágenes por página: Número de resultados de la búsqueda de imágenes para mostrar a la vez (el máximo son 150 resultados por página).

La configuración siguiente se mostrará si hace clic en **Mostrar configuraciones avanzadas**.  
  
- Habilitar filtros: Agrega filtros que el usuario puede utilizar para filtrar las imágenes que devuelve Bing. Por ejemplo, el usuario puede filtrar los resultados para que aparezcan solo GIF animados.

### <a name="video-search-configurations"></a>Configuraciones de búsqueda de vídeos

- Resultados de vídeo habilitados: Determina si está habilitada la búsqueda de vídeos (verá una pestaña Vídeos en la parte superior de la página).
- Resultados de vídeo por página: Número de resultados de la búsqueda de vídeos para mostrar a la vez (el máximo son 150 resultados por página).

La configuración siguiente se mostrará si hace clic en **Mostrar configuraciones avanzadas**.  
  
- Habilitar filtros: Agrega filtros que el usuario puede utilizar para filtrar los vídeos que devuelve Bing. Por ejemplo, el usuario puede filtrar los resultados de vídeos con una resolución específica o vídeos detectados en las últimas 24 horas.

### <a name="miscellaneous-configurations"></a>Configuraciones diversas

- Título de la página: El texto mostrado en el área de título de la página de resultados de búsqueda (no para el diseño de ventana emergente).
- Tema de la barra de herramientas: Determina el color de fondo del área de título de la página de resultados de la búsqueda.

Las configuraciones siguientes se mostrarán si hace clic en **Mostrar configuraciones avanzadas**.  

|Column1  |Column2  |
|---------|---------|
|Marcador de posición de texto del cuadro de búsqueda   | Texto mostrado en el cuadro de búsqueda antes de la entrada.        |
|Dirección URL del vínculo de título    |Destino para el vínculo del título.         |
|URL del logotipo     | Imagen que se muestra junto al título.         |
|Icono de favoritos    | Icono que se muestra en la barra de título del explorador.          |

Las siguientes configuraciones se aplican solo si usa la interfaz de usuario hospedada mediante el punto de conexión HTML (no se aplican si usa el fragmento de código de JavaScript).

- Título de la página
- Tema de la barra de herramientas
- Dirección URL del vínculo de título
- URL del logotipo
- Dirección URL del icono de favoritos  

## <a name="next-steps"></a>Pasos siguientes

- [Uso de marcadores de decoración para resaltar texto](../bing-web-search/hit-highlighting.md)
- [Paginación de páginas web](./page-webpages.md)
