---
title: Búsqueda de una API específica necesaria para una aplicación desarrollada internamente | Microsoft Docs
description: Configuración de los permisos necesarios para acceder a una API determinada en una aplicación desarrollada internamente con Azure AD
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: ryanwi
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4a017c13008288b26ddb11bf58be1966d652bbae
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540551"
---
# <a name="how-to-find-a-specific-api-needed-for-a-custom-developed-application"></a>Búsqueda de una API específica necesaria para una aplicación desarrollada internamente

El acceso a API requiere configurar los ámbitos y roles de acceso. Si desea exponer su API web de aplicaciones de recursos a aplicaciones cliente, debe configurar los roles y ámbitos de acceso de la API. Si desea que una aplicación cliente pueda acceder a una API web, debe configurar los permisos de acceso a la API en el registro de la aplicación.

## <a name="configuring-a-resource-application-to-expose-web-apis"></a>Configuración de una aplicación de recursos para exponer las API web

Al exponer la API web, la API se muestra en la lista **Seleccionar una API** al agregar permisos al registro de una aplicación. Para agregar ámbitos de acceso, siga los pasos descritos en [Agregar ámbitos de acceso a la aplicación de recursos](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications).

## <a name="configuring-a-client-application-to-access-web-apis"></a>Configuración de una aplicación cliente para acceder a las API web

Al agregar permisos al registro de la aplicación, también puede **agregar acceso a API** a las API web expuestas. Para acceder a API web, siga los pasos descritos en [Para agregar credenciales o permisos para acceder a las API web](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications).

## <a name="next-steps"></a>Pasos siguientes

-   [Integración de aplicaciones con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)

-   [Descripción del manifiesto de aplicación de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-application-manifest)


