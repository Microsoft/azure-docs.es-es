---
title: 'Conversación entre varios dispositivos (versión preliminar): Servicio de voz'
titleSuffix: Azure Cognitive Services
description: ''
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: trbye
ms.openlocfilehash: 15b69e664ae9a01635afb44d7b6b7c24d7551ce9
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921425"
---
# <a name="what-is-multi-device-conversation-preview"></a>¿Qué es la conversación entre varios dispositivos (versión preliminar)?

La **conversación entre varios dispositivos** facilita la creación de una conversación de voz o texto entre varios clientes y la coordinación de los mensajes que se envían entre ellos.

Con la **conversación entre varios dispositivos**, puede:

- Conectar varios clientes a la misma conversación y administrar el envío y la recepción de mensajes entre ellos.
- Transcribir fácilmente el audio de cada cliente y enviar la transcripción a los demás, con traducción opcional.
- Enviar fácilmente mensajes de texto entre clientes, con traducción opcional.

Puede crear una característica o solución que funcione en una matriz de dispositivos. Cada dispositivo puede enviar mensajes de forma independiente (ya sea transcripciones de mensajes de audio o mensajes instantáneos) a todos los demás dispositivos.

Mientras que [**transcripción de conversaciones**](conversation-transcription.md) funciona en un solo dispositivo con una matriz multicanal de micrófonos, la **conversación entre varios dispositivos** es adecuada para escenarios con varios dispositivos, cada uno de ellos con un solo micrófono.

>[!IMPORTANT]
> La conversación entre varios dispositivos **no** admite el envío de archivos de audio entre clientes: solo la transcripción o la traducción.

## <a name="key-features"></a>Principales características

