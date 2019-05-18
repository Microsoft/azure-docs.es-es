---
title: 'Empezar a trabajar con directivas personalizadas: Azure Active Directory B2C | Microsoft Docs'
description: Obtenga información sobre cómo empezar a trabajar con directivas personalizadas en Azure Active Directory B2C.
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/03/2019
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: 6a66d45028b9fbf9c421b10ffb7863fcb9be5bec
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779858"
---
# <a name="get-started-with-custom-policies-in-azure-active-directory-b2c"></a>Introducción a las directivas personalizadas en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Las [directivas personalizadas](active-directory-b2c-overview-custom.md) son archivos de configuración que definen el comportamiento del inquilino de Azure Active Directory (Azure AD) B2C. En este artículo se crea una directiva personalizada que admite el registro o el inicio de sesión de la cuenta local mediante una dirección de correo electrónico y una contraseña. También debe preparar el entorno para agregar proveedores de identidades.

## <a name="prerequisites"></a>Requisitos previos

- Si todavía no tiene uno, debe [crear un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.
- [Registrar la aplicación](tutorial-register-applications.md) en el inquilino que ha creado para que pueda comunicarse con Azure AD B2C.

## <a name="add-signing-and-encryption-keys"></a>Agregar claves de firma y cifrado

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.
2. Asegúrese de que usa el directorio que contiene al inquilino de Azure AD B2C. Haga clic en el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene el inquilino. 
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.
4. En la página de introducción, seleccione **Identity Experience Framework**.

### <a name="create-the-signing-key"></a>Crear la clave de firma

1. Seleccione **Claves de directiva** y luego **Agregar**.
2. En **Opciones**, elija `Generate`.
3. En **Nombre**, escriba `TokenSigningKeyContainer`. Es posible que se agregue automáticamente el prefijo `B2C_1A_`.
4. En **Tipo de clave**, seleccione **RSA**.
5. En **Uso de claves**, seleccione **Firma**.
6. Haga clic en **Create**(Crear).

### <a name="create-the-encryption-key"></a>Crear la clave de cifrado

1. Seleccione **Claves de directiva** y luego **Agregar**.
2. En **Opciones**, elija `Generate`.
3. En **Nombre**, escriba `TokenEncryptionKeyContainer`. Es posible que se agregue automáticamente el prefijo `B2C_1A`_.
4. En **Tipo de clave**, seleccione **RSA**.
5. En **Uso de la clave**, seleccione **Cifrado**.
6. Haga clic en **Create**(Crear).

### <a name="create-the-facebook-key"></a>Crear la clave de Facebook

Si ya tiene un [secreto de aplicación de Facebook](active-directory-b2c-setup-fb-app.md), agréguelo como clave de directiva al inquilino. De lo contrario, debe crear la clave con un valor de marcador de posición para que las directivas pasen la validación.

1. Seleccione **Claves de directiva** y luego **Agregar**.
2. En **Opciones**, elija `Manual`.
3. En **Nombre**, escriba `FacebookSecret`. Es posible que se agregue automáticamente el prefijo `B2C_1A_`.
4. En **Secreto**, escriba el secreto de Facebook de developers.facebook.com o `0` como marcador de posición. Este valor es el secreto, no el identificador de aplicación.
5. En **Uso de claves**, seleccione **Firma**.
6. Haga clic en **Create**(Crear).

## <a name="register-identity-experience-framework-applications"></a>Registro de las aplicaciones del marco de experiencia de identidad

Azure AD B2C requiere que registre dos aplicaciones que se usen para registrar usuarios e iniciar su sesión: IdentityExperienceFramework (una aplicación web) y ProxyIdentityExperienceFramework (una aplicación nativa) con permiso delegado de la aplicación IdentityExperienceFramework. Las cuentas locales solo existen en su inquilino. Los usuarios se registran con una combinación única de dirección de correo electrónico/contraseña para obtener acceso a las aplicaciones registradas en el inquilino.

### <a name="register-the-identityexperienceframework-application"></a>Registrar la aplicación IdentityExperienceFramework

1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
2. Seleccione **Nuevo registro de aplicaciones**.
3. En **Nombre**, escriba `IdentityExperienceFramework`.
4. En **Tipo de aplicación**, elija **Aplicación web o API**.
5. En **URL de inicio de sesión**, escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com`, donde `your-tenant-name` es el nombre de dominio del inquilino de Azure AD B2C.
6. Haga clic en **Create**(Crear). 
7. Una vez creada, copie el identificador de aplicación y guárdelo para usarlo más adelante.

### <a name="register-the-proxyidentityexperienceframework-application"></a>Registrar la aplicación ProxyIdentityExperienceFramework

1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro de aplicaciones**.
2. En **Nombre**, escriba `ProxyIdentityExperienceFramework`.
3. En **Tipo de aplicación**, elija **Nativa**.
4. En **URI de redirección**, escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com`, donde `yourtenant` es el inquilino de Azure AD B2C.
5. Haga clic en **Create**(Crear). Una vez creada, copie el identificador de aplicación y guárdelo para usarlo más adelante.
6. En la página Configuración, seleccione **Permisos necesarios** y luego **Agregar**.
7. Elija **seleccionar una API**, busque y seleccione **IdentityExperienceFramework**y, a continuación, haga clic en **seleccione**.
9. Active la casilla situada junto a **Access IdentityExperienceFramework** (Acceder a IdentityExperienceFramework), haga clic en **Seleccionar** y luego en **Listo**.
10. Seleccione **Conceder permisos** y haga clic en **Sí** para confirmar.

## <a name="download-starter-pack-and-modify-policies"></a>Descarga del paquete de inicio y modificación de directivas

Las directivas personalizadas son un conjunto de archivos XML que se deben cargar al inquilino de Azure AD B2C. Se proporcionan paquetes de inicio de archivos para poder ponerse en marcha con rapidez. Cada paquete de inicio de la lista siguiente contiene el menor número de perfiles técnicos y recorridos del usuario necesarios para lograr los escenarios descritos:

- LocalAccounts: habilita el uso solo de cuentas locales.
- SocialAccounts: habilita el uso solo de cuentas sociales (o federadas).
- SocialAndLocalAccounts: habilita el uso de cuentas locales y sociales.
- SocialAndLocalAccountsWithMFA: habilita opciones sociales, locales y de Multi-Factor Authentication.

Cada paquete de inicio contiene lo siguiente:

- Archivo base. Se requieren algunas modificaciones en el archivo base.
- Archivo de extensión.  Este archivo es donde se hace la mayoría de los cambios de configuración.
- Archivos del usuario de confianza. Archivos específicos de la tarea a los que llama la aplicación.

>[!NOTE]
>Si el editor XML admite la validación, valide los archivos con el archivo de esquema XML TrustFrameworkPolicy_0.3.0.0.xsd que se encuentra en el directorio raíz del paquete de inicio. La validación del esquema XML identifica los errores antes de realizar la carga.

1. [Descargue el archivo .zip](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/archive/master.zip) o ejecute:

    ```console
    git clone https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack
    ```

2. En la carpeta SocialAndLocalAccounts, sustituya `yourtenant` por el nombre del inquilino en todos los archivos. Por ejemplo, `contosoTenant.onmicrosoft.com`. Si necesita un editor XML, [pruebe Visual Studio Code](https://code.visualstudio.com/download), un editor multiplataforma ligero.

### <a name="add-application-ids-to-the-custom-policy"></a>Agregar identificadores de aplicación a la directiva personalizada

Agregue los identificadores de aplicación al archivo de extensiones *TrustFrameworkExtensions.xml*.

1. Abra el archivo *TrustFrameworkExtensions.xml* y busque el elemento `<TechnicalProfile Id="login-NonInteractive">`.
2. Reemplace ambas instancias de `IdentityExperienceFrameworkAppId` por el identificador de la aplicación del marco de experiencia de identidad que creó antes. Reemplace ambas instancias de `ProxyIdentityExperienceFrameworkAppId` por el identificador de la aplicación del marco de experiencia de identidad del proxy que ha creado anteriormente.
3. Guarde el archivo de extensiones.

## <a name="upload-the-policies"></a>Cargar las directivas

1. En la página Directivas personalizadas de Identity Experience Framework, seleccione **Cargar directiva**.
1. Cargue *TrustFrameworkBase.xml*, *TrustFrameworkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* y *PasswordReset.xml* en este orden. Cuando se carga un archivo, se antepone `B2C_1A_` al nombre del archivo de directiva.

## <a name="test-the-custom-policy"></a>Prueba de la directiva personalizada

1. En la página Directivas personalizadas, seleccione **B2C_1A_signup_signin**.
2. Para **seleccione aplicación** en la página información general de la directiva personalizada, seleccione la aplicación web denominada *webapp1* que registró anteriormente. Asegúrese de que el **dirección URL de respuesta** es `https://jwt.ms`.
3. Seleccione **Ejecutar ahora**.
4. Debe poder registrarse con una dirección de correo electrónico.
5. Inicie sesión con la misma cuenta para confirmar que tiene una configuración correcta.

## <a name="add-facebook-as-an-identity-provider"></a>Incorporación de Facebook como proveedor de identidades

1. Configure una [aplicación de Facebook](active-directory-b2c-setup-fb-app.md).
2. En el archivo *TrustFrameworkExtensions.xml*, sustituya el valor de `client_id` por el identificador de aplicación de Facebook:

   ```xml
   <TechnicalProfile Id="Facebook-OAUTH">
     <Metadata>
     <!--Replace the value of client_id in this technical profile with the Facebook app ID"-->
       <Item Key="client_id">00000000000000</Item>
   ```
3. Cargue el archivo *TrustFrameworkExtensions.xml* en el inquilino.
4. Realice la prueba mediante **Ejecutar ahora** o invoque la directiva directamente desde la aplicación registrada.

## <a name="next-steps"></a>Pasos siguientes

- Agregue Azure Active Directory como proveedor de identidades. El archivo base que se ha usado en esta guía de introducción ya contiene parte del contenido que necesita para agregar otros proveedores de identidades. Para obtener información sobre cómo configurar inicios de sesión, vea el artículo [Azure Active Directory B2C: inicio de sesión con cuentas de Azure AD](active-directory-b2c-setup-aad-custom.md).
