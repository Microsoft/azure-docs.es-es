---
title: 'Protocolo de la extensión gRPC: Azure'
description: En este artículo, aprenderá a usar el protocolo de la extensión gRPC para enviar mensajes entre un módulo de Live Video Analytics y la extensión personalizada de inteligencia artificial o CV.
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 288dcd1a11c7c42d8796d3b17f2bfd56f562aaf1
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89448098"
---
# <a name="grpc-extension-protocol"></a>Protocolo de la extensión gRPC

En este artículo, aprenderá a usar el protocolo de la extensión gRPC para enviar mensajes entre un módulo de Live Video Analytics y la extensión personalizada de inteligencia artificial o CV.

gRPC es un marco de RPC moderno, de código abierto y de alto rendimiento que se puede ejecutar en cualquier entorno. El servicio de transporte gRPC utiliza streaming bidireccional HTTP/2 entre:

* el cliente de gRPC (módulo de Live Video Analytics en IoT Edge) y 
* el servidor de gRPC (su extensión personalizada).

Una sesión de gRPC es una conexión única desde el cliente de gRPC al servidor de gRPC a través del puerto TCP/TLS. 

En una sola sesión: El cliente envía un descriptor de flujo multimedia seguido de fotogramas de vídeo al servidor como un mensaje [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) en la sesión de flujo de gRPC. El servidor valida el descriptor de flujo, analiza el fotograma de vídeo y devuelve los resultados de la inferencia como un mensaje protobuf.

![Contrato de extensión de gRPC](./media/grpc-extension-protocol/grpc.png)

## <a name="implementing-grpc-protocol"></a>Implementación del protocolo gRPC

### <a name="creating-a-grpc-connection"></a>Creación de una conexión de gRPC

La extensión personalizada tiene que implementar el siguiente servicio gRPC:

```
service MediaGraphExtension {
  rpc ProcessMediaStream(stream MediaStreamMessage) returns (stream MediaStreamMessage);
}
```

Cuando se llama, se abrirá una secuencia bidireccional para que los mensajes fluyan entre la extensión gRPC y el grafo de Live Video Analytics. El primer mensaje enviado en esta secuencia por cada entidad contendrá un MediaStreamDescriptor, que define qué información se enviará en los siguientes MediaSamples.

Por ejemplo, la extensión del grafo puede enviar el mensaje (expresado aquí en JSON) para indicar que enviará los fotogramas codificados en rgb24 de 416x416 insertados en los mensajes gRPC a la extensión personalizada.

```
 {
    "sequence_number": 1,
    "ack_sequence_number": 0,
    "media_stream_descriptor": {
        "graph_identifier": {
            "media_services_arm_id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resourceGroupName/providers/microsoft.media/mediaservices/mediaAccountName",
            "graph_instance_name": "mediaGraphName",
            "graph_node_name": "grpcExtension"
        },
        "media_descriptor": {
            "timescale": 90000,
            "video_frame_sample_format": {
                "encoding": "RAW",
                "pixel_format": "RGB24",
                "dimensions": {
                    "width": 416,
                    "height": 416
                },
                "stride_bytes": 1248
            }
        }
    }
}
```

La extensión personalizada, en respuesta, enviará el mensaje siguiente para indicar que solo devuelve inferencias.

```
{
    "sequence_number": 1,
    "ack_sequence_number": 1,
    "media_stream_descriptor": {
        "extension_identifier": "customExtensionName"    }
}
```

Ahora que ambos lados han intercambiado los descriptores multimedia, Live Video Analytics comenzará a transmitir fotogramas a la extensión.

### <a name="sequence-numbers"></a>Números de secuencia

Tanto el nodo de extensión gRPC como la extensión personalizada mantienen un conjunto independiente de números de secuencia, que se asignan a sus mensajes. Estos números de secuencia deben aumentar con una progresión continua a partir de 1. `ack_sequence_number` se puede omitir si no se ha confirmando ningún mensaje, lo que puede ocurrir cuando se envía el primer mensaje.

Una solicitud se tiene que confirmarse una vez que se haya procesado completamente el mensaje correspondiente. En el caso de una transferencia de memoria compartida, esta confirmación indica que el remitente puede liberar la memoria compartida y que el receptor ya no tendrá acceso a ella. El remitente tiene que contener cualquier memoria compartida hasta que se confirme.

### <a name="reading-embedded-content"></a>Lectura del contenido insertado

El contenido insertado puede leerse directamente del mensaje `MediaSample` mediante el campo `content_byte`s. Los datos se codificarán de acuerdo con el `MediaDescriptor`.

