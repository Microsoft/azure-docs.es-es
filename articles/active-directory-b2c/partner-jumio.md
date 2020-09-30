---
title: Tutorial para configurar Azure Active Directory B2C con Jumio
titleSuffix: Azure AD B2C
description: En este tutorial, configurará Azure Active Directory B2C con Jumio para la verificación de identificadores automatizada, a fin de proteger los datos del cliente.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/20/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 36a402669f8af465a1cedbf1585e16c972b18015
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89293122"
---
# <a name="tutorial-for-configuring-jumio-with-azure-active-directory-b2c"></a>Tutorial para configurar Jumio con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar Azure Active Directory B2C (Azure AD B2C) con [Jumio](https://www.jumio.com/). Jumio es un servicio de verificación de identificadores que habilita la comprobación automatizada de identificadores en tiempo real para proteger los datos de los clientes.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](https://review.docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant) vinculado a la suscripción de Azure.

## <a name="scenario-description"></a>Descripción del escenario

La integración de Jumio incluye los siguientes componentes:

- Azure AD B2C: el servidor de autorización responsable de verificar las credenciales del usuario. También se conoce como proveedor de identidades.

- Jumio: el servicio que toma los detalles del id. proporcionados por el usuario y los verifica.

- API REST intermedia: esta API implementa la integración entre Azure AD B2C y el servicio Jumio.

