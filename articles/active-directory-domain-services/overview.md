---
title: Información general de Azure Active Directory Domain Services | Microsoft Docs
description: En esta introducción, sabrá lo que ofrece Azure Active Directory Domain Services y cómo usarlo en su organización para ofrecer servicios de identidad a las aplicaciones y los servicios en la nube.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: overview
ms.date: 06/08/2020
ms.author: iainfou
ms.openlocfilehash: 6efd33b9f8825b5b5699b6106dadafec851ed454
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87488557"
---
# <a name="what-is-azure-active-directory-domain-services"></a>¿Qué es Azure Active Directory Domain Services?

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directivas de grupo, protocolo ligero de acceso a directorios (LDAP) y autenticación Kerberos o NTLM. Puede usar estos servicios de dominio sin necesidad de implementar o administrar los controladores de dominio de la nube, ni de aplicarles revisiones.

Cuando cree un dominio administrado de Azure AD DS, defina un espacio de nombres único. Este espacio de nombres es el nombre de dominio (por ejemplo, *aaddscontoso.com*). Luego, se implementan dos controladores de dominio en la región de Azure seleccionada. Esta implementación de controladores de dominio se conoce como "conjunto de réplicas". El dominio administrado se integra con el inquilino de Azure AD existente, lo que posibilita que los usuarios inicien sesión con sus credenciales existentes. También puede usar los grupos y las cuentas de usuario existentes para proteger el acceso a los recursos, lo que ofrece una mejor migración mediante lift-and-shift de los recursos en el entorno local a Azure.

Puede ampliar un dominio administrado para que tenga más de un conjunto de réplicas por cada inquilino de Azure AD. Los conjuntos de réplicas pueden agregarse a cualquier red virtual emparejada en cualquier región de Azure que admita Azure AD DS. Contar con conjuntos de réplicas adicionales en diferentes regiones de Azure facilita la recuperación ante desastres geográficos de las aplicaciones heredadas si una región de Azure se queda sin conexión. Actualmente, los conjuntos de réplicas están en versión preliminar. Para más información, consulte el artículo sobre los [conceptos y características de los conjuntos de réplicas en dominios administrados][concepts-replica-sets].

Azure AD DS se integra con el inquilino de Azure AD existente. Esta integración permite a los usuarios iniciar sesión en el servicio y las aplicaciones conectadas al dominio administrado con sus credenciales existentes. También puede usar grupos y cuentas de usuario existentes para proteger el acceso a los recursos. Estas características proporcionan una migración mediante lift-and-shift más fluida de los recursos locales a Azure.

> [!div class="nextstepaction"]
> [Para empezar, crear un dominio administrado de Azure AD DS con Azure Portal][tutorial-create]

Azure AD DS replica la información de identidad desde Azure AD, por lo que funciona con los inquilinos de Azure AD que solo están en la nube o se sincronizan con un entorno de Active Directory Domain Services (AD DS) local. El mismo conjunto de características de Azure AD DS existe para ambos entornos.

* Si tiene un entorno de AD DS local, puede sincronizar la información de las cuentas de usuario para proporcionar una identidad coherente para los usuarios. Para más información, consulte [Procedimiento para sincronizar objetos y credenciales en un dominio administrado][synchronization].
* En el caso de los entornos solo en la nube, no se necesita un entorno de AD DS local tradicional para usar los servicios de identidad centralizados de Azure AD DS.

Para obtener información sobre cómo se administraun dominio administrado, consulte [Conceptos de administración para cuentas de usuario, contraseñas y administración en Azure AD DS][administration-concepts].

El vídeo siguiente proporciona información general sobre el modo en que Azure AD DS se integra con las aplicaciones y las cargas de trabajo para proporcionar servicios de identidad en la nube:

<br />

