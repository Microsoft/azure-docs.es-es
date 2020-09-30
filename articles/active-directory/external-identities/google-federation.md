---
title: 'Incorporación de Google como proveedor de identidades para B2B: Azure AD'
description: Federación con Google para permitir el inicio de sesión en sus aplicaciones de Azure AD con las cuentas de Gmail propias de los usuarios
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: eef04be1891eac35577a5f4cb18d5b83b8d0f301
ms.sourcegitcommit: 5d7f8c57eaae91f7d9cf1f4da059006521ed4f9f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89669391"
---
# <a name="add-google-as-an-identity-provider-for-b2b-guest-users"></a>Incorporación de Google como proveedor de identidades para los usuarios invitados de B2B

Si configura la federación con Google, puede permitir que los usuarios invitados puedan iniciar sesión en sus aplicaciones y recursos compartidos con sus propias cuentas de Google, sin tener que crear cuentas Microsoft (MSA). 

> [!NOTE]
> La federación de Google está diseñada específicamente para los usuarios de Gmail. Para realizar la federación con los dominios de G Suite, use la característica [federación directa](direct-federation.md).

## <a name="what-is-the-experience-for-the-google-user"></a>¿Cuál es la experiencia del usuario de Google?
Cuando envíe una invitación a un usuario de Gmail de Google, el usuario invitado debe acceder a sus aplicaciones o recursos compartidos mediante un vínculo que incluya el contexto del inquilino. Su experiencia varía en función de si ya han iniciado sesión en Google:
  - Si el usuario invitado no ha iniciado sesión en Google, se le pedirá que lo haga.
  - Si el usuario invitado ya ha iniciado sesión en Google, se pedirá que elija la cuenta que desea usar. Deben elegir la cuenta donde hayan recibido la invitación.

Si el usuario invitado ve un error de "encabezado demasiado largo", puede intentar eliminar las cookies, o abrir una ventana privada o de incógnito y probar a iniciar sesión de nuevo.

![Captura de pantalla que muestra la página de inicio de sesión de Google](media/google-federation/google-sign-in.png)

## <a name="limitations"></a>Limitaciones

Teams admite totalmente a los usuarios invitados de Google en todos los dispositivos. Los usuarios de Google pueden iniciar sesión en Teams desde un punto de conexión común, como `https://teams.microsoft.com`.

Es posible que los puntos de conexión comunes de otras aplicaciones no admitan a los usuarios de Google. Los usuarios invitados de Google deberán iniciar sesión con un vínculo que incluya la información del inquilino. A continuación, se muestran algunos ejemplos:
  * `https://myapps.microsoft.com/?tenantid=<your tenant id>`
  * `https://portal.azure.com/<your tenant id>`
  * `https://myapps.microsoft.com/<your verified domain>.onmicrosoft.com`

   Si los usuarios invitados de Google intentan usar un vínculo como `https://myapps.microsoft.com` o `https://portal.azure.com`, recibirán un error.

También puede proporcionar a los usuarios invitados de Google un vínculo directo a una aplicación o recurso, siempre que este incluya su información de inquilino, por ejemplo `https://myapps.microsoft.com/signin/Twitter/<application ID?tenantId=<your tenant ID>`. 

## <a name="step-1-configure-a-google-developer-project"></a>Paso 1: configuración de un proyecto de desarrollador de Google
En primer lugar, cree un nuevo proyecto en la consola de desarrolladores de Google para obtener un identificador y un secreto de cliente que pueda agregar después a Azure AD. 
1. Vaya a las API de Google de https://console.developers.google.com e inicie sesión con su cuenta de Google. Se recomienda utilizar una cuenta de Google compartida con el equipo.
2. Si se le solicita, acepte los términos de servicio.
3. Cree un nuevo proyecto: En el panel, seleccione **Crear proyecto**, asigne un nombre al proyecto (por ejemplo, "Azure AD B2B") y, luego, elija **Crear**. 
   
   ![Captura de pantalla que muestra la página de nuevo proyecto de Google](media/google-federation/google-new-project.png)

4. En la página **API & Services** (API y servicios) que se le presenta ahora, haga clic en **Ver** en el nuevo proyecto.

5. Haga clic en **Go to APIs overview** (Ir a la información general de las API) en la tarjeta de API. Seleccione **OAuth consent screen** (Pantalla de consentimiento de OAuth).

6. Seleccione **Externo** y después **Crear**. 

7. En **Pantalla de consentimiento de OAuth**, especifique un **nombre de aplicación**. 

   ![Captura de pantalla que muestra la opción de Pantalla de consentimiento de OAuth de Google](media/google-federation/google-oauth-consent-screen.png)

8. Desplácese hasta la sección **Dominios autorizados**  y escriba microsoftonline.com.

   ![Captura de pantalla que muestra la sección de dominios autorizados](media/google-federation/google-oauth-authorized-domains.PNG)

9. Seleccione **Guardar**.

