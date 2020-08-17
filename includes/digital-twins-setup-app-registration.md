---
author: baanders
description: archivo de inclusión para el paso de permisos de acceso en la configuración de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/17/2020
ms.author: baanders
ms.openlocfilehash: 18aa039977beeff3f6b16249822fbd98370ca404
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87408177"
---
Una vez que configura una instancia de Azure Digital Twins, es habitual interactuar con esa instancia a través de una aplicación cliente. Para crear una aplicación cliente que funcione, deberá asegurarse de que la aplicación cliente pueda autenticarse en Azure Digital Twins. Esto se hace mediante la configuración de un **registro de aplicaciones** de [Azure Active Directory (Azure AD)](../articles/active-directory/fundamentals/active-directory-whatis.md) para que lo use la aplicación cliente.

Este registro de la aplicación es donde se configuran los permisos de acceso a las [API de Azure Digital Twins](../articles/digital-twins/how-to-use-apis-sdks.md). Más tarde, la aplicación cliente se autentica en el registro de la aplicación y, como resultado, se le conceden los permisos de acceso configurados a las API.

>[!TIP]
> Como propietario de una suscripción, puede que prefiera configurar un registro de aplicaciones nuevo para cada instancia nueva de Azure Digital Twins *o*, para hacer esto solo una vez, mediante el establecimiento de un registro de aplicaciones único que se compartirá entre todas las instancias de Azure Digital Twins en la suscripción.

### <a name="create-the-registration"></a>Creación del registro