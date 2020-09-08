---
title: Protección del acceso remoto a máquinas virtuales en Azure AD Domain Services | Microsoft Docs
description: Aprenda a proteger el acceso remoto a máquinas virtuales mediante el Servidor de directivas de redes (NPS) y Azure Multi-Factor Authentication con una implementación de Servicios de Escritorio remoto en un dominio administrado de Azure Active Directory Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: 462e0ff8ef8baadf40ba81ed2ae137d8904fbf7a
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88722524"
---
# <a name="secure-remote-access-to-virtual-machines-in-azure-active-directory-domain-services"></a>Protección del acceso remoto a máquinas virtuales en Azure AD Domain Services

Para proteger el acceso remoto a máquinas virtuales (VM) que se ejecutan en un dominio administrado de Azure Active Directory Domain Services (Azure AD DS), puede usar Servicios de Escritorio remoto (RDS) y el Servidor de directivas de redes (NPS). Azure AD DS autentica a los usuarios a medida que solicitan acceso en el entorno de RDS. Para mejorar la seguridad, puede integrar Azure Multi-Factor Authentication para proporcionar una solicitud de autenticación adicional durante los eventos de inicio de sesión. Azure Multi-Factor Authentication usa una extensión para que NPS proporcione esta característica.

> [!IMPORTANT]
> La manera recomendada de conectarse de forma segura a las máquinas virtuales en un dominio administrado de Azure AD DS es usar Azure Bastion, un servicio PaaS totalmente administrado por la plataforma que se aprovisiona dentro de la red virtual. Un host bastión proporciona una conexión del Protocolo de escritorio remoto (RDP) segura e ininterrumpida a las máquinas virtuales directamente en Azure Portal a través de SSL. Cuando se conecta a través de un host bastión, las máquinas virtuales no necesitan una dirección IP pública ni es necesario usar grupos de seguridad de red para exponer el acceso a RDP en el puerto TCP 3389.
>
> Se recomienda encarecidamente que use Azure Bastion en todas las regiones donde sea compatible. En las regiones en las que Azure Bastion no está disponible, siga los pasos que se detallan en este artículo hasta que esté disponible. Tenga cuidado con la asignación de direcciones IP públicas a las máquinas virtuales unidas a Azure AD DS donde se permite todo el tráfico RDP entrante.
>
> Para más información, consulte [¿Qué es Azure Bastion?][bastion-overview].

En este artículo se muestra cómo configurar RDS en Azure AD DS y, opcionalmente, cómo usar la extensión NPS para Azure Multi-Factor Authentication.

