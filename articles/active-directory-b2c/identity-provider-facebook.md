---
title: Configuración del registro y del inicio de sesión con una cuenta de Facebook
titleSuffix: Azure AD B2C
description: Permita suscribirse e iniciar sesión en sus aplicaciones a los clientes con cuentas de Facebook mediante Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/26/2019
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 5879d745de85fe702267dce109e26ebcb07195ce
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116264"
---
# <a name="set-up-sign-up-and-sign-in-with-a-facebook-account-using-azure-active-directory-b2c"></a>Configuración de la suscripción y del inicio de sesión con una cuenta de Facebook mediante Azure Active Directory B2C

## <a name="create-a-facebook-application"></a>Creación de una aplicación de Facebook

Para usar una cuenta de Facebook como [proveedor de identidades](authorization-code-flow.md) en Azure Active Directory B2C (Azure AD B2C), tiene que crear una aplicación en su inquilino que la represente. Si aún no tiene una cuenta de Facebook, puede suscribirse en [https://www.facebook.com/](https://www.facebook.com/).

1. Inicie sesión en [Facebook for developers](https://developers.facebook.com/) con las credenciales de su cuenta de Facebook.
1. Si aún no lo ha hecho, debe registrarse como desarrollador de Facebook. Para ello, seleccione **Get Started** (Comenzar) en la esquina superior derecha de la página, acepte las directivas de Facebook y complete los pasos de registro.
1. Seleccione **Mis aplicaciones** y después **Crear una aplicación**.
1. Especifique el valor de **Display Name** (Nombre para mostrar) y un valor de **Contact Email** (Correo electrónico de contacto) válido.
1. Seleccione **Create App ID** (Crear identificador de aplicación). Es posible que deba aceptar las políticas de la plataforma Facebook y realizar una comprobación de seguridad en línea.
1. Seleccione **Settings** (Configuración)  > **Basic** (Básica).
1. Elija una **Categoría**, por ejemplo, `Business and Pages`. Este valor es obligatorio para Facebook, pero no se usa para Azure AD B2C.
1. En la parte inferior de la página, seleccione **Add Platform** (Agregar plataforma) y, después, seleccione **Website** (Sitio web).
1. En **URL del sitio**, escriba la dirección del sitio web, por ejemplo `https://contoso.com`. Escriba una dirección URL en **Privacy Policy URL** (URL de directiva de privacidad), por ejemplo `http://www.contoso.com/privacy`. La dirección URL de directiva es una página que sirve para proporcionar información de privacidad de la aplicación.
1. Seleccione **Save changes** (Guardar los cambios).
1. En la parte superior de la página, copie el valor de **App ID** (Id. de la aplicación).
1. Seleccione **Show** (Mostrar) y copie el valor de **App Secret** (Secreto de la aplicación). Use ambos para configurar Facebook como proveedor de identidades de su inquilino. El **secreto de la aplicación** es una credencial de seguridad importante.
1. Seleccione el signo más junto a **PRODUCTS** (Productos) y, después, seleccione **Set up** (Configurar) en **Facebook Login** (Inicio de sesión de Facebook).
1. En **Facebook Login** (Inicio de sesión de Facebook), seleccione **Settings** (Configuración).
1. En **Valid OAuth redirect URIs** (URI de redireccionamiento OAuth válidos), escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Reemplace `your-tenant-name` por el nombre del inquilino. Seleccione **Save Changes** (Guardar cambios) en la parte inferior de la página.
1. Para que la aplicación de Facebook esté disponible para Azure AD B2C, seleccione el selector de Estado situado en la parte superior derecha de la página y **actívelo** para hacer que la aplicación sea pública y, después, seleccione **Switch Mode** (Modo de conmutador).  En este momento el estado debería cambiar de **Desarrollo** a **Activo**.

## <a name="configure-a-facebook-account-as-an-identity-provider"></a>Configuración de una cuenta de Facebook como proveedor de identidades

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.
1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Proveedores de identidades** y luego **Facebook**.
1. Escriba un **nombre**. Por ejemplo, *Facebook*.
1. En **Id. de cliente**, escriba el identificador de aplicación de Facebook que ha creado anteriormente.
1. En **Secreto de cliente**, escriba el secreto de aplicación que ha anotado.
1. Seleccione **Guardar**.