- **Transcripción en tiempo real**: todos recibirán una transcripción de la conversación para que puedan seguir el texto en tiempo real o guardarlo para más adelante.
- **Traducción en tiempo real**: con más de 60 [idiomas admitidos](language-support.md#text-languages) para la traducción de texto, los usuarios pueden traducir la conversación a sus idiomas preferidos.
- **Transcripciones legibles**: la transcripción y la traducción son fáciles de seguir, con signos de puntuación y pausas entre oraciones.
- **Entradas de voz o de texto**: cada usuario puede hablar o escribir en su propio dispositivo según las funcionalidades de compatibilidad habilitadas para el idioma elegido del participante. Consulte [Compatibilidad con idiomas](language-support.md#speech-to-text).
- **Retransmisión de mensajes**: el servicio de conversación entre dispositivos distribuye los mensajes que envía un cliente a todos los demás, en los idiomas de su elección.
- **Identificación de mensajes**: todos los mensajes que los usuarios reciben en la conversación se etiquetan con el alias del usuario que lo envió.

## <a name="use-cases"></a>Casos de uso

### <a name="lightweight-conversations"></a>Conversaciones ligeras

Crear una conversación y unirse a ella es fácil. Un usuario actúa como "anfitrión" y crea una conversación, lo cual genera un código de conversación aleatorio de cinco letras y un código QR. Todos los demás usuarios pueden unirse a la conversación escribiendo el código de conversación o digitalizando el código QR. 

Dado que los usuarios se unen mediante el código de conversación y no es necesario que compartan una información de contacto, es fácil crear conversaciones rápidas, en el momento.

### <a name="inclusive-meetings"></a>Reuniones inclusivas

La transcripción y traducción en tiempo real puede ayudar a que las conversaciones sean accesibles para los usuarios que hablan idiomas diferentes o que son sordos o presentan dificultades auditivas. Cada usuario puede también participar activamente en la conversación hablando su idioma preferido o enviando mensajes instantáneos.

### <a name="presentations"></a>Presentaciones

También puede proporcionar subtítulos para presentaciones y conferencias tanto en pantalla como en los propios dispositivos de los miembros de la audiencia. Una vez que el público se une con el código de la conversación, puede ver la transcripción en su idioma preferido y en su propio dispositivo.

> [!NOTE]
> Para ver un ejemplo, consulte [Traductor de presentaciones](https://www.microsoft.com/translator/apps/presentation-translator/), un complemento de PowerPoint que usa el servicio de conversación entre varios dispositivos. Puede descargarlo [aquí](https://download.cnet.com/s/powerpoint-add-in/).

## <a name="how-it-works"></a>Funcionamiento

Todos los clientes usan el SDK de Voz para crear una conversación o unirse a ella. Speech SDK interactúa con el servicio de conversación entre varios dispositivos, que administra la duración de una conversación, lo que incluye la lista de participantes, los idiomas elegidos de cada cliente y los mensajes enviados.  

Cada cliente puede enviar mensajes de audio o instantáneos. El servicio usará el reconocimiento de voz para convertir audio en texto y enviar mensajes instantáneos tal cual. Si los clientes eligen idiomas diferentes, el servicio traducirá todos los mensajes a los idiomas especificados de cada cliente.

![Diagrama general de una conversación entre varios dispositivos](media/scenarios/multi-device-conversation.png)

## <a name="overview-of-conversation-host-and-participant"></a>Información general sobre la conversación, el anfitrión y los participantes

Una **conversación** es una sesión que un usuario inicia para que los otros usuarios participantes se unan. Todos los clientes se conectan a la conversación mediante el **código de la conversación** de cinco letras.

Cada conversación crea metadatos que incluyen:
-    Marcas de tiempo de cuando se inició y finalizó la conversación.
-    La lista de todos los participantes en la conversación, la cual incluye el alias elegido de cada usuario y el idioma principal para la entrada de texto o de voz.


Hay dos tipos de usuarios en una conversación: **anfitrión** y **participante**.

El **anfitrión** es el usuario que inicia una conversación y que actúa como administrador de esa conversación.
- Cada conversación solo puede tener un anfitrión.
- El anfitrión debe estar conectado a la conversación mientras dure la conversación. Si el anfitrión abandona la conversación, la conversación finalizará para el resto de participantes.
- El anfitrión tiene algunos controles adicionales para administrar la conversación: 
    - Bloquear la conversación: impide que se unan participantes adicionales
    - Silenciar a todos los participantes: impide que los demás participantes envíen mensajes a la conversación, ya sean transcritos a partir de voz o mensajes instantáneos.
    - Silenciar a participantes individuales
    - Reactivar el audio de todos los participantes
    - Reactivar el audio de participantes individuales

Un **participante** es un usuario que se une a una conversación.
- Un participante puede dejar una conversación y volver a unirse a la misma en cualquier momento, sin que por ello finalice la conversación para otros participantes.
- Los participantes no pueden bloquear la conversación ni silenciar o reactivar el audio de los demás

> [!NOTE]
> Cada conversación puede tener hasta 100 participantes, de los cuales 10 pueden hablar simultáneamente en cualquier momento dado.

## <a name="language-support"></a>Compatibilidad con idiomas

Al crear una conversación o unirse a ella, cada usuario debe elegir un **idioma principal**: el idioma en el que hablará y se enviarán los mensajes instantáneos, así como el idioma en el que verá los mensajes de los demás usuarios.

Hay dos clases de idiomas: los que admiten la conversión de **voz a texto** y los que admiten **solo texto**:
- Si el usuario elige la opción de **voz a texto** como opción principal, podrá usar la entrada de texto y de voz en la conversación.

- Si el usuario elige la opción de **solo texto**, solo podrá usar la entrada de texto y enviar mensajes instantáneos en la conversación. Los idiomas que admiten solo texto son aquellos que admiten la traducción de texto, pero no la conversión de voz a texto. Puede ver los idiomas disponibles en la página [Compatibilidad con idiomas](supported-languages.md).

Además de su idioma principal, cada participante también puede especificar idiomas adicionales para traducir la conversación.

A continuación se muestra un resumen de lo que el usuario podrá hacer en una conversación entre varios dispositivos, en función del idioma principal elegido.


| Acciones que puede realizar el usuario en la conversación | Voz a texto | Solo texto |
|-----------------------------------|----------------|------|
| Usar entradas de voz | ✔️ | ❌ |
| Enviar mensajes instantáneos | ✔️ | ✔️ |
| Traducir la conversación | ✔️ | ✔️ |

> [!NOTE]
> Si desea obtener una lista completa de los idiomas disponibles para la conversión de voz a texto y para la traducción de texto, consulte [Compatibilidad con idiomas](supported-languages.md).



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Traducción de conversaciones en tiempo real](quickstarts/multi-device-conversation.md)
