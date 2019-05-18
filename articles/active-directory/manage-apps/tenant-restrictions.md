---
title: 'Uso de restricciones de inquilino para administrar el acceso a SaaS en la nube aplicaciones: Azure | Microsoft Docs'
description: Cómo usar restricciones de inquilino para administrar qué usuarios pueden acceder las aplicaciones basadas en su inquilino de Azure AD.
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
ms.date: 03/28/2019
ms.author: mimart
ms.reviewer: richagi
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4a340663a1ec4ddf748c6dc2bc3a4e2ce0c4228e
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824378"
---
# <a name="use-tenant-restrictions-to-manage-access-to-saas-cloud-applications"></a>Usar restricciones de inquilino para administrar el acceso a aplicaciones de SaaS en la nube

Las organizaciones grandes que hacen hincapié en la seguridad desean moverse a servicios en la nube como Office 365 pero deben saber que sus usuarios solo podrán acceder a recursos aprobados. Tradicionalmente, las empresas restringen los nombres de dominio o las direcciones IP cuando desean administrar el acceso. Este método produce un error en un mundo donde se hospedan aplicaciones de software como servicio (o SaaS) en una nube pública, que se ejecutan en los nombres de dominio compartidos como [outlook.office.com](https://outlook.office.com/) y [login.microsoftonline.com](https://login.microsoftonline.com/). El bloqueo de estas direcciones evitaría por completo que los usuarios accedieran a Outlook en la web, en lugar de simplemente limitarlos a identidades y recursos aprobados.

La solución de Azure Active Directory (Azure AD) para este desafío es una característica denominada restricciones de inquilino. Con las restricciones de inquilino, las organizaciones controlar el acceso a aplicaciones de SaaS en la nube, en función de las aplicaciones que se utilizan para el inicio de sesión único en el inquilino de Azure AD. Por ejemplo, puede que le interese permitir el acceso a aplicaciones de Office 365 de su organización y, al mismo tiempo, impedir el acceso a instancias de otras organizaciones de estas mismas aplicaciones.  

Con las restricciones de inquilino, las organizaciones pueden especificar la lista de inquilinos que pueden tener acceso a sus usuarios. Después, Azure AD solo concede el acceso a estos inquilinos con permiso.

En este artículo se centra en restricciones de inquilino de Office 365, pero la característica debe funcionar con cualquier aplicación de nube de SaaS que usa los protocolos de autenticación moderna con Azure AD para inicio de sesión único. Si usa aplicaciones SaaS con un inquilino de Azure AD diferente al inquilino que usa Office 365, asegúrese de que todos los inquilinos necesarios tienen permiso. Para más información sobre aplicaciones en la nube SaaS, consulte [Active Directory Marketplace](https://azure.microsoft.com/marketplace/active-directory/).

## <a name="how-it-works"></a>Funcionamiento

La solución general consta de los siguientes componentes:

1. **Azure AD**: Si el `Restrict-Access-To-Tenants: <permitted tenant list>` está presentes, Azure AD solo emite tokens de seguridad para los inquilinos permitidos.

2. **Infraestructura de servidor proxy local**: Esta infraestructura es un dispositivo proxy capaz de inspección de la capa de Sockets seguros (SSL). Debe configurar el proxy para insertar el encabezado que contiene la lista de inquilinos permitidos en el tráfico destinado a Azure AD.

3. **Software de cliente**: Para admitir restricciones de inquilino, el software cliente debe solicitar tokens directamente de Azure AD para que la infraestructura del proxy pueda interceptar el tráfico. Aplicaciones de Office 365 basadas en explorador actualmente admiten restricciones de inquilino, así como los clientes de Office que usan autenticación moderna (como OAuth 2.0).

4. **Autenticación moderna**: Servicios en la nube deben usar la autenticación moderna para utilizar restricciones de inquilino y bloquear el acceso a todos los inquilinos no permitidos. Debe configurar los servicios de Office 365 en la nube para usar protocolos de autenticación moderna de forma predeterminada. Para ver la información más reciente sobre la compatibilidad con Office 365 para la autenticación moderna, lea [Updated Office 365 modern authentication](https://www.microsoft.com/en-us/microsoft-365/blog/2015/03/23/office-2013-modern-authentication-public-preview-announced/) (Autenticación moderna actualizada de Office 365).

En el siguiente diagrama se ilustra el flujo de tráfico de alto nivel. Restricciones de inquilino que requiera la inspección SSL solo en el tráfico a Azure AD, no a los servicios de nube de Office 365. Esta distinción es importante, porque el volumen del tráfico para la autenticación a Azure AD es normalmente mucho menor que el volumen de tráfico a las aplicaciones de SaaS como Exchange Online y SharePoint Online.

![Flujo de tráfico de restricciones de inquilino: diagrama](./media/tenant-restrictions/traffic-flow.png)

## <a name="set-up-tenant-restrictions"></a>Configurar restricciones de inquilino

Hay dos pasos para empezar a trabajar con restricciones de inquilino. En primer lugar, asegúrese de que los clientes pueden conectarse a las direcciones correctas. En segundo lugar, configure la infraestructura del proxy.

### <a name="urls-and-ip-addresses"></a>Direcciones URL e IP reservadas

Para usar restricciones de inquilino, deben ser capaz de conectarse a las siguientes direcciones URL de Azure para autenticar los clientes: [login.microsoftonline.com](https://login.microsoftonline.com/), [login.microsoft.com](https://login.microsoft.com/), y [ Login.Windows.NET](https://login.windows.net/). Además, para acceder a Office 365, también deben ser capaz de conectarse a los nombres de dominio completo (FQDN), las direcciones URL, los clientes y direcciones IP definen en [intervalos de direcciones IP y direcciones URL de Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2). 

### <a name="proxy-configuration-and-requirements"></a>Requisitos y configuración de proxy

La siguiente configuración es necesaria para habilitar las restricciones de inquilino a través de la infraestructura del proxy. Esta guía es genérica, por lo que debe remitirse a la documentación del proveedor del proxy para conocer los pasos de implementación específicos.

#### <a name="prerequisites"></a>Requisitos previos

- El proxy debe ser capaz de realizar la intercepción de SSL y la inserción de encabezados HTTP, así como filtrar destinos mediante direcciones URL o FQDN.

- Los clientes deben confiar en la cadena de certificados que presenta el proxy para las comunicaciones SSL. Por ejemplo, si los certificados de una instancia interna [infraestructura de clave pública (PKI)](/windows/desktop/seccertenroll/public-key-infrastructure) son utilizado, el certificado de entidad de certificado de raíz emisora interna debe ser de confianza.

- Esta característica está incluida en suscripciones de Office 365, pero si desea usar restricciones de inquilino para controlar el acceso a otras aplicaciones SaaS, licencias de Azure AD Premium 1 son necesarias.

#### <a name="configuration"></a>Configuración

Para cada solicitud entrante de login.microsoftonline.com, login.microsoft.com y login.windows.net, inserte dos encabezados HTTP: *Restrict-Access-To-Tenants* y *Restrict-Access-Context*.

Los encabezados deben incluir los siguientes elementos:

- Para *Restrict-Access-To-Tenants*, use un valor de \<permite una lista de inquilino\>, que es una lista separada por comas de los inquilinos que desea permitir que los usuarios accedan a. Se puede utilizar cualquier dominio que esté registrado con un inquilino para identificar al inquilino en esta lista. Por ejemplo, para permitir el acceso a los inquilinos Contoso y Fabrikam, el par nombre-valor puede ser algo así como: `Restrict-Access-To-Tenants: contoso.onmicrosoft.com,fabrikam.onmicrosoft.com`

- Para *Restrict-Access-Context*, use un valor de Id. de un único directorio, declarar qué inquilino está estableciendo el inquilino de restricciones. Por ejemplo, para declarar Contoso como el inquilino que establece la directiva de restricciones de inquilino, el par nombre/valor aspecto: `Restrict-Access-Context: 456ff232-35l2-5h23-b3b3-3236w0826f3d`  

> [!TIP]
> Puede encontrar el identificador de directorio en el [portal de Azure Active Directory](https://aad.portal.azure.com/). Inicie sesión como administrador, seleccione **Azure Active Directory** y luego seleccione **propiedades**.

Para evitar que los usuarios inserten su propio encabezado HTTP con inquilinos no aprobados, el proxy debe reemplazar el *Restrict-Access-To-Tenants* encabezado si ya está presente en la solicitud entrante.

Se debe exigir a los clientes que usen el proxy para todas las solicitudes para login.microsoftonline.com, login.microsoft.com y login.windows.net. Por ejemplo, si se usan archivos PAC para dirigir a los clientes que usen al proxy, los usuarios finales no debería poder editar o deshabilitar los archivos PAC.

## <a name="the-user-experience"></a>La experiencia del usuario final

En esta sección se describe la experiencia para los usuarios finales y administradores.

### <a name="end-user-experience"></a>Experiencia del usuario final

Un usuario de ejemplo se encuentra en la red de Contoso, pero está intentando acceder a la instancia de Fabrikam de una aplicación SaaS compartida como Outlook en línea. Si Fabrikam es un inquilino no permitida para la instancia de Contoso, el usuario ve un mensaje de denegación de acceso, que dice que está intentando obtener acceso a un recurso que pertenece a una organización no aprobada por el departamento de TI.

### <a name="admin-experience"></a>Experiencia del administrador

Mientras se realiza la configuración de restricciones de inquilino en la infraestructura del proxy corporativo, los administradores pueden acceder a los informes de restricciones de inquilino en el portal de Azure directamente. Para ver los informes:

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/). El **centro de administración de Azure Active Directory** aparecerá el panel.

2. En el panel izquierdo, seleccione **Azure Active Directory**. Aparece la página de información general de Azure Active Directory.

3. En el **otras capacidades** encabezado, seleccione **restricciones de inquilino**.

Identificador de la administración del inquilino especificado como el inquilino Restricted-Access-Context puede usar este informe para ver los inicios de sesión bloqueados debido a la directiva de restricciones de inquilino, incluida la identidad utilizada y el directorio de destino. Los inicios de sesión se incluyen si el inquilino que establece la restricción es el inquilino del usuario o el inquilino del recurso para el inicio de sesión.

Al igual que otros informes en Azure Portal, puede usar filtros para especificar el ámbito del informe. Se puede filtrar por un intervalo de tiempo específico, usuario, aplicación, cliente o estado. Si selecciona el **columnas** botón, puede elegir mostrar los datos con cualquier combinación de los siguientes campos:

- **User**
- **Aplicación**
- **Estado**
- **Fecha**
- **Fecha (UTC)** (donde UTC es la hora Universal coordinada)
- **Método de autenticación de MFA** (método de autenticación multifactor)
- **Detalles de la autenticación MFA** (detalles de la autenticación multifactor)
- **Resultado de MFA**
- **Dirección IP**
- **Cliente**
- **Nombre de usuario**
- **Ubicación**
- **Id. de inquilino de destino**

## <a name="office-365-support"></a>Compatibilidad con Office 365

Las aplicaciones de Office 365 deben cumplir dos criterios para poder admitir restricciones de inquilino:

1. El cliente utilizado admite la autenticación moderna.
2. La autenticación moderna está habilitada como el protocolo de autenticación predeterminado para el servicio en la nube.

Consulte [Updated Office 365 modern authentication](https://www.microsoft.com/en-us/microsoft-365/blog/2015/03/23/office-2013-modern-authentication-public-preview-announced/) (Autenticación moderna actualizada de Office 365) para ver la información más reciente sobre qué clientes de Office admiten actualmente la autenticación moderna. Esa página también incluye vínculos a instrucciones para habilitar la autenticación moderna en inquilinos específicos de Exchange Online y Skype Empresarial Online. SharePoint Online ya habilita la autenticación moderna de forma predeterminada.

Las aplicaciones de Office 365 basadas en explorador (sitios de Portal de Office, Yammer, SharePoint, Outlook en la Web etc.) actualmente admite las restricciones de inquilino. Clientes gruesos (Outlook, Skype para Business, Word, Excel, PowerPoint etc.) pueden exigir restricciones de inquilino solo cuando se usa la autenticación moderna.  

Outlook y Skype para los clientes empresariales que admiten autenticación moderna pueden seguir usando protocolos heredados en inquilinos donde no está habilitada la autenticación moderna, evitando eficazmente restricciones de inquilino. Restricciones de inquilino pueden bloquear las aplicaciones que usan protocolos heredados si entran en contacto con login.microsoftonline.com, login.microsoft.com o login.windows.net durante la autenticación.

Para Outlook en Windows, los clientes pueden optar por implementar restricciones que impidan a los usuarios finales agregar cuentas de correo no aprobadas a sus perfiles. Por ejemplo, vea el establecimiento de directiva de grupo [Prevent adding non-default Exchange accounts](https://gpsearch.azurewebsites.net/default.aspx?ref=1) (Impedir la incorporación de cuentas de Exchange no predeterminadas).

## <a name="testing"></a>Pruebas

Si desea probar restricciones de inquilino antes de implementarla para toda la organización, tiene dos opciones: un enfoque basado en host mediante una herramienta como Fiddler o un lanzamiento por fases de configuración de proxy.

### <a name="fiddler-for-a-host-based-approach"></a>Fiddler para un enfoque basado en host

Fiddler es un proxy de depuración web gratis que puede usarse para capturar y modificar el tráfico HTTP/HTTPS, incluida la inserción de encabezados HTTP. Para configurar Fiddler para probar restricciones de inquilino, realice los pasos siguientes:

1. [Descargue e instale Fiddler](https://www.telerik.com/fiddler).

2. Configure Fiddler para descifrar el tráfico HTTPS siguiendo las indicaciones de la [documentación de la ayuda de Fiddler](https://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/DecryptHTTPS).

3. Configure Fiddler para insertar los encabezados *Restrict-Access-To-Tenants* y *Restrict-Access-Context* utilizando reglas personalizadas:

   1. En la herramienta Fiddler Web Debugger, seleccione el menú **Reglas** y luego **Personalizar reglas...** para abrir el archivo CustomRules.

   2. Agregue las siguientes líneas al principio de la `OnBeforeRequest` función. Reemplace \<dominio del inquilino\> con un dominio registrado con el inquilino (por ejemplo, `contoso.onmicrosoft.com`). Reemplace \<directory ID\> por el identificador GUID de Azure AD del inquilino.

      ```JScript.NET
      if (
          oSession.HostnameIs("login.microsoftonline.com") ||
          oSession.HostnameIs("login.microsoft.com") ||
          oSession.HostnameIs("login.windows.net")
      )
      {
          oSession.oRequest["Restrict-Access-To-Tenants"] = "<tenant domain>";
          oSession.oRequest["Restrict-Access-Context"] = "<directory ID>";
      }
      ```

      Si necesita permitir varios inquilinos, use una coma para separar los nombres de los mismos. Por ejemplo:

      `oSession.oRequest["Restrict-Access-To-Tenants"] = "contoso.onmicrosoft.com,fabrikam.onmicrosoft.com";`

4. Guarde y cierre el archivo CustomRules.

Después de configurar Fiddler, podrá capturar el tráfico yendo al menú **File** (Archivo) y seleccionando **Capture Traffic** (Capturar tráfico).

### <a name="staged-rollout-of-proxy-settings"></a>Lanzamiento por fases de configuración de proxy

Dependiendo de las funcionalidades de la infraestructura del proxy, es posible que pueda llevar a cabo el lanzamiento de la configuración a los usuarios. Aquí tiene un par de opciones de alto nivel a tener en cuenta:

1. Usar archivos PAC para dirigir a los usuarios de prueba a una infraestructura del proxy de prueba, mientras que los usuarios convencionales siguen usando la infraestructura del proxy de producción.
2. Algunos servidores proxy pueden admitir distintas configuraciones mediante grupos.

Para obtener información detallada, consulte la documentación del servidor proxy.

## <a name="next-steps"></a>Pasos siguientes

- Lea [Updated Office 365 modern authentication](https://www.microsoft.com/en-us/microsoft-365/blog/2015/03/23/office-2013-modern-authentication-public-preview-announced/) (Autenticación moderna actualizada de Office 365)
- Revise [URL de Office 365 e intervalos de direcciones IP](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)
