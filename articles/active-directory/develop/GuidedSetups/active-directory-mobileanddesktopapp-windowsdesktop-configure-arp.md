---
title: 'Introducción al escritorio de Windows en Azure AD v2: configuración | Microsoft Docs'
description: Cómo puede obtener una aplicación .NET de escritorio de Windows (XAML) un token de acceso y llamar a una API protegida por un punto de conexión de Azure Active Directory v2.
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/09/2017
ms.author: andret
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: ef32cef67ae5f32959e026157f5126647b01401d
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540581"
---
# <a name="add-the-applications-registration-information-to-your-app"></a>Adición de información de registro de la aplicación a su aplicación
En este paso, debe agregar el identificador de aplicación a su proyecto.

1.  Abra `App.xaml.cs` y reemplace la línea que contiene el `ClientId` con:

```csharp
private static string ClientId = "[Enter the application Id here]";
```

### <a name="what-is-next"></a>Pasos siguientes

[!INCLUDE [Test and Validate](../../../../includes/active-directory-develop-guidedsetup-windesktop-test.md)]
