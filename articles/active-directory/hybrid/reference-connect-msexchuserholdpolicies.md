---
title: 'Azure AD Connect: msExchUserHoldPolicies y cloudMsExchUserHoldPolicies | Microsoft Docs'
description: En este tema se describe el comportamiento de atributo de los atributos msExchUserHoldPolicies y cloudMsExchUserHoldPolicies
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 08/23/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e4950906e016b9e1915d18ecacc1edcfda8b4d09
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279404"
---
# <a name="azure-ad-connect---msexchuserholdpolicies-and-cloudmsexchuserholdpolicies"></a>Azure AD Connect: msExchUserHoldPolicies y cloudMsExchUserHoldPolicies
En el documento de referencia siguiente se describen estos atributos usados por Exchange y la manera adecuada de editar las reglas de sincronización predeterminadas.

## <a name="what-are-msexchuserholdpolicies-and-cloudmsexchuserholdpolicies"></a>¿Qué son msExchUserHoldPolicies y cloudMsExchUserHoldPolicies?
Hay dos tipos de [suspensiones](/Exchange/policy-and-compliance/holds/holds?view=exchserver-2019) para una instancia de Exchange Server: suspensión por litigio y conservación local. Cuando se habilita la suspensión por litigio, todos los elementos del buzón se colocan en suspensión.  Una conservación local se usa para conservar solo los elementos que cumplen los criterios de una consulta de búsqueda que se ha definido mediante la herramienta eDiscovery local.

Los atributos MsExchUserHoldPolcies y cloudMsExchUserHoldPolicies permiten a AD y Azure AD locales determinar qué usuarios están sujetos a una suspensión en función de si usan Exchange local o Exchange en línea.

## <a name="msexchuserholdpolicies-synchronization-flow"></a>Flujo de sincronización de msExchUserHoldPolicies
De forma predeterminada, MsExchUserHoldPolcies se sincroniza mediante Azure AD Connect directamente con el atributo msExchUserHoldPolicies del metaverso y, después, con el atributo msExchUserHoldPolices en Azure AD

Las tablas siguientes describen el flujo:

Entrada desde Active Directory local:

|Atributo de Active Directory|Nombre del atributo|Tipo de flujo|Atributo de metaverso|Regla de sincronización|
|-----|-----|-----|-----|-----|
|Active Directory local|msExchUserHoldPolicies|Directo|msExchUserHoldPolices|Entrada desde AD: intercambio de usuarios|

Salida a Azure AD:

|Atributo de metaverso|Nombre del atributo|Tipo de flujo|Atributo de Azure AD|Regla de sincronización|
|-----|-----|-----|-----|-----|
|Azure Active Directory|msExchUserHoldPolicies|Directo|msExchUserHoldPolicies|Salida a AAD: UserExchangeOnline|

## <a name="cloudmsexchuserholdpolicies-synchronization-flow"></a>Flujo de sincronización de cloudMsExchUserHoldPolicies
De forma predeterminada, cloudMsExchUserHoldPolicies se sincroniza mediante Azure AD Connect directamente con el atributo cloudMsExchUserHoldPolicies del metaverso. A continuación, si msExchUserHoldPolices no es null en el metaverso, el atributo fluirá a Active Directory.

Las tablas siguientes describen el flujo:

Entrada desde Azure AD:

|Atributo de Active Directory|Nombre del atributo|Tipo de flujo|Atributo de metaverso|Regla de sincronización|
|-----|-----|-----|-----|-----|
|Active Directory local|cloudMsExchUserHoldPolicies|Directo|cloudMsExchUserHoldPolicies|Entrada desde AAD: intercambio de usuarios|

Salida a Active Directory local:

|Atributo de metaverso|Nombre del atributo|Tipo de flujo|Atributo de Azure AD|Regla de sincronización|
|-----|-----|-----|-----|-----|
|Azure Active Directory|cloudMsExchUserHoldPolicies|IF(NOT NULL)|msExchUserHoldPolicies|Salida a AD: UserExchangeOnline|

## <a name="information-on-the-attribute-behavior"></a>Información sobre el comportamiento del atributo
msExchangeUserHoldPolicies es un atributo de autoridad única.  Se puede establecer un atributo de autoridad única en un objeto (en este caso, el objeto de usuario) en el directorio local o en el directorio en la nube.  Las reglas de inicio de autoridad indican que si el atributo se sincroniza desde el entorno local, Azure AD no podrá actualizar este atributo.

Para permitir que los usuarios establezcan una directiva de conservación en un objeto de usuario en la nube, se usa el atributo cloudMSExchangeUserHoldPolicies. Este atributo se usa porque Azure AD no puede establecer msExchangeUserHoldPolicies directamente en función de las reglas descritas anteriormente.  Este atributo se sincronizará de nuevo con el directorio local si, msExchangeUserHoldPolicies no es NULL y reemplaza al valor actual de msExchangeUserHoldPolicies.

En determinadas circunstancias, por ejemplo, si ambos se cambiaron en el entorno local y en Azure al mismo tiempo, esto podría causar algunos problemas.  

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).