---
title: Generación del perfil de un servicio en la nube en modo local en el emulador de proceso | Microsoft Docs
services: cloud-services
description: Investigar los problemas de rendimiento en servicios en la nube con el generador de perfiles de Visual Studio
documentationcenter: ''
author: mikejo
manager: jillfra
editor: ''
tags: ''
ms.assetid: 25e40bf3-eea0-4b0b-9f4a-91ffe797f6c3
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/18/2016
ms.author: mikejo
ms.openlocfilehash: 503b1ca8c8f1ff06bbdd54316611d886fb1e9d87
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462316"
---
# <a name="testing-the-performance-of-a-cloud-service-locally-in-the-azure-compute-emulator-using-the-visual-studio-profiler"></a>Prueba del rendimiento de un servicio en la nube de manera local en el emulador de Azure Compute con el generador de perfiles de Visual Studio
Se encuentran disponibles diversas herramientas y técnicas para probar el rendimiento de los servicios en la nube.
Al publicar un servicio en la nube en Azure, puede usar Visual Studio para recopilar datos para la generación de perfiles y, luego, analizarlos localmente, como se describe en [Generación de un perfil de una aplicación de Azure][1].
También puede usar un diagnóstico para hacer un seguimiento de una amplia variedad de contadores de rendimiento, como se describe en [Uso de contadores de rendimiento en Azure][2].
También es posible que desee generar un perfil de la aplicación localmente en el emulador de proceso antes de implementarlo en la nube.

Este artículo abarca el método de muestreo de CPU de la generación de perfiles, que se puede realizar localmente en el emulador. El muestreo de CPU es un método para generar perfiles que no es muy intrusivo. A un intervalo de muestreo designado, el generador de perfiles realiza una instantánea de la pila de llamadas. Los datos se recopilan por un lapso de tiempo y se muestran en un informe. Este método de generación de perfiles tiende a indicar dónde se está realizando la mayoría del trabajo de la CPU en una aplicación informáticamente intensiva.  Esto le da la oportunidad de centrarse en la "ruta de acceso activa" donde su aplicación pasa la mayor parte del tiempo.

## <a name="1-configure-visual-studio-for-profiling"></a>1: Configuración de Visual Studio para la generación de perfiles
Primero, existen unas pocas opciones de configuración de Visual Studio que podrían ser útiles para la generación de perfiles. Para que los informes de generación de perfiles tengan sentido, necesitará símbolos (archivos .pdb) para su aplicación y también símbolos para las bibliotecas del sistema. Necesitará asegurarse de que hace referencia a los servidores de símbolos disponibles. Para hacer esto, en el menú **Herramientas** de Visual Studio, elija **Opciones** y, a continuación, elija **Depuración** y luego, **Símbolos**. Asegúrese de que los servidores de símbolos de Microsoft aparezcan en **Ubicaciones del archivo de símbolos (.pdb)** .  También puede hacer referencia a https://referencesource.microsoft.com/symbols, que podría tener archivos de símbolos adicionales.

![Opciones Símbolo][4]

Si lo desea, puede simplificar los informes que genera el generador de perfiles al configurar Solo mi código. Si Solo mi código está habilitado, las pilas de llamadas de función se simplifican de modo que las llamadas que son completamente internas para las bibliotecas y el .NET Framework están ocultas de los informes. En el menú **Herramientas**, elija **Opciones**. A continuación, expanda el nodo **Herramientas de rendimiento** y elija **General**. Seleccione la casilla **Habilitar solo mi código para informes del generador de perfiles**.

![Opciones Solo mi código][17]

Puede usar estas instrucciones con un proyecto existente o con un proyecto nuevo.  Si crea un proyecto nuevo para probar las técnicas que se describen a continuación, elija un proyecto de C# de **Servicio en la nube de Azure** y seleccione un **rol web** y un **rol de trabajo**.

![Roles de proyecto del servicio en la nube de Azure][5]

Para propósitos de ejemplo, agregue parte del código al proyecto que tarda demasiado tiempo y demuestra un problema de rendimiento obvio. Por ejemplo, agregue el siguiente código a un proyecto de rol de trabajo:

