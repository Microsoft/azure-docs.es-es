---
title: Sondeo de operaciones de larga duración | Microsoft Docs
description: Azure Media Services ofrece determinadas API que envían solicitudes a Media Services para iniciar operaciones (por ejemplo, crear, iniciar, detener o eliminar un canal). Estas operaciones son de larga duración. En este tema se muestra cómo sondear las operaciones de larga duración.
services: media-services
documentationcenter: ''
author: Juliako
writer: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 44cecbd8d2cdc95e342d7aaf2b33f6cc0192e182
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89262042"
---
# <a name="delivering-live-streaming-with-azure-media-services"></a>Entrega de transmisión en directo con Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general

Microsoft Azure Media Services ofrece determinadas API que envían solicitudes a los Media Services para iniciar operaciones (por ejemplo, crear, iniciar, detener o eliminar un canal). Estas son operaciones de larga duración.

El SDK .NET de Media Services proporciona las API que envían la solicitud y esperan a que la operación se complete (internamente, las API sondean el progreso de la operación a intervalos determinados). Por ejemplo, cuando se llama a channel.Start(), se devuelve el método después de que se inicie el canal. También puede usar la versión asincrónica: await channel.StartAsync() (para obtener información sobre el patrón asincrónico basado en tareas, consulte [TAP](./media-services-mes-schema.md)). Las API que envían una solicitud de operación y, después, sondean el estado hasta que se completa la operación se llaman “métodos de sondeo”. Estos métodos (sobre todo la versión asincrónica) se recomiendan para las aplicaciones cliente enriquecidas y para servicios con estado.

En ciertos casos, una aplicación no puede esperar a una solicitud HTTP de ejecución prolongada y desea sondear el progreso de la operación manualmente. Un ejemplo típico sería un explorador que interactúa con un servicio web sin estado: cuando el explorador solicita crear un canal, el servicio web inicia una operación de ejecución prolongada y devuelve el identificador de la operación al explorador. A continuación, el explorador puede solicitar al servicio web que obtenga el estado de la operación en función del identificador. El SDK .NET de Media Services proporciona las API que son útiles para este escenario. Estas API se denominan “métodos sin sondeo”.
Los "métodos sin sondeo" tienen el patrón de nomenclatura siguiente: Send*OperationName*Operation (por ejemplo, SendCreateOperation). Los métodos Send*OperationName*Operation devuelven el objeto **IOperation** ; el objeto devuelto contiene información que puede usarse para realizar un seguimiento de la operación. Los métodos Send*OperationName*OperationAsync devuelven **Task\<IOperation>** .

Las siguientes clases admiten métodos sin sondeo actualmente:  **Channel**, **StreamingEndpoint** y **Program**.

Para sondear el estado de la operación, use el método **GetOperation** en la clase **OperationBaseCollection**. Use los siguientes intervalos para comprobar el estado de la operación: para las operaciones **Channel** y **StreamingEndpoint**, use 30 segundos; para las operaciones **Program**, use 10 segundos.

## <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio

Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md).

## <a name="example"></a>Ejemplo

En el ejemplo siguiente se define una clase llamada **ChannelOperations**. Esta definición de clase puede constituir un punto de partida para la definición de clase del servicio web. Para simplificar, en los siguientes ejemplos se usan las versiones no asincrónicas de los métodos.

También se muestra cómo el cliente puede usar esta clase.

### <a name="channeloperations-class-definition"></a>Definición de la clase ChannelOperations

```csharp
using Microsoft.WindowsAzure.MediaServices.Client;
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Net;

/// <summary> 
/// The ChannelOperations class only implements 
/// the Channel’s creation operation. 
/// </summary> 
public class ChannelOperations
{
    // Read values from the App.config file.
    private static readonly string _AADTenantDomain =
        ConfigurationManager.AppSettings["AMSAADTenantDomain"];
    private static readonly string _RESTAPIEndpoint =
        ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
    private static readonly string _AMSClientId =
        ConfigurationManager.AppSettings["AMSClientId"];
    private static readonly string _AMSClientSecret =
        ConfigurationManager.AppSettings["AMSClientSecret"];

    // Field for service context.
    private static CloudMediaContext _context = null;

    public ChannelOperations()
    {
        AzureAdTokenCredentials tokenCredentials = 
            new AzureAdTokenCredentials(_AADTenantDomain,
                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                AzureEnvironments.AzureCloudEnvironment);

        var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
    }

    /// <summary>  
    /// Initiates the creation of a new channel.  
    /// </summary>  
    /// <param name="channelName">Name to be given to the new channel</param>  
    /// <returns>  
    /// Operation Id for the long running operation being executed by Media Services. 
    /// Use this operation Id to poll for the channel creation status. 
    /// </returns> 
    public string StartChannelCreation(string channelName)
    {
        var operation = _context.Channels.SendCreateOperation(
            new ChannelCreationOptions
            {
                Name = channelName,
                Input = CreateChannelInput(),
                Preview = CreateChannelPreview(),
                Output = CreateChannelOutput()
            });

        return operation.Id;
    }

    /// <summary> 
    /// Checks if the operation has been completed. 
    /// If the operation succeeded, the created channel Id is returned in the out parameter.
    /// </summary> 
    /// <param name="operationId">The operation Id.</param> 
    /// <param name="channel">
    /// If the operation succeeded, 
    /// the created channel Id is returned in the out parameter.</param>
    /// <returns>Returns false if the operation is still in progress; otherwise, true.</returns> 
    public bool IsCompleted(string operationId, out string channelId)
    {
        IOperation operation = _context.Operations.GetOperation(operationId);
        bool completed = false;

        channelId = null;

        switch (operation.State)
        {
            case OperationState.Failed:
                // Handle the failure. 
                // For example, throw an exception. 
                // Use the following information in the exception: operationId, operation.ErrorMessage.
                break;
            case OperationState.Succeeded:
                completed = true;
                channelId = operation.TargetEntityId;
                break;
            case OperationState.InProgress:
                completed = false;
                break;
        }
        return completed;
    }

    private static ChannelInput CreateChannelInput()
    {
        return new ChannelInput
        {
            StreamingProtocol = StreamingProtocol.RTMP,
            AccessControl = new ChannelAccessControl
            {
                IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "TestChannelInput001",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    }
            }
        };
    }

    private static ChannelPreview CreateChannelPreview()
    {
        return new ChannelPreview
        {
            AccessControl = new ChannelAccessControl
            {
                IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "TestChannelPreview001",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    }
            }
        };
    }

    private static ChannelOutput CreateChannelOutput()
    {
        return new ChannelOutput
        {
            Hls = new ChannelOutputHls { FragmentsPerSegment = 1 }
        };
    }
}
```

### <a name="the-client-code"></a>El código de cliente

```csharp
ChannelOperations channelOperations = new ChannelOperations();
string opId = channelOperations.StartChannelCreation("MyChannel001");

string channelId = null;
bool isCompleted = false;

while (isCompleted == false)
{
    System.Threading.Thread.Sleep(TimeSpan.FromSeconds(30));
    isCompleted = channelOperations.IsCompleted(opId, out channelId);
}

// If we got here, we should have the newly created channel id.
Console.WriteLine(channelId);
```

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]