---
title: Autenticación basada en encabezados con PingAccess para el proxy de la aplicación de Azure AD | Microsoft Docs
description: Publique aplicaciones con PingAccess y el proxy de la aplicación que admitan la autenticación basada en encabezados.
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
ms.date: 10/24/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 512aed93906968b14f7c6a13e08f74bbeb2f5f31
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431076"
---
# <a name="header-based-authentication-for-single-sign-on-with-application-proxy-and-pingaccess"></a>Autenticación basada en el encabezado para el inicio de sesión único con el proxy de aplicación y PingAccess

Azure Active Directory (Azure AD) Application Proxy se ha asociado con PingAccess para que los clientes de Azure AD puedan tener acceso a un mayor número de sus aplicaciones. PingAccess expande las [ofertas actuales del proxy de aplicación](application-proxy.md) para incluir el acceso de inicio sesión único a las aplicaciones que utilicen encabezados para la autenticación.

## <a name="whats-pingaccess-for-azure-ad"></a>¿Qué es PingAccess para Azure AD?

Con PingAccess para Azure AD, los usuarios pueden acceder y realizar un inicio de sesión único (SSO) en las aplicaciones que usan encabezados para la autenticación. El proxy de aplicación trata estas aplicaciones como a las demás, usa Azure AD para autenticar el acceso y, después, pasa el tráfico a través del servicio de conector. PingAccess se coloca delante de las aplicaciones y convierte el token de acceso de Azure AD en un encabezado. Luego, la aplicación recibe la autenticación en un formato que pueda leer.

Los usuarios no notarán ninguna diferencia al iniciar sesión para usar las aplicaciones corporativas. Podrán seguir trabajando desde cualquier lugar y en cualquier dispositivo. Los conectores de Application Proxy dirigen el tráfico remoto a todas las aplicaciones, independientemente de su tipo de autenticación, con lo cual seguirán equilibrando la carga de forma automática.

## <a name="how-do-i-get-access"></a>¿Cómo puedo obtener acceso?

Dado que este escenario nace de una asociación entre Azure Active Directory y PingAccess, se necesitarán licencias de ambos servicios. Sin embargo, las suscripciones Premium de Azure Active Directory incluyen una licencia básica de PingAccess que abarca hasta 20 aplicaciones. Si tiene que publicar más de 20 aplicaciones basadas en el encabezado, puede adquirir una licencia adicional de PingAccess.

Para obtener más información, consulte [Ediciones de Azure Active Directory](../fundamentals/active-directory-whatis.md).

## <a name="publish-your-application-in-azure"></a>Publicación de una aplicación en Azure

Este artículo va dirigido a quienes publican por primera vez una aplicación con este escenario. Aparte de detallar los pasos de publicación, sirve de guía para empezar a usar Application Proxy y PingAccess. Si ya configuró ambos servicios, pero desea revisar los pasos de publicación, vaya a la sección [Agregar la aplicación a Azure AD con el proxy de aplicación](#add-your-application-to-azure-ad-with-application-proxy).

> [!NOTE]
> Dado que este escenario es una asociación entre Azure AD y PingAccess, algunas de las instrucciones se encuentran en el sitio de Ping Identity.

### <a name="install-an-application-proxy-connector"></a>Instalación de un conector del proxy de la aplicación

Si ya tiene habilitado Application Proxy y tiene instalado un conector, puede omitir esta sección e ir a [Agregar la aplicación a Azure AD con el proxy de aplicación](#add-your-application-to-azure-ad-with-application-proxy).

El conector de Application Proxy es un servicio de Windows Server que dirige el tráfico de los empleados remotos a las aplicaciones publicadas. Para obtener instrucciones de instalación detalladas, vea [Tutorial: Adición de una aplicación local para el acceso remoto mediante Application Proxy en Azure Active Directory](application-proxy-add-on-premises-application.md).

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/) como administrador de la aplicación. Aparecerá la página del **Centro de administración de Azure Active Directory**.
1. Seleccione **Azure Active Directory** > **Proxy de la aplicación** > **Descargar servicio de conector**. Se abre la página **Descarga de conectores del proxy de aplicación**.

   ![Descarga del conector Application Proxy](./media/application-proxy-configure-single-sign-on-with-ping-access/application-proxy-connector-download.png)

1. Siga las instrucciones de instalación.

La descarga del conector debería habilitar automáticamente Application Proxy en el directorio, pero si no lo hace, puede seleccionar **Habilitar el proxy de la aplicación**.

### <a name="add-your-application-to-azure-ad-with-application-proxy"></a>Incorporación de la aplicación a Azure AD con Application Proxy

Hay dos acciones que se deben tomar en Azure Portal. En primer lugar, debe publicar la aplicación con el proxy de aplicación. Después, debe recopilar información acerca de la aplicación que puede usar en los pasos de PingAccess.

#### <a name="publish-your-application"></a>Publicación de la aplicación

En primer lugar, deberá publicar la aplicación. Esta acción conlleva lo siguiente:

- Agregar una aplicación local a Azure AD.
- Designar un usuario para probar la aplicación y seleccionar SSO basado en encabezados.
- Configurar la dirección URL de redireccionamiento de la aplicación.
- Conceder permisos para que los usuarios y otras aplicaciones puedan usar la aplicación local.

Para publicar su propia aplicación local:

1. Si no lo hizo en la última sección, inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/) como administrador de la aplicación.
1. Seleccione **Aplicaciones empresariales** > **Nueva aplicación** > **Incorporación de una aplicación local**. Se abre la página **Agregar aplicación local propia**.

   ![Agregar aplicación local propia](./media/application-proxy-configure-single-sign-on-with-ping-access/add-your-own-on-premises-application.png)
