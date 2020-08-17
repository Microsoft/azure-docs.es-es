---
title: Creación y ejecución de pruebas de disponibilidad personalizadas mediante Azure Functions
description: En este documento se explica cómo crear una función de Azure con TrackAvailability() que se ejecutará periódicamente según la configuración especificada en la función TimerTrigger. Los resultados de esta prueba se enviarán al recurso de Application Insights, donde podrá consultar los datos de los resultados de disponibilidad y generar alertas sobre ellos. Las pruebas personalizadas le permitirán escribir pruebas de disponibilidad más complejas de las que puede crear con la interfaz de usuario del portal, supervisar una aplicación dentro de la red virtual de Azure, cambiar la dirección del punto de conexión o crear una prueba de disponibilidad si no hay ninguna disponible en su región.
ms.topic: conceptual
author: mrbullwinkle
ms.author: mbullwin
ms.date: 05/04/2020
ms.openlocfilehash: e2603d921973aefdcc1a6f4a76bdf70d69dcb68f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320636"
---
# <a name="create-and-run-custom-availability-tests-using-azure-functions"></a>Creación y ejecución de pruebas de disponibilidad personalizadas mediante Azure Functions

En este artículo se explica cómo crear una función de Azure con TrackAvailability() que se ejecutará periódicamente según la configuración especificada en la función TimerTrigger con lógica de negocios propia. Los resultados de esta prueba se enviarán al recurso de Application Insights, donde podrá consultar los datos de los resultados de disponibilidad y generar alertas sobre ellos. Esto le permite crear pruebas personalizadas de forma parecida a lo que puede hacer mediante [Supervisión de la disponibilidad](./monitor-web-app-availability.md) en el portal. Las pruebas personalizadas le permitirán escribir pruebas de disponibilidad más complejas de las que puede crear con la interfaz de usuario del portal, supervisar una aplicación dentro de la red virtual de Azure, cambiar la dirección del punto de conexión o crear una prueba de disponibilidad incluso si esta característica no está disponible en su región.

> [!NOTE]
> Este ejemplo se ha diseñado únicamente para mostrar la mecánica de cómo actúa la llamada a la API de TrackAvailability() en una función de Azure. No muestra cómo se escribe el código de prueba HTTP o la lógica de negocios subyacentes que se necesitarían para convertir esta llamada en una prueba de disponibilidad totalmente funcional. De forma predeterminada, si recorre este ejemplo, creará una prueba de disponibilidad que siempre generará un error.

## <a name="create-timer-triggered-function"></a>Creación de una función desencadenada por el temporizador

- Si tiene un recurso de Application Insights:
    - De forma predeterminada Azure Functions crea un recurso de Application Insights, pero si desea usar uno de los recursos que ya ha creado, deberá especificarlo durante la creación.
    - Siga las instrucciones sobre cómo [crear un recurso de Azure Functions y una función desencadenada por el temporizador](../../azure-functions/functions-create-scheduled-function.md) (detener antes de la limpieza) con las siguientes opciones.
        -  Seleccione la pestaña **Supervisión** situada cerca de la parte superior.

            ![ Creación de una aplicación de Azure Functions con su propio recurso de App Insights](media/availability-azure-functions/create-function-app.png)

        - Seleccione el cuadro desplegable Application Insights y escriba o seleccione el nombre del recurso.

            ![Seleccionar recurso existente de Application Insights](media/availability-azure-functions/app-insights-resource.png)

        - Seleccionar **Revisar y crear**.
- Si aún no tiene un recurso de Application Insights creado para la función desencadenada por el temporizador:
    - De forma predeterminada, si crea una aplicación de Azure Functions, se creará un recurso de Application Insights.
    - Siga las instrucciones sobre cómo [crear un recurso de Azure Functions y una función desencadenada por el temporizador](../../azure-functions/functions-create-scheduled-function.md) (detener antes de la limpieza).

## <a name="sample-code"></a>Código de ejemplo

Copie el código siguiente en el archivo run.csx (esto reemplazará el código ya existente). Para ello, vaya a la aplicación Azure Functions y seleccione la función de desencadenador de temporizador a la izquierda.

>[!div class="mx-imgBorder"]
>![Archivo run.csx de la función de Azure en Azure Portal](media/availability-azure-functions/runcsx.png)

