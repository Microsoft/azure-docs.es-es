---
title: Prueba de Azure Functions
description: Creación de pruebas automatizadas para una función de C# en Visual Studio y una función de JavaScript en VS Code
author: craigshoemaker
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: cshoe
ms.openlocfilehash: be406744dc9752dd3c924c636e85f43e133c1cca
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085085"
---
# <a name="strategies-for-testing-your-code-in-azure-functions"></a>Estrategias para probar el código en Azure Functions

En este artículo se muestra cómo crear pruebas automatizadas para Azure Functions.

Se recomienda probar todo el código, sin embargo, es posible que obtenga los mejores resultados encapsulando una lógica de función y creando pruebas fuera de la función. La abstracción lógica siempre limita las líneas de código de una función y permite esta sea el único responsable de llamar a otras clases o módulos. Pero en este artículo, se muestra cómo crear pruebas automatizadas mediante funciones HTTP o desencadenadas por temporizador.

El contenido siguiente se divide en dos secciones distintas diseñadas para diferentes lenguajes y entornos de destino. Puede aprender a compilar pruebas en:

- [C# en Visual Studio con xUnit](#c-in-visual-studio)
- [JavaScript en VS Code con Jest](#javascript-in-vs-code)

El repositorio de ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/azure-functions-tests).

## <a name="c-in-visual-studio"></a>C# en Visual Studio

El ejemplo siguiente describe cómo crear una aplicación de función C# en Visual Studio y cómo ejecutar pruebas con [xUnit](https://xunit.github.io).

![Prueba de Azure Functions con C# en Visual Studio](./media/functions-test-a-function/azure-functions-test-visual-studio-xunit.png)

### <a name="setup"></a>Configurar

Para configurar el entorno, cree una función y pruebe la aplicación. Los pasos siguientes le ayudarán a crear las aplicaciones y funciones necesarias para admitir las pruebas:

1. [Cree una nueva aplicación de Functions](./functions-create-first-azure-function.md) y asígnele el nombre **Functions**.
2. [Cree una función HTTP a partir de la plantilla ](./functions-create-first-azure-function.md) y asígnele el nombre **MyHttpTrigger**.
3. [Cree una función de temporizador a partir de la plantilla](./functions-create-scheduled-function.md) y asígnele el nombre **MyTimerTrigger**.
4. [Cree una aplicación de prueba en xUnit](https://xunit.github.io/docs/getting-started-dotnet-core) en la solución y asígnele el nombre **Functions.Test**.
5. Use NuGet para agregar referencias desde la aplicación de prueba a [Microsoft.AspNetCore.Mvc](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc/)
6. [Haga referencia a la aplicación *Functions*](/visualstudio/ide/managing-references-in-a-project?view=vs-2017) desde la aplicación *Functions.Tests*.

### <a name="create-test-classes"></a>Crear clases de prueba

Ahora que se han creado los proyectos, puede crear las clases usadas para ejecutar las pruebas automatizadas.

Cada función toma una instancia de [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger) para controlar el registro de mensajes. Algunas pruebas no registran los mensajes o no tiene que preocuparse por cómo se implementa el registro. Otras pruebas deben evaluar los mensajes registrados para determinar si están pasando una prueba.

Creará una nueva clase denominada `ListLogger` que contiene una lista interna de los mensajes que se evaluarán durante las pruebas. Para implementar la interfaz de `ILogger` necesaria, la clase necesita un ámbito. La siguiente clase simula un ámbito para los casos de prueba que se van a pasar a la clase `ListLogger`.

Cree una nueva clase en el proyecto *Functions.Tests* denominada **NullScope.cs** y escriba el código siguiente:

```csharp
using System;

namespace Functions.Tests
{
    public class NullScope : IDisposable
    {
        public static NullScope Instance { get; } = new NullScope();

        private NullScope() { }

        public void Dispose() { }
    }
}
```

A continuación, cree una nueva clase en el proyecto *Functions.Tests* denominada **ListLogger.cs** y escriba el código siguiente:

```csharp
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Text;

namespace Functions.Tests
{
    public class ListLogger : ILogger
    {
        public IList<string> Logs;

        public IDisposable BeginScope<TState>(TState state) => NullScope.Instance;

        public bool IsEnabled(LogLevel logLevel) => false;

        public ListLogger()
        {
            this.Logs = new List<string>();
        }

        public void Log<TState>(LogLevel logLevel,
                                EventId eventId,
                                TState state,
                                Exception exception,
                                Func<TState, Exception, string> formatter)
        {
            string message = formatter(state, exception);
            this.Logs.Add(message);
        }
    }
}
```

La clase `ListLogger` implementa los siguientes miembros según los contrata la interfaz `ILogger`:

- **BeginScope**: Los ámbitos agregan contexto al registro. En este caso, la prueba simplemente apunta a la instancia estática en la clase `NullScope` para permitir la prueba de la función.

- **IsEnabled**: Se proporciona un valor predeterminado de `false`.

- **Log**: Este método usa la función `formatter` proporcionada para dar formato al mensaje y, a continuación, agrega el texto resultante a la colección `Logs`.

La colección `Logs` es una instancia de `List<string>` y se inicializa en el constructor.

A continuación, cree un archivo nuevo en el proyecto *Functions.Tests* denominado **LoggerTypes.cs** y escriba el código siguiente:

```csharp
namespace Functions.Tests
{
    public enum LoggerTypes
    {
        Null,
        List
    }
}
```

Esta enumeración especifica el tipo de registrador utilizado por las pruebas.

A continuación, cree una clase nueva en el proyecto *Functions.Tests* denominada **TestFactory.cs** y escriba el código siguiente:

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Http.Internal;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Logging.Abstractions;
using Microsoft.Extensions.Primitives;
using System.Collections.Generic;

namespace Functions.Tests
{
    public class TestFactory
    {
        public static IEnumerable<object[]> Data()
        {
            return new List<object[]>
            {
                new object[] { "name", "Bill" },
                new object[] { "name", "Paul" },
                new object[] { "name", "Steve" }

            };
        }

        private static Dictionary<string, StringValues> CreateDictionary(string key, string value)
        {
            var qs = new Dictionary<string, StringValues>
            {
                { key, value }
            };
            return qs;
        }

        public static HttpRequest CreateHttpRequest(string queryStringKey, string queryStringValue)
        {
            var context = new DefaultHttpContext();
            var request = context.Request;
            request.Query = new QueryCollection(CreateDictionary(queryStringKey, queryStringValue));
            return request;
        }

        public static ILogger CreateLogger(LoggerTypes type = LoggerTypes.Null)
        {
            ILogger logger;

            if (type == LoggerTypes.List)
            {
                logger = new ListLogger();
            }
            else
            {
                logger = NullLoggerFactory.Instance.CreateLogger("Null Logger");
            }

            return logger;
        }
    }
}
```

La clase `TestFactory` implementa los siguientes miembros:

- **Data**: Esta propiedad devuelve una colección [IEnumerable](/dotnet/api/system.collections.ienumerable) de datos de ejemplo. Los pares clave-valor representan los valores que se pasan en una cadena de consulta.

- **CreateDictionary**: Este método acepta un par clave-valor como argumento y devuelve un `Dictionary` nuevo usado para crear `QueryCollection` para representar los valores de cadena de consulta.

- **CreateHttpRequest**: Este método crea una solicitud HTTP que se inicializa con los parámetros de cadena de consulta indicados.

- **CreateLogger**: Según el tipo de registrador, este método devuelve una clase de registrador que se usa para las pruebas. `ListLogger` realiza un seguimiento de los mensajes registrados disponibles para su evaluación en las pruebas.

Finalmente, cree una clase nueva en el proyecto *Functions.Tests* denominada **FunctionsTests.cs** y escriba el código siguiente:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using Xunit;

namespace Functions.Tests
{
    public class FunctionsTests
    {
        private readonly ILogger logger = TestFactory.CreateLogger();

        [Fact]
        public async void Http_trigger_should_return_known_string()
        {
            var request = TestFactory.CreateHttpRequest("name", "Bill");
            var response = (OkObjectResult)await MyHttpTrigger.Run(request, logger);
            Assert.Equal("Hello, Bill. This HTTP triggered function executed successfully.", response.Value);
        }

        [Theory]
        [MemberData(nameof(TestFactory.Data), MemberType = typeof(TestFactory))]
        public async void Http_trigger_should_return_known_string_from_member_data(string queryStringKey, string queryStringValue)
        {
            var request = TestFactory.CreateHttpRequest(queryStringKey, queryStringValue);
            var response = (OkObjectResult)await MyHttpTrigger.Run(request, logger);
            Assert.Equal($"Hello, {queryStringValue}. This HTTP triggered function executed successfully.", response.Value);
        }

        [Fact]
        public void Timer_should_log_message()
        {
            var logger = (ListLogger)TestFactory.CreateLogger(LoggerTypes.List);
            MyTimerTrigger.Run(null, logger);
            var msg = logger.Logs[0];
            Assert.Contains("C# Timer trigger function executed at", msg);
        }
    }
}
```

Los miembros implementados en esta clase son:

- **Http_trigger_should_return_known_string**: Esta prueba crea una solicitud con los valores de la cadena de consulta de `name=Bill` en una función HTTP y comprueba que se devuelve la respuesta esperada.

- **Http_trigger_should_return_string_from_member_data**: Esta prueba usa atributos de xUnit para proporcionar datos de ejemplo a la función HTTP.

- **Timer_should_log_message**: Esta prueba crea una instancia de `ListLogger` y la pasa a una función de temporizador. Una vez que se ejecuta la función, se comprueba el registro para garantizar que esté presente el mensaje esperado.

Si quiere obtener acceso a la configuración de la aplicación en las pruebas, puede usar [System.Environment.GetEnvironmentVariable](./functions-dotnet-class-library.md#environment-variables).

### <a name="run-tests"></a>Ejecución de las pruebas

Para ejecutar las pruebas, vaya al **Explorador de pruebas** y haga clic en **Ejecutar todas**.

![Prueba de Azure Functions con C# en Visual Studio](./media/functions-test-a-function/azure-functions-test-visual-studio-xunit.png)

### <a name="debug-tests"></a>Depuración de pruebas

Para depurar las pruebas, establezca un punto de interrupción en una prueba, vaya al **Explorador de pruebas** y haga clic en **Ejecutar > Última ejecución de depuración**.

## <a name="javascript-in-vs-code"></a>JavaScript en VS Code

El ejemplo siguiente describe cómo crear una aplicación de función de JavaScript en VS Code y cómo ejecutar pruebas con [Jest](https://jestjs.io). Este procedimiento utiliza la [extensión Functions de VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) para crear funciones de Azure Functions.

![Prueba de Azure Functions con JavaScript en VS Code](./media/functions-test-a-function/azure-functions-test-vs-code-jest.png)

### <a name="setup"></a>Configurar

Para configurar el entorno, inicialice una nueva aplicación de Node.js en una carpeta vacía mediante la ejecución de `npm init`.

```bash
npm init -y
```

A continuación, instale Jest ejecutando el siguiente comando:

```bash
npm i jest
```

Ahora actualice _package.json_ para reemplazar el comando de prueba existente por el siguiente comando:

```bash
"scripts": {
    "test": "jest"
}
```

### <a name="create-test-modules"></a>Creación de módulos de prueba

Con el proyecto inicializado, puede crear los módulos usados para ejecutar las pruebas automatizadas. Empiece creando una nueva carpeta denominada *pruebas* para mantener la compatibilidad con los módulos.

En la carpeta *pruebas*, agregue un nuevo archivo, asígnele el nombre **defaultContext.js** y agregue el código siguiente:

```javascript
module.exports = {
    log: jest.fn()
};
```

Este módulo simula la función *log* para representar el contexto de ejecución predeterminado.

A continuación, agregue un nuevo archivo, asígnele el nombre **defaultTimer.js** y agregue el código siguiente:

```javascript
module.exports = {
    IsPastDue: false
};
```

Este módulo implementa la propiedad `IsPastDue` para que permanezca como una instancia de temporizador falsa. Aquí no se requieren configuraciones de temporizador, como expresiones NCRONTAB, ya que la herramienta de ejecución de pruebas llama a la función directamente para probar el resultado.

A continuación, utilice la extensión Functions de VS Code para [crear una nueva función HTTP de JavaScript](/azure/developer/javascript/tutorial-vscode-serverless-node-01) y asígnele el nombre *HttpTrigger*. Una vez creada la función, agregue un nuevo archivo en la misma carpeta denominado **index.test.js** y agregue el código siguiente:

```javascript
const httpFunction = require('./index');
const context = require('../testing/defaultContext')

test('Http trigger should return known text', async () => {

    const request = {
        query: { name: 'Bill' }
    };

    await httpFunction(context, request);

    expect(context.log.mock.calls.length).toBe(1);
    expect(context.res.body).toEqual('Hello Bill');
});
```

La función HTTP de la plantilla devuelve una cadena de "Hello" concatenada con el nombre proporcionado en la cadena de consulta. Esta prueba crea una instancia falsa de una solicitud y la pasa a la función HTTP. La prueba comprueba que se llame una vez al método *log* y que el texto devuelto sea igual a "Hello Bill".

A continuación, utilice la extensión Functions de VS Code para crear una nueva función de temporizador de JavaScript y asígnele el nombre *TimerTrigger*. Una vez creada la función, agregue un nuevo archivo en la misma carpeta denominado **index.test.js** y agregue el código siguiente:

```javascript
const timerFunction = require('./index');
const context = require('../testing/defaultContext');
const timer = require('../testing/defaultTimer');

test('Timer trigger should log message', () => {
    timerFunction(context, timer);
    expect(context.log.mock.calls.length).toBe(1);
});
```

La función de temporizador de la plantilla registra un mensaje al final del cuerpo de la función. Esta prueba garantiza que la función *log* se llame una vez.

### <a name="run-tests"></a>Ejecución de las pruebas

Para ejecutar las pruebas, presione **CTRL + ~** para abrir la ventana de comandos y ejecute `npm test`:

```bash
npm test
```

![Prueba de Azure Functions con JavaScript en VS Code](./media/functions-test-a-function/azure-functions-test-vs-code-jest.png)

### <a name="debug-tests"></a>Depuración de pruebas

Para depurar las pruebas, agregue la siguiente configuración al archivo *launch.json*:

```json
{
  "type": "node",
  "request": "launch",
  "name": "Jest Tests",
  "disableOptimisticBPs": true,
  "program": "${workspaceRoot}/node_modules/jest/bin/jest.js",
  "args": [
      "-i"
  ],
  "internalConsoleOptions": "openOnSessionStart"
}
```

A continuación, establezca un punto de interrupción en la prueba y presione **F5**.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a escribir pruebas automatizadas para las funciones, continúe con estos recursos:

- [Ejecución manual de una función no desencadenada por HTTP](./functions-manually-run-non-http.md)
- [Control de errores de Azure Functions](./functions-bindings-error-pages.md)
- [Depuración local de funciones de Azure desencadenadas por Event Grid](./functions-debug-event-grid-trigger-local.md)
