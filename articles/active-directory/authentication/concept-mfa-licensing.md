---
title: Versiones y planes de consumo de Azure Multi-Factor Authentication
description: Conozca el cliente de Multi-Factor Authentication, así como los distintos métodos y versiones disponibles.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 06/15/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: e294aa2580dd9f0cceb539a2696e7ebbf08c4afe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87051014"
---
# <a name="features-and-licenses-for-azure-multi-factor-authentication"></a>Características y licencias de Azure Multi-Factor Authentication

Para proteger las cuentas de usuario de la organización, se debe usar la autenticación multifactor. Esta característica resulta especialmente importante en las cuentas que tienen acceso con privilegios a los recursos. Los administradores de Microsoft 365 y Azure Active Directory (Azure AD) disponen de las características básicas de la autenticación multifactor sin cargo adicional. Si quiere actualizar las características para los administradores o extender la autenticación multifactor al resto de los usuarios, puede comprar Azure Multi-Factor Authentication de varias maneras.

> [!IMPORTANT]
> En este artículo se detallan las diferentes formas en las que se puede obtener la licencia de Azure Multi-Factor Authentication y cómo se puede usar. Para información específica sobre precios y facturación, consulte la [página de precios de Azure Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/).

## <a name="available-versions-of-azure-multi-factor-authentication"></a>Versiones disponibles de Azure Multi-Factor Authentication

La licencia de Azure Multi-Factor Authentication se puede conseguir de varias maneras, en función de las necesidades de la organización. Esto también se aplica a la forma en la que se usa Azure Multi-Factor Authentication. Es posible que ya tenga derecho a usar Azure Multi-Factor Authentication, dependiendo de la licencia de Azure AD, EMS o Microsoft 365 que tenga actualmente. En la tabla siguiente se indican las diferentes formas de obtener Azure Multi-Factor Authentication y algunas de las características y los casos de uso de cada uno de ellas.

| Si es usuario de | Funcionalidades y casos de uso |
| --- | --- |
| Microsoft 365 Empresa Premium y EMS o Microsoft 365 E3 y E5 | EMS E3, Microsoft 365 E3 y Microsoft 365 Empresa Premium incluye Azure AD Premium P1. EMS E5 o Microsoft 365 E5 incluyen Azure AD Premium P2. Puede usar las mismas características de acceso condicional que se indican en las siguientes secciones para proporcionar la autenticación multifactor a los usuarios. |
| Azure AD Premium P1 | Puede usar el [acceso condicional de Azure AD](../conditional-access/howto-conditional-access-policy-all-users-mfa.md) para solicitar a los usuarios la autenticación multifactor en determinados escenarios o eventos y adaptarse así a los requisitos empresariales. |
| Azure AD Premium P2 | Proporciona la opción de seguridad más potente y una experiencia de usuario mejorada. Agrega [acceso condicional basado en riesgos](../conditional-access/howto-conditional-access-policy-risk.md) a las características de Azure AD Premium P1; se adapta a los patrones del usuario y reduce el número de solicitudes de la autenticación multifactor. |
| Todos los planes de Microsoft 365 | Azure Multi-Factor Authentication se puede [habilitar por usuario](howto-mfa-userstates.md), o habilitar o deshabilitar para todos los usuarios mediante los [valores predeterminados de seguridad](../fundamentals/concept-fundamentals-security-defaults.md). La administración de Azure Multi-Factor Authentication se realiza a través del portal de Microsoft 365. Para mejorar la experiencia del usuario, actualice a Azure AD Premium P1 o P2 y use el acceso condicional. Para obtener más información, consulte el artículo sobre [la protección de los recursos de Microsoft 365 con la autenticación multifactor](/microsoft-365/admin/security-and-compliance/set-up-multi-factor-authentication). |
| Azure AD Gratis | Puede usar [valores predeterminados de seguridad](../fundamentals/concept-fundamentals-security-defaults.md) para habilitar la autenticación multifactor para todos los usuarios. No tendrá un control pormenorizado de los usuarios o los escenarios habilitados, pero conseguirá ese paso de seguridad adicional.<br /> Incluso cuando no se usan los valores predeterminados de seguridad para habilitar la autenticación multifactor para todos los usuarios, se puede configurar el rol de *administrador global de Azure AD* para usar la autenticación multifactor. Esta característica del nivel de servicio gratuito garantiza que las cuentas de administrador críticas están protegidas por la autenticación multifactor. |

## <a name="feature-comparison-of-versions"></a>Comparación de características de las versiones

