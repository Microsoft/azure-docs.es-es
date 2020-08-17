---
title: Comparación de servicios basados en Active Directory en Azure | Microsoft Docs
description: En esta introducción se comparan las distintas ofertas de identidad para Active Directory Domain Services, Azure Active Directory y Azure Active Directory Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: overview
ms.date: 06/08/2020
ms.author: iainfou
ms.openlocfilehash: 1ed1305939f4f8a9de62de85a2f09c323a2636b1
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489595"
---
# <a name="compare-self-managed-active-directory-domain-services-azure-active-directory-and-managed-azure-active-directory-domain-services"></a>Comparación de Active Directory Domain Services autoadministrado, Azure Active Directory y Azure Active Directory Domain Services administrado

Para proporcionar a las aplicaciones, los servicios o los dispositivos acceso a una identidad central, existen tres formas comunes de usar los servicios basados en Active Directory en Azure. Esta variedad de soluciones de identidad permite usar el directorio más adecuado para las necesidades de su organización. Por ejemplo, si administra principalmente usuarios solo en la nube que ejecutan dispositivos móviles, puede que no tenga sentido que compile y ejecute su propia solución de identidad de Active Directory Domain Services (AD DS). En su lugar, puede usar simplemente Azure Active Directory.

Aunque las tres soluciones de identidad basadas en Active Directory comparten un nombre y una tecnología comunes, se han diseñado para proporcionar servicios que satisfagan diferentes exigencias de los clientes. De forma general, estas son las soluciones de identidad y los conjuntos de características:

* **Active Directory Domain Services (AD DS)** : servidor LDAP (protocolo ligero de acceso a directorios) preparado para la empresa que proporciona características clave como la identidad y la autenticación, la administración de objetos de equipo, la directiva de grupo y confianzas.
    * AD DS es un componente central de muchas organizaciones con un entorno de TI local y proporciona características básicas de administración de equipos y autenticación de cuentas de usuario.
    * Para más información, consulte [Introducción a Active Directory Domain Services][overview-adds] en la documentación de Windows Server.
* **Azure Active Directory (Azure AD)** : administración de identidades basadas en la nube y de dispositivos móviles que proporciona servicios de cuentas de usuario y autenticación para recursos como Office 365, Azure Portal o las aplicaciones SaaS.
    * Azure AD se puede sincronizar con un entorno de AD DS local para proporcionar una identidad única a los usuarios que funcionan de forma nativa en la nube.
    * Para más información sobre Azure AD, consulte [¿Qué es Azure Active Directory?][whatis-azuread]
* **Azure Active Directory Domain Services (Azure AD DS)** : proporciona servicios de dominio administrados con un subconjunto de características de AD DS tradicionales totalmente compatibles como, por ejemplo, unión a un dominio, directiva de grupo, LDAP y autenticación Kerberos o NTLM.
    * Azure AD DS se integra con Azure AD, que se puede sincronizar con un entorno de AD DS local. Esta capacidad amplía los casos de uso de identidad central a las aplicaciones web tradicionales que se ejecutan en Azure como parte de una estrategia de migración mediante lift-and-shift.
    * Para más información sobre la sincronización con Azure AD y local, consulte [Procedimiento para sincronizar objetos y credenciales en un dominio administrado][synchronization].

En este artículo de introducción se compara cómo estas soluciones de identidad pueden actuar juntas o usarse de forma independiente, en función de las necesidades de su organización.

> [!div class="nextstepaction"]
> [Para empezar, crear un dominio administrado de Azure AD DS con Azure Portal][tutorial-create]

## <a name="azure-ad-ds-and-self-managed-ad-ds"></a>Azure AD DS y AD DS autoadministrado

Si tiene aplicaciones y servicios que necesitan acceder a mecanismos de autenticación tradicionales como Kerberos o NTLM, hay dos maneras de contar con Active Directory Domain Services en la nube:

* Un *dominio administrado* que se crea mediante Azure Active Directory Domain Services (Azure AD DS). Microsoft crea y administra los recursos necesarios.
* Un dominio *autoadministrado* que tiene que crear y configurar mediante recursos tradicionales como, por ejemplo, máquinas virtuales (VM), el sistema operativo invitado Windows Server y Active Directory Domain Services (AD DS). Después se encarga de seguir administrando estos recursos.

Con Azure AD DS, Microsoft implementa y mantiene los componentes de servicios básicos como una experiencia de dominio *administrado*. Usted no tiene que implementar, administrar, aplicar revisiones ni proteger la infraestructura de AD DS para componentes como las máquinas virtuales, el sistema operativo Windows Server o los controladores de dominio (DC).

