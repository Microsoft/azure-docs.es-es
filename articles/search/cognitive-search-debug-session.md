---
title: Conceptos de las sesiones de depuración (versión preliminar)
titleSuffix: Azure Cognitive Search
description: Las sesiones de depuración, a las que se accede desde Azure Portal, proporcionan un entorno similar a un IDE en el que se pueden identificar y corregir errores, validar cambios y enviar cambios en conjuntos de aptitudes de la canalización de enriquecimiento con IA. Sesiones de depuración está en versión preliminar.
manager: nitinme
author: tchristiani
ms.author: terrychr
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/19/2020
ms.openlocfilehash: b428ff4a9a4f4b9110e88dba57cfad20f2221cf1
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936170"
---
# <a name="debug-sessions-in-azure-cognitive-search"></a>Sesiones de depuración en Azure Cognitive Search

Sesiones de depuración es un editor visual que funciona con un conjunto de aptitudes existente en Azure Portal. En una sesión de depuración, puede identificar y resolver errores, validar cambios e introducir cambios en un conjunto de aptitudes en la canalización de enriquecimiento con IA.

> [!Important]
> Las sesiones de depuración constituyen una característica en versión preliminar que se ofrece sin Acuerdo de Nivel de Servicio, por lo que no se recomienda usarlas con cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="using-debug-sessions"></a>Uso de sesiones de depuración

Cuando se inicia una sesión, el servicio crea una copia del conjunto de aptitudes, el indizado y el índice donde se usa un único documento para probar el conjunto. Los cambios realizados en la sesión se guardan en la sesión. Los cambios realizados en la sesión de depuración no afectan al conjunto de aptitudes a menos que los cambios se confirmen en él. Al confirmar los cambios, se sobrescribirá el conjunto de aptitudes de producción.

Durante una sesión de depuración, puede editar un conjunto de aptitudes, inspeccionar y validar cada nodo del árbol de enriquecimiento en el contexto de un documento específico. Una vez que se resuelven los problemas de la canalización de enriquecimiento, se pueden guardar los cambios en la sesión y se pueden confirmar en el conjunto de aptitudes en producción. 

Si la canalización de enriquecimiento no tiene errores, se puede usar una sesión de depuración para enriquecer incrementalmente un documento, y probar y validar cada cambio antes de confirmarlo.

## <a name="creating-a-debug-session"></a>Creación de una sesión de depuración

Para iniciar una sesión de depuración, debe tener una canalización de enriquecimiento con IA que incluya un origen de datos, un conjunto de aptitudes, un indizador y un índice. Para configurar una sesión de depuración, debe asignarle un nombre y proporcionar una cuenta de almacenamiento de uso general que se usará para almacenar en caché las ejecuciones de las aptitudes al ejecutar el indizador. También tendrá que seleccionar el indizador que se ejecutará. El indizador tiene referencias almacenadas en el origen de datos, el conjunto de aptitudes y el índice. La sesión de depuración se establecerá de forma predeterminada en el primer documento del origen de datos. También puede especificar un documento para iniciar el proceso.

> [!div class="mx-imgBorder"]
> ![Creación de una sesión de depuración](media/cognitive-search-debug/debug-session-new.png)

## <a name="debug-session-features"></a>Características de las sesiones de depuración

Para empezar la sesión de depuración, se ejecuta el conjunto de aptitudes en el documento seleccionado. La sesión de depuración registrará los metadatos adicionales asociados con cada operación en el conjunto de aptitudes. Los metadatos creados por las ejecuciones de aptitudes de la canalización informan del siguiente conjunto de características, que, a continuación, se usan para ayudar a identificar y corregir problemas con el conjunto de aptitudes.

## <a name="ai-enrichments"></a>Enriquecimientos con IA

A medida que se ejecutan las aptitudes, se genera un árbol de enriquecimientos que representa el documento. El uso de un árbol para visualizar las salidas de las aptitudes o enriquecimientos proporciona una visión completa de todos los enriquecimientos realizados. Puede buscar en todo el documento e inspeccionar cada nodo del árbol de enriquecimiento. Esta perspectiva facilita la acción de dar forma a los objetos. Este formato también proporciona indicaciones visuales sobre el tipo, la ruta de acceso y el contenido de cada nodo del árbol.

## <a name="skill-graph"></a>Gráfico de aptitudes

La vista del **gráfico de aptitudes** proporciona una representación visual jerárquica del conjunto de aptitudes. El gráfico es una representación descendente del orden en que se ejecutan las aptitudes. Las aptitudes que dependen de la salida de otros conocimientos se muestran en un nivel inferior del gráfico. Las aptitudes del mismo nivel en la jerarquía se pueden ejecutar en paralelo. 

