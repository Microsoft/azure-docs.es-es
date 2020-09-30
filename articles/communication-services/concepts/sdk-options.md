---
title: Bibliotecas cliente y API REST para Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga más información acerca de las bibliotecas cliente de Azure Communication Services y las API REST.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/18/2020
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: f39aa76e4bd2ce3d298e555f56b09d0218ef0862
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931993"
---
# <a name="client-libraries-and-rest-apis"></a>Bibliotecas cliente y API REST

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Las funcionalidades de Azure Communication Services están organizadas conceptualmente en seis áreas. Algunas áreas tienen bibliotecas cliente de código abierto completamente. La biblioteca cliente de llamadas utiliza interfaces de red propietarias y tiene actualmente formato de código cerrado, y la biblioteca de chat incluye una dependencia de código cerrado. En el [repositorio de GitHub de Azure Communication Services](https://github.com/Azure/communication) se mantienen ejemplos y vínculos a todos los SDK.

## <a name="client-libraries"></a>Bibliotecas de clientes

| Ensamblado               | Protocolos             |Código abierto vs. cerrado| Espacios de nombres                          | Funcionalidades                                                      |
| ---------------------- | --------------------- | ---|-------------------------- | --------------------------------------------------------------------------- |
| Azure Resource Manager | REST | Abrir            | Azure.ResourceManager.Communication | Aprovisiona y administra recursos de Communication Services.             |
| Comunes                 | REST | Abrir               | Azure.Communication.Common          | Proporciona tipos base para otras bibliotecas cliente. |
| Administración         | REST |                | Azure.Communication.Administration  | Administra usuarios, tokens de acceso y números de teléfono, asigna servidores STUN y TURN de acuerdo con las normativas. |
| Chat                   | REST con señalización propietaria | Se abre con el paquete de señalización de código cerrado.    | Azure.Communication.Chat            | Incorpora chat basado en texto en tiempo real a las aplicaciones.  |
| SMS                    | REST | Abrir              | Azure.Communication.SMS             | Envía y recibe mensajes SMS. |
| Llamar                | Transporte propietario | Closed |Azure.Communication.Calling         | Aprovecha la voz, el vídeo, el uso compartido de pantalla y otras capacidades de comunicación de datos en tiempo real.          |

### <a name="client-library-language-support"></a>Compatibilidad con el lenguaje de la biblioteca cliente.

A continuación se detallan las instrucciones y escalas de tiempo de disponibilidad para los paquetes de biblioteca cliente individuales. En la [hoja de ruta de Azure](https://azure.microsoft.com/updates/) se proporciona información adicional sobre las próximas características.

| Área           | JavaScript | .NET | Python | Java | Swift u Obj-C | Java (Android) | Otros                          |
| -------------- | ---------- | ---- | ------ | ---- | -------------- | -------------- | ------------------------------ |
| Azure Resource Manager | ✔️         | ✔️    | ✔️      | -    | -              | *Todavía no se admite*  | *No se admite aún* GO y CLI de Azure. |
| Comunes         | ✔️         | ✔️    | -      | ✔️   | ✔️            | ✔️             | -                              |
| Administración | ✔️         | ✔️    | ✔️      | ✔️   | -              | -              | CLI                            |
| Chat           | ✔️         | ✔️    | ✔️      | ✔️   | *Todavía no se admite*  | *Todavía no se admite*  | -                              |
| SMS            | ✔️         | ✔️    | ✔️      | ✔️   | -              | -              | -                              |
| Llamar        | ✔️         | -      | -      | -     | (Obj-C) ✔️     | ✔️            | -                              |

### <a name="client-library-public-repository-support"></a>Compatibilidad con el repositorio público de la biblioteca cliente

Communication Services publica bibliotecas compiladas en varios repositorios públicos.

| Lenguaje       | Optimizado para...                       | Packaging |
| -------------- | ------------------------------------ | --------- |
| .NET           | Multiplataforma                       | NuGet     |
| Python         | Servidores de Windows/Linux              | Pypi      |
| Java (J2EE)    | JVM en servidores Windows o Linux      | Maven     |
| Java (Android) | Aplicaciones cliente de Android          | Maven     |
| JavaScript     | Aplicaciones cliente del explorador y Node | Npm       |

## <a name="rest-apis"></a>API de REST

Las API de Communication Services están documentadas junto con otras API REST de Azure en [docs.microsoft.com](https://docs.microsoft.com/rest/api/azure/). Esta documentación le indicará cómo estructurar los mensajes HTTP y ofrece instrucciones para el uso de Postman. Esta documentación también se ofrece en formato Swagger en [GitHub](https://github.com/Azure/azure-rest-api-specs).

## <a name="additional-support-details"></a>Detalles adicionales de compatibilidad

### <a name="ios-and-android-support-details"></a>Detalles de compatibilidad con iOS y Android

- Las bibliotecas cliente de iOS de Communication Services tienen como destino iOS versión 13+ y Xcode 11+.
- Las bibliotecas cliente de Java para Android se dirigen al nivel de API de Android 21+ y Android Studio 4.0+.

### <a name="net-support-details"></a>Detalles de compatibilidad de .NET

A excepción de las llamadas, los paquetes de Communication Services tienen como destino .NET Standard 2.0, que es compatible con las plataformas que se enumeran a continuación.

Compatibilidad mediante .NET Framework 4.6.1
- Windows 10, 8.1, 8 y 7
- Windows Server 2012 R2, 2012 y 2008 R2 SP1

Compatibilidad mediante .NET Core 2.0:
- Windows 10 (1607+), 7 SP1+, 8.1
- Windows Server 2008 R2 SP1 y versiones posteriores
- Max OS X 10.12+
- Varias versiones o distribuciones de Linux
- UWP 10.0.16299 (RS3) septiembre 2017
- Unity 2018.1
- Mono 5.4
- Xamarin iOS 10.14
- Xamarin Mac 3.8

## <a name="api-stability-expectations"></a>Expectativas de estabilidad de API 

> [!IMPORTANT]
> En esta sección se proporcionan instrucciones sobre las API REST y las bibliotecas cliente marcadas como **estables**. Las API marcadas como versión preliminar o beta se pueden cambiar o dejar de usar **sin previo aviso**. Actualmente, Azure Communication Services se encuentra en **versión preliminar pública** y las API se marcan como tales.

En el futuro, es posible que retiremos las versiones de las bibliotecas cliente de Communication Services y que introduzcamos cambios importantes en nuestras API REST y bibliotecas cliente publicadas. Azure Communication Services *generalmente* sigue dos directivas de compatibilidad para retirar versiones de servicio:

- Se le notificará con una antelación de al menos tres años cuando sea necesario cambiar el código debido a un cambio en la interfaz de Communication Services. Todas las API REST documentadas y las API de la biblioteca cliente generalmente disfrutan de un período de al menos tres años de advertencia antes de que se retiren las interfaces.
- Se le notificará al menos un año antes de que tenga que actualizar los ensamblados de la biblioteca cliente a la versión secundaria más reciente. Estas actualizaciones necesarias no deben requerir ningún cambio en el código porque están en la misma versión principal. Esto es especialmente pertinente en el caso de las bibliotecas de llamadas y chat que tienen componentes en tiempo real que requieren con frecuencia actualizaciones de rendimiento y seguridad. Le recomendamos encarecidamente que mantenga actualizadas sus bibliotecas cliente de Communication Services.

### <a name="api-and-client-library-decommissioning-examples"></a>Ejemplos de retirada de API y biblioteca cliente

**Ha integrado la versión 24 de la API REST de SMS en la aplicación. Publicaciones de Azure Communication Services v25.**

Recibirá una advertencia 3 años antes de que estas API dejen de funcionar y sea obligatorio actualizarlas a v25. Es posible que esta actualización requiera un cambio de código.

**Ha integrado la versión v2.02 de la biblioteca cliente de llamadas en la aplicación. Publicaciones de Azure Communication Services v2.05.**

Es posible que se le pida que actualice a la versión v2.05 de la biblioteca cliente de llamadas en un plazo de 12 meses a partir de la publicación de la v2.05. Debe ser un reemplazo sencillo del artefacto sin necesidad de un cambio de código porque la v2.05 está en la versión principal v2 y no tiene cambios importantes.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte la siguiente información general sobre la biblioteca cliente:

- [Información general de la biblioteca cliente de llamadas](../concepts/voice-video-calling/calling-sdk-features.md)
- [Información general de la biblioteca cliente de chat](../concepts/chat/sdk-features.md)
- [Información general de la biblioteca cliente de SMS](../concepts/telephony-sms/sdk-features.md)

Para empezar a trabajar con Azure Communication Services:

- [Cree recursos de Azure Communication Services](../quickstarts/create-communication-resource.md).
- Genere [tokens de acceso de usuario](../quickstarts/access-tokens.md).
