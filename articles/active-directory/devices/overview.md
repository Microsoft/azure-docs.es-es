---
title: ¿Qué es la identidad de dispositivos de Azure Active Directory?
description: Conozca cómo la administración de identidades de dispositivos puede ayudarle a administrar los dispositivos que acceden a los recursos de su entorno.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: overview
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8399f7101697af429b8c073c101dbfea203e98ea
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87025598"
---
# <a name="what-is-a-device-identity"></a>¿Qué es una identidad de dispositivo?

Con la proliferación de dispositivos de todas las formas y tamaños, así como las opciones que ofrece el concepto Bring Your Own Device (BYOD), los profesionales de TI se enfrentan con dos objetivos más o menos opuestos:

- Permitir que los usuarios finales sean productivos donde sea y cuando sea.
- Proteger los recursos de la organización.

Para proteger los recursos, los profesionales de IT deben primero administrar las identidades de los dispositivos. Pueden compilar la identidad del dispositivo con herramientas como Microsoft Intune para garantizar que se respetan las normas de seguridad y cumplimiento. Azure Active Directory (Azure AD) habilita el inicio de sesión único en dispositivos, aplicaciones y servicios desde cualquier ubicación mediante estos dispositivos.

- Los usuarios obtienen acceso a los recursos de la organización que necesitan. 
- Los profesionales de IT tienen el control de lo que necesitan proteger en la organización.

La administración de identidades de los dispositivos es la base del [acceso condicional basado en dispositivos](../conditional-access/require-managed-devices.md). Con las directivas de acceso condicional basado en dispositivos puede asegurarse de que el acceso a los recursos del entorno solo es posible con los dispositivos administrados.

## <a name="getting-devices-in-azure-ad"></a>Introducción a los dispositivos de Azure AD

Para obtener un dispositivo en Azure AD, tiene varias opciones:

- **Azure AD registered**
   - Los dispositivos que están registrados en Azure AD normalmente son dispositivos móviles o de propiedad personal y en ellos se inicia sesión con una cuenta personal de Microsoft u otra cuenta local.
      - Windows 10
      - iOS
      - Android
      - MacOS
- **Azure AD joined**
   - Los dispositivos que están unidos a Azure AD son propiedad de una organización y en ellos se inicia sesión con una cuenta de Azure AD que pertenece a dicha organización. Solo existen en la nube.
      - Windows 10 
      - [Máquinas virtuales Windows Server 2019 que se ejecutan en Azure](howto-vm-sign-in-azure-ad-windows.md) (No se admite Server Core)
- **Unido a Azure AD híbrido**
   - Los dispositivos que están unidos a Azure AD híbrido son propiedad de una organización y se inicia sesión con una cuenta de Active Directory Domain Services que pertenece a dicha organización. Existen en la nube y en el entorno local.
      - Windows 7, 8.1 o 10
      - Windows Server 2008 o versiones posteriores

![Dispositivos que se muestran en la hoja de dispositivos de Azure AD](./media/overview/azure-active-directory-devices-all-devices.png)

> [!NOTE]
> Un estado híbrido hace referencia a algo más que el estado de un dispositivo. Para que un estado híbrido sea válido, también se necesita un usuario de Azure AD válido.

## <a name="device-management"></a>Administración de dispositivos

Los dispositivos de Azure AD pueden administrarse mediante herramientas de administración de dispositivos móviles (MDM) como Microsoft Intune, Microsoft Endpoint Configuration Manager, directiva de grupo (unido a Azure AD híbrido), herramientas de administración de aplicaciones móviles (MAM) u otras herramientas de terceros.

## <a name="resource-access"></a>Acceso a los recursos

El registro y la unión de dispositivos a Azure AD proporciona a los usuarios un inicio de sesión único de conexión directa (SSO) a los recursos en la nube. Este proceso también proporciona a los administradores la capacidad de aplicar directivas de acceso condicional a los recursos en función del dispositivo desde el que se acceda a ellos. 

> [!NOTE]
> Las directivas de acceso condicional basadas en dispositivos requieren dispositivos híbridos unidos a Azure AD o unidos a Azure AD compatibles o dispositivos registrados de Azure AD.

El token de actualización principal (PRT) contiene información sobre el dispositivo y es necesario para el inicio de sesión único. Si tiene una directiva de acceso condicional basada en un dispositivo establecida en una aplicación, sin el PRT, se deniega el acceso. Las directivas de acceso condicional híbridas requieren un dispositivo de estado híbrido y un usuario válido que haya iniciado sesión.

Los dispositivos que están unidos a Azure AD o híbridos unidos a Azure AD se benefician del Inicio de sesión único de conexión directa para los recursos locales de la organización, al igual que para los recursos en la nube. Para más información, consulte [How SSO to on-premises resources works on Azure AD joined devices](azuread-join-sso.md) (Funcionamiento del inicio de sesión único en los recursos locales en dispositivos unidos a Azure AD).

## <a name="device-security"></a>Seguridad de los dispositivos

- Los **dispositivos registrados en Azure AD** utilizan una cuenta administrada por el usuario final; esta es una cuenta Microsoft o protegida localmente mediante la administración de credenciales, con uno o más de los siguientes elementos.
   - Contraseña
   - PIN
   - Patrón
   - Windows Hello
- Los **dispositivos unidos a Azure AD o híbridos unidos a Azure AD** utilizan una cuenta de organización en Azure AD protegida por uno o más de los siguientes elementos.
   - Contraseña
   - Windows Hello para empresas

## <a name="provisioning"></a>Aprovisionamiento

A Azure AD se pueden agregar dispositivos como autoservicio o mediante un proceso de aprovisionamiento controlado por parte de los administradores.

## <a name="summary"></a>Resumen

Con la administración de identidades de dispositivos en Azure AD, puede:

- Simplificar el proceso de llevar los dispositivos a Azure AD y administrarlos ahí
- Facilitar a los usuarios el acceso a los recursos de la organización basados en la nube

## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [Active Directory P1 license](../../../includes/active-directory-p1-license.md)]

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre los [dispositivos registrados en Azure AD](concept-azure-ad-register.md).
- Obtenga más información sobre los [dispositivos unidos a Azure AD](concept-azure-ad-join.md).
- Obtenga más información sobre los [dispositivos híbridos unidos a Azure AD](concept-azure-ad-join-hybrid.md).
- Para obtener información general sobre cómo administrar identidades de dispositivos en Azure Portal, consulte [Administración de identidades de dispositivos mediante Azure Portal](device-management-azure-portal.md).
- Para obtener más información sobre el acceso condicional basado en dispositivo, consulte [Configuración de directivas de acceso condicional basadas en dispositivo de Azure Active Directory](../conditional-access/require-managed-devices.md).
