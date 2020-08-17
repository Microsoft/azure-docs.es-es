---
title: Introducción a las directivas personalizadas
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo empezar a usar directivas personalizadas en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/28/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: cb3bb363b1d6be72fe8941cad509b4c3525cfa15
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87169436"
---
# <a name="get-started-with-custom-policies-in-azure-active-directory-b2c"></a>Introducción a las directivas personalizadas en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Las [directivas personalizadas](custom-policy-overview.md) son archivos de configuración que definen el comportamiento del inquilino de Azure Active Directory B2C (Azure AD B2C). En este artículo se crea una directiva personalizada que admite el registro o el inicio de sesión de la cuenta local mediante una dirección de correo electrónico y una contraseña. También debe preparar el entorno para agregar proveedores de identidades.

## <a name="prerequisites"></a>Requisitos previos

- Si todavía no tiene uno, [cree un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.
- [Registre la aplicación](tutorial-register-applications.md) en el inquilino que ha creado para que pueda comunicarse con Azure AD B2C.
- Complete los pasos descritos en [Configuración de la suscripción y del inicio de sesión con una cuenta de Facebook](identity-provider-facebook.md) para configurar una aplicación de Facebook. Aunque no se requiere una aplicación de Facebook para el uso de directivas personalizadas, se usa en este tutorial para demostrar la habilitación del inicio de sesión social en una directiva personalizada.

## <a name="add-signing-and-encryption-keys"></a>Agregar claves de firma y cifrado

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En la página de información general, en **Directivas**, seleccione **Identity Experience Framework**.

### <a name="create-the-signing-key"></a>Crear la clave de firma

1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Generate`.
1. En **Nombre**, escriba `TokenSigningKeyContainer`. Es posible que se agregue automáticamente el prefijo `B2C_1A_`.
1. En **Tipo de clave**, seleccione **RSA**.
1. En **Uso de claves**, seleccione **Firma**.
1. Seleccione **Crear**.

### <a name="create-the-encryption-key"></a>Crear la clave de cifrado

1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Generate`.
1. En **Nombre**, escriba `TokenEncryptionKeyContainer`. Es posible que se agregue automáticamente el prefijo `B2C_1A`_.
1. En **Tipo de clave**, seleccione **RSA**.
1. En **Uso de la clave**, seleccione **Cifrado**.
1. Seleccione **Crear**.

### <a name="create-the-facebook-key"></a>Crear la clave de Facebook

Agregue el [secreto de la aplicación](identity-provider-facebook.md) de la aplicación de Facebook como una clave de directiva. Puede usar el secreto de aplicación de la aplicación que creó como parte de los requisitos previos de este artículo.

1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Manual`.
1. En **Nombre**, escriba `FacebookSecret`. Es posible que se agregue automáticamente el prefijo `B2C_1A_`.
1. En **Secreto**, escriba el *secreto de aplicación* de la aplicación de Facebook de developers.facebook.com. Este valor es el secreto, no el identificador de la aplicación.
1. En **Uso de claves**, seleccione **Firma**.
1. Seleccione **Crear**.

## <a name="register-identity-experience-framework-applications"></a>Registro de las aplicaciones del marco de experiencia de identidad

Azure AD B2C requiere que registre dos aplicaciones que se usen para registrar usuarios e iniciar su sesión con cuentas locales: *IdentityExperienceFramework* (una API web) y *ProxyIdentityExperienceFramework* (una aplicación nativa) con permiso delegado de la aplicación IdentityExperienceFramework. Los usuarios pueden registrarse con una dirección de correo electrónico o un nombre de usuario y una contraseña para acceder a las aplicaciones registradas en el inquilino, lo que crea una "cuenta local". Las cuentas locales solo existen en el inquilino de Azure AD B2C.

Solo tiene que registrar estas dos aplicaciones en el inquilino de Azure AD B2C una vez.

### <a name="register-the-identityexperienceframework-application"></a>Registrar la aplicación IdentityExperienceFramework

Para registrar una aplicación en el inquilino de Azure AD B2C, puede usar la experiencia **Registros de aplicaciones**.

1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. En **Nombre**, escriba `IdentityExperienceFramework`.
1. En **Tipos de cuenta admitidos**, seleccione **Solo las cuentas de este directorio organizativo**.
1. En **URI de redirección**, seleccione **Web** y, a continuación, escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com`, donde `your-tenant-name` es el nombre de dominio del inquilino de Azure AD B2C.
1. En **Permisos**, active la casilla *Conceda consentimiento del administrador a los permisos openid y offline_access*.
1. Seleccione **Registrar**.
1. Anote el **Id. de aplicación (cliente)** para usarlo en un paso posterior.

A continuación, exponga la API agregando un ámbito:

1. En el menú de la izquierda, en **Administrar**, seleccione **Exponer una API**.
1. Seleccione **agregar un ámbito** y, a continuación, **Guardar y continuar** para aceptar el URI de identificador de aplicación predeterminado.
1. Escriba los valores siguientes para crear un ámbito que permita la ejecución de la directiva personalizada en el inquilino de Azure AD B2C:
    * **Nombre de ámbito**: `user_impersonation`
    * **Nombre para mostrar del consentimiento del administrador**: `Access IdentityExperienceFramework`
    * **Descripción del consentimiento del administrador**: `Allow the application to access IdentityExperienceFramework on behalf of the signed-in user.`
1. Seleccione **Agregar ámbito**.

* * *

### <a name="register-the-proxyidentityexperienceframework-application"></a>Registrar la aplicación ProxyIdentityExperienceFramework

1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. En **Nombre**, escriba `ProxyIdentityExperienceFramework`.
1. En **Tipos de cuenta admitidos**, seleccione **Solo las cuentas de este directorio organizativo**.
1. En **URI de redirección**, use la lista desplegable para seleccionar **Cliente público o nativo (móvil y escritorio)** .
1. En **URI de redirección**, escriba `myapp://auth`.
1. En **Permisos**, active la casilla *Conceda consentimiento del administrador a los permisos openid y offline_access*.
1. Seleccione **Registrar**.
1. Anote el **Id. de aplicación (cliente)** para usarlo en un paso posterior.

A continuación, especifique que la aplicación se debe tratar como un cliente público:

1. En el menú izquierdo, en **Administrar**, seleccione **Autenticación**.
1. En **Configuración avanzada**, habilite **Trate la aplicación como cliente público** (seleccione **Sí**). Asegúrese de que **"allowPublicClient": true** esté establecido en el manifiesto de aplicación. 
1. Seleccione **Guardar**.

Ahora, conceda permisos al ámbito de la API que expuso anteriormente en el registro de *IdentityExperienceFramework*:

1. En el menú de la izquierda, en **Administrar**, seleccione **Permisos de API**.
1. En **Permisos configurados**, seleccione **Agregar un permiso**.
1. Seleccione la pestaña **Mis API** y, después, seleccione la aplicación **IdentityExperienceFramework**.
1. En **Permiso**, seleccione el ámbito **user_impersonation** que definió anteriormente.
1. Seleccione **Agregar permisos**. Como se indicó, espere unos minutos antes de continuar con el paso siguiente.
1. Seleccione **Conceder consentimiento de administrador para (el nombre de inquilino)** .
1. Seleccione la cuenta de administrador que tiene actualmente la sesión iniciada o inicie sesión con una cuenta en el inquilino de Azure AD B2C que tenga asignado al menos el rol *Administrador de aplicaciones en la nube*.
1. Seleccione **Aceptar**.
1. Seleccione **Actualizar** y luego compruebe que "Concedido para..." aparece en **Estado** para los ámbitos offline_access, openid y user_impersonation. Los permisos pueden tardar unos minutos en propagarse.

* * *

## <a name="custom-policy-starter-pack"></a>Paquete de inicio de directivas personalizadas

Las directivas personalizadas son un conjunto de archivos XML que se cargan en el inquilino de Azure AD B2C para definir perfiles técnicos y recorridos de usuario. Proporcionamos paquetes de inicio con varias directivas predefinidas para que pueda empezar a trabajar rápidamente. Cada uno de estos paquetes de inicio contiene el menor número de perfiles técnicos y recorridos del usuario necesarios para lograr los escenarios descritos:

- **LocalAccounts**: habilita el uso solo de cuentas locales.
- **SocialAccounts**: habilita el uso solo de cuentas sociales (o federadas).
- **SocialAndLocalAccounts**: habilita el uso de cuentas locales y sociales.
- **SocialAndLocalAccountsWithMFA**: habilita opciones sociales, locales y de autenticación multifactor.

Cada paquete de inicio contiene lo siguiente:

- **Archivo base**: se requieren algunas modificaciones en el archivo base. Ejemplo: *TrustFrameworkBase.xml*
- **Archivo de extensión**: este archivo es donde se hace la mayoría de los cambios de configuración. Ejemplo: *TrustFrameworkExtensions.xml*
- **Archivos de usuario de confianza**: archivos específicos de la tarea a los que llama la aplicación. Ejemplos: *SignUpOrSignin.xml*, *ProfileEdit.xml*, *PasswordReset.xml*.

En este artículo, editará los archivos de directivas personalizadas XML del paquete de inicio **SocialAndLocalAccounts**. Si necesita un editor XML, [pruebe Visual Studio Code](https://code.visualstudio.com/download), un editor multiplataforma ligero.

### <a name="get-the-starter-pack"></a>Obtención del paquete de inicio

Obtenga los paquetes de inicio de directivas personalizadas en GitHub y, luego, actualice los archivos XML del paquete de inicio SocialAndLocalAccounts con el nombre del inquilino de Azure AD B2C.

1. [Descargue el archivo .zip](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/archive/master.zip) o clone el repositorio:

    ```console
    git clone https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack
    ```

1. En todos los archivos del directorio **SocialAndLocalAccounts**, reemplace la cadena `yourtenant` por el nombre de su inquilino de Azure AD B2C.

    Por ejemplo, si el nombre del inquilino de B2C es *contosotenant*, todas las instancias de `yourtenant.onmicrosoft.com` se convierten en `contosotenant.onmicrosoft.com`.

### <a name="add-application-ids-to-the-custom-policy"></a>Agregar identificadores de aplicación a la directiva personalizada

Agregue los identificadores de aplicación al archivo de extensiones *TrustFrameworkExtensions.xml*.

1. Abra `SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`** y busque el elemento `<TechnicalProfile Id="login-NonInteractive">`.
1. Reemplace ambas instancias de `IdentityExperienceFrameworkAppId` por el identificador de la aplicación IdentityExperienceFramework que creó antes.
1. Reemplace ambas instancias de `ProxyIdentityExperienceFrameworkAppId` por el identificador de la aplicación ProxyIdentityExperienceFramework que creó antes.
1. Guarde el archivo.

## <a name="upload-the-policies"></a>Cargar las directivas

1. Seleccione el elemento de menú **Identity Experience Framework** en el inquilino de B2C en Azure Portal.
1. Seleccione **Cargar directiva personalizada**.
1. En este orden, cargue los archivos de directiva:
    1. *TrustFrameworkBase.xml*
    1. *TrustFrameworkExtensions.xml*
    1. *SignUpOrSignin.xml*
    1. *ProfileEdit.xml*
    1. *PasswordReset.xml*

A medida que cargue los archivos, Azure agregará el prefijo `B2C_1A_` a cada uno.

> [!TIP]
> Si el editor XML admite la validación, valide los archivos con el archivo de esquema XML `TrustFrameworkPolicy_0.3.0.0.xsd` que se encuentra en el directorio raíz del paquete de inicio. La validación del esquema XML identifica los errores antes de realizar la carga.

## <a name="test-the-custom-policy"></a>Prueba de la directiva personalizada

1. En **Directivas personalizadas**, seleccione **B2C_1A_signup_signin**.
1. Para **Seleccionar aplicación** en la página Información general de la directiva personalizada, seleccione la aplicación web denominada *webapp1* que ha registrado antes.
1. Asegúrese de que la **URL de respuesta** sea `https://jwt.ms`.
1. Seleccione **Ejecutar ahora**.
1. Regístrese con una dirección de correo electrónico.
1. Vuelva a seleccionar **Ejecutar ahora**.
1. Inicie sesión con la misma cuenta para confirmar que tiene una configuración correcta.

## <a name="add-facebook-as-an-identity-provider"></a>Incorporación de Facebook como proveedor de identidades

Como se mencionó en [Requisitos previos](#prerequisites), *no* se requiere Facebook para el uso de directivas personalizadas, pero aquí se usa para mostrar cómo se puede habilitar el inicio de sesión social federado en una directiva personalizada.

1. En el archivo `SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`** , sustituya el valor de `client_id` por el identificador de aplicación de Facebook:

   ```xml
   <TechnicalProfile Id="Facebook-OAUTH">
     <Metadata>
     <!--Replace the value of client_id in this technical profile with the Facebook app ID"-->
       <Item Key="client_id">00000000000000</Item>
   ```

1. Cargue el archivo *TrustFrameworkExtensions.xml* en el inquilino.
1. En **Directivas personalizadas**, seleccione **B2C_1A_signup_signin**.
1. Seleccione **Ejecutar ahora** y elija Facebook para iniciar sesión con Facebook y probar la directiva personalizada.

## <a name="next-steps"></a>Pasos siguientes

Ahora, pruebe a agregar Azure Active Directory (Azure AD) como proveedor de identidades. El archivo base que se ha usado en esta guía de introducción ya contiene parte del contenido que necesita para agregar otros proveedores de identidades como Azure AD.

Para obtener información sobre cómo configurar Azure AD como proveedor de identidades, vea [Configurar el inicio de sesión con una cuenta de Azure Active Directory mediante directivas personalizadas en Azure Active Directory B2C](identity-provider-azure-ad-single-tenant-custom.md).
