---
title: Canales de telemetría en Azure Application Insights | Microsoft Docs
description: Cómo personalizar los canales de telemetría en el SDK de Azure Application Insights para .NET y .NET Core.
ms.topic: conceptual
ms.date: 05/14/2019
ms.reviewer: mbullwin
ms.openlocfilehash: b5ae1ee1e4bf9f64eb4587f0ceb76972a4571b2e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318936"
---
# <a name="telemetry-channels-in-application-insights"></a>Canales de telemetría en Application Insights

Los canales de telemetría son una parte integral de los [SDK de Azure Application Insights](./app-insights-overview.md). Administran el almacenamiento en búfer y la transmisión de telemetría al servicio Application Insights. Las versiones de los SDK de .NET y .NET Core tienen dos canales de telemetría integrados: `InMemoryChannel` y `ServerTelemetryChannel`. En este artículo se describe cada canal en detalle, incluido el procedimiento para personalizar el comportamiento del canal.

## <a name="what-are-telemetry-channels"></a>¿Cuáles son los canales de telemetría?

Los canales de telemetría son responsables de almacenar en búfer los elementos de telemetría y enviarlos al servicio Application Insights, donde se almacenan con finalidades de consultas y análisis. Un canal de telemetría es cualquier clase que implementa la interfaz [`Microsoft.ApplicationInsights.ITelemetryChannel`](/dotnet/api/microsoft.applicationinsights.channel.itelemetrychannel?view=azure-dotnet).

El método `Send(ITelemetry item)` de un canal de telemetría se invoca después de llamar a todos los inicializadores y procesadores de telemetría. Por lo tanto, todos los elementos que proporciona un procesador de telemetría no alcanzarán el canal. Normalmente, el método `Send()` no envía los elementos al back-end de manera instantánea. Por lo general, los almacena en búfer y los envía en lotes a fin de que la transmisión sea eficaz.

[Live Metrics Stream](live-stream.md) también tiene un canal personalizado que sustenta el streaming en vivo de la telemetría. Este canal es independiente del canal de telemetría normal, por lo que no se cubre en este documento.

## <a name="built-in-telemetry-channels"></a>Canales de telemetría integrados

Los SDK de .NET Core y .NET de Application Insights se envían con dos canales integrados:

* `InMemoryChannel`: un canal ligero que almacena en búfer los elementos en la memoria hasta que se envían. Los elementos se almacenan en búfer en la memoria y se vacían una vez cada 30 segundos, o cada vez que se almacenan en búfer 500 elementos. Este canal ofrece garantías de confiabilidad mínimas porque no reintenta el envío de telemetría tras un error. Asimismo, no conserva los elementos en el disco, por lo que todos los elementos no enviados se pierden permanentemente cuando se cierra la aplicación (independiente de si se trata de un cierre estable o no). Este canal implementa un método `Flush()` que puede usarse para vaciar por la fuerza todos los elementos de telemetría que hay en la memoria de manera sincrónica. Resulta adecuado para las aplicaciones de ejecución breve en las que es ideal realizar un vaciado sincrónico.

    El canal forma parte del mayor paquete Microsoft.ApplicationInsights de NuGet y es el que usa el SDK de manera predeterminada cuando no hay nada más configurado.

