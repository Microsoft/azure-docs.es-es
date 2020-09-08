---
title: Procesamiento de la fuente de cambios en Azure Blob Storage (versión preliminar) | Microsoft Docs
description: Obtenga información sobre cómo procesar los registros de fuente de cambios en una aplicación cliente de .NET.
author: normesta
ms.author: normesta
ms.date: 06/18/2020
ms.topic: article
ms.service: storage
ms.subservice: blobs
ms.reviewer: sadodd
ms.custom: devx-track-csharp
ms.openlocfilehash: cc70204a56c3a853c6c6c7f0e1573c71854c0f19
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018967"
---
# <a name="process-change-feed-in-azure-blob-storage-preview"></a>Procesamiento de la fuente de cambios en Azure Blob Storage (versión preliminar)

La fuente de cambios proporciona registros de transacciones de todos los cambios que se producen en los blobs y en los metadatos de blobs de la cuenta de almacenamiento. En este artículo se muestra cómo leer los registros de la fuente de cambios con la biblioteca de procesadores de la fuente de cambios de blob.

Para más información sobre la fuente de cambios, consulte [Fuente de cambios en Azure Blob Storage (versión preliminar)](storage-blob-change-feed.md).

> [!NOTE]
> La fuente de cambios está en versión preliminar pública y se encuentra disponible en regiones limitadas. Para más información sobre esta característica junto con limitaciones y problemas conocidos, consulte [Compatibilidad con la fuente de cambios en Azure Blob Storage](storage-blob-change-feed.md). La biblioteca de procesadores de la fuente de cambios está sujeta a cambios entre este momento y cuando esta biblioteca esté disponible con carácter general.

## <a name="get-the-blob-change-feed-processor-library"></a>Obtención de la biblioteca de procesadores de la fuente de cambios de blob

