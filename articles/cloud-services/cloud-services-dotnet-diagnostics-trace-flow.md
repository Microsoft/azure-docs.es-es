---
title: Seguimiento del flujo en una aplicación de Cloud Services con Azure Diagnostics
titleSuffix: Azure Cloud Services
description: Agregar mensajes de seguimiento a una aplicación de Azure para facilitar la depuración, medición del rendimiento, supervisión, análisis del tráfico y mucho más.
services: cloud-services
documentationcenter: .net
author: tgore03
ms.service: cloud-services
ms.devlang: dotnet
ms.custom: devx-track-dotnet
ms.topic: article
ms.date: 02/20/2016
ms.author: tagore
ms.openlocfilehash: 8270766413729454181c461d469d49e418a1aa67
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88932311"
---
# <a name="trace-the-flow-of-a-cloud-services-application-with-azure-diagnostics"></a>Seguimiento del flujo en una aplicación de Cloud Services con Diagnósticos de Azure
El seguimiento es una manera de supervisar la ejecución de la aplicación mientras se está ejecutando. Puede usar las clases [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace), [System.Diagnostics.Debug](/dotnet/api/system.diagnostics.debug) y [System.Diagnostics.TraceSource](/dotnet/api/system.diagnostics.tracesource) para registrar información sobre errores y ejecución de la aplicaciones en registros, archivos de texto u otros dispositivos para su análisis posterior. Para obtener más información acerca del seguimiento, consulte [Seguimiento e instrumentación de aplicaciones](/dotnet/framework/debug-trace-profile/tracing-and-instrumenting-applications).

## <a name="use-trace-statements-and-trace-switches"></a>Uso de las instrucciones de seguimiento y los modificadores de seguimiento
Implemente el seguimiento en la aplicación de Cloud Services al agregar [DiagnosticMonitorTraceListener](/previous-versions/azure/reference/ee758610(v=azure.100)) a la configuración de la aplicación y realizar llamadas a System.Diagnostics.Trace o a System.Diagnostics.Debug en el código de aplicación. Use el archivo de configuración *app.config* para los roles de trabajo y *web.config* para los roles web. Cuando se crea un nuevo servicio hospedado mediante una plantilla de Visual Studio, Diagnósticos de Azure se agrega automáticamente al proyecto y DiagnosticMonitorTraceListener se agrega al archivo de configuración adecuado para los roles que se añadan.

Para más información acerca de cómo colocar instrucciones de seguimiento, consulte [ Procedimiento para agregar instrucciones de seguimiento al código de una aplicación](/dotnet/framework/debug-trace-profile/how-to-add-trace-statements-to-application-code).

Al colocar [modificadores de seguimiento](/dotnet/framework/debug-trace-profile/trace-switches) en el código, puede controlar si se realiza el seguimiento y cuál es su alcance. Con ello puede supervisar el estado de la aplicación en un entorno de producción. Esto es especialmente importante en una aplicación empresarial que usa varios componentes que se ejecutan en varios equipos. Para más información, vea: [Cómo: Procedimiento para crear, inicializar y configurar modificadores de seguimiento](/dotnet/framework/debug-trace-profile/how-to-create-initialize-and-configure-trace-switches).

## <a name="configure-the-trace-listener-in-an-azure-application"></a>Configuración de la escucha de seguimiento en una aplicación de Azure
Tendrá que configurar "agentes de escucha" para Trace, Debug y TraceSource, para recopilar y registrar los mensajes que se envían. Los agentes de escucha de recopilan, almacenan y enrutan los mensajes de seguimiento. Estos dirigen los resultados del seguimiento a un destino apropiado, como un registro, una ventana o un archivo de texto. Diagnósticos de Azure usa la clase [DiagnosticMonitorTraceListener](/previous-versions/azure/reference/ee758610(v=azure.100)) .

Antes de completar el siguiente procedimiento, tiene que inicializar el monitor de diagnóstico de Azure. Para ello, consulte [Habilitación de diagnósticos en Microsoft Azure](cloud-services-dotnet-diagnostics.md).

Tenga en cuenta que si usa las plantillas proporcionadas por Visual Studio, la configuración del agente de escucha se agrega automáticamente.

### <a name="add-a-trace-listener"></a>Incorporación de un agente de escucha de seguimiento
1. Abra el archivo web.config o app.config para su rol.
2. Agregue el siguiente código al archivo. Cambie el atributo Version para usar el número de versión del ensamblado al que se hace referencia. La versión de ensamblado no cambia necesariamente con cada versión del SDK de Azure, a menos que haya actualizaciones.
   
    ```
    <system.diagnostics>
        <trace>
            <listeners>
                <add type="Microsoft.WindowsAzure.Diagnostics.DiagnosticMonitorTraceListener,
                  Microsoft.WindowsAzure.Diagnostics,
                  Version=2.8.0.0,
                  Culture=neutral,
                  PublicKeyToken=31bf3856ad364e35"
                  name="AzureDiagnostics">
                    <filter type="" />
                </add>
            </listeners>
        </trace>
    </system.diagnostics>
    ```
   > [!IMPORTANT]
   > Asegúrese de que tiene una referencia de proyecto al ensamblado Microsoft.WindowsAzure.Diagnostics. Actualice el número de versión en el xml anterior para que coincida con la versión del ensamblado de Microsoft.WindowsAzure.Diagnostics al que se hace referencia.
   > 
   > 
3. Guarde el archivo de configuración.

Para más información sobre los agentes de escucha, vea [Agentes de escucha de seguimiento](/dotnet/framework/debug-trace-profile/trace-listeners).

Después de completar los pasos para agregar el agente de escucha, puede agregar instrucciones de seguimiento al código.

### <a name="to-add-trace-statement-to-your-code"></a>Para agregar instrucciones de seguimiento al código
1. Abra un archivo de origen para la aplicación. Por ejemplo, el archivo \<RoleName>.cs para el rol de trabajo o el rol web.
2. Agregue la siguiente directiva using si aún no lo ha hecho:
    ```
        using System.Diagnostics;
    ```
3. Agregue instrucciones de seguimiento en donde desee capturar información sobre el estado de la aplicación. Puede usar diversos métodos para dar formato al resultado de la instrucción de seguimiento. Para más información, vea: [Cómo: Procedimiento para agregar instrucciones de seguimiento al código de una aplicación](/dotnet/framework/debug-trace-profile/how-to-add-trace-statements-to-application-code).
4. Guarde el archivo de origen.