En la tabla siguiente se proporciona una lista de las características que están disponibles en las distintas versiones de Azure Multi-Factor Authentication. Planee sus necesidades de protección para la autenticación de usuarios y, a continuación, determine qué enfoque cumple esos requisitos. Por ejemplo, aunque Azure AD Free aporte valores predeterminados de seguridad que proporcionan Azure Multi-Factor Authentication, en la solicitud de autenticación solo se puede usar la aplicación de autenticación para dispositivos móviles, no una llamada telefónica ni un SMS. Este enfoque puede ser una limitación si no se puede garantizar que la aplicación de autenticación para dispositivos móviles esté instalada en el dispositivo personal de un usuario.

| Característica | Azure AD Free: valores predeterminados de seguridad | Azure AD Free: administradores globales de Azure AD | Aplicaciones de Office 365 | Azure AD Premium (P1 o P2) |
| --- |:---:|:---:|:---:|:---:|
| Protección de cuentas de administración de inquilinos de Azure AD con MFA | ● | ● (solo cuentas de *administrador global de Azure AD*) | ● | ● |
| Aplicación móvil como segundo factor | ● | ● | ● | ● |
| Llamada de teléfono como segundo factor | | ● | ● | ● |
| SMS como segundo factor | | ● | ● | ● |
| Control administrativo sobre métodos de comprobación | | ● | ● | ● |
| Alerta de fraude | | | | ● |
| Informes de MFA | | | | ● |
| Saludos personalizados para las llamadas de teléfono | | | | ● |
| Identificador de llamada personalizado para llamadas telefónicas | | | | ● |
| IP de confianza | | | | ● |
| Recordar MFA para dispositivos de confianza | | ● | ● | ● |
| MFA para aplicaciones locales | | | | ● |

## <a name="purchase-and-enable-azure-multi-factor-authentication"></a>Compra y habilitación de Azure Multi-Factor Authentication

Para usar Azure Multi-Factor Authentication, regístrese o compre un nivel de Azure AD válido. Azure AD está disponible en cuatro ediciones: Gratis, aplicaciones de Office 365, Premium P1 y Premium P2.

La edición Free se incluye con una suscripción de Azure. Consulte la [sección siguiente](#azure-ad-free-tier) para obtener información sobre cómo usar los valores predeterminados de seguridad o cómo proteger las cuentas con el rol de *administrador global de Azure AD*.

Las ediciones de Azure AD Premium están disponibles a través del representante de Microsoft, el [programas de licencias por volumen Open](https://www.microsoft.com/licensing/licensing-programs/open-license.aspx) y el [programa de proveedores de soluciones en la nube](https://go.microsoft.com/fwlink/?LinkId=614968&clcid=0x409). Los suscriptores de Azure y Microsoft 365 también pueden comprar Azure Active Directory Premium P1 y P2 en línea. [Inicie sesión](https://portal.office.com/Commerce/Catalog.aspx) para comprar.

Una vez que haya adquirido el nivel de Azure AD necesario, [planee e implemente Azure Multi-Factor Authentication](howto-mfa-getstarted.md).

### <a name="azure-ad-free-tier"></a>Nivel Azure AD Free

Todos los usuarios de un inquilino de Azure AD Free pueden usar Azure Multi-Factor Authentication mediante el uso de los valores predeterminados de seguridad. Cuando se usan los valores predeterminados de seguridad de Azure AD Free, el único método que se puede usar con Azure Multi-Factor Authentication es la aplicación de autenticación para dispositivos móviles.

* [Más información sobre los valores predeterminados de seguridad de Azure AD](../fundamentals/concept-fundamentals-security-defaults.md)
* [Habilitación de los valores predeterminados de seguridad para los usuarios de Azure AD Free](../fundamentals/concept-fundamentals-security-defaults.md#enabling-security-defaults)

Si no desea habilitar Azure Multi-Factor Authentication para todos los usuarios, puede proteger solo las cuentas de usuario con el rol de *administrador global de Azure AD*. Este enfoque proporciona solicitudes de autenticación adicionales para las cuentas de administrador críticas. Puede habilitar Azure Multi-Factor Authentication de una de las siguientes maneras, según el tipo de cuenta que use:

* Si usa una cuenta Microsoft, [regístrese para la autenticación multifactor](https://support.microsoft.com/help/12408/microsoft-account-about-two-step-verification).
* Si no usa una cuenta Microsoft, [active la autenticación multifactor para un usuario o grupo en Azure AD](howto-mfa-userstates.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los costos, consulte [Precios de Azure Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/).
* [¿Qué es el acceso condicional?](../conditional-access/overview.md)

