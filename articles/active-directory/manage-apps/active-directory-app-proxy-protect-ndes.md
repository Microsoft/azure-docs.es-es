---
title: Integración con AD Application Proxy en un servidor NDES
titleSuffix: Azure Active Directory
description: Instrucciones para implementar Azure Active Directory Application Proxy para proteger el servidor NDES.
services: active-directory
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/17/2020
ms.author: kenwith
ms.reviewer: mimart
ms.openlocfilehash: a385a339122197b7055ef6f54b8e37edea8eae4a
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88078944"
---
# <a name="integrate-with-azure-ad-application-proxy-on-a-network-device-enrollment-service-ndes-server"></a>Integración con Azure AD Application Proxy en un servidor de servicio de inscripción de dispositivos de red (NDES)

Azure Active Directory (AD) Application Proxy le permite publicar aplicaciones dentro de la red. Estas aplicaciones son, por ejemplo, sitios de SharePoint, aplicaciones web de Microsoft Outlook y otras aplicaciones web. También proporciona acceso seguro a los usuarios ajenos a la red a través de Azure.

Si no está familiarizado con Azure AD Application Proxy y quiere obtener más información, consulte [Acceso remoto a aplicaciones locales mediante Azure AD Application Proxy](application-proxy.md).

Azure AD Application Proxy forma parte de Azure. Proporciona una cantidad masiva de ancho de banda de red y de infraestructura de servidor para una mejor protección frente a ataques de denegación de servicio distribuidos (DDOS) y una disponibilidad óptima. Además, no es necesario abrir puertos del firewall externos en la red local y no se requiere ningún servidor de red perimetral. Todo el tráfico se origina en la entrada. Para obtener una lista completa de los puertos de salida, consulte [Tutorial: Adición de una aplicación local para el acceso remoto mediante Application Proxy en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-add-on-premises-application#prepare-your-on-premises-environment).

> Azure AD Application Proxy es una característica que solo está disponible si usa la edición Prémium o Básica de Azure Active Directory. Para más información, consulte los [precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/). 
> Si tiene licencias de Enterprise Mobility + Security (EMS), puede usar esta solución.
> El conector de Azure AD Application Proxy solo se instala en Windows Server 2012 R2 o sus versiones posteriores. Este también es un requisito del servidor NDES.

## <a name="install-and-register-the-connector-on-the-ndes-server"></a>Instalación y registro del conector en el servidor NDES

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de aplicaciones del directorio que usa el proxy de aplicación. Por ejemplo, si el dominio del inquilino es contoso.com, el administrador debe ser admin@contoso.com o cualquier otro alias de administrador de ese dominio.
1. En la esquina superior derecha, seleccione su nombre de usuario. Compruebe que ha iniciado sesión en el directorio que usa Application Proxy. Si necesita cambiar directorios, seleccione **Cambiar directorio** y elija un directorio que use Application Proxy.
1. En el panel de navegación izquierdo, seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Application proxy**.
1. Seleccione **Descargar servicio de conector**.

    ![Descargue el servicio de conector para ver los términos del servicio.](./media/active-directory-app-proxy-protect-ndes/application-proxy-download-connector-service.png)

1. Lea los términos del servicio. Cuando esté listo, seleccione **Aceptar las condiciones y descargar**.
1. Copie el archivo de instalación del conector de Azure AD Application Proxy en el servidor NDES. 
   > Puede instalar el conector en cualquier servidor de la red corporativa con acceso a NDES. No tiene que instalarlo en el propio servidor NDES.
1. Ejecute el archivo de instalación, como *AADApplicationProxyConnectorInstaller.exe*. Acepte los términos de licencia del software.
1. Durante la instalación, se le pedirá que registre el conector en Application Proxy en el directorio de Azure AD.
   * Proporcione las credenciales de un administrador global o de aplicaciones en el directorio de Azure AD. Las credenciales de administrador global o de aplicaciones de Azure AD pueden ser distintas de las credenciales de Azure en el portal.

        > [!NOTE]
        > La cuenta de administrador global o de aplicaciones que se usa para registrar el conector debe pertenecer al mismo directorio donde haya habilitado el servicio Application Proxy.
        >
        > Por ejemplo, si el dominio de Azure AD es *contoso.com*, el administrador global o de aplicaciones debe ser `admin@contoso.com` u otro alias válido en ese dominio.

   * Si la configuración de seguridad mejorada de Internet Explorer está activada en el servidor en el que instala el conector, la pantalla de registro podría bloquearse. Para permitir el acceso, siga las instrucciones del mensaje de error o desactive la seguridad mejorada de Internet Explorer durante el proceso de instalación.
   * Si el registro del conector no funciona, consulte [Solución de problemas de Proxy de aplicación](application-proxy-troubleshoot.md).
1. Al final de la instalación, se muestra una nota para los entornos con un proxy de salida. Para configurar el conector de Azure AD Application Proxy para que funcione a través del proxy de salida, ejecute el script proporcionado, como `C:\Program Files\Microsoft AAD App Proxy connector\ConfigureOutBoundProxy.ps1`.
1. En la página Proxy de aplicación de Azure Portal, el conector nuevo aparece con estado *Activo*, tal como se muestra en el ejemplo siguiente:

    ![El nuevo conector de Azure AD Application Proxy se muestra como activo en Azure Portal](./media/active-directory-app-proxy-protect-ndes/connected-app-proxy.png)

    > [!NOTE]
    > Para proporcionar alta disponibilidad a las aplicaciones que se autentican a través de Azure AD Application Proxy, puede instalar conectores en varias máquinas virtuales. Repita los mismos pasos indicados en la sección anterior para instalar el conector en otros servidores unidos al dominio administrado de Azure AD DS.

1. Después de una instalación correcta, vuelva a Azure Portal.

1. Seleccione **Aplicaciones empresariales**.

   ![asegúrese de que interactúa con las partes interesadas adecuadas](./media/active-directory-app-proxy-protect-ndes/azure-active-directory-enterprise-applications.png)

1. Seleccione **+Nueva aplicación** y, después, seleccione **Aplicación local**. 

1. En la **Agregar aplicación local propia**, configure los siguientes campos:

   * **Name**: Escriba un nombre para la aplicación.
   * **Dirección URL interna**: Escriba la dirección URL interna o el FQDN del servidor NDES en el que instaló el conector.
   * **Autenticación previa**: Seleccione **Acceso directo**. No es posible usar ninguna forma de autenticación previa. El protocolo usado para las solicitudes de certificado (SCEP) no proporciona esta opción.
   * Copie la **dirección URL externa** proporcionada en el portapapeles.

1. Seleccione **+Agregar** para guardar la aplicación.

1. Pruebe si puede acceder a su servidor NDES a través de Azure AD Application Proxy pegando el vínculo que copió en el paso 10 en un explorador. Debería ver una página principal predeterminada de IIS.

1. Como prueba final, agregue la ruta acceso de *mscep.dll* a la dirección URL existente que pegó en el paso anterior:

   https://scep-test93635307549127448334.msappproxy.net/certsrv/mscep/mscep.dll

1. Debería ver una respuesta **Error HTTP 403: prohibido**.

1. Cambie la dirección URL de NDES proporcionada (mediante Microsoft Intune) a los dispositivos. Este cambio puede aplicarse en Microsoft Endpoint Configuration Manager o en el centro de administración de Microsoft Endpoint Manager.

   * En Configuration Manager, acceda al punto de registro de certificados y modifique la dirección URL. Esta es la dirección URL a la que los dispositivos llaman y presentan su desafío.
   * En Intune independiente, edite o cree una nueva directiva de SCEP y añada la nueva dirección URL.

## <a name="next-steps"></a>Pasos siguientes

Con la instancia de Azure AD Application Proxy integrada con NDES, publique aplicaciones para que los usuarios tengan acceso a ellas. Para más información, consulte [Publicación de aplicaciones mediante Azure AD Application Proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-add-on-premises-application).
