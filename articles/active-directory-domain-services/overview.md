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
ms.date: 08/14/2020
ms.author: iainfou
ms.custom: contperfq1
ms.openlocfilehash: 2255f2193378d0cc3611680c6aa5f8e0837e99aa
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245186"
---
# <a name="what-is-azure-active-directory-domain-services"></a>¿Qué es Azure Active Directory Domain Services?

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directiva de grupo, protocolo ligero de acceso a directorios (LDAP) y autenticación Kerberos o NTLM. Puede usar estos servicios de dominio sin necesidad de implementar o administrar los controladores de dominio de la nube, ni de aplicarles revisiones.

Un dominio administrado de Azure AD DS le permite ejecutar aplicaciones heredadas en la nube que no pueden usar métodos de autenticación modernos o cuando no quiere que las búsquedas de directorio regresen siempre a un entorno de AD DS local. Esas aplicaciones heredadas se pueden migrar mediante "lift-and-shift" del entorno local a un dominio administrado, sin necesidad de administrar el entorno de AD DS en la nube.

Azure AD DS se integra con el inquilino de Azure AD existente. Esta integración permite a los usuarios iniciar sesión en el servicio y las aplicaciones conectadas al dominio administrado con sus credenciales existentes. También puede usar grupos y cuentas de usuario existentes para proteger el acceso a los recursos. Estas características proporcionan una migración mediante lift-and-shift más fluida de los recursos locales a Azure.

> [!div class="nextstepaction"]
> [Para empezar, crear un dominio administrado de Azure AD DS con Azure Portal][tutorial-create]

## <a name="how-does-azure-ad-ds-work"></a>¿Cómo funciona Azure AD DS?

Cuando cree un dominio administrado de Azure AD DS, defina un espacio de nombres único. Este espacio de nombres es el nombre de dominio, por ejemplo, *aaddscontoso.com*. A continuación, se implementan dos controladores de dominio de Windows Server en la región de Azure seleccionada. Esta implementación de controladores de dominio se conoce como "conjunto de réplicas".

No es necesario administrar, configurar ni actualizar estos controladores de dominio. La plataforma Azure los administra como parte del dominio administrado, incluidas las copias de seguridad.

Un dominio administrado está configurado para realizar una sincronización unidireccional desde Azure AD y proporcionar acceso a un conjunto central de usuarios, grupos y credenciales. Puede crear los recursos directamente en el dominio administrado, pero no se vuelven a sincronizar con Azure AD. Las aplicaciones, los servicios y las máquinas virtuales de Azure que se conectan al dominio administrado pueden usar las características de AD DS comunes, como unión a un dominio, directiva de grupo, LDAP y autenticación Kerberos o NTLM.

En un entorno híbrido con un entorno de AD DS local, [Azure AD Connect][azure-ad-connect] sincroniza la información de identidad con Azure AD, que se sincroniza posteriormente con el dominio administrado.

![Sincronización en Azure AD Domain Services con Azure AD y AD DS local mediante AD Connect](./media/active-directory-domain-services-design-guide/sync-topology.png)

Azure AD DS replica la información de identidad de Azure AD, por lo que funciona con inquilinos de Azure AD que solo están en la nube o que están sincronizados con un entorno de AD DS local. El mismo conjunto de características de Azure AD DS existe para ambos entornos.

* Si tiene un entorno de AD DS local, puede sincronizar la información de las cuentas de usuario para proporcionar una identidad coherente para los usuarios. Para más información, consulte [Procedimiento para sincronizar objetos y credenciales en un dominio administrado][synchronization].
* En el caso de los entornos solo en la nube, no se necesita un entorno de AD DS local tradicional para usar los servicios de identidad centralizados de Azure AD DS.

Puede ampliar un dominio administrado para que tenga más de un conjunto de réplicas por cada inquilino de Azure AD. Los conjuntos de réplicas pueden agregarse a cualquier red virtual emparejada en cualquier región de Azure que admita Azure AD DS. Contar con conjuntos de réplicas adicionales en diferentes regiones de Azure facilita la recuperación geográfica ante desastres de las aplicaciones heredadas si una región de Azure se queda sin conexión. Actualmente, los conjuntos de réplicas están en versión preliminar. Para más información, consulte el artículo sobre los [conceptos y características de los conjuntos de réplicas en dominios administrados][concepts-replica-sets].

El vídeo siguiente proporciona información general sobre el modo en que Azure AD DS se integra con las aplicaciones y las cargas de trabajo para proporcionar servicios de identidad en la nube:

<br />

