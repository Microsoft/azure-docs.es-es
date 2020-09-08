---
title: Autenticación y autorización
description: Obtenga información sobre las distintas formas en que puede autenticar una aplicación o servicio para Azure Spatial Anchors y los niveles de control para los que tiene que obtener acceso para Azure Spatial Anchors.
author: craigktreasure
manager: vriveras
services: azure-spatial-anchors
ms.author: crtreasu
ms.date: 05/28/2019
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.custom: devx-track-csharp
ms.openlocfilehash: e4d25637498bec223e294eecf2be6dc88fa2aa0d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997173"
---
# <a name="authentication-and-authorization-to-azure-spatial-anchors"></a>Autenticación y autorización en Azure Spatial Anchors

En esta sección, trataremos las diversas maneras en las que puede autenticarse en Azure Spatial Anchors desde la aplicación o servicio web y las formas en que puede usar el Control de acceso basado en roles de Azure Directory (Azure AD) para controlar el acceso a las cuentas de Spatial Anchors.

## <a name="overview"></a>Información general

![Introducción a la autenticación y autorización en Azure Spatial Anchors](./media/spatial-anchors-authentication-overview.png)

Para obtener acceso a una determinada cuenta de Azure Spatial Anchors, los clientes deben obtener primero un token de acceso del servicio de token de seguridad (STS) de Azure Mixed Reality. Los tokens obtenidos de STS se publican durante 24 horas e incluyen información de los servicios de Spatial Anchors para tomar decisiones de autorización acerca de la cuenta y asegurarse de que solo entidades de seguridad autorizadas pueden tener acceso a esta.

Los tokens de acceso pueden obtenerse a cambio de claves de cuenta o desde los tokens emitidos por Azure AD.

Las claves de cuenta le permiten comenzar rápidamente a usar el servicio de Azure Spatial Anchors; sin embargo, antes de implementar la aplicación en producción, se recomienda que la actualice para usar la autenticación basada en Azure AD.

Los tokens de autenticación de Azure AD pueden obtenerse de dos maneras:

- Si está creando una aplicación empresarial y la empresa usa Azure AD como sistema de identidad, puede usar la autenticación de Azure AD basada en el usuario en la aplicación y conceder acceso a las cuentas de Spatial Anchors mediante los grupos de seguridad existentes de Azure AD o directamente a los usuarios de la organización.
- En caso contrario, se recomienda que obtenga los tokens de Azure AD desde un servicio web compatible con la aplicación. El uso de un servicio web compatible es el método de autenticación recomendado para las aplicaciones de producción, ya que evita la incrustación de las credenciales para tener acceso a Azure Spatial Anchors en la aplicación cliente.

## <a name="account-keys"></a>Claves de cuenta

El uso de claves de cuenta para tener acceso a la cuenta de Azure Spatial Anchors es la manera más sencilla de comenzar. Encontrará las claves de cuenta en Azure Portal. Vaya a la cuenta y seleccione la pestaña "Claves".

![Introducción a la autenticación y autorización en Azure Spatial Anchors](../../../includes/media/spatial-anchors-get-started-create-resource/view-account-key.png)

Se ponen a disposición dos claves, ambas válidas simultáneamente para tener acceso a la cuenta de Spatial Anchors. Se recomienda que actualice periódicamente la clave que usa para tener acceso a la cuenta, que tenga dos claves válidas independientes como actualizaciones sin tiempo de inactividad y que solo deba actualizar la clave principal y la clave secundaria de manera alternativa.

