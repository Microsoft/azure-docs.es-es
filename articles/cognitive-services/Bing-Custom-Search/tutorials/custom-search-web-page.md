---
title: 'Tutorial: Creación de una página web de búsqueda personalizada con Bing Custom Search'
titleSuffix: Azure Cognitive Services
description: Aprenda a configurar una instancia de búsqueda personalizada de Bing y a integrarla en una página web con este tutorial.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: tutorial
ms.date: 03/05/2019
ms.author: aahi
ms.openlocfilehash: da1ada27b9420643f6f0b15f0986c245ba585acd
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85921947"
---
# <a name="tutorial-build-a-custom-search-web-page"></a>Tutorial: Creación de una página web de Custom Search

Bing Custom Search le permite crear experiencias de búsqueda a medida de los temas que le interesan. Por ejemplo, si tiene un sitio web de artes marciales que incluye la posibilidad de realizar búsquedas, puede especificar los dominios, subsitios y páginas web en las que busca Bing. Los usuarios ven los resultados de búsqueda adaptados al contenido que les interesa en lugar de recorrer las páginas de resultados de búsqueda generales que pueden contener contenido carente de interés. 

En este tutorial se muestra cómo configurar una instancia de búsqueda personalizada e integrarla en una nueva página web.

Las tareas que se tratan son:

> [!div class="checklist"]
> - Creación de una instancia de búsqueda personalizada
> - Agregar entradas activas
> - Agregar entradas bloqueadas
> - Agregar entradas ancladas
> - Integrar la búsqueda personalizada en una página web

## <a name="prerequisites"></a>Requisitos previos