1. Rellene los campos requeridos con la información de la aplicación nueva. Utilice las instrucciones siguientes para configurarla.

   > [!NOTE]
   > Para obtener un tutorial más detallado de este paso, vea [ Adición de una aplicación local a Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).

   1. **Dirección URL interna**: normalmente, especifica la dirección URL que lleva a la página de inicio de sesión la aplicación cuando esté en la red corporativa. En este escenario, el conector tiene que tratar el proxy de PingAccess como la página principal de la aplicación. Use este formato: `https://<host name of your PingAccess server>:<port>`. El puerto predeterminado es 3000, pero puedo configurar uno distinto en PingAccess.

      > [!WARNING]
      > En este tipo de inicio de sesión único, debe usar la dirección URL interna `https` y no se puede usar `http`.

   1. **Método de autenticación previa**: elija **Azure Active Directory**.
   1. **Traducir URL en encabezados**: elija **No**.

   > [!NOTE]
   > Si esta es su primera aplicación, utilice el puerto 3000 para empezar y regrese para actualizar este valor si cambia la configuración de PingAccess. En aplicaciones posteriores, el puerto deberá coincidir con la escucha que ha configurado en PingAccess. Obtenga más información sobre [escucha en PingAccess](https://support.pingidentity.com/s/document-item?bundleId=pingaccess-52&topicId=reference/ui/pa_c_Listeners.html).

1. Seleccione **Agregar**. Aparecerá la página de información general de la nueva aplicación.

Ahora, designe a un usuario para probar la aplicación y elija un inicio de sesión único basado en encabezado:

1. En la barra lateral de la aplicación, seleccione **Usuarios y grupos** > **Agregar usuario** > **Usuarios y grupos (\<Number> seleccionado)** . Aparece una lista de usuarios y grupos para su selección.

   ![Pantalla con la lista de usuarios y grupos](./media/application-proxy-configure-single-sign-on-with-ping-access/users-and-groups.png)

1. Seleccione un usuario para probar la aplicación y seleccione **Seleccionar**. Asegúrese de que esta cuenta de prueba tiene acceso a la aplicación local.
1. Seleccione **Asignar**.
1. En la barra lateral de la aplicación, seleccione **Inicio de sesión único** > **Basado en encabezado**.

   > [!TIP]
   > Si se trata de la primera vez que usa el inicio de sesión único basado en el encabezado, debe instalar PingAccess. Para asegurarse de que su suscripción de Azure se asocia automáticamente con la instalación de PingAccess, utilice el vínculo de esta página de inicio de sesión único para descargar PingAccess. Puede abrir el sitio de descarga ahora o volver a esta página más adelante.

   ![Pantalla de inicio de sesión basado en encabezado y PingAccess](./media/application-proxy-configure-single-sign-on-with-ping-access/sso-header.png)

1. Seleccione **Guardar**.

Luego, asegúrese de que la dirección URL de redireccionamiento está establecida en su dirección URL externa:

1. En la barra lateral del **Centro de administración de Azure Active Directory**, seleccione **Azure Active Directory** > **Registros de aplicaciones**. Aparece una lista de aplicaciones.
1. Seleccione la aplicación.
1. Seleccione el vínculo junto a **URI de redirección**, que muestra el número de URI de redirección configurados para clientes públicos y de web. Se muestra la página **\<application name>Autenticación**.
1. Compruebe si la dirección URL externa que asignó a la aplicación anteriormente está incluida en la lista **URI de redirección**. Si no lo está, agregue la dirección URL externa ahora (con el tipo de URI de redirección **Web**) y seleccione **Guardar**.

Además de la dirección URL externa, se debe agregar a la lista de identificadores URI de redirección un punto de conexión de autorización de Azure Active Directory en la dirección URL externa.

`https://*.msappproxy.net/pa/oidc/cb`
`https://*.msappproxy.net/`

Por último, configure la aplicación local para que los usuarios tengan acceso de lectura y otras aplicaciones tengan acceso de lectura/escritura:

1. En la barra lateral **Registros de aplicaciones** de la aplicación, seleccione **Permisos de API** > **Agregar un permiso** > **API de Microsoft** > **Microsoft Graph**. Se abre la página **Solicitud de permisos de API** de **Microsoft Graph**, que contiene las API de Microsoft Azure Active Directory.

   ![Pantalla con la página Solicitud de permisos de API](./media/application-proxy-configure-single-sign-on-with-ping-access/required-permissions.png)

1. Seleccione **Permisos delegados** > **Usuario** > **User.Read**.
1. Seleccione **Permisos de la aplicación** > **Aplicación** > **Application.ReadWrite.All**.
1. Seleccione **Agregar permisos**.
1. En la página **Permisos de API**, seleccione **Conceder consentimiento de administrador para \<your directory name>** .

#### <a name="collect-information-for-the-pingaccess-steps"></a>Recopilación de información para los pasos de PingAccess

Para configurar la aplicación con PingAccess, es preciso recopilar estos tres tipos de información (todos GUID):

| Nombre del campo de Azure AD | Nombre del campo de PingAccess | Formato de datos |
| --- | --- | --- |
| **Id. de la aplicación (cliente)** | **Id. de cliente** | GUID |
| **Id. de directorio (inquilino)** | **Emisor** | GUID |
| `PingAccess key` | **Secreto de cliente** | Cadena aleatoria |

Para recopilar esta información:

1. En la barra lateral del **Centro de administración de Azure Active Directory**, seleccione **Azure Active Directory** > **Registros de aplicaciones**. Aparece una lista de aplicaciones.
1. Seleccione la aplicación. Se abre la página **Registros de aplicaciones** de la aplicación.

   ![Información general de registro de una aplicación](./media/application-proxy-configure-single-sign-on-with-ping-access/registration-overview-for-an-application.png)

1. Junto al valor de **Id. de la aplicación (cliente)** , seleccione el icono **Copiar en el Portapapeles**, cópielo y guárdelo. Especificaremos este valor más adelante como el identificador de cliente de PingAccess.
1. Junto al valor de **Id. de directorio (inquilino)** , seleccione también **Copiar en el Portapapeles**, cópielo y guárdelo. Especificaremos este valor más adelante como el emisor de PingAccess.
1. En la barra lateral de **Registros de aplicaciones** de la aplicación, seleccione **Certificados y secretos** > **Nuevo secreto de cliente**. Se abre la página **Agregar un secreto de cliente**.

   ![Pantalla con la página Agregar un secreto de cliente](./media/application-proxy-configure-single-sign-on-with-ping-access/add-a-client-secret.png)

1. En **Descripción**, escriba `PingAccess key`.
1. En **Expira**, seleccione cómo configurar la caducidad de la clave de PingAccess: **En 1 año**, **En 2 años** o **nunca**.
1. Seleccione **Agregar**. La clave de PingAccess aparece en la tabla de secretos de cliente, con una cadena aleatoria que se rellena automáticamente en el campo **VALOR**.
1. Junto al campo **VALOR** de la clave de PingAccess, seleccione el icono **Copiar en el Portapapeles**, cópielo y guárdelo. Especificaremos este valor más adelante como el secreto de cliente de PingAccess.

**Actualice el campo de `acceptMappedClaims`:**

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/) como administrador de la aplicación.
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones**. Aparece una lista de aplicaciones.
1. Seleccione la aplicación.
1. En la barra lateral de la página **Registros de aplicaciones** de la aplicación, seleccione **Manifiesto**. Aparece el código JSON del manifiesto correspondiente al registro de la aplicación.
1. Busque el campo `acceptMappedClaims` y cambie el valor a `True`.
1. Seleccione **Guardar**.

