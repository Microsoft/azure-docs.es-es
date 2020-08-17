---
title: 'Uso de restricciones de inquilino para administrar el acceso a las aplicaciones SaaS: Azure AD'
description: Cómo usar restricciones de inquilino para administrar los usuarios que pueden tener acceso a las aplicaciones según su inquilino de Azure AD.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 03/28/2019
ms.author: kenwith
ms.reviewer: hpsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0f45cc2444a14fc138d201e3d7f81e687f53d3ac
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285907"
---
# <a name="use-tenant-restrictions-to-manage-access-to-saas-cloud-applications"></a>Uso de restricciones de inquilino para administrar el acceso a aplicaciones en la nube SaaS

Las organizaciones grandes que hacen hincapié en la seguridad desean moverse a servicios en la nube como Office 365 pero deben saber que sus usuarios solo podrán acceder a recursos aprobados. Tradicionalmente, las empresas restringen los nombres de dominio o las direcciones IP cuando desean administrar el acceso. Este enfoque no sirve en un mundo donde las aplicaciones de software como servicio (o SaaS) se hospedan en una nube pública, ejecutándose en nombres de dominio compartidos como [outlook.office.com](https://outlook.office.com/) y [login.microsoftonline.com](https://login.microsoftonline.com/). El bloqueo de estas direcciones evitaría por completo que los usuarios accedieran a Outlook en la web, en lugar de simplemente limitarlos a identidades y recursos aprobados.

La solución de Azure Active Directory (Azure AD) para este desafío es una característica denominada restricciones de inquilino. Con las restricciones de inquilino, las organizaciones pueden controlar el acceso a aplicaciones en la nube SaaS según el inquilino de Azure AD que usan las aplicaciones para el inicio de sesión único. Por ejemplo, puede que le interese permitir el acceso a aplicaciones de Office 365 de su organización y, al mismo tiempo, impedir el acceso a instancias de otras organizaciones de estas mismas aplicaciones.  

Con las restricciones de inquilino, las organizaciones pueden especificar la lista de inquilinos a los que sus usuarios pueden tener acceso. Después, Azure AD solo concede el acceso a estos inquilinos con permiso.

Este artículo se centra en las restricciones de inquilino para Office 365, pero la característica debe funcionar con cualquier aplicación en la nube SaaS que use protocolos de autenticación moderna con Azure AD para inicio de sesión único. Si usa aplicaciones SaaS con un inquilino de Azure AD diferente al inquilino que usa Office 365, asegúrese de que todos los inquilinos necesarios tienen permiso. Para más información sobre aplicaciones en la nube SaaS, consulte [Active Directory Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory).

## <a name="how-it-works"></a>Funcionamiento

La solución general consta de los siguientes componentes:

1. **Azure AD**: si `Restrict-Access-To-Tenants: <permitted tenant list>` está presente, Azure AD solo emite tokens de seguridad para los inquilinos permitidos.

2. **Infraestructura de servidor proxy local**: Esta infraestructura es un dispositivo proxy compatible con la inspección de Seguridad de la capa de transporte (TLS). Debe configurar el proxy para insertar el encabezado que contiene la lista de inquilinos permitidos en el tráfico destinado a Azure AD.

3. **Software de cliente**: para admitir restricciones de inquilino, el software de cliente debe solicitar tokens directamente de Azure AD, de forma que la infraestructura del proxy pueda interceptar el tráfico. Actualmente, las aplicaciones de Office 365 basadas en explorador admiten las restricciones de inquilino, así como los clientes de Office que usan la autenticación moderna (como OAuth 2.0).

4. **Autenticación moderna**: los servicios en la nube deben usar la autenticación moderna para utilizar restricciones de inquilino y bloquear el acceso a todos los inquilinos no permitidos. Debe configurar los servicios en la nube de Office 365 para usar protocolos de autenticación moderna de manera predeterminada. Para ver la información más reciente sobre la compatibilidad con Office 365 para la autenticación moderna, lea [Updated Office 365 modern authentication](https://www.microsoft.com/en-us/microsoft-365/blog/2015/03/23/office-2013-modern-authentication-public-preview-announced/) (Autenticación moderna actualizada de Office 365).

En el siguiente diagrama se ilustra el flujo de tráfico de alto nivel. Las restricciones de inquilino solo requieren la inspección TLS en tráfico hacia Azure AD, no hacia los servicios en la nube de Office 365. Esta distinción es importante porque el volumen de tráfico para autenticación hacia Azure AD es normalmente mucho menor que el volumen de tráfico hacia aplicaciones SaaS como Exchange Online y SharePoint Online.

![Flujo de tráfico de restricciones de inquilino: diagrama](./media/tenant-restrictions/traffic-flow.png)

## <a name="set-up-tenant-restrictions"></a>Configuración de restricciones de inquilino

Se deben realizar dos pasos para empezar a trabajar con restricciones de inquilino. En primer lugar, debe asegurarse de que los clientes pueden conectarse a las direcciones correctas. En segundo lugar, configure la infraestructura del proxy.

### <a name="urls-and-ip-addresses"></a>Direcciones URL e IP reservadas

Para usar restricciones de inquilino, los clientes deben ser capaces de conectarse a las siguientes direcciones URL de Azure AD para autenticarse: [login.microsoftonline.com](https://login.microsoftonline.com/), [login.microsoft.com](https://login.microsoft.com/) y [login.windows.net](https://login.windows.net/). Además, para acceder a Office 365, los clientes también deben ser capaces de conectarse a los nombre de dominio completo (FQDN), las direcciones URL y las direcciones IP que se definen en [URL de Office 365 e intervalos de direcciones IP](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2). 

### <a name="proxy-configuration-and-requirements"></a>Requisitos y configuración de proxy

Se necesita la configuración siguiente para habilitar restricciones de inquilino a través de la infraestructura del proxy. Esta guía es genérica, por lo que debe remitirse a la documentación del proveedor del proxy para conocer los pasos de implementación específicos.

#### <a name="prerequisites"></a>Requisitos previos

- El proxy debe ser capaz de realizar la intercepción de TLS y la inserción de encabezados HTTP, así como de filtrar destinos mediante direcciones URL o FQDN.

- Los clientes deben confiar en la cadena de certificados que presenta el proxy para las comunicaciones TLS. Por ejemplo, si se usan certificados de una [infraestructura de clave pública (PKI)](/windows/desktop/seccertenroll/public-key-infrastructure) interna, el certificado de la entidad de certificación raíz emisora interna debe ser de confianza.

- Esta característica se incluye en las suscripciones de Office 365, pero si quiere usar restricciones de inquilino para controlar el acceso a otras aplicaciones SaaS, necesitará licencias de Azure AD Premium 1.

#### <a name="configuration"></a>Configuración

Para cada solicitud entrante de login.microsoftonline.com, login.microsoft.com y login.windows.net, inserte dos encabezados HTTP: *Restrict-Access-To-Tenants* y *Restrict-Access-Context*.

> [!NOTE]
> Al configurar la inyección de encabezado y la interceptación de SSL, asegúrese de que se excluya el tráfico a https://device.login.microsoftonline.com. Esta dirección URL se usa para la autenticación de dispositivos, y la realización de operaciones de interrupción e inspección de TLS puede interferir con la autenticación de certificados de cliente, lo que puede causar problemas con el registro de dispositivos y el acceso condicional basado en dispositivos.



Los encabezados deben incluir los siguientes elementos:

- Para *Restrict-Access-To-Tenants* (Restringir acceso a inquilinos), use un valor de \<permitted tenant list\>, que es una lista separada por comas de los inquilinos a los que quiere que los usuarios puedan tener acceso. Se puede utilizar cualquier dominio que esté registrado con un inquilino para identificar al inquilino en esta lista. Por ejemplo, para permitir el acceso a los inquilinos Contoso y Fabrikam, el par nombre-valor puede ser algo así como: `Restrict-Access-To-Tenants: contoso.onmicrosoft.com,fabrikam.onmicrosoft.com`

- Para *Restrict-Access-Context* (Contexto para restringir acceso), use un valor de un identificador de directorio único que declare qué inquilino está estableciendo restricciones de inquilino. Por ejemplo, para declarar Contoso como el inquilino que establece la directiva de restricciones de inquilino, el par nombre-valor puede ser algo así como: `Restrict-Access-Context: 456ff232-35l2-5h23-b3b3-3236w0826f3d`.  

> [!TIP]
> Puede encontrar el identificador de directorio en el [portal de Azure Active Directory](https://aad.portal.azure.com/). Inicie sesión como administrador, seleccione **Azure Active Directory** y luego seleccione **propiedades**.

Para evitar que los usuarios inserten su propio encabezado HTTP con inquilinos no aprobados, el proxy debe reemplazar el encabezado *Restrict-Access-To-Tenants* (Restringir acceso para inquilinos) si ya está presente en la solicitud entrante.

Se debe exigir a los clientes que usen el proxy para todas las solicitudes para login.microsoftonline.com, login.microsoft.com y login.windows.net. Por ejemplo, si los archivos PAC se emplean para indicar a los clientes que usen el proxy, los usuarios finales no deben poder editar ni deshabilitar los archivos PAC.

> [!NOTE]
> No incluya subdominios en *.login.microsoftonline.com en la configuración del proxy. Si lo hace, incluirá device.login.microsoftonline.com y puede interferir con la autenticación de certificados de cliente, que se usa en los escenarios de registro de dispositivos y acceso condicional basado en dispositivos. Configure el servidor proxy para que excluya device.login.microsoftonline.com de la inyección de encabezados y la interrupción e inspección de TLS.

## <a name="the-user-experience"></a>La experiencia del usuario final

En esta sección se describe la experiencia de los usuarios finales y los administradores.

### <a name="end-user-experience"></a>Experiencia del usuario final

Un usuario de ejemplo se encuentra en la red de Contoso, pero está intentando acceder a la instancia de Fabrikam de una aplicación SaaS compartida como Outlook en línea. Si Fabrikam es un inquilino no permitido para la instancia de Contoso, el usuario ve un mensaje de denegación de acceso, que indica que está intentando obtener acceso a un recurso que pertenece a una organización no aprobada por el departamento de TI.

### <a name="admin-experience"></a>Experiencia del administrador

Mientras la configuración de restricciones de inquilino se realice en la infraestructura del proxy corporativo, los administradores podrán tener acceso a los informes de restricciones de inquilino directamente en Azure Portal. Para ver los informes:

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/). Aparecerá el panel del **Centro de administración de Azure Active Directory**.

2. En el panel izquierdo, seleccione **Azure Active Directory**. Se mostrará la página de introducción de Azure Active Directory.

3. En la página Información general, seleccione **Restricciones de inquilino**.

El administrador del inquilino especificado como inquilino Restricted-Access-Context puede usar este informe para ver todos los inicios de sesión bloqueados debido a la directiva de restricciones de inquilino, incluida la identidad que se usa y el identificador de directorio de destino. Los inicios de sesión se incluyen si el inquilino que establece la restricción es el inquilino del usuario o el inquilino del recurso para el inicio de sesión.

> [!NOTE]
> El informe puede contener información limitada, como el identificador del directorio de destino, cuando un usuario que está en un inquilino distinto del inquilino Restricted-Access-Context inicia sesión. En este caso, la información de identificación del usuario, como el nombre y el nombre principal de usuario, se enmascara para proteger los datos de usuario en otros inquilinos.

Al igual que otros informes en Azure Portal, puede usar filtros para especificar el ámbito del informe. Puede filtrar por un usuario, una aplicación, un cliente, un estado o un intervalo de tiempo específico. Si selecciona el botón **Columnas**, puede elegir mostrar los datos con cualquier combinación de los siguientes campos:

- **User**
- **Aplicación**
- **Estado**
- **Date**
- **Fecha (UTC)** (donde UTC es la hora Universal coordinada)
- **Método de autenticación de MFA** (método de autenticación multifactor)
- **Detalles de la autenticación de MFA** (detalles de la autenticación multifactor)
- **Resultado de MFA**
- **Dirección IP**
- **Cliente**
- **Nombre de usuario**
- **Ubicación**
- **Identificador de inquilino de destino**

## <a name="office-365-support"></a>Compatibilidad con Office 365

Las aplicaciones de Office 365 deben cumplir dos criterios para que sean totalmente compatibles con restricciones de inquilino:

1. El cliente utilizado admite la autenticación moderna.
2. La autenticación moderna está habilitada como el protocolo de autenticación predeterminado para el servicio en la nube.

Consulte [Updated Office 365 modern authentication](https://www.microsoft.com/en-us/microsoft-365/blog/2015/03/23/office-2013-modern-authentication-public-preview-announced/) (Autenticación moderna actualizada de Office 365) para ver la información más reciente sobre qué clientes de Office admiten actualmente la autenticación moderna. Esa página también incluye vínculos a instrucciones para habilitar la autenticación moderna en inquilinos específicos de Exchange Online y Skype Empresarial Online. SharePoint Online ya habilita la autenticación moderna de manera predeterminada.

Actualmente, las aplicaciones de Office 365 basadas en explorador (portal de Office, Yammer, sitios de SharePoint, Outlook en la web, etc.) son compatibles con restricciones de inquilino. Los clientes pesados (Outlook, Skype Empresarial, Word, Excel, PowerPoint, etc.) pueden exigir restricciones de inquilino solo cuando se usa la autenticación moderna.  

Es posible que los clientes de Outlook y Skype Empresarial que admiten la autenticación moderna puedan seguir usando protocolos heredados en inquilinos donde la autenticación moderna no está habilitada, evitando eficazmente restricciones de inquilino. Es posible que las restricciones de inquilino bloqueen aplicaciones que usan protocolos heredados si entran en contacto con login.microsoftonline.com, login.microsoft.com o login.windows.net durante la autenticación.

Para Outlook en Windows, los clientes pueden optar por implementar restricciones que impidan a los usuarios finales agregar cuentas de correo no aprobadas a sus perfiles. Por ejemplo, vea el establecimiento de directiva de grupo [Prevent adding non-default Exchange accounts](https://gpsearch.azurewebsites.net/default.aspx?ref=1) (Impedir la incorporación de cuentas de Exchange no predeterminadas).

## <a name="testing"></a>Prueba

Si quiere probar la característica de restricciones de inquilino antes de implementarla para toda la organización, tiene dos opciones: un enfoque basado en host mediante una herramienta como Fiddler o un lanzamiento por fases de configuración de proxy.

### <a name="fiddler-for-a-host-based-approach"></a>Fiddler para un enfoque basado en host

Fiddler es un proxy de depuración web gratis que puede usarse para capturar y modificar el tráfico HTTP/HTTPS, incluida la inserción de encabezados HTTP. Para configurar Fiddler para probar las restricciones de inquilino, realice los pasos siguientes:

1. [Descargue e instale Fiddler](https://www.telerik.com/fiddler).

2. Configure Fiddler para descifrar el tráfico HTTPS siguiendo las indicaciones de la [documentación de la ayuda de Fiddler](https://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/DecryptHTTPS).

3. Configure Fiddler para insertar los encabezados *Restrict-Access-To-Tenants* y *Restrict-Access-Context* utilizando reglas personalizadas:

   1. En la herramienta Fiddler Web Debugger, seleccione el menú **Reglas** y luego **Personalizar reglas...** para abrir el archivo CustomRules.

   2. Agregue las líneas siguientes al principio de la función `OnBeforeRequest`. Reemplace \<tenant domain\> (dominio del inquilino) por un dominio registrado en el inquilino (por ejemplo, `contoso.onmicrosoft.com`). Reemplace \<directory ID\> por el identificador GUID de Azure AD del inquilino.

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

Para obtener detalles específicos, consulte la documentación del servidor proxy.

## <a name="next-steps"></a>Pasos siguientes

- Lea [Updated Office 365 modern authentication](https://www.microsoft.com/en-us/microsoft-365/blog/2015/03/23/office-2013-modern-authentication-public-preview-announced/) (Autenticación moderna actualizada de Office 365)
- Revise [URL de Office 365 e intervalos de direcciones IP](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)
