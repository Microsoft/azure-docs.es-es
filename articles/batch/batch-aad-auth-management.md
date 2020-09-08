---
title: Uso de Azure Active Directory para autenticar soluciones de administración de Batch
description: Explore con Azure Active Directory para realizar la autenticación desde aplicaciones que utilizan la biblioteca Batch Management .NET.
ms.topic: how-to
ms.date: 04/27/2017
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: ff49d5e88df7c56ed4dee0685f09e45eb372aa5c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88930220"
---
# <a name="authenticate-batch-management-solutions-with-active-directory"></a>Autenticación de soluciones de administración de Batch con Active Directory

Las aplicaciones que llaman al servicio de administración de Azure Batch se autentican con [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD). Azure AD es el directorio basado en la nube multiinquilino y el servicio de administración de identidades de Microsoft. El propio Azure usa Azure Active Directory (AAD) para la autenticación de sus clientes, administradores de servicios y usuarios de la organización.

La biblioteca Batch Management .NET expone tipos para trabajar con cuentas, claves de cuenta, aplicaciones y paquetes de aplicaciones de Batch. La biblioteca Batch Management .NET es un cliente de proveedor de recursos de Azure y se utiliza junto con [Azure Resource Manager](../azure-resource-manager/management/overview.md) para administrar estos recursos mediante programación. Se necesita Azure AD para autenticar las solicitudes realizadas a través de cualquier cliente de proveedor de recursos de Azure, incluida la biblioteca de .NET Batch Management y a través de Azure Resource Manager.

En este artículo, se describe el uso de Azure AD para autenticar desde aplicaciones que utilizan la biblioteca Batch Management .NET. Se muestra cómo usar Azure AD para autenticar a un administrador o coadministrador de la suscripción mediante la autenticación integrada. Se utiliza el proyecto de ejemplo [AccountManagement](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement), disponible en GitHub, para guiarle en el uso de Azure AD con la biblioteca Batch Management .NET.

Para más información sobre el uso de la biblioteca Batch Management .NET y el ejemplo de AccountManagement, consulte [Administración de cuentas y cuotas de Batch con la biblioteca cliente Batch Management .NET](batch-management-dotnet.md).

## <a name="register-your-application-with-azure-ad"></a>Registro de la aplicación con Azure AD

La [biblioteca de autenticación de Azure Active Directory](../active-directory/azuread-dev/active-directory-authentication-libraries.md) (ADAL) proporciona una interfaz de programación para Azure AD que puede usar en las aplicaciones. Para llamar a ADAL desde la aplicación, debe registrar la aplicación en un inquilino de Azure AD. Al registrar la aplicación, se facilita a Azure AD información acerca de esta, incluido un nombre para ella en el inquilino de Azure AD. Azure AD proporciona un identificador de aplicación que se utiliza para asociar la aplicación con Azure AD en tiempo de ejecución. Para conocer más detalles acerca del identificador de la aplicación, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

Siga los pasos que aparecen en la sección [Incorporación de una aplicación](../active-directory/develop/quickstart-register-app.md) en [Integración de aplicaciones con Azure Active Directory](../active-directory/develop/quickstart-register-app.md) para registrar la aplicación de ejemplo AccountManagement. Especifique **Aplicación de cliente nativo** como tipo de aplicación. El identificador URI OAuth 2.0 estándar del sector para el **URI de redireccionamiento** es `urn:ietf:wg:oauth:2.0:oob`. Pero puede especificar cualquier identificador URI válido (como `http://myaccountmanagementsample`) para el **URI de redirección**, ya que no es necesario que sea un punto de conexión real.

![Agregar una aplicación](./media/batch-aad-auth-management/app-registration-management-plane.png)

Una vez completado el proceso de registro, verá que aparecen el identificador de la aplicación y el identificador de objeto (entidad de servicio) para la aplicación.

![Proceso de registro completado](./media/batch-aad-auth-management/app-registration-client-id.png)

## <a name="grant-the-azure-resource-manager-api-access-to-your-application"></a>Concesión a la API de Azure Resource Manager de acceso a la aplicación

A continuación, debe delegar el acceso a la aplicación a la API de Azure Resource Manager. El identificador de Azure AD para la API de Resource Manager es **Windows Azure Service Management API**.

Siga estos pasos en Azure Portal:

1. En el panel de navegación izquierdo de Azure Portal, elija **Todos los servicios**, haga clic en **Registros de aplicaciones**y, luego, en **Agregar**.
2. Busque el nombre de la aplicación en la lista de registros de aplicación:

    ![Buscar el nombre de la aplicación](./media/batch-aad-auth-management/search-app-registration.png)