### <a name="use-of-optional-claims-optional"></a>Usar notificaciones opcionales (opcionales)

Las notificaciones opcionales permiten agregar notificaciones estándar (pero no incluidas de forma predeterminada) que cada usuario e inquilino tienen. Puede configurar notificaciones opcionales de la aplicación al modificar el manifiesto de esta. Para más información, vea el artículo explicativo [Manifiesto de aplicación de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest/).

Ejemplo de cómo incluir la dirección de correo electrónico en el access_token que PingAccess va a consumir:
```
    "optionalClaims": {
        "idToken": [],
        "accessToken": [
            {
                "name": "email",
                "source": null,
                "essential": false,
                "additionalProperties": []
            }
        ],
        "saml2Token": []
    },
```

### <a name="use-of-claims-mapping-policy-optional"></a>Uso de una directiva de asignación de notificaciones (opcional)

La [directiva de asignación de notificaciones (versión preliminar)](https://docs.microsoft.com/azure/active-directory/develop/active-directory-claims-mapping#claims-mapping-policy-properties) está pensada para los atributos que no existen en Azure AD. La asignación de notificaciones permite migrar a la nube aplicaciones locales antiguas, agregando para ello notificaciones personalizadas respaldadas por los objetos de usuario o AD FS.

Para que la aplicación use una notificación personalizada e incluya campos adicionales, asegúrese de que también ha [creado una directiva de asignación de notificaciones personalizadas y la ha asignado a la aplicación](../develop/active-directory-claims-mapping.md#claims-mapping-policy-assignment).

> [!NOTE]
> Para usar una notificación personalizada, también debe tener una directiva personalizada definida y asignada a la aplicación. Esta directiva debe incluir todos los atributos personalizados necesarios.
>
> La asignación y definición de directivas se pueden realizar a través de PowerShell o Microsoft Graph. Si usa PowerShell, primero debe usar `New-AzureADPolicy` y, después, asignarlo a la aplicación con `Add-AzureADServicePrincipalPolicy`. Para más información, vea [Asignación de directivas de asignación de notificaciones](../develop/active-directory-claims-mapping.md#claims-mapping-policy-assignment).

Ejemplo:
```powershell
$pol = New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true", "ClaimsSchema": [{"Source":"user","ID":"employeeid","JwtClaimType":"employeeid"}]}}') -DisplayName "AdditionalClaims" -Type "ClaimsMappingPolicy"

Add-AzureADServicePrincipalPolicy -Id "<<The object Id of the Enterprise Application you published in the previous step, which requires this claim>>" -RefObjectId $pol.Id
```

### <a name="enable-pingaccess-to-use-custom-claims"></a>Permitir a PingAccess usar notificaciones personalizadas

Permitir que PingAccess use notificaciones personalizadas es opcional, si bien necesario si se espera que la aplicación consuma notificaciones adicionales.

Cuando configure PingAccess en el paso siguiente, la sesión web que cree en Settings->Access->Web Sessions (Configuración -> Acceso -> Sesiones web) debe tener la opción **Request Profile** (Solicitar perfil) sin seleccionar y la opción **Refresh User Attributes** (Actualizar atributos de usuario) establecida en **No**.

## <a name="download-pingaccess-and-configure-your-application"></a>Descarga de PingAccess y configuración de la aplicación

Ahora que ha completado todos los pasos de instalación de Azure Active Directory, puede pasar a configurar PingAccess.

Los pasos detallados de la parte de PingAccess de este escenario se pueden encontrar en la documentación de Ping Identity. Siga las instrucciones sobre cómo [configurar PingAccess para Azure AD para proteger las aplicaciones publicadas mediante Microsoft Azure AD Application Proxy](https://support.pingidentity.com/s/document-item?bundleId=pingaccess-52&topicId=agents/azure/pa_c_PAAzureSolutionOverview.html) en el sitio web de Ping Identity y descargue la [versión más reciente de PingAccess](https://www.pingidentity.com/en/lp/azure-download.html?).

Esos pasos le permiten instalar PingAccess y configurar una cuenta de PingAccess (si aún no tiene una). Luego, para crear una conexión de Azure AD OpenID Connect (OIDC), configure un proveedor de tokens con el valor de **Id. de directorio (inquilino)** que copió desde el portal de Azure AD. Tras ello, para crear una sesión web en PingAccess, utilice los valores de **Id. de la aplicación (cliente)** y `PingAccess key`. Después, puede configurar la asignación de identidades y crear un host virtual, un sitio y una aplicación.

### <a name="test-your-application"></a>Prueba de la aplicación

Cuando haya completado todos los pasos, la aplicación debería estar en funcionamiento. Para probarla, abra un explorador y navegue a la dirección URL externa que creó al publicar la aplicación en Azure. Inicie sesión con la cuenta de prueba que asignó a la aplicación.

## <a name="next-steps"></a>Pasos siguientes

- [Configurar PingAccess para Azure AD para proteger las aplicaciones publicadas mediante Microsoft Azure AD Application Proxy](https://support.pingidentity.com/s/document-item?bundleId=pingaccess-52&topicId=agents/azure/pa_c_PAAzureSolutionOverview.html)
- [Inicio de sesión único en aplicaciones en Azure Active Directory](what-is-single-sign-on.md)
- [Solución de problemas y mensajes de error de Proxy de aplicación](application-proxy-troubleshoot.md)
