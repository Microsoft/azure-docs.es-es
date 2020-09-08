---
title: ¿Qué es la identidad híbrida con Azure Active Directory?
description: La identidad híbrida consiste en tener una identidad de usuario común para la autenticación y autorización tanto a nivel local como en la nube.
keywords: introducción a Azure AD Connect, información general de Azure AD Connect, qué es Azure AD Connect, instalación de active directory
services: active-directory
author: billmath
manager: daveba
ms.assetid: 59bd209e-30d7-4a89-ae7a-e415969825ea
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 05/17/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: f74798eeed4764bf2f7b71cb5049fb1c6b3ae077
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89278656"
---
# <a name="what-is-hybrid-identity-with-azure-active-directory"></a>¿Qué es la identidad híbrida con Azure Active Directory?

En la actualidad, tanto en las empresas como en las organizaciones se puede encontrar cada vez más una combinación de aplicaciones locales y en la nube.  Los usuarios requieren acceso a esas aplicaciones tanto en el entorno local como en la nube. La administración de usuarios tanto a nivel local como en la nube plantea escenarios complicados. 

Las soluciones de identidad de Microsoft abarcan funcionalidades locales y basadas en la nube.  Estas soluciones crean una identidad de usuario común para la autenticación y autorización en todos los recursos, independientemente de la ubicación. A esto lo llamamos **identidad híbrida**.

Con la identidad híbrida en Azure AD y su administración, se pueden abordar estos escenarios.

Para lograr la identidad híbrida con Azure AD, se puede usar uno de tres métodos de autenticación, en función de los escenarios.   Los tres métodos son: 

- **[Sincronización de hash de contraseña (PHS)](whatis-phs.md)**  
- **[Autenticación de paso a través (PTA)](how-to-connect-pta.md)**  
- **[Federación (AD FS)](whatis-fed.md)** 

Estos métodos de autenticación también proporcionan funcionales de [inicio de sesión único](how-to-connect-sso.md).  El inicio de sesión único permite que los usuarios inicien sesión automáticamente cuando usan sus dispositivos corporativos y están conectados a la red de la empresa.

Para información adicional, consulte [Selección del método de autenticación adecuado para la solución de identidad híbrida de Azure Active Directory](./choose-ad-authn.md). 

## <a name="common-scenarios-and-recommendations"></a>Escenarios comunes y recomendaciones 

Estos son algunos escenarios comunes de identidad híbrida y administración de acceso con recomendaciones sobre la opción (u opciones) de identidad que podría ser más adecuada. 

|Necesitará:|PHS y SSO<sup>1</sup>| PTA y SSO<sup>2</sup> | AD FS<sup>3</sup>| 
|-----|-----|-----|-----| 
|Sincronice nuevas cuentas de usuarios, contactos y grupos creadas en Active Directory local con la nube automáticamente.|![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| ![Recomendado](./media/whatis-hybrid-identity/ic195031.png) |![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| 
|Configurar mi inquilino para escenarios híbridos de Office 365|![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| ![Recomendado](./media/whatis-hybrid-identity/ic195031.png) |![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| 
|Permitir que mis usuarios inicien sesión en Cloud Services y accedan con su contraseña local.|![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| ![Recomendado](./media/whatis-hybrid-identity/ic195031.png) |![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| 
|Implementar el inicio de sesión único mediante credenciales corporativas.|![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| ![Recomendado](./media/whatis-hybrid-identity/ic195031.png) |![Recomendado](./media/whatis-hybrid-identity/ic195031.png)|  
|Asegurarse de que no se almacenan hashes de contraseña en la nube.| |![Recomendado](./media/whatis-hybrid-identity/ic195031.png)|![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| 
|Habilitar soluciones de autenticación multifactor basadas en la nube.|![Recomendado](./media/whatis-hybrid-identity/ic195031.png)|![Recomendado](./media/whatis-hybrid-identity/ic195031.png)|![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| 
|Habilitar soluciones locales de autenticación multifactor| | |![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| 
|Admitir la autenticación mediante tarjeta inteligente para mis usuarios.<sup>4</sup>| | |![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| 
|Mostrar notificaciones de expiración de contraseña en el Portal de Office y en el escritorio de Windows 10.| | |![Recomendado](./media/whatis-hybrid-identity/ic195031.png)| 

> <sup>1</sup> Sincronización de hash de contraseñas con inicio de sesión único. 
> 
> <sup>2</sup> Autenticación de paso a través e inicio de sesión único.  
> 
> <sup>3</sup> Inicio de sesión único federado con AD FS.  
>  
> <sup>4</sup> AD FS se puede integrar con el PKI de la empresa para permitir el inicio de sesión con certificados. Estos certificados pueden ser certificados flexibles implementados a través de canales de aprovisionamiento de confianza, como certificados MDM o GPO o de tarjetas inteligentes (incluidas las tarjetas PIV/CAC) o Hello para empresas (cert-trust). Para más información sobre la compatibilidad con la autenticación de tarjetas inteligentes, consulte [este blog](/archive/blogs/samueld/adfs-certauth-aad-o365). 
> 

## <a name="license-requirements-for-using-azure-ad-connect"></a>Requisitos de licencia para usar Azure AD Connect

[!INCLUDE [active-directory-free-license.md](../../../includes/active-directory-free-license.md)]

## <a name="next-steps"></a>Pasos siguientes 

- [¿Qué es Azure AD Connect y Connect Health?](whatis-azure-ad-connect.md) 
- [¿Qué es la sincronización de hash de contraseñas (PHS)?](whatis-phs.md) 
- [¿Qué es la autenticación de paso a través (PTA)?](how-to-connect-pta.md) 
- [¿Qué es la federación?](whatis-fed.md) 
- [¿Qué es el inicio de sesión único?](how-to-connect-sso.md)