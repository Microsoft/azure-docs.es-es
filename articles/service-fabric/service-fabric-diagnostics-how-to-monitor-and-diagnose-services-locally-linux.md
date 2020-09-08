---
title: Depuración de aplicaciones Azure Service Fabric en Linux
description: Aprenda a supervisar y diagnosticar sus servicios de Service Fabric en una máquina de desarrollo local Linux.
ms.topic: conceptual
ms.date: 2/23/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 523cb0d1a8e8f322c1936f1fe52a954399b2acc5
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88999774"
---
# <a name="monitor-and-diagnose-services-in-a-local-linux-machine-development-setup"></a>Supervisión y diagnóstico de servicios en una configuración de desarrollo de máquina Linux local


> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)
> * [Linux](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md)
>
>

Acciones como supervisar, detectar, diagnosticar y solucionar problemas permiten a los servicios continuar con una interrupción mínima de la experiencia del usuario. La supervisión y el diagnóstico resultan fundamentales en un entorno de producción implementado real. La adopción de un modelo similar durante el desarrollo de servicios garantiza que la canalización del diagnóstico funcione cuando pasa a un entorno de producción. Service Fabric facilita a los desarrolladores de servicio la implementación de diagnósticos que puede funcionar sin problemas en configuraciones de desarrollo local de máquina única y en configuraciones de clúster de producción del mundo real.


## <a name="debugging-service-fabric-java-applications"></a>Depuración de aplicaciones Java de Service Fabric

Para aplicaciones Java, hay [varias plataformas de registro](https://en.wikipedia.org/wiki/Java_logging_framework) disponibles. Puesto que `java.util.logging` es la opción predeterminada con JRE, también se usa para los [ejemplos de código en GitHub](https://github.com/Azure-Samples/service-fabric-java-getting-started). En la siguiente discusión se explica cómo configurar la plataforma `java.util.logging` .

Con java.util.logging puede redirigir los registros de aplicaciones a la memoria, flujos de salida, archivos de consolas o sockets. Para cada una de estas opciones, hay controladores predeterminados que se proporcionan en la plataforma. Puede crear un archivo `app.properties` para configurar el controlador de archivo para la aplicación a fin de redirigir todos los registros a un archivo local.

El fragmento de código siguiente contiene una configuración de ejemplo:

```java
handlers = java.util.logging.FileHandler

java.util.logging.FileHandler.level = ALL
java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter
java.util.logging.FileHandler.limit = 1024000
java.util.logging.FileHandler.count = 10
java.util.logging.FileHandler.pattern = /tmp/servicefabric/logs/mysfapp%u.%g.log
```

La carpeta señalada por el archivo `app.properties` debe existir. Una vez creado el archivo `app.properties`, debe modificar igualmente el script de punto de entrada, `entrypoint.sh` en la carpeta `<applicationfolder>/<servicePkg>/Code/` para establecer la propiedad `java.util.logging.config.file` en el archivo `app.properties`. La entrada debería parecerse al siguiente fragmento:

```sh
java -Djava.library.path=$LD_LIBRARY_PATH -Djava.util.logging.config.file=<path to app.properties> -jar <service name>.jar
```


Esta configuración hace que se recopilen los registros por orden de rotación en `/tmp/servicefabric/logs/`. El archivo de registro en este caso se denomina mysfapp%u.%g.log donde:
* **%u** es un número único para resolver conflictos entre los procesos simultáneos de Java.
* **g** es el número de generación para distinguir entre los registros de rotación.

De forma predeterminada, si no hay ningún controlador configurado explícitamente, se registra el controlador de la consola. Se pueden ver los registros de syslog en /var/log/syslog.

Para más información, consulte los [ejemplos de código en GitHub](https://github.com/Azure-Samples/service-fabric-java-getting-started).


## <a name="debugging-service-fabric-c-applications"></a>Depuración de aplicaciones C# de Service Fabric


Hay varios marcos disponibles para realizar un seguimiento de aplicaciones de CoreCLR en Linux. Para obtener más información, consulte [Extensiones de .NET para el registro](https://github.com/dotnet/extensions/tree/master/src/Logging).  Dado que los desarrolladores de C# ya conocen EventSource, este artículo utiliza EventSource para realizar un seguimiento en muestras de CoreCLR en Linux.

El primer paso es incluir System.Diagnostics.Tracing para que pueda escribir los registros en la memoria, en flujos de salida o en archivos de consola.  Para registrarse mediante EventSource, añada el siguiente proyecto a project.json:

```json
    "System.Diagnostics.StackTrace": "4.0.1"
```

Puede usar un EventListener personalizado para escuchar el evento de servicio y, a continuación, redirigirlos correctamente a los archivos de seguimiento. El fragmento de código siguiente muestra un ejemplo de implementación de un registro con EventSource y un EventListener personalizado:


```csharp

public class ServiceEventSource : EventSource
{
        public static ServiceEventSource Current = new ServiceEventSource();

        [NonEvent]
        public void Message(string message, params object[] args)
        {
            if (this.IsEnabled())
            {
                var finalMessage = string.Format(message, args);
                this.Message(finalMessage);
            }
        }

        // TBD: Need to add method for sample event.

}

```


```csharp
internal class ServiceEventListener : EventListener
{

        protected override void OnEventSourceCreated(EventSource eventSource)
        {
            EnableEvents(eventSource, EventLevel.LogAlways, EventKeywords.All);
        }
        protected override void OnEventWritten(EventWrittenEventArgs eventData)
        {
                using (StreamWriter Out = new StreamWriter( new FileStream("/tmp/MyServiceLog.txt", FileMode.Append)))
                {
                        // report all event information
                        Out.Write(" {0} ", Write(eventData.Task.ToString(), eventData.EventName, eventData.EventId.ToString(), eventData.Level,""));
                        if (eventData.Message != null)
                                Out.WriteLine(eventData.Message, eventData.Payload.ToArray());
                        else
                        {
                                string[] sargs = eventData.Payload != null ? eventData.Payload.Select(o => o.ToString()).ToArray() : null; 
                                Out.WriteLine("({0}).", sargs != null ? string.Join(", ", sargs) : "");
                        }
                }
        }
}
```


El fragmento de código anterior genera los registros en un archivo en `/tmp/MyServiceLog.txt`. Este nombre de archivo debe actualizarse correctamente. En caso de que desee redirigir los registros a la consola, use el siguiente fragmento de código en la clase de EventListener personalizada:

```csharp
public static TextWriter Out = Console.Out;
```

Los ejemplos de [C# Samples](https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started) (Ejemplos de C#) usan EventSource y un EventListener personalizado para registrar eventos en un archivo.



## <a name="next-steps"></a>Pasos siguientes
El mismo código de seguimiento que agregó a la aplicación también funciona con los diagnósticos de la aplicación en un clúster de Azure. Consulte estos artículos que tratan sobre las distintas opciones de las herramientas y describen cómo configurarlas.
* [Recopilación de registros con Diagnósticos de Azure](./service-fabric-diagnostics-event-aggregation-lad.md)
