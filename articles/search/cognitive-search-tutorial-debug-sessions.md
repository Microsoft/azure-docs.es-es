---
title: 'Tutorial: Uso de sesiones de depuración para diagnosticar, corregir y confirmar cambios en el conjunto de aptitudes'
titleSuffix: Azure Cognitive Search
description: Las sesiones de depuración (versión preliminar) proporcionan una interfaz basada en el portal para evaluar y reparar incidencias y errores en el conjunto de aptitudes
author: tchristiani
ms.author: terrychr
manager: nitinme
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 05/19/2020
ms.openlocfilehash: 2f25cab211d24facea7863c15b6d7671a9657ae9
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290684"
---
# <a name="tutorial-diagnose-repair-and-commit-changes-to-your-skillset"></a>Tutorial: Diagnóstico, reparación y confirmación de cambios en el conjunto de aptitudes

En este artículo, usará Azure Portal para acceder a las sesiones de depuración y reparar incidencias con el conjunto de aptitudes proporcionado. El conjunto de aptitudes tiene algunos errores que deben solucionarse. Este tutorial le guiará a través de una sesión de depuración para identificar y resolver incidencias con las entradas y salidas de aptitudes.

> [!Important]
> Las sesiones de depuración constituyen una característica en versión preliminar que se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

> [!div class="checklist"]
> * Suscripción a Azure. Cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) o use su suscripción actual
> * Una instancia del servicio Azure Cognitive Search
> * Una cuenta de Azure Storage.
> * [Aplicación de escritorio Postman](https://www.getpostman.com/)

## <a name="create-services-and-load-data"></a>Creación de servicios y carga de datos

En este tutorial se usan los servicios Azure Cognitive Search y Azure Storage.

* [Descargue datos de ejemplo](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/clinical-trials-pdf-19) (19 archivos).

* [Cree una cuenta de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal) o [busque una cuenta existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/). 

   Elija la misma región de Azure Cognitive Search para evitar cargos de ancho de banda.
   
   Elija el tipo de cuenta StorageV2 (uso general V2).

* Abra las páginas de los servicios de almacenamiento y cree un contenedor. El procedimiento recomendado es especificar el nivel de acceso "privado". Asigne al contenedor el nombre `clinicaltrialdataset`.

* En el contenedor, haga clic en **Cargar** para cargar los archivos de ejemplo que descargó y descomprimió en el primer paso.

* [Cree un servicio de Azure Cognitive Search](search-create-service-portal.md) o [busque un servicio existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Puede usar un servicio gratuito para este inicio rápido.

## <a name="get-a-key-and-url"></a>Obtención de una clave y una dirección URL

Las llamadas de REST requieren la dirección URL del servicio y una clave de acceso en cada solicitud. Con ambos se crea un servicio de búsqueda, por lo que, si ha agregado Azure Cognitive Search a su suscripción, siga estos pasos para obtener la información necesaria:

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

1. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

![Obtención de una clave de acceso y un punto de conexión HTTP](media/search-get-started-postman/get-url-key.png "Obtención de una clave de acceso y un punto de conexión HTTP")

Todas las solicitudes requieren una clave de API en cada solicitud enviada al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

## <a name="create-data-source-skillset-index-and-indexer"></a>Creación de un origen de datos, un conjunto de aptitudes, un índice y un indexador

En esta sección, se usan Postman y una colección proporcionada para crear el origen de datos del servicio de búsqueda, el conjunto de aptitudes, el índice y el indexador.

1. Si no tiene Postman, puede [descargar la aplicación de escritorio de Postman aquí](https://www.getpostman.com/).
1. [Descargue la colección de Postman de sesiones de depuración](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Debug-sessions).
1. Inicie Postman.
1. En **Archivos** > **Nuevo**, seleccione la colección que desee importar.
1. Una vez importada la colección, expanda la lista de acciones (...).
1. Haga clic en **Editar**.
1. Escriba el nombre de searchService (por ejemplo, si el punto de conexión es `https://mydemo.search.windows.net`, el nombre del servicio es "`mydemo`").
1. Escriba el valor de apiKey con la clave principal o secundaria del servicio de búsqueda.
1. Escriba el valor de storageConnectionString de la página de claves de la cuenta de Azure Storage.
1. Escriba el valor de containerName del contenedor que creó en la cuenta de almacenamiento.

> [!div class="mx-imgBorder"]
> ![Edición de variables en Postman](media/cognitive-search-debug/postman-enter-variables.png)

La colección contiene cuatro llamadas REST diferentes que se usan para completar esta sección.

La primera llamada crea el origen de datos. `clinical-trials-ds`. La segunda llamada crea el conjunto de aptitudes, `clinical-trials-ss`. La tercera llamada crea el índice, `clinical-trials`. La cuarta y última llamada crea el indexador, `clinical-trials-idxr`. Una vez completadas todas las llamadas de la colección, cierre Postman y vuelva a Azure Portal.

> [!div class="mx-imgBorder"]
> ![Uso de Postman para crear el origen de datos](media/cognitive-search-debug/postman-create-data-source.png)

## <a name="check-the-results"></a>Comprobar los resultados

El conjunto de aptitudes contiene algunos errores comunes. En esta sección, la ejecución de una consulta vacía para devolver todos los documentos mostrará varios errores. En pasos posteriores, las incidencias se resolverán mediante una sesión de depuración.

1. Vaya al servicio de búsqueda en Azure Portal. 
1. Seleccione la pestaña **Índices**. 
1. Seleccione el índice `clinical-trials`.
1. Haga clic en **Buscar** para ejecutar una consulta vacía. 

Una vez finalizada la búsqueda, se enumeran dos campos sin datos; en la ventana se enumeran "organizaciones" y "ubicaciones". Siga los pasos para detectar todas las incidencias generadas por el conjunto de aptitudes.

1. Vuelva a la página de información general del servicio de búsqueda.
1. Seleccione la pestaña **Indexadores**. 
1. Haga clic en `clinical-trials-idxr` y seleccione la notificación de advertencias. 

Hay muchas incidencias con las asignaciones de campos de salida de proyección y en la tercera página hay advertencias porque una o más entradas de aptitudes no son válidas.

Vuelva a la pantalla de información general del servicio de búsqueda.

## <a name="start-your-debug-session"></a>Inicio de la sesión de depuración

> [!div class="mx-imgBorder"]
> ![Inicio de una nueva sesión de depuración](media/cognitive-search-debug/new-debug-session-screen-required.png)

1. Haga clic en la pestaña Sesiones de depuración (versión preliminar).
1. Seleccione +NewDebugSession.
1. Asigne un nombre a la sesión. 
1. Conecte la sesión a su cuenta de almacenamiento. 
1. Proporcione el nombre del indexador. El indexador hace referencia al origen de datos, al conjunto de aptitudes y al índice.
1. Acepte la opción de documento predeterminada para el primer documento de la colección. 
1. **Guarde** la sesión. Al guardar la sesión se iniciará la canalización de enriquecimiento con IA tal y como se define en el conjunto de aptitudes.

> [!Important]
> Una sesión de depuración solo funciona con un único documento. Se puede seleccionar un documento específico del conjunto de datos. De lo contrario, la sesión se establecerá de forma predeterminada en el primer documento.

> [!div class="mx-imgBorder"]
> ![Nueva sesión de depuración iniciada](media/cognitive-search-debug/debug-execution-complete1.png)

Una vez completada la ejecución de la sesión de depuración, la sesión muestra de forma predeterminada la pestaña Enriquecimientos con IA, resaltando el gráfico de aptitudes.

+ El gráfico de aptitudes proporciona una jerarquía visual del conjunto de aptitudes y su orden de ejecución de arriba a abajo. Las habilidades que se encuentran en paralelo en el gráfico se ejecutan en paralelo. La codificación de colores de las aptitudes en el gráfico indica los tipos de aptitudes que se ejecutan en el conjunto de aptitudes. En el ejemplo, las aptitudes verdes son texto y la aptitud roja es visual. Al hacer clic en una aptitud determinada en el gráfico, se mostrarán los detalles de esa instancia de la aptitud en el panel derecho de la ventana de sesión. La configuración de las aptitudes, un editor JSON, los detalles de ejecución y los errores y advertencias están disponibles para su revisión y edición.
+ La estructura de datos enriquecidos detalla los nodos del árbol de enriquecimiento generados por las aptitudes a partir del contenido del documento de origen.

La pestaña Errores o advertencias proporcionará una lista mucho más reducida que la mostrada anteriormente, ya que esta lista solo detalla los errores de un único documento. Al igual que la lista mostrada por el indexador, puede hacer clic en un mensaje de advertencia y ver los detalles de dicha advertencia.

## <a name="fix-missing-skill-input-value"></a>Corrección de valores de entrada de aptitud que faltan

En la pestaña Errores o advertencias, se muestra un error en una operación con la etiqueta `Enrichment.NerSkillV2.#1`. Los detalles de este error explican que hay un problema con un valor de entrada de la aptitud "/document/languageCode". 

1. Vuelva a la pestaña Enriquecimientos con IA.
1. Haga clic en el **gráfico de aptitudes**.
1. Haga clic en la aptitud #1 para mostrar los detalles en el panel derecho.
1. Busque la entrada de "languageCode".
1. Seleccione el símbolo **</>** al principio de la línea y abra el evaluador de expresiones.
1. Haga clic en el botón **Evaluar** para confirmar que esta expresión genera un error. Esto confirmará que la propiedad "languageCode" no es una entrada válida.

> [!div class="mx-imgBorder"]
> ![Evaluador de expresiones](media/cognitive-search-debug/expression-evaluator-language.png)

Existen dos maneras de investigar este error en la sesión. La primera es examinar de dónde procede la entrada. ¿Qué aptitud de la jerarquía se supone que genera este resultado? En la pestaña Ejecuciones del panel de detalles de la aptitud debe aparecer el origen de la entrada. Si no hay ningún origen, esto indica un error de asignación de campos.

1. Haga clic en la pestaña **Ejecuciones**.
1. Examine las entradas y busque "languageCode". No hay ningún origen para esta entrada en la lista. 
1. Cambie el panel izquierdo para mostrar la estructura de datos enriquecidos. No hay ninguna ruta de acceso asignada que se corresponda con "languageCode".

> [!div class="mx-imgBorder"]
> ![Estructura de datos enriquecidos](media/cognitive-search-debug/enriched-data-structure-language.png)

Hay una ruta de acceso asignada para "language". Por lo tanto, hay un error tipográfico en la configuración de aptitudes. Para corregirlo, se debe actualizar la expresión de la aptitud #1 con la expresión "/document/language".

1. Abra el evaluador de expresiones **</>** para la ruta de acceso "language".
1. Copie la expresión. Cierre la ventana.
1. Vaya a la configuración de aptitudes de la aptitud #1 y abra el evaluador de expresiones **</>** para la entrada "languageCode".
1. Pegue el nuevo valor, "/document/language", en el cuadro Expresión y haga clic en **Evaluar**.
1. Debe mostrar la entrada correcta "en". Haga clic en Aplicar para actualizar la expresión.
1. Haga clic en **Guardar** en el panel derecho de detalles de la aptitud.
1. Haga clic en **Ejecutar** en el menú de la ventana de la sesión. Esto iniciará otra ejecución del conjunto de aptitudes con el documento. 

Una vez finalizada la ejecución de la sesión de depuración, haga clic en la pestaña Errores o advertencias, que mostrará que el error "Enrichment.NerSkillV2.#1" ha desaparecido. Sin embargo, sigue habiendo dos advertencias que indican que el servicio no pudo asignar campos de salida para organizaciones y ubicaciones al índice de búsqueda. Faltan los valores "/document/merged_content/organizations" y "/document/merged_content/locations".

## <a name="fix-missing-skill-output-values"></a>Corrección de valores de salida de aptitud que faltan

> [!div class="mx-imgBorder"]
> ![Errores y advertencias](media/cognitive-search-debug/warnings-missing-value-locations-organizations.png)

Faltan valores de salida de una aptitud. Para identificar la aptitud que tiene el error, vaya a la estructura de datos enriquecidos, busque el nombre del valor y observe su origen. En el caso de valores de organizaciones y ubicaciones que faltan, son salidas de la aptitud #1. Al abrir el evaluador de expresiones </> para cada ruta de acceso, se mostrarán las expresiones enumeradas como "/document/content/organizations" y "/document/content/locations" respectivamente.

> [!div class="mx-imgBorder"]
> ![Entidad de organizaciones del evaluador de expresiones](media/cognitive-search-debug/expression-eval-missing-value-locations-organizations.png)

La salida de estas entidades está vacía y no debería ser así. ¿Cuáles son las entradas que producen este resultado?

1. Vaya al **gráfico de aptitudes** y seleccione la aptitud #1.
1. En el panel de detalles de la aptitud de la derecha, seleccione la pestaña **Ejecuciones**.
1. Abra el evaluador de expresiones **</>** para la entrada "text".

> [!div class="mx-imgBorder"]
> ![Entrada de aptitud de texto](media/cognitive-search-debug/input-skill-missing-value-locations-organizations.png)

El resultado mostrado para esta entrada no parece una entrada de texto. Parece una imagen que está rodeada por líneas nuevas. La falta de texto indica que no se puede identificar ninguna entidad. Al examinar la jerarquía del conjunto de aptitudes, se muestra que la aptitud #6 (OCR) procesa primero el contenido que, a continuación, se pasa a la aptitud #5 (Combinar). 

1. Seleccione la aptitud #5 (Combinar) en el **gráfico de aptitudes**.
1. Seleccione la pestaña **Ejecuciones** en el panel de detalles de la aptitud de la derecha y abra el evaluador de expresiones **</>** para las salidas "mergedText".

> [!div class="mx-imgBorder"]
> ![Salida de la aptitud de combinación](media/cognitive-search-debug/merge-output-detail-missing-value-locations-organizations.png)

Aquí el texto se empareja con la imagen. Al examinar la expresión "/document/merged_content", se puede ver el error en las rutas de acceso de las entidades "organizations" y "locations" de la aptitud #1. En lugar de usar "/document/content", debe usar "/document/merged_content" para las entradas de "text".

1. Copie la expresión de la salida "mergedText" y cierre la ventana del evaluador de expresiones.
1. Seleccione la aptitud #1 en el **gráfico de aptitudes**.
1. En el panel de detalles de la aptitud de la derecha, seleccione la pestaña **Configuración de las aptitudes**.
1. Abra el evaluador de expresiones **</>** para la entrada "text".
1. Pegue la nueva expresión en el cuadro. Haga clic en **Evaluar**.
1. Se debe mostrar la entrada correcta con el texto agregado. Haga clic en **Aplicar** para actualizar la configuración de las aptitudes.
1. Haga clic en **Guardar** en el panel derecho de detalles de la aptitud.
1. Haga clic en **Ejecutar** en el menú de la ventana de sesiones. Esto iniciará otra ejecución del conjunto de aptitudes con el documento.

Una vez finalizada la ejecución del indexador, los errores siguen estando ahí. Vuelva a la aptitud #1 e investigue. La entrada de la aptitud se cambió de "content" a "merged_content". ¿Cuáles son las salidas de estas entidades en la aptitud?

1. Seleccione la pestaña **Enriquecimientos con IA**.
1. Seleccione el **gráfico de aptitudes** y haga clic en la aptitud #1.
1. Navegue por la **configuración de las aptitudes** y busque "outputs".
1. Abra el evaluador de expresiones **</>** para la entidad "organizations".

> [!div class="mx-imgBorder"]
> ![Salida para la entidad "organizations"](media/cognitive-search-debug/skill-output-detail-missing-value-locations-organizations.png)

Al evaluar el resultado de la expresión, se obtiene el resultado correcto. La aptitud trabaja para identificar el valor correcto de la entidad "organizations". Sin embargo, la asignación de salida en la ruta de acceso de la entidad sigue produciendo un error. Al comparar la ruta de acceso de salida de la aptitud con la ruta de acceso de salida del error, la aptitud es el objeto primario en una relación jerárquica de las salidas, organizaciones y ubicaciones en el nodo /document/content. Sin embargo, la asignación de campos de salida espera los resultados en el nodo /document/merged_content. En el paso anterior, la entrada cambió de "/document/content" a "/document/merged_content". El contexto de la configuración de aptitudes debe cambiarse para asegurarse de que la salida se genera con el contexto correcto.

1. Seleccione la pestaña **Enriquecimientos con IA**.
1. Seleccione el **gráfico de aptitudes** y haga clic en la aptitud #1.
1. Navegue por la **configuración de las aptitudes** y busque "context".
1. Haga doble clic en el valor de "context" y cámbielo a "/document/merged_content".
1. Haga clic en **Guardar** en el panel derecho de detalles de la aptitud.
1. Haga clic en **Ejecutar** en el menú de la ventana de sesiones. Esto iniciará otra ejecución del conjunto de aptitudes con el documento.

> [!div class="mx-imgBorder"]
> ![Corrección de contexto en la configuración de aptitudes](media/cognitive-search-debug/skill-setting-context-correction-missing-value-locations-organizations.png)

Todos los errores se han resuelto.

## <a name="commit-changes-to-the-skillset"></a>Confirmación de los cambios en el conjunto de aptitudes

Cuando se inició la sesión de depuración, el servicio de búsqueda creó una copia del conjunto de aptitudes. Esto se hizo para que los cambios realizados no afectaran al sistema de producción. Ahora que ha finalizado la depuración del conjunto de aptitudes, las correcciones se pueden confirmar (sobrescribir el conjunto de aptitudes original) en el sistema de producción. Si desea seguir realizando cambios en el conjunto de aptitudes sin que afecten al sistema de producción, la sesión de depuración se puede guardar y volver a abrir posteriormente.

1. Haga clic en **Confirmar cambios** en el menú principal de sesiones de depuración.
1. Haga clic en **Aceptar** para confirmar que desea actualizar el conjunto de aptitudes.
1. Cierre la sesión de depuración y seleccione la pestaña **Indizadores**.
1. Abra 'clinical-trials-idxr'.
1. Haga clic en **Restablecer**.
1. Haga clic en **Ejecutar**. Haga clic en **ACEPTAR** para continuar.

Cuando el indexador haya terminado de ejecutarse, debe mostrarse una marca de verificación verde y la palabra Correcto junto a la marca de tiempo de la última ejecución en la pestaña Historial de ejecución. Para asegurarse de que se han aplicado los cambios:

1. Salga de **Indizador** y seleccione la pestaña **Índice**.
1. Abra el índice "clinical-trials" y, en la pestaña Explorador de búsqueda, haga clic en **Buscar**.
1. La ventana de resultados debe mostrar que las entidades de organizaciones y ubicaciones ahora contienen los valores esperados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los conjuntos de aptitudes](https://docs.microsoft.com/azure/search/cognitive-search-working-with-skillsets)
> [Más información sobre el enriquecimiento incremental y el almacenamiento en caché](https://docs.microsoft.com/azure/search/cognitive-search-incremental-indexing-conceptual)