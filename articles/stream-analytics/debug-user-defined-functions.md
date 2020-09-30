---
title: Depuración de funciones definidas por el usuario en Azure Stream Analytics
description: En este artículo se describe cómo depurar funciones definidas por el usuario en Azure Stream Analytics.
author: jenssuessmeyer
ms.author: jenss
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/24/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 25be55021e4f07b0803d8b7cc1bb48ab05a3ea3b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90889606"
---
# <a name="debug-user-defined-functions-in-azure-stream-analytics"></a>Depuración de funciones definidas por el usuario en Azure Stream Analytics 

Cuando las funciones definidas por el usuario (UDF) no funcionan como se espera, es necesario depurarlas para encontrar el problema. Puede depurar las UDF de los trabajos de Stream Analytics al ejecutar los trabajos localmente mediante [Visual Studio Code](visual-studio-code-local-run-live-input.md) o [Visual Studio](stream-analytics-vs-tools-local-run.md).

Cuando se ejecuta el trabajo en un flujo de entrada activo de forma local, solo se imita la ejecución del motor de Azure Stream Analytics en la nube en un nodo. Las pruebas locales de datos activos no pueden reemplazar las pruebas de rendimiento y escalabilidad que se realizan en la nube, pero puede ahorrar tiempo durante las pruebas funcionales al no tener que enviar el trabajo a la nube cada vez que quiera probarlo. Además, las directivas de tiempo se deshabilitan para la ejecución local con datos de ejemplo o locales, pero se admiten para las pruebas de datos activos.

## <a name="pick-your-language"></a>Selección del lenguaje