3. Se mostrará la hoja **Configuración**. En la sección **Acceso de API**, seleccione **Permisos necesarios**.
4. Haga clic en **Agregar** para agregar un nuevo permiso necesario.
5. En el paso 1, escriba **Windows Azure Service Management API**, seleccione esa API en la lista de resultados y haga clic en el botón **Seleccionar**.
6. En el paso 2, active la casilla de verificación situada junto a **Access Azure classic deployment model as organization users** (Acceder al modelo de implementación clásica como usuarios de la organización) y haga clic en el botón **Seleccionar**.
7. Haga clic en el botón **Listo**.

La hoja **Permisos necesarios** ya muestra que se han concedido permisos a la aplicación para las API de ADAL y de Resource Manager. La primera vez que registra la aplicación con Azure AD, se conceden permisos a ADAL de forma predeterminada.

![Delegar permisos para la API de Azure Resource Manager](./media/batch-aad-auth-management/required-permissions-management-plane.png)

## <a name="azure-ad-endpoints"></a>Puntos de conexión de Azure AD

Para autenticar sus soluciones de administración de Batch con Azure AD, necesitará dos puntos de conexión conocidos.

- El **punto de conexión común de Azure AD** proporciona una interfaz genérica de recopilación de credenciales cuando no se proporciona un inquilino específico, como en el caso de la autenticación integrada:

    `https://login.microsoftonline.com/common`

- El **punto de conexión de Azure Resource Manager** se usa para adquirir un token para la autenticación de solicitudes en el servicio de administración de Batch:

    `https://management.core.windows.net/`

La aplicación de ejemplo de AccountManagement define las constantes para estos puntos de conexión. No cambie estas constantes:

```csharp
// Azure Active Directory "common" endpoint.
private const string AuthorityUri = "https://login.microsoftonline.com/common";
// Azure Resource Manager endpoint
private const string ResourceUri = "https://management.core.windows.net/";
```

## <a name="reference-your-application-id"></a>Referencia al identificador de aplicación

La aplicación cliente usa el identificador de aplicación (conocido también como identificador de cliente) para acceder a Azure AD en tiempo de ejecución. Una vez que haya registrado la aplicación en Azure Portal, actualice el código para usar el identificador de aplicación proporcionado por Azure AD para la aplicación registrada. En la aplicación de ejemplo AccountManagement, copie el identificador de la aplicación desde Azure Portal a la constante adecuada:

```csharp
// Specify the unique identifier (the "Client ID") for your application. This is required so that your
// native client application (i.e. this sample) can access the Microsoft Graph API. For information
// about registering an application in Azure Active Directory, please see "Register an application with the Microsoft identity platform" here:
// https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app
private const string ClientId = "<application-id>";
```
Copie también el URI de redirección especificado durante el proceso de registro. El identificador URI de redireccionamiento especificado en el código debe coincidir con el identificador URI de redireccionamiento que proporcionó al registrar la aplicación.

```csharp
// The URI to which Azure AD will redirect in response to an OAuth 2.0 request. This value is
// specified by you when you register an application with AAD (see ClientId comment). It does not
// need to be a real endpoint, but must be a valid URI (e.g. https://accountmgmtsampleapp).
private const string RedirectUri = "http://myaccountmanagementsample";
```

## <a name="acquire-an-azure-ad-authentication-token"></a>Adquisición de un token de autenticación de Azure AD

Después de registrar el ejemplo de AccountManagement en el inquilino de Azure AD y actualizar el código fuente de ejemplo con sus valores, el ejemplo estará listo para autenticarse con Azure AD. Cuando ejecute el ejemplo, ADAL intentará adquirir un token de autenticación. En este paso, le pedirá las credenciales de Microsoft:

```csharp
// Obtain an access token using the "common" AAD resource. This allows the application
// to query AAD for information that lies outside the application's tenant (such as for
// querying subscription information in your Azure account).
AuthenticationContext authContext = new AuthenticationContext(AuthorityUri);
AuthenticationResult authResult = authContext.AcquireToken(ResourceUri,
                                                        ClientId,
                                                        new Uri(RedirectUri),
                                                        PromptBehavior.Auto);
```

Después de proporcionar sus credenciales, la aplicación de ejemplo emitirá las solicitudes autenticadas para el servicio de administración de Batch.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre la ejecución de la [aplicación de ejemplo AccountManagement](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement), consulte [Administración de cuentas y cuotas de Batch con la biblioteca cliente de administración de Batch para .NET](batch-management-dotnet.md).
- Para más información acerca de Azure AD, consulte la [Documentación de Azure Active Directory](../active-directory/index.yml).
- Puede consultar los ejemplos detallados sobre el uso de ADAL que están disponibles en la biblioteca de [ejemplos de código de Azure](https://azure.microsoft.com/resources/samples/?service=active-directory).
- Para autenticar aplicaciones de servicio de Batch con Azure AD, vea [Autenticación de soluciones de servicio de Batch con Active Directory](batch-aad-auth.md).
