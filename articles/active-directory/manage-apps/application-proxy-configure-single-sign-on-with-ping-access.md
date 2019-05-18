---
title: Autenticación basada en encabezados con PingAccess para el proxy de la aplicación de Azure AD | Microsoft Docs
description: Publique aplicaciones con PingAccess y el proxy de la aplicación que admitan la autenticación basada en encabezados.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/11/2017
ms.author: mimart
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: ecba35a54c530e97a3f96bd8cfb5fa217d300f83
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783577"
---
# <a name="header-based-authentication-for-single-sign-on-with-application-proxy-and-pingaccess"></a>Autenticación basada en el encabezado para el inicio de sesión único con el proxy de aplicación y PingAccess

El proxy de la aplicación Azure Active Directory y PingAccess se han asociado para proporcionar a los clientes de Azure Active Directory acceso a más aplicaciones. PingAccess expande las [ofertas actuales del proxy de aplicación](application-proxy.md) para incluir el acceso de inicio sesión único a las aplicaciones que utilicen encabezados para la autenticación.

## <a name="what-is-pingaccess-for-azure-ad"></a>¿Qué es PingAccess para Azure AD?

PingAccess para Azure Active Directory es una oferta de PingAccess que permite conceder acceso a los usuarios y el inicio de sesión único a las aplicaciones que utilizan encabezados para la autenticación. El proxy de la aplicación trata estas aplicaciones como a las demás, usa Azure AD para autenticar el acceso y, después, pasa el tráfico a través del servicio de conector. PingAccess se coloca delante de las aplicaciones y convierte el token de acceso de Azure AD en un encabezado, con el fin de que la aplicación reciba la autenticación en un formato que pueda leer.

Los usuarios no notarán ninguna diferencia al iniciar sesión para usar las aplicaciones corporativas. Podrán seguir trabajando desde cualquier lugar y en cualquier dispositivo. 

Dado que los conectores del proxy de la aplicación dirigen el tráfico remoto a todas las aplicaciones, independientemente de su tipo de autenticación, seguirán equilibrando la carga de forma automática.

## <a name="how-do-i-get-access"></a>¿Cómo puedo obtener acceso?

Dado que este escenario se ofrece a través de una asociación entre Azure Active Directory y PingAccess, se necesitarán licencias de ambos servicios. Sin embargo, las suscripciones Premium de Azure Active Directory incluyen una licencia básica de PingAccess que abarca hasta 20 aplicaciones. Si tiene que publicar más de 20 aplicaciones basadas en el encabezado, puede adquirir una licencia adicional de PingAccess. 

Para obtener más información, consulte [Ediciones de Azure Active Directory](../fundamentals/active-directory-whatis.md).

## <a name="publish-your-application-in-azure"></a>Publicación de una aplicación en Azure