Azure AD DS proporciona un subconjunto más pequeño de características para el entorno de AD DS autoadministrado tradicional, lo que reduce parte de la complejidad del diseño y la administración. Por ejemplo, no hay bosques, dominios, sitios ni vínculos de replicación de AD que diseñar y mantener. Todavía puede [crear confianzas de bosque entre Azure AD DS y los entornos locales][create-forest-trust].

En el caso de las aplicaciones y los servicios que se ejecutan en la nube y necesitan acceso a mecanismos de autenticación tradicionales como Kerberos o NTLM, Azure AD DS proporciona una experiencia de dominio administrado con la mínima cantidad posible de sobrecarga administrativa. Para más información, consulte [Conceptos de administración para cuentas de usuario, contraseñas y administración en Azure AD DS][administration-concepts].

Al implementar y ejecutar un entorno de AD DS autoadministrado, tiene que mantener todos los componentes de infraestructura y directorio asociados. Con un entorno de AD DS autoadministrado existe una sobrecarga de mantenimiento adicional, pero después podrá realizar tareas adicionales, como extender el esquema o crear confianzas de bosques.

Entre los modelos de implementación comunes para un entorno de AD DS autoadministrado que proporciona identidad a aplicaciones y servicios en la nube se encuentran los siguientes:

* **AD DS solo en la nube independiente**: las máquinas virtuales de Azure se configuran como controladores de dominio y se crea un entorno de AD DS solo en la nube independiente. Este entorno de AD DS no se integra con un entorno de AD DS local. Se usa un conjunto diferente de credenciales para iniciar sesión en las máquinas virtuales en la nube y administrarlas.
* **Implementación en un bosque de recursos**: las máquinas virtuales de Azure se configuran como controladores de dominio y se crea un dominio de AD DS como parte de un bosque existente. Después, se configura una relación de confianza con un entorno de AD DS local. Puede unir a un dominio otras máquinas virtuales de Azure en este bosque de recursos en la nube. La autenticación de usuarios se realiza a través de una conexión VPN o ExpressRoute con el entorno de AD DS local.
* **Ampliación del dominio local a Azure**: una red virtual de Azure se conecta a una red local mediante una conexión VPN o ExpressRoute. Las máquinas virtuales de Azure se conectan a esta red virtual de Azure, lo que les permite unirse a un dominio en el entorno de AD DS local.
    * También se pueden crear máquinas virtuales de Azure y promoverlas como controladores de dominio de réplica del dominio de AD DS local. Estos controladores de dominio se replican a través de una conexión VPN o ExpressRoute en el entorno de AD DS local. De este modo, el dominio de AD DS local se amplía de manera eficaz a Azure.

En la tabla siguiente se describen algunas de las características que puede necesitar en su organización, así como las diferencias entre un dominio de Azure AD DS administrado o un dominio de AD DS autoadministrado:

| **Característica** | **Azure AD DS** | **AD DS autoadministrado** |
| ----------- |:---------------:|:----------------------:|
| **Servicio administrado**                               | **&#x2713;** | **&#x2715;** |
| **Implementaciones seguras**                            | **&#x2713;** | El administrador protege la implementación |
| **Servidor DNS**                                    | **&#x2713;** (servicio administrado) |**&#x2713;** |
| **Privilegios de administrador de dominio o empresa** | **&#x2715;** | **&#x2713;** |
| **Unión a un dominio**                                   | **&#x2713;** | **&#x2713;** |
| **Autenticación de dominios mediante NTLM y Kerberos** | **&#x2713;** | **&#x2713;** |
| **Delegación limitada de Kerberos**               | Basada en recursos | Basada en recursos y basada en cuentas|
| **Estructura de unidad organizativa personalizada**                           | **&#x2713;** | **&#x2713;** |
| **Directiva de grupo**                                  | **&#x2713;** | **&#x2713;** |
| **Extensiones de esquema**                             | **&#x2715;** | **&#x2713;** |
| **Confianzas de bosques o dominios de AD**                     | **&#x2713;** (solo relaciones de confianza de bosque de salida unidireccional) | **&#x2713;** |
| **LDAP seguro (LDAPS)**                           | **&#x2713;** | **&#x2713;** |
| **Lectura LDAP**                                     | **&#x2713;** | **&#x2713;** |
| **Escritura LDAP**                                    | **&#x2713;** (dentro del dominio administrado) | **&#x2713;** |
| **Implementaciones distribuidas geográficamente**                   | **&#x2715;** | **&#x2713;** |

## <a name="azure-ad-ds-and-azure-ad"></a>Azure AD DS y Azure AD

