---
title: Configuración del inicio de sesión para una organización de Azure AD
titleSuffix: Azure AD B2C
description: Configuración del inicio de sesión para una determinada organización de Azure Active Directory en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/20/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: e4fa4b64c6519df90d5883e8c5760b3ed2ce0337
ms.sourcegitcommit: 3c66bfd9c36cd204c299ed43b67de0ec08a7b968
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "90004466"
---
# <a name="set-up-sign-in-for-a-specific-azure-active-directory-organization-in-azure-active-directory-b2c"></a>Configuración del inicio de sesión para una determinada organización de Azure Active Directory en Azure Active Directory B2C

Para usar una cuenta de Azure Active Directory (Azure AD) como [proveedor de identidades](authorization-code-flow.md) en Azure AD B2C, tiene que crear una aplicación que la represente. En este artículo se muestra cómo habilitar el inicio de sesión para los usuarios de una organización de Azure AD específica mediante un flujo de usuario en Azure AD B2C.

[!INCLUDE [active-directory-b2c-identity-provider-azure-ad](../../includes/active-directory-b2c-identity-provider-azure-ad.md)]

## <a name="configure-azure-ad-as-an-identity-provider"></a>Configuración de Azure AD como proveedor de identidades

1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripción** del menú superior y el directorio que contiene el inquilino de Azure AD B2C.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. Seleccione **Proveedores de identidades** y luego **Nuevo proveedor de OpenID Connect**.
1. Escriba un **nombre**. Por ejemplo, escriba *Contoso Azure AD*.
1. En **URL de metadatos**, escriba la dirección URL siguiente y sustituya `{tenant}` por el nombre de dominio del inquilino de Azure AD:

    ```
    https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration
    ```

    Por ejemplo, `https://login.microsoftonline.com/contoso.onmicrosoft.com/v2.0/.well-known/openid-configuration`.
    Por ejemplo, `https://login.microsoftonline.com/contoso.com/v2.0/.well-known/openid-configuration`.

1. En **Id. de cliente**, escriba el identificador de aplicación que ha anotado anteriormente.
1. En **Secreto de cliente**, escriba el secreto de cliente que ha anotado anteriormente.
1. En **Ámbito**, escriba el valor de `openid profile`.
1. Deje los valores predeterminados para **Tipo de respuesta**y **Modo de respuesta**.
1. (Opcional) En **Sugerencia de dominio**, escriba `contoso.com`. Para más información, consulte [Configuración de inicio de sesión directo con Azure Active Directory B2C](direct-signin.md#redirect-sign-in-to-a-social-provider).
1. En **Asignación de notificaciones del proveedor de identidades**, seleccione las siguientes notificaciones:

    * **Id. de usuario**: *oid*
    * **Nombre para mostrar**: *name*
    * **Nombre propio**: *given_name*
    * **Apellido**: *family_name*
    * **Correo electrónico**: *unique_name*

1. Seleccione **Guardar**.
