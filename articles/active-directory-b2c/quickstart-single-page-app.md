---
title: 'Inicio rápido: Configuración del inicio de sesión de una aplicación de una sola página (SPA)'
titleSuffix: Azure AD B2C
description: En este inicio rápido, se ejecuta una aplicación de página única de ejemplo que usa Azure Active Directory B2C para proporcionar el inicio de sesión de la cuenta.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: quickstart
ms.date: 04/04/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: db18c71b09cd9beb58799ad4fbcbc944061153fb
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921765"
---
# <a name="quickstart-set-up-sign-in-for-a-single-page-app-using-azure-active-directory-b2c"></a>Inicio rápido: Configuración del inicio de sesión en una aplicación de una sola página mediante Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) proporciona administración de identidades en la nube para mantener la protección de su aplicación, empresa y clientes. Azure AD B2C permite que las aplicaciones puedan autenticarse con cuentas de redes sociales y cuentas de empresa mediante protocolos estándar abiertos. En esta guía de inicio rápido, se utiliza una aplicación de página única para iniciar sesión mediante un proveedor de identidades de redes sociales y llamar a una API web protegida por Azure AD B2C.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

- [Visual Studio Code](https://code.visualstudio.com/)
- [Node.js](https://nodejs.org/en/download/)
- Cuenta de redes sociales de Facebook, Google o Microsoft
- Código de ejemplo de GitHub: [active-directory-b2c-javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp)

    Puede [descargar el archivo .zip](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp/archive/master.zip) o clonar el repositorio:

    ```console
    git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git
    ```

## <a name="run-the-application"></a>Ejecución de la aplicación

1. Inicie el servidor mediante la ejecución de los siguientes comandos desde el símbolo del sistema de Node.js:

    ```console
    cd active-directory-b2c-javascript-msal-singlepageapp
    npm install && npm update
    npm start
    ```

    El servidor que *server.js* inicia muestra el puerto en el que está escuchando:

    ```console
    Listening on port 6420...
    ```

1. Vaya a la dirección URL de la aplicación. Por ejemplo, `http://localhost:6420`.

    ![Aplicación de página única de ejemplo en el explorador](./media/quickstart-single-page-app/sample-app-spa.png)

## <a name="sign-in-using-your-account"></a>Inicio de sesión mediante su cuenta

1. Seleccione **Iniciar sesión** para iniciar el recorrido del usuario.
1. Azure AD B2C presenta una página de inicio de sesión para una empresa ficticia llamada Fabrikam para la aplicación web de ejemplo. Para registrarse con un proveedor de identidades de redes sociales, seleccione el botón del proveedor de identidades que desee usar.

    ![Página de inicio de sesión o registro que muestra los botones del proveedor de identidades](./media/quickstart-single-page-app/sign-in-or-sign-up-spa.png)

    Debe autenticarse (iniciar sesión) con las credenciales de su cuenta de redes sociales y autorizar a la aplicación para que lea la información de su cuenta de redes sociales. Al conceder acceso, la aplicación puede recuperar la información del perfil de la cuenta de redes sociales como el nombre y la ciudad.

1. Finalice el proceso de inicio de sesión para el proveedor de identidades.

## <a name="access-a-protected-api-resource"></a>Acceso a un recurso de API protegido

Seleccione **Call API** (Llamar a la API) para obtener el nombre para mostrar de la llamada de API web como un objeto JSON.

![Aplicación de ejemplo en el explorador con la respuesta de la API web](./media/quickstart-single-page-app/call-api-spa.png)

La aplicación de página única de ejemplo incluye un token de acceso en la solicitud al recurso de API web protegido.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede usar el inquilino de Azure AD B2C si tiene previsto leer otros tutoriales o guías de inicio rápido de Azure AD B2C. Cuando ya no sea necesario, puede [eliminar el inquilino de Azure AD B2C](faq.md#how-do-i-delete-my-azure-ad-b2c-tenant).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha utilizado una aplicación de una página única de ejemplo para lo siguiente:

- Iniciar sesión con un proveedor de identidades de redes sociales
- Crear una cuenta de usuario de Azure AD B2C (que se crea automáticamente al iniciar sesión)
- Llamar a una API web protegida por Azure AD B2C

Empiece a crear su propio inquilino de Azure AD B2C.

> [!div class="nextstepaction"]
> [Creación de un inquilino de Azure Active Directory B2C en Azure Portal](tutorial-create-tenant.md)