El SDK tiene compatibilidad integrada para la autenticación con claves de cuenta; basta con establecer la propiedad AccountKey en el objeto cloudSession.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AccountKey = @"MyAccountKey";
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
_cloudSession.configuration.accountKey = @"MyAccountKey";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.accountKey = "MyAccountKey"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAccountKey("MyAccountKey");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AccountKey(R"(MyAccountKey)");
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AccountKey(LR"(MyAccountKey)");
```

---

Una vez hecho esto, el SDK controlará el intercambio de la clave de cuenta para un token de acceso, así como el almacenamiento en caché necesaria de tokens para la aplicación.

> [!WARNING]
> Se recomienda el uso de claves de cuenta para una incorporación rápida, pero solo durante el desarrollo y la creación de prototipos. Se recomienda encarecidamente no enviar la aplicación a producción mediante una clave de cuenta incrustada en ella; en su lugar, use los enfoques de autenticación de Azure AD basada en el usuario o en el servicio que se incluyen a continuación.

## <a name="azure-ad-user-authentication"></a>Autenticación de usuario de Azure AD

En el caso de las aplicaciones que tienen como destino a usuarios de Azure Active Directory, el enfoque recomendado consiste en usar un token de Azure AD para el usuario, que se puede obtener mediante [la biblioteca de MSAL](../../active-directory/develop/msal-overview.md). Debe seguir los pasos que se indican en la [guía de inicio rápido para registrar una aplicación](../../active-directory/develop/quickstart-register-app.md), que incluyen:

1. Configuración en Azure Portal
    1.  Registre la aplicación en Azure AD como **Aplicación nativa**. Como parte del registro, deberá determinar si la aplicación debe ser multiempresa, así como proporcionar las direcciones URL de redireccionamiento permitidas para la aplicación.
        1.  Cambie a la pestaña **Permisos de API**.
        2.  Seleccione **Agregar un permiso**.
            1.  Seleccione **Microsoft Mixed Reality** (Proveedor de recursos de realidad mixta) en la pestaña **API usadas en mi organización**.
            2.  Seleccione **Permisos delegados**.
            3.  Active la casilla **mixedreality.signin** en **mixedreality**
            4.  Seleccione **Agregar permisos**.
        3.  Seleccione **Conceder consentimiento de administrador**.
    2.  Conceda acceso a la aplicación o a los usuarios para el recurso:
        1.  Vaya al recurso de Spatial Anchors en Azure Portal.
        2.  Cambie a la pestaña **Control de acceso (IAM)** .
        3.  Presione **Agregar asignación de roles**.
            1.  [Seleccionar un rol](#role-based-access-control).
            2.  En el campo **Seleccionar**, escriba el nombre de los usuarios, grupos o aplicaciones a los que quiere asignar acceso.
            3.  Presione **Guardar**.
2. En el código:
    1.  Asegúrese de usar el **Id. de aplicación** y el **URI de redirección** de la propia aplicación de Azure AD como parámetros de **Id. de cliente** y **RedirectUri** de MSAL.
    2.  Establezca la información del inquilino:
        1.  Si la aplicación admite **Solo mi organización**, reemplace este valor por el **identificador de inquilino** o el **nombre de inquilino** (por ejemplo, contoso.microsoft.com).
        2.  Si la aplicación admite **Cuentas en cualquier directorio organizativo**, reemplace este valor por **Organizaciones**.
        3.  Si la aplicación admite **Todos los usuarios de cuentas Microsoft**, reemplace este valor por**Común**.
    3.  En la solicitud de token, establezca el **ámbito** en "https://sts.mixedreality.azure.com//.default". Este ámbito indicará a Azure AD que la aplicación solicita un token para el servicio de token de seguridad (STS) de Mixed Reality.

De este modo, la aplicación podrá obtener de MSAL un token de Azure AD; dicho token de Azure AD se puede establecer como **authenticationToken** en el objeto de configuración de la sesión en la nube.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AuthenticationToken = @"MyAuthenticationToken";
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
_cloudSession.configuration.authenticationToken = @"MyAuthenticationToken";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.authenticationToken = "MyAuthenticationToken"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAuthenticationToken("MyAuthenticationToken");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AuthenticationToken(R"(MyAuthenticationToken)");
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AuthenticationToken(LR"(MyAuthenticationToken)");
```

---

## <a name="azure-ad-service-authentication"></a>Autenticación de servicio de Azure AD

La opción recomendada para implementar aplicaciones mediante el aprovechamiento de Azure Spatial Anchors en producción consiste en aprovechar un servicio back-end que gestione las solicitudes de autenticación. El esquema general debe ser como se describe en este diagrama:

![Introducción a la autenticación y autorización en Azure Spatial Anchors](./media/spatial-anchors-aad-authentication.png)

En este caso, se supone que la aplicación usa su propio mecanismo (por ejemplo: cuenta de Microsoft, PlayFab, Facebook, Id. de Google, nombre de usuario/contraseña personalizado, etc.) para autenticarse en su servicio back-end. Una vez que los usuarios se autentican en el servicio back-end, dicho servicio puede recuperare un token de Azure AD, cambiarlo por un token de acceso para Azure Spatial Anchors y devolverlo a la aplicación cliente.

