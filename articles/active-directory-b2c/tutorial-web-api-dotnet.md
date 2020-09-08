---
title: 'Tutorial: Concesión de acceso a una API web de ASP.NET'
titleSuffix: Azure AD B2C
description: Tutorial sobre cómo usar Active Directory B2C para proteger una API web de ASP.NET y llamarla desde una aplicación web para ASP.NET.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/14/2019
ms.custom: devx-track-csharp, mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: bfa8943af16fe62015a4736f561875235e205fc1
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88163890"
---
# <a name="tutorial-grant-access-to-an-aspnet-web-api-using-azure-active-directory-b2c"></a>Tutorial: Concesión de acceso a una API web de ASP.NET mediante Azure Active Directory B2C

En este tutorial se muestra cómo llamar a un recurso de API web protegido de Azure Active Directory B2C (Azure AD B2C) desde una aplicación web para ASP.NET.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Incorporar una aplicación de API web
> * Configurar los ámbitos para una API web
> * Conceder permisos a la API web
> * Configurar el ejemplo para que use la aplicación

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Complete los pasos y requisitos previos en [Tutorial: Habilitación de la autenticación en una aplicación web mediante Azure Active Directory B2C](tutorial-web-app-dotnet.md).

## <a name="add-a-web-api-application"></a>Incorporar una aplicación de API web

Los recursos de API web tienen que registrarse en el inquilino antes de que puedan aceptar y responder a solicitudes de recursos protegidos por aplicaciones cliente que presenten un token de acceso.

