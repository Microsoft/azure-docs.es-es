---
title: Preguntas más frecuentes sobre la migración del método clásico al de Azure Resource Manager
description: Preguntas más frecuentes sobre la migración del método clásico al de Azure Resource Manager
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: 0f9a8a1c4631fc667d0dd31ae62f7d57aefccc90
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855851"
---
# <a name="frequently-asked-questions-about-classic-to-azure-resource-manager-migration"></a>Preguntas más frecuentes sobre la migración del método clásico al de Azure Resource Manager

> [!IMPORTANT]
> En la actualidad, aproximadamente el 90 % de las VM de IaaS usan [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). A partir del 28 de febrero de 2020, las máquinas virtuales clásicas han quedado en desuso y se van a retirar por completo el 1 de marzo de 2023. [Obtenga más información]( https://aka.ms/classicvmretirement) sobre esta caída en desuso y [cómo se verá afectado](./classic-vm-deprecation.md#how-does-this-affect-me).

## <a name="what-is-azure-service-manager-and-what-does-it-mean-by-classic"></a>¿Qué es Azure Service Manager y qué significa "clásica"?

La palabra "clásica" en la máquina virtual de IaaS (clásica) hace referencia a las máquinas virtuales administradas por Azure Service Manager (ASM). Azure Service Manager (ASM) es el plano de control anterior de Azure responsable de crear, administrar y eliminar máquinas virtuales, y de realizar otras operaciones de plano de control. 

## <a name="what-is-azure-resource-manager"></a>¿Qué es Azure Resource Manager?

[Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/management/overview) es el plano de control de Azure más reciente responsable de crear, administrar y eliminar máquinas virtuales, y de realizar otras operaciones de plano de control. 

## <a name="what-is-the-time-required-for-migration"></a>¿Cuál es el tiempo necesario para la migración?

El planeamiento y la ejecución de la migración depende en gran medida de la complejidad de la arquitectura y puede tardar unos dos meses.  

## <a name="what-is-the-definition-of-a-new-customer-on-iaas-vms-classic"></a>¿Cuál es la definición de un cliente nuevo en máquinas virtuales de IaaS (clásicas)?

Los clientes que no disponían de máquinas virtuales de IaaS (clásicas) en sus suscripciones durante el mes de febrero de 2020 (un mes antes de que entraran en desuso) se consideran clientes nuevos. 

## <a name="what-is-the-definition-of-an-existing-customer-on-iaas-virtual-machines-classic"></a>¿Cuál es la definición de cliente existente en máquinas virtuales IaaS (clásicas)?

Aquel cliente con máquinas virtuales IaaS (clásicas) activas, o detenidas pero asignadas, en sus suscripciones en el mes de febrero de 2020 se considera un cliente existente. Únicamente estos clientes tienen hasta el 1 de marzo de 2023 para migrar sus máquinas virtuales de Azure Service Manager a Azure Resource Manager. 

## <a name="why-am-i-getting-an-error-stating-newclassicvmcreationnotallowedforsubscription"></a>¿Por qué me aparece un error que indica "NewClassicVMCreationNotAllowedForSubscription"?

Como parte del proceso de retirada, las máquinas virtuales IaaS (clásicas) ya no están disponibles para nuevos clientes. Se le ha identificado como nuevo cliente y, por lo tanto, su operación no se ha autorizado. Se recomienda encarecidamente usar [Azure Virtual Machines con ARM](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-powershell). Si no puede usar máquinas virtuales de Azure con ARM, póngase en contacto con soporte técnico para agregar su suscripción a la lista de permitidos.

## <a name="does-this-migration-plan-affect-any-of-my-existing-services-or-applications-that-run-on-azure-virtual-machines"></a>¿Afecta este plan de migración a alguno de los servicios o las aplicaciones existentes que se ejecutan en máquinas virtuales de Azure? 

No hasta el 1 de marzo de 2023 para las máquinas virtuales de IaaS (clásicas). Las máquinas virtuales de IaaS (clásicas) son servicios totalmente compatibles en la disponibilidad general. Puede seguir usando estos recursos para expandir su huella en Microsoft Azure. El 1 de marzo de 2023, estas máquinas virtuales se retirarán completamente y todas las máquinas virtuales activas o asignadas se detendrán y desasignarán. No habrá ningún impacto en otros recursos clásicos, como Cloud Services (clásico), cuentas de almacenamiento (clásicas), etc.   

## <a name="what-happens-to-my-vms-if-i-dont-plan-on-migrating-in-the-near-future"></a>¿Qué ocurre con mis máquinas virtuales si por ahora no tengo planeado migrar? 

El 1 de marzo de 2023, las máquinas virtuales de IaaS (clásicas) se retirarán completamente y todas las máquinas virtuales activas o asignadas se detendrán y desasignarán. Para evitar el impacto en el negocio, le recomendamos encarecidamente que empiece a planear la migración hoy mismo y que la complete antes del 1 de marzo de 2023. Las API, Cloud Services y el modelo de recursos clásicos existentes no van a quedar en desuso. Queremos facilitar la migración, dadas las características avanzadas disponibles en el modelo de implementación de Resource Manager. Se recomienda empezar a planear la migración de estos recursos a Azure Resource Manager. 

## <a name="what-does-this-migration-plan-mean-for-my-existing-tooling"></a>¿Qué supone este plan de migración para las herramientas existentes? 

La actualización de sus herramientas al modelo de implementación de Resource Manager es uno de los cambios más importantes que tendrá que considerar en sus planes de migración.

## <a name="how-long-will-the-management-plane-downtime-be"></a>¿Cuánto tiempo durará el tiempo de inactividad del plano de administración? 

Depende del número de recursos que se vayan a migrar. Para las implementaciones más pequeñas (unas decenas de máquinas virtuales), el proceso completo de migración debería tardar menos de una hora. Para implementaciones a gran escala (cientos de máquinas virtuales), la migración puede tardar unas horas.

## <a name="can-i-roll-back-after-my-migrating-resources-are-committed-in-resource-manager"></a>¿Puedo revertir el proceso después de que la migración de los recursos esté confirmada en Resource Manager? 

Puede anular la migración siempre que los recursos se encuentren en el estado preparado. No se puede revertir una vez que los recursos se hayan migrado correctamente mediante la operación de confirmación.

## <a name="can-i-roll-back-my-migration-if-the-commit-operation-fails"></a>¿Puedo revertir la migración si se produce un error en la operación de confirmación? 

No se puede anular la migración si se produce un error en la operación de confirmación. Todas las operaciones de migración, incluida la de confirmación, son idempotentes. Por consiguiente, se recomienda que vuelva a intentar la operación en breve. Si sigue experimentando un error, cree una incidencia de soporte técnico.

## <a name="do-i-have-to-buy-another-express-route-circuit-if-i-have-to-use-iaas-under-resource-manager"></a>¿Tengo que comprar otro circuito de ExpressRoute para usar IaaS en Resource Manager? 

No. Hace poco hemos habilitado la [transición de los circuitos ExpressRoute del modelo de implementación clásica al modelo de implementación de Resource Manager](../expressroute/expressroute-move.md). No hace falta que compre un nuevo circuito de ExpressRoute si ya tiene uno.

## <a name="what-if-i-had-configured-role-based-access-control-policies-for-my-classic-iaas-resources"></a>¿Qué ocurre si había configurado las directivas de control de acceso basado en rol para mis recursos IaaS clásicos? 

Durante la migración, los recursos se transforman del modelo clásico al de Resource Manager. Por lo tanto, se recomienda que planee las actualizaciones de directiva de RBAC que deben producirse después de la migración.

## <a name="i-backed-up-my-classic-vms-in-a-vault-can-i-migrate-my-vms-from-classic-mode-to-resource-manager-mode-and-protect-them-in-a-recovery-services-vault"></a>He realizado copias de seguridad de mis máquinas virtuales clásicas en un almacén. ¿Puedo migrar mis máquinas virtuales del modo clásico al modo de Resource Manager y protegerlos en un almacén de Recovery Services?

Cuando traslade una máquina virtual del modo clásico al de Resource Manager, las copias de seguridad realizadas antes de la migración no migrarán a la máquina virtual de Resource Manager recién migrada. Sin embargo, si quiere mantener las copias de seguridad de las máquinas virtuales clásicas, siga estos pasos antes de la migración. 

1. En el almacén de Recovery Services, vaya a la pestaña **Elementos protegidos** y seleccione la máquina virtual. 
2. Haga clic en Stop Protection (Detener protección). Deje la opción *Eliminar los datos de copia de seguridad asociados* **desactivada**.

> [!NOTE]
> Se le cobrará el costo de la instancia de la copia de seguridad hasta que retenga los datos. Las copias de seguridad se eliminarán según el intervalo de retención. Sin embargo, siempre se conserva la última copia de seguridad hasta que elimina explícitamente los datos de copia de seguridad. Se recomienda comprobar el intervalo de retención de la máquina virtual y el desencadenador "Eliminar datos de copia de seguridad" en el elemento protegido en el almacén una vez finalizado el intervalo de retención. 
>
>

Para migrar la máquina virtual al modo de Resource Manager: 

1. Elimine la extensión de instantánea o copia de seguridad de la máquina virtual.
2. Migre la máquina virtual del modo clásico al modo de Resource Manager. Asegúrese de que la información de almacenamiento y red correspondiente a la máquina virtual también se migra al modo de Resource Manager.

Además, si quiere hacer una copia de seguridad de la máquina virtual migrada, vaya a la hoja Administración de máquina virtual para [habilitar la copia de seguridad](../backup/quick-backup-vm-portal.md#enable-backup-on-a-vm).

## <a name="can-i-validate-my-subscription-or-resources-to-see-if-theyre-capable-of-migration"></a>¿Puedo validar mi suscripción o mis recursos para ver si son aptos para la migración? 

Sí. En la opción de migración compatible con la plataforma, el primer paso para preparar la migración consiste en validar si los recursos pueden migrarse. Si se produce un error en la operación de validación, recibe todos los mensajes con todas las razones por las que no se puede completar la migración.

## <a name="what-happens-if-i-run-into-a-quota-error-while-preparing-the-iaas-resources-for-migration"></a>¿Qué ocurre si se produce un error de cuota mientras preparo los recursos IaaS para la migración? 

Se recomienda que anule la migración y, a continuación, presente una solicitud de soporte técnico para que se aumenten las cuotas en la región donde vaya a migrar las máquinas virtuales. Una vez aprobada la solicitud de cuotas, puede empezar a ejecutar los pasos de la migración.

## <a name="how-do-i-report-an-issue"></a>¿Cómo se informa de un problema? 

Publique sus problemas y preguntas sobre la migración en nuestra [Página de preguntas y respuestas de Microsoft sobre VM](https://docs.microsoft.com/answers/topics/azure-virtual-machines.html) con la palabra clave ClassicIaaSMigration. Se recomienda registrar todas sus preguntas en este foro. Si tiene un contrato de soporte técnico, también puede presentar una incidencia de soporte técnico.

## <a name="what-if-i-dont-like-the-names-of-the-resources-that-the-platform-chose-during-migration"></a>¿Qué hago si no me gustan los nombres que eligió la plataforma para los recursos durante la migración? 

Todos los recursos para los que se proporcionen nombres explícitamente en el modelo de implementación clásica se conservan durante la migración. En algunos casos, se crean recursos nuevos. Por ejemplo, se crea una interfaz de red para cada máquina virtual. Actualmente, no se admite la capacidad de controlar los nombres de estos recursos nuevos creados durante la migración. Registre sus votos para esta característica en el [foro de comentarios de Azure](https://feedback.azure.com).

## <a name="can-i-migrate-expressroute-circuits-used-across-subscriptions-with-authorization-links"></a>¿Puedo migrar circuitos de ExpressRoute utilizados en las suscripciones con vínculos de autorización? 

Los circuitos de ExpressRoute que usan vínculos de autorización entre suscripciones no se pueden migrar automáticamente sin tiempo de inactividad. Contamos con instrucciones sobre cómo se pueden migrar mediante pasos manuales. Para conocer los pasos que deben darse y obtener más información, consulte [Migración de circuitos ExpressRoute y las redes virtuales asociadas del modelo de implementación clásica a Resource Manager](../expressroute/expressroute-migration-classic-resource-manager.md).

## <a name="i-got-the-message-vm-is-reporting-the-overall-agent-status-as-not-ready-hence-the-vm-cannot-be-migrated-ensure-that-the-vm-agent-is-reporting-overall-agent-status-as-ready-or-vm-contains-extension-whose-status-is-not-being-reported-from-the-vm-hence-this-vm-cannot-be-migrated"></a>Aparece el mensaje *"La máquina virtual notifica el estado de agente general No preparado. Por lo tanto, la VM no se puede migra. Asegúrese de que el agente de VM notifica el estado de agente general como Preparado"* o *"la máquina virtual contiene la extensión cuyo estado no se notifica desde la VM. Por lo tanto, esta máquina virtual no se puede migrar."*

Este mensaje se recibe cuando la máquina virtual no tiene conectividad saliente a Internet. El agente de VM utiliza conectividad saliente para llegar a la cuenta de almacenamiento de Azure a fin de actualizar el estado del agente cada cinco minutos.


## <a name="next-steps"></a>Pasos siguientes

Para Linux:

* [Información general sobre la migración compatible con la plataforma de recursos de IaaS desde el modelo de implementación clásica a Azure Resource Manager](./linux/migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](./migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](./linux/migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager)
* [Migración de recursos de IaaS de la implementación clásica a Resource Manager con Azure PowerShell](./windows/migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure](./linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Herramientas de la comunidad para ayudar con la migración de recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](./windows/migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de los errores más comunes en la migración](./linux/migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

Para Windows:

* [Información general sobre la migración compatible con la plataforma de recursos de IaaS desde el modelo de implementación clásica a Azure Resource Manager](./windows/migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](./migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](./windows/migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager)
* [Migración de recursos de IaaS de la implementación clásica a Resource Manager con Azure PowerShell](./windows/migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure](./linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Herramientas de la comunidad para ayudar con la migración de recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](./windows/migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de los errores más comunes en la migración](./windows/migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
