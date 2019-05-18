---
title: Publicación de Escritorio remoto con el proxy de aplicación de Azure AD | Microsoft Docs
description: Se explican los conceptos básicos acerca de los conectores del Proxy de aplicación de Azure AD.
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
ms.date: 06/27/2018
ms.author: mimart
ms.custom: it-pro
ms.reviewer: harshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2ecd55cbb3a8eefc150db731901458561d90f033
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783405"
---
# <a name="publish-remote-desktop-with-azure-ad-application-proxy"></a>Publicación de Escritorio Remoto con el Proxy de aplicación de Azure AD

El Servicio de Escritorio remoto y el proxy de aplicación de Azure AD funcionan conjuntamente para mejorar la productividad de los trabajadores que están lejos de la red corporativa. 

La audiencia objetivo para este artículo es:
- Clientes del proxy de aplicación actuales que desean ofrecer más aplicaciones a los usuarios finales mediante la publicación de aplicaciones locales a través de Servicios de Escritorio remoto.
- Clientes de Servicios de Escritorio remoto actuales que desean reducir la superficie expuesta a ataques de su implementación mediante el proxy de aplicación de Azure AD. Este escenario ofrece un conjunto limitado de controles de acceso condicional y verificación en dos pasos a RDS.

## <a name="how-application-proxy-fits-in-the-standard-rds-deployment"></a>Cómo se adapta el proxy de aplicación a la implementación de RDS estándar