- Para seguir con el tutorial, necesita una clave de suscripción para Bing Custom Search API.  Para obtener una clave, [cree un recurso de Bing Custom Search](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesBingCustomSearch) en Azure Portal.
- Si aún no tiene instalado Visual Studio 2017, o cualquier versión posterior, puede descargar y usar la edición **gratuita** de [Visual Studio 2019 Community](https://www.visualstudio.com/downloads/).

## <a name="create-a-custom-search-instance"></a>Creación de una instancia de búsqueda personalizada

Para crear una instancia de Bing Custom Search:

1. Abra un explorador de Internet.  
  
2. Navegue al [portal](https://customsearch.ai) de Custom Search.  
  
3. Inicie sesión en el portal mediante una cuenta Microsoft (MSA). Si no tiene una MSA, haga clic en **Crear una cuenta Microsoft**. Si es la primera vez que usa el portal, se le pedirán los permisos necesarios para acceder a los datos. Haga clic en **Sí**.  
  
4. Después de iniciar sesión, haga clic en **New custom search** (Nueva búsqueda personalizada). En la ventana **Create a new custom search instance** (Crear una instancia de búsqueda personalizada), escriba un nombre que sea significativo y que describa el tipo de contenido que devuelve la búsqueda. Puede cambiar el nombre en cualquier momento.  
  
   ![Captura de pantalla del cuadro de instancia Crear una nueva búsqueda personalizada](../media/newCustomSrch.png)  
  
5. Haga clic en Aceptar, especifique una dirección URL y si se deben incluir subpáginas de la dirección URL.  
  
   ![Captura de pantalla de la página de definición de dirección URL](../media/newCustomSrch1-a.png)  


## <a name="add-active-entries"></a>Agregar entradas activas

Para incluir los resultados de direcciones URL o sitios web específicos, agréguelos a la pestaña **Activos**.

1. En la página **Configuración**, haga clic en la pestaña **Activos** y escriba la dirección URL de uno o varios sitios web que va a incluir en la búsqueda.

    ![Captura de pantalla de la pestaña Activos del Editor de definiciones](../media/customSrchEditor.png)

2. Para confirmar que la instancia devuelve resultados, escriba una consulta en el panel de vista previa de la derecha. Bing devuelve solo los resultados de sitios web públicos que ha indizado.

## <a name="add-blocked-entries"></a>Agregar entradas bloqueadas

Para excluir resultados de direcciones URL o sitios web específicos, agréguelos a la pestaña **Bloqueados**.

1. En la página **Configuración**, haga clic en la pestaña **Bloqueados** y escriba la dirección URL de uno o más sitios web que quiera excluir de la búsqueda.

    ![Captura de pantalla de la pestaña Bloqueados del Editor de definiciones](../media/blockedCustomSrch.png)


2. Para confirmar que la instancia no devuelve resultados de los sitios web bloqueados, escriba una consulta en el panel de vista previa de la derecha. 

## <a name="add-pinned-entries"></a>Agregar entradas ancladas

Para anclar una página web específica a la parte superior de los resultados de búsqueda, agregue la página web y el término de consulta a la pestaña **Pinned** (Anclado). La pestaña **Pinned** (Anclado) contiene una lista de pares de término de consulta y página web que especifican la página web que aparece como mejor resultado para una consulta específica. La página web se ancla solo si la cadena de consulta del usuario coincide con la cadena de consulta del ancla en función de la condición de coincidencia del ancla. En las búsquedas solo se mostrarán las páginas web indexadas. Para más información, consulte [Definición de una vista personalizada](../define-your-custom-view.md#pin-slices-to-the-top-of-search-results).

1. En la página **Configuración**, haga clic en la pestaña **Anclados** y escriba el término de consulta y de página web de la página web que desee que se devuelva como el primer resultado.  
  
2. De forma predeterminada, la cadena de consulta del usuario debe coincidir exactamente con la cadena de consulta de su ancla para que Bing devuelva la página web como el primer resultado. Para cambiar la condición de coincidencia, edite el código de anclado (haga clic en el icono de lápiz), haga clic en Exacta en la columna **Condición de coincidencia de la consulta** y seleccione la condición de coincidencia que sea adecuada para su aplicación.  
  
    ![Captura de pantalla del Editor de definición de la pestaña Anclados](../media/pinnedCustomSrch.png)
  
3. Para confirmar que la instancia devuelve la página web especificada como mejor resultado, escriba el término de consulta que ancló en el panel de vista previa de la derecha.

## <a name="configure-hosted-ui"></a>Configuración de la interfaz de usuario hospedada

Custom Search proporciona una interfaz de usuario hospedada para presentar la respuesta JSON de la instancia de búsqueda personalizada. Para definir la experiencia de interfaz de usuario:

1. Haga clic en la pestaña **Hosted UI** (UI hospedada).  
  
2. Seleccione un diseño.  
  
   ![Captura de pantalla de Seleccionar diseño de interfaz de usuario hospedada](./media/custom-search-hosted-ui-select-layout.png)  
  
3. Seleccione un tema de color.  
  
   ![Captura de pantalla de Seleccionar tema de color de interfaz de usuario hospedada](./media/custom-search-hosted-ui-select-color-theme.png)  

   Si necesita ajustar el tema de color para integrar mejor su aplicación web, haga clic en **Personalizar tema**. No todas las configuraciones de color se aplican a todos los temas de diseño. Para cambiar un color, escriba el valor HEXADECIMAL RGB del color (por ejemplo, #366eb8) en el cuadro de texto correspondiente. O bien, haga clic en el botón de color y después en el tono que le convenga. Tenga en cuenta la accesibilidad siempre que seleccione colores.
  
   ![Captura de pantalla de Personalizar tema de color de interfaz de usuario hospedada](./media/custom-search-hosted-ui-customize-color-theme.png)  

  
4. Especifique opciones de configuración adicionales.  
  
   ![Captura de pantalla del paso Configuración adicional de interfaz de usuario hospedada](./media/custom-search-hosted-ui-additional-configurations.png)  
  
   Para obtener configuraciones avanzadas, haga clic en **Mostrar configuraciones avanzadas**. Esto agrega las configuraciones como *Destino del vínculo* a las opciones de búsqueda web, *Habilitar filtros* a las opciones de imagen y vídeo, y *Marcador de texto del cuadro de búsqueda* a Otras opciones.

   ![Captura de pantalla del paso Configuración avanzada de interfaz de usuario hospedada](./media/custom-search-hosted-ui-advanced-configurations.png)  
  
5. Seleccione las claves de suscripción en las listas desplegables. O bien, puede escribir manualmente la clave de suscripción.
  
   ![Captura de pantalla del paso Configuración adicional de interfaz de usuario hospedada](./media/custom-search-hosted-ui-subscription-key.png)

[!INCLUDE [publish or revert](../includes/publish-revert.md)]

<a name="consuminghostedui"></a>
## <a name="consuming-hosted-ui"></a>Consumo de la UI hospedada

Hay dos maneras de consumir la interfaz de usuario hospedada.  

- Opción 1: Integrar el fragmento de código de JavaScript proporcionado en la aplicación.
- Opción 2: Usar el punto de conexión HTML proporcionado.

En el resto del tutorial, se ilustra la **Opción 1: fragmento de código de JavaScript**.  

## <a name="set-up-your-visual-studio-solution"></a>Configuración de la solución de Visual Studio

1. Abra **Visual Studio** en el equipo.  
  
2. En el menú **Archivo**, seleccione **Nuevo** y elija **Proyecto**.  
  
3. En la ventana **Nuevo proyecto**, seleccione **Visual C#/Web/Aplicación web ASP.NET Core**, nombre el proyecto y, luego, haga clic en **OK** (Aceptar).  
  
   ![Captura de pantalla de la ventana Nuevo proyecto](./media/custom-search-new-project.png)  
  
4. En la ventana **Nueva aplicación web ASP.NET Core**, seleccione **Aplicación web** y haga clic en **Aceptar**.  
  
   ![Captura de pantalla de la ventana Nuevo proyecto](./media/custom-search-new-webapp.png)  

## <a name="edit-indexcshtml"></a>Edite index.cshtml.

1. En el **Explorador de soluciones**, expanda **Páginas** y haga doble clic en **index.cshtml** para abrir el archivo.  
  
   ![Captura de pantalla del Explorador de soluciones con las páginas expandidas e index.cshtml seleccionado](./media/custom-search-visual-studio-webapp-solution-explorer-index.png)  
  
2. En index.cshtml, elimine todo desde la línea 7 en adelante.  
  
   ```razor
   @page
   @model IndexModel
   @{
      ViewData["Title"] = "Home page";
   }    
   ```  
  
3. Agregue un elemento de salto de línea y una etiqueta div para que actúe como contenedor.  
  
   ```html
   @page
   @model IndexModel
   @{
      ViewData["Title"] = "Home page";
   }
   <br />
   <div id="customSearch"></div>
   ```  
  
4. En la página **UI hospedada**, desplácese hacia abajo hasta la sección titulada **Consumo de la interfaz de usuario**. Haga clic en el *Puntos de conexión* para tener acceso al fragmento de código de JavaScript. También puede obtener el fragmento de código haciendo clic en **Producción** y después en la pestaña **UI hospedada**.
  
   <!-- Get new screenshot after prod gets new bits
   ![Screenshot of the Hosted UI save button](./media/custom-search-hosted-ui-consuming-ui.png)  
   -->
  
5. Pegue el elemento de script en el contenedor que agregó.  
  
   ``` html
   @page
   @model IndexModel
   @{
      ViewData["Title"] = "Home page";
   }
   <br />
   <div id="customSearch">
      <script type="text/javascript" 
          id="bcs_js_snippet"
          src="https://ui.customsearch.ai /api/ux/rendering-js?customConfig=<YOUR-CUSTOM-CONFIG-ID>&market=en-US&safeSearch=Moderate&version=latest&q=">
      </script>
   </div>
   ```  
  
6. En el **Explorador de soluciones**, haga clic con el botón derecho en **wwwroot** y haga clic en **View in Browser** (Ver en el explorador).  
  
   ![Captura de pantalla del explorador de soluciones con Ver en el explorador seleccionado en el menú contextual de wwwroot](./media/custom-search-webapp-view-in-browser.png)  

La nueva página web de búsqueda personalizada debe parecerse a esta:

![Captura de pantalla de la página web de búsqueda personalizada](./media/custom-search-webapp-browse-index.png)

Realizar una búsqueda representa resultados similares al siguiente:

![Captura de pantalla de resultados de búsqueda personalizada](./media/custom-search-webapp-results.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Llamada a un punto de conexión de Bing Custom Search (C#)](../call-endpoint-csharp.md)
