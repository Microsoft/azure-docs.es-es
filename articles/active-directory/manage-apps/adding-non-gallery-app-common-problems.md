---
title: Problemas al agregar una aplicación que no pertenece a la galería | Microsoft Docs
description: Comprenda los problemas habituales a los que se enfrentan los usuarios al agregar aplicaciones que no pertenecen a la galería
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: 38a9ef04389318d3588649117c930ff6efa3fe4e
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784479"
---
# <a name="problem-adding-a-non-gallery-application"></a>Problemas al agregar una aplicación que no pertenece a la galería

Este artículo le ayuda a conocer los problemas habituales a los que se enfrentan los usuarios al agregar **aplicaciones personalizadas que no pertenecen a la galería** y los pasos que se deben dar para resolverlos. 

## <a name="i-clicked-the-add-button-and-my-application-took-a-long-time-to-appear"></a>Hice clic en el botón "Agregar" y mi aplicación tardó mucho tiempo en aparecer

En algunas circunstancias, una aplicación puede tardar entre 1 y 2 minutos (a veces, más) en aparecer después de agregarla a su directorio. Aunque este no es el rendimiento normal esperable, puede ver si la incorporación de la aplicación está en curso haciendo clic en el icono de **notificaciones** (la campana) de la esquina superior derecha de [Azure Portal](https://portal.azure.com/) y buscando una notificación **En curso** o **Completado** con la etiqueta **Crear aplicación**.

Si la aplicación nunca se agrega o se produce un error al hacer clic en el botón **Agregar**, verá una **notificación** con un estado de **Error**. Si desea obtener más detalles sobre el error para conocer más información o compartirlo con un ingeniero de soporte técnico, puede ver más información sobre el error siguiendo los pasos descritos en [Visualización de los detalles de una notificación del portal](#how-to-see-the-details-of-a-portal-notification).

## <a name="i-clicked-the-add-button-and-my-application-didnt-appear"></a>Hice clic en el botón "Agregar" y mi aplicación no apareció

En ocasiones, debido a problemas transitorios, problemas de red o un error, se produce un error al agregar la aplicación. Puede indicar si esto ocurre si hace clic en el icono de **notificaciones** (la campana) de la esquina superior derecha de Azure Portal y aparece un icono (!) de color rojo junto a la notificación **Crear aplicación**. Esto indica que se produjo un error al crear la aplicación.

Si se produce un error al hacer clic en el botón **Agregar**, verá una **notificación** con un estado de **Error**. Si desea obtener más detalles sobre el error para conocer más información o compartirlo con un ingeniero de soporte técnico, puede ver más información sobre el error siguiendo los pasos descritos en [Visualización de los detalles de una notificación del portal](#how-to-see-the-details-of-a-portal-notification).

## <a name="i-dont-know-how-to-set-up-my-application-once-ive-added-it"></a>No sé cómo configurar mi aplicación una vez que la he agregado

Si necesita ayuda para obtener más detalles sobre las aplicaciones personalizadas, la [biblioteca de documentos de las aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-apps-index) le ayudará a obtener más información sobre el inicio de sesión único con Azure AD y cómo funciona.

## <a name="how-to-see-the-details-of-a-portal-notification"></a>Visualización de los detalles de una notificación del portal

Puede ver los detalles de cualquier notificación del portal si sigue los pasos siguientes:

1. Haga clic en el icono de **notificaciones** (la campana) de la esquina superior derecha de Azure Portal.

2. Seleccione cualquier notificación con un estado de **Error** (aquellas con un icono (!) de color rojo situado junto a ellas).

   >[!NOTE]
   >No puede hacer clic en notificaciones con un estado **Correcto** o **En curso**.
   >
   >

4. Utilice la información que se encuentra en **Detalles de notificación** para conocer más detalles acerca del problema.

5. Si aún necesita ayuda, también puede compartir esta información con un ingeniero de soporte técnico o el grupo de producto para obtener ayuda.

6. Haga clic en el **icono de copia** situado a la derecha del cuadro de texto **Copiar error** para copiar todos los detalles de la notificación para compartirlos con un ingeniero de soporte técnico o un grupo de producto.

## <a name="how-to-get-help-by-sending-notification-details-to-a-support-engineer"></a>Obtención de ayuda mediante el envío de detalles de la notificación a un ingeniero de soporte técnico

Es muy importante que comparta **todos los detalles que se muestran a continuación** con un ingeniero de soporte técnico si necesita ayuda, para que pueda ayudarle rápidamente. Puede hacer esto fácilmente **realizando una captura de pantalla** o haciendo clic en el **icono Copiar error**, que se encuentra a la derecha del cuadro de texto **Copiar error**.

## <a name="notification-details-explained"></a>Explicación de los detalles de la notificación

Consulte las descripciones siguientes para más información acerca de las notificaciones.

### <a name="essential-notification-items"></a>Elementos esenciales de notificación

- **Título**: el título descriptivo de la notificación
  *  Por ejemplo: **Configuración del proxy de aplicación**

- **Descripción**: la descripción de lo que se produjo como resultado de la operación

  *  Por ejemplo: **la dirección url interna especificada ya se está usando en otra aplicación**

- **Identificador de notificación**: el identificador único de la notificación

  *  Por ejemplo: **clientNotification-2adbfc06-2073-4678-a69f-7eb78d96b068**

- **Identificador de solicitud de cliente**: el identificador de solicitud específico generado por el explorador

  *  Por ejemplo: **302fd775-3329-4670-a9f3-bea37004f0bc**

- **Marca de tiempo UTC**: la marca de tiempo durante la que tuvo lugar la notificación, en formato UTC

  *  Por ejemplo: **2017-03-23T19:50:43.7583681Z**

- **Identificador de transacción interno**: el identificador interno que podemos utilizar para buscar el error en nuestros sistemas

  *  Por ejemplo: **71a2f329-ca29-402f-aa72-bc00a7aca603**

- **UPN**: el usuario que realizó la operación

  *  Por ejemplo: **tperkins\@f128.info**

- **Identificador de inquilino**: el identificador único del inquilino del que formaba parte el usuario que realizó la operación

  *  Por ejemplo: **7918d4b5-0442-4a97-be2d-36f9f9962ece**

- **Identificador de objeto de usuario**: el identificador único del usuario que realizó la operación

  *  Por ejemplo: **17f84be4-51f8-483a-b533-383791227a99**

### <a name="detailed-notification-items"></a>Elementos detallados de notificación

- **Nombre para mostrar**: **(puede estar vacío)** un nombre para mostrar más detallado del error

  *  Por ejemplo: **Configuración del proxy de aplicación**

- **Estado**: el estado específico de la notificación

  *  Por ejemplo: **Error**

- **Identificador de objeto**: **(puede estar vacío)** el identificador del objeto en el que se realizó la operación

  *  Por ejemplo: **8e08161d-f2fd-40ad-a34a-a9632d6bb599**

- **Detalles**: la descripción detallada de lo que se produjo como resultado de la operación

  *  Por ejemplo: **dirección url interna `https://bing.com/` no es válido porque ya está en uso**

- **Copiar error**: haga clic en el **icono de copia** situado a la derecha del cuadro de texto **Copiar error** para copiar todos los detalles de la notificación para compartirlos con un ingeniero de soporte técnico o grupo de  
- producto

  *  Por ejemplo, ```{"errorCode":"InternalUrl\_Duplicate","localizedErrorDetails":{"errorDetail":"Internal url 'https://google.com/' is invalid since it is already in use"},"operationResults":\[{"objectId":null,"displayName":null,"status":0,"details":"Internal url 'https://bing.com/' is invalid since it is already in use"}\],"timeStampUtc":"2017-03-23T19:50:26.465743Z","clientRequestId":"302fd775-3329-4670-a9f3-bea37004f0bb","internalTransactionId":"ea5b5475-03b9-4f08-8e95-bbb11289ab65","upn":"tperkins@f128.info","tenantId":"7918d4b5-0442-4a97-be2d-36f9f9962ece","userObjectId":"17f84be4-51f8-483a-b533-383791227a99"}```