Puede escribir UDF para Azure Stream Analytics con .NET (C#) o JavaScript. 

### <a name="functions-in-c"></a>Funciones en C# 

Al [escribir UDF de .NET con Visual Studio](stream-analytics-edge-csharp-udf-methods.md), obtendrá el mismo nivel de compatibilidad que para cualquier proyecto de clase de .NET. Esta compatibilidad incluye:

* Compatibilidad con la compilación, como comprobación de sintaxis y compatibilidad con el compilador.

* Capacidad de agregar y compilar artefactos y proyectos de C# y hacer referencia a estos en la solución de Stream Analytics. 

* Reutilización sencilla del código encapsulado en un proyecto que se puede compartir. 

* Compatibilidad con la depuración directamente en Visual Studio. Establezca el proyecto de Stream Analytics como proyecto de inicio y establezca puntos de interrupción en código de C#. Luego, presione **F5** para depurar el código de C# como haría con cualquier otro proyecto de C#. 

### <a name="functions-in-javascript"></a>Funciones en JavaScript

JavaScript es otra opción para crear funciones en Stream Analytics. El código de JavaScript se coloca directamente en el área de función del proyecto de Stream Analytics, lo que dificulta el uso compartido entre los proyectos.

La compilación se produce cuando se compila o se ejecuta el proyecto de Stream Analytics. La posibilidad de detectar un problema solo en tiempo de ejecución es mayor. No hay compatibilidad con la depuración de las funciones de JavaScript directamente en Stream Analytics.

## <a name="debug-options-for-javascript"></a>Opciones de depuración de JavaScript

Dado que no hay compatibilidad con la depuración de las funciones de JavaScript directamente en Stream Analytics, puede realizar la depuración si encapsula la función en un sitio HTML y obtiene la salida desde allí.

En el ejemplo siguiente se muestra cómo se depuran UDF de JavaScript con algunas limitaciones en un entorno de ejecución integrado en [Visual Studio Code](quick-create-visual-studio-code.md).

### <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que el proyecto de Azure Stream Analytics tenga los siguientes elementos:

* Una entrada 
* Una salida 
* Una consulta de Stream Analytics (.asaql) 
* Una configuración de trabajos de Stream Analytics (JobConfig.json)
* Una UDF de JavaScript

### <a name="prepare-files"></a>Preparación de los archivos

En la imagen siguiente, el archivo de consulta *.asaql* incluye solo la llamada a la UDF (*fxcharCount*). Este cambio garantiza que siga pudiendo compilar el proyecto después de que se realicen cambios.

> [!div class="mx-imgBorder"]
> ![Archivo de consulta de prueba de Stream Analytics](./media/debug-user-defined-functions/asaql-file.png)

Cree una carpeta adicional en **Tests** (Prueba) para hospedar el archivo de prueba, al que se llama para ejecutar la prueba con la función de JavaScript. En este ejemplo, el nombre de la carpeta es *fxcharCount* y el nombre de la prueba es *Test_UDF.js*. 

En la imagen siguiente se muestra el código del archivo de prueba, que carga el archivo de función y ejecuta la función. Este ejemplo es sencillo, pero podría cargar archivos de datos de prueba adicionales y recorrer en bucle otras pruebas para obtener la salida. La notación de la llamada de función es ligeramente diferente de las llamadas comunes, ya que se hace referencia al archivo y no se carga en el entorno de ejecución, lo que hace posible la depuración. 

> [!div class="mx-imgBorder"]
> ![Archivo de prueba de Stream Analytics](./media/debug-user-defined-functions/test-file.png)

En la función, agregue las siguientes líneas de código al archivo para exponer los métodos. No afectan a la capacidad de compilar el código en Visual Studio Code.

```javascript
var methods = {};
methods.fxchartCount = fxchartCount;
exports.data = methods;
``` 

> [!div class="mx-imgBorder"]
> ![UDF de JavaScript de Stream Analytics](./media/debug-user-defined-functions/udf-file.png)
  
## <a name="install-debug-support"></a>Instalación de compatibilidad con la depuración

Para depurar, debe [descargar](https://nodejs.org/en/download/) e instalar **node.js**. Instale la versión correcta en función de la plataforma que use. Después de instalar el entorno de ejecución de node.js, reinicie Visual Studio Code para implementar los cambios. 

Seleccione **Run and Debug** (Ejecutar y depurar) o presione **CTRL+Mayús+D** para iniciar la depuración. Aparece un cuadro combinado en el que puede seleccionar **node.js** como entorno de ejecución. Si solo tiene instalado node.js, se usa de forma predeterminada. Si es necesario, debería poder avanzar paso a paso por el código hasta el archivo satélite con F11. 

> [!div class="mx-imgBorder"]
> ![Ejecución y depuración de la UDF en Stream Analytics](./media/debug-user-defined-functions/run-debug-udf.png)

### <a name="debug-user-defined-aggregates"></a>Depuración de agregados definidos por el usuario 

Puede usar el método de depuración de las UDF de JavaScript para depurar agregados definidos por el usuario (UDA). En este ejemplo, se agrega un UDA al archivo de consulta *.asaql* y al archivo de prueba.

Al igual que con la UDF, debe incluir una llamada al UDA para asegurarse de que el proyecto se compile una vez que se hayan realizado los cambios. 

> [!div class="mx-imgBorder"]
> ![Adición de un UDA a asaql](./media/debug-user-defined-functions/asaql-uda.png)

En el archivo *Test_UDA.js*, haga referencia al archivo de UDA como hizo con la UDF. Además, llame a `main()`, `init()` y `accumulate()`. Se llama al método `accumulate()` en un bucle para colocar los valores en la pila de estado. Se llama al método `computeresult()` para crear la consulta final. 

> [!div class="mx-imgBorder"]
> ![Archivo de prueba de UDA](./media/debug-user-defined-functions/uda-test.png)

Como en el ejemplo de UDF, es necesario agregar código al UDA para exponer los métodos pertinentes.

```javascript
var methods = {};
methods.main = main;
methods.init = main.init;
methods.accumulate = main.accumulate;
methods.computeResult = main.computeResult;
exports.data = methods;
``` 

> [!div class="mx-imgBorder"]
> ![Código agregado al UDA](./media/debug-user-defined-functions/uda-expose-methods.png)

Seleccione **Run and Debug** (Ejecutar y depurar) o presione **CTRL+Mayús+D** para iniciar la depuración. Aparece un cuadro combinado en el que puede seleccionar **node.js** como entorno de ejecución. Si solo tiene instalado node.js, se usa de forma predeterminada. Si es necesario, debería poder avanzar paso a paso por el código hasta el archivo satélite con F11.

> [!div class="mx-imgBorder"]
> ![Ejecución y depuración del UDA en Stream Analytics](./media/debug-user-defined-functions/run-debug-uda.png)


## <a name="next-steps"></a>Pasos siguientes

* [Desarrollo y depuración local de trabajos de Azure Stream Analytics](develop-locally.md)
* [Depuración de las consultas de Azure Stream Analytics localmente mediante un diagrama de trabajos de Visual Studio](debug-locally-using-job-diagram.md)
* [Funciones definidas por el usuario de Azure Stream Analytics](functions-overview.md)
 