>[!VIDEO https://www.youtube.com/embed/T1Nd9APNceQ]

## <a name="common-ways-to-provide-identity-solutions-in-the-cloud"></a>Formas comunes de proporcionar soluciones de identidad en la nube

Al migrar las cargas de trabajo existentes a la nube, las aplicaciones que tienen en cuenta el directorio pueden usar LDAP para el acceso de lectura o escritura a un directorio de AD DS local. Las aplicaciones que se ejecutan en Windows Server suelen implementarse en máquinas virtuales unidas a un dominio, de modo que se pueden administrar de forma segura mediante directivas de grupo. Para autenticar a los usuarios finales, las aplicaciones también pueden confiar en la autenticación integrada de Windows, como la autenticación Kerberos o NTLM.

A menudo, los administradores de TI usan una de las siguientes soluciones para proporcionar un servicio de identidad a las aplicaciones que se ejecutan en Azure:

* Configure una conexión VPN de sitio a sitio entre las cargas de trabajo que se ejecuten en Azure y en un entorno de AD DS local.
    * Los controladores de dominio locales proporcionan autenticación a través de la conexión VPN.
* Cree controladores de dominio de réplica con máquinas virtuales de Azure para extender el dominio o bosque de AD DS.
    * Los controladores de dominio que se ejecutan en máquinas virtuales de Azure proporcionan autenticación y replican información de directorio entre el entorno de AD DS local.
* Implemente un entorno de AD DS independiente en Azure mediante controladores de dominio que se ejecuten en máquinas virtuales Azure.
    * Los controladores de dominio que se ejecutan en máquinas virtuales de Azure proporcionan autenticación, pero no se replica la información de directorio desde un entorno de AD DS local.

Con estos métodos, las conexiones VPN con el directorio en el entorno local hacen que las aplicaciones sean vulnerables a interrupciones o problemas de red transitorios. Si implementa controladores de dominio mediante máquinas virtuales en Azure, el equipo de TI deben ocuparse de las tareas de administración, protección, revisión, supervisión, copia de seguridad y solución de problemas de las máquinas virtuales.

Azure AD DS ofrece alternativas a la necesidad de volver a crear las conexiones VPN en un entorno de AD DS local o ejecutar y administrar máquinas virtuales en Azure para proporcionar servicios de identidad. Como servicio administrado, Azure AD DS reduce la complejidad de crear una solución de identidad integrada para entornos tanto híbridos como solo en la nube.

> [!div class="nextstepaction"]
> [Comparar Azure AD DS con Azure AD y AD DS autoadministrado en VM de Azure o locales][compare]

## <a name="how-does-azure-ad-ds-work"></a>¿Cómo funciona Azure AD DS?

Para proporcionar servicios de identidad, Azure crea un dominio administrado AD DS en la red virtual que prefiera. En segundo plano, se crea un par de controladores de dominio de Windows Server que se ejecutan en máquinas virtuales de Azure. No es necesario administrar, configurar ni actualizar estos controladores de dominio. La plataforma de Azure administra los controladores de dominio como parte del servicio Azure AD DS.

El dominio administrado está configurado para realizar una sincronización unidireccional desde Azure AD y proporcionar acceso a un conjunto central de usuarios, grupos y credenciales. Puede crear los recursos directamente en el dominio administrado, pero no se vuelven a sincronizar con Azure AD. Las aplicaciones, los servicios y las máquinas virtuales de Azure que se conectan a esta red virtual pueden usar las características de AD DS comunes, como la unión a un dominio, la directiva de grupo, LDAP y la autenticación Kerberos/NTLM.

En un entorno híbrido con un entorno de AD DS local, [Azure AD Connect][azure-ad-connect] sincroniza la información de identidad con Azure AD, que se sincroniza entonces en Azure AD DS.

![Sincronización en Azure AD Domain Services con Azure AD y Active Directory Domain Services en el entorno local mediante AD Connect](./media/active-directory-domain-services-design-guide/sync-topology.png)

Para ver Azure AD DS en acción, echemos un vistazo a un par de ejemplos:

* [Azure AD DS para organizaciones híbridas](#azure-ad-ds-for-hybrid-organizations)
* [Azure AD DS para organizaciones solo en la nube](#azure-ad-ds-for-cloud-only-organizations)

### <a name="azure-ad-ds-for-hybrid-organizations"></a>Azure AD DS para organizaciones híbridas

Muchas organizaciones ejecutan una infraestructura híbrida que incluye cargas de trabajo de aplicaciones en el entorno local y en la nube. Las aplicaciones heredadas migradas a Azure como parte de una migración mediante lift-and-shift pueden usar las conexiones LDAP tradicionales para proporcionar información de identidad. Para admitir esta infraestructura híbrida, la información de identidad de un entorno de AD DS local se puede sincronizar con un inquilino de Azure AD. A continuación, Azure AD DS proporciona estas aplicaciones heredadas en Azure con un origen de identidad, sin necesidad de configurar y administrar la conectividad de la aplicación de nuevo en los servicios de directorio en el entorno local.

Echemos un vistazo a un ejemplo de Litware Corporation, una organización híbrida que ejecuta recursos locales y de Azure:

![Azure Active Directory Domain Services para una organización híbrida que incluye sincronización en el entorno local](./media/overview/synced-tenant.png)

* Las aplicaciones y cargas de trabajo de servidor que requieren servicios de dominio se implementan en una red virtual en Azure.
    * Puede que se incluyan aplicaciones heredadas migradas a Azure como parte de una estrategia de migración mediante lift-and-shift.
* Para sincronizar la información de identidad desde el directorio en el entorno local a su inquilino de Azure AD, Litware Corporation implementa [Azure AD Connect][azure-ad-connect].
    * La información de identidad que se sincroniza incluye las cuentas de usuario y las pertenencias a grupos.
* El equipo de TI de Litware habilita Azure AD DS para su inquilino de Azure AD en esta red virtual o en una emparejada.
* Las aplicaciones y las máquinas virtuales implementadas en la red virtual pueden usar características de Azure AD DS como, por ejemplo, la unión a un dominio, la lectura LDAP, el enlace LDAP, la autenticación NTLM y Kerberos, y directiva de grupo.

> [!IMPORTANT]
> Azure AD Connect solo debe instalarse y configurarse para la sincronización con entornos de AD DS locales. No se admite la instalación de Azure AD Connect en un dominio administrado para volver a sincronizar los objetos con Azure AD.

### <a name="azure-ad-ds-for-cloud-only-organizations"></a>Azure AD DS para organizaciones solo en la nube

Un inquilino de Azure AD solo en la nube no tiene un origen de identidad en el entorno local. Las cuentas de usuario y las pertenencias a grupos, por ejemplo, se crean y administran directamente en Azure AD.

Ahora veamos un ejemplo de Contoso, una organización solo en la nube que usa Azure AD para la identidad. Todas las identidades de usuario, sus credenciales y las pertenencias a grupos se crean y administran en Azure AD. No hay ninguna configuración adicional de Azure AD Connect para sincronizar cualquier información de identidad desde un directorio en el entorno local.

![Azure Active Directory Domain Services para una organización solo en la nube sin sincronización fuera del entorno local](./media/overview/cloud-only-tenant.png)

* Las aplicaciones y cargas de trabajo de servidor que requieren servicios de dominio se implementan en una red virtual en Azure.
* El equipo de TI de Litware Contoso habilita Azure AD DS para su inquilino de Azure AD en esta red virtual o en una emparejada.
* Las aplicaciones y las máquinas virtuales implementadas en la red virtual pueden usar características de Azure AD DS como, por ejemplo, la unión a un dominio, la lectura LDAP, el enlace LDAP, la autenticación NTLM y Kerberos, y directiva de grupo.

## <a name="azure-ad-ds-features-and-benefits"></a>Características y ventajas de Azure AD DS

Para proporcionar servicios de identidad a aplicaciones y máquinas virtuales en la nube, Azure AD DS es totalmente compatible con un entorno de AD DS tradicional para las operaciones como la unión a un dominio, LDAP seguro (LDAPS), directiva de grupo, administración de DNS y la compatibilidad con la lectura y el enlace LDAP. La compatibilidad con la escritura LDAP está disponible para los objetos creados en el dominio administrado con Azure AD DS, pero no los recursos sincronizados desde Azure AD.

Para más información sobre las opciones de identidad, [compare Azure AD DS con Azure AD, Active Directory Domain Services en máquinas virtuales de Azure y Active Directory Domain Services en entornos locales][compare].

Las siguientes características de Azure AD DS simplifican las operaciones de implementación y administración:

* **Experiencia de implementación simplificada:** Azure AD DS está habilitado para el inquilino de Azure AD con un solo asistente en la Azure Portal.
* **Integración con Azure AD:** Estas cuentas de usuario, las pertenencias a grupos y las credenciales están disponibles automáticamente dentro del inquilino de Azure AD. Los nuevos usuarios, grupos o cambios de atributos que se producen en el inquilino o en el directorio en el entorno local de Azure AD se sincronizan automáticamente con Azure AD DS.
    * Las cuentas de los directorios externos vinculados a su instancia de Azure AD no están disponibles en Azure AD DS. Las credenciales no están disponibles para esos directorios externos, por lo que no se pueden sincronizar en un dominio administrado de Azure AD DS.
* **Utilizar las credenciales y contraseñas corporativas:** Las contraseñas para usuarios de Azure AD DS son las mismas que en el inquilino de Azure AD. Los usuarios pueden utilizar sus credenciales corporativas para las máquinas de unión al dominio, iniciar sesión de forma interactiva o a través de escritorio remoto y autenticarse en el dominio administrado de Azure AD DS.
* **Autenticación Kerberos y NTLM:** con compatibilidad para la autenticación Kerberos y NTLM, puede implementar las aplicaciones que dependen de la autenticación integrada de Windows.
* **Alta disponibilidad:** Azure AD DS incluye varios controladores de dominio, que proporcionan alta disponibilidad para el dominio administrado. Esta alta disponibilidad garantiza el tiempo de actividad del servicio y la resistencia a los errores.
    * En las regiones que admiten [Azure Availability Zones][availability-zones], estos controladores de dominio también se distribuyen entre zonas para obtener resistencia adicional.

Entre algunos de los aspectos clave de un dominio administrado de Azure AD DS se incluyen los siguientes:

* El dominio administrado de Azure AD DS es un dominio independiente. No es una extensión de un dominio local.
    * Si es necesario, puede crear relaciones de confianza de bosque de salida unidireccionales de Azure AD DS a un entorno de AD DS local. Para más información, consulte [Conceptos y características del bosque de recursos en Azure AD DS][ forest-trusts].
* El equipo de TI no necesita administrar ni supervisar controladores de dominio de este dominio administrado de Azure AD DS, ni tampoco aplicarles revisiones.

En entornos híbridos que se ejecutan AD DS de forma local, no es necesario administrar la replicación de AD en el dominio administrado de Azure AD DS. Las cuentas de usuario, las pertenencias a grupos y las credenciales del directorio local se sincronizan con Azure AD mediante [Azure AD Connect][azure-ad-connect]. Estas cuentas de usuario, las pertenencias a grupos y las credenciales están disponibles automáticamente dentro del dominio administrado de Azure AD DS.


## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las comparaciones de Azure AD DS con otras soluciones de identidad y cómo funciona la sincronización, consulte los siguientes artículos:

* [Comparación de Azure AD DS con Azure AD, Active Directory Domain Services en máquinas virtuales de Azure y Active Directory Domain Services en entornos locales][compare]
* [Información sobre la sincronización de Azure AD Domain Services con el directorio Azure AD][synchronization]

Para empezar, [cree un dominio administrado con Azure Portal][tutorial-create].

<!-- INTERNAL LINKS -->
[compare]: compare-identity-solutions.md
[synchronization]: synchronization.md
[tutorial-create]: tutorial-create-instance.md
[azure-ad-connect]: ../active-directory/hybrid/whatis-azure-ad-connect.md
[password-hash-sync]: ../active-directory/hybrid/how-to-connect-password-hash-synchronization.md
[availability-zones]: ../availability-zones/az-overview.md
[forest-trusts]: concepts-resource-forest.md
[administration-concepts]: administration-concepts.md
[synchronization]: synchronization.md
[concepts-replica-sets]: concepts-replica-sets.md
