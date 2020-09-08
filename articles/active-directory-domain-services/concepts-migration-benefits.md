---
title: Ventajas de la migración de implementación clásica en Azure AD Domain Services | Microsoft Docs
description: Más información sobre las ventajas de migrar una implementación clásica de Azure Active Directory Domain Services al modelo de implementación de Resource Manager
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: iainfou
ms.openlocfilehash: 93dcc1202c08be905cf08513f38e79a8a7674e01
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88650135"
---
# <a name="benefits-of-migration-from-the-classic-to-resource-manager-deployment-model-in-azure-active-directory-domain-services"></a>Ventajas de la migración del modelo de implementación clásico al de Resource Manager en Azure Active Directory Domain Services

Azure Active Directory Domain Services (Azure AD DS) le permite migrar un dominio administrado que usa el modelo de implementación clásica al modelo de implementación de Resource Manager. Los dominios administrados de Azure AD DS que usan el modelo de implementación de Resource Manager proporcionan características adicionales, como la directiva de contraseñas específica, los registros de auditoría y la protección de bloqueo de cuentas.

En este artículo se describen las ventajas de la migración. Para empezar, consulte [Migración de Azure AD Domain Services desde el modelo de red virtual clásica a Resource Manager][howto-migrate].

> [!NOTE]
> En 2017, Azure AD Domain Services se comenzó a poder hospedar en una red de Azure Resource Manager. Desde entonces, hemos podido crear un servicio más seguro gracias a las modernas funcionalidades de Azure Resource Manager. Como las implementaciones de Azure Resource Manager están reemplazando totalmente a las implementaciones clásicas, las implementaciones de las redes virtuales clásicas de Azure AD DS se retirarán el 1 de marzo de 2023.
>
> Para obtener más información, vea el [aviso de desuso oficial](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/).

## <a name="migration-benefits"></a>Ventajas de la migración

El proceso de migración toma un dominio administrado existente que usa el modelo de implementación clásica para adoptar el modelo de implementación de Resource Manager. Al migrar un dominio administrado del modelo de implementación clásica a Resource Manager, evitará tener que volver a unir las máquinas al dominio administrado o tener que eliminar el dominio administrado y crear uno desde cero. Las máquinas virtuales siguen unidas al dominio administrado al final del proceso de migración.

Después de la migración, Azure AD DS proporciona muchas características que solo están disponibles en los dominios que usan el modelo de implementación de Resource Manager como, por ejemplo:

* [Compatibilidad con la directiva de contraseñas específica][password-policy].
* Velocidades de sincronización más rápidas entre Azure AD y Azure AD Domain Services.
* Dos nuevos [atributos que se sincronizan desde Azure AD][attributes] - *manager* y *employeeID*.
* Acceso a controladores de dominio de mayor potencia al [actualizar la SKU][skus].
* Protección de bloqueo de cuentas de AD
* [Notificaciones por correo electrónico para alertas en el dominio administrado][email-alerts].
* [Uso de libros de Azure y de Azure Monitor para ver los registros de auditoría y la actividad de inicio de sesión][workbooks].
* En las regiones admitidas, [Azure Availability Zones][availability-zones].
* Integraciones con otros productos de Azure, como [Azure Files][azure-files], [HD Insights][hd-insights]y [Windows Virtual Desktop][wvd].
* La compatibilidad tiene acceso a más datos de telemetría y puede ayudar a solucionar los problemas de forma más eficaz.
* El cifrado en reposo mediante [Azure Managed Disks][managed-disks] para los datos de los controladores de dominio administrados.

Los dominios administrados que usan el modelo de implementación de Resource Manager ayudan a estar al día de las características más recientes. No hay nuevas características disponibles para los dominios administrados que usen el modelo de implementación clásica.

## <a name="next-steps"></a>Pasos siguientes

Para empezar, consulte [Migración de Azure AD Domain Services desde el modelo de red virtual clásica a Resource Manager][howto-migrate].

<!-- LINKS - INTERNAL -->
[password-policy]: password-policy.md
[skus]: change-sku.md
[email-alerts]: notifications.md
[workbooks]: use-azure-monitor-workbooks.md
[azure-files]: ../storage/files/storage-files-identity-auth-active-directory-domain-service-enable.md
[hd-insights]: ../hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds.md
[wvd]: ../virtual-desktop/overview.md
[availability-zones]: ../availability-zones/az-overview.md
[howto-migrate]: migrate-from-classic-vnet.md
[attributes]: synchronization.md#attribute-synchronization-and-mapping-to-azure-ad-ds
[managed-disks]: ../virtual-machines/managed-disks-overview.md
