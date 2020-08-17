---
title: Información general sobre el control del mantenimiento de máquinas virtuales de Azure mediante Azure Portal
description: Obtenga información sobre cómo controlar cuándo se aplica mantenimiento a las máquinas virtuales de Azure mediante el control de mantenimiento.
author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 04/22/2020
ms.author: cynthn
ms.openlocfilehash: c3e914b904b0f6f1d3a4fae6c43c81cdf4eae819
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080019"
---
# <a name="managing-platform-updates-with-maintenance-control"></a>Administración de las actualizaciones de la plataforma con el control de mantenimiento 

Administre las actualizaciones de la plataforma, que no requieren un reinicio, mediante el control de mantenimiento. Azure actualiza con frecuencia su infraestructura para mejorar la confiabilidad, el rendimiento y la seguridad o para poner en marcha nuevas características. La mayoría de las actualizaciones son transparentes para los usuarios. Algunas cargas de trabajo especialmente delicadas, como los juegos, el streaming multimedia o las transacciones financieras, no pueden tolerar ni siquiera unos segundos de bloqueo o desconexión por mantenimiento de una máquina virtual. El control de mantenimiento ofrece la opción de poner en espera las actualizaciones de la plataforma y aplicarlas en un período sucesivo de 35 días. 

El control de mantenimiento le permite decidir cuándo se aplican las actualizaciones a las máquinas virtuales aisladas y los hosts dedicados de Azure.

Con el control de mantenimiento, puede:
- Aplicar actualizaciones por lotes en un único paquete de actualización.
- Esperar hasta 35 días para aplicar las actualizaciones. 
- Automatice las actualizaciones de la plataforma en su ventana de mantenimiento mediante [Azure Functions](https://github.com/Azure/azure-docs-powershell-samples/tree/master/maintenance-auto-scheduler).
- Mantener el trabajo de las configuraciones entre suscripciones y grupos de recursos. 

## <a name="limitations"></a>Limitaciones

- Las máquinas virtuales deben estar en un [host dedicado](./linux/dedicated-hosts.md) o bien crearse con un [tamaño de máquina virtual aislada](isolation.md).
- Después de 35 días, se aplicará automáticamente una actualización.
- El usuario debe tener acceso de **colaborador del recurso**.

## <a name="management-options"></a>Opciones de administración

Puede crear y administrar configuraciones de mantenimiento con cualquiera de estas opciones:

- [CLI de Azure](maintenance-control-cli.md)
- [Azure PowerShell](maintenance-control-powershell.md)
- [Azure Portal](maintenance-control-portal.md)

En el artículo sobre la [programación de actualizaciones de mantenimiento con control de mantenimiento y Azure Functions](https://github.com/Azure/azure-docs-powershell-samples/tree/master/maintenance-auto-scheduler) encontrará un ejemplo de Azure Functions.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte [Mantenimiento y actualizaciones](maintenance-and-updates.md).
