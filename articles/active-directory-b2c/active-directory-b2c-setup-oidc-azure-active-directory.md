---
title: 'Configuración del inicio de sesión para una organización de Azure Active Directory: Azure Active Directory B2C | Microsoft Docs'
description: Configuración del inicio de sesión para una determinada organización de Azure Active Directory en Azure Active Directory B2C.
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/14/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: bae5759beb6a817c411ee52d7eb27dbff4cfe01c
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785248"
---
# <a name="set-up-sign-in-for-a-specific-azure-active-directory-organization-in-azure-active-directory-b2c"></a>Configuración del inicio de sesión para una determinada organización de Azure Active Directory en Azure Active Directory B2C

>[!NOTE]
> Esta característica está en versión preliminar pública. No use esta característica en entornos de producción.

Para usar una cuenta de Azure Active Directory (Azure AD) como [proveedor de identidades](active-directory-b2c-reference-oauth-code.md) en Azure AD B2C, tiene que crear una aplicación que la represente. En este artículo se muestra cómo habilitar el inicio de sesión para los usuarios de una organización de Azure AD específica mediante un flujo de usuario en Azure AD B2C.

## <a name="create-an-azure-ad-app"></a>Creación de una aplicación de Azure AD

Si desea habilitar el inicio de sesión para los usuarios de una organización específica de Azure AD, es preciso registrar una aplicación en el inquilino de Azure AD de la organización, que no es el mismo que el inquilino de Azure AD B2C.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD. Seleccione el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene el inquilino de Azure AD. Esto no es el mismo inquilino que el inquilino de Azure AD B2C.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
4. Seleccione **Nuevo registro**.
5. Escriba un nombre para la aplicación. Por ejemplo, `Azure AD B2C App`.
6. Acepte la selección de **cuentas en este directorio organizativa solo** para esta aplicación.
7. Para el **URI de redireccionamiento**, acepte el valor de **Web**y escriba la dirección URL siguiente en todas las letras en minúscula, donde `your-B2C-tenant-name` se reemplaza por el nombre del inquilino de Azure AD B2C. Por ejemplo, `https://fabrikam.b2clogin.com/fabrikam.onmicrosoft.com/oauth2/authresp`:

    ```
    https://your--B2C-tenant-name.b2clogin.com/your-B2C-tenant-name.onmicrosoft.com/oauth2/authresp
    ```

    Todas las direcciones URL ahora deberían estar utilizando [b2clogin.com](b2clogin.md).

8. Haga clic en **Registrar**. Copia el **Id. de aplicación (cliente)** para usarlo más adelante.
9. Seleccione **certificados y secretos** en el menú de la aplicación y, a continuación, seleccione **nuevo secreto de cliente**.
10. Escriba un nombre para el secreto de cliente. Por ejemplo, `Azure AD B2C App Secret`.
11. Seleccione el período de expiración. Para esta aplicación, acepte la selección de **en 1 año**.
12. Seleccione **agregar** y copie el valor del nuevo secreto de cliente que se muestra para su uso posterior.

## <a name="configure-azure-ad-as-an-identity-provider"></a>Configuración de Azure AD como proveedor de identidades

1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene el inquilino de Azure AD B2C.
2. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
3. Seleccione **Proveedores de identidades** y, después, **Agregar**.
4. Escriba un **nombre**. Por ejemplo, escriba: `Contoso Azure AD`.
5. Seleccione **Tipo de proveedor de identidades**, seleccione **Open ID Connect (versión preliminar)** y haga clic en **Aceptar**.
6. Seleccione **configurar este proveedor de identidades**
7. En **URL de metadatos**, escriba la dirección URL siguiente, sustituyendo `your-AD-tenant-domain` por el nombre de dominio del inquilino de Azure AD. Por ejemplo: `https://login.microsoftonline.com/contoso.onmicrosoft.com/.well-known/openid-configuration`:

    ```
    https://login.microsoftonline.com/your-AD-tenant-domain/.well-known/openid-configuration
    ```

8. Para **Id. de cliente**, escriba el identificador de aplicación que haya registrado previamente y para **secreto de cliente**, escriba el secreto de cliente que haya registrado previamente.
9. También puede escribir un valor en **Sugerencia de dominio**. Por ejemplo, `ContosoAD`. Este es el valor que se usa al hacer referencia a este proveedor de identidades mediante *domain_hint* en la solicitud. 
10. Haga clic en **OK**.
11. Seleccione **Asignar las notificaciones de este proveedor de identidades** y establezca las siguientes notificaciones:
    
    - En **Id. de usuario**, escriba `oid`.
    - En **Nombre para mostrar**, escriba `name`.
    - En **Nombre propio**, escriba `given_name`.
    - En **Apellido**, escriba `family_name`.
    - En **Correo electrónico**, escriba `unique_name`.

12. Haga clic en **Aceptar** y luego en **Crear** para guardar la configuración.