- Azure Blob Storage: el servicio que proporciona archivos de interfaz de usuario personalizados a las directivas de Azure AD B2C.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Diagrama de la arquitectura de una integración de Azure AD B2C con Jumio.](./media/partner-jumio/jumio-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página para iniciar sesión o registrarse para crear una cuenta. Azure AD B2C recopila los atributos de usuario.
| 2. | Azure AD B2C llama a la API de nivel intermedio y pasa los atributos de usuario.
| 3. | La API de nivel intermedio recopila atributos de usuario y los transforma en un formato que la API de Jumio puede consumir. A continuación, envía los atributos a Jumio.
| 4. | Una vez que Jumio consume la información y la procesa, devuelve el resultado a la API de nivel intermedio.
| 5. | La API de nivel intermedio procesa la información y envía la información pertinente de nuevo a Azure AD B2C.
| 6. | Azure AD B2C recibe información de la API de nivel intermedio. Si aparece una respuesta de error, se muestra un mensaje de error al usuario. Si aparece una respuesta correcta, el usuario se autentica y se escribe en el directorio.

## <a name="sign-up-with-jumio"></a>Registro con Jumio

Para crear una cuenta de Jumio, póngase en contacto con [Jumio](https://www.jumio.com/contact/).

## <a name="configure-azure-ad-b2c-with-jumio"></a>Configuración de Azure AD B2C con Jumio

Después de crear una cuenta de Jumio, use la cuenta para configurar Azure AD B2C. En las secciones siguientes se describe el proceso en secuencia.

### <a name="deploy-the-api"></a>Implementación de la API

Implemente el [código de la API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/API/Jumio.Api) proporcionado en un servicio de Azure. El código se puede publicar desde Visual Studio. Para hacerlo, siga [estas instrucciones](https://docs.microsoft.com/visualstudio/deployment/quickstart-deploy-to-azure?view=vs-2019).

>[!NOTE]
>Necesitará la dirección URL del servicio implementado para configurar Azure AD con la configuración necesaria.

### <a name="deploy-the-client-certificate"></a>Implementación del certificado de cliente

1. La llamada a la API de Jumio está protegida por un certificado de cliente. Cree un certificado autofirmado con el código de ejemplo de PowerShell siguiente:

   ``` PowerShell
   $cert = New-SelfSignedCertificate -Type Custom -Subject "CN=Demo-SigningCertificate" -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3") -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -NotAfter (Get-Date).AddYears(2) -CertStoreLocation "Cert:\CurrentUser\My"
   $cert.Thumbprint
   $pwdText = "Your password"
   $pwd = ConvertTo-SecureString -String $pwdText -Force -AsPlainText
   Export-PfxCertificate -Cert $Cert -FilePath "{your-local-path}\Demo-SigningCertificate.pfx" -Password $pwd.

   ```

   A continuación, el certificado se exportará a la ubicación especificada para ``{your-local-path}``.

3. Para importar el certificado a Azure App Service, siga las instrucciones de [este artículo](https://docs.microsoft.com/azure/app-service/configure-ssl-certificate#upload-a-private-certificate).

### <a name="create-a-signingencryption-key"></a>Creación de una clave de firma o cifrado

Cree una cadena aleatoria con una longitud superior a 64 caracteres y que contenga solo letras o números.

Por ejemplo: ``C9CB44D98642A7062A0D39B94B6CDC1E54276F2E7CFFBF44288CEE73C08A8A65``

Use el siguiente script de PowerShell para crear la cadena:

```PowerShell
-join ((0x30..0x39) + ( 0x41..0x5A) + ( 0x61..0x7A) + ( 65..90 ) | Get-Random -Count 64  | % {[char]$_})

```

### <a name="configure-the-api"></a>Configuración de la API

Puede [configurar las opciones de la aplicación en Azure App Service](https://docs.microsoft.com/azure/app-service/configure-common#configure-app-settings). Con este método, los valores se pueden configurar de forma segura sin registrarlos en un repositorio. Debe especificar la configuración siguiente para la API de REST:

| Configuración de la aplicación | Source | Notas |
| :-------- | :------------| :-----------|
|JumioSettings:AuthUsername | Configuración de la cuenta de Jumio |     |
|JumioSettings:AuthPassword | Configuración de la cuenta de Jumio |     |
|AppSettings:SigningCertThumbprint|Huella digital del certificado autofirmado creado|  |
|AppSettings:IdTokenSigningKey| Clave de firma creada con PowerShell | |
| AppSettings:IdTokenEncryptionKey |Clave de cifrado creada con PowerShell
| AppSettings:IdTokenIssuer | Emisor que se va a usar para el token JWT (se prefiere un valor de GUID) |
| AppSettings:IdTokenAudience  | Audiencia que se va a usar para el token JWT (se prefiere un valor de GUID) |
|AppSettings:BaseRedirectUrl | URL base de la directiva de Azure AD B2C | https://{your-tenant-name}.b2clogin.com/{your-application-id}|
| WEBSITE_LOAD_CERTIFICATES| Huella digital del certificado autofirmado creado |

### <a name="deploy-the-ui"></a>Implementación de la interfaz de usuario

1. Configure un [contenedor de blobs en la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal#create-a-container).

2. Almacene los archivos de la interfaz de usuario de la [carpeta de la interfaz de usuario](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/UI) en el contenedor de blobs.

#### <a name="update-ui-files"></a>Actualización de los archivos de interfaz de usuario

1. En los archivos de la interfaz de usuario, vaya a la carpeta [ocean_blue](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/UI/ocean_blue).

2. Abra cada archivo HTML.

3. Busque `{your-ui-blob-container-url}` y reemplace el valor con la dirección URL del contenedor de blobs.

4. Busque `{your-intermediate-api-url}` y reemplace el valor por la dirección URL del servicio de aplicaciones de API intermedias.

>[!NOTE]
> Como procedimiento recomendado, se sugiere que agregue una notificación de consentimiento en la página de colección de atributos. Notifique a los usuarios que la información se enviará a servicios de terceros para la verificación de identidad.

### <a name="configure-the-azure-ad-b2c-policy"></a>Configuración de la directiva de Azure AD B2C

1. Vaya a la [directiva de Azure AD B2C](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/Policies) en la carpeta Directivas.

2. Siga [este artículo](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications#custom-policy-starter-pack) para descargar el [paquete de inicio de LocalAccounts](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts).

3. Configure la directiva para el inquilino de Azure AD B2C.

>[!NOTE]
>Actualice las directivas proporcionadas para que se relacionen con su inquilino específico.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C. En **Directivas**, seleccione **Identity Experience Framework**.

2. Seleccione el valor de **SignUpSignIn** que creó anteriormente.

3. Seleccione **Ejecutar el flujo del usuario** y, a continuación:

   a. Para **Aplicación**, seleccione la aplicación registrada (la muestra es JWT).

   b. Para **URL de respuesta**, seleccione la **URL de redireccionamiento**.

   c. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Se llamará al servicio Jumio durante el flujo, una vez creado el atributo de usuario. Si el flujo está incompleto, compruebe que el usuario no esté guardado en el directorio.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Introducción a las directivas personalizadas en Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)