>[!VIDEO https://www.youtube.com/embed/T1Nd9APNceQ]

Para ver escenarios de implementación de Azure AD DS en acción, puede explorar los ejemplos siguientes:

* [Azure AD DS para organizaciones híbridas](scenarios.md#azure-ad-ds-for-hybrid-organizations)
* [Azure AD DS para organizaciones solo en la nube](scenarios.md#azure-ad-ds-for-cloud-only-organizations)

## <a name="azure-ad-ds-features-and-benefits"></a>Características y ventajas de Azure AD DS

Para proporcionar servicios de identidad a aplicaciones y máquinas virtuales en la nube, Azure AD DS es totalmente compatible con un entorno de AD DS tradicional para las operaciones como la unión a un dominio, LDAP seguro (LDAPS), directiva de grupo, administración de DNS y la compatibilidad con la lectura y el enlace LDAP. La compatibilidad con la escritura LDAP está disponible para los objetos creados en el dominio administrado, pero no para los recursos sincronizados desde Azure AD.

Para más información sobre las opciones de identidad, [compare Azure AD DS con Azure AD, AD DS en máquinas virtuales de Azure y AD DS local][compare].

Las siguientes características de Azure AD DS simplifican las operaciones de implementación y administración:

* **Experiencia de implementación simplificada:** Azure AD DS está habilitado para el inquilino de Azure AD con un solo asistente en la Azure Portal.
* **Integración con Azure AD:** Estas cuentas de usuario, las pertenencias a grupos y las credenciales están disponibles automáticamente dentro del inquilino de Azure AD. Los nuevos usuarios, grupos o cambios de atributos que se producen en el inquilino o en el directorio en el entorno local de Azure AD se sincronizan automáticamente con Azure AD DS.
    * Las cuentas de los directorios externos vinculados a su instancia de Azure AD no están disponibles en Azure AD DS. No hay credenciales para esos directorios externos, por lo que no se pueden sincronizar con un dominio administrado.
* **Utilizar las credenciales y contraseñas corporativas:** Las contraseñas para usuarios de Azure AD DS son las mismas que en el inquilino de Azure AD. Los usuarios pueden utilizar sus credenciales corporativas para las máquinas de unión al dominio, iniciar sesión de forma interactiva o a través de escritorio remoto y autenticarse en el dominio administrado.
* **Autenticación Kerberos y NTLM:** con compatibilidad para la autenticación Kerberos y NTLM, puede implementar las aplicaciones que dependen de la autenticación integrada de Windows.
* **Alta disponibilidad:** Azure AD DS incluye varios controladores de dominio, que proporcionan alta disponibilidad para el dominio administrado. Esta alta disponibilidad garantiza el tiempo de actividad del servicio y la resistencia a los errores.
    * En las regiones que admiten [Azure Availability Zones][availability-zones], estos controladores de dominio también se distribuyen entre zonas para obtener resistencia adicional.
    * Se pueden usar también [conjuntos de réplicas][concepts-replica-sets] para proporcionar recuperación ante desastres geográfica de las aplicaciones heredadas si una región de Azure se queda sin conexión.

Algunos de los aspectos clave de un dominio administrado son los siguientes:

* El dominio administrado es un dominio independiente. No es una extensión de un dominio local.
    * Si es necesario, puede crear relaciones de confianza de bosque de salida unidireccionales de Azure AD DS a un entorno de AD DS local. Para más información, consulte [Conceptos y características del bosque de recursos en Azure AD DS][ forest-trusts].
* El equipo de TI no necesita administrar, revisar ni supervisar controladores de dominio de este dominio administrado.

En entornos híbridos que ejecutan AD DS de forma local, no es necesario administrar la replicación de AD en el dominio administrado. Las cuentas de usuario, las pertenencias a grupos y las credenciales del directorio local se sincronizan con Azure AD mediante [Azure AD Connect][azure-ad-connect]. Estas cuentas de usuario, las pertenencias a grupos y las credenciales están disponibles automáticamente dentro de este dominio administrado.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las comparaciones de Azure AD DS con otras soluciones de identidad y cómo funciona la sincronización, consulte los siguientes artículos:

* [Comparación de Azure AD DS con Azure AD, Active Directory Domain Services en máquinas virtuales de Azure y Active Directory Domain Services en entornos locales][compare]
* [Información sobre la sincronización de Azure AD Domain Services con el directorio Azure AD][synchronization]
* Para obtener información sobre cómo se administraun dominio administrado, consulte [Conceptos de administración para cuentas de usuario, contraseñas y administración en Azure AD DS][administration-concepts].

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