El token de acceso de Azure AD se recupera mediante la [biblioteca de MSAL](../../active-directory/develop/msal-overview.md). Debe seguir los pasos que se indican en la [guía de inicio rápido para registrar una aplicación](../../active-directory/develop/quickstart-register-app.md), que incluyen:

1.  Configuración en Azure Portal:
    1.  Registre la aplicación en Azure AD:
        1.  En Azure Portal, vaya a **Azure Active Directory** y seleccione **Registros de aplicaciones**.
        2.  Seleccione **Nuevo registro de aplicaciones**.
        3.  Escriba el nombre de la aplicación, seleccione **Aplicación web o API** como tipo de aplicación y escriba la dirección URL de autenticación para el servicio. A continuación, presione **Crear**.
        4.  En esa aplicación, presione **Configuración** y, a continuación, seleccione la pestaña **Certificates and secrets** (Certificados y servicios). Cree un nuevo secreto de cliente, seleccione una duración y presione **Agregar**. Asegúrese de guardar el valor de secreto, ya que lo necesitará para incluirlo en el código del servicio web.
    2.  Conceda acceso a la aplicación o a los usuarios para el recurso:
        1.  Vaya al recurso de Spatial Anchors en Azure Portal.
        2.  Cambie a la pestaña **Control de acceso (IAM)** .
        3.  Presione **Agregar asignación de roles**.
        1.  [Seleccionar un rol](#role-based-access-control).
        2.  En el campo **Seleccionar**, escriba el nombre de las aplicaciones que ha creado y a las que quiere asignar acceso. Si quiere que los usuarios de la aplicación tengan distintos roles en la cuenta de Spatial Anchors, debe registrar varias aplicaciones en Azure AD y asignar a cada una un rol independiente. A continuación, implemente la lógica de autorización para usar el rol correcto para los usuarios.
        3.  Nota: En la selección **Agregar asignación de roles**, **Asignar acceso a** se debe establecer en "Usuario, grupo o entidad de servicio de Azure AD".
    3.  Presione **Guardar**.
2.  En el código (nota: puede usar el servicio de ejemplo incluido en GitHub):
    1.  Asegúrese de usar el Id. de aplicación, el secreto de aplicación y el URI de redirección de la propia aplicación de Azure AD como parámetros de Id. de cliente, secreto y RedirectUri de MSAL.
    2.  Establezca el id. de inquilino en su propio id. de inquilino de Azure AD en el parámetro de autoridad de MSAL.
    3.  En la solicitud de token, establezca el **ámbito** en "https://sts.mixedreality.azure.com//.default".

Con esto, el servicio back-end puede recuperar un token de Azure AD. A continuación, puede cambiarlo por un token de MR que va a devolver al cliente. El uso de un token de Azure AD para recuperar un token de MR se realiza a través de una llamada de REST. A continuación, se muestra una llamada de ejemplo:

```
GET https://sts.mixedreality.azure.com/Accounts/35d830cb-f062-4062-9792-d6316039df56/token HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni<truncated>FL8Hq5aaOqZQnJr1koaQ
Host: sts.mixedreality.azure.com
Connection: Keep-Alive

HTTP/1.1 200 OK
Date: Sun, 24 Feb 2019 08:00:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 1153
Accept: application/json
MS-CV: 05JLqWeKFkWpbdY944yl7A.0
{"AccessToken":"eyJhbGciOiJSUzI1NiIsImtpZCI6IjI2MzYyMTk5ZTI2NjQxOGU4ZjE3MThlM2IyMThjZTIxIiwidHlwIjoiSldUIn0.eyJqdGkiOiJmMGFiNWIyMy0wMmUxLTQ1MTQtOWEzNC0xNzkzMTA1NTc4NzAiLCJjYWkiOiIzNWQ4MzBjYi1mMDYyLTQwNjItOTc5Mi1kNjMxNjAzOWRmNTYiLCJ0aWQiOiIwMDAwMDAwMC0wMDAwLTAwMDAtMDAwMC0wMDAwMDAwMDAwMDAiLCJhaWQiOiIzNWQ4MzBjYi1mMDYyLTQwNjItOTc5Mi1kNjMxNjAzOWRmNTYiLCJhYW8iOi0xLCJhcHIiOiJlYXN0dXMyIiwicmlkIjoiL3N1YnNjcmlwdGlvbnMvNzIzOTdlN2EtNzA4NC00ODJhLTg3MzktNjM5Y2RmNTMxNTI0L3Jlc291cmNlR3JvdXBzL3NhbXBsZV9yZXNvdXJjZV9ncm91cC9wcm92aWRlcnMvTWljcm9zb2Z0Lk1peGVkUmVhbGl0eS9TcGF0aWFsQW5jaG9yc0FjY291bnRzL2RlbW9fYWNjb3VudCIsIm5iZiI6MTU0NDU0NzkwMywiZXhwIjoxNTQ0NjM0MzAzLCJpYXQiOjE1NDQ1NDc5MDMsImlzcyI6Imh0dHBzOi8vbXJjLWF1dGgtcHJvZC50cmFmZmljbWFuYWdlci5uZXQvIiwiYXVkIjoiaHR0cHM6Ly9tcmMtYW5jaG9yLXByb2QudHJhZmZpY21hbmFnZXIubmV0LyJ9.BFdyCX9UJj0i4W3OudmNUiuaGgVrlPasNM-5VqXdNAExD8acFJnHdvSf6uLiVvPiQwY1atYyPbOnLYhEbIcxNX-YAfZ-xyxCKYb3g_dbxU2w8nX3zDz_X3XqLL8Uha-rkapKbnNgxq4GjM-EBMCill2Svluf9crDmO-SmJbxqIaWzLmlUufQMWg_r8JG7RLseK6ntUDRyDgkF4ex515l2RWqQx7cw874raKgUO4qlx0cpBAB8cRtGHC-3fA7rZPM7UQQpm-BC3suXqRgROTzrKqfn_g-qTW4jAKBIXYG7iDefV2rGMRgem06YH_bDnpkgUa1UgJRRTckkBuLkO2FvA"}
```

El encabezado de autorización tiene el formato siguiente: `Bearer <Azure_AD_token>`

Y la respuesta contiene el token de MR en texto sin formato.

Ese token de MR se devuelve después al cliente. La aplicación cliente, a continuación, puede establecerlo como su token de acceso en la configuración de la sesión en la nube.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AccessToken = @"MyAccessToken";
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
_cloudSession.configuration.accessToken = @"MyAccessToken";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.accessToken = "MyAccessToken"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAccessToken("MyAccessToken");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AccessToken(R"(MyAccessToken)");
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AccessToken(LR"(MyAccessToken)");
```

---

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Para ayudar a controlar el nivel de acceso concedido a las aplicaciones, los servicios o los usuarios de Azure AD de su servicio, se crearon los siguientes roles de asignación según fue necesario en las cuentas de Azure Spatial Anchors:

- **Propietario de cuenta de Spatial Anchors**: las aplicaciones o los usuarios que tienen este rol pueden crear anclajes espaciales, realizar consultas y eliminarlos. Al autenticarse en la cuenta mediante claves de cuenta, el rol **Propietario de cuenta de Spatial Anchors** se asigna a la entidad de seguridad autenticada.
- **Colaborador de cuenta de Spatial Anchors**: las aplicaciones o los usuarios que tienen este rol pueden crear anclajes espaciales y realizar consultas, pero no pueden eliminarlos.
- **Lector de cuenta de Spatial Anchors**: las aplicaciones o los usuarios que tienen este rol solo pueden realizar consultas para los anclajes espaciales, pero no pueden crear otros nuevos, eliminar los existentes ni actualizar los metadatos de dichos anclajes espaciales. Esto se utiliza normalmente para las aplicaciones en las que algunos usuarios ajustan el entorno, mientras que otros solo pueden recuperar los anclajes colocados anteriormente en ese entorno.

## <a name="next-steps"></a>Pasos siguientes

Cree su primera aplicación con Azure Spatial Anchors.

> [!div class="nextstepaction"]
> [Unity (HoloLens)](../quickstarts/get-started-unity-hololens.md)

> [!div class="nextstepaction"]
> [Unity (iOS)](../quickstarts/get-started-unity-ios.md)

> [!div class="nextstepaction"]
> [Unity (Android)](../quickstarts/get-started-unity-android.md)

> [!div class="nextstepaction"]
> [iOS](../quickstarts/get-started-ios.md)

> [!div class="nextstepaction"]
> [Android](../quickstarts/get-started-android.md)

> [!div class="nextstepaction"]
> [HoloLens](../quickstarts/get-started-hololens.md)

> [!div class="nextstepaction"]
> [Xamarin (Android)](../quickstarts/get-started-xamarin-android.md)

> [!div class="nextstepaction"]
> [Xamarin (iOS)](../quickstarts/get-started-xamarin-ios.md)
