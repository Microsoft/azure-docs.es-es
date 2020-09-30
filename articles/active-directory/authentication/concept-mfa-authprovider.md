---
title: 'Proveedores de Azure Multi-Factor Authentication: Azure Active Directory'
description: ¿Cuándo se debe usar un proveedor de autenticación con Azure MFA?
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: a7e4772ba2b3d8adf163d6f2932461e1e939ed84
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90052450"
---
# <a name="when-to-use-an-azure-multi-factor-authentication-provider"></a>Cuándo usar un proveedor de Azure Multi-Factor Authentication

> [!IMPORTANT]
> A partir del 1 de septiembre de 2018, no se pueden crear nuevos clientes de autenticación. Los proveedores de autenticación existentes se pueden seguir usando y actualizando, pero ya no se puede migrar. La autenticación multifactor seguirá estando disponible como una característica en las licencias de Azure AD Premium.

La verificación en dos pasos está disponible de forma predeterminada para los administradores globales que tienen usuarios de Azure Active Directory y Microsoft 365. Sin embargo, si desea aprovechar las [características avanzadas](howto-mfa-mfasettings.md), debe adquirir la versión completa de Azure Multi-Factor Authentication (MFA).

Un proveedor de Azure Multi-Factor Authentication se usa para aprovechar las características que Azure Multi-factor Authentication proporciona para los usuarios que **no tienen licencias**.

## <a name="caveats-related-to-the-azure-mfa-sdk"></a>Advertencias relacionadas con el SDK de Azure MFA

Tenga en cuenta que el SDK ha quedado en desuso y solo seguirá funcionando hasta el 14 de noviembre de 2018. Después de ese momento, se producirá un error en las llamadas al SDK.

## <a name="what-is-an-mfa-provider"></a>¿Qué es un proveedor de MFA?

Hay dos tipos de proveedores de autenticación y la diferencia radica en cómo se aplicarán cargos a su suscripción de Azure. La opción "por autenticación" calcula el número de autenticaciones que se realizan en el inquilino en un mes. Esta opción es la más adecuada si tiene un número de usuarios que se autentican solo en algunas ocasiones. La opción "por usuario" calcula el número de personas en el inquilino que realizan la verificación en dos pasos en un mes. Esta opción es mejor si tiene algunos usuarios con licencias pero necesita ampliar MFA a más usuarios de los permitidos por su licencia.

## <a name="manage-your-mfa-provider"></a>Administración del proveedor de MFA

No se puede cambiar el modelo de uso (por usuario habilitado o por autenticación) después de crear un proveedor de MFA.

Si compró licencias suficientes para cubrir todos los usuarios que están habilitados para MFA, puede eliminar por completo el proveedor de MFA.

Si el proveedor de MFA no está vinculado a un inquilino de Azure AD o si vincula el nuevo proveedor de MFA a un inquilino de Azure AD diferente, las opciones de configuración y parámetros de usuario no se transferirán. Además, los servidores Azure MFA existentes se tendrán que reactivar mediante las credenciales de activación generadas a través del proveedor de MFA.

### <a name="removing-an-authentication-provider"></a>Eliminación de un proveedor de autenticación

> [!CAUTION]
> No se produce ninguna confirmación al eliminar un proveedor de autenticación. La selección de **Eliminar** es un proceso permanente.

Los proveedores de autenticación se pueden encontrar en **Azure Portal** > **Azure Active Directory** > **Seguridad** > **MFA** > **Proveedores**. Haga clic en los proveedores de la lista para ver los detalles y las configuraciones asociadas a ese proveedor.

Antes de eliminar un proveedor de autenticación, tome nota de los valores personalizados configurados en el proveedor. Decida qué valores se deben migrar a la configuración general de MFA desde el proveedor y complete la migración de esos valores. 

Los servidores de Azure MFA vinculados a los proveedores deberán reactivarse con las credenciales generadas en **Azure Portal** > **Azure Active Directory** > **Seguridad** > **MFA** > **Configuración del servidor**. Antes de reactivar, se deben eliminar los siguientes archivos del directorio `\Program Files\Multi-Factor Authentication Server\Data\` de los servidores de Azure MFA de su entorno:

- caCert
- cert
- groupCACert
- groupKey
- groupName
- licenseKey
- pkey

![Eliminación de un proveedor de autenticación de Azure Portal](./media/concept-mfa-authprovider/authentication-provider-removal.png)

Cuando haya confirmado que se han migrado todos los valores, puede ir a **Azure Portal** > **Azure Active Directory** > **Seguridad** > **MFA** > **Proveedores**, seleccionar los puntos suspensivos **...** y seleccionar **Eliminar**.

> [!WARNING]
> Al eliminar un proveedor de autenticación, se eliminarán todos los datos de los informes asociados a ese proveedor. Es posible que desee guardar los informes de actividades antes de eliminar el proveedor.

> [!NOTE]
> Es posible que los usuarios con versiones anteriores de la aplicación Microsoft Authenticator y el servidor de Azure MFA tengan que volver a registrar la aplicación.

## <a name="next-steps"></a>Pasos siguientes

[Configuración de Azure Multi-Factor Authentication](howto-mfa-mfasettings.md)
