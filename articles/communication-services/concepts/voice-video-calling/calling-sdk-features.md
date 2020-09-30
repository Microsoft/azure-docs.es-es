---
title: Información general de la biblioteca cliente de llamadas de Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Proporciona información general sobre la biblioteca cliente de llamadas.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 9cc00cfa9f44c69a5880d53c0b7ac623f60be16b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945763"
---
# <a name="calling-client-library-overview"></a>Información general de la biblioteca cliente de llamadas

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

Hay dos familias independientes de bibliotecas cliente de llamadas, la de *clientes* y la de *servicios*. Las bibliotecas cliente disponibles actualmente están pensadas para experiencias de usuario final: sitios web y aplicaciones nativas.

Las bibliotecas cliente de servicio aún no están disponibles. Estas proporcionan acceso a los planos de datos de vídeo y de voz sin procesar, que son adecuados para la integración con bots y otros servicios.

## <a name="calling-client-library-capabilities"></a>Funcionalidades de la biblioteca cliente de llamadas

En la lista siguiente se presenta el conjunto de características que están disponibles actualmente en las bibliotecas cliente de llamadas de Azure Communication Services.

| Grupo de características | Capacidad                                                                                                          | JS  | Java (Android) | Objective-C (iOS) 
| ----------------- | ------------------------------------------------------------------------------------------------------------------- | ---  | -------------- | -------------
| Funcionalidades principales | Realizar una llamada uno a uno entre dos usuarios                                                                           | ✔️   | ✔️            | ✔️  
|                   | Realizar una llamada de grupo con más de dos usuarios (hasta 350)                                                       | ✔️   | ✔️            | ✔️ 
|                   | Promocionar una llamada uno a uno con dos usuarios a una llamada de grupo con más de dos usuarios                                 | ✔️   | ✔️            | ✔️ 
|                   | Unirse a una llamada de grupo después de que se haya iniciado                                                                              | ✔️   | ✔️            | ✔️ 
|                   | Invitar a otro participante de VoIP a unirse a una llamada de grupo en curso                                                       | ✔️   | ✔️            | ✔️ 
|                   | Desactivar/activar audio del micrófono                                                                                                     | ✔️   | ✔️            | ✔️         
|                   | Cambiar entre las cámaras                                                                                              | ✔️   | ✔️            | ✔️           
|                   | Retención/reanudación local                                                                                                  | ✔️   | ✔️            | ✔️           
|                   | Altavoz activo                                                                                                      | ✔️   | ✔️            | ✔️           
|                   | Elegir el altavoz para llamadas                                                                                            | ✔️   | ✔️            | ✔️           
|                   | Elegir el micrófono para llamadas                                                                                         | ✔️   | ✔️            | ✔️           
|                   | Mostrar el estado de un participante<br/>*Inactivo, elementos multimedia iniciales, conectando, conectado, en espera, en la sala de espera, desconectado*         | ✔️   | ✔️            | ✔️           
|                   | Mostrar el estado de una llamada<br/>*Elementos multimedia iniciales, entrante, conectando, llamando, conectada, en espera, desconectando, desconectada* | ✔️   | ✔️            | ✔️           
|                   | Mostrar si un participante está silenciado                                                                                      | ✔️   | ✔️            | ✔️           
|                   | Mostrar el motivo por el que un participante abandonó una llamada                                                                       | ✔️   | ✔️            | ✔️     
| Uso compartido de la pantalla    | Compartir la pantalla completa desde la aplicación                                                                 | ✔️   | ❌            | ❌           
|                   | Compartir una aplicación específica (desde la lista de aplicaciones en ejecución)                                                | ✔️   | ❌            | ❌           
|                   | Compartir una pestaña del explorador web desde la lista de pestañas abiertas                                                                  | ✔️   | ❌            | ❌           
|                   | El participante puede ver el uso compartido de pantalla remota                                                                            | ✔️   | ✔️            | ✔️         
| Lista            | Enumerar participantes                                                                                                   | ✔️   | ✔️            | ✔️           
|                   | Quitar un participante                                                                                                | ✔️   | ✔️            | ✔️         
| RTC              | Realizar una llamada uno a uno con un participante de RTC                                                                     | ✔️   | ✔️            | ✔️   
|                   | Realizar una llamada de grupo con participantes de RTC                                                                           | ✔️   | ✔️            | ✔️
|                   | Promocionar una llamada uno a uno con un participante de RTC a una llamada de grupo                                                 | ✔️   | ✔️            | ✔️
|                   | Llamada saliente de una llamada de grupo como participante de RTC                                                                    | ✔️   | ✔️            | ✔️   
| General           | Participar en una llamada de grupo con un máximo de 350 participantes                                                       |  ✔️  | ✔️            | ✔️    
|                   | Probar el micrófono, el altavoz y la cámara con un servicio de prueba de audio (disponible llamando a 8:echo123).                   |  ✔️  | ✔️            | ✔️   

## <a name="calling-client-library-browser-support"></a>Compatibilidad del explorador con la biblioteca cliente de llamadas

En la tabla siguiente se representa el conjunto de exploradores y versiones compatibles que están disponibles actualmente.

|                                  | Windows          | macOS          | Ubuntu | Linux  | Android | iOS    |
| -------------------------------- | ---------------- | -------------- | ------- | ------ | ------ | ------ |
| **Biblioteca cliente de llamadas** | Chrome*, nuevo Edge | Chrome *, Safari** | Chrome*  | Chrome* | Chrome* | Safari** |


*Tenga en cuenta que se admite la versión más reciente de Chrome además de las dos versiones anteriores.<br/>

**Tenga en cuenta que se admiten las versiones de Safari 13.1 y posteriores. Todavía no se admite el vídeo de salida para Safari macOS, pero es compatible con iOS. El uso compartido de pantalla saliente solo se admite en iOS de escritorio.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a las llamadas](../../quickstarts/voice-video-calling/getting-started-with-calling.md)

Para más información, consulte los siguientes artículos.
- Familiarización con los [flujos de llamada](../call-flows.md) generales
- Más información sobre los [tipos de llamada](../voice-video-calling/about-call-types.md)
- [Planeación de una solución RTC](../telephony-sms/plan-solution.md)
