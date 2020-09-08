---
title: Lectura de la entrada en cualquier formato mediante los deserializadores personalizados de .NET en Azure Stream Analytics
description: En este artículo se explica el formato de serialización y las interfaces que definen los deserializadores de .NET personalizados para trabajos perimetrales y en la nube de Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 1/28/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 4616f6c567b0bba13fe04aed56fd5e4ddc293f90
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008393"
---
# <a name="read-input-in-any-format-using-net-custom-deserializers"></a>Lectura de entradas en cualquier formato mediante deserializadores personalizados

Los deserializadores de .NET personalizados permiten que el trabajo de Azure Stream Analytics lea los datos en formatos distintos de los tres [formatos de datos integrados](stream-analytics-parsing-json.md). En este artículo se explica el formato de serialización y las interfaces que definen los deserializadores de .NET personalizados para trabajos perimetrales y en la nube de Azure Stream Analytics. También hay deserializadores de ejemplo para el búfer de protocolo y el formato CSV.

## <a name="net-custom-deserializer"></a>Deserializador personalizado de .NET

Los siguientes ejemplos de código son las interfaces que definen al deserializador personalizado e implementan `StreamDeserializer<T>`.

`UserDefinedOperator` es la clase base para todos los operadores de streaming personalizados. Este inicializa `StreamingContext`, que proporciona el contexto que incluye el mecanismo para publicar diagnósticos para los que necesitará depurar cualquier problema con el deserializador.

```csharp
    public abstract class UserDefinedOperator
    {
        public abstract void Initialize(StreamingContext streamingContext);
    }
```

El siguiente fragmento de código es la deserialización de los datos de streaming. 

Los errores que se pueden omitir deben emitirse mediante el elemento `IStreamingDiagnostics` pasado a través del método de inicialización de `UserDefinedOperator`. Todas las excepciones se tratarán como errores y se volverá a crear el deserializador. Después de un número determinado de errores, el trabajo pasará a un estado de error.

`StreamDeserializer<T>` deserializa una secuencia en un objeto de tipo `T`. Se deben cumplir las condiciones siguientes:

1. T es una clase o una estructura.
1. Todos los campos públicos de T son
    1. Uno de los valores [sbyte, byte, corto, ushort, entero, uint, largo, DateTime, cadena, float, doble] o sus equivalentes que aceptan valores NULL.
    1. Otra estructura o clase que sigue las mismas reglas.
    1. Matriz de tipo `T2` que sigue las mismas reglas.
    1. IList`T2` donde T2 sigue las mismas reglas.
    1. No tiene ningún tipo recursivo.

El parámetro `stream` es la secuencia que contiene el objeto serializado. `Deserialize` devuelve una colección de instancias de `T`.

```csharp
    public abstract class StreamDeserializer<T> : UserDefinedOperator
    {
        public abstract IEnumerable<T> Deserialize(Stream stream);
    }
```

`StreamingContext` proporciona el contexto que incluye el mecanismo para publicar diagnósticos para el operador de usuario.

```csharp
    public abstract class StreamingContext
    {
        public abstract StreamingDiagnostics Diagnostics { get; }
    }
```

`StreamingDiagnostics` es el diagnóstico de los operadores definidos por el usuario, incluidos el serializador, el deserializador y las funciones definidas por el usuario.

`WriteError` escribe un mensaje de error en los registros de recursos y envía el error al diagnóstico.

`briefMessage` es un breve mensaje de error. Este mensaje se muestra en los diagnósticos y el equipo del producto lo usa para la depuración. No incluya información confidencial y mantenga el mensaje menor de 200 caracteres.

`detailedMessage` es un mensaje de error detallado que solo se agrega a los registros de recursos en el almacenamiento. El mensaje debe tener menos de 2000 caracteres.

```csharp
    public abstract class StreamingDiagnostics
    {
        public abstract void WriteError(string briefMessage, string detailedMessage);
    }
```

## <a name="deserializer-examples"></a>Ejemplos de deserializador

