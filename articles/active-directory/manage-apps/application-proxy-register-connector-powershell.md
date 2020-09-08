---
title: Instalación silenciosa del conector del proxy de la aplicación de Azure AD | Microsoft Docs
description: Explica cómo realizar una instalación no atendida del conector de Proxy de aplicación de Azure AD para proporcionar acceso remoto seguro a las aplicaciones locales.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/24/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: a5fc1b84e624828d7feb64bd53e8fe8ffff2a7ff
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88054841"
---
# <a name="create-an-unattended-installation-script-for-the-azure-ad-application-proxy-connector"></a>Creación de un script de instalación desatendida para el conector del proxy de la aplicación de Azure AD

Este tema le ayuda a crear un script de Windows PowerShell que hace posible la instalación desatendida y el registro para el conector del proxy de la aplicación de Azure AD.

Esta capacidad resulta útil cuando desea hacer lo siguiente:

* Instale el conector en los servidores de Windows que no tienen interfaz de usuario habilitada o a los que no puede acceder con Escritorio remoto.
* Instalar y registrar muchos conectores a la vez.
* Integrar la instalación del conector y el registro como parte de otro procedimiento.
* Crear una imagen de servidor estándar que contiene los bits de conector, pero no está registrada.

Para que el [conector del proxy de la aplicación](application-proxy-connectors.md) funcione, debe registrarse con el directorio de Azure AD mediante un administrador de aplicaciones y una contraseña. Normalmente, esta información se especifica durante la instalación del conector en un cuadro de diálogo emergente, pero también puede usar PowerShell para automatizar este proceso.

Hay dos pasos para una instalación desatendida. Primero, instale el conector. Después, registre el conector con Azure AD.