1. Abra una ventana de comando (por ejemplo: Windows PowerShell).
2. En el directorio del proyecto, instale el [paquete NuGet **Azure.Storage.Blobs.Changefeed**](https://www.nuget.org/packages/Azure.Storage.Blobs.ChangeFeed/).

```console
dotnet add package Azure.Storage.Blobs --version 12.5.1
dotnet add package Azure.Storage.Blobs.ChangeFeed --version 12.0.0-preview.4
```
## <a name="read-records"></a>Lectura de registros

> [!NOTE]
> La fuente de cambios es una entidad inmutable y de solo lectura en la cuenta de almacenamiento. Cualquier número de aplicaciones puede leer y procesar la fuente de cambios de manera simultáneamente y de manera independiente a su comodidad. Los registros no se quitan de la fuente de cambios cuando una aplicación los lee. El estado de lectura o de iteración de cada lector de consumo es independiente y solo lo mantiene la aplicación.

En este ejemplo se recorren en iteración todos los registros de la fuente de cambios, se agregan a una lista y, a continuación, se devuelve esa lista al autor de la llamada.
 
```csharp
public async Task<List<BlobChangeFeedEvent>> ChangeFeedAsync(string connectionString)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();

    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    // Get all the events in the change feed. 
    await foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedClient.GetChangesAsync())
    {
        changeFeedEvents.Add(changeFeedEvent);
    }

    return changeFeedEvents;
}
```

En este ejemplo se imprimen en la consola algunos valores de cada registro de la lista. 

```csharp
public void showEventData(List<BlobChangeFeedEvent> changeFeedEvents)
{
    foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedEvents)
    {
        string subject = changeFeedEvent.Subject;
        string eventType = changeFeedEvent.EventType.ToString();
        string api = changeFeedEvent.EventData.Api;

        Console.WriteLine("Subject: " + subject + "\n" +
        "Event Type: " + eventType + "\n" +
        "Api: " + api);
    }
}
```

## <a name="resume-reading-records-from-a-saved-position"></a>Reanudación de la lectura de registros a partir de una posición guardada

Puede optar por guardar la posición de lectura en la fuente de cambios y, después, reanudar la iteración de los registros en el futuro. Puede obtener el cursor de la fuente de cambios para guardar la posición de lectura. El cursor es una **string** y la aplicación puede guardarla de cualquier forma que funcione para el diseño de la aplicación (por ejemplo, en un archivo o base de datos).

En este ejemplo se recorren en iteración todos los registros de la fuente de cambios, se agregan a una lista y se guarda el cursor. La lista y el cursor se devuelven al autor de la llamada. 

```csharp
public async Task<(string, List<BlobChangeFeedEvent>)> ChangeFeedResumeWithCursorAsync
    (string connectionString,  string cursor)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();
    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    IAsyncEnumerator<Page<BlobChangeFeedEvent>> enumerator = changeFeedClient
        .GetChangesAsync(continuation: cursor)
        .AsPages(pageSizeHint: 10)
        .GetAsyncEnumerator();

    await enumerator.MoveNextAsync();

    foreach (BlobChangeFeedEvent changeFeedEvent in enumerator.Current.Values)
    {
    
        changeFeedEvents.Add(changeFeedEvent);             
    }
    
    // Update the change feed cursor.  The cursor is not required to get each page of events,
    // it is intended to be saved and used to resume iterating at a later date.
    cursor = enumerator.Current.ContinuationToken;
    return (cursor, changeFeedEvents);
}
```

## <a name="stream-processing-of-records"></a>Flujo del procesamiento de los registros

Puede elegir procesar los registros de la fuente de cambios a medida que se confirmen en la fuente de cambios. Consulte las [especificaciones](storage-blob-change-feed.md#specifications). Los eventos de cambios se publican en la fuente de cambios en un período de 60 segundos de media. Se recomienda que sondee los nuevos cambios con este plazo en mente al especificar el intervalo de sondeo.

Este ejemplo sondea periódicamente los cambios.  Si existen cambios en los registros, este código procesa dichos registros y guarda el cursor de la fuente de cambios. De este modo, si el proceso se detiene y, a continuación, se vuelve a iniciar, la aplicación puede usar el cursor para reanudar el procesamiento de los registros en el punto en que se quedó por última vez. En este ejemplo se guarda el cursor en un archivo de configuración de la aplicación local, pero su aplicación puede guardarlo en cualquier formato que sea más adecuado para su caso. 

```csharp
public async Task ChangeFeedStreamAsync
    (string connectionString, int waitTimeMs, string cursor)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();

    while (true)
    {
        IAsyncEnumerator<Page<BlobChangeFeedEvent>> enumerator = changeFeedClient
        .GetChangesAsync(continuation: cursor).AsPages().GetAsyncEnumerator();

        while (true) 
        {
            var result = await enumerator.MoveNextAsync();

            if (result)
            {
                foreach (BlobChangeFeedEvent changeFeedEvent in enumerator.Current.Values)
                {
                    string subject = changeFeedEvent.Subject;
                    string eventType = changeFeedEvent.EventType.ToString();
                    string api = changeFeedEvent.EventData.Api;

                    Console.WriteLine("Subject: " + subject + "\n" +
                        "Event Type: " + eventType + "\n" +
                        "Api: " + api);
                }
            
                // helper method to save cursor. 
                SaveCursor(enumerator.Current.ContinuationToken);
            }
            else
            {
                break;
            }

        }
        await Task.Delay(waitTimeMs);
    }

}

public void SaveCursor(string cursor)
{
    System.Configuration.Configuration config = 
        ConfigurationManager.OpenExeConfiguration
        (ConfigurationUserLevel.None);

    config.AppSettings.Settings.Clear();
    config.AppSettings.Settings.Add("Cursor", cursor);
    config.Save(ConfigurationSaveMode.Modified);
}
```

## <a name="reading-records-within-a-time-range"></a>Lectura de registros dentro de un intervalo de tiempo

Puede leer los registros que se encuentran dentro de un intervalo de tiempo específico. En este ejemplo se recorren en iteración todos los registros de la fuente de cambios que se encuentran entre las 15:00 del 2 de marzo de 2020 y las 2:00 del 7 de agosto de 2020, se agregan a una lista y, a continuación, se devuelve esa lista al autor de la llamada.

### <a name="selecting-segments-for-a-time-range"></a>Selección de segmentos para un intervalo de tiempo

```csharp
public async Task<List<BlobChangeFeedEvent>> ChangeFeedBetweenDatesAsync(string connectionString)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();
    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    // Create the start and end time.  The change feed client will round start time down to
    // the nearest hour, and round endTime up to the next hour if you provide DateTimeOffsets
    // with minutes and seconds.
    DateTimeOffset startTime = new DateTimeOffset(2020, 3, 2, 15, 0, 0, TimeSpan.Zero);
    DateTimeOffset endTime = new DateTimeOffset(2020, 8, 7, 2, 0, 0, TimeSpan.Zero);

    // You can also provide just a start or end time.
    await foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedClient.GetChangesAsync(
        start: startTime,
        end: endTime))
    {
        changeFeedEvents.Add(changeFeedEvent);
    }

    return changeFeedEvents;
}
```

La hora de inicio que proporcione se redondea hacia abajo a la hora más cercana y la hora de finalización se redondea hacia arriba a la hora más cercana. Es posible que los usuarios vean los eventos que se produjeron antes de la hora de inicio y después de la hora de finalización. También es posible que no aparezcan algunos eventos que se producen entre las horas de inicio y finalización. Esto se debe a que los eventos se pueden registrar durante la hora anterior a la hora de inicio o durante la hora posterior a la hora de finalización.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los registros de la fuente de cambios. Consulte [Fuente de cambios en Azure Blob Storage (versión preliminar)](storage-blob-change-feed.md)