* `ServerTelemetryChannel`: se trata de un canal más avanzado con directivas de reintento y la capacidad de almacenar datos en un disco local. Este canal reintenta el envío de telemetría si se producen errores transitorios. Asimismo, usa el almacenamiento en disco local para mantener los elementos en el disco durante las interrupciones de red o grandes volúmenes de telemetría. Debido a estos mecanismos de reintento y el almacenamiento en disco local, este canal se considera más confiable y se recomienda para todos los escenarios de producción. Es la opción predeterminada para las aplicaciones [ASP.NET](./asp-net.md) y [ASP.NET Core](./asp-net-core.md) que están configuradas según la documentación oficial. Este canal está optimizado para escenarios de servidor con procesos de larga ejecución. El método [`Flush()`](#which-channel-should-i-use) que implementa este canal no es sincrónico.

    Este canal se incluye con el paquete Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel NuGet y se adquiere automáticamente cuando se utiliza el paquete de NuGet Microsoft.ApplicationInsights.AspNetCore o Microsoft.ApplicationInsights.Web.

## <a name="configure-a-telemetry-channel"></a>Configuración de un canal de telemetría

Para configurar un canal de telemetría, debe establecerlo en la configuración de telemetría activa. En el caso de las aplicaciones de ASP.NET, la configuración implica establecer la instancia del canal de telemetría en `TelemetryConfiguration.Active` o bien modificar `ApplicationInsights.config`. Para las aplicaciones de ASP.NET Core, la configuración implica agregar el canal al contenedor de inserción de dependencias.

En las secciones siguientes, se muestran ejemplos de configuración del valor `StorageFolder` para el canal en distintos tipos de aplicaciones. `StorageFolder` es solo uno de los valores configurables. Para obtener la lista completa de valores de configuración, consulte [la sección configuración](#configurable-settings-in-channels) más adelante en este artículo.

### <a name="configuration-by-using-applicationinsightsconfig-for-aspnet-applications"></a>Configuración mediante ApplicationInsights.config para aplicaciones de ASP.NET

En la sección siguiente de [ApplicationInsights.config](configuration-with-applicationinsights-config.md) se muestra el canal configurado `ServerTelemetryChannel` con `StorageFolder` establecido en una ubicación personalizada:

```xml
    <TelemetrySinks>
        <Add Name="default">
            <TelemetryProcessors>
                <!-- Telemetry processors omitted for brevity  -->
            </TelemetryProcessors>
            <TelemetryChannel Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel, Microsoft.AI.ServerTelemetryChannel">
                <StorageFolder>d:\temp\applicationinsights</StorageFolder>
            </TelemetryChannel>
        </Add>
    </TelemetrySinks>
```

### <a name="configuration-in-code-for-aspnet-applications"></a>Configuración del código para las aplicaciones de ASP.NET

El código siguiente configura una instancia de "ServerTelemetryChannel" con `StorageFolder` establecido en una ubicación personalizada. Agregue este código al principio de la aplicación, normalmente en el método `Application_Start()` de Global.aspx.cs.

```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
protected void Application_Start()
{
    var serverTelemetryChannel = new ServerTelemetryChannel();
    serverTelemetryChannel.StorageFolder = @"d:\temp\applicationinsights";
    serverTelemetryChannel.Initialize(TelemetryConfiguration.Active);
    TelemetryConfiguration.Active.TelemetryChannel = serverTelemetryChannel;
}
```

### <a name="configuration-in-code-for-aspnet-core-applications"></a>Configuración del código para las aplicaciones de ASP.NET Core

Modifique el método `ConfigureServices` de la clase `Startup.cs` de la siguiente manera:

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

public void ConfigureServices(IServiceCollection services)
{
    // This sets up ServerTelemetryChannel with StorageFolder set to a custom location.
    services.AddSingleton(typeof(ITelemetryChannel), new ServerTelemetryChannel() {StorageFolder = @"d:\temp\applicationinsights" });

    services.AddApplicationInsightsTelemetry();
}

```

> [!IMPORTANT]
> No se recomienda configurar el canal mediante `TelemetryConfiguration.Active` para las aplicaciones de ASP.NET Core.

### <a name="configuration-in-code-for-netnet-core-console-applications"></a>Configuración del código para las aplicaciones de consola de ASP.NET Core o .NET

En el caso de las aplicaciones de consola, el código es el mismo para .NET y .NET Core:

```csharp
var serverTelemetryChannel = new ServerTelemetryChannel();
serverTelemetryChannel.StorageFolder = @"d:\temp\applicationinsights";
serverTelemetryChannel.Initialize(TelemetryConfiguration.Active);
TelemetryConfiguration.Active.TelemetryChannel = serverTelemetryChannel;
```

## <a name="operational-details-of-servertelemetrychannel"></a>Detalles de funcionamiento de ServerTelemetryChannel

`ServerTelemetryChannel` almacena los elementos que llegan de un búfer en memoria. Los elementos se serializan, comprimen y almacenan en una instancia de `Transmission` una vez cada 30 segundos, o cuando se han almacenado en búfer 500 elementos. Una sola instancia de `Transmission`contiene hasta 500 elementos y representa un lote de datos de telemetría que se envía a través de una sola llamada HTTPS al servicio de Application Insights.

De forma predeterminada, se puede enviar un máximo de 10 instancias de `Transmission` en paralelo. Si llegan datos de telemetría a velocidades más rápidas, o si el back-end de application Insights o la red funcionan con lentitud, las instancias de `Transmission` se almacenan en memoria. La capacidad predeterminada de este búfer `Transmission` en memoria es de 5 MB. Cuando se ha superado la capacidad en memoria, las instancias de `Transmission` se almacenan en el disco local hasta un límite de 50 MB. Las instancias de `Transmission` también se almacenan en el disco local cuando hay problemas de red. Solo los elementos que se almacenan en un disco local sobreviven a un bloqueo de la aplicación. Se envían siempre que la aplicación se inicia de nuevo.

## <a name="configurable-settings-in-channels"></a>Valores configurables en los canales

Para obtener la lista completa de valores configurables de cada canal, consulte:

* [Inmemorychannel](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/BASE/src/Microsoft.ApplicationInsights/Channel/InMemoryChannel.cs)

* [ServerTelemetryChannel](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/BASE/src/ServerTelemetryChannel/ServerTelemetryChannel.cs)

A continuación, le mostramos la configuración que se usa con más frecuencia para `ServerTelemetryChannel`:

1. `MaxTransmissionBufferCapacity`: cantidad máxima de memoria, en bytes, que usa el canal para las transmisiones de búfer en memoria. Cuando se alcanza esta capacidad, los nuevos elementos se almacenan directamente en el disco local. El valor predeterminado es de 5 MB. Si se establece un valor superior, el uso del disco se reduce, sin embargo, recuerde que los elementos en memoria se perderán si se bloquea la aplicación.

1. `MaxTransmissionSenderCapacity`: número máximo de instancias de `Transmission` que se enviarán a Application Insights al mismo tiempo. El valor predeterminado es 10. Este valor puede configurarse en un número mayor, cosa que se recomienda cuando se genera un gran volumen de telemetría. El gran volumen normalmente tiene lugar durante las pruebas de carga o cuando se desactiva el muestreo.

1. `StorageFolder`: Carpeta que usa el canal para almacenar elementos en el disco según sea necesario. En Windows, se usa %LOCALAPPDATA% o %TEMP% si no se especifica ninguna otra ruta de acceso de forma explícita. En entornos que no son de Windows, debe especificar una ubicación válida o, de lo contrario, la telemetría no se almacenará en el disco local.

## <a name="which-channel-should-i-use"></a>¿Qué canal debería usar?

Se recomienda el canal `ServerTelemetryChannel` para los escenarios de mayor producción que implican aplicaciones de ejecución prolongada. El método `Flush()` que implementa `ServerTelemetryChannel` no es sincrónico, y tampoco garantiza que se envíen todos los elementos pendientes de la memoria o disco. Si utiliza este canal en escenarios en que la aplicación está a punto de cerrarse, le recomendamos que introduzca algún retraso después de llamar a `Flush()`. La cantidad exacta de retraso que puede necesitar no es predecible. Depende de factores como el número de elementos o de instancias de `Transmission` que hay en la memoria, de cuántos están en el disco y cuántos se están transmitiendo al back-end y de si el canal está en medio de escenarios de retroceso exponenciales.

Si necesita realizar un vaciado sincrónico, le recomendamos que use `InMemoryChannel`.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="does-the-application-insights-channel-guarantee-telemetry-delivery-if-not-what-are-the-scenarios-in-which-telemetry-can-be-lost"></a>¿El canal de Application Insights garantiza la entrega de los datos de telemetría? Si no es así, ¿cuáles son los escenarios en los que se pueden perder los datos de telemetría?

La respuesta corta es que ninguno de los canales integrados ofrece una garantía de tipo de transacción de entrega de telemetría al back-end. El canal `ServerTelemetryChannel` es más avanzado comparado con `InMemoryChannel` en lo que respecta a la entrega confiable, pero solo hace un intento de mejor esfuerzo para enviar la telemetría. Aún así, igualmente se pueden perder datos de telemetría en diversas situaciones, incluidos los escenarios comunes siguientes:

1. Los elementos en la memoria se pierden cuando se bloquea la aplicación.

1. Los datos de telemetría se pierden durante largos períodos de problemas de red. Los datos de telemetría se almacenan en el disco local durante las interrupciones de red o cuando se producen problemas con el back-end de Application Insights. Sin embargo, se descartan los elementos anteriores a 48 horas.

1. Las ubicaciones predeterminadas del disco en las que se almacenan datos de telemetría en Windows son %LOCALAPPDATA% o %TEMP%. Normalmente, estas ubicaciones se encuentran en el entorno local de la máquina. Si la aplicación se migra físicamente desde una ubicación a otra, se pierde cualquier telemetría almacenada en la ubicación original.

1. En Web Apps en Windows, la ubicación de almacenamiento en disco predeterminada es D:\local\LocalAppData. Esta ubicación no se conserva. Se borra cuando se realizan reinicios de la aplicación, escalados horizontales y otras operaciones similares, lo que da lugar a la pérdida de cualquier telemetría almacenada. Puede reemplazar el valor predeterminado y especificar el almacenamiento en una ubicación persistente como D:\home. Sin embargo, dado que estas ubicaciones persistentes las proporciona el almacenamiento remoto, pueden ser lentas.

### <a name="does-servertelemetrychannel-work-on-systems-other-than-windows"></a>¿ServerTelemetryChannel funciona en sistemas distintos de Windows?

Aunque el nombre de su paquete y el espacio de nombres incluye "WindowsServer", este canal se admite en sistemas distintos de Windows, con la siguiente excepción. En los sistemas distintos de Windows, el canal no crea una carpeta de almacenamiento local de forma predeterminada. Debe crear una carpeta de almacenamiento local y configurar el canal para que la use. Una vez configurado el almacenamiento local, el canal funciona del mismo modo en todos los sistemas.

### <a name="does-the-sdk-create-temporary-local-storage-is-the-data-encrypted-at-storage"></a>¿El SDK crea almacenamiento local temporal? ¿Se cifran los datos en el almacenamiento?

El SDK almacena elementos de telemetría en el almacenamiento local cuando se producen problemas de red o hay una limitación. Estos datos no se cifran localmente.

En el caso de los sistemas Windows, el SDK crea automáticamente una carpeta temporal local en el directorio %LOCALAPPDATA% o e %TEMP% y restringe el acceso solo a los administradores y al usuario actual.

Para los sistemas que distintos de Windows, no se crea ningún almacenamiento local automáticamente mediante el SDK, por lo que no se almacena ningún dato localmente de forma predeterminada. Puede crear un directorio de almacenamiento usted mismo y configurar el canal para que lo utilice. En este caso, es su responsabilidad asegurarse de que se protege el directorio.
Obtenga más información sobre [la protección y la privacidad de los datos](data-retention-privacy.md#does-the-sdk-create-temporary-local-storage).

## <a name="open-source-sdk"></a>SDK de código abierto
Al igual que todos los SDK de Application Insights, los canales son código abierto. Puede leer el código y contribuir en este, o bien notificar problemas en [el repositorio oficial de GitHub](https://github.com/Microsoft/ApplicationInsights-dotnet).

## <a name="next-steps"></a>Pasos siguientes

* [Muestreo](./sampling.md)
* [Solución de problemas del SDK](./asp-net-troubleshoot-no-data.md)