> [!IMPORTANT]
> Si va a instalar el conector para la nube de Azure Government, consulte los [requisitos previos](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-government-cloud#allow-access-to-urls) y los [pasos de instalación](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-government-cloud#install-the-agent-for-the-azure-government-cloud). Esto requiere habilitar el acceso a un conjunto diferente de direcciones URL y que un parámetro adicional ejecute la instalación.

## <a name="install-the-connector"></a>Instalación del conector
Siga estos pasos para instalar el conector sin registrarlo:

1. Abra un símbolo del sistema.
2. Ejecute el siguiente comando, en el que el parámetro /q significa instalación desatendida. Una instalación desatendida no le pide que acepte el contrato de licencia para el usuario final.

   ```
   AADApplicationProxyConnectorInstaller.exe REGISTERCONNECTOR="false" /q
   ```

## <a name="register-the-connector-with-azure-ad"></a>Registro del conector con Azure AD
Hay dos métodos posibles para registrar el conector:

* Registro del conector mediante un objeto de credenciales de Windows PowerShell
* Registro del conector mediante un token creado sin conexión

### <a name="register-the-connector-using-a-windows-powershell-credential-object"></a>Registro del conector mediante un objeto de credenciales de Windows PowerShell
1. Cree un objeto de credenciales de Windows PowerShell `$cred` que contiene un nombre de usuario y una contraseña administrativos para el directorio. Ejecute el comando siguiente, reemplazando *\<username\>* y *\<password\>* :

   ```powershell
   $User = "<username>"
   $PlainPassword = '<password>'
   $SecurePassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
   $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $User, $SecurePassword
   ```
2. Vaya a **C:\Archivos de programa\Microsoft AAD App Proxy Connector** y ejecute el siguiente script mediante el objeto `$cred` que creó:

   ```powershell
   .\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Credentials -Usercredentials $cred -Feature ApplicationProxy
   ```

### <a name="register-the-connector-using-a-token-created-offline"></a>Registro del conector mediante un token creado sin conexión
1. Cree un token sin conexión mediante la clase AuthenticationContext con los valores de este fragmento de código o con los cmdlets de PowerShell que tiene a continuación:

   **Con C#:**

   ```csharp
   using System;
   using System.Linq;
   using System.Collections.Generic;
   using Microsoft.Identity.Client;

   class Program
   {
   #region constants
   /// <summary>
   /// The AAD authentication endpoint uri
   /// </summary>
   static readonly string AadAuthenticationEndpoint = "https://login.microsoftonline.com/common/oauth2/v2.0/authorize";

   /// <summary>
   /// The application ID of the connector in AAD
   /// </summary>
   static readonly string ConnectorAppId = "55747057-9b5d-4bd4-b387-abf52a8bd489";
 
   /// <summary>
   /// The AppIdUri of the registration service in AAD
   /// </summary>
   static readonly string RegistrationServiceAppIdUri = "https://proxy.cloudwebappproxy.net/registerapp/user_impersonation";

   #endregion

   #region private members
   private string token;
   private string tenantID;
   #endregion

   public void GetAuthenticationToken()
   {
    
   IPublicClientApplication clientApp = PublicClientApplicationBuilder
      .Create(ConnectorAppId)
      .WithDefaultRedirectUri() // will automatically use the default Uri for native app
      .WithAuthority(AadAuthenticationEndpoint)
      .Build();

      AuthenticationResult authResult = null;
            
      IAccount account = null;

      IEnumerable<string> scopes = new string[] { RegistrationServiceAppIdUri };

      try
      {
       authResult = await clientApp.AcquireTokenSilent(scopes, account).ExecuteAsync();
      }
       catch (MsalUiRequiredException ex)
      {
       authResult = await clientApp.AcquireTokenInteractive(scopes).ExecuteAsync();
      }


      if (authResult == null || string.IsNullOrEmpty(authResult.AccessToken) || string.IsNullOrEmpty(authResult.TenantId))
      {
       Trace.TraceError("Authentication result, token or tenant id returned are null");
       throw new InvalidOperationException("Authentication result, token or tenant id returned are null");
      }

      token = authResult.AccessToken;
      tenantID = authResult.TenantId;
      }
      ```

   **Con PowerShell:**

   ```powershell
   # Load MSAL (Tested with version 4.7.1) 

   Add-Type -Path "..\MSAL\Microsoft.Identity.Client.dll" 
        
   # The AAD authentication endpoint uri
        
   $authority = "https://login.microsoftonline.com/common/oauth2/v2.0/authorize"

   #The application ID of the connector in AAD

   $connectorAppId = "55747057-9b5d-4bd4-b387-abf52a8bd489";

   #The AppIdUri of the registration service in AAD
   $registrationServiceAppIdUri = "https://proxy.cloudwebappproxy.net/registerapp/user_impersonation"

   # Define the resources and scopes you want to call 

   $scopes = New-Object System.Collections.ObjectModel.Collection["string"] 

   $scopes.Add($registrationServiceAppIdUri)

   $app = [Microsoft.Identity.Client.PublicClientApplicationBuilder]::Create($connectorAppId).WithAuthority($authority).WithDefaultRedirectUri().Build()

   [Microsoft.Identity.Client.IAccount] $account = $null

   # Acquiring the token 

   $authResult = $null

   $authResult = $app.AcquireTokenInteractive($scopes).WithAccount($account).ExecuteAsync().ConfigureAwait($false).GetAwaiter().GetResult()

   # Check AuthN result
   If (($authResult) -and ($authResult.AccessToken) -and ($authResult.TenantId)) {
        
   $token = $authResult.AccessToken
   $tenantId = $authResult.TenantId

   Write-Output "Success: Authentication result returned."
        
   }
   Else {
         
   Write-Output "Error: Authentication result, token or tenant id returned with null."
        
   } 
   ```

2. Una vez que tenga el token, cree SecureString con dicho token:

   ```powershell
   $SecureToken = $Token | ConvertTo-SecureString -AsPlainText -Force
   ```

3. Ejecute el siguiente comando de Windows PowerShell, reemplazando \<tenant GUID\> por su identificador de directorio:

   ```powershell
   .\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Token -Token $SecureToken -TenantId <tenant GUID> -Feature ApplicationProxy
   ```

## <a name="next-steps"></a>Pasos siguientes
* [Publicar aplicaciones mediante su propio nombre de dominio](application-proxy-configure-custom-domain.md)
* [Habilitar el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md)
* [Solucionar los problemas que tiene con el Proxy de aplicación](application-proxy-troubleshoot.md)
