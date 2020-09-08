---
title: ¿Qué es un dispositivo unido a Azure AD?
description: Conozca cómo la administración de identidades de dispositivos puede ayudarle a administrar los dispositivos que acceden a los recursos de su entorno.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 31374b851d90e43e7380024923c3ad025859b7a0
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87923737"
---
# <a name="azure-ad-joined-devices"></a>Dispositivos unidos a Azure AD

La unión a Azure AD está pensaba para las organizaciones en las que la nube es prioritaria o exclusiva. Cualquier organización puede implementar dispositivos unidos a Azure AD, sin importar tamaño ni sector. La unión a Azure AD funciona incluso en un entorno híbrido y puede habilitar el acceso a aplicaciones y recursos de nube y locales.

| Azure AD Join | Descripción |
| --- | --- |
| **Definición** | Dispositivos unidos solo a Azure AD que requieren una cuenta de la organización para iniciar sesión en el dispositivo |
| **Público principal** | Adecuado tanto para organizaciones híbridas como para las que solo están en la nube. |
|   | Se aplica a todos los usuarios de una organización. |
| **Propiedad del dispositivo** | Organización |
| **Sistemas operativos** | Todos los dispositivos con Windows 10, excepto Windows 10 Home |
|   | [Máquinas virtuales Windows Server 2019 que se ejecutan en Azure](howto-vm-sign-in-azure-ad-windows.md) (No se admite Server Core) |
| **Aprovisionamiento** | Autoservicio: OOBE o Configuración de Windows |
|   | Inscripción masiva |
|   | Windows Autopilot |
| **Opciones de inicio de sesión en el dispositivo** | Cuentas organizativas que usan: |
|   | Contraseña |
|   | Windows Hello para empresas |
|   | Claves de seguridad de FIDO2.0 (versión preliminar) |
| **Administración de dispositivos** | Administración de dispositivos móviles (por ejemplo, Microsoft Intune) |
|   | Administración conjunta con Microsoft Intune y Microsoft Endpoint Configuration Manager |
| **Principales funcionalidades** | SSO tanto a los recursos en la nube como a los recursos locales |
|   | Acceso condicional a través de la inscripción de MDM y la evaluación de cumplimiento de MDM |
|   | Autoservicio de restablecimiento de contraseña y restablecimiento de PIN de Windows Hello |
|   | Enterprise State Roaming entre dispositivos |

Los dispositivos unidos a Azure AD inician sesión con una cuenta organizativa de Azure AD. Es posible limitar aún más el acceso a los recursos de la organización en función de esa cuenta de Azure AD y a las [directivas de acceso condicional](../conditional-access/howto-conditional-access-policy-compliant-device.md) aplicadas a la identidad del dispositivo.

Los administradores pueden proteger y controlar aún más los dispositivos unidos a Azure AD con herramientas de administración de dispositivos móviles (MDM), como Microsoft Intune o en escenarios de administración conjunta con Microsoft Endpoint Configuration Manager. Estas herramientas proporcionan una manera de aplicar las configuraciones que requiere la organización, como el cifrado del almacenamiento, la complejidad de las contraseñas y las instalaciones y actualizaciones de software. Los administradores pueden hacer que las aplicaciones de la organización estén disponibles en los dispositivos unidos a Azure AD con Configuration Manager para [administrar aplicaciones de Microsoft Store para Empresas y Educación](/configmgr/apps/deploy-use/manage-apps-from-the-windows-store-for-business).

La unión a Azure AD se puede lograr mediante opciones de autoservicio, como la configuración rápida (OOBE), la inscripción masiva o [Windows Autopilot](/intune/enrollment-autopilot).

Los dispositivos unidos a Azure AD todavía puede mantener el acceso de inicio de sesión único a los recursos locales cuando están en la red de la organización. Los dispositivos que están unidos a Azure AD todavía pueden autenticarse en los servidores locales como archivo, impresión y otras aplicaciones.

## <a name="scenarios"></a>Escenarios

Aunque la unión a Azure AD esté pensada principalmente para aquellas organizaciones que no tengan una infraestructura de Windows Server Active Directory local, sin duda se puede utilizar en escenarios donde:

- Quiere realizar la transición a la infraestructura basada en la nube con Azure AD y un sistema MDM, como Intune.
- No puede usar una unión a un dominio local, por ejemplo, si tiene que controlar dispositivos móviles como tabletas y teléfonos.
- Los usuarios necesitan acceder sobre todo a Office 365 u otras aplicaciones de SaaS integradas con Azure AD.
- Desea administrar un grupo de usuarios en Azure AD en lugar de en Active Directory. Este escenario se puede aplicar, por ejemplo, a los trabajadores temporales, contratistas o alumnos.
- Desea proporcionar capacidades de unión a los trabajadores de sucursales remotas con infraestructura local limitada.

Puede configurar dispositivos Unidos a Azure AD para todos los dispositivos con Windows 10, con la excepción de Windows 10 Home.

El objetivo de los dispositivos unidos a Azure AD es simplificar:

- Las implementaciones de Windows de los dispositivos de trabajo
- El acceso a recursos y aplicaciones de la organización desde cualquier dispositivo Windows
- Administración basada en la nube de dispositivos de trabajo
- El inicio de sesión de los usuarios en sus dispositivos con sus cuentas profesionales o educativas de Azure AD o de Active Directory sincronizadas.

![Dispositivos unidos a Azure AD](./media/concept-azure-ad-join/azure-ad-joined-device.png)

La unión a Azure AD se puede implementar mediante cualquiera de los métodos siguientes:

- [Windows Autopilot](/windows/deployment/windows-autopilot/windows-10-autopilot)
- [Implementación masiva](/intune/windows-bulk-enroll)
- [Experiencia de autoservicio](azuread-joined-devices-frx.md)

## <a name="next-steps"></a>Pasos siguientes

- [Planeación de la implementación de la unión a Azure AD](azureadjoin-plan.md)
- [Administración del grupo de administradores locales en dispositivos unidos a Azure AD](assign-local-admin.md)
- [Administración de identidades de dispositivos con Azure Portal](device-management-azure-portal.md)
- [Administración de dispositivos obsoletos en Azure AD](manage-stale-devices.md)
