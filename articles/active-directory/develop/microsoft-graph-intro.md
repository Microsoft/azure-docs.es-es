---
title: Microsoft Graph API
description: Microsoft Graph API es una API web de RESTful que le permite tener acceso a recursos de servicio de Microsoft Cloud.
author: davidmu1
services: active-directory
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 02/13/2020
ms.author: davidmu
ms.custom: aaddev
ms.openlocfilehash: 13cbeca909b445d75fcbcf7b3751183421479d42
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90052569"
---
# <a name="microsoft-graph-api"></a>Microsoft Graph API

Microsoft Graph API es una API web de RESTful que le permite tener acceso a recursos de servicio de Microsoft Cloud. Después de registrar la aplicación y obtener los tokens de autenticación para un usuario o servicio, puede realizar solicitudes a Microsoft Graph API. Para más información, consulte [Introducción a Microsoft Graph](/graph/overview).

Microsoft Graph expone las API REST y las bibliotecas cliente para obtener acceso a los datos en los siguientes servicios de Microsoft 365:
- Servicios de Microsoft 365: Delve, Excel, Microsoft Bookings, Microsoft Teams, OneDrive, OneNote, Outlook/Exchange, Planner y SharePoint
- Servicios de Enterprise Mobility + Security: Advanced Threat Analytics, Advanced Threat Protection, Azure Active Directory, Identity Manager e Intune
- Servicios de Windows 10: actividades, dispositivos y notificaciones
- Dynamics 365 Business Central

## <a name="versions"></a>Versiones

Actualmente Microsoft Graph admite dos versiones: v1.0 y la versión beta. La versión v1.0 incluye API de disponibilidad general. Use la versión v1.0 para todas las aplicaciones de producción. La versión beta incluye las API que están en versión preliminar actualmente. Dado que podríamos introducir cambios importantes en nuestras API de la versión beta, se recomienda usar la versión beta solo para probar las aplicaciones que están en desarrollo. No use las API de la versión beta en las aplicaciones de producción. Para obtener más información, consulte [Control de versiones, compatibilidad y directivas de cambios importantes en Microsoft Graph](/graph/versioning-and-support).

Para empezar a usar las API de la versión beta, consulte [Referencia de puntos de conexión de la versión beta de Microsoft Graph](/graph/api/overview?view=graph-rest-beta).

Para empezar a usar las API de la versión v1.0, consulte [Referencia de la API REST de Microsoft Graph versión v1.0](/graph/api/overview?view=graph-rest-1.0).

## <a name="get-started"></a>Introducción

Para leer o escribir en un recurso, como un usuario o un mensaje de correo electrónico, construya una solicitud similar a la siguiente:

`{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}`

Para obtener más información sobre los elementos de la solicitud construida, consulte [Uso de Microsoft Graph API](/graph/use-the-api).

Los ejemplos de inicio rápido están disponibles para mostrarle cómo acceder a la eficacia de Microsoft Graph API. Los ejemplos que están disponibles tienen acceso a dos servicios con una autenticación: Cuenta de Microsoft y Outlook. Cada inicio rápido tiene acceso a información de los perfiles de usuarios de cuentas de Microsoft y muestra los eventos de su calendario.
El inicio rápido implica cuatro pasos:
- Seleccione la plataforma.
- Obtenga el id. de la aplicación (Id. de cliente).
- Compilación del ejemplo
- Inicie sesión y vea los eventos en el calendario.

Al completar la guía de inicio rápido, tiene una aplicación que está lista para ejecutarse. Para más información, consulte las [Preguntas más frecuentes sobre el inicio rápido de Microsoft Graph](/graph/quick-start-faq). Para empezar a trabajar con los ejemplos, consulte [Inicio rápido de Microsoft Graph](https://developer.microsoft.com/graph/quick-start).

## <a name="tools"></a>Herramientas

Microsoft Graph Explorer es una herramienta basada en la web que puede usar para compilar y probar solicitudes con Microsoft Graph API. Puede tener acceso a Microsoft Graph Explorer en `https://developer.microsoft.com/graph/graph-explorer`.

Postman es una herramienta que también puede usar para compilar y probar solicitudes con Microsoft Graph API. Puede descargar Postman en `https://www.getpostman.com/`. Para interactuar con Microsoft Graph en Postman, use la colección de Microsoft Graph en Postman. Para más información, consulte [Uso de Postman con Microsoft Graph API](/graph/use-postman?context=graph%2Fapi%2Fbeta&view=graph-rest-beta).