> [!NOTE]
> Para la dirección del punto de conexión debería usar: `EndpointAddress= https://dc.services.visualstudio.com/v2/track`. A menos que el recurso se encuentre en una región como Azure Government o Azure China, en cuyo caso consulte este artículo sobre [sustitución de los puntos de conexión predeterminados](./custom-endpoints.md#regions-that-require-endpoint-modification), seleccione el punto de conexión adecuado del canal de telemetría de su región.

```C#
#load "runAvailabilityTest.csx"
 
using System;
using System.Diagnostics;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
 
// The Application Insights Instrumentation Key can be changed by going to the overview page of your Function App, selecting configuration, and changing the value of the APPINSIGHTS_INSTRUMENTATIONKEY Application setting.
// DO NOT replace the code below with your instrumentation key, the key's value is pulled from the environment variable/application setting key/value pair.
private static readonly string instrumentationKey = Environment.GetEnvironmentVariable("APPINSIGHTS_INSTRUMENTATIONKEY");
 
//[CONFIGURATION_REQUIRED]
// If your resource is in a region like Azure Government or Azure China, change the endpoint address accordingly.
// Visit https://docs.microsoft.com/azure/azure-monitor/app/custom-endpoints#regions-that-require-endpoint-modification for more details.
private const string EndpointAddress = "https://dc.services.visualstudio.com/v2/track";
 
private static readonly TelemetryConfiguration telemetryConfiguration = new TelemetryConfiguration(instrumentationKey, new InMemoryChannel { EndpointAddress = EndpointAddress });
private static readonly TelemetryClient telemetryClient = new TelemetryClient(telemetryConfiguration);
 
public async static Task Run(TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"Entering Run at: {DateTime.Now}");
 
    if (myTimer.IsPastDue)
    {
        log.LogWarning($"[Warning]: Timer is running late! Last ran at: {myTimer.ScheduleStatus.Last}");
    }
 
    // [CONFIGURATION_REQUIRED] provide {testName} accordingly for your test function
    string testName = "AvailabilityTestFunction";
 
    // REGION_NAME is a default environment variable that comes with App Service
    string location = Environment.GetEnvironmentVariable("REGION_NAME");
 
    log.LogInformation($"Executing availability test run for {testName} at: {DateTime.Now}");
    string operationId = Guid.NewGuid().ToString("N");
 
    var availability = new AvailabilityTelemetry
    {
        Id = operationId,
        Name = testName,
        RunLocation = location,
        Success = false
    };
 
    var stopwatch = new Stopwatch();
    stopwatch.Start();
 
    try
    {
        await RunAvailbiltyTestAsync(log);
        availability.Success = true;
    }
    catch (Exception ex)
    {
        availability.Message = ex.Message;
 
        var exceptionTelemetry = new ExceptionTelemetry(ex);
        exceptionTelemetry.Context.Operation.Id = operationId;
        exceptionTelemetry.Properties.Add("TestName", testName);
        exceptionTelemetry.Properties.Add("TestLocation", location);
        telemetryClient.TrackException(exceptionTelemetry);
    }
    finally
    {
        stopwatch.Stop();
        availability.Duration = stopwatch.Elapsed;
        availability.Timestamp = DateTimeOffset.UtcNow;
 
        telemetryClient.TrackAvailability(availability);
        // call flush to ensure telemetry is sent
        telemetryClient.Flush();
    }
}

```

En la derecha, debajo de Ver archivos, seleccione **Agregar**. Llame al nuevo archivo **function.proj** con la configuración siguiente.

```C#
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.8.2" /> <!-- Ensure you’re using the latest version -->
    </ItemGroup>
</Project>

```

>[!div class="mx-imgBorder"]
>![A la derecha, seleccione Agregar. Asigne al archivo el nombre function.proj.](media/availability-azure-functions/addfile.png)

En la derecha, debajo de Ver archivos, seleccione **Agregar**. Llame al nuevo archivo **runAvailabilityTest.csx** con la configuración siguiente.

```C#
public async static Task RunAvailbiltyTestAsync(ILogger log)
{
    // Add your business logic here.
    throw new NotImplementedException();
}

```

## <a name="check-availability"></a>Comprobación de la disponibilidad

Para asegurarse de que todo funciona, puede examinar el gráfico de la pestaña Disponibilidad del recurso de Application Insights.

> [!NOTE]
> Si ha implementado su propia lógica de negocios en runAvailabilityTest.csx, verá resultados correctos como en las capturas de pantallas siguientes; si no lo hizo, verá resultados con errores. Las pruebas creadas con `TrackAvailability()` aparecerán con **CUSTOM** junto al nombre de la prueba.

>[!div class="mx-imgBorder"]
>![Pestaña Disponibilidad con resultados correctos](media/availability-azure-functions/availability-custom.png)

Para ver los detalles de la transacción completa, seleccione **Correcto** o **Erróneo** en Aumentar detalle y, a continuación, seleccione un ejemplo. También puede obtener los detalles de la transacción de un extremo a otro seleccionando un punto de datos en el gráfico.

>[!div class="mx-imgBorder"]
>![Selección de una prueba de disponibilidad de ejemplo](media/availability-azure-functions/sample.png)

>[!div class="mx-imgBorder"]
>![Detalles de la transacción completa](media/availability-azure-functions/end-to-end.png)

Si ejecutó todo tal cual (sin agregar lógica de negocios), verá que se produjo un error en la prueba.

## <a name="query-in-logs-analytics"></a>Consulta en Registros (Analytics)

Puede usar registros (Analytics) para ver los resultados de disponibilidad, dependencias, etc. Para más información acerca de los registros, visite [Introducción a las consultas de registros](../log-query/log-query-overview.md).

>[!div class="mx-imgBorder"]
>![Resultados de disponibilidad](media/availability-azure-functions/availabilityresults.png)

>[!div class="mx-imgBorder"]
>![Dependencias](media/availability-azure-functions/dependencies.png)

## <a name="next-steps"></a>Pasos siguientes

- [Mapa de aplicación](./app-map.md)
- [Diagnósticos de transacción](./transaction-diagnostics.md)

