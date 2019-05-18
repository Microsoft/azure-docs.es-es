---
title: Habilitar el acceso remoto a SharePoint con el Proxy de aplicación de Azure AD | Microsoft Docs
description: Explica los conceptos básicos sobre cómo integrar un servidor de SharePoint local con el proxy de aplicación de Azure AD.
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
ms.date: 12/10/2018
ms.author: mimart
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9e491f0c452b7b51eac4e8cccab1cc7ed8430e49
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783443"
---
# <a name="enable-remote-access-to-sharepoint-with-azure-ad-application-proxy"></a>Habilitar el acceso remoto a SharePoint con el proxy de aplicación de Azure AD

En este artículo se describe cómo integrar un servidor de SharePoint local con el proxy de aplicación de Azure Active Directory (Azure AD).

Para habilitar el acceso remoto a SharePoint con el proxy de aplicación de Azure AD, siga los pasos de las secciones de este artículo.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por supuesto que ya tiene SharePoint 2013 o una versión más reciente instalada en su entorno. Además, tenga en cuenta los siguientes requisitos previos:

* SharePoint incluye compatibilidad nativa con Kerberos. Por tanto, los usuarios que tengan acceso a sitios internos de forma remota a través del proxy de aplicación de Azure AD pueden suponer que tienen a su disposición una experiencia de inicio de sesión único (SSO) sin interrupciones.

* Este escenario incluye cambios de configuración en el servidor de SharePoint. Se recomienda usar un entorno de ensayo. Así, podrá realizar actualizaciones en el servidor de ensayo en primer lugar y, después, facilitar un ciclo de pruebas antes de pasar a producción.

* Se necesita SSL en la dirección URL publicada. También se requiere SSL en la dirección URL interna para asegurarse de que los vínculos se envían o asignan correctamente.

## <a name="step-1-configure-kerberos-constrained-delegation-kcd"></a>Paso 1: Configuración de la delegación restringida de Kerberos (KCD)

