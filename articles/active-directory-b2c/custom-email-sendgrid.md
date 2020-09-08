---
title: Verificación de correo electrónico personalizado con SendGrid
titleSuffix: Azure AD B2C
description: Obtenga información sobre la integración con SendGrid para personalizar la verificación del correo electrónico que se envía a los clientes cuando se registran para usar las aplicaciones habilitadas para Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/18/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 952659746bbb99108c6177166ad60ad2272cbce6
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88584946"
---
# <a name="custom-email-verification-with-sendgrid"></a>Verificación de correo electrónico personalizado con SendGrid

Use el correo electrónico personalizado en Azure Active Directory B2C (Azure AD B2C) para enviar mensajes personalizados a los usuarios que se registran para usar las aplicaciones. Mediante el uso de elementos [DisplayControl](display-controls.md) (actualmente en versión preliminar) y el proveedor de correo electrónico de terceros SendGrid, puede usar su propia plantilla de correo electrónico, dirección *De:* y asunto, además de admitir la localización y la configuración personalizada de la contraseña de un solo uso (OTP).

La verificación del correo electrónico personalizado requiere el uso de un proveedor de correo electrónico de terceros, como [SendGrid](https://sendgrid.com), [Mailjet](https://Mailjet.com) o [SparkPost](https://sparkpost.com), una API de REST personalizada o cualquier proveedor de correo electrónico basado en HTTP (incluido el suyo propio). En este artículo se describe cómo configurar una solución que usa SendGrid.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="create-a-sendgrid-account"></a>Creación de una cuenta de SendGrid

Si aún no tiene una, empiece por configurar una cuenta de SendGrid (los clientes de Azure pueden desbloquear 25 000 mensajes de correo electrónico gratuitos al mes). Para obtener instrucciones de configuración, consulte la sección [Creación de una cuenta de SendGrid](../sendgrid-dotnet-how-to-send-email.md#create-a-sendgrid-account) en [Envío de correos electrónicos con SendGrid y Azure](../sendgrid-dotnet-how-to-send-email.md).

Asegúrese de completar la sección en la que [crea una clave de API de SendGrid](../sendgrid-dotnet-how-to-send-email.md#to-find-your-sendgrid-api-key). Anote la clave de API para usarla en un paso posterior.

## <a name="create-azure-ad-b2c-policy-key"></a>Creación de la clave de directiva de Azure AD B2C

A continuación, almacene la clave de API de SendGrid en una clave de directiva de Azure AD B2C que servirá de referencia para las directivas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio + suscripción** en el menú superior y elija el directorio de Azure AD B2C.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. En la página de introducción, seleccione **Identity Experience Framework**.
1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija **Manual**.
1. Escriba un **nombre** para la clave de directiva. Por ejemplo, `SendGridSecret`. Se agregará el prefijo `B2C_1A_` automáticamente al nombre de la clave.
1. En **Secreto**, escriba el secreto de cliente que haya registrado previamente.
1. En **Uso de claves**, seleccione **Firma**.
1. Seleccione **Crear**.

## <a name="create-sendgrid-template"></a>Creación de la plantilla de SendGrid

Con la cuenta de SendGrid creada y la clave de API de SendGrid almacenada en una clave de directiva de Azure AD B2C, cree una [plantilla transaccional dinámica](https://sendgrid.com/docs/ui/sending-email/how-to-send-an-email-with-dynamic-transactional-templates/) de SendGrid.

1. En el sitio de SendGrid, abra la página [Transactional Templates](https://sendgrid.com/dynamic_templates) (Plantillas transaccionales) y seleccione **Create Template** (Crear plantilla).
1. Escriba un nombre de plantilla único, como por ejemplo `Verification email`, y, a continuación, seleccione **Save** (Guardar).
1. Para empezar a editar la nueva plantilla, seleccione **Add Version** (Agregar versión).
1. Seleccione **Code Editor** (Editor de código) y, a continuación, **Continue** (Continuar).
1. En el editor de HTML, pegue la siguiente plantilla HTML o use la suya propia. Los parámetros `{{otp}}` y `{{email}}` se reemplazarán dinámicamente con el valor de la contraseña de un solo uso y la dirección de correo electrónico del usuario.

    ```HTML
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml" dir="ltr" lang="en"><head id="Head1">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"><title>Contoso demo account email verification code</title><meta name="ROBOTS" content="NOINDEX, NOFOLLOW">
       <!-- Template B O365 -->
       <style>
           table td {border-collapse:collapse;margin:0;padding:0;}
       </style>
    </head>
    <body dir="ltr" lang="en">
       <table width="100%" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en">
            <tr>
               <td valign="top" width="50%"></td>
               <td valign="top">
                  <!-- Email Header -->
                  <table width="640" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en" style="border-left:1px solid #e3e3e3;border-right: 1px solid #e3e3e3;">
                   <tr style="background-color: #0072C6;">
                       <td width="1" style="background:#0072C6; border-top:1px solid #e3e3e3;"></td>
                       <td width="24" style="border-top:1px solid #e3e3e3;border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td width="310" valign="middle" style="border-top:1px solid #e3e3e3; border-bottom:1px solid #e3e3e3;padding:12px 0;">
                           <h1 style="line-height:20pt;font-family:Segoe UI Light; font-size:18pt; color:#ffffff; font-weight:normal;">
                            <span id="HeaderPlaceholder_UserVerificationEmailHeader"><font color="#FFFFFF">Verify your email address</font></span>
                           </h1>
                       </td>
                       <td width="24" style="border-top: 1px solid #e3e3e3;border-bottom: 1px solid #e3e3e3;">&nbsp;</td>
                   </tr>
                  </table>
                  <!-- Email Content -->
                  <table width="640" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en">
                   <tr>
                       <td width="1" style="background:#e3e3e3;"></td>
                       <td width="24">&nbsp;</td>
                       <td id="PageBody" width="640" valign="top" colspan="2" style="border-bottom:1px solid #e3e3e3;padding:10px 0 20px;border-bottom-style:hidden;">
                           <table cellpadding="0" cellspacing="0" border="0">
                               <tr>
                                   <td width="630" style="font-size:10pt; line-height:13pt; color:#000;">
                                       <table cellpadding="0" cellspacing="0" border="0" width="100%" style="" dir="ltr" lang="en">
                                           <tr>
                                               <td>

       <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333;">
           <span id="BodyPlaceholder_UserVerificationEmailBodySentence1">Thanks for verifying your {{email}} account!</span>
       </div>
       <br>
       <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333; font-weight: bold">
           <span id="BodyPlaceholder_UserVerificationEmailBodySentence2">Your code is: {{otp}}</span>
       </div>
       <br>
       <br>

                                                   <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333;">
                                                   Sincerely,
                                                   </div>
                                                   <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; font-style:italic; color:#333;">
                                                       Contoso
                                                   </div>
                                               </td>
                                           </tr>
                                       </table>
                                   </td>
                               </tr>
                           </table>

                       </td>

                       <td width="1">&nbsp;</td>
                       <td width="1"></td>
                       <td width="1">&nbsp;</td>
                       <td width="1" valign="top"></td>
                       <td width="29">&nbsp;</td>
                       <td width="1" style="background:#e3e3e3;"></td>
                   </tr>
                   <tr>
                       <td width="1" style="background:#e3e3e3; border-bottom:1px solid #e3e3e3;"></td>
                       <td width="24" style="border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td id="PageFooterContainer" width="585" valign="top" colspan="6" style="border-bottom:1px solid #e3e3e3;padding:0px;">


                       </td>

                       <td width="29" style="border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td width="1" style="background:#e3e3e3; border-bottom:1px solid #e3e3e3;"></td>
                   </tr>
                  </table>

               </td>
               <td valign="top" width="50%"></td>
           </tr>
       </table>
    <img src="https://mucp.api.account.microsoft.com/m/v2/v?d=AIAACWEPFYXYIUTJIJVV4ST7XLBHVI5MLLYBKJAVXHBDTBHUM5VBSVVPTTVRWDFIXJ5JQTHYOH5TUYIPO4ZAFRFK52UAMIS3UNIPPI7ZJNDZPRXD5VEJBN4H6RO3SPTBS6AJEEAJOUYL4APQX5RJUJOWGPKUABY&amp;i=AIAACL23GD2PFRFEY5YVM2XQLM5YYWMHFDZOCDXUI2B4LM7ETZQO473CVF22PT6WPGR5IIE6TCS6VGEKO5OZIONJWCDMRKWQQVNP5VBYAINF3S7STKYOVDJ4JF2XEW4QQVNHMAPQNHFV3KMR3V3BA4I36B6BO7L4VQUHQOI64EOWPLMG5RB3SIMEDEHPILXTF73ZYD3JT6MYOLAZJG7PJJCAXCZCQOEFVH5VCW2KBQOKRYISWQLRWAT7IINZ3EFGQI2CY2EMK3FQOXM7UI3R7CZ6D73IKDI" width="1" height="1"></body>
    </html>
    ```

1. Expanda **Settings** (Configuración) a la izquierda y, en **Email Subject** (Asunto del correo electrónico), escriba `{{subject}}`.
1. Seleccione **Save Template** (Guardar plantilla).
1. Seleccione la flecha atrás para volver a la página **Transactional Templates** (Plantillas transaccionales).
1. Anote el valor de **ID** de la plantilla que creó para usarlo en un paso posterior. Por ejemplo, `d-989077fbba9746e89f3f6411f596fb96`. Este identificador se especifica al [agregar la transformación de notificaciones](#add-the-claims-transformation).

## <a name="add-azure-ad-b2c-claim-types"></a>Incorporación de tipos de notificaciones de Azure AD B2C

En la directiva, agregue los siguientes tipos de notificaciones al elemento `<ClaimsSchema>` dentro de `<BuildingBlocks>`.

Estos tipos de notificaciones son necesarios para generar y verificar la dirección de correo electrónico mediante un código de contraseña de un solo uso (OTP).

```xml
<ClaimType Id="Otp">
  <DisplayName>Secondary One-time password</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="emailRequestBody">
  <DisplayName>SendGrid request body</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="VerificationCode">
  <DisplayName>Secondary Verification Code</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Enter your email verification code</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```

## <a name="add-the-claims-transformation"></a>Incorporación de la transformación de notificaciones

A continuación, necesita una transformación de notificaciones para generar una notificación de cadena JSON que será el cuerpo de la solicitud enviada a SendGrid.

La estructura del objeto JSON se define mediante los identificadores de la notación de puntos de InputParameters y los valores de TransformationClaimTypes de InputClaims. Los números de la notación de puntos implican matrices. Los valores provienen de los valores de InputClaims y de las propiedades "value" de InputParameters. Para más información sobre las transformaciones de notificaciones de JSON, consulte [Transformaciones de notificaciones de JSON](json-transformations.md).

Agregue la siguiente transformación de notificaciones al elemento `<ClaimsTransformations>` dentro de `<BuildingBlocks>`. Realice las siguientes actualizaciones en el XML de la transformación de notificaciones:

* Actualice el valor `template_id` de InputParameter con el identificador de la plantilla transaccional de SendGrid que creó anteriormente en [Creación de la plantilla de SendGrid](#create-sendgrid-template).
* Actualice el valor de dirección de `from.email`. Use una dirección de correo electrónico válida para evitar que el correo electrónico de verificación se marque como correo no deseado.
* Actualice el valor del parámetro de entrada de la línea de asunto `personalizations.0.dynamic_template_data.subject` con una línea de asunto adecuada para su organización.

```xml
<ClaimsTransformation Id="GenerateEmailRequestBody" TransformationMethod="GenerateJson">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.to.0.email" />
    <InputClaim ClaimTypeReferenceId="otp" TransformationClaimType="personalizations.0.dynamic_template_data.otp" />
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.dynamic_template_data.email" />
  </InputClaims>
  <InputParameters>
    <!-- Update the template_id value with the ID of your SendGrid template. -->
    <InputParameter Id="template_id" DataType="string" Value="d-989077fbba9746e89f3f6411f596fb96"/>
    <InputParameter Id="from.email" DataType="string" Value="my_email@mydomain.com"/>
    <!-- Update with a subject line appropriate for your organization. -->
    <InputParameter Id="personalizations.0.dynamic_template_data.subject" DataType="string" Value="Contoso account email verification code"/>
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="emailRequestBody" TransformationClaimType="outputClaim"/>
  </OutputClaims>
</ClaimsTransformation>
```

## <a name="add-datauri-content-definition"></a>Incorporación de definición de contenido de DataUri

Bajo las transformaciones de notificaciones dentro de `<BuildingBlocks>`, agregue el siguiente elemento [ContentDefinition](contentdefinitions.md) para hacer referencia al URI de datos de la versión 2.0.0:

```xml
<ContentDefinitions>
 <ContentDefinition Id="api.localaccountsignup">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.0.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.0.0</DataUri>
  </ContentDefinition>
</ContentDefinitions>
```

## <a name="create-a-displaycontrol"></a>Creación de un elemento DisplayControl

Para verificar la dirección de correo electrónico con un código de verificación que se envía al usuario, se utiliza un control de visualización de verificación.

Este control de visualización de ejemplo está configurado para:

1. Recopilar el tipo de notificación de dirección `email` del usuario.
1. Esperar a que el usuario proporcione el tipo de notificación `verificationCode` con el código que se le envía.
1. Devolver el valor de `email` al perfil técnico autoafirmado que tiene una referencia a este control de visualización.
1. Con la acción `SendCode`, generar un código OTP y enviar un correo electrónico con este código al usuario.

![Acción de correo electrónico de envío de código de verificación](media/custom-email-sendgrid/display-control-verification-email-action-01.png)

En las definiciones de contenido, todavía dentro de `<BuildingBlocks>`, agregue el siguiente elemento [DisplayControl](display-controls.md) de tipo [VerificationControl](display-control-verification.md) a la directiva.

```xml
<DisplayControls>
  <DisplayControl Id="emailVerificationControl" UserInterfaceControlType="VerificationControl">
    <DisplayClaims>
      <DisplayClaim ClaimTypeReferenceId="email" Required="true" />
      <DisplayClaim ClaimTypeReferenceId="verificationCode" ControlClaimType="VerificationCode" Required="true" />
    </DisplayClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="email" />
    </OutputClaims>
    <Actions>
      <Action Id="SendCode">
        <ValidationClaimsExchange>
          <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="GenerateOtp" />
          <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="SendOtp" />
        </ValidationClaimsExchange>
      </Action>
      <Action Id="VerifyCode">
        <ValidationClaimsExchange>
          <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="VerifyOtp" />
        </ValidationClaimsExchange>
      </Action>
    </Actions>
  </DisplayControl>
</DisplayControls>
```

## <a name="add-otp-technical-profiles"></a>Incorporación de perfiles técnicos de OTP

El perfil técnico `GenerateOtp` genera un código para la dirección de correo electrónico. El perfil técnico `VerifyOtp` verifica el código asociado a la dirección de correo electrónico. Puede cambiar la configuración del formato y la expiración de la contraseña de un solo uso. Para más información sobre los perfiles técnicos de OTP, consulte [Definición de un perfil técnico de una contraseña de un solo uso en una directiva personalizada de Azure AD B2C](one-time-password-technical-profile.md).

Agregue los siguientes perfiles técnicos al elemento `<ClaimsProviders>`.

```xml
<ClaimsProvider>
  <DisplayName>One time password technical profiles</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="GenerateOtp">
      <DisplayName>Generate one time password</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.OneTimePasswordProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="Operation">GenerateCode</Item>
        <Item Key="CodeExpirationInSeconds">1200</Item>
        <Item Key="CodeLength">6</Item>
        <Item Key="CharacterSet">0-9</Item>
        <Item Key="ReuseSameCode">true</Item>
        <Item Key="MaxNumAttempts">5</Item>
      </Metadata>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="identifier" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="otp" PartnerClaimType="otpGenerated" />
      </OutputClaims>
    </TechnicalProfile>

    <TechnicalProfile Id="VerifyOtp">
      <DisplayName>Verify one time password</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.OneTimePasswordProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="Operation">VerifyCode</Item>
      </Metadata>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="identifier" />
        <InputClaim ClaimTypeReferenceId="verificationCode" PartnerClaimType="otpToVerify" />
      </InputClaims>
    </TechnicalProfile>
   </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="add-a-rest-api-technical-profile"></a>Incorporación de un perfil técnico de API REST

Este perfil técnico de API REST genera el contenido del correo electrónico (con el formato de SendGrid). Para más información sobre los perfiles técnicos de RESTful, consulte [Definición de un perfil técnico de RESTful en una directiva personalizada en Azure Active Directory B2C](restful-technical-profile.md).

Al igual que hizo con los perfiles técnicos de OTP, agregue los siguientes perfiles técnicos al elemento `<ClaimsProviders>`.

```xml
<ClaimsProvider>
  <DisplayName>RestfulProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SendOtp">
      <DisplayName>Use SendGrid's email API to send the code the the user</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://api.sendgrid.com/v3/mail/send</Item>
        <Item Key="AuthenticationType">Bearer</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="ClaimUsedForRequestPayload">emailRequestBody</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BearerAuthenticationToken" StorageReferenceId="B2C_1A_SendGridSecret" />
      </CryptographicKeys>
      <InputClaimsTransformations>
        <InputClaimsTransformation ReferenceId="GenerateEmailRequestBody" />
      </InputClaimsTransformations>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="emailRequestBody" />
      </InputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="make-a-reference-to-the-displaycontrol"></a>Referencia al elemento DisplayControl

En el paso final, agregue una referencia al elemento DisplayControl que creó. Reemplace sus perfiles técnicos autoafirmados `LocalAccountSignUpWithLogonEmail` y `LocalAccountDiscoveryUsingEmailAddress` existentes por lo siguiente. Si usó una versión anterior de la directiva de Azure AD B2C. Estos perfiles técnicos usan `DisplayClaims` con una referencia al elemento DisplayControl.

Para más información, consulte [Definición de un perfil técnico de RESTful en una directiva personalizada en Azure Active Directory B2C](restful-technical-profile.md) y [Controles de visualización](display-controls.md).

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
      <Metadata>
        <!--OTP validation error messages-->
        <Item Key="UserMessageIfSessionDoesNotExist">You have exceed the maximum time allowed.</Item>
        <Item Key="UserMessageIfMaxRetryAttempted">You have exceed the number of retries allowed.</Item>
        <Item Key="UserMessageIfInvalidCode">You have entered the wrong code.</Item>
        <Item Key="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</Item>
      </Metadata>
      <DisplayClaims>
        <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
        <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="newPassword" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
      </DisplayClaims>
    </TechnicalProfile>
    <TechnicalProfile Id="LocalAccountDiscoveryUsingEmailAddress">
      <Metadata>
        <!--OTP validation error messages-->
        <Item Key="UserMessageIfSessionDoesNotExist">You have exceed the maximum time allowed.</Item>
        <Item Key="UserMessageIfMaxRetryAttempted">You have exceed the number of retries allowed.</Item>
        <Item Key="UserMessageIfInvalidCode">You have entered the wrong code.</Item>
        <Item Key="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</Item>
      </Metadata>
      <DisplayClaims>
        <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
      </DisplayClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="optional-localize-your-email"></a>(Opcional) Localización del correo electrónico

Para localizar el correo electrónico, debe enviar las cadenas localizadas a SendGrid o al proveedor de correo electrónico. Por ejemplo, puede localizar el asunto del correo electrónico, el cuerpo, el mensaje de código o la firma del correo electrónico. Para ello, puede usar la transformación de notificaciones [GetLocalizedStringsTransformation](string-transformations.md) para copiar cadenas localizadas en tipos de notificación. La transformación de notificaciones `GenerateEmailRequestBody`, que genera la carga JSON, utiliza notificaciones de entrada que contienen las cadenas localizadas.

1. En la directiva, defina las siguientes notificaciones de cadena: subject, message, codeIntro y signature.
1. Defina una transformación de notificaciones [GetLocalizedStringsTransformation](string-transformations.md) que sustituya los valores de cadena localizados por las notificaciones del paso 1.
1. Cambie la transformación de notificaciones `GenerateEmailRequestBody` para que use notificaciones de entrada con el siguiente fragmento de código XML.
1. Actualice la plantilla de SendGrid para que use los parámetros dinámicos en lugar de todas las cadenas que se localizarán mediante Azure AD B2C.

    ```xml
    <ClaimsTransformation Id="GetLocalizedStringsForEmail" TransformationMethod="GetLocalizedStringsTransformation">
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="subject" TransformationClaimType="email_subject" />
        <OutputClaim ClaimTypeReferenceId="message" TransformationClaimType="email_message" />
        <OutputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="email_code" />
        <OutputClaim ClaimTypeReferenceId="signature" TransformationClaimType="email_signature" />
      </OutputClaims>
    </ClaimsTransformation>
    <ClaimsTransformation Id="GenerateEmailRequestBody" TransformationMethod="GenerateJson">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.to.0.email" />
        <InputClaim ClaimTypeReferenceId="subject" TransformationClaimType="personalizations.0.dynamic_template_data.subject" />
        <InputClaim ClaimTypeReferenceId="otp" TransformationClaimType="personalizations.0.dynamic_template_data.otp" />
        <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.dynamic_template_data.email" />
        <InputClaim ClaimTypeReferenceId="message" TransformationClaimType="personalizations.0.dynamic_template_data.message" />
        <InputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="personalizations.0.dynamic_template_data.codeIntro" />
        <InputClaim ClaimTypeReferenceId="signature" TransformationClaimType="personalizations.0.dynamic_template_data.signature" />
      </InputClaims>
      <InputParameters>
        <InputParameter Id="template_id" DataType="string" Value="d-1234567890" />
        <InputParameter Id="from.email" DataType="string" Value="my_email@mydomain.com" />
      </InputParameters>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="emailRequestBody" TransformationClaimType="outputClaim" />
      </OutputClaims>
    </ClaimsTransformation>
    ```

1. Agregue el siguiente elemento [Localization](localization.md).

    ```xml
    <Localization Enabled="true">
      <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
        <SupportedLanguage>en</SupportedLanguage>
        <SupportedLanguage>es</SupportedLanguage>
      </SupportedLanguages>
      <LocalizedResources Id="api.custom-email.en">
        <LocalizedStrings>
          <!--Email template parameters-->
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for validating the account</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
        </LocalizedStrings>
      </LocalizedResources>
      <LocalizedResources Id="api.custom-email.es">
        <LocalizedStrings>
          <!--Email template parameters-->
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Código de verificación del correo electrónico de la cuenta de Contoso</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Gracias por comprobar la cuenta de </LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Su código es</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sinceramente</LocalizedString>
        </LocalizedStrings>
      </LocalizedResources>
    </Localization>
    ```

1. Agregue referencias a los elementos LocalizedResources actualizando el elemento [ContentDefinitions](contentdefinitions.md).

    ```XML
    <ContentDefinitions>
      <ContentDefinition Id="api.localaccountsignup">
        <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.0.0</DataUri>
        <LocalizedResourcesReferences MergeBehavior="Prepend">
          <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.custom-email.en" />
          <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.custom-email.es" />
        </LocalizedResourcesReferences>
      </ContentDefinition>
      <ContentDefinition Id="api.localaccountpasswordreset">
        <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.0.0</DataUri>
        <LocalizedResourcesReferences MergeBehavior="Prepend">
          <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.custom-email.en" />
          <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.custom-email.es" />
        </LocalizedResourcesReferences>
      </ContentDefinition>
    </ContentDefinitions>
    ```

1. Por último, agregue la siguiente transformación de las notificaciones de entrada a los perfiles técnicos `LocalAccountSignUpWithLogonEmail` y `LocalAccountDiscoveryUsingEmailAddress`.

    ```XML
    <InputClaimsTransformations>
      <InputClaimsTransformation ReferenceId="GetLocalizedStringsForEmail" />
    </InputClaimsTransformations>
    ```

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar un ejemplo de una directiva de verificación de correo electrónico personalizado en GitHub:

- [Verificación de correo electrónico personalizado: DisplayControls](https://github.com/azure-ad-b2c/samples/tree/master/policies/custom-email-verifcation-displaycontrol)
- Para obtener información sobre el uso de una API REST personalizada o de cualquier proveedor de correo electrónico SMTP basado en HTTP, consulte [Definición de un perfil técnico de RESTful en una directiva personalizada en Azure Active Directory B2C](restful-technical-profile.md).