En esta sección se muestra cómo codificar deserializadores personalizados para Protobuf y CSV. Para ver ejemplos adicionales, como el formato AVRO para la captura de centros de eventos, visite [Azure Stream Analytics en GitHub](https://github.com/Azure/azure-stream-analytics/tree/master/CustomDeserializers).

### <a name="protocol-buffer-protobuf-format"></a>Formato de búfer de protocolo (Protobuf)

Este es un ejemplo que usa el formato de búfer de protocolo.

Asuma la siguiente definición de búfer de protocolo.

```proto
syntax = "proto3";
// protoc.exe from nuget "Google.Protobuf.Tools" is used to generate .cs file from this schema definition.
// Run below command to generate the csharp class
// protoc.exe --csharp_out=. MessageBodyProto.proto

package SimulatedTemperatureSensor;
message MessageBodyProto {
    message Ambient {
      double temperature = 1;
      int64 humidity = 2;
    }

    message Machine {
      double temperature = 1;
      double pressure = 2;
    }

    Machine machine = 1;
    Ambient ambient = 2;
    string timeCreated = 3;
}
```

La ejecución de `protoc.exe` desde el paquete NuGet **Google.Protobuf.Tools** genera un archivo. CS con la definición. El archivo generado no se muestra aquí. Debe asegurarse de que la versión de Protobuf Nuget que usa en el proyecto Stream Analytics coincide con la versión de Protobuf que se usó para generar la entrada. 

El siguiente fragmento de código es la implementación del deserializador, suponiendo que el archivo generado se incluye en el proyecto. Esta implementación es simplemente un contenedor fino sobre el archivo generado.

```csharp
    public class MessageBodyDeserializer : StreamDeserializer<SimulatedTemperatureSensor.MessageBodyProto>
    {
        public override IEnumerable<SimulatedTemperatureSensor.MessageBodyProto> Deserialize(Stream stream)
        {
            while (stream.Position < stream.Length)
            {
                yield return SimulatedTemperatureSensor.MessageBodyProto.Parser.ParseDelimitedFrom(stream);
            }
        }

        public override void Initialize(StreamingContext streamingContext)
        {
        }
    }
```

### <a name="csv"></a>CSV

El siguiente fragmento de código es un deserializador de CSV sencillo que también muestra errores propagados.

```csharp
using System.Collections.Generic;
using System.IO;

using Microsoft.Azure.StreamAnalytics;
using Microsoft.Azure.StreamAnalytics.Serialization;

namespace ExampleCustomCode.Serialization
{
    public class CustomCsvDeserializer : StreamDeserializer<CustomEvent>
    {
        private StreamingDiagnostics streamingDiagnostics;

        public override IEnumerable<CustomEvent> Deserialize(Stream stream)
        {
            using (var sr = new StreamReader(stream))
            {
                string line = sr.ReadLine();
                while (line != null)
                {
                    if (line.Length > 0 && !string.IsNullOrWhiteSpace(line))
                    {
                        string[] parts = line.Split(',');
                        if (parts.Length != 3)
                        {
                            streamingDiagnostics.WriteError("Did not get expected number of columns", $"Invalid line: {line}");
                        }
                        else
                        {
                            yield return new CustomEvent()
                            {
                                Column1 = parts[0],
                                Column2 = parts[1],
                                Column3 = parts[2]
                            };
                        }
                    }

                    line = sr.ReadLine();
                }
            }
        }

        public override void Initialize(StreamingContext streamingContext)
        {
            this.streamingDiagnostics = streamingContext.Diagnostics;
        }
    }

    public class CustomEvent
    {
        public string Column1 { get; set; }

        public string Column2 { get; set; }

        public string Column3 { get; set; }
    }
}

```

## <a name="serialization-format-for-rest-apis"></a>Formato de serialización para las API REST

Cada entrada de Stream Analytics tiene un **formato de serialización**. Para obtener más información sobre las opciones de entrada, consulte la documentación sobre la [API REST de entradas](https://docs.microsoft.com/rest/api/streamanalytics/stream-analytics-input).

El siguiente código de JavaScript es un ejemplo del formato de serialización del deserializador de .NET cuando se usa la API REST:

```javascript
{    
   "properties":{    
      "type":"stream",  
      "serialization":{    
         "type":"CustomCLR",  
         "properties":{    
            "serializationDllPath":"<path to the dll inside UserCustomCode\CLR\ folder>", 
            "serializationClassName":"<Full name of the deserializer class name>" 
         }  
      }
   }  
}  
```

`serializationClassName` debe ser una clase que implemente `StreamDeserializer<T>`. Estos pasos se describen en la siguiente sección.

## <a name="region-support"></a>Regiones admitidas

Esta característica está disponible en las siguientes regiones:

* Centro-Oeste de EE. UU.
* Norte de Europa
* Este de EE. UU.
* Oeste de EE. UU.
* Este de EE. UU. 2
* Oeste de Europa

Puede [solicitar al soporte técnico](https://aka.ms/ccodereqregion) que agreguen regiones adicionales.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="when-will-this-feature-be-available-in-all-azure-regions"></a>¿Cuándo estará disponible esta característica en todas las regiones de Azure?

Esta característica está disponible en [6 regiones](https://docs.microsoft.com/azure/stream-analytics/custom-deserializer-examples#region-support). Si está interesado en usar esta funcionalidad en cualquier otra región, puede [enviar una solicitud](https://aka.ms/ccodereqregion). La compatibilidad con todas las regiones de Azure está en el plan de desarrollo.

### <a name="can-i-access-metadatapropertyvalue-from-my-inputs-similar-to-getmetadatapropertyvalue-function"></a>¿Puedo acceder a MetadataPropertyValue desde mis entradas de forma similar a la función GetMetadataPropertyValue?

No se admite esta funcionalidad. Si necesita esta funcionalidad, puede votar por esta solicitud en [UserVoice](https://feedback.azure.com/forums/270577-stream-analytics/suggestions/38779801-accessing-input-metadata-properties-in-custom-dese).

### <a name="can-i-share-my-deserializer-implementation-with-the-community-so-that-others-can-benefit"></a>¿Puedo compartir mi implementación de deserializador con la comunidad para que otros puedan beneficiarse?

Una vez que haya implementado el deserializador, puede ayudar a otros usuarios compartiéndolo en la comunidad. Envíe el código al [repositorio de GitHub de Azure Stream Analytics](https://github.com/Azure/azure-stream-analytics/tree/master/CustomDeserializers).

### <a name="what-are-the-other-limitation-of-using-custom-deserializers-in-stream-analytics"></a>¿Cuáles son las otras limitaciones de usar deserializadores personalizados en Stream Analytics?

Si la entrada tiene el formato Protobuf con un esquema que contiene el tipo MapField, no podrá implementar un deserializador personalizado. Estamos trabajando para admitir este tipo en el futuro.

## <a name="next-steps"></a>Pasos siguientes

* [Deserializadores de .NET personalizados para trabajos en la nube de Azure Stream Analytics](custom-deserializer.md)