Una implementación de RDS estándar incluye diversos servicios de rol de Escritorio remoto que se ejecutan en Windows Server. Al examinar la [arquitectura de Servicios de Escritorio remoto](https://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/desktop-hosting-logical-architecture), vemos que hay varias opciones de implementación. A diferencia de otras opciones de implementación de RDS, la [implementación de RDS con el proxy de aplicación de Azure AD](https://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/desktop-hosting-logical-architecture) (mostrada en el siguiente diagrama) tiene una conexión de salida permanente desde el servidor que ejecuta el servicio del conector. Otras implementaciones dejan conexiones entrantes abiertas a través de un equilibrador de carga.

![El proxy de aplicación se sitúa entre la máquina virtual RDS y la red pública de Internet](./media/application-proxy-integrate-with-remote-desktop-services/rds-with-app-proxy.png)

En una implementación de RDS, el rol web de Escritorio remoto y el rol Puerta de enlace de Escritorio remoto se ejecutan en máquinas accesibles desde Internet. Estos puntos de conexión se exponen por las siguientes razones:
- Acceso web de Escritorio remoto ofrece al usuario un punto de conexión público para iniciar sesión y ver los diversos escritorios y aplicaciones locales a los que puede tener acceso. Al seleccionar un recurso, se crea una conexión RDP mediante la aplicación nativa en el SO.
- Puerta de enlace de Escritorio remoto aparece en escena una vez que un usuario inicia la conexión RDP. Puerta de enlace de Escritorio remoto controla el tráfico RDP que llega a través de Internet y lo traduce al servidor local al que se conecta el usuario. En este escenario, el tráfico que recibe Puerta de enlace de Escritorio remoto procede del proxy de aplicación de Azure AD.

>[!TIP]
>Si no ha implementado nunca RDS o desea más información antes de empezar, obtenga información sobre cómo [implementar RDS sin problemas con Azure Resource Manager y Azure Marketplace](https://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/rds-in-azure).

## <a name="requirements"></a>Requisitos

- Use un cliente que no sea el cliente web de Escritorio remoto, ya que el cliente web no es compatible con el proxy de aplicación.

- Los puntos de conexión de Acceso web y Puerta de enlace de Escritorio remoto deben estar en la misma máquina y compartir una raíz. Acceso de Escritorio remoto y Puerta de enlace de Escritorio remoto se publican como una sola aplicación con el proxy de aplicación para que pueda tener experiencia de inicio de sesión único entre las dos aplicaciones.

- Ya debe tener [RDS implementados](https://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/rds-in-azure) y el [proxy de aplicación habilitado](application-proxy-add-on-premises-application.md).

- En este escenario se da por hecho que los usuarios finales acceden a través de Internet Explorer en escritorios de Windows 7 y Windows 10 que se conectan a través de la página Acceso web de Escritorio remoto. Si necesita compatibilidad con otros sistemas operativos, consulte [Compatibilidad con otras configuraciones de cliente](#support-for-other-client-configurations).

- Al publicar la web del escritorio remoto, se recomienda usar el mismo FQDN interno y externo. Si el FQDN interno y externo son diferentes, debe desactivar la traducción del encabezado de solicitud para evitar que el cliente reciba enlaces no válidos. 

- En Internet Explorer, habilite el complemento ActiveX de RDS.

## <a name="deploy-the-joint-rds-and-application-proxy-scenario"></a>Implementar el escenario conjunto de RDS y el proxy de aplicación

Una vez configurados RDS y el proxy de aplicación de Azure AD para su entorno, siga los pasos para combinar las dos soluciones. Estos pasos le guían a través de la publicación de los dos puntos de conexión de RDS accesibles desde la Web (Acceso web y Puerta de enlace de Escritorio remoto) como aplicaciones y de la posterior dirección del tráfico de RDS para que pase por el proxy de aplicación.

### <a name="publish-the-rd-host-endpoint"></a>Publicar el punto de conexión del host de RD

1. [Publique una nueva aplicación del proxy de aplicación](application-proxy-add-on-premises-application.md) con los valores siguientes:
   - Dirección URL interna: `https://\<rdhost\>.com/`, donde `\<rdhost\>` es la raíz común que comparten Acceso web y Puerta de enlace de Escritorio remoto.
   - Dirección URL externa: este campo se rellena automáticamente según el nombre de la aplicación, pero puede modificarlo. Los usuarios visitarán esta dirección URL cuando tengan acceso a RDS.
   - Método de autenticación previa: Azure Active Directory
   - Traducir URL en encabezados: No
2. Asigne usuarios a la aplicación publicada de RD. Asegúrese también de que todos tienen acceso a RDS.
3. Deje el método de inicio de sesión único de la aplicación como **Se desactivó el inicio de sesión único de Azure AD**. Se solicita a los usuarios que se autentiquen una vez en Azure AD y una vez en Acceso web de Escritorio remoto, pero tienen inicio de sesión único en Puerta de enlace de Escritorio remoto.
4. Vaya a **Azure Active Directory** > **Registros de aplicaciones** > *Su aplicación* > **Configuración**.
5. Seleccione **Propiedades** y actualice el campo **Dirección URL de la página principal** para que apunte al punto de conexión de Acceso web de Escritorio remoto (como `https://\<rdhost\>.com/RDWeb`).

### <a name="direct-rds-traffic-to-application-proxy"></a>Dirigir el tráfico RDS al proxy de aplicación

Conéctese a la implementación de RDS como administrador y cambie el nombre del servidor de Puerta de enlace de Escritorio remoto para la implementación. Esta configuración garantiza que las conexiones pasen por el servicio del proxy de aplicación de Azure AD.

1. Conéctese al servidor RDS que ejecuta el rol Agente de conexión a Escritorio remoto.
2. Inicie **Administrador del servidor**.
3. Seleccione **Servicios de Escritorio remoto** en el panel de la izquierda.
4. Seleccione **Información general**.
5. En la sección Descripción general de la implementación, seleccione el menú desplegable y elija **Editar propiedades de implementación**.
6. En la pestaña Puerta de enlace de Escritorio remoto, cambie el campo **Nombre del servidor** por la dirección URL externa que ha establecido para el punto de conexión del host de RD en el proxy de aplicación.
7. Cambie el campo **Método de inicio de sesión** por **Autenticación de contraseña**.

   ![Pantalla Propiedades de implementación de RDS](./media/application-proxy-integrate-with-remote-desktop-services/rds-deployment-properties.png)

8. Ejecute este comando para cada colección. Reemplace *\<yourcollectionname\>* y *\<proxyfrontendurl\>* por su propia información. Este comando habilita el inicio de sesión único entre Acceso web de Escritorio remoto y Puerta de enlace de Escritorio remoto, además de optimizar el rendimiento:

   ```
   Set-RDSessionCollectionConfiguration -CollectionName "<yourcollectionname>" -CustomRdpProperty "pre-authentication server address:s:<proxyfrontendurl>`nrequire pre-authentication:i:1"
   ```

   **Por ejemplo:**
   ```
   Set-RDSessionCollectionConfiguration -CollectionName "QuickSessionCollection" -CustomRdpProperty "pre-authentication server address:s:https://remotedesktoptest-aadapdemo.msappproxy.net/`nrequire pre-authentication:i:1"
   ```
   >[!NOTE]
   >El comando anterior usa un acento grave en "`nrequire".

9. Para comprobar la modificación de las propiedades de RDP personalizadas, así como para ver el contenido del archivo RDP que se descargará desde RDWeb para esta colección, ejecute el siguiente comando:
    ```
    (get-wmiobject -Namespace root\cimv2\terminalservices -Class Win32_RDCentralPublishedRemoteDesktop).RDPFileContents
    ```

Ahora que ha configurado Escritorio remoto, el proxy de aplicación de Azure AD ha asumido el papel de componente accesible desde Internet de RDS. Puede quitar los otros puntos de conexión accesibles desde Internet públicos de sus máquinas Acceso web de Escritorio remoto y Puerta de enlace de Escritorio remoto.

## <a name="test-the-scenario"></a>Probar el escenario

Pruebe el escenario con Internet Explorer en un equipo con Windows 7 o Windows 10.

1. Vaya a la dirección URL externa que ha configurado o busque su aplicación en el [panel MyApps](https://myapps.microsoft.com).
2. Se le solicita que se autentique en Azure Active Directory. Use una cuenta que haya asignado a la aplicación.
3. Se le solicita que se autentique en Acceso web de Escritorio remoto.
4. Una vez que la autenticación RDS se realice correctamente, podrá seleccionar el escritorio o aplicación que desee y empezar a trabajar.

## <a name="support-for-other-client-configurations"></a>Compatibilidad con otras configuraciones de cliente

La configuración descrita en este artículo es para usuarios de Windows 7 o 10, con Internet Explorer y el complemento de ActiveX de RDS. No obstante, en caso necesario, también se ofrece compatibilidad con otros sistemas operativos o exploradores. La diferencia estriba en el método de autenticación que utilice.

| Método de autenticación | Configuración de cliente compatible |
| --------------------- | ------------------------------ |
| Autenticación previa    | Windows 7/10 con Internet Explorer + complemento ActiveX de RDS |
| Acceso directo | Cualquier otro sistema operativo compatible con la aplicación Escritorio remoto de Microsoft |

El flujo de autenticación previa ofrece más ventajas de seguridad que el flujo de acceso directo. Con la autenticación previa puede utilizar características de autenticación de Azure AD como el inicio de sesión único, el acceso condicional y la verificación en dos pasos para recursos locales. También garantiza que solo el tráfico autenticado alcance la red.

Para usar la autenticación de acceso directo, solo es necesario realizar dos modificaciones en los pasos indicados en este artículo:
1. En el paso 1 [Publicar el punto de conexión del host de RD](#publish-the-rd-host-endpoint), establezca el método de autenticación previa en **Acceso directo**.
2. Omita completamente el paso 8, [Dirigir el tráfico RDS al proxy de aplicación](#direct-rds-traffic-to-application-proxy).

## <a name="next-steps"></a>Pasos siguientes

[Habilitar el acceso remoto a SharePoint con el Proxy de aplicación de Azure AD](application-proxy-integrate-with-sharepoint-server.md)  
[Consideraciones de seguridad al obtener acceso a aplicaciones de forma remota con el proxy de aplicación de Azure AD](application-proxy-security.md)
