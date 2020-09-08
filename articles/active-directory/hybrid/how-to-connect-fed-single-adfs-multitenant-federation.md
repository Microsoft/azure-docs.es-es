---
title: Federación de varias instancias de Azure AD con una instancia única de AD FS - Azure
description: En este documento aprenderá a federar varias instancias de Azure AD con una única de AD FS.
keywords: federar, ADFS, AD FS, varios inquilinos, AD FS único, un ADFS, federación de varios inquilinos, adfs de varios bosques, aad connect, federación, federación entre inquilinos
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/17/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 956428b6f197912e2ab7c3a94133ed9d59f37749
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279931"
---
# <a name="federate-multiple-instances-of-azure-ad-with-single-instance-of-ad-fs"></a>Federación de varias instancias de Azure AD con una instancia única de AD FS

Una sola granja de AD FS de alta disponibilidad puede federar varios bosques si existe confianza bidireccional entre ellos. Estos bosques pueden corresponderse a la misma instancia de Azure Active Directory o no. En este artículo se proporcionan instrucciones para configurar la federación entre una sola implementación de AD FS y varios bosques que se sincronicen con instancias diferentes de Azure AD.

![Federación de varios inquilinos con una sola instancia de AD FS](./media/how-to-connect-fed-single-adfs-multitenant-federation/concept.png)
 
> [!NOTE]
> No se admiten la escritura diferida de dispositivos ni la unión automática de dispositivos en este escenario.

> [!NOTE]
> No se puede usar Azure AD Connect para configurar la federación en este escenario porque Azure AD Connect puede configurar la federación para dominios en una única instancia de Azure AD.

## <a name="steps-for-federating-ad-fs-with-multiple-azure-ad"></a>Pasos para la federación de AD FS con varias instancias de Azure AD

Piense en un dominio contoso.com en una instancia de Azure Active Directory, contoso.onmicrosoft.com, que ya está federado con la instancia de AD FS local instalada en el entorno de Active Directory local de contoso.com. Fabrikam.com es un dominio en la instancia de Azure Active Directory fabrikam.onmicrosoft.com.

## <a name="step-1-establish-a-two-way-trust"></a>Paso 1: Establecimiento de una confianza bidireccional
 
Para que la instancia de AD FS en contoso.com pueda autenticar a usuarios en fabrikam.com, se necesita una confianza bidireccional entre contoso.com y fabrikam.com. Siga las instrucciones en este [artículo](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc816590(v=ws.10)) para crear dicha confianza bidireccional.
 
## <a name="step-2-modify-contosocom-federation-settings"></a>Paso 2: Modificación de la configuración de federación de contoso.com 
 
El emisor predeterminado establecido para un único dominio federado en AD FS es "http\://ADFSServiceFQDN/adfs/services/trust", por ejemplo, `http://fs.contoso.com/adfs/services/trust`. Azure Active Directory requiere un emisor único para cada dominio federado. Puesto que la misma instancia de AD FS va a federar dos dominios, se debe modificar el valor de emisor para que sea único para cada dominio que AD FS federe con Azure Active Directory. 
 
En el servidor de AD FS, abra Azure AD PowerShell (asegúrese de que está instalado el módulo MSOnline) y realice los pasos siguientes:
 
Conéctese a la instancia de Azure Active Directory que contiene el dominio contoso.com. Connect-MsolService Actualice la configuración de federación de contoso.com. Update-MsolFederatedDomain -DomainName contoso.com –SupportMultipleDomain
 
El emisor en la configuración de federación del dominio se cambiará a "http\://contoso.com/adfs/services/trust" y se agregará una regla de notificaciones de emisión para que la relación de confianza para usuario autenticado de Azure AD emita el valor de issuerId correcto basándose en el sufijo UPN.
 
## <a name="step-3-federate-fabrikamcom-with-ad-fs"></a>Paso 3: Federación de fabrikam.com con AD FS
 
En la sesión de PowerShell de Azure AD, realice los pasos siguientes: Conéctese a la instancia de Azure Active Directory que contiene el dominio fabrikam.com.

```powershell
Connect-MsolService
```
Convierta el dominio administrado fabrikam.com en federado:

```powershell
Convert-MsolDomainToFederated -DomainName fabrikam.com -Verbose -SupportMultipleDomain
```
 
La operación anterior federará el dominio fabrikam.com con la misma instancia de AD FS. Puede comprobar la configuración del dominio mediante Get-MsolDomainFederationSettings para ambos dominios.

## <a name="next-steps"></a>Pasos siguientes
[Conexión de Active Directory con Azure Active Directory](whatis-hybrid-identity.md)