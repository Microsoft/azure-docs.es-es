---
title: 'Almacenamiento de datos de identidad para clientes europeos: Azure AD'
description: Obtenga información sobre dónde almacena Azure Active Directory los datos relacionados con la identidad de los clientes Europaos.
services: active-directory
author: ajburnle
manager: daveba
ms.author: ajburnle
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 03/04/2019
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 99d8ae8c5e3218959a483e78f70a6502bf7a5a47
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87799113"
---
# <a name="identity-data-storage-for-european-customers-in-azure-active-directory"></a>Almacenamiento de datos de identidad para los clientes Europaos en Azure Active Directory
Azure AD almacena los datos de identidad en una ubicación geográfica en función de la dirección que proporcione la organización al suscribirse a un servicio de Microsoft Online, como Office 365 y Azure. Para obtener información sobre dónde se almacenan los datos de identidad, puede consultar la sección [Where is your data located?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) (Dónde se encuentran sus datos) de Microsoft Trust Center.

Para los clientes que proporcionaron una dirección en Europa, Azure AD conserva la mayoría de los datos de identidad en centros de datos europeos. En este documento se proporciona información sobre los datos que los servicios de Azure AD almacenan fuera de Europa.

## <a name="microsoft-azure-multi-factor-authentication-mfa"></a>Microsoft Azure Multi-Factor Authentication (MFA)
    
- La autenticación en dos fases que use llamadas telefónicas o mensajes de texto proceden de los centros de datos de EE. UU. y también se enruta mediante proveedores internacionales.
- Las notificaciones de inserción que usan la aplicación Microsoft Authenticator proceden de los centros de datos de EE. UU. Además, es posible que se usen servicios específicos del proveedor del dispositivo y estos servicios podrían encontrarse fuera de Europa.
- Los códigos OATH se validan siempre en EE. UU. 

Para más información sobre qué información de usuario recopila el Servidor Microsoft Azure Multi-Factor Authentication (Servidor MFA) y Azure MFA basada en la nube, consulte [Recopilación de datos de usuario de Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-reporting-datacollection).

## <a name="microsoft-azure-active-directory-b2c-azure-ad-b2c"></a>Microsoft Azure Active Directory B2C (Azure AD B2C)

Los datos de configuración de directivas y contenedores de claves de Azure AD B2C se almacenan en los centros de datos de EE. UU. Estos no contienen datos personales de los usuarios. Para obtener más información sobre las configuraciones de directivas, consulte el artículo [Azure Active Directory B2C: directivas integradas](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-policies).

## <a name="microsoft-azure-active-directory-b2b-azure-ad-b2b"></a>Microsoft Azure Active Directory B2B (Azure AD B2B) 
    
Azure AD B2B almacena las invitaciones con vínculos de canje y la información sobre la dirección URL de redirección en centros de datos de EE. UU. Además, la dirección de correo electrónico de los usuarios que cancelan la suscripción a invitaciones B2B también se almacena en centros de datos de EE. UU.

## <a name="microsoft-azure-active-directory-domain-services-azure-ad-ds"></a>Microsoft Azure Active Directory Domain Services (Azure AD DS)

Azure AD DS almacena los datos de usuario en la misma ubicación que la instancia de Azure Virtual Network que seleccionó el cliente. Por lo tanto, si la red está fuera de Europa, los datos se replican y almacenan fuera de Europa.

## <a name="federation-in-microsoft-exchange-server-2013"></a>Federación en Microsoft Exchange Server 2013
    
- Identificador de la aplicación (AppID): un número único que genera el sistema de autenticación de Azure Active Directory para identificar las organizaciones de Exchange.
- Lista de dominios federados aprobados para la aplicación
- Clave pública de firma de tokens de la aplicación 

Para más información acerca de la federación en Microsoft Exchange Server, consulte el artículo [Federación: Ayuda de Exchange 2013](https://docs.microsoft.com/exchange/federation-exchange-2013-help).


## <a name="other-considerations"></a>Otras consideraciones

Los servicios y las aplicaciones que se integran en Azure AD tienen acceso a datos de identidad. Evalúe cada servicio y aplicación que use para determinar la manera en que cada servicio y aplicación específico procesa los datos de identidad, y si cumple los requisitos de almacenamiento de datos de la empresa.

Para obtener más información sobre la residencia de los datos de los servicios de Microsoft, consulte la sección [Where is your data located?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) (Dónde se encuentran sus datos) de Microsoft Trust Center.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre cualquiera de las características y funcionalidades que se han descrito anteriormente, consulte estos artículos:
- [¿Qué es Multi-Factor Authentication?](https://docs.microsoft.com/azure/active-directory/authentication/multi-factor-authentication)

- [Autoservicio de restablecimiento de contraseña de Azure AD](https://docs.microsoft.com/azure/active-directory/authentication/active-directory-passwords-overview)

- [¿Qué es Azure Active Directory B2C?](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview)

- [¿Qué es la colaboración B2B de Azure AD?](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b)

- [Azure Active Directory (AD) Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-overview)
