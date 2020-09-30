---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: matthewrobertson
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 08/20/2020
ms.topic: include
ms.custom: include file
ms.author: marobert
ms.openlocfilehash: c0ba8e76e069bf9dc1c96aecdc670699c32b3c96
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945754"
---
## <a name="object-model"></a>Modelo de objetos

Los tokens de acceso de usuario permiten autorizar a los usuarios de la aplicación para que se conecten directamente a su recurso de Azure Communication Services. Estos tokens se generan en el servidor, se devuelven a la aplicación cliente y, a continuación, se usan para inicializar las bibliotecas cliente de Communication Services. Para lograrlo, debe crear un punto de conexión de servicio de confianza que pueda autenticar a los usuarios y emitir tokens de acceso de usuario. Este servicio debe mantener una asignación persistente entre las identidades de usuario de la aplicación y los usuarios de Azure Communication Services.

La clase CommunicationIdentityClient proporciona toda la funcionalidad relacionada con el token de acceso y el usuario. Cree una instancia de esta con una cadena de conexión y, a continuación, úsela para administrar los usuarios y emitir tokens de acceso.

> [!WARNING]
> Los tokens son datos confidenciales, ya que conceden acceso a los recursos de un usuario. Por lo tanto, es fundamental proteger los tokens para que no se vean comprometidos. Debe crear un punto de conexión de servicio de confianza que solo emita tokens de acceso para los usuarios autorizados de la aplicación. Si almacena en caché los tokens de acceso de usuario en una memoria auxiliar, se recomienda encarecidamente usar el cifrado.

### <a name="access-token-scopes"></a>Ámbitos de los tokens de acceso

Los ámbitos permiten especificar la funcionalidad exacta de Azure Communication Services que un token de acceso de usuario podrá autorizar. Los ámbitos se aplican a tokens de acceso de usuario individuales. Azure Communication Services admite los siguientes ámbitos para los tokens de acceso de usuario:

| Nombre   | Descripción                                                                         |
| ------ | ----------------------------------------------------------------------------------- |
| `Chat` | Ofrece la posibilidad de participar en un chat                                         |
| `VoIP` | Ofrece la posibilidad de realizar y recibir llamadas VoIP mediante la biblioteca de cliente de llamadas* |
| `Pstn` | Ofrece la posibilidad de realizar llamadas RTC mediante la biblioteca de cliente de llamadas*           |

La característica \* todavía no se admite, pero está disponible próximamente.

Si quiere quitar la capacidad de un usuario para tener acceso a alguna funcionalidad específica, primero debe [revocar los tokens de acceso existentes](#revoke-user-access-tokens) que pueden incluir ámbitos no deseados antes de emitir un nuevo token con un conjunto de ámbitos más limitado.
