---
title: 'Esquema de metadatos de inferencia: Azure'
description: En este artículo, obtendrá información acerca del esquema de metadatos de inferencia.
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 6239713fc92b75b8ed026a8f04953e92a24c4596
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88701946"
---
# <a name="inference-metadata-schema"></a>Esquema de metadatos de inferencia 

Cada objeto de inferencia, independientemente de que se use el contrato basado en HTTP o el contrato basado en gRPC, sigue el modelo de objetos que se describe en este tema.

## <a name="object-model"></a>Modelo de objetos

![Modelo de objetos](./media/inference-metadata-schema/object-model.png)
 
|Definición de tipo|Descripción|
|---|---|
|Etiqueta|Etiqueta asociada al resultado. Además del etiquetado, obtiene además el valor de confianza asociado a la etiqueta.|
|Atributo|Atributos adicionales asociados al resultado. Puede agregar nuevos atributos que reciba del motor de inferencia junto con el valor de confianza.|
|Lista de atributos|Lista de atributos opcionales.|
|Rectángulo|Región rectangular relativa a la imagen de la esquina superior izquierda de la imagen. Las propiedades necesarias serán "longitud", "ancho", "alto" y "distancia al borde superior desde el origen".|
|clasificación|La etiqueta del clasificador suele ser aplicable a todo el ejemplo. Con la ayuda de "etiqueta", puede clasificar el resultado.|
|Entidad|Entidad detectada o identificada en el ejemplo. Cuando el motor de inferencia detecta una entidad, obtiene una "etiqueta", los atributos adicionales que se deducen y se devuelven las coordenadas de un cuadro rectangular alrededor de la entidad encontrada.|
|Evento|Evento detectado en el ejemplo. Cuando se detecta un evento en el ejemplo, se devuelven el nombre del evento y las propiedades específicas del mismo.|
|Movimiento|Movimiento detectado en el ejemplo. Cuando se detecta movimiento en el ejemplo, se devuelven las coordenadas de un rectángulo de selección en el que se detecta movimiento.|
|Texto|Se devuelve el texto asociado al ejemplo junto con la marca de tiempo de inicio y finalización del texto.|
|Otros|Devuelve otra información de carga genérica.|

El ejemplo siguiente contiene un solo evento con algunos de los tipos de inferencia admitidos:

```
[ 
  // Light Detection 
  { 
    "type": "classification", 
    "subtype": "lightDetection", 
    "classification": { 
      "tag": { "value": "daylight", "confidence": 0.86 }, 
      "attributes": [ 
          { "name": "isBlackAndWhite", "value": "false", "confidence": 0.71 } 
      ] 
    } 
  }, 
 
  // Motion Detection 
  { 
    "type": "motion", 
    "subtype": "motionDetection", 
    "motion": 
    { 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // Yolo V3 
  { 
    "type": "entity", 
    "subtype": "objectDetection",     
    "entity": 
    { 
      "tag": { "value": "dog", "confidence": 0.97 }, 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // Vehicle Identification 
  { 
    "type": "entity", 
    "subtype": "vehicleIdentification",     
    "entity": 
    { 
      "tag": { "value": "007-SPY", "confidence": 0.82 }, 
      "attributes":[   
        { "name": "color", "value": "black", "confidence": 0.90 }, 
        { "name": "body", "value": "coupe", "confidence": 0.87 }, 
        { "name": "make", "value": "Aston Martin", "confidence": 0.35 }, 
        { "name": "model", "value": "DBS V12", "confidence": 0.33 } 
      ], 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // People Identification 
  { 
    "type": "entity", 
    "subtype": "peopleIdentification",     
    "entity": 
    { 
      "tag": { "value":"Erwin Schrödinger", "confidence": 0.50 }, 
      "attributes":[   
        { "name": "age", "value": "73", "confidence": 0.87 }, 
        { "name": "glasses", "value": "yes", "confidence": 0.94 } 
      ], 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    }, 
 
    // Open type coming from the gRPC Map 
    "extensions":  
    { 
      "vector": "e1xkaXNwbGF5c3R5bGUgaVxoYmFyIHtcZnJhYyB7ZH17ZHR9fVx2ZXJ0IFxQc2kgKHQpXHJhbmdsZSA9e1xoYXQge0h9fVx2ZXJ0IFxQc2kgKHQpXHJhbmdsZSB9KQ==", 
      "skeleton": "p1,p2,p3,p4" 
    } 
  }, 
 
  // Captions 
  {     
    "type": "text", 
    "subtype": "speechToText",   
    "text": 
    { 
      "value": "Humor 75%. Confirmed. Self-destruct sequence in T minus 10… 9…", 
      "language": "en-US", 
      "startTimestamp": 12000, 
      "endTimestamp": 13000 
    } 
]
```

## <a name="next-steps"></a>Pasos siguientes

- [Contrato de datos de gRPC](grpc-data-contract.md)
- [Contrato de datos de HTTP](http-data-contract.md)
