---
title: Preguntas más frecuentes sobre Azure AD Domain Services | Microsoft Docs
description: Lea y comprenda algunas de las preguntas más frecuentes sobre la configuración, la administración y la disponibilidad de Azure Active Directory Domain Services
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 48731820-9e8c-4ec2-95e8-83dba1e58775
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 06/05/2020
ms.author: iainfou
ms.openlocfilehash: cc78df7ea904bf85f5f2561319e6fc773244e971
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005220"
---
# <a name="frequently-asked-questions-faqs-about-azure-active-directory-ad-domain-services"></a>Preguntas más frecuentes (P+F) sobre Azure Active Directory (AD) Domain Services

Esta página responde a las preguntas más frecuentes acerca de Azure Active Directory Domain Services.

## <a name="configuration"></a>Configuración

* [¿Se pueden crear varios dominios administrados para un único directorio de Azure AD?](#can-i-create-multiple-managed-domains-for-a-single-azure-ad-directory)
* [¿Puedo habilitar Azure AD Domain Services en una red virtual clásica?](#can-i-enable-azure-ad-domain-services-in-a-classic-virtual-network)
* [¿Puedo habilitar Azure AD Domain Services en una red virtual de Azure Resource Manager?](#can-i-enable-azure-ad-domain-services-in-an-azure-resource-manager-virtual-network)
* [¿Puedo migrar mi dominio administrado existente de una red virtual clásica a una red virtual de Resource Manager?](#can-i-migrate-my-existing-managed-domain-from-a-classic-virtual-network-to-a-resource-manager-virtual-network)
* [¿Puedo habilitar Azure AD Domain Services en una suscripción de Azure CSP (proveedor de soluciones en la nube)?](#can-i-enable-azure-ad-domain-services-in-an-azure-csp-cloud-solution-provider-subscription)
* [¿Puedo habilitar Azure AD Domain Services en un directorio de Azure AD federado? No sincronizo los valores hash de contraseña con Azure AD. ¿Puedo habilitar Azure AD Domain Services para este directorio?](#can-i-enable-azure-ad-domain-services-in-a-federated-azure-ad-directory-i-do-not-synchronize-password-hashes-to-azure-ad-can-i-enable-azure-ad-domain-services-for-this-directory)
* [¿Puedo hacer que Azure AD Domain Services esté disponible en varias redes virtuales dentro de mi suscripción?](#can-i-make-azure-ad-domain-services-available-in-multiple-virtual-networks-within-my-subscription)
* [¿Puedo habilitar Servicios de dominio de Azure AD mediante PowerShell?](#can-i-enable-azure-ad-domain-services-using-powershell)
* [¿Puedo habilitar Azure AD Domain Services mediante la plantilla de Resource Manager?](#can-i-enable-azure-ad-domain-services-using-a-resource-manager-template)
* [¿Puedo agregar controladores de dominio a un dominio administrado de Servicios de dominio de Azure AD?](#can-i-add-domain-controllers-to-an-azure-ad-domain-services-managed-domain)
* [¿Pueden los usuarios invitados a mi directorio usar Azure AD Domain Services?](#can-guest-users-invited-to-my-directory-use-azure-ad-domain-services)
* [¿Puedo trasladar un dominio administrado existente de Azure AD Domain Services a otra suscripción, grupo de recursos, región o red virtual?](#can-i-move-an-existing-azure-ad-domain-services-managed-domain-to-a-different-subscription-resource-group-region-or-virtual-network)
* [¿Azure AD Domain Services incluye opciones de alta disponibilidad?](#does-azure-ad-domain-services-include-high-availability-options)

### <a name="can-i-create-multiple-managed-domains-for-a-single-azure-ad-directory"></a>¿Se pueden crear varios dominios administrados para un único directorio de Azure AD?
No. Solo se puede crear un único dominio administrado atendido por Azure AD Domain Services para un único directorio de Azure AD.

### <a name="can-i-enable-azure-ad-domain-services-in-a-classic-virtual-network"></a>¿Puedo habilitar Azure AD Domain Services en una red virtual clásica?
No se admiten redes virtuales clásicas para nuevas implementaciones. Los dominios administrados existentes implementados en redes virtuales clásicas seguirán siendo compatibles hasta que se retiren el 1 de marzo de 2023. Para las implementaciones existentes, puede [migrar Azure AD Domain Services desde el modelo de red virtual clásica a Resource Manager](migrate-from-classic-vnet.md).

Para obtener más información, vea el [aviso de desuso oficial](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/).

### <a name="can-i-enable-azure-ad-domain-services-in-an-azure-resource-manager-virtual-network"></a>¿Puedo habilitar Azure AD Domain Services en una red virtual de Azure Resource Manager?
Sí. Azure AD Domain Services se puede habilitar en una red virtual de Azure Resource Manager. Ya no hay redes virtuales clásicas de Azure disponibles cuando crea un nuevo dominio administrado.

### <a name="can-i-migrate-my-existing-managed-domain-from-a-classic-virtual-network-to-a-resource-manager-virtual-network"></a>¿Puedo migrar mi dominio administrado existente de una red virtual clásica a una red virtual de Resource Manager?
Sí. Para más información, consulte [Migración de Azure AD Domain Services desde el modelo de red virtual clásica a Resource Manager](migrate-from-classic-vnet.md).

### <a name="can-i-enable-azure-ad-domain-services-in-an-azure-csp-cloud-solution-provider-subscription"></a>¿Puedo habilitar Azure AD Domain Services en una suscripción de Azure CSP (proveedor de soluciones en la nube)?
Sí. Para obtener más información, consulte [cómo habilitar Azure AD Domain Services en las suscripciones de Azure CSP](csp.md).

### <a name="can-i-enable-azure-ad-domain-services-in-a-federated-azure-ad-directory-i-do-not-synchronize-password-hashes-to-azure-ad-can-i-enable-azure-ad-domain-services-for-this-directory"></a>¿Puedo habilitar Azure AD Domain Services en un directorio de Azure AD federado? No sincronizo los valores hash de contraseña con Azure AD. ¿Puedo habilitar Azure AD Domain Services para este directorio?
No. Para autenticar a los usuarios a través de NTLM o Kerberos, Azure AD Domain Services necesita acceso a los hashes de contraseña de las cuentas de usuario . En un directorio federado, los hashes de contraseña no se almacenan en el directorio de Azure AD. Por tanto, Azure AD Domain Services no funciona con estos directorios de Azure AD.

Sin embargo, si usa Azure AD Connect para la sincronización de hash de contraseñas, puede usar Azure AD Domain Services porque los valores hash de contraseña se almacenan en Azure AD.

### <a name="can-i-make-azure-ad-domain-services-available-in-multiple-virtual-networks-within-my-subscription"></a>¿Puedo hacer que Azure AD Domain Services esté disponible en varias redes virtuales dentro de mi suscripción?
El propio servicio no admite directamente este escenario. Su dominio administrado está disponible en una sola red virtual a la vez. Sin embargo, puede configurar la conectividad entre varias redes virtuales con el fin de exponer Azure AD Domain Services a otras redes virtuales. Para obtener más información, consulte [cómo conectar redes virtuales en Azure mediante puertas de enlace VPN](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md) o [emparejamiento de redes virtuales](../virtual-network/virtual-network-peering-overview.md).

### <a name="can-i-enable-azure-ad-domain-services-using-powershell"></a>¿Puedo habilitar Servicios de dominio de Azure AD mediante PowerShell?
Sí. Para más información, consulte [cómo habilitar Azure AD Domain Services mediante PowerShell](powershell-create-instance.md).

### <a name="can-i-enable-azure-ad-domain-services-using-a-resource-manager-template"></a>¿Puedo habilitar Azure AD Domain Services mediante la plantilla de Resource Manager?
Sí, puede crear un dominio administrado de Azure AD Domain Services mediante una plantilla de Resource Manager. Se deben crear una entidad de servicio y un grupo de Azure AD para la administración mediante Azure Portal o Azure PowerShell antes de que se implemente la plantilla. Para más información, consulte [Creación de un dominio administrado de Azure Active Directory Domain Services mediante una plantilla de Resource Manager](template-create-instance.md). Al crear un dominio administrado de Azure AD Domain Services en Azure Portal, existe también la opción de exportar la plantilla para su uso en implementaciones adicionales.

### <a name="can-i-add-domain-controllers-to-an-azure-ad-domain-services-managed-domain"></a>¿Puedo agregar controladores de dominio a un dominio administrado de Servicios de dominio de Azure AD?
No. El dominio de Servicios de dominio de Azure AD es un dominio administrado. No es necesario aprovisionar, configurar o administrar de otro modo controladores de dominio para este dominio. Microsoft proporciona estas actividades de administración como servicio. Por lo tanto, no podrá agregar controladores de dominio adicionales (ni de lectura y escritura ni de solo lectura) para el dominio administrado.

### <a name="can-guest-users-invited-to-my-directory-use-azure-ad-domain-services"></a>¿Pueden los usuarios invitados a mi directorio usar Azure AD Domain Services?
No. Los usuarios invitados a su directorio de Azure AD mediante el proceso de invitación [B2B de Azure AD](../active-directory/active-directory-b2b-what-is-azure-ad-b2b.md) se sincronizan en el dominio administrado de Azure AD Domain Services. Sin embargo, las contraseñas para estos usuarios no se almacenan en el directorio de Azure AD. Por lo tanto, Azure AD Domain Services no tiene ninguna manera de sincronizar los códigos hash de Kerberos y NTLM para estos usuarios en el dominio administrado. Estos usuario no pueden iniciar sesión o unir equipos al dominio administrado.

### <a name="can-i-move-an-existing-azure-ad-domain-services-managed-domain-to-a-different-subscription-resource-group-region-or-virtual-network"></a>¿Puedo trasladar un dominio administrado existente de Azure AD Domain Services a otra suscripción, grupo de recursos, región o red virtual?
No. Después de crear un dominio administrado de Azure AD Domain Services, no puede trasladar el dominio administrado a otro grupo de recursos, red virtual, suscripción, etc. Tenga cuidado a la hora de seleccionar la suscripción, el grupo de recursos, la región y la red virtual más adecuados al implementar el dominio administrado.

### <a name="does-azure-ad-domain-services-include-high-availability-options"></a>¿Azure AD Domain Services incluye opciones de alta disponibilidad?

Sí. Cada dominio administrado de Azure AD Domain Services contiene dos controladores de dominio. No necesita administrar estos controladores de dominio ni conectarse a ellos; forman parte del servicio administrado. Si implementa Azure AD Domain Services en una región que admite Availability Zones, los controladores de dominio se distribuirán por diferentes zonas. En las regiones que no admitan Availability Zones, los controladores de dominio se distribuirán por diferentes conjuntos de disponibilidad. No tiene ninguna opción de configuración ni ningún control que le permita administrar esta distribución. Para más información, consulte [Opciones de disponibilidad para máquinas virtuales de Azure](../virtual-machines/windows/availability.md).

## <a name="administration-and-operations"></a>Administración y operaciones

* [¿Puedo conectarme al controlador de dominio para mi dominio administrado mediante Escritorio remoto?](#can-i-connect-to-the-domain-controller-for-my-managed-domain-using-remote-desktop)
* [He habilitado Azure AD Domain Services. ¿Qué cuenta de usuario utilizo para unir máquinas a este dominio?](#ive-enabled-azure-ad-domain-services-what-user-account-do-i-use-to-domain-join-machines-to-this-domain)
* [¿Puedo tener privilegios de administrador de dominio para el dominio administrado proporcionado por Azure AD Domain Services?](#do-i-have-domain-administrator-privileges-for-the-managed-domain-provided-by-azure-ad-domain-services)
* [¿Puedo modificar pertenencias a grupos mediante LDAP u otras herramientas administrativas de AD en dominios administrados?](#can-i-modify-group-memberships-using-ldap-or-other-ad-administrative-tools-on-managed-domains)
* [¿Cuánto tardan los cambios realizados en el directorio de Azure AD en estar visibles en mi dominio administrado?](#how-long-does-it-take-for-changes-i-make-to-my-azure-ad-directory-to-be-visible-in-my-managed-domain)
* [¿Puedo extender el esquema del dominio administrado por Azure AD Domain Services?](#can-i-extend-the-schema-of-the-managed-domain-provided-by-azure-ad-domain-services)
* [¿Puedo modificar o agregar registros DNS en mi dominio administrado?](#can-i-modify-or-add-dns-records-in-my-managed-domain)
* [¿Qué es la directiva de duración de la contraseña en un dominio administrado?](#what-is-the-password-lifetime-policy-on-a-managed-domain)
* [¿Azure AD Domain Services proporciona protección de bloqueo de cuentas de AD?](#does-azure-ad-domain-services-provide-ad-account-lockout-protection)
* [¿Puedo configurar el sistema de archivos distribuido (DFS) y la replicación en Azure AD Domain Services?](#can-i-configure-distributed-file-system-and-replication-within-azure-ad-domain-services)

### <a name="can-i-connect-to-the-domain-controller-for-my-managed-domain-using-remote-desktop"></a>¿Puedo conectarme al controlador de dominio para mi dominio administrado mediante Escritorio remoto?
No. Los administradores de inquilinos no tienen privilegios para conectarse a controladores de dominio en el dominio administrado con el Escritorio remoto. Los miembros del grupo *Administradores de controlador de dominio de AAD* pueden administrar el dominio administrado usando las herramientas de administración de AD, como el Centro de administración de Active Directory (ADAC) o AD PowerShell. Estas herramientas se instalan mediante la característica *Herramientas de administración de servidor remoto* en un servidor de Windows unido al dominio administrado. Para obtener más información, consulte [Creación de una máquina virtual de administración para configurar y administrar un dominio administrado de Azure AD Domain Services](tutorial-create-management-vm.md).

### <a name="ive-enabled-azure-ad-domain-services-what-user-account-do-i-use-to-domain-join-machines-to-this-domain"></a>He habilitado Azure AD Domain Services. ¿Qué cuenta de usuario utilizo para unir máquinas a este dominio?
Cualquier cuenta de usuario que forme parte del dominio administrado puede unirse a una VM. A los miembros del grupo *Administradores de AAD DC* se les concede acceso de Escritorio remoto a las máquinas que se han unido al dominio administrado.

### <a name="do-i-have-domain-administrator-privileges-for-the-managed-domain-provided-by-azure-ad-domain-services"></a>¿Puedo tener privilegios de administrador de dominio para el dominio administrado proporcionado por Azure AD Domain Services?
No. No se le conceden privilegios administrativos en el dominio administrado. Los privilegios *Administrador de dominio* y *Administrador de organización* no estarán disponibles dentro del dominio. A los miembros de los grupos de administradores de dominio o administradores de organización dentro del directorio local de Active Directory tampoco se les concederán privilegios de administrador de dominio o de organización en el dominio administrado.

### <a name="can-i-modify-group-memberships-using-ldap-or-other-ad-administrative-tools-on-managed-domains"></a>¿Puedo modificar pertenencias a grupos mediante LDAP u otras herramientas administrativas de AD en dominios administrados?
Los usuarios y grupos que se sincronizan desde Azure Active Directory con Azure AD Domain Services no se pueden modificar porque su origen es Azure Active Directory. Cualquier grupo o usuario que se origine en el dominio administrado se puede modificar.

### <a name="how-long-does-it-take-for-changes-i-make-to-my-azure-ad-directory-to-be-visible-in-my-managed-domain"></a>¿Cuánto tardan los cambios realizados en el directorio de Azure AD en estar visibles en mi dominio administrado?
Los cambios realizados en el directorio de Azure AD mediante la interfaz de usuario de Azure AD o PowerShell se sincronizan automáticamente con el dominio administrado. Este proceso de sincronización se ejecuta en segundo plano. No hay ningún período de tiempo definido para que esta sincronización complete todos los cambios en el objeto.

### <a name="can-i-extend-the-schema-of-the-managed-domain-provided-by-azure-ad-domain-services"></a>¿Puedo extender el esquema del dominio administrado por Azure AD Domain Services?
No. Microsoft administra el esquema del dominio administrado. No se admiten extensiones de esquema en los Servicios de dominio de Azure AD.

### <a name="can-i-modify-or-add-dns-records-in-my-managed-domain"></a>¿Puedo modificar o agregar registros DNS en mi dominio administrado?
Sí. Los miembros del grupo *Administradores de controlador de dominio de AAD* reciben privilegios de *Administrador de DNS* con el fin de modificar los registros DNS en el dominio administrado. Estos usuarios pueden utilizar la consola de administrador de DNS en una máquina que ejecuta Windows Server unido al dominio administrado para administrar DNS. A fin de utilizar la consola de administrador de DNS, instale *Herramientas del servidor DNS*, que forma parte de la característica opcional *Herramientas de administración remota del servidor* en el servidor. Para obtener más información, consulte [Administración de DNS en un dominio administrado con Azure AD Domain Services](manage-dns.md)

### <a name="what-is-the-password-lifetime-policy-on-a-managed-domain"></a>¿Qué es la directiva de duración de la contraseña en un dominio administrado?
La duración predeterminada de la contraseña en un dominio administrado de Azure AD Domain Services es de 90 días. Esta duración de la contraseña no está sincronizada con la duración de la contraseña configurada en Azure AD. Por lo tanto, podría darse una situación donde las contraseñas de los usuarios hayan expirado en el dominio administrado, pero sigan siendo válidas en Azure AD. En tales escenarios, los usuarios deben cambiar sus contraseñas en Azure AD y la nueva contraseña se sincronizará con el dominio administrado. Además, los atributos *password-does-not-expire* y *user-must-change-password-at-next-logon* de las cuentas de usuario no se sincronizan con el dominio administrado.

### <a name="does-azure-ad-domain-services-provide-ad-account-lockout-protection"></a>¿Azure AD Domain Services proporciona protección de bloqueo de cuentas de AD?
Sí. Tras escribir una contraseña incorrecta del dominio administrado cinco veces en un lapso de dos minutos, la cuenta de usuario quedará bloqueada durante 30 minutos. Pasados los 30 minutos, la cuenta de usuario se desbloqueará automáticamente. Los intentos no válidos de escribir la contraseña del dominio administrado no provocarán el bloqueo de la cuenta de usuario en Azure AD. La cuenta de usuario solo se bloqueará en el dominio administrado de Azure AD Domain Services. Para obtener más información [Directivas de bloqueo de cuenta y contraseña en dominios administrados](password-policy.md).

### <a name="can-i-configure-distributed-file-system-and-replication-within-azure-ad-domain-services"></a>¿Puedo configurar el sistema de archivos distribuido y la replicación en Azure AD Domain Services?
No. El sistema de archivos distribuido (DFS) y la replicación no están disponibles cuando se usa Azure AD Domain Services.

## <a name="billing-and-availability"></a>Facturación y disponibilidad

* [¿Azure AD Domain Services es un servicio de pago?](#is-azure-ad-domain-services-a-paid-service)
* [¿Hay una versión de prueba gratuita para el servicio?](#is-there-a-free-trial-for-the-service)
* [¿Puedo pausar un dominio administrado de Azure AD Domain Services?](#can-i-pause-an-azure-ad-domain-services-managed-domain)
* [¿Puedo conmutar por error Azure AD Domain Services a otra región en un evento de recuperación ante desastres?](#can-i-pause-an-azure-ad-domain-services-managed-domain)
* [¿Puedo obtener Servicios de dominio de Azure AD como parte de Enterprise Mobility Suite (EMS)? ¿Necesito Azure AD Premium para usar Azure AD Domain Services?](#can-i-failover-azure-ad-domain-services-to-another-region-for-a-dr-event)
* [¿En qué regiones de Azure está disponible el servicio?](#can-i-get-azure-ad-domain-services-as-part-of-enterprise-mobility-suite-ems-do-i-need-azure-ad-premium-to-use-azure-ad-domain-services)

### <a name="is-azure-ad-domain-services-a-paid-service"></a>¿Azure AD Domain Services es un servicio de pago?
Sí. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/active-directory-ds/)para obtener más información.

### <a name="is-there-a-free-trial-for-the-service"></a>¿Hay una versión de prueba gratuita para el servicio?
Se incluye Azure AD Domain Services en la evaluación gratuita de Azure. Puede suscribirse a un [mes de evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).

### <a name="can-i-pause-an-azure-ad-domain-services-managed-domain"></a>¿Puedo pausar un dominio administrado de Azure AD Domain Services?
No. Después de habilitar un dominio administrado de Azure AD Domain Services, el servicio está disponible en la red virtual administrada hasta que se elimina el dominio administrado. No hay manera de pausar el servicio. La facturación continúa cada hora hasta que se elimine el dominio administrado.

### <a name="can-i-failover-azure-ad-domain-services-to-another-region-for-a-dr-event"></a>¿Puedo conmutar por error Azure AD Domain Services a otra región en un evento de recuperación ante desastres?
No. Azure AD Domain Services no proporciona actualmente un modelo de implementación con redundancia geográfica. Se limita a una única red virtual en una región de Azure. Si quiere usar varias regiones de Azure, debe ejecutar los controladores de dominio de Active Directory en máquinas virtuales de IaaS de Azure. Para la guía de arquitectura, consulte [Amplíe el dominio local de Active Directory a Azure](https://docs.microsoft.com/azure/architecture/reference-architectures/identity/adds-extend-domain).

### <a name="can-i-get-azure-ad-domain-services-as-part-of-enterprise-mobility-suite-ems-do-i-need-azure-ad-premium-to-use-azure-ad-domain-services"></a>¿Puedo obtener Servicios de dominio de Azure AD como parte de Enterprise Mobility Suite (EMS)? ¿Necesito Azure AD Premium para usar Azure AD Domain Services?
No. No, Azure AD Domain Services es un servicio de pago por uso de Azure y no forma parte de EMS. Azure AD Domain Services puede utilizarse con todas las ediciones de Azure AD (Gratis y Premium). Se le facturará por cada hora en función del uso.

### <a name="what-azure-regions-is-the-service-available-in"></a>¿En qué regiones de Azure está disponible el servicio?
Consulte la página de [servicios de Azure por región](https://azure.microsoft.com/regions/#services/) para saber en qué regiones de Azure está disponible Azure AD Domain Services.

## <a name="troubleshooting"></a>Solución de problemas

Consulte nuestra [Guía de solución de problemas](troubleshoot.md) para obtener soluciones a los problemas más habituales al configurar o administrar Azure AD Domain Services.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure AD Domain Services, consulte [¿Qué es Azure Active Directory Domain Services?](overview.md).

Para comenzar, consulte [Crear y configurar una dominio administrado de Azure Active Directory Domain Services](tutorial-create-instance.md).
