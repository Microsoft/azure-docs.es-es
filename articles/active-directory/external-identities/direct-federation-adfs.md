---
title: 'Configuración de la federación directa con AD FS para B2B: Azure AD'
description: Aprenda a configurar AD FS como un proveedor de identidades para la federación directa para que los invitados puedan iniciar sesión en las aplicaciones de Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 07/01/2019
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1b3d7c47ff0a2c533bf12a67958a913b22915f75
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907734"
---
# <a name="example-direct-federation-with-active-directory-federation-services-ad-fs-preview"></a>Ejemplo: Federación directa con Servicios de federación de Active Directory (AD FS) (versión preliminar)

> [!NOTE]
> La federación directa es una característica en versión preliminar pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se describe cómo configurar la [federación directa](direct-federation.md) con los Servicios de federación de Active Directory (AD FS) como proveedor de identidades de SAML 2.0 o WS-Fed. Para admitir la federación directa, deben ser configurados ciertos atributos y notificaciones en el proveedor de identidades. Para ilustrar cómo configurar un proveedor de identidades para la federación directa, usaremos los Servicios de federación de Active Directory (AD FS) como ejemplo. Le mostraremos cómo configurar AD FS como proveedor de identidades de SAML y como un proveedor de identidades de WS-Fed.

> [!NOTE]
> En este artículo se describe cómo configurar AD FS para SAML y WS-Fed con fines meramente ilustrativos. Para las integraciones de federación directa en las que el proveedor de identidades es AD FS, se recomienda utilizar WS-Fed como protocolo. 

## <a name="configure-ad-fs-for-saml-20-direct-federation"></a>Configuración de AD FS para la federación directa de SAML 2.0
Azure AD B2B se puede configurar para federarse con proveedores de identidades que usan el protocolo SAML con los requisitos específicos que se indican a continuación. Para ilustrar los pasos de configuración de SAML, esta sección muestra cómo configurar AD FS para SAML 2.0. 

Para establecer una federación directa, se deben recibir los siguientes atributos en la respuesta de SAML 2.0 del proveedor de identidades. Estos atributos se pueden configurar mediante la vinculación con el archivo XML del servicio de token de seguridad en línea o la introducción manual. En el paso 12 de [Create a test AD FS instance](https://medium.com/in-the-weeds/create-a-test-active-directory-federation-services-3-0-instance-on-an-azure-virtual-machine-9071d978e8ed) (Creación de una instancia de AD FS de prueba) se describe cómo buscar los puntos de conexión de AD FS o generar la dirección URL de metadatos, por ejemplo `https://fs.iga.azure-test.net/federationmetadata/2007-06/federationmetadata.xml`. 

|Atributo  |Value  |
|---------|---------|
|AssertionConsumerService     |`https://login.microsoftonline.com/login.srf`         |
|Público     |`urn:federation:MicrosoftOnline`         |
|Emisor     |El URI del emisor del asociado IdP, por ejemplo`http://www.example.com/exk10l6w90DHM0yi...`         |

Las siguientes notificaciones deben configurarse en el token SAML 2.0 emitido por el proveedor de identidades:


|Atributo  |Value  |
|---------|---------|
|Formato de NameID     |`urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`         |
|emailaddress     |`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`         |


En la sección siguiente se muestra cómo configurar los atributos y las notificaciones necesarios mediante AD FS como un ejemplo de un proveedor de identidades de SAML 2.0.

### <a name="before-you-begin"></a>Antes de empezar

Antes de iniciar este procedimiento, se debe configurar y poner en funcionamiento el servidor AD FS. Para obtener ayuda sobre la configuración de un servidor AD FS, consulte [Create a test AD FS 3.0 instance on an Azure virtual machine](https://medium.com/in-the-weeds/create-a-test-active-directory-federation-services-3-0-instance-on-an-azure-virtual-machine-9071d978e8ed) (Creación de una instancia de prueba de AD FS 3.0 en una máquina virtual de Azure).

### <a name="add-the-claim-description"></a>Adición de la descripción de notificación

1. En el servidor de AD FS, seleccione **Herramientas** > **Administración de AD FS**.
2. En el panel de navegación, seleccione **Servicio** > **Descripciones de notificaciones**.
3. En **Acciones**, seleccione **Agregar descripción de notificación**.
4. En la ventana **Agregar descripción de notificación**, especifique los valores siguientes:

   - **Nombre para mostrar**: Identificador persistente
   - **Identificador de notificación**: `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent` 
   - Active la casilla **Publicar esta descripción de notificación en los metadatos de federación como un tipo de notificación que este Servicio de federación pueda aceptar**.
   - Active la casilla **Publicar esta descripción de notificación en los metadatos de federación como un tipo de notificación que este Servicio de federación pueda enviar**.

5. Haga clic en **Aceptar**.

### <a name="add-the-relying-party-trust-and-claim-rules"></a>Adición de las reglas de notificación y de la veracidad de usuarios de confianza

1. En el servidor de AD FS, vaya a **Herramientas** > **Administración de AD FS**.
2. En el panel de navegación, seleccione **Relaciones de confianza** > **Veracidades de usuarios de confianza**.
3. En **Acciones**, seleccione **Agregar veracidad del usuario de confianza**. 
4. En el Asistente para agregar veracidad del usuario de confianza para **Seleccionar origen de datos**, utilice la opción **Importar los datos sobre el usuario de confianza publicado en línea o en una red local**. Especifique la dirección URL de metadatos de federación https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml. Deje las otras selecciones predeterminadas. Seleccione **Cerrar**.
5. Se abre el asistente **Editar reglas de notificación**.
6. En el asistente **Editar reglas de notificación**, seleccione **Agregar regla**. En **Elegir tipo de regla**, seleccione **Enviar atributos LDAP como notificaciones**. Seleccione **Next** (Siguiente).
7. En **Configurar regla de notificación**, especifique los siguientes valores: 

   - **Nombre de la regla de notificación**: Regla de notificación de correo electrónico 
   - **Almacén de atributos**: Active Directory 
   - **Atributo LDAP** Direcciones de correo electrónico 
   - **Tipo de notificación saliente**: Dirección de correo electrónico

8. Seleccione **Finalizar**.
9. La ventana **Editar reglas de notificación** mostrará la nueva regla. Haga clic en **Aplicar**. 
10. Haga clic en **Aceptar**.  

### <a name="create-an-email-transform-rule"></a>Creación de una regla de transformación de correo electrónico
1. Vaya a **Editar reglas de notificación** y haga clic en **Agregar regla**. En **Elegir tipo de regla**, seleccione **Transformar una notificación entrante** y haga clic en **Siguiente**. 
2. En **Configurar regla de notificación**, especifique los siguientes valores: 

   - **Nombre de la regla de notificación**: Regla de transformación de correo electrónico 
   - **Tipo de notificación entrante**: Dirección de correo electrónico 
   - **Tipo de notificación saliente**: Identificador de nombre 
   - **Formato de id. de nombre saliente**: Identificador persistente 
   - Seleccione **Pasar a través todos los valores de notificaciones**.

3. Haga clic en **Finalizar** 
4. La ventana **Editar reglas de notificación** mostrará las nuevas reglas. Haga clic en **Aplicar**. 
5. Haga clic en **OK**. El servidor de AD FS ahora está configurado para la federación directa mediante el protocolo SAML 2.0.

## <a name="configure-ad-fs-for-ws-fed-direct-federation"></a>Configuración de AD FS para la federación directa de WS-Fed 
Azure AD B2B puede configurarse para federar con proveedores de identidades que usan el protocolo WS-Fed con los requisitos específicos que se enumeran a continuación. Actualmente, los dos proveedores de WS-Fed se han probado para determinar su compatibilidad con Azure AD e incluyen AD FS y Shibboleth. En este caso, utilizaremos los Servicios de federación de Active Directory (AD FS) como ejemplo del proveedor de identidades de WS-Fed. Para más información sobre cómo establecer la veracidad de un usuario de confianza entre un proveedor compatible con WS-Fed y Azure AD, descargue los documentos de compatibilidad del proveedor de identidades de Azure AD.

Para configurar una federación directa, se deben recibir los siguientes atributos en el mensaje de WS-Fed del proveedor de identidades. Estos atributos se pueden configurar mediante la vinculación con el archivo XML del servicio de token de seguridad en línea o la introducción manual. En el paso 12 de [Create a test AD FS instance](https://medium.com/in-the-weeds/create-a-test-active-directory-federation-services-3-0-instance-on-an-azure-virtual-machine-9071d978e8ed) (Creación de una instancia de AD FS de prueba) se describe cómo buscar los puntos de conexión de AD FS o generar la dirección URL de metadatos, por ejemplo `https://fs.iga.azure-test.net/federationmetadata/2007-06/federationmetadata.xml`.
 
|Atributo  |Value  |
|---------|---------|
|PassiveRequestorEndpoint     |`https://login.microsoftonline.com/login.srf`         |
|Público     |`urn:federation:MicrosoftOnline`         |
|Emisor     |El URI del emisor del asociado IdP, por ejemplo`http://www.example.com/exk10l6w90DHM0yi...`         |

Las notificaciones necesarias para el token de WS-Fed emitido por el proveedor de identidades:

|Atributo  |Value  |
|---------|---------|
|ImmutableID     |`http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID`         |
|emailaddress     |`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`         |

En la sección siguiente se muestra cómo configurar los atributos y las notificaciones necesarios mediante AD FS como un ejemplo de un proveedor de identidades de WS-Fed.

### <a name="before-you-begin"></a>Antes de empezar
Antes de iniciar este procedimiento, se debe configurar y poner en funcionamiento el servidor AD FS. Para obtener ayuda sobre la configuración de un servidor AD FS, consulte [Create a test AD FS 3.0 instance on an Azure virtual machine](https://medium.com/in-the-weeds/create-a-test-active-directory-federation-services-3-0-instance-on-an-azure-virtual-machine-9071d978e8ed) (Creación de una instancia de prueba de AD FS 3.0 en una máquina virtual de Azure).


### <a name="add-the-relying-party-trust-and-claim-rules"></a>Adición de las reglas de notificación y de la veracidad de usuarios de confianza 
1. En el servidor de AD FS, vaya a **Herramientas** > **Administración de AD FS**. 
1. En el panel de navegación, seleccione **Relaciones de confianza** > **Veracidades de usuarios de confianza**. 
1. En **Acciones**, seleccione **Agregar veracidad del usuario de confianza**.  
1. En el Asistente para agregar veracidad del usuario de confianza para **Seleccionar origen de datos**, utilice la opción **Importar los datos sobre el usuario de confianza publicado en línea o en una red local**. Especifique la dirección URL de metadatos de federación `https://nexus.microsoftonline-p.com/federationmetadata/2007-06/federationmetadata.xml`.  Deje las otras selecciones predeterminadas. Seleccione **Cerrar**.
1. Se abre el asistente **Editar reglas de notificación**. 
1. En el asistente **Editar reglas de notificación**, seleccione **Agregar regla**. En **Elegir tipo de regla**, seleccione **Enviar notificaciones con una regla personalizada**. Seleccione *Next* (Siguiente). 
1. En **Configurar regla de notificación**, especifique los siguientes valores:

   - **Nombre de la regla de notificación**: Identificador inmutable del problema  
   - **Regla personalizada**: `c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"] => issue(store = "Active Directory", types = ("http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"), query = "samAccountName={0};objectGUID;{1}", param = regexreplace(c.Value, "(?<domain>[^\\]+)\\(?<user>.+)", "${user}"), param = c.Value);`

1. Seleccione **Finalizar**. 
1. La ventana **Editar reglas de notificación** mostrará la nueva regla. Haga clic en **Aplicar**.  
1. En el mismo asistente **Editar reglas de notificación**, seleccione **Agregar regla**. En **Elegir tipo de regla**, seleccione **Enviar atributos LDAP como notificaciones**. Seleccione **Next** (Siguiente).
1. En **Configurar regla de notificación**, especifique los siguientes valores: 

   - **Nombre de la regla de notificación**: Regla de notificación de correo electrónico  
   - **Almacén de atributos**: Active Directory  
   - **Atributo LDAP** Direcciones de correo electrónico  
   - **Tipo de notificación saliente**: Dirección de correo electrónico 

1.  Seleccione **Finalizar**. 
1.  La ventana **Editar reglas de notificación** mostrará la nueva regla. Haga clic en **Aplicar**.  
1.  Haga clic en **OK**. El servidor de AD FS ahora está configurado para la federación directa mediante WS-Fed.

## <a name="next-steps"></a>Pasos siguientes
A continuación, va a [configurar la federación directa en Azure AD](direct-federation.md#step-2-configure-direct-federation-in-azure-ad) bien en el portal de Azure AD o con PowerShell. 
