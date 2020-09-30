---
title: IntelliSense en las herramientas de Azure Stream Analytics para Visual Studio Code
description: En este artículo se describe cómo usar las características de IntelliSense en las herramientas de Azure Stream Analytics para Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 4/11/2020
ms.topic: how-to
ms.openlocfilehash: bd72866e3e3db21691fda6aaec864a76af7414a3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903730"
---
# <a name="intellisense-in-azure-stream-analytics-tools-for-visual-studio-code"></a>IntelliSense en las herramientas de Azure Stream Analytics para Visual Studio Code

IntelliSense está disponible para el [lenguaje de consulta de Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference?toc=https%3A%2F%2Fdocs.microsoft.com%2Fazure%2Fstream-analytics%2Ftoc.json&bc=https%3A%2F%2Fdocs.microsoft.com%2Fazure%2Fbread%2Ftoc.json) en las [herramientas de Azure Stream Analytics para VS Code](https://marketplace.visualstudio.com/items?itemName=ms-bigdatatools.vscode-asa&ssr=false#overview). IntelliSense es una ayuda para la finalización de código que incluye varias características: enumerar miembros, información de parámetros, información rápida y completar palabra. A veces las características de IntelliSense reciben otros nombres, como "finalización de código", "asistente de contenido" y "sugerencias de código".

![Demostración de IntelliSense](./media/vs-code-intellisense/intellisense.gif)

## <a name="intellisense-features"></a>Características de IntelliSense

Las características de IntelliSense en las herramientas de Stream Analytics para VS Code tienen la tecnología de un servicio de lenguaje. Un servicio de lenguaje analiza el código fuente y proporciona finalizaciones de código inteligentes basadas en la semántica del lenguaje. Si un servicio de lenguaje conoce las posibles finalizaciones, aparecen sugerencias de IntelliSense mientras se escribe. Si se sigue escribiendo, se establece un filtro en una lista de miembros, como variables y métodos, con el fin de que solo incluya los miembros que contengan los caracteres que se hayan escrito. Al presionar las teclas `Tab` o `Enter`, IntelliSense inserta el miembro que se ha seleccionado.

IntelliSense se puede desencadenar en cualquier ventana del editor escribiendo un carácter de desencadenador, como el carácter de punto `.`.

![autocompletar intellisense](./media/vs-code-intellisense/auto-completion.gif)

> [!TIP]
> El widget de sugerencias admite el filtrado de CamelCase. Para limitar las sugerencias, puede escribir las letras, que están en mayúsculas en un nombre de método. Por ejemplo, "cra" hará que aparezca rápidamente "crearAplicación".

### <a name="types-of-completions"></a>Tipos de finalizaciones

IntelliSense en las herramientas de Stream Analytics para VS Code ofrece distintos tipos de finalizaciones, como sugerencias de servidores de lenguaje, fragmentos de código y finalizaciones de texto simples basadas en palabras.

|Completion     |  Tipo       |
| ----- | ------- |
| Palabras clave | `keyword`
| Functions | `build-in function`, `user defined function`  |
| Nombre de conjunto de datos| `input`, `output`, `intermediate result set`|
| Nombre de columna de conjunto de datos|`input`, `intermediate result set`|

#### <a name="name-completion"></a>Finalización de nombres

Además de la finalización automática de palabras clave, las herramientas de Stream Analytics para VS Code leen la lista de nombres de entrada y salida del trabajo, así como los nombres de las columnas de los orígenes de datos cuando están configurados. La extensión recuerda esta información para proporcionar funcionalidades de finalización de nombres que son útiles para escribir instrucciones presionando pocas teclas:

Durante la codificación, no es preciso salir del editor para realizar búsquedas en los nombres de entrada del trabajo, en el nombre de salida y en los nombres de columna. Puede conservar el contexto, buscar la información necesaria, insertar elementos directamente en el código y hacer que IntelliSense termine de escribir por usted.

Tenga en cuenta que debe configurar la entrada local o la entrada en directo, y guardar el archivo de configuración para poder usar la finalización de nombres.

![finalización de nombres](./media/vs-code-intellisense/name-completion.gif)

### <a name="parameter-info"></a>Información de parámetros

La **información de parámetros** de IntelliSense abre una lista de parámetros que proporciona información sobre el número, los nombres y los tipos de parámetros que necesarios para una función. El parámetro en negrita indica el siguiente parámetro que se necesita al escribir una función.

La lista de parámetros también aparece para las funciones anidadas. Si se escribe una función como parámetro de otra función, la lista de parámetros muestra los parámetros de la función interna. Así, cuando la lista de parámetros de la función interna está completa, pasa a mostrar los parámetros de la función externa.

![información de parámetros](./media/vs-code-intellisense/parameter-info.gif)

### <a name="quick-info"></a>Información rápida

Tal y como lo proporciona el servicio de lenguaje, puede ver la **información rápida** de cada identificador en el código. Algunos ejemplos de identificadores son la entrada, la salida, un conjunto de resultados intermedio o una función. Al mover el puntero del mouse sobre un identificador, su declaración se muestra en una ventana emergente. Se muestran las propiedades y los esquemas de datos de las entradas, si están configuradas, así como el conjunto de datos intermedio.

![información rápida](./media/vs-code-intellisense/quick-info.gif)

## <a name="troubleshoot-intellisense"></a>Solución de problemas de IntelliSense

Este problema se debe a que falta la configuración de entrada que proporciona datos. Puede comprobar si se ha configurado correctamente una [entrada local](visual-studio-code-local-run.md#define-a-local-input) o una [entrada en directo](visual-studio-code-local-run-live-input.md#define-a-live-stream-input).

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-visual-studio-code.md)
* [Prueba de consultas de Stream Analytics localmente con datos de ejemplo mediante Visual Studio Code](visual-studio-code-local-run.md)
* [Prueba de las consultas de Stream Analytics localmente en una entrada de streaming en vivo con Visual Studio Code](visual-studio-code-local-run-live-input.md)