Para las aplicaciones locales que usan la autenticación de Windows, puede realizar el inicio de sesión único (SSO) mediante el protocolo de autenticación Kerberos y una característica denominada delegación restringida de Kerberos (KCD). La KCD, cuando está configurada, permite que el conector del proxy de aplicación obtenga un token de Windows para un usuario, incluso si el usuario no inició sesión en Windows directamente. Para obtener más información acerca de KCD, vea [Introducción a la delegación limitada de Kerberos](https://technet.microsoft.com/library/jj553400.aspx).

Para configurar KCD para un servidor de SharePoint, use los procedimientos de las siguientes secciones secuenciales:

### <a name="ensure-that-sharepoint-web-application-is-running-under-a-domain-account"></a>Asegúrese de que la aplicación web de SharePoint se está ejecutando bajo una cuenta de dominio.

En primer lugar, asegúrese de que SharePoint se ejecuta en una cuenta de dominio, no de sistema local, servicio local o servicio de red. Para ello, es necesario que pueda asociar nombres de entidad de servicio (SPN) a esta cuenta. Los SPN son la forma que utiliza el protocolo de Kerberos para distinguir los diferentes servicios. Y necesitará la cuenta más adelante para configurar KCD.

> [!NOTE]
> Debe tener una cuenta de Azure AD previamente creada para el servicio. Se recomienda que permita un cambio de contraseña automático. Para más información sobre el conjunto completo de pasos y la solución de problemas, consulte [Configurar el cambio de contraseña automático en SharePoint Server](https://technet.microsoft.com/library/ff724280.aspx).

Para asegurarse de que los sitios se ejecutan en una cuenta de servicio definido, sita estos pasos:

1. Abra el sitio **Administración central de SharePoint**.
2. Vaya a **Seguridad** y seleccione **Configurar cuentas de servicio**.
3. Seleccione **Grupo de aplicaciones web - SharePoint - 80**. Las opciones pueden ser ligeramente diferentes según el nombre del grupo web o si el grupo web usa SSL de forma predeterminada.

   ![Opciones para configurar una cuenta de servicio](./media/application-proxy-integrate-with-sharepoint-server/service-web-application.png)

4. Si el campo **Seleccione una cuenta para este componente** está establecido en **Servicio Local** o **Servicio de red**, tendrá que crear una cuenta. Si no es así, ya habrá terminado y puede ir a la sección siguiente.
5. Haga clic en **Registrar una nueva cuenta administrada**. Una vez creada la cuenta, debe configurar el **Grupo de aplicaciones web** antes de poder usar la cuenta.

### <a name="set-a-service-principal-name-for-the-sharepoint-service-account"></a>Establecer un nombre de entidad de servicio para la cuenta de servicio de SharePoint

Antes de configurar KCD, es preciso:

* Identificar la cuenta de dominio que ejecuta la aplicación web de SharePoint que el proxy de Azure AD va a exponer.
* Elegir una dirección URL interna que se configurará tanto en el proxy de Azure AD como en SharePoint. Esta dirección URL interna no se debe usar en la aplicación web y nunca aparecerá en el explorador web.

Suponiendo que la dirección URL interna elegida es <https://sharepoint>, el SPN es:

```
HTTP/SharePoint
```

> [!NOTE]
> Respete las siguientes recomendaciones para la dirección URL interna:
> * Use HTTPS
> * No utilice puertos personalizados
> * En el DNS, cree un Host (A), seleccione WFE de SharePoint (o equilibrador de carga) y no a un Alias (CName)

Para registrar este SPN, ejecute el comando siguiente desde el símbolo del sistema como administrador del dominio:

```
setspn -S HTTP/SharePoint demo\spAppPoolAccount
```

Este comando establece el SPN _HTTP/SharePoint_ para la cuenta del grupo de aplicaciones de SharePoint _demo\spAppPoolAccount_.

Reemplace _HTTP/SharePoint_ con el SPN de su dirección URL interna y _demo\spAppPoolAccount_ con la cuenta del grupo de aplicaciones en su entorno. El comando Setspn busca el SPN antes de agregarlo. Si ya existe, verá el error **Valor SPN duplicado**. En este caso, considere la posibilidad de quitar el SPN existente, si no se establece en la cuenta del grupo de aplicaciones correcto.

Puede comprobar que el SPN se agregó mediante la ejecución del comando Setspn con la opción -l. Para más información sobre este comando, vea [Setspn](https://technet.microsoft.com/library/cc731241.aspx).

### <a name="ensure-that-the-connector-is-trusted-for-delegation-to-the-spn-added-to-the-sharepoint-application-pool-account"></a>Asegúrese de que el conector es de confianza para delegación en el SPN que se agrega a la cuenta del grupo de aplicaciones de SharePoint.

Configure KCD para que el servicio de proxy de aplicación de Azure AD pueda delegar las identidades de usuario en el servicio de SharePoint. Para configurar KCD, habilite el conector del proxy de aplicación para recuperar los vales Kerberos para los usuarios que se autenticaron en Azure AD. A continuación, ese servidor pasa el contexto a la aplicación de destino, o a SharePoint en este caso.

Para configurar KCD, repita los pasos siguientes para cada equipo de conexión:

1. Inicie sesión como administrador en un controlador de dominio y, después, abra **Usuarios y equipos de Active Directory**.
2. Busque el equipo en el que se ejecuta el conector. En este ejemplo, se trata del mismo servidor de SharePoint.
3. Haga doble clic en el equipo y, después, en la pestaña **Delegación**.
4. Asegúrese de que la configuración de delegación esté establecida en **Confiar en este equipo para la delegación solo a los servicios especificados**. Después, seleccione **Usar cualquier protocolo de autenticación**.
5. Haga clic en el botón **Agregar**, haga clic en **Usuarios o equipos** y busque la cuenta del grupo de aplicaciones de SharePoint, por ejemplo _demo\spAppPoolAccount_.
6. En la lista de los SPN, seleccione el que creó anteriormente para la cuenta de servicio.
7. Haga clic en **OK**. Haga clic en **Aceptar** de nuevo para guardar los cambios.
  
   ![Configuración de delegación](./media/application-proxy-integrate-with-sharepoint-server/delegation-box2.png)

## <a name="step-2-configure-azure-ad-proxy"></a>Paso 2: Configuración del proxy de Azure AD

Ahora que ha configurado KCD, está listo para configurar Azure AD Application Proxy.

1. Publique el sitio de SharePoint con la siguiente configuración. Para obtener instrucciones detalladas, consulte [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](application-proxy-publish-azure-portal.md).
   * **Dirección URL interna**: dirección URL interna de SharePoint que se ha elegido anteriormente, como **<https://SharePoint/>**.
   * **Método de autenticación previa**: Azure Active Directory
   * **Traducir URL en encabezados**: NO

   >[!TIP]
   >SharePoint usa el valor _Encabezado de host_ para buscar el sitio. También genera vínculos basándose en este valor. El efecto es que cualquier vínculo que genere SharePoint sea una dirección URL publicada que esté configurada correctamente para usar la dirección URL externa. Al establecer el valor en **SÍ** se permite también que el conector reenvíe la solicitud a la aplicación de back-end. Pero establecer el valor en **NO** significa que el conector no enviará el nombre de host interno. En su lugar, el conector envía el encabezado de host como la dirección URL publicada a la aplicación de back-end.

   ![Publicación de SharePoint como aplicación](./media/application-proxy-integrate-with-sharepoint-server/publish-app.png)

2. Una vez publicada la aplicación, establezca la configuración de inicio de sesión único según los pasos siguientes:

   1. En la página de la aplicación en el portal, seleccione **Inicio de sesión único**.
   2. Para el modo de inicio de sesión único, seleccione **Autenticación de Windows integrada**.
   3. Establezca el SPN de la aplicación interno en el valor establecido anteriormente. Para este ejemplo, sería **HTTP/SharePoint**.
   4. En "Identidad de inicio de sesión delegada", seleccione **Nombre de cuenta SAM local**.

   ![Configuración de la autenticación de Windows integrada para SSO](./media/application-proxy-integrate-with-sharepoint-server/configure-iwa.png)

3. Para finalizar la configuración de la aplicación, vaya a la sección **Usuarios y grupos** y asigne usuarios para que accedan a esta aplicación. 

## <a name="step-3-configure-sharepoint-to-use-kerberos-and-azure-ad-proxy-urls"></a>Paso 3: Configuración de SharePoint para usar direcciones URL de proxy de Azure AD y Kerberos

El siguiente paso es extender una aplicación web de SharePoint a una nueva zona, configurada con Kerberos y la asignación de acceso alternativa adecuada para permitir que SharePoint controle las solicitudes entrantes enviadas a la dirección URL interna, y responder con vínculos creados para la dirección URL externa.

1. Inicie el **shell de administración de SharePoint**.
2. Ejecute el siguiente script para ampliar la aplicación web a la zona Extranet y habilitar la autenticación Kerberos:

   ```powershell
   # Replace "http://spsites/" with the URL of your web application
   # Replace "https://sharepoint-f128.msappproxy.net/" with the External URL in your Azure AD proxy application
   $winAp = New-SPAuthenticationProvider -UseWindowsIntegratedAuthentication -DisableKerberos:$false
   Get-SPWebApplication "http://spsites/" | New-SPWebApplicationExtension -Name "SharePoint - AAD Proxy" -SecureSocketsLayer -Zone "Extranet" -Url "https://sharepoint-f128.msappproxy.net/" -AuthenticationProvider $winAp
   ```

3. Abra el sitio **Administración central de SharePoint**.
4. En **Configuración del sistema**, seleccione **Configurar asignaciones de acceso alternativas**. Se abre el cuadro Asignaciones de acceso alternativas.
5. Seleccione el sitio, por ejemplo **SharePoint - 80**. Por el momento, la zona Extranet aún no tiene la dirección URL interna establecida adecuadamente:

   ![Cuadro Asignaciones de acceso alternativas](./media/application-proxy-integrate-with-sharepoint-server/alternate-access1.png)

6. Haga clic en **Agregar dirección URL interna**.
7. En el cuadro de texto **Protocolo, host y puerto de dirección URL**, escriba la **Dirección URL interna** configurada en el proxy de Azure AD, por ejemplo, <https://SharePoint/>.
8. Seleccione Zona **Extranet** en la lista desplegable.
9. Haga clic en **Save**(Guardar).
10. Las asignaciones de acceso alternativas deberían ser ahora similares a la siguiente:

    ![Asignaciones de acceso alternativas correctas](./media/application-proxy-integrate-with-sharepoint-server/alternate-access3.png)

## <a name="step-4-ensure-that-an-https-certificate-is-configured-for-the-iis-site-of-the-extranet-zone"></a>Paso 4: Asegúrese de que hay configurado un certificado HTTPS para el sitio IIS de la zona Extranet.

La configuración de SharePoint se ha realizado ya, pero, dado que la dirección URL interna de la zona Extranet es <https://SharePoint/>, se debe establecer un certificado para este sitio.

1. Abra la consola de Windows PowerShell.
2. Ejecute el siguiente script para generar un certificado autofirmado y agregarlo al equipo Mi almacén:

   ```powershell
   # Replace "SharePoint" with the actual hostname of the Internal URL of your Azure AD proxy application
   New-SelfSignedCertificate -DnsName "SharePoint" -CertStoreLocation "cert:\LocalMachine\My"
   ```

   > [!NOTE]
   > Los certificados autofirmados son adecuados solo para pruebas. En los entornos de producción, se recomienda encarecidamente usar certificados emitidos por una entidad de certificación.

3. Abra la consola del Administrador de Internet Information Services.
4. Expanda el servidor en la vista de árbol, expanda "Sitios", seleccione el sitio "SharePoint – Proxy de AAD" y haga clic en **Enlaces**.
5. Seleccione el enlace HTTPS y haga clic en **Editar...** 
6. En el campo Certificado SSL, elija el certificado de **SharePoint** y haga clic en Aceptar.

Ahora puede tener acceso al sitio de SharePoint externamente mediante el proxy de aplicación de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

* [Uso de dominios personalizados en el proxy de la aplicación de Azure AD](application-proxy-configure-custom-domain.md)
* [Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md)
