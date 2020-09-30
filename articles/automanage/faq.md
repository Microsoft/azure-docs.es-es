---
title: Preguntas más frecuentes sobre Azure Automanage para máquinas virtuales
description: Respuestas a las preguntas más frecuentes sobre Azure Automanage para máquinas virtuales.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: troubleshooting
ms.date: 09/04/2020
ms.author: deanwe
ms.openlocfilehash: ed97f7861f5dd959fd41ac22b4e497f492dbc3a3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932418"
---
# <a name="frequently-asked-questions-for-azure-automanage-for-vms"></a>Preguntas más frecuentes sobre Azure Automanage para máquinas virtuales

En este artículo se proporcionan respuestas a algunas de las preguntas más comunes sobre [Azure Automanage para máquinas virtuales](automanage-virtual-machines.md).

Si su problema con Azure no se trata en este artículo, visite los foros de Azure en [MSDN y Stack Overflow](https://azure.microsoft.com/support/forums/). Puede publicar su problema en ellos o [@AzureSupport en Twitter](https://twitter.com/AzureSupport). También puede enviar una solicitud de soporte técnico de Azure. Para enviar una solicitud de soporte técnico, en la [página de Soporte técnico de Azure](https://azure.microsoft.com/support/options/), seleccione **Obtener soporte técnico**.


## <a name="azure-automanage-for-virtual-machines"></a>Azure Automanage para máquinas virtuales

**¿Qué requisitos previos se necesitan para habilitar Azure Automanage?**

Los requisitos previos para habilitar Azure Automanage son los siguientes:
- Solo máquinas virtuales de Windows Server.
- Las máquinas virtuales deben estar en ejecución.
- Las máquinas virtuales deben estar en una región admitida.
- El usuario debe tener los permisos correctos.
- Solo máquinas virtuales que no son de conjunto de escalado.
- Las máquinas virtuales no se deben vincular a un área de trabajo de análisis de registros en una suscripción distinta.

**¿Qué permiso de RBAC es necesario para habilitar Automanage?**

Los usuarios deben tener el rol Propietario. Para aplicar Automanage, los usuarios también pueden tener el rol Colaborador junto con el rol Administrador de acceso de usuario.


**¿Qué regiones se admiten?**

Se admiten máquinas virtuales en las regiones siguientes: Oeste de Europa, Este de EE. UU., Oeste de EE. UU. 2, Centro de Canadá, Centro-oeste de EE. UU.


**¿Qué funcionalidades automatiza Azure Automanage?**

Automanage inscribe, configura y supervisa los servicios mencionados [aquí](virtual-machines-best-practices.md) durante todo el ciclo de vida de la máquina virtual.


**¿Se pueden personalizar configuraciones en Azure Automanage?**

Los clientes pueden personalizar la configuración de servicios específicos, como la retención de Azure Backup, a través de las preferencias de configuración. Consulte la documentación que aparece [aquí](virtual-machines-best-practices.md) para ver la lista completa de las configuraciones que se pueden modificar.


**¿Azure Automanage funciona con máquinas virtuales Linux y Windows?**

Actualmente, Automanage admite máquinas virtuales de Azure con Windows Server.


**¿Se puede aplicar Automanage de manera selectiva a un solo conjunto de máquinas virtuales?**

Automanage se puede habilitar simplemente haciendo clic y señalando máquinas virtuales nuevas y existentes. Automanage también se pueden deshabilitar en cualquier momento.


**¿Azure Automanage admite máquinas virtuales en un conjunto de escalado de máquinas virtuales?**

No, actualmente Azure Automanage no admite máquinas virtuales en un conjunto de escalado de máquinas virtuales.


**¿Cuánto cuesta Azure Automanage?**

Azure Automanage está disponible sin costo adicional en la versión preliminar pública. Los recursos adjuntos de Azure, como Azure Backup, supondrán un costo.


**¿Se puede aplicar Automanage a través de Azure Policy?**

Sí, tenemos una directiva integrada que aplicará automáticamente Automanage a todas las máquinas virtuales dentro del ámbito que se defina. También se especificará el perfil de configuración (DevTest o Producción) junto con su cuenta de Automanage. Obtenga más información [aquí](virtual-machines-policy-enable.md) sobre cómo habilitar Automanage a través de Azure Policy.


**¿Qué es una cuenta de Automanage?**

Una cuenta de Automanage es una identidad de servicio administrado (MSI) que proporciona el contexto de seguridad o la identidad bajo la cual se producen las operaciones automatizadas.


**¿Se puede cambiar el perfil de configuración de una máquina virtual?**

En este momento, tendrá que deshabilitar Automanage para esa máquina virtual y, a continuación, volver a habilitarlo con las preferencias y el perfil de configuración que desee.


**Si una máquina virtual ya está configurada para un servicio, como Update Management, ¿se volverá a configurar con Automanage?**
No, no se volverá a configurar con Automanage. Se iniciará la supervisión del desfase de los recursos asociados a ese servicio.


**¿Por qué una máquina virtual tiene un estado de error en el portal de Automanage?**

Si ve que el estado es *Erróneo*, puede solucionar los problemas de implementación a través del grupo de recursos donde se encuentra la máquina virtual. Vaya a **Grupos de recursos**, seleccione el grupo de recursos, haga clic en **Implementaciones** y consulte el estado *Erróneo* que aparece ahí junto con los detalles del error.

**¿Cómo se puede obtener ayuda para solucionar problemas de Automanage?**

Se puede presentar una [incidencia de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). En la opción **Servicio**, busque y seleccione *Automanage* en la sección *Supervisión y administración*.


## <a name="next-steps"></a>Pasos siguientes

Intente habilitar Automanage para máquinas virtuales en Azure Portal.

> [!div class="nextstepaction"]
> [Habilitación de Automanage para máquinas virtuales en Azure Portal](quick-create-virtual-machines-portal.md)