Al seleccionar una aptitud en el gráfico, se resaltarán los conocimientos conectados a ella, los nodos que crean sus entradas y los nodos que aceptan sus salidas. Cada nodo de aptitud muestra su tipo, errores o advertencias y recuentos de ejecución. En el **gráfico de aptitudes** puede seleccionar la aptitud que se depurará o mejorará. Al seleccionar una aptitud, sus detalles se mostrarán en el panel de detalles de la aptitud, a la derecha del gráfico.

> [!div class="mx-imgBorder"]
> ![Gráfico de aptitudes](media/cognitive-search-debug/skills-graph.png)

## <a name="skill-details"></a>Detalles de aptitud

En el panel de detalles de la aptitud se muestra un conjunto de áreas para trabajar con una aptitud específica cuando esa aptitud está resaltada en el **gráfico de aptitudes**. Puede revisar y editar los detalles de la configuración de la aptitud. Se proporciona la definición JSON de la aptitud. También se muestran los detalles de la ejecución de la aptitud y los errores y advertencias. La pestaña**Configuración de las aptitudes** y la **Aptitud relativa al editor de JSON** permiten la edición directa de la aptitud. [`</>`](#expression-evaluator) abre una ventana para ver y editar las expresiones de las entradas y salidas de las aptitudes.

Los controles de entrada anidados en la ventana de configuración de aptitudes se pueden usar para crear formas complejas para proyecciones, asignaciones de campos de salida para un campo de tipo complejo o una entrada a una aptitud. Cuando se usa con el **Evaluador de expresiones**, las entradas anidadas proporcionan una prueba sencilla y validan el generador de expresiones.

## <a name="skill-execution-history"></a>Historial de ejecución de aptitudes

Una aptitud se puede ejecutar varias veces en un conjunto de aptitudes para un único documento. Por ejemplo, la aptitud de OCR se ejecutará una vez para cada imagen extraída de cada documento. En el panel de detalles de la aptitud, la pestaña **Ejecuciones** muestra el historial de ejecución de la aptitud, lo que proporciona una mirada más profunda a cada invocación de la aptitud. 

El historial de ejecución permite realizar el seguimiento de un enriquecimiento específico hasta la habilidad que lo generó. Al hacer clic en una entrada de aptitud, se navega a la habilidad que generó la entrada. Esto permite la identificación de la causa principal de un problema que puede manifestarse en una aptitud de bajada. 

Cuando se identifica un posible problema, el historial de ejecución muestra vínculos a los conocimientos que generaron las entradas específicas y proporciona una característica similar al seguimiento de la pila. Al hacer clic en la aptitud asociada con una entrada, navega a la aptitud para corregir los errores o continúa para hacer un seguimiento del problema específico.

Al crear una aptitud personalizada o depurar un error con una aptitud personalizada, existe la opción de generar una solicitud de invocación de aptitud en el historial de ejecución.

## <a name="enriched-data-structure"></a>Estructura de datos enriquecidos

En el panel **Estructura de datos enriquecidos** se muestran los enriquecimientos del documento mediante el conjunto de aptitudes y se detalla el contexto de cada enriquecimiento, así como la aptitud de origen. También se puede usar el **evaluador de expresiones** para ver el contenido de cada enriquecimiento.

> [!div class="mx-imgBorder"]
> ![Estructura de datos enriquecidos](media/cognitive-search-debug/enriched-data-structure-display.png)

## <a name="expression-evaluator"></a>Evaluador de expresiones

El **evaluador de expresiones** ofrece un vistazo rápido al valor de cualquier ruta de acceso. Permite editar la ruta de acceso y probar los resultados antes de actualizar cualquiera de las entradas o el contexto de una aptitud o proyección.

## <a name="errorswarnings"></a>Errores o advertencias

Esta ventana muestra todos los errores y advertencias que genera el conjunto de aptitudes al ejecutarse para el documento en la sesión de depuración.

## <a name="limitations"></a>Limitaciones

Las sesiones de depuración funcionan con todos los orígenes de datos disponibles en general, así como con la mayoría de los orígenes de datos de versión preliminar. La API de MongoDB (versión preliminar) y Cassandra API (versión preliminar) de Cosmos DB no se admiten actualmente.

## <a name="next-steps"></a>Pasos siguientes

Ahora que comprende los elementos de las sesiones de depuración, pruebe el tutorial para obtener una experiencia práctica.

> [!div class="nextstepaction"]
> [Tutorial para explorar la característica de sesiones de depuración](./cognitive-search-tutorial-debug-sessions.md)