10. Elija **Credenciales**. En el menú **Create credentials** (Crear credenciales), elija **OAuth client ID** (Id. de cliente de OAuth).

    ![Captura de pantalla que muestra la opción de creación de credenciales de las API de Google](media/google-federation/google-api-credentials.png)

11. En **Tipo de aplicación**, elija **Aplicación web** y proporcione a la aplicación un nombre adecuado, por ejemplo, "Azure AD B2B"; a continuación, en **Authorized redirect URIs** (URI de redireccionamiento autorizados), escriba los siguientes URI:
    - `https://login.microsoftonline.com` 
    - `https://login.microsoftonline.com/te/<directory id>/oauth2/authresp` <br>(donde `<directory id>` es el identificador del directorio)
   
    > [!NOTE]
    > Para buscar el identificador de directorio, vaya a https://portal.azure.com y, en **Azure Active Directory**, elija **Properties** (Propiedades) y copie el valor de **Directory ID** (Id. de directorio).

    ![Captura de pantalla que muestra la sección de identificadores URI de redirección autorizados](media/google-federation/google-create-oauth-client-id.png)

12. Seleccione **Crear**. Copie el identificador y el secreto de cliente, que se usarán al agregar el proveedor de identidades en el portal de Azure AD.

    ![Captura de pantalla que muestra el Id. de cliente de OAuth y el secreto de cliente](media/google-federation/google-auth-client-id-secret.png)

## <a name="step-2-configure-google-federation-in-azure-ad"></a>Paso 2: configuración de la federación de Google en Azure AD 
Ahora podrá establecer el identificador y el secreto de cliente, ya sea escribiéndolo en el portal de Azure AD o con PowerShell. No olvide probar la configuración de la federación de Google; para ello, invítese a usted mismo mediante una dirección de Gmail e intente canjear la invitación con su cuenta de invitado de Google. 

#### <a name="to-configure-google-federation-in-the-azure-ad-portal"></a>Configuración de la federación de Google en el portal de Azure AD 
1. Vaya a [Azure Portal](https://portal.azure.com). En el panel izquierdo, seleccione **Azure Active Directory**. 
2. Seleccione **External Identities**.
3. Seleccione **Todos los proveedores de identidades** y haga clic en el botón de **Google**.
4. A continuación, escriba el identificador y el secreto de cliente que obtuvo anteriormente. Seleccione **Guardar**. 

   ![Captura de pantalla que muestra la página para agregar un proveedor de identidades de Google](media/google-federation/google-identity-provider.png)

#### <a name="to-configure-google-federation-by-using-powershell"></a>Configuración de la federación de Google con PowerShell
1. Instale la versión más reciente de Azure AD PowerShell para el módulo Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Ejecute el siguiente comando: `Connect-AzureAD`.
3. En el símbolo del sistema, inicie sesión con la cuenta de administrador global administrada.  
4. Ejecute el siguiente comando: 
   
   `New-AzureADMSIdentityProvider -Type Google -Name Google -ClientId [Client ID] -ClientSecret [Client secret]`
 
   > [!NOTE]
   > Use el identificador y secreto de cliente de la aplicación que creó en el "Paso 1: configuración de un proyecto de desarrollador de Google". Para más información, consulte el artículo sobre [New AzureADMSIdentityProvider](https://docs.microsoft.com/powershell/module/azuread/new-azureadmsidentityprovider?view=azureadps-2.0-preview). 
 
## <a name="how-do-i-remove-google-federation"></a>¿Cómo se quita la federación de Google?
La configuración de la federación de Google se puede eliminar. Si lo hace, los usuarios invitados de Google que hayan canjeado su invitación ya no podrán iniciar sesión, pero podrán acceder a sus recursos de nuevo si los elimina del directorio y los vuelve a invitar. 
 
### <a name="to-delete-google-federation-in-the-azure-ad-portal"></a>Para eliminar la federación de Google desde el portal de Azure AD: 
1. Vaya a [Azure Portal](https://portal.azure.com). En el panel izquierdo, seleccione **Azure Active Directory**. 
2. Seleccione **External Identities**.
3. Seleccione **Todos los proveedores de identidades**.
4. En la línea de **Google**, seleccione el menú contextual ( **...** ) y, a continuación, seleccione **Eliminar**. 
   
   ![Captura de pantalla que muestra la opción de eliminación del proveedor de identidades sociales](media/google-federation/google-social-identity-providers.png)

1. Seleccione **Sí** para confirmar la eliminación. 

### <a name="to-delete-google-federation-by-using-powershell"></a>Eliminación de la federación de Google con PowerShell: 
1. Instale la versión más reciente de Azure AD PowerShell para el módulo Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Ejecute `Connect-AzureAD`.  
4. En el símbolo del sistema, inicie sesión con la cuenta de administrador global administrada.  
5. Escriba el comando siguiente:

    `Remove-AzureADMSIdentityProvider -Id Google-OAUTH`

   > [!NOTE]
   > Para más información, consulte [Remove-AzureADMSIdentityProvider](https://docs.microsoft.com/powershell/module/azuread/Remove-AzureADMSIdentityProvider?view=azureadps-2.0-preview). 