Azure AD le permite administrar la identidad de los dispositivos usados por la organización y controlar el acceso a los recursos corporativos desde estos dispositivos. Los usuarios también pueden registrar su dispositivo personal (modelo BYO o Bring Your Own) con Azure AD, lo que proporcionará una identidad al dispositivo. A continuación, Azure AD autentica el dispositivo cuando un usuario inicia sesión en Azure AD y utiliza el dispositivo para acceder a los recursos protegidos. El dispositivo se puede controlar con software de administración de dispositivos móviles (MDM), como Microsoft Intune. Esta capacidad de administración permite restringir el acceso a recursos confidenciales desde dispositivos administrados que cumplen las directivas.

Los equipos y portátiles tradicionales también pueden unirse a Azure AD. Este mecanismo ofrece las mismas ventajas que el registro de un dispositivo personal con Azure AD, como permitir que los usuarios inicien sesión en el dispositivo con sus credenciales corporativas.

Los dispositivos unidos a Azure AD ofrecen las siguientes ventajas:

* Inicio de sesión único (SSO) en aplicaciones protegidas por Azure AD.
* Itinerancia de configuraciones de usuario que cumplen las directivas corporativas entre dispositivos.
* Acceso a Microsoft Store para Empresas con credenciales corporativas.
* Windows Hello para empresas.
* Acceso restringido a aplicaciones y recursos desde dispositivos que cumplen las directivas corporativas.

Los dispositivos pueden unirse a Azure AD con o sin una implementación híbrida que incluya un entorno de AD DS local. En la tabla siguiente se describen los modelos de propiedad de dispositivo habituales y cómo se suelen unir a un dominio:

| **Tipo de dispositivo**                                        | **Plataformas de dispositivo**             | **Mecanismo**          |
|:----------------------------------------------------------| -------------------------------- | ---------------------- |
| Dispositivos personales                                          | Windows 10, iOS, Android y macOS | Registrado en Azure AD    |
| Dispositivo propiedad de la organización no unido a AD DS local | Windows 10                       | Unido a Azure AD        |
| Dispositivo propiedad de la organización unido a AD DS local  | Windows 10                       | Híbrido unido a Azure AD |

En un dispositivo registrado en Azure AD o unido a Azure AD, la autenticación de usuario se realiza mediante modernos protocolos basados en OAuth u OpenID Connect. Estos protocolos están diseñados para funcionar a través de Internet, por lo que son excelentes en escenarios móviles donde los usuarios acceden a los recursos corporativos desde cualquier lugar.

Con los dispositivos unidos a Azure AD DS, las aplicaciones pueden usar los protocolos Kerberos y NTLM para la autenticación, por lo que se pueden admitir aplicaciones heredadas migradas para ejecutarse en máquinas virtuales de Azure como parte de una estrategia de migración mediante lift-and-shift. En la tabla siguiente se describen las diferencias en el modo en que se representan los dispositivos y cómo se pueden autenticar a sí mismos en el directorio:

| **Aspecto**                      | **Unido a Azure AD**                                 | **Unido a Azure AD DS**                                                    |
|:--------------------------------| --------------------------------------------------- | ------------------------------------------------------------------------- |
| Dispositivo controlado por            | Azure AD                                            | Dominio administrado de Azure AD DS                                                |
| Representación en el directorio | Objetos de dispositivo en el directorio de Azure AD            | Objetos de equipo en el dominio administrado de Azure AD DS                        |
| Authentication                  | Protocolos basados en OAuth y OpenID Connect              | Protocolos Kerberos y NTLM                                               |
| Administración                      | Software de administración de dispositivos móviles (MSM), como Intune | Directiva de grupo                                                              |
| Redes                      | Funciona a través de Internet                             | Debe estar conectado a la red virtual en la que se ha implementado el dominio administrado, o emparejarse con ella. |
| Idóneo para...                    | Dispositivos móviles o de escritorio de usuario final                  | Máquinas virtuales de servidor implementadas en Azure                                              |

## <a name="next-steps"></a>Pasos siguientes

Para empezar a usar Azure AD DS, [cree un dominio administrado de Azure AD DS con Azure Portal][tutorial-create].

También puede obtener más información acerca de los [conceptos de administración de cuentas de usuario, contraseñas y administración en Azure AD DS][administration-concepts] y el [proceso de sincronización de objetos y credenciales en un dominio administrado][synchronization].

<!-- INTERNAL LINKS -->
[manage-dns]: manage-dns.md
[deploy-kcd]: deploy-kcd.md
[custom-ou]: create-ou.md
[manage-gpos]: manage-group-policy.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-create]: tutorial-create-instance.md
[whatis-azuread]: ../active-directory/fundamentals/active-directory-whatis.md
[overview-adds]: /windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview
[create-forest-trust]: tutorial-create-forest-trust.md
[administration-concepts]: administration-concepts.md
[synchronization]: synchronization.md