Este artículo va dirigido a quienes publican por primera vez una aplicación con este escenario. Es una guía a través de cómo empezar a trabajar con Application y PingAccess, además de explicar los pasos de la publicación. Si ya configuró ambos servicios, pero desea revisar los pasos de publicación, puede omitir la instalación del conector y pasar a [agregar la aplicación a Azure AD con el proxy de aplicación](#add-your-app-to-azure-ad-with-application-proxy).

>[!NOTE]
>Dado que este escenario es una asociación entre Azure AD y PingAccess, algunas de las instrucciones se encuentran en el sitio de Ping Identity.

### <a name="install-an-application-proxy-connector"></a>Instalación de un conector del proxy de la aplicación

Si ya tiene habilitado el proxy de aplicación y tiene instalado un conector, puede omitir esta sección y pasar a [agregar la aplicación a Azure AD con el proxy de aplicación](#add-your-app-to-azure-ad-with-application-proxy).

El conector del proxy de la aplicación es un servicio de Windows Server que dirige el tráfico de los empleados remotos a las aplicaciones publicadas. Para más información acerca de las instrucciones de instalación, consulte [Habilitación del proxy de aplicación en Azure Portal](application-proxy-add-on-premises-application.md).

1. Inicie sesión en el [portal Azure](https://portal.azure.com) como administrador de aplicaciones.
2. Seleccione **Azure Active Directory** > **Proxy de la aplicación**.
3. Seleccione **Download Connector** (Descargar conector) para iniciar la descarga de conector del proxy de la aplicación. Siga las instrucciones de instalación.

   ![Habilitar el proxy de la aplicación y descargar el conector](./media/application-proxy-configure-single-sign-on-with-ping-access/install-connector.png)

4. La descarga del conector debería habilitar automáticamente el proxy de la aplicación en el directorio, pero si no lo hace, puede seleccionar **Habilitar el proxy de la aplicación**.


### <a name="add-your-app-to-azure-ad-with-application-proxy"></a>Incorporación de la aplicación a Azure AD con el proxy de la aplicación

Hay dos acciones que se deben tomar en Azure Portal. En primer lugar, debe publicar la aplicación con el proxy de aplicación. Después, debe recopilar información acerca de la aplicación que puede usar en los pasos de PingAccess.

Siga estos pasos para publicar la aplicación. Para obtener un tutorial más detallado de los pasos 1 a 8, consulte [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](application-proxy-add-on-premises-application.md).

1. Si no lo hizo en la última sección, inicie sesión en el [portal Azure](https://portal.azure.com) como administrador de aplicaciones.
2. Seleccione **Azure Active Directory** > **Aplicaciones empresariales**.
3. Seleccione **Agregar** en la parte superior de la hoja.
4. Seleccione **Aplicación local**.
5. Rellene los campos requeridos con la información de la aplicación nueva. Utilice las siguientes instrucciones para realizar la configuración:
   - **Dirección URL interna**: Normalmente, especifica la dirección URL que lleva a la página de inicio de sesión la aplicación cuando esté en la red corporativa. En este escenario, el conector tiene que tratar el proxy de PingAccess como la página principal de la aplicación. Use este formato: `https://<host name of your PA server>:<port>`. El puerto predeterminado es 3000, pero puedo configurar uno distinto en PingAccess.

     > [!WARNING]
     > Para este tipo de inicio de sesión único, la dirección URL interna debe usar https y no puede emplear http.

   - **Método de autenticación previa**: Azure Active Directory
   - **Traducir URL en encabezados**: No

   >[!NOTE]
   >Si esta es su primera aplicación, utilice el puerto 3000 para empezar y regrese para actualizar este valor si cambia la configuración de PingAccess. Si no es la primera, tendrá que coincidir con la escucha que ha configurado en PingAccess. Obtenga más información sobre [escucha en PingAccess](https://documentation.pingidentity.com/pingaccess/pa31/index.shtml#Listeners.html).

6. Seleccione **Agregar** en la parte inferior de la hoja. Se agrega la aplicación y se abre el menú de inicio rápido.
7. En dicho menú, seleccione **Asignar un usuario para las pruebas**  y agregue al menos un usuario a la aplicación. Asegúrese de que esta cuenta de prueba tiene acceso a la aplicación local.
8. Seleccione **Asignar** para guardar la asignación del usuario de prueba.
9. En la hoja de administración de la aplicación, seleccione **Inicio de sesión único**.
10. Elija **Header-based sign-on** (Inicio de sesión basado en encabezado) en el menú desplegable. Seleccione **Guardar**.

    >[!TIP]
    >Si se trata de la primera vez que usa el inicio de sesión único basado en el encabezado, debe instalar PingAccess. Para asegurarse de que su suscripción de Azure se asocia automáticamente con la instalación de PingAccess, utilice el vínculo de esta página de inicio de sesión único para descargar PingAccess. Puede abrir el sitio de descarga ahora o volver a esta página más adelante. 

    ![Selección del inicio de sesión basado en encabezados](./media/application-proxy-configure-single-sign-on-with-ping-access/sso-header.PNG)

11. Cierre la hoja Aplicaciones empresariales o desplácese completamente hacia la izquierda para volver al menú de Azure Active Directory.
12. Seleccione **App registrations** (Registros de aplicaciones).

    ![Selección de los registros de aplicaciones](./media/application-proxy-configure-single-sign-on-with-ping-access/app-registrations.png)

13. Seleccione la aplicación que se acaba de agregar y seleccione **Direcciones URL de respuesta**.

    ![Selección de las direcciones URL de respuesta](./media/application-proxy-configure-single-sign-on-with-ping-access/reply-urls.png)

14. Compruebe si la dirección URL externa que asignó a la aplicación en el paso 5 se encuentra en la lista de direcciones URL de respuesta. Si no está, agréguela ahora.
15. En la hoja de configuración de la aplicación, seleccione **Permisos necesarios**.

    ![Selección de los permisos necesarios](./media/application-proxy-configure-single-sign-on-with-ping-access/required-permissions.png)

16. Seleccione **Agregar**. Para la API, elija **Microsoft Azure Active Directory** y, después, **Seleccionar**. Para los permisos, elija **Read and write all applications** (Leer y escribir en todas las aplicaciones) y **Sign in and read user profile** (Iniciar sesión y leer perfil de usuario), luego **Seleccionar** y, finalmente, **Listo**.  

    ![Seleccionar permisos](./media/application-proxy-configure-single-sign-on-with-ping-access/select-permissions.png)

17. Conceda los permisos antes de cerrar la pantalla de permisos. 
    ![Concesión de permisos](./media/application-proxy-configure-single-sign-on-with-ping-access/grantperms.png)

### <a name="collect-information-for-the-pingaccess-steps"></a>Recopilación de información para los pasos de PingAccess

1. En la hoja de configuración de la aplicación, seleccione **Propiedades**. 

   ![Selección de las propiedades](./media/application-proxy-configure-single-sign-on-with-ping-access/properties.png)

2. Guarde el valor de **Id. de la aplicación**, ya que se usa para el identificador de cliente cuando se configura PingAccess.
3. En la misma hoja, seleccione **Claves**.

   ![Selección de las claves](./media/application-proxy-configure-single-sign-on-with-ping-access/Keys.png)

4. Cree una clave, para lo que debe escribir una descripción de la misma y elegir una fecha de expiración en el menú desplegable.
5. Seleccione **Guardar**. Aparece un GUID en el campo **Valor**.

   Guarde este valor, ya que no lo volverá a ver una vez que cierre esta ventana.

   ![Creación de una nueva clave](./media/application-proxy-configure-single-sign-on-with-ping-access/create-keys.png)

6. Cierre la hoja Registros de aplicaciones o desplácese completamente hacia la izquierda para volver al menú de Azure Active Directory.
7. Seleccione **Propiedades**.
8. Guarde el GUID de **Id. de directorio**.

### <a name="optional---update-graphapi-to-send-custom-fields"></a>Opcional: actualice GraphAPI para enviar los campos personalizados

Para obtener una lista de los tokens de seguridad que Azure AD envía para la autenticación, consulte la [referencia de tokens de Azure AD](../develop/v1-id-and-access-tokens.md). Si necesita una notificación personalizada que envíe otros tokens, utilice Probador de Graph o el manifiesto para la aplicación en Azure Portal, para establecer el campo de aplicación *acceptMappedClaims* en **True**.    

En este ejemplo se usa el Explorador de Graph:

```
PATCH https://graph.windows.net/myorganization/applications/<object_id_GUID_of_your_application> 

{
  "acceptMappedClaims":true
}
```
En este ejemplo, se utiliza [Azure Portal](https://portal.azure.com) para actualizar el campo *acceptedMappedClaims*:
1. Inicie sesión en el [portal Azure](https://portal.azure.com) como administrador de aplicaciones.
2. Seleccione **Azure Active Directory** > **Registros de aplicaciones**.
3. Seleccione aplicación > **Manifiesto**.
4. Seleccione **Editar**, busque el campo *acceptedMappedClaims* y cambie el valor a **true**.
![Manifiesto de aplicación](./media/application-proxy-configure-single-sign-on-with-ping-access/application-proxy-ping-access-manifest.PNG)
1. Seleccione **Guardar**.

>[!NOTE]
>Para usar una notificación personalizada, también debe tener una directiva personalizada definida y asignada a la aplicación.  Esta directiva debe incluir todos los atributos personalizados necesarios.
>
>La asignación y definición de directivas se pueden realizar a través de PowerShell, el Explorador de Azure AD Graph o MS Graph.  Si hace esto en PowerShell, debe usar primero `New-AzureADPolicy` y, a continuación, asignarlo a la aplicación con `Set-AzureADServicePrincipalPolicy`.  Para más información, consulte la [documentación de Azure AD Policy](../develop/active-directory-claims-mapping.md#claims-mapping-policy-assignment).

### <a name="optional---use-a-custom-claim"></a>Opcional: uso de una notificación personalizada
Para que la aplicación use una notificación personalizada e incluya campos adicionales, asegúrese de que también ha [creado una directiva de asignación de notificaciones personalizadas y la ha asignado a la aplicación](../develop/active-directory-claims-mapping.md#claims-mapping-policy-assignment).

## <a name="download-pingaccess-and-configure-your-app"></a>Descarga de PingAccess y configuración de la aplicación

Ahora que ha completado todos los pasos de instalación de Azure Active Directory, puede pasar a configurar PingAccess. 

Los pasos detallados de la parte de PingAccess de este escenario se pueden encontrar en la documentación de Ping Identity, [Configure PingAccess for Azure AD](https://docs.pingidentity.com/bundle/paaad_m_ConfigurePAforMSAzureADSolution_paaad43/page/pa_c_PAAzureSolutionOverview.html) (Configuración de PingAccess para Azure AD).

Dichos pasos le guiarán a través del proceso de obtención de una cuenta de PingAccess, en caso de que aún no la tenga, instalación del servidor de PingAccess y creación de una conexión de proveedor de OIDC de Azure AD con el Id. de directorio que ha copiado de Azure Portal. A continuación, utilice los valores de Id. de la aplicación y Clave para crear una sesión Web en PingAccess. Después, puede configurar la asignación de identidades y crear un host virtual, un sitio y una aplicación.

### <a name="test-your-app"></a>Prueba de la aplicación

Cuando haya completado todos los pasos, la aplicación debería estar en funcionamiento. Para probarla, abra un explorador y navegue a la dirección URL externa que creó al publicar la aplicación en Azure. Inicie sesión con la cuenta de prueba que asignó a la aplicación.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de PingAccess para Azure AD](https://docs.pingidentity.com/bundle/paaad_m_ConfigurePAforMSAzureADSolution_paaad43/page/pa_c_PAAzureSolutionOverview.html)
- [¿Cómo permite el proxy de aplicación de Azure AD el inicio de sesión único?](application-proxy-single-sign-on.md)
- [Solucionar problemas de Proxy de aplicación](application-proxy-troubleshoot.md)