Para registrar una aplicación en su inquilino de Azure AD B2C, puede usar la nueva experiencia unificada **Registros de aplicaciones**, o bien la experiencia anterior **Aplicaciones (heredado)** . [Más información acerca de la nueva experiencia](https://aka.ms/b2cappregtraining).

#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. Escriba un **Nombre** para la aplicación. Por ejemplo, *webapi1*.
1. En **URI de redirección**, seleccione **Web** y escriba el punto de conexión donde Azure AD B2C deberá devolver los tokens que solicite la aplicación. En este tutorial, el ejemplo se ejecuta localmente y escucha en el puerto `https://localhost:44332`.
1. Seleccione **Registrar**.
1. Anote el **Id. de aplicación (cliente)** para usarlo en un paso posterior.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. Seleccione **Aplicaciones (heredado)** y, a continuación, **Agregar**.
5. Escriba un nombre para la aplicación. Por ejemplo, *webapi1*.
6. En **Incluir aplicación web o API web**, seleccione **Sí**.
7. En **Dirección URL de respuesta**, escriba un punto de conexión donde Azure AD B2C devolverá los tokens que solicite la aplicación. En este tutorial, el ejemplo se ejecuta localmente y escucha en el puerto `https://localhost:44332`.
8. En **URI de id. de aplicación**, escriba el identificador usado para la API web. Se genera el identificador URI completo, incluido el dominio. Por ejemplo, `https://contosotenant.onmicrosoft.com/api`.
9. Haga clic en **Crear**.
10. En la página de propiedades, registre el identificador de la aplicación que usará cuando configure la aplicación web.

* * *

## <a name="configure-scopes"></a>Configuración de ámbitos

Los ámbitos proporcionan una manera de controlar el acceso a los recursos protegidos. La API web utiliza los ámbitos para implementar el control de acceso basado en el ámbito. Por ejemplo, los usuarios de la API web pueden tener ambos accesos de lectura y de escritura, o pueden tener solo acceso de lectura. En este tutorial, utilizará ámbitos para definir los permisos de lectura y escritura para la API web.

[!INCLUDE [active-directory-b2c-scopes](../../includes/active-directory-b2c-scopes.md)]

## <a name="grant-permissions"></a>Concesión de permisos

Para llamar a una API web protegida desde una aplicación, deberá conceder permisos de aplicación a la API. En el tutorial de requisitos previos, ha creado una aplicación web en Azure AD B2C denominada *webapp1*. Puede usar esta aplicación para llamar a la API web.

[!INCLUDE [active-directory-b2c-permissions-api](../../includes/active-directory-b2c-permissions-api.md)]

La aplicación está registrada para llamar a la API web protegida. Un usuario se autentica con Azure AD B2C para utilizar la aplicación. La aplicación obtiene una concesión de autorización de Azure AD B2C para acceder a la API web protegida.

## <a name="configure-the-sample"></a>Configuración del ejemplo

Ahora que se ha registrado la API web y tiene ámbitos definidos, debe configurar la API web para usar el inquilino de Azure AD B2C. En este tutorial, configurará una API web de ejemplo. La API web de ejemplo se incluye en el proyecto que descargó en el tutorial indicado en los requisitos previos.

Hay dos proyectos en la solución de ejemplo:

* **TaskWebApp**: para crear y editar una lista de tareas. El ejemplo utiliza el flujo de usuario de **registro o de inicio de sesión** para que los usuarios se registren o inicien sesión.
* **TaskService**: admite la funcionalidad para crear, leer, actualizar y eliminar la lista de tareas. Azure AD B2C protege la API y TaskWebApp la llama.

### <a name="configure-the-web-application"></a>Configuración de la aplicación web

1. Abra la solución **B2C-WebAPI-DotNet** en Visual Studio.
1. En el proyecto **TaskWebApp**, abra **Web.config**.
1. Para ejecutar la API localmente, utilice la configuración de localhost para **api:TaskServiceUrl**. Cambie el archivo Web.config de la manera siguiente:

    ```csharp
    <add key="api:TaskServiceUrl" value="https://localhost:44332/"/>
    ```

1. Configure el identificador URI de la API. Es el URI que utiliza la aplicación web para realizar la solicitud de API. Configure además los permisos solicitados.

    ```csharp
    <add key="api:ApiIdentifier" value="https://<Your tenant name>.onmicrosoft.com/api/" />
    <add key="api:ReadScope" value="demo.read" />
    <add key="api:WriteScope" value="demo.write" />
    ```

### <a name="configure-the-web-api"></a>Configuración de la API web

1. En el proyecto **TaskService**, abra **Web.config**.
1. Configure la API para usar el inquilino.

    ```csharp
    <add key="ida:AadInstance" value="https://<Your tenant name>.b2clogin.com/{0}/{1}/v2.0/.well-known/openid-configuration" />
    <add key="ida:Tenant" value="<Your tenant name>.onmicrosoft.com" />
    ```

1. Establezca el identificador de cliente en el identificador de la aplicación de la API web registrada, *webapi1*.

    ```csharp
    <add key="ida:ClientId" value="<application-ID>"/>
    ```

1. Actualice la configuración del flujo de usuario con el nombre de su flujo de usuario de registro y de inicio de sesión, *B2C_1_signupsignin1*.

    ```csharp
    <add key="ida:SignUpSignInPolicyId" value="B2C_1_signupsignin1" />
    ```

1. Configure los ámbitos para que coincidan con lo que ha creado en el portal.

    ```csharp
    <add key="api:ReadScope" value="demo.read" />
    <add key="api:WriteScope" value="demo.write" />
    ```

## <a name="run-the-sample"></a>Ejecución del ejemplo

Debe ejecutar tanto el proyecto **TaskWebApp** como el de **TaskService**.

1. En el Explorador de soluciones, haga clic con el botón derecho en la solución y seleccione **Establecer proyectos de inicio**.
1. Seleccione **Proyectos de inicio múltiples**.
1. Cambie la **Acción** de ambos proyectos a **Iniciar**.
1. Haga clic en **Aceptar** para guardar la configuración.
1. Presione **F5** para ejecutar las dos aplicaciones. Cada aplicación se abre en su propia ventana del explorador.
    * `https://localhost:44316/` es la aplicación web.
    * `https://localhost:44332/` es la API web.

1. En la aplicación web, haga clic en **sign-up / sign-in** (registrar o iniciar sesión) para iniciar sesión en la aplicación web. Use la cuenta que creó anteriormente.
1. Después de iniciar sesión, seleccione **Lista de tareas pendientes** y cree un elemento de dicha lista.

Al crear un elemento de lista de tareas pendientes, la aplicación web realiza una solicitud a la API web para generar este elemento. Su aplicación web protegida llama a la API web protegida por Azure AD B2C.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Incorporar una aplicación de API web
> * Configurar los ámbitos para una API web
> * Conceder permisos a la API web
> * Configurar el ejemplo para que use la aplicación

> [!div class="nextstepaction"]
> [Tutorial: Adición de proveedores de identidades a las aplicaciones en Azure Active Directory B2C](tutorial-add-identity-providers.md)
