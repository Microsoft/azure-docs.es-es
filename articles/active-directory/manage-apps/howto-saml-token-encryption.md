---
title: Cifrado de tokens SAML en Azure Active Directory
description: Obtenga información para configurar el cifrado de tokens SAML de Azure Active Directory.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/06/2019
ms.author: mimart
ms.reviewer: paulgarn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 75f8b785b8eadd21f1f94cf82fe137d6f747e738
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824765"
---
# <a name="how-to-configure-azure-ad-saml-token-encryption-preview"></a>Procedimientos para: Configuración del cifrado de tokens SAML de Azure AD (versión preliminar)

> [!NOTE]
> El cifrado de tokens es una característica Premium de Azure Active Directory (Azure AD). Para más información sobre las ediciones, las características y los precios de Azure AD, consulte [Precios de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

El cifrado de tokens SAML habilita el uso de aserciones SAML cifradas con una aplicación que lo admite. Cuando se configure para una aplicación, Azure AD cifrará las aserciones SAML que emite para esa aplicación mediante la clave pública obtenida desde un certificado almacenado en Azure AD. La aplicación debe usar la clave privada coincidente para cifrar el token antes de que se pueda usar como prueba de la autenticación del usuario conectado.

Cifrar las aserciones SAML entre Azure AD y la aplicación proporciona una garantía adicional de que el contenido del token no se puede interceptar y que los datos personales o corporativos no quedan expuestos.

Incluso sin el cifrado de tokens, los tokens SAML de Azure AD nunca se transmiten por la red sin cifrar. Azure AD requiere que los intercambios de solicitud/respuesta de token se realicen a través de canales HTTPS/TLS cifrados para que las comunicaciones entre el IDP, el explorador y la aplicación se realicen en vínculos cifrados. Considere el valor del cifrado de tokens para su situación en comparación con la sobrecarga de administrar certificados adicionales.   

Para configurar el cifrado de tokens, debe cargar un archivo de certificado X.509 que contenga la clave pública en el objeto de aplicación de Azure AD que representa la aplicación. Para obtener el certificado X.509, puede descargarlo desde la misma aplicación u obtenerlo del proveedor de la aplicación en los casos donde este proporciona las claves de cifrado, o bien, en casos donde la aplicación espera que el usuario proporcione una clave cifrada, se puede crear usando herramientas de criptografía, la parte de la clave privada que se cargó en el almacén de claves de la aplicación y el certificado de clave pública coincidente que se cargó en Azure AD.

Azure AD usa AES-256 para cifrar los datos de aserción SAML.

## <a name="configure-saml-token-encryption"></a>Configuración del cifrado de tokens SAML

Para configurar el cifrado de tokens SAML, siga estos pasos:

1. Obtenga un certificado de clave pública que coincida con una clave privada configurada en la aplicación.

    Cree un par de claves asimétrico para usarlo en el cifrado. O bien, si la aplicación suministra una clave pública para usarlo en el cifrado, siga las instrucciones de la aplicación para descargar el certificado X.509.

    La clave pública se debe almacenar en un archivo de certificado X.509 en formato .cer.

    Si la aplicación usa una clave creada para la instancia, siga las instrucciones que proporciona la aplicación para instalar la clave privada que la aplicación usará para descifrar los tokens del inquilino de Azure AD.

1. Agregue el certificado a la configuración de la aplicación en Azure AD.

### <a name="to-configure-token-encryption-in-the-azure-portal"></a>Para configurar el cifrado de tokens en Azure Portal

Puede agregar el certificado público a la configuración de la aplicación dentro de Azure Portal.

1. Vaya a [Azure Portal](https://portal.azure.com).

1. Vaya a la hoja **Azure Active Directory > Aplicaciones empresariales** y luego seleccione la aplicación para la que quiere configurar el cifrado de tokens.

1. En la página de la aplicación, seleccione **Cifrado de tokens**.

    ![Opción de cifrado de tokens en Azure Portal](./media/howto-saml-token-encryption/token-encryption-option-small.png)

    > [!NOTE]
    > La opción **Cifrado de tokens** solo está disponible para aplicaciones SAML que se hayan configurado desde la hoja **Aplicaciones empresariales** de Azure Portal, ya sea desde la Galería de aplicaciones o desde una aplicación que no pertenece a la galería. En el caso de otras aplicaciones, esta opción de menú está deshabilitada. En el caso de las aplicaciones registradas mediante la experiencia **Registros de aplicaciones** de Azure Portal, puede configurar el cifrado de los tokens SAML con el manifiesto de aplicación, a través de Microsoft Graph o PowerShell.

1. En la página **Token encryption** (Cifrado de tokens), seleccione **Import Certificate** (Importar certificado) para importar el archivo .cer que contiene el certificado X.509 público.

    ![Importar el archivo .cer que contiene el certificado X.509](./media/howto-saml-token-encryption/import-certificate-small.png)

1. Una vez que se importa el certificado y la clave privada se configura para usarla en el lado de la aplicación, para activar el cifrado debe seleccionar **...** junto al estado de la huella digital y, luego, seleccionar **Activate token encryption** (Activar el certificado de tokens) desde las opciones del menú desplegable.

1. Seleccione **Sí** para confirmar la activación del certificado de cifrado de tokens.

1. Confirme que se cifran las aserciones SAML emitidas para la aplicación.

### <a name="to-deactivate-token-encryption-in-the-azure-portal"></a>Para desactivar el cifrado de tokens en Azure Portal

1. En Azure Portal, vaya a **Azure Active Directory > Aplicaciones empresariales** y seleccione la aplicación que tenga habilitado el cifrado de tokens SAML.

1. En la página de la aplicación, seleccione **Cifrado de tokens**, busque el certificado y seleccione la opción **...** para mostrar el menú desplegable.

1. Seleccione **Deactivate token encryption** (Desactivar el cifrado de tokens).

## <a name="configure-saml-token-encryption-using-graph-api-powershell-or-app-manifest"></a>Configuración del cifrado de tokens SAML mediante Graph API, PowerShell o el manifiesto de aplicación

Los certificados de cifrado se almacenan en el objeto de la aplicación de Azure AD con una etiqueta de uso `encrypt`. Puede configurar varios certificados de cifrado y el que está activo para el cifrado de tokens se identifica con el atributo `tokenEncryptionKeyID`.

Necesitará el identificador de objeto de la aplicación para configurar el cifrado de tokens mediante Microsoft Graph API o PowerShell. Puede encontrar este valor mediante programación, o bien en la página **Propiedades** de la aplicación en Azure Portal, anotando el valor de **Object ID**.

Cuando configure un valor keyCredential mediante Graph, PowerShell o en el manifiesto de aplicación, debe generar un GUID para usarlo para keyId.

### <a name="to-configure-token-encryption-using-microsoft-graph"></a>Para configurar el cifrado de tokens mediante Microsoft Graph

1. Actualice el objeto `keyCredentials` de la aplicación con un certificado X.509 para el cifrado. El ejemplo siguiente muestra cómo hacerlo.

    ```
    Patch https://graph.microsoft.com/beta/applications/<application objectid>

    { 
       "keyCredentials":[ 
          { 
             "type":"AsymmetricX509Cert","usage":"Encrypt",
             "keyId":"fdf8c5d8-f727-43fd-beaf-0f1521cf3d35",    (Use a GUID generator to obtain a value for the keyId)
             "key": "MIICADCCAW2gAwIBAgIQ5j9/b+n2Q4pDvQUCcy3…"  (Base64Encoded .cer file)
          }
        ]
    }
    ```

1. Identifique el certificado de cifrado activo para cifrar los tokens. El ejemplo siguiente muestra cómo hacerlo.

    ```
    Patch https://graph.microsoft.com/beta/applications/<application objectid> 

    { 
       "tokenEncryptionKeyId":"fdf8c5d8-f727-43fd-beaf-0f1521cf3d35" (The keyId of the keyCredentials entry to use)
    }
    ```

### <a name="to-configure-token-encryption-using-powershell"></a>Para configurar el cifrado de tokens mediante PowerShell

Esta funcionalidad estará disponible próximamente. 

<!--
1. Use the latest Azure AD PowerShell module to connect to your tenant.

1. Set the token encryption settings using the **[Set-AzureApplication](https://docs.microsoft.com/powershell/module/azuread/set-azureadapplication?view=azureadps-2.0-preview)** command.

    ```
    Set-AzureADApplication -ObjectId <ApplicationObjectId> -KeyCredentials “<KeyCredentialsObject>”  -TokenEncryptionKeyId <keyID>
    ```

1. Read the token encryption settings using the following commands.

    ```powershell
    $app=Get-AzureADApplication -ObjectId <ApplicationObjectId>
    $app.KeyCredentials
    $app.TokenEncryptionKeyId
    ```

-->

### <a name="to-configure-token-encryption-using-the-application-manifest"></a>Para configurar el cifrado de tokens mediante el manifiesto de aplicación

1. En Azure Portal, vaya a **Azure Active Directory > Registros de aplicaciones**.

1. Seleccione **Todas las aplicaciones** en el menú desplegable para ver todas las aplicaciones y, luego, seleccione la aplicación empresarial que quiere configurar.

1. En la página de la aplicación, seleccione **Manifiesto** para editar el [manifiesto de aplicación](../develop/reference-app-manifest.md).

1. Establezca el valor del atributo `tokenEncryptionKeyId`.

    En el ejemplo siguiente se muestra un manifiesto de aplicación configurado con dos certificados de cifrado, donde el segundo seleccionado es el activo que usa tokenEnryptionKeyId.

    ```json
    { 
      "id": "3cca40e2-367e-45a5-8440-ed94edd6cc35",
      "accessTokenAcceptedVersion": null,
      "allowPublicClient": false,
      "appId": "cb2df8fb-63c4-4c35-bba5-3d659dd81bf1",
      "appRoles": [],
      "oauth2AllowUrlPathMatching": false,
      "createdDateTime": "2017-12-15T02:10:56Z",
      "groupMembershipClaims": "SecurityGroup",
      "informationalUrls": { 
         "termsOfService": null, 
         "support": null, 
         "privacy": null, 
         "marketing": null 
      },
      "identifierUris": [ 
        "https://testapp"
      ],
      "keyCredentials": [ 
        { 
          "customKeyIdentifier": "Tog/O1Hv1LtdsbPU5nPphbMduD=", 
          "endDate": "2039-12-31T23:59:59Z", 
          "keyId": "8be4cb65-59d9-404a-a6f5-3d3fb4030351", 
          "startDate": "2018-10-25T21:42:18Z", 
          "type": "AsymmetricX509Cert", 
          "usage": "Encrypt", 
          "value": <Base64EncodedKeyFile> 
          "displayName": "CN=SAMLEncryptTest" 
        }, 
        {
          "customKeyIdentifier": "U5nPphbMduDmr3c9Q3p0msqp6eEI=",
          "endDate": "2039-12-31T23:59:59Z", 
          "keyId": "6b9c6e80-d251-43f3-9910-9f1f0be2e851",
          "startDate": "2018-10-25T21:42:18Z", 
          "type": "AsymmetricX509Cert", 
          "usage": "Encrypt", 
          "value": <Base64EncodedKeyFile> 
          "displayName": "CN=SAMLEncryptTest2" 
        } 
      ], 
      "knownClientApplications": [], 
      "logoUrl": null, 
      "logoutUrl": null, 
      "name": "Test SAML Application", 
      "oauth2AllowIdTokenImplicitFlow": true, 
      "oauth2AllowImplicitFlow": false, 
      "oauth2Permissions": [], 
      "oauth2RequirePostResponse": false, 
      "orgRestrictions": [], 
      "parentalControlSettings": { 
         "countriesBlockedForMinors": [], 
         "legalAgeGroupRule": "Allow" 
        }, 
      "passwordCredentials": [], 
      "preAuthorizedApplications": [], 
      "publisherDomain": null, 
      "replyUrlsWithType": [], 
      "requiredResourceAccess": [], 
      "samlMetadataUrl": null, 
      "signInUrl": "https://127.0.0.1:444/applications/default.aspx?metadata=customappsso|ISV9.1|primary|z" 
      "signInAudience": "AzureADMyOrg",
      "tags": [], 
      "tokenEncryptionKeyId": "6b9c6e80-d251-43f3-9910-9f1f0be2e851" 
    }  
    ```

## <a name="next-steps"></a>Pasos siguientes

* Descubra [cómo Azure AD usa el protocolo SAML](../develop/active-directory-saml-protocol-reference.md)
* Conozca el formato, las características de seguridad y el contenido de los [tokens SAML en Azure AD](../develop/reference-saml-tokens.md)