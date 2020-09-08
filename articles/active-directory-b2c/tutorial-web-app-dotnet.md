---
title: 'Tutorial: Habilitación de la autenticación en una aplicación web'
titleSuffix: Azure AD B2C
description: Tutorial sobre cómo usar Azure Active Directory B2C para proporcionar inicios de sesión de usuario en una aplicación web de ASP.NET.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/14/2019
ms.custom: devx-track-csharp, mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: b23bed8163ffed6a610eda7677099989e966a646
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88163822"
---
# <a name="tutorial-enable-authentication-in-a-web-application-using-azure-active-directory-b2c"></a>Tutorial: Habilitación de la autenticación en una aplicación web mediante Azure Active Directory B2C

En este tutorial se muestra cómo usar Azure Active Directory B2C (Azure AD B2C) para registrar e iniciar sesión con usuarios en una aplicación web de ASP.NET. Azure AD B2C permite que las aplicaciones puedan autenticarse en cuentas de las redes sociales, cuentas de empresa y cuentas de Azure Active Directory mediante protocolos estándar abiertos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Actualizar la aplicación en Azure AD B2C
> * Configurar el ejemplo para que use la aplicación
> * Registrarse mediante el flujo de usuario

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

* [Cree flujos de usuario](tutorial-create-user-flows.md) para habilitar las experiencias de usuario en la aplicación.
* Instalar [Visual Studio 2019](https://www.visualstudio.com/downloads/) con la carga de trabajo de **ASP.NET y desarrollo web**.

## <a name="update-the-application-registration"></a>Actualización del registro de la aplicación

En el tutorial que completó como parte de los requisitos previos, registró una aplicación web en Azure AD B2C. Para habilitar la comunicación con el ejemplo de este tutorial, debe agregar un URI de redirección y crear un secreto de cliente (clave) para la aplicación registrada.

### <a name="add-a-redirect-uri-reply-url"></a>Adición de un URI de redirección (URL de respuesta)

Para actualizar una aplicación en su inquilino de Azure AD B2C, puede usar la nueva experiencia unificada **Registros de aplicaciones**, o bien la experiencia anterior **Aplicaciones (heredado)** . [Más información acerca de la nueva experiencia](https://aka.ms/b2cappregtraining).

#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones**, la pestaña **Aplicaciones propias** y, por último, la aplicación *webapp1*.
1. En **Web**, seleccione el vínculo **Agregar URI**, escriba `https://localhost:44316` y, a continuación, seleccione **Guardar**.
1. Seleccione **Información general**.
1. Anote el **Id. de aplicación (cliente)** ; lo usará en un paso posterior al configurar la aplicación web.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. Seleccione **Aplicaciones (heredado)** y, a continuación, la aplicación *webapp1*.
1. En **URL de respuesta**, agregue `https://localhost:44316`.
1. Seleccione **Guardar**.
1. En la página de propiedades, anote el identificador de la aplicación; lo usará en un paso posterior al configurar la aplicación web.

* * *

### <a name="create-a-client-secret"></a>Creación de un secreto de cliente

A continuación, cree un secreto de cliente para la aplicación web registrada. El código de ejemplo de la aplicación web lo usa para demostrar su identidad al solicitar tokens.

[!INCLUDE [active-directory-b2c-client-secret](../../includes/active-directory-b2c-client-secret.md)]

## <a name="configure-the-sample"></a>Configuración del ejemplo

En este tutorial, va a configurar un ejemplo que puede descargar desde GitHub. El ejemplo usa ASP.NET para proporcionar una lista sencilla de tareas pendientes. El ejemplo utiliza [componentes de middleware de Microsoft OWIN](https://docs.microsoft.com/aspnet/aspnet/overview/owin-and-katana/). [Descargue un archivo zip](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi/archive/master.zip) o clone el ejemplo de GitHub. Asegúrese de extraer el archivo de ejemplo en una carpeta donde la longitud total de caracteres de la ruta de acceso sea inferior a 260.

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

Los dos proyectos siguientes están en la solución de ejemplo:

* **TaskWebApp**: para crear y editar una lista de tareas. En este ejemplo se utiliza el flujo de usuario de **registro o de inicio de sesión** para que los usuarios se registren e inicien sesión.
* **TaskService**: admite la funcionalidad para crear, leer, actualizar y eliminar la lista de tareas. Azure AD B2C protege la API y TaskWebApp la llama.

Puede cambiar el ejemplo para que use la aplicación que está registrada en su inquilino, la cual incluye el identificador de aplicación y la clave que registró anteriormente. También puede configurar los flujos de usuario que ha creado. El ejemplo define los valores de configuración como valores en el archivo *Web.config*.

Actualice la configuración en el archivo Web.config para que funcione con el flujo de usuario:

1. Abra la solución **B2C-WebAPI-DotNet** en Visual Studio.
1. En el proyecto **TaskWebApp**, abra el archivo **Web.config**.
    1. Actualice el valor de `ida:Tenant` y `ida:AadInstance` con el nombre del inquilino de Azure AD B2C que ha creado. Por ejemplo, reemplace `fabrikamb2c` por `contoso`.
    1. Reemplace el valor de `ida:TenantId` por el identificador de directorio, que puede encontrar en las propiedades de su inquilino de Azure B2C (en Azure Portal en **Azure Active Directory** > **Propiedades** > **Id. de directorio**).
    1. Reemplace el valor de `ida:ClientId` por el identificador de la aplicación que ha registrado.
    1. Reemplace el valor de `ida:ClientSecret` con la clave que ha registrado. Si el secreto de cliente contiene entidades XML predefinidas, por ejemplo, menor que (`<`), mayor que (`>`), signo et (`&`) o comillas dobles (`"`), debe evitarlas mediante la codificación XML del secreto de cliente antes de agregarlo al archivo Web.config.
    1. Reemplace el valor de `ida:SignUpSignInPolicyId` por `b2c_1_signupsignin1`.
    1. Reemplace el valor de `ida:EditProfilePolicyId` por `b2c_1_profileediting1`.
    1. Reemplace el valor de `ida:ResetPasswordPolicyId` por `b2c_1_passwordreset1`.

## <a name="run-the-sample"></a>Ejecución del ejemplo

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **TaskWebApp** y haga clic en **Establecer como proyecto de inicio**.
1. Presione **F5**. Inicia el explorador predeterminado en la dirección del sitio web local `https://localhost:44316/`.

### <a name="sign-up-using-an-email-address"></a>Registro con una dirección de correo electrónico

1. Seleccione **Registrarse o en Iniciar sesión** para registrarse como usuario de la aplicación. Se usa el flujo de usuario **b2c_1_signupsignin1**.
1. Azure AD B2C presenta una página de inicio de sesión con un vínculo de registro. Como no tiene aún una cuenta, seleccione **Registrarse ahora**. El flujo de trabajo del registro presenta una página para recopilar y verificar la identidad del usuario con una dirección de correo electrónico. El flujo de trabajo de registro también recopila la contraseña del usuario y los atributos requeridos definidos en el flujo de usuario.
1. Utilice una dirección de correo electrónico válida y valídela mediante el código de verificación. Establezca una contraseña. Especifique valores para los atributos solicitados.

    ![Página de registro que se muestra como parte del flujo de trabajo de inicio de sesión o registro](./media/tutorial-web-app-dotnet/sign-up-workflow.PNG)

1. Seleccione **Crear** para crear una cuenta local en el inquilino de Azure AD B2C.

Ahora el usuario de la aplicación puede utilizar su dirección de correo electrónico para iniciar sesión y usar la aplicación web.

Sin embargo, la característica **Lista de tareas pendientes**  no funcionará hasta que complete el siguiente tutorial de la serie, [Tutorial: Uso de Azure AD B2C para proteger una API web de ASP.NET](tutorial-web-api-dotnet.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Actualizar la aplicación en Azure AD B2C
> * Configurar el ejemplo para que use la aplicación
> * Registrarse mediante el flujo de usuario

Ahora, pase al siguiente tutorial para habilitar la característica **Lista de tareas pendientes** de la aplicación web. En ella, registrará una aplicación de API web en su propio inquilino de Azure AD B2C y, después, modificará el código de ejemplo para usar el inquilino para la autenticación de la API.

> [!div class="nextstepaction"]
> [Tutorial: Uso de Azure Active Directory B2C para proteger una API web de ASP.NET >](tutorial-web-api-dotnet.md)