### <a name="reading-content-from-shared-memory"></a>Lectura del contenido de la memoria compartida

Al transferir contenido a través de la memoria compartida, el remitente incluirá `SharedMemoryBufferTransferProperties` en su `MediaStreamDescriptor` cuando establezca una sesión por primera vez. Esto puede tener el siguiente aspecto (expresado en JSON):

```
{
  "handle_name": "inference_client_share_memory_2146989006636459346"
  "length_bytes": 20971520
}
```

A continuación, el receptor abre el archivo `/dev/shm/inference_client_share_memory_2146989006636459346`. El remitente enviará un mensaje de `MediaSample`, que contiene un `ContentReference` que hace referencia a una ubicación específica en este archivo.

```
{
    "timestamp": 143598615750000,
    "content_reference": {
        "address_offset": 519168,
        "length_bytes": 173056
    }
}
```

A continuación, el receptor lee los datos en esta ubicación en el archivo.

Para que el contenedor de Live Video Analytics se comunique a través de la memoria compartida, el modo IPC del contenedor tiene que estar configurado correctamente. Esto se puede hacer de muchas maneras, pero estas son algunas de las configuraciones recomendadas.

* Al comunicarse con un motor de inferencia de gRPC que se ejecuta en el dispositivo host, el modo IPC debe establecerse en host.
* Al comunicarse con un servidor de gRPC que se ejecuta en otro módulo de IoT Edge, el modo IPC debe establecerse en shareable para el módulo de Live Video Analytics y `container:liveVideoAnalytics` para la extensión personalizada, donde `liveVideoAnalytics` es el nombre del módulo de Live Video Analytics.

Este es el aspecto que podría tener esto en el dispositivo gemelo cuando se usa la primera de las opciones anteriores.

```
"liveVideoAnalytics": {
  "version": "1.0",
  "type": "docker",
  "status": "running",
  "restartPolicy": "always",
  "settings": {
    "image": "mcr.microsoft.com/media/live-video-analytics:1",
    "createOptions": 
      "HostConfig": {
        "IpcMode": "host"
      }
    }
  }
}
```

Para más información sobre los modos IPC, consulte https://docs.docker.com/engine/reference/run/#ipc-settings---ipc.

## <a name="media-graph-grpc-extension-contract-definitions"></a>Definiciones del contrato de extensión de gRPC de grafos multimedia

En esta sección define el contrato de gRPC que define el flujo de datos.

### <a name="protocol-messages"></a>Mensajes de protocolo

![Mensajes de protocolo](./media/grpc-extension-protocol/grpc2.png)
 
### <a name="client-authentication"></a>Autenticación de cliente

Los implementadores de extensiones personalizadas pueden validar la autenticidad de las conexiones gRPC entrantes para asegurarse de que provienen del nodo de extensión gRPC. El nodo proporcionará una entrada en los encabezados de solicitud que sirve para la validación.

Las credenciales de nombre de usuario y contraseña se pueden usar con este fin. Al crear un nodo de extensión gRPC, las credenciales se proporcionan como se indica a continuación:

```
{
  "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
  "name": "{moduleIdentifier}",
  "endpoint": {
    "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
    "url": "tcp://customExtension:8081",
    "credentials": {
      "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
      "username": "username",
      "password": "password"
    }
  }
  // Other fields omitted
}
```

Cuando se envía la solicitud gRPC, el encabezado siguiente se incluirá en los metadatos de la solicitud, imitando la autenticación HTTP Basic.

`x-ms-authentication: Basic (Base64 Encoded username:password)`

## <a name="using-grpc-over-tls"></a>Uso de gRPC a través de TLS

Una conexión gRPC utilizada para la inferencia se puede proteger a través de TLS. Esto resulta útil en situaciones en las que no se puede garantizar la seguridad de la red entre Live Video Analytics y el motor de inferencia. TLS cifrará todo el contenido insertado en los mensajes de gRPC, lo que provocará una sobrecarga adicional de la CPU al transmitir fotogramas a alta velocidad.

Las opciones de comprobación IgnoreHostname y IgnoreSignature no son compatibles con gRPC, por lo que el certificado de servidor que presente el motor de inferencia tiene que contener un CN que coincida exactamente con la dirección IP o el nombre de host en la dirección URL del punto de conexión del nodo de extensión gRPC.

## <a name="next-steps"></a>Pasos siguientes

[Información sobre el esquema de metadatos de inferencia](inference-metadata-schema.md)
