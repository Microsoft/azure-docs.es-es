---
title: Conexión a Bing Search
description: Automatizar tareas y flujos de trabajo que encuentren resultados en Bing Search mediante Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 05/21/2018
tags: connectors
ms.openlocfilehash: 52bf42434640dc965999895549b4fa12a139dcce
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284071"
---
# <a name="find-results-in-bing-search-by-using-azure-logic-apps"></a>Buscar resultados en Bing Search mediante Azure Logic Apps

En este artículo se muestra cómo se pueden buscar noticias, vídeos y otros artículos mediante Bing Search desde dentro de una aplicación lógica con el conector de Bing Search. De este modo, puede crear aplicaciones lógicas que automatizan las tareas y los flujos de trabajo para procesar los resultados de búsqueda y hacer que esos elementos estén disponibles para otras acciones. 

Por ejemplo, puede encontrar noticias basadas en criterios de búsqueda y hacer que Twitter las publique como tweets en su fuente de Twitter.

Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).
Para obtener información técnica específica del conector, consulte la [referencia sobre el conector de Bing Search](/connectors/bingsearch/).

## <a name="prerequisites"></a>Prerrequisitos

* Una [cuenta de Cognitive Services](../cognitive-services/cognitive-services-apis-create-account.md)

* Una [clave de API de Bing Search](https://azure.microsoft.com/try/cognitive-services/?api=bing-news-search-api), que proporciona acceso desde su aplicación lógica a las API de Bing Search.

* La aplicación lógica donde quiere acceder al centro de eventos. Para iniciar la aplicación lógica con un desencadenador de Bing Search, necesita una [aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="add-trigger"></a>

## <a name="add-a-bing-search-trigger"></a>Adición de un desencadenador de Bing Search

En Azure Logic Apps, cada aplicación lógica debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts), que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica y empieza a ejecutar el flujo de trabajo de la aplicación.

1. En Azure Portal o Visual Studio, cree una aplicación lógica en blanco que abre el diseñador de aplicaciones lógicas. En este ejemplo se usa Azure Portal.

2. En el cuadro de búsqueda, escriba "Bing Search" como filtro. En la lista de desencadenadores, seleccione el que desee.

   En este ejemplo se utiliza este desencadenador: **Bing Search: en el nuevo artículo de noticias**

   ![Búsqueda del desencadenador de Bing Search](./media/connectors-create-api-bing-search/add-trigger.png)

3. Si se le piden los detalles de la conexión, [cree ahora su conexión de Bing Search](#create-connection).
O bien, si la conexión ya existe, especifique la información necesaria para el desencadenador.

   Para este ejemplo, proporcione criterios para devolver artículos de noticias coincidentes de Bing Search.

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | Search Query (Consulta de búsqueda) | Sí | <*search-words*> | Escriba las palabras clave de búsqueda que desee usar. |
   | Market | Sí | <*locale*> | Configuración regional de búsqueda. El valor predeterminado es "en-US", pero puede seleccionar otro valor. |
   | Safe Search | Sí | <*search-level*> | El nivel de filtro para excluir el contenido para adultos. El valor predeterminado es "Moderado", pero seleccione otro nivel. |
   | Count | No | <*results-count*> | Devuelve solo el número especificado de mensajes. El valor predeterminado es 20, pero puede especificar otro valor. El número real de los resultados devueltos puede ser menor que el número especificado. |
   | Offset | No | <*skip-value*> | El número de resultados que se van a omitir antes de devolver resultados |
   |||||

   Por ejemplo:

   ![Configuración del desencadenador](./media/connectors-create-api-bing-search/bing-search-trigger.png)

4. Seleccione el intervalo y la frecuencia con la que desea que el desencadenador busque resultados.

5. Cuando esté listo, seleccione **Guardar** en la barra de herramientas del diseñador.

6. Ahora, agregue a la aplicación lógica una o varias acciones, en función de las tareas que desea realizar con los resultados del desencadenador.

<a name="add-action"></a>

## <a name="add-a-bing-search-action"></a>Adición de una acción de Bing Search

En Azure Logic Apps, una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) es un paso del flujo de trabajo que sigue a un desencadenador u otra acción. Para este ejemplo, la aplicación lógica comienza con un desencadenador de Bing Search que devuelve artículos de noticias que cumplen los criterios especificados.

1. En Azure Portal o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas. En este ejemplo se usa Azure Portal.

2. En el desencadenador o la acción, seleccione **Nuevo paso** > **Agregar una acción**.

   En este ejemplo se utiliza este desencadenador:

   **Bing Search: en el nuevo artículo de noticias**

   ![Agregar una acción](./media/connectors-create-api-bing-search/add-action.png)

   Para agregar una acción entre los pasos existentes, mueva el mouse sobre la flecha de conexión. 
   Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

3. En el cuadro de búsqueda, escriba "Bing Search" como filtro.
En la lista de acciones, seleccione la que desee.

   Este ejemplo utiliza esta acción:

   **Bing Search: enumera las noticias por consulta**

   ![Búsqueda de acción de Bing Search](./media/connectors-create-api-bing-search/bing-search-select-action.png)

4. Si se le piden los detalles de la conexión, [cree ahora su conexión de Bing Search](#create-connection). O bien, si la conexión ya existe, especifique la información necesaria para la acción.

   Para este ejemplo, proporcione los criterios para devolver un subconjunto de los resultados del desencadenador.

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | Search Query (Consulta de búsqueda) | Sí | <*search-expression*> | Escriba una expresión para consultar los resultados del desencadenador. Puede seleccionar entre los campos de la lista de contenido dinámico o crear una expresión con el generador de expresiones. |
   | Market | Sí | <*locale*> | Configuración regional de búsqueda. El valor predeterminado es "en-US", pero puede seleccionar otro valor. |
   | Safe Search | Sí | <*search-level*> | El nivel de filtro para excluir el contenido para adultos. El valor predeterminado es "Moderado", pero seleccione otro nivel. |
   | Count | No | <*results-count*> | Devuelve solo el número especificado de mensajes. El valor predeterminado es 20, pero puede especificar otro valor. El número real de los resultados devueltos puede ser menor que el número especificado. |
   | Offset | No | <*skip-value*> | El número de resultados que se van a omitir antes de devolver resultados |
   |||||

   Por ejemplo, supongamos que desea aquellos resultados cuyo nombre de categoría incluya la palabra "tech".

   1. Haga clic en el cuadro **Consulta de búsqueda** para que aparezca la lista de contenido dinámico. 
   En esa lista, seleccione **Expresión** para que aparezca el generador de expresiones. 

      ![Desencadenador de Bing Search](./media/connectors-create-api-bing-search/bing-search-action.png)

      Ahora puede empezar a crear la expresión.

   2. En la lista de funciones, seleccione la función **contains()** , que luego aparece en el cuadro de expresión. Haga clic en **Contenido dinámico** para que vuelva a aparecer en la lista de campos, pero asegúrese de que el cursor permanece dentro de los paréntesis.

      ![Selección de una función](./media/connectors-create-api-bing-search/expression-select-function.png)

   3. En la lista de campos, seleccione **Categoría**, que se convierte a un parámetro. 
   Agregue una coma después del primer parámetro y, después de la coma, agregue esta palabra: `'tech'` 

      ![Selección de un campo](./media/connectors-create-api-bing-search/expression-select-field.png)

   4. Cuando finalice, seleccione **Aceptar**.

      La expresión ahora aparece en el cuadro **Consulta de búsqueda** en este formato:

      ![Expresión terminada](./media/connectors-create-api-bing-search/resolved-expression.png)

      En la vista de código, esta expresión aparece en este formato:

      `"@{contains(triggerBody()?['category'],'tech')}"`

5. Cuando esté listo, seleccione **Guardar** en la barra de herramientas del diseñador.

<a name="create-connection"></a>

## <a name="connect-to-bing-search"></a>Conexión a Bing Search

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Cuando se le solicite la información de conexión, proporcione estos detalles:

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | Nombre de la conexión | Sí | <*connection-name*> | El nombre que se va a crear para su conexión |
   | Versión de API | Sí | <*Versión de API*> | De forma predeterminada, la versión de la API de Bing Search se establece en la versión actual. Puede seleccionar una versión anterior si es necesario. |
   | Clave de API | Sí | <*API-key*> | La clave de API de Bing Search que obtuvo anteriormente. Si no tiene una clave, obtenga su [clave de API ahora](https://azure.microsoft.com/try/cognitive-services/?api=bing-news-search-api). |  
   |||||  

   Por ejemplo:

   ![Crear conexión](./media/connectors-create-api-bing-search/bing-search-create-connection.png)

2. Seleccione **Crear** cuando haya terminado.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener datos técnica, como los desencadenadores, las acciones y los límites, tal como lo describe el archivo Swagger del conector, consulte la [página de referencia del conector](/connectors/bingsearch/).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

