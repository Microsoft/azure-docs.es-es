---
title: Depuración local de Azure Functions de PowerShell
description: Obtenga información sobre cómo depurar funciones de PowerShell durante ejecuciones en el entorno local.
author: tylerleonhardt
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: tyleonha
ms.reviewer: glenga
ms.openlocfilehash: 6be397631621c727bb8979df2ee8eec3aca43096
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799373"
---
# <a name="debug-powershell-azure-functions-locally"></a>Depuración local de Azure Functions de PowerShell

Azure Functions le permite desarrollar sus funciones como scripts de PowerShell.

Puede depurar las funciones de PowerShell de forma local como lo haría con cualquier script de PowerShell con las siguientes herramientas de desarrollo estándar:

* [Visual Studio Code](https://code.visualstudio.com/): editor de texto de código abierto, ligero y gratuito de Microsoft con la extensión de PowerShell que ofrece una experiencia completa de desarrollo de PowerShell.
* Una consola de PowerShell: depure con los mismos comandos que usaría para depurar cualquier otro proceso de PowerShell.

[Azure Functions Core Tools](functions-run-local.md) admite la depuración local de Azure Functions, incluidas las funciones de PowerShell.

## <a name="example-function-app"></a>Aplicación de funciones de ejemplo

La aplicación de funciones usada en este artículo tiene una sola función desencadenada por HTTP y tiene los siguientes archivos:

```
PSFunctionApp
 | - HttpTriggerFunction
 | | - run.ps1
 | | - function.json
 | - local.settings.json
 | - host.json
 | - profile.ps1
```

Esta aplicación de funciones es similar a la que obtiene al completar el [inicio rápido de PowerShell](./functions-create-first-function-vs-code.md?pivots=programming-language-powershell).

El código de función de `run.ps1` es similar al siguiente script:

```powershell
param($Request)

$name = $Request.Query.Name

if($name) {
    $status = 200
    $body = "Hello $name"
}
else {
    $status = 400
    $body = "Please pass a name on the query string or in the request body."
}

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = $status
    Body = $body
})
```

## <a name="set-the-attach-point"></a>Establecimiento del punto de asociación

Para depurar cualquier función de PowerShell, esta se debe detener para que se asocie el depurador. El cmdlet `Wait-Debugger` detiene la ejecución y espera por el depurador.

Lo único que tiene que hacer es agregar una llamada al cmdlet `Wait-Debugger` justo encima de la instrucción `if`, como se indica a continuación:

```powershell
param($Request)

$name = $Request.Query.Name

# This is where we will wait for the debugger to attach
Wait-Debugger

if($name) {
    $status = 200
    $body = "Hello $name"
}
# ...
```

La depuración se inicia en la instrucción `if`. 

Con `Wait-Debugger` en su lugar, ya puede depurar las funciones con Visual Studio Code o una consola de PowerShell.

## <a name="debug-in-visual-studio-code"></a>Depuración en Visual Studio Code

Para depurar sus funciones de PowerShell en Visual Studio Code, debe tener instalado lo siguiente:

* [Extensión de PowerShell para Visual Studio Code](/powershell/scripting/components/vscode/using-vscode)
* [Extensión de Azure Functions para Visual Studio Code](functions-create-first-function-vs-code.md)
* [PowerShell Core 6.2 o una versión posterior](/powershell/scripting/install/installing-powershell-core-on-windows)

Después de instalar estas dependencias, cargue un proyecto existente de Functions de PowerShell o [cree su primer proyecto de Functions de PowerShell](./functions-create-first-function-vs-code.md?pivots=programming-language-powershell).

>[!NOTE]
> Si el proyecto no tiene los archivos de configuración necesarios, se le pedirá que los agregue.

### <a name="set-the-powershell-version"></a>Establecimiento de la versión de PowerShell

PowerShell Core se instala en paralelo con Windows PowerShell. Establezca PowerShell Core como la versión de PowerShell que se usará con la extensión de PowerShell para Visual Studio Code.

1. Presione F1 para mostrar la paleta de comandos y luego busque `Session`.

1. Elija **PowerShell: Show Session Menu** (PowerShell: Mostrar menú de sesión).

1. Si su **sesión actual** no es **PowerShell Core 6**, elija **Cambiar a: PowerShell Core 6**.

Cuando haya abierto un archivo de PowerShell, la versión se muestra en verde en la parte inferior derecha de la ventana. Al seleccionar este texto, también se muestra el menú de sesiones. Para obtener más información, consulte [Elección de una versión de PowerShell para usarla con la extensión](/powershell/scripting/components/vscode/using-vscode#choosing-a-version-of-powershell-to-use-with-the-extension).

### <a name="start-the-function-app"></a>Inicio de la aplicación de funciones

Compruebe que `Wait-Debugger` está establecido en la función en la que quiere asociar el depurador.  Con `Wait-Debugger` agregado, puede depurar la aplicación de funciones mediante Visual Studio Code.

Elija el panel **Depurar** y luego **Attach to PowerShell function** (Asociar con la función de PowerShell).

![depurador](https://user-images.githubusercontent.com/2644648/56166073-8a7b3780-5f89-11e9-85ce-36ed38e221a2.png)

También puede presionar la tecla F5 para iniciar la depuración.

La operación de inicio de la depuración realiza las siguientes tareas:

* Ejecuta `func extensions install` en el terminal para instalar las extensiones de Azure Functions que requiere la aplicación de funciones.
* Ejecuta `func host start` en el terminal para iniciar la aplicación de funciones en el host de Functions.
* Asocia el depurador de PowerShell con espacio de ejecución de PowerShell del entorno de ejecución de Functions.

>[!NOTE]
> Debe asegurarse de que PSWorkerInProcConcurrencyUpperBound esté establecido en 1 para garantizar una experiencia de depuración correcta en Visual Studio Code. Este es el valor predeterminado.

Con la aplicación de funciones en ejecución, necesita una consola de PowerShell independiente para llamar a la función desencadenada por HTTP.

En este caso, la consola de PowerShell es el cliente. Se usa `Invoke-RestMethod` para desencadenar la función.

En una consola de PowerShell, ejecute el siguiente comando:

```powershell
Invoke-RestMethod "http://localhost:7071/api/HttpTrigger?Name=Functions"
```

Observará que no se devuelve una respuesta de inmediato. Eso se debe a que `Wait-Debugger` ha asociado el depurador y la ejecución de PowerShell ha entrado en el modo de interrupción en cuanto pudo. Esto se debe al [concepto BreakAll](#breakall-might-cause-your-debugger-to-break-in-an-unexpected-place), que se explica más adelante. Después de hacer clic en el botón `continue`, el depurador se interrumpe en la línea inmediatamente posterior a `Wait-Debugger`.

En este momento, el depurador está asociado y puede hacer todas las operaciones normales del depurador. Para obtener más información sobre cómo usar el depurador en Visual Studio Code, consulte [la documentación oficial](https://code.visualstudio.com/Docs/editor/debugging#_debug-actions).

Después de seguir e invocar el script por completo, observará lo siguiente:

* La consola de PowerShell que invocó el comando `Invoke-RestMethod` ha devuelto un resultado.
* La consola integrada de PowerShell en Visual Studio Code está esperando a que se ejecute un script.

Más adelante, cuando invoque la misma función, el depurador de la extensión de PowerShell se interrumpe inmediatamente después de `Wait-Debugger`.

## <a name="debugging-in-a-powershell-console"></a>Depuración en una consola de PowerShell

>[!NOTE]
> En esta sección, se asume que ha leído la [documentación de Azure Functions Core Tools](functions-run-local.md) y sabe cómo usar el comando `func host start` para iniciar la aplicación de funciones.

Abra una consola, escriba `cd` en el directorio de la aplicación de funciones y ejecute el comando siguiente:

```sh
func host start
```

Con la aplicación de funciones en ejecución y `Wait-Debugger` en su lugar, puede asociarlo al proceso. Necesita dos consolas más de PowerShell.

Una de ellas actúa como el cliente. Desde esta, se llama a `Invoke-RestMethod` para desencadenar la función. Por ejemplo, puede ejecutar el siguiente comando:

```powershell
Invoke-RestMethod "http://localhost:7071/api/HttpTrigger?Name=Functions"
```

Observará que no devuelve una respuesta, que es el resultado de `Wait-Debugger`. El espacio de ejecución de PowerShell ahora espera a que se asocie un depurador. Es hora de asociarlo.

En la otra consola de PowerShell, ejecute el siguiente comando:

```powershell
Get-PSHostProcessInfo
```

Este cmdlet devuelve una tabla que es similar a la siguiente salida:

```output
ProcessName ProcessId AppDomainName
----------- --------- -------------
dotnet          49988 None
pwsh            43796 None
pwsh            49970 None
pwsh             3533 None
pwsh            79544 None
pwsh            34881 None
pwsh            32071 None
pwsh            88785 None
```

Apunte el valor de `ProcessId` del elemento de la tabla con `ProcessName` como `dotnet`. Este proceso es la aplicación de funciones.

Después, ejecute el siguiente fragmento de código:

```powershell
# This enters into the Azure Functions PowerShell process.
# Put your value of `ProcessId` here.
Enter-PSHostProcess -Id $ProcessId

# This triggers the debugger.
Debug-Runspace 1
```

Una vez iniciado, el depurador se interrumpe y muestra algo parecido a la siguiente salida:

```
Debugging Runspace: Runspace1

To end the debugging session type the 'Detach' command at the debugger prompt, or type 'Ctrl+C' otherwise.

At /Path/To/PSFunctionApp/HttpTriggerFunction/run.ps1:13 char:1
+ if($name) { ...
+ ~~~~~~~~~~~
[DBG]: [Process:49988]: [Runspace1]: PS /Path/To/PSFunctionApp>>
```

En este momento, se ha detenido en un punto de interrupción del [depurador de PowerShell](/powershell/module/microsoft.powershell.core/about/about_debuggers). Desde aquí, puede realizar todas las operaciones de depuración usuales, depurar paso a paso por procedimientos, depurar paso a paso por instrucciones, continuar, salir, etc. Para ver el conjunto completo de comandos de depuración disponibles en la consola, ejecute los comandos `h` o `?`.

También puede establecer puntos de interrupción en este nivel con el cmdlet `Set-PSBreakpoint`.

Después de seguir e invocar el script por completo, observará lo siguiente:

* La consola de PowerShell en la que ha ejecutado `Invoke-RestMethod` ahora ha devuelto un resultado.
* La consola de PowerShell en la que ha ejecutado `Debug-Runspace` está esperando a que se ejecute un script.

Puede volver a invocar la misma función (con `Invoke-RestMethod`, por ejemplo) y el depurador se interrumpe justo después del comando `Wait-Debugger`.

## <a name="considerations-for-debugging"></a>Consideraciones de la depuración

Tenga en cuenta los siguientes problemas al depurar el código de Functions.

### <a name="breakall-might-cause-your-debugger-to-break-in-an-unexpected-place"></a>`BreakAll` puede provocar que el depurador se interrumpa en un lugar inesperado.

La extensión de PowerShell usa `Debug-Runspace`, que a su vez se basa en la característica `BreakAll` de PowerShell. Esta característica indica a PowerShell que se detenga en el primer comando que se ejecuta. Este comportamiento le permite establecer puntos de interrupción en el espacio de ejecución depurado.

El entorno de ejecución de Azure Functions ejecuta algunos comandos antes de invocar realmente el script `run.ps1`, por lo que es posible que el depurador se acabe interrumpiendo en `Microsoft.Azure.Functions.PowerShellWorker.psm1` o `Microsoft.Azure.Functions.PowerShellWorker.psd1`.

Si se produce esta interrupción, ejecute el comando `continue` o `c` para saltar este punto de interrupción. De este modo, se detendrá en el punto de interrupción esperado.

## <a name="troubleshooting"></a>Solución de problemas

Si tiene dificultades durante la depuración, debe comprobar lo siguiente:

| Comprobar | Acción |
|------|------|
| Ejecute `func --version` desde el terminal. Si recibe un error indicando que no se puede encontrar `func`, es posible que falte Core Tools (func.exe) en la variable `path` local.| [Vuelva a instalar Core Tools](functions-run-local.md#v2).|  
| En Visual Studio Code, el terminal predeterminado debe tener acceso a func.exe. Asegúrese de que no está usando un terminal predeterminado que no tenga instalado Core Tools, como el subsistema de Windows para Linux (WSL).  | Establezca el shell predeterminado en Visual Studio Code en PowerShell 7 (recomendado) o Windows PowerShell 5.1.|
  

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo desarrollar Functions con PowerShell, consulte [Guía del desarrollador de PowerShell para Azure Functions](functions-reference-powershell.md).