![Introducción a los Servicios de Escritorio remoto (RDS)](./media/enable-network-policy-server/remote-desktop-services-overview.png)

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesita los siguientes recursos:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, [cree y configure un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Una subred de *cargas de trabajo* creada en la red virtual de Azure Active Directory Domain Services.
    * Si es necesario, [configure las redes virtuales para un dominio administrado de Azure Active Directory Domain Services][configure-azureadds-vnet].
* Una cuenta de usuario que sea miembro del grupo de *administradores de Azure AD DC* en el inquilino de Azure AD.

## <a name="deploy-and-configure-the-remote-desktop-environment"></a>Implementación y configuración del entorno de Escritorio remoto

Para empezar, cree un mínimo de dos máquinas virtuales de Azure que ejecuten Windows Server 2016 o Windows Server 2019. Para lograr redundancia y alta disponibilidad del entorno de Escritorio remoto (RD), puede agregar hosts adicionales más adelante y equilibrar la carga de estos.

Una implementación de RDS sugerida incluye las dos máquinas virtuales siguientes:

* *RDGVM01*: ejecuta el servidor del agente de conexión a Escritorio remoto, el servidor del acceso web a Escritorio remoto y el servidor de puerta de enlace de Escritorio remoto.
* *RDSHVM01*: ejecuta el servidor host de sesión de Escritorio remoto.

Asegúrese de que las máquinas virtuales se implementan en una subred de *cargas de trabajo* de la red virtual de Azure AD DS y, a continuación, una las máquinas virtuales al dominio administrado. Para más información, consulte cómo [crear una máquina virtual con Windows Server y unirla a un dominio administrado][tutorial-create-join-vm].

La implementación del entorno de Escritorio remoto contiene una serie de pasos. La guía de implementación de Escritorio remoto existente se puede usar sin ningún cambio específico para usarla en un dominio administrado:

1. Inicie sesión en las máquinas virtuales creadas para el entorno de Escritorio remoto con una cuenta que forme parte del grupo *Administradores del controlador de dominio de Azure AD*, como *contosoadmin*.
1. Para crear y configurar RDS, use la [guía de implementación del entorno de Escritorio remoto][deploy-remote-desktop] existente. Distribuya los componentes del servidor de Escritorio remoto en las máquinas virtuales de Azure, según sea necesario.
    * Específico de Azure AD DS: al configurar la licencia de Escritorio remoto, establézcala en **Por dispositivo**, no **Por usuario**, como se indica en la guía de implementación.
1. Si desea proporcionar acceso mediante un explorador web, [configure el cliente web de Escritorio remoto para los usuarios][rd-web-client].

Con Escritorio remoto implementado en el dominio administrado, puede administrar y usar el servicio como lo haría con un dominio de AD DS local.

## <a name="deploy-and-configure-nps-and-the-azure-mfa-nps-extension"></a>Implementación y configuración de la extensión NPS para Azure MFA

Si desea aumentar la seguridad de la experiencia de inicio de sesión del usuario, también puede integrar el entorno de Escritorio remoto con Azure Multi-Factor Authentication. Con esta configuración, los usuarios reciben una solicitud adicional durante el inicio de sesión para confirmar su identidad.

Para proporcionar esta capacidad, se instala un Servidor de directivas de redes (NPS) adicional en el entorno junto con la extensión NPS para Azure Multi-Factor Authentication. Esta extensión se integra con Azure AD para solicitar y devolver el estado de las solicitudes de la autenticación multifactor.

Los usuarios deben estar [registrados para usar Azure Multi-Factor Authentication][user-mfa-registration], lo que puede requerir licencias de Azure AD adicionales.

Para integrar Azure Multi-Factor Authentication en el entorno de Escritorio remoto de Azure AD DS, cree un servidor NPS e instale la extensión:

1. Cree una máquina virtual Windows Server 2016 o 2019 adicional, como *NPSVM01*, que esté conectada a una subred de *cargas de trabajo* en la red virtual de Azure AD DS. Una la máquina virtual al dominio administrado.
1. Inicie sesión en la máquina virtual con NPS con una cuenta que forme parte del grupo *Administradores del controlador de dominio de Azure AD*, como *contosoadmin*.
1. En **Administrador del servidor**, seleccione **Agregar roles y características** y, a continuación, instale el rol *Servicios de acceso y directivas de redes*.
1. Use el artículo paso a paso existente para [instalar y configurar la extensión NPS para Azure MFA][nps-extension].

Con el servidor NPS y la extensión NPS para Azure Multi-Factor Authentication instalada, complete la sección siguiente para configurarla para su uso con el entorno de Escritorio remoto.

## <a name="integrate-remote-desktop-gateway-and-azure-multi-factor-authentication"></a>Integración de la puerta de enlace de Escritorio remoto y Azure Multi-Factor Authentication

Para integrar la extensión NPS para Azure Multi-Factor Authentication, use el artículo paso a paso existente para [integrar la infraestructura de la puerta de enlace de Escritorio remoto con la extensión del Servidor de directivas de redes (NPS) y Azure AD][azure-mfa-nps-integration].

Se necesitan las siguientes opciones de configuración adicionales para la integración con un dominio administrado:

1. No [registre el servidor NPS en Active Directory][register-nps-ad]. Este paso produce un error en un dominio administrado.
1. En el [paso 4 de configuración de la directiva de red][create-nps-policy], active también la casilla **Omitir las propiedades de acceso telefónico de las cuentas de usuario**.
1. Si usa Windows Server 2019 para el servidor NPS y la extensión NPS para Azure Multi-Factor Authentication, ejecute el siguiente comando para actualizar el canal seguro para permitir que el servidor NPS se comunique correctamente:

    ```powershell
    sc sidtype IAS unrestricted
    ```

Ahora se solicitará a los usuarios un factor de autenticación adicional cuando inicien sesión, como un mensaje de texto o una solicitud en la aplicación Microsoft Authenticator.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo mejorar la resistencia de la implementación, vea [Servicios de Escritorio remoto: alta disponibilidad][rds-high-availability].

Para más información sobre cómo proteger el inicio de sesión de usuario, vea [Funcionamiento: Azure Multi-Factor Authentication][concepts-mfa].

<!-- INTERNAL LINKS -->
[bastion-overview]: ../bastion/bastion-overview.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[configure-azureadds-vnet]: tutorial-configure-networking.md
[tutorial-create-join-vm]: join-windows-vm.md
[user-mfa-registration]: ../active-directory/authentication/howto-mfa-nps-extension.md#register-users-for-mfa
[nps-extension]: ../active-directory/authentication/howto-mfa-nps-extension.md
[azure-mfa-nps-integration]: ../active-directory/authentication/howto-mfa-nps-extension-rdg.md
[register-nps-ad]:../active-directory/authentication/howto-mfa-nps-extension-rdg.md#register-server-in-active-directory
[create-nps-policy]: ../active-directory/authentication/howto-mfa-nps-extension-rdg.md#configure-network-policy
[concepts-mfa]: ../active-directory/authentication/concept-mfa-howitworks.md

<!-- EXTERNAL LINKS -->
[deploy-remote-desktop]: /windows-server/remote/remote-desktop-services/rds-deploy-infrastructure
[rd-web-client]: /windows-server/remote/remote-desktop-services/clients/remote-desktop-web-client-admin
[rds-high-availability]: /windows-server/remote/remote-desktop-services/rds-plan-high-availability