```csharp
public class Concatenator
{
    public static string Concatenate(int number)
    {
        int count;
        string s = "";
        for (count = 0; count < number; count++)
        {
            s += "\n" + count.ToString();
        }
        return s;
    }
}
```

Llame a este código desde el método RunAsync en la clase derivada de RoleEntryPoint del rol de trabajo. (Ignore la advertencia sobre el método que se ejecuta sincrónicamente).

```csharp
private async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following with your own logic.
    while (!cancellationToken.IsCancellationRequested)
    {
        Trace.TraceInformation("Working");
        Concatenator.Concatenate(10000);
    }
}
```

Compile y ejecute localmente su servicio en la nube sin depuración (Ctrl+F5), con la configuración de solución establecida en **Liberar**. Esto asegura que todos los archivos y carpetas se crean para ejecutar la aplicación localmente y asegura que se inicien todos los emuladores. Comience la interfaz de usuario del emulador de proceso desde la barra de tareas para comprobar que el rol de trabajo se está ejecutando.

## <a name="2-attach-to-a-process"></a>2: Asociación a un proceso
En vez de generar un perfil en la aplicación al iniciarla desde Visual Studio 2010 IDE, debe asociar el generador de perfiles a un proceso en ejecución. 

Para asociar el generador de perfiles a un proceso, en el menú **Analizar**, elija **Generador de perfiles** y **Asociar/desasociar**.

![Opción para adjuntar perfil][6]

Para un rol de trabajo, busque el proceso WaWorkerHost.exe.

![Proceso WaWorkerHost][7]

Si la carpeta de su proyecto se encuentra en una unidad de red, el generador de perfiles le pedirá proporcionar otra ubicación para guardar los informes de generación de perfiles.

 Se puede también asociar a un rol web asociándose al archivo WaIISHost.exe.
Si hay varios procesos de rol de trabajo en su aplicación, necesita usar processID para distinguirlos. Puede consultar el processID de manera programática al tener acceso al objeto del proceso. Por ejemplo, si agrega este código al método Run de la clase derivada de RoleEntryPoint en un rol, puede ver el registro en la interfaz de usuario del emulador de proceso para conocer el proceso al que debe conectarse.

```csharp
var process = System.Diagnostics.Process.GetCurrentProcess();
var message = String.Format("Process ID: {0}", process.Id);
Trace.WriteLine(message, "Information");
```

Para ver el registro, inicie la interfaz de usuario del emulador de proceso.

![Inicio de la interfaz de usuario del emulador de proceso][8]

Abra la ventana de consola de registro del rol de trabajo en la interfaz de usuario del emulador de proceso al hacer clic en la barra de título de la ventana de la consola. Puede ver el identificador de proceso en el registro.

![Visualización del identificador de proceso][9]

Después de que se haya asociado, realice los pasos en la interfaz de usuario de su aplicación (si es necesario) para reproducir el escenario.

Cuando desee detener la generación de perfiles, seleccione el vínculo **Detener generación de perfiles** .

![Opción Detener generación de perfiles][10]

## <a name="3-view-performance-reports"></a>3: Vista de informes de rendimiento
Aparece el informe de rendimiento de la aplicación.

En este punto, el generador de perfiles detiene su ejecución, guarda los datos en un archivo .vsp y exhibe un informe que muestra un análisis de estos datos.

![Informe del generador de perfiles][11]

Si ve String.wstrcpy en la ruta de acceso activa, haga clic en Solo mi código para cambiar la vista a fin de mostrar el código de usuario solamente.  Si ve String.Concat, intente presionar el botón Mostrar todo el código.

Debería ver el método Concatenate y String.Concat que ocupan una gran parte del tiempo de ejecución.

![Análisis del informe][12]

Si agregó el código de concatenación de cadena en este artículo, debería ver una advertencia en la lista de tareas para esta. Es posible que también vea una advertencia de que hay una cantidad excesiva de elementos no utilizados, lo que se debe a la cantidad de cadenas que se crearon y desplegaron.

![Advertencias de rendimiento][14]

## <a name="4-make-changes-and-compare-performance"></a>4: Realización de cambios y comparación del rendimiento
Puede también comparar el rendimiento antes y después de un cambio en el código.  Detenga el proceso de ejecución y edite el código para reemplazar la operación de concatenación de cadena con el uso de StringBuilder:

```csharp
public static string Concatenate(int number)
{
    int count;
    System.Text.StringBuilder builder = new System.Text.StringBuilder("");
    for (count = 0; count < number; count++)
    {
        builder.Append("\n" + count.ToString());
    }
    return builder.ToString();
}
```

Realice otra ejecución de rendimiento y, a continuación, compárelo. En el Explorador de rendimiento, si las ejecuciones se encuentran en la misma sesión, simplemente puede seleccionar ambos informes, abrir el menú de acceso directo y seleccionar **Comparar informes de rendimiento**. Si desea realizar una comparación con una ejecución en otra sesión de rendimiento, abra el menú **Analizar** y seleccione **Comparar informes de rendimiento**. En el cuadro de diálogo que aparece, especifique ambos archivos.

![Opción para comparar informes de rendimiento][15]

Los informes resaltan las diferencias entre las dos ejecuciones.

![Informe de comparación][16]

Felicidades. Ya ha empezado a usar el generador de perfiles.

## <a name="troubleshooting"></a>Solución de problemas
* Asegúrese de que va a generar un perfil de una compilación de versión e iniciar sin depuración.
* Si la opción Asociar/Desasociar no está habilitada en el menú del Generador de perfiles, ejecute el Asistente de rendimiento.
* Use la interfaz de usuario del emulador de proceso para ver el estado de la aplicación. 
* Si tiene problemas al iniciar aplicaciones en el emulador o asociar el generador de perfiles, apague el emulador de proceso y reinícielo. Si el problema no se soluciona, intente reiniciar. Este problema se produce si usa el emulador de proceso para suspender y quitar implementaciones en ejecución.
* Si ha utilizado cualquiera de los comandos de generación de perfiles desde la línea de comandos, especialmente la configuración global, asegúrese de que se haya llamado a VSPerfClrEnv /globaloff y que VsPerfMon.exe se haya apagado.
* Si al realizar el muestreo, ve el mensaje "PRF0025: no se recopilaron datos", compruebe que el proceso al que se asoció tiene actividad de CPU. Es posible que las aplicaciones que no están realizando ningún trabajo informático no produzcan datos de muestreo.  También es posible que el proceso haya finalizado antes de que se haya realizado muestreo alguno. Compruebe que el método de ejecución de un rol para el cual está generando un perfil no termine.

## <a name="next-steps"></a>Pasos siguientes
La instrumentación de binarios de Azure en el emulador no es compatible en el generador de perfiles de Visual Studio; sin embargo, si desea probar la asignación de memoria, puede seleccionar esta opción al generar perfiles. También puede seleccionar una generación de perfiles de concurrencia, la cual le ayuda a determinar si los subprocesos están desperdiciando tiempo al competir por bloqueos, o la generación de perfiles de interacción de capa, que le ayuda a hacer un seguimiento de los problemas de rendimiento cuando se interactúa entre las capas de una aplicación, con mayor frecuencia entre la capa de datos y un rol de trabajo.  Puede ver las consultas de la base de datos que genera la aplicación y usar los datos de generación de perfiles para mejorar el uso que hace de la base de datos. Para obtener información acerca de la generación de perfiles de interacción de capa, consulte la entrada de blog [Walkthrough: Uso del generador de perfiles de interacción de capa en Visual Studio Team System 2010][3].

[1]: https://docs.microsoft.com/azure/application-insights/app-insights-profiler
[2]: https://msdn.microsoft.com/library/azure/hh411542.aspx
[3]: https://docs.microsoft.com/archive/blogs/habibh/walkthrough-using-the-tier-interaction-profiler-in-visual-studio-team-system-2010
[4]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally09.png
[5]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally10.png
[6]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally02.png
[7]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally05.png
[8]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally010.png
[9]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally07.png
[10]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally06.png
[11]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally03.png
[12]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally011.png
[14]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally04.png 
[15]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally013.png
[16]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally012.png
[17]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally08.png
