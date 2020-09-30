---
title: Solución de problemas comunes al agregar una aplicación a Azure Active Directory
description: Solucione los problemas comunes a los que los usuarios se enfrentan al agregar una aplicación a Azure Active Directory.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 09/11/2018
ms.author: kenwith
ms.openlocfilehash: 1502c97e625d3af128772451be8db4dd6187e3a7
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89410369"
---
# <a name="troubleshoot-common-problem-adding-an-application-to-azure-active-directory"></a>Solución de problemas comunes al agregar una aplicación a Azure Active Directory
Este artículo le ayuda a comprender los problemas comunes a los que los usuarios se enfrentan al agregar una aplicación a Azure Active Directory.

## <a name="i-clicked-the-add-button-and-my-application-took-a-long-time-to-appear"></a>Hice clic en el botón "Agregar" y mi aplicación tardó mucho tiempo en aparecer
En algunas circunstancias, una aplicación puede tardar entre 1 y 2 minutos (a veces, más) en aparecer después de agregarla a su directorio. Aunque este no es el rendimiento normal esperado, puede ver si la incorporación de la aplicación está en curso haciendo clic en el icono de **notificaciones** (con forma de campana) de la esquina superior derecha de [Azure Portal](https://portal.azure.com/) y buscando una notificación que tenga un estado **En curso** o **Completado** y la etiqueta **Crear aplicación.**

Si la aplicación nunca se agrega o se produce un error al hacer clic en el botón **Agregar**, verá una **notificación** con un estado de **Error**. Si desea obtener más detalles sobre el error para conocer más información o compartirlo con un ingeniero de soporte técnico, puede ver más información sobre el error siguiendo los pasos descritos en [Visualización de los detalles de una notificación del portal](#how-to-see-the-details-of-a-portal-notification).

## <a name="i-clicked-the-add-button-and-my-application-didnt-appear"></a>Hice clic en el botón "Agregar" y mi aplicación no apareció
En ocasiones y debido a problemas transitorios, problemas de red o un error, se produce un error al agregar la aplicación. Para saber si le ha ocurrido esto, haga clic en el icono de **notificaciones** (con forma de campana) de la esquina superior derecha de Azure Portal y aparecerá un icono (!) de color rojo junto a la notificación **Crear aplicación**. Esto indica que se produjo un error al crear la aplicación.

Si se produce un error al hacer clic en el botón **Agregar**, verá una **notificación** con un estado de **Error**. Si desea obtener más detalles sobre el error para conocer más información o compartirlo con un ingeniero de soporte técnico, puede ver más información sobre el error siguiendo los pasos descritos en [Visualización de los detalles de una notificación del portal](#how-to-see-the-details-of-a-portal-notification).

## <a name="i-dont-know-how-to-set-up-my-application-once-ive-added-it"></a>No sé cómo configurar mi aplicación una vez que la he agregado
Si necesita ayuda para obtener información sobre las aplicaciones, el artículo [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list) es un buen punto de partida.

Además, la [biblioteca de documentos de las aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-apps-index) le ayudará a obtener más información sobre el inicio de sesión único con Azure AD y así poder saber cómo funciona.

## <a name="how-to-see-the-details-of-a-portal-notification"></a>Visualización de los detalles de una notificación del portal
Puede ver los detalles de cualquier notificación del portal si sigue los pasos siguientes:
1.  Seleccione el icono **Notificaciones** (con forma de campana) de la esquina superior derecha de Azure Portal.
2.  Seleccione cualquier notificación con un estado de **Error** (aquellas con un icono (!) de color rojo situado junto a ellas).
    >[!NOTE]
    >No puede hacer clic en notificaciones con un estado **Correcto** o **En curso**.
4.  Utilice la información que se encuentra en **Detalles de notificación** para conocer más detalles acerca del problema.
5.  Si aún necesita ayuda, también puede compartir esta información con un ingeniero de soporte técnico o el grupo de producto para obtener ayuda.
6.  Seleccione el **icono de copa** a la derecha del cuadro de texto **Copiar error** para copiar todos los detalles de la notificación y compartirlos con un ingeniero de soporte técnico o de un grupo de un producto.

## <a name="how-to-get-help-by-sending-notification-details-to-a-support-engineer"></a>Obtención de ayuda mediante el envío de detalles de la notificación a un ingeniero de soporte técnico
Es muy importante que comparta **todos los detalles que se muestran a continuación** con un ingeniero de soporte técnico si necesita ayuda, para que pueda ayudarle rápidamente. **Haga una captura de pantalla** o seleccione el **icono Copiar error**, que se encuentra a la derecha del cuadro de texto **Copiar error**.

## <a name="notification-details-explained"></a>Explicación de los detalles de la notificación
Consulte las descripciones siguientes para más información acerca de las notificaciones.

### <a name="essential-notification-items"></a>Elementos esenciales de notificación
- **Título**: el título descriptivo de la notificación
  * Por ejemplo: **Configuración del proxy de aplicación**
- **Descripción**: la descripción de lo que se produjo como resultado de la operación
  -   Por ejemplo: **la dirección url interna especificada ya se está usando en otra aplicación**
- **Identificador de notificación**: el identificador único de la notificación
  -   Por ejemplo: **clientNotification-2adbfc06-2073-4678-a69f-7eb78d96b068**
- **Identificador de solicitud de cliente**: el identificador de solicitud específico generado por el explorador
  -   Por ejemplo: **302fd775-3329-4670-a9f3-bea37004f0bc**
- **Marca de tiempo UTC**: la marca de tiempo durante la que tuvo lugar la notificación, en formato UTC
  -   Por ejemplo: **2017-03-23T19:50:43.7583681Z**
- **Identificador de transacción interno**: identificador interno que podemos usar para buscar el error en nuestros sistemas
  -   Por ejemplo: **71a2f329-ca29-402f-aa72-bc00a7aca603**
- **UPN**: el usuario que realizó la operación
  -   Por ejemplo: **tperkins\@f128.info**
- **Identificador de inquilino**: el identificador único del inquilino del que formaba parte el usuario que realizó la operación
  -   Por ejemplo: **7918d4b5-0442-4a97-be2d-36f9f9962ece**
- **Identificador de objeto de usuario**: el identificador único del usuario que realizó la operación
  -   Por ejemplo: **17f84be4-51f8-483a-b533-383791227a99**

### <a name="detailed-notification-items"></a>Elementos detallados de notificación
-   **Nombre para mostrar**: **(puede estar vacío)** un nombre para mostrar más detallado del error
    -   Por ejemplo: **Configuración del proxy de aplicación**
-   **Estado**: el estado específico de la notificación
    -   Por ejemplo: **Error**
-   **Identificador de objeto**: **(puede estar vacío)** el identificador del objeto en el que se realizó la operación
    -   Por ejemplo: **8e08161d-f2fd-40ad-a34a-a9632d6bb599**
-   **Detalles**: la descripción detallada de lo que se produjo como resultado de la operación
    -   Por ejemplo: **la dirección url interna `https://bing.com/` no es válida puesto que ya está en uso**
-   **Copiar error**: seleccione el **icono de copia** situado a la derecha del cuadro de texto **Copiar error** para copiar todos los detalles de la notificación y compartirlos con un grupo de producto o de soporte técnico. 
-   producto
    -   Por ejemplo, ```{"errorCode":"InternalUrl\_Duplicate","localizedErrorDetails":{"errorDetail":"Internal url 'https://google.com/' is invalid since it is already in use"},"operationResults":\[{"objectId":null,"displayName":null,"status":0,"details":"Internal url 'https://bing.com/' is invalid since it is already in use"}\],"timeStampUtc":"2017-03-23T19:50:26.465743Z","clientRequestId":"302fd775-3329-4670-a9f3-bea37004f0bb","internalTransactionId":"ea5b5475-03b9-4f08-8e95-bbb11289ab65","upn":"tperkins@f128.info","tenantId":"7918d4b5-0442-4a97-be2d-36f9f9962ece","userObjectId":"17f84be4-51f8-483a-b533-383791227a99"}```
