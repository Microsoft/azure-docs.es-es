---
title: Guía de publicación de ofertas de plantillas de solución de aplicaciones de Azure - Azure Marketplace
description: En este artículo se describen los requisitos para publicar plantillas de solución en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: msjogarrig
ms.author: jogarrig
ms.date: 04/22/2020
ms.openlocfilehash: c66682a7f457612b0ea77b90cc98ec1e6b4a85e9
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322269"
---
# <a name="publishing-guide-for-azure-applications-solution-template-offers"></a>Guía de publicación de ofertas de plantillas de solución de aplicaciones de Azure

En este artículo se explican los requisitos para publicar ofertas de plantillas de solución, que son una manera de publicar una oferta de aplicaciones de Azure en Azure Marketplace. El tipo de oferta de plantilla de solución requiere una [plantilla de Azure Resource Manager (plantilla de ARM)](../azure-resource-manager/templates/overview.md) para implementar automáticamente la infraestructura de la solución.

Use el tipo de oferta de *plantilla de solución* de aplicación de Azure con las siguientes condiciones:

- La solución requiere la automatización adicional de la implementación y la configuración no solo de una máquina virtual (VM), sino de una combinación de VM, redes y recursos de almacenamiento.
- Los clientes van a administrar la solución por su cuenta.

La llamada a la acción que un cliente ve para este tipo de oferta es *Obtenerla ahora*.

## <a name="requirements-for-solution-template-offers"></a>Requisitos de las ofertas de plantillas de solución

| **Requisitos** | **Detalles**  |
| ---------------  | -----------  |
|Facturación y medición    |  Las ofertas de plantillas de solución no son ofertas de transacción, pero se pueden usar para implementar ofertas de VM de pago facturadas mediante el marketplace comercial de Microsoft. Los recursos que implementa la plantilla de ARM de la solución se configuran en la suscripción de Azure del cliente. Las transacciones de máquinas virtuales de pago por uso se realizan con el cliente mediante Microsoft y se facturan con la suscripción de Azure del cliente.<br/> En el caso de la facturación denominada "traiga su propia licencia" (BYOL), aunque Microsoft factura los costos de infraestructura derivados de la suscripción del cliente, usted realizará la transacción de las tarifas de licencia de software directamente con el cliente.   |
|Disco duro virtual (VHD) compatible con Azure  |   Las máquinas virtuales deben estar basadas en Windows o Linux. Para más información, consulte: <ul> <li>[Creación de una oferta de aplicación de Azure](./partner-center-portal/create-new-azure-apps-offer.md) (para discos duros virtuales Windows).</li><li>[Distribuciones de Linux aprobadas en Azure](../virtual-machines/linux/endorsed-distros.md) (para discos duros virtuales de Linux).</li></ul> |
| Atribución de uso del cliente | Se requiere habilitar la atribución de uso del cliente en todas las plantillas de solución que se publican en Azure Marketplace. Para obtener más información sobre la atribución de uso del cliente y cómo habilitarla, consulte [Atribución de uso del cliente para asociados de Azure](./azure-partner-customer-usage-attribution.md).  |
| Uso de discos administrados | Los [discos administrados](../virtual-machines/windows/managed-disks-overview.md) son la opción predeterminada para los discos persistentes de VM de infraestructura como servicio (IaaS) en Azure. Debe usar discos administrados en las plantillas de solución. <ul><li>Para actualizar las plantillas de solución, siga las instrucciones de [Uso de discos administrados en plantillas de Azure Resource Manager](../virtual-machines/using-managed-disks-template-deployments.md) y use [las muestras](https://github.com/Azure/azure-quickstart-templates) proporcionadas.<br><br> </li><li>Para publicar VHD como imagen en Azure Marketplace, importe el VHD subyacente de los discos administrados a una cuenta de almacenamiento mediante uno de los métodos siguientes:<ul><li>[Azure PowerShell](../virtual-machines/scripts/virtual-machines-powershell-sample-copy-managed-disks-vhd.md) </li> <li> [La CLI de Azure](../virtual-machines/scripts/virtual-machines-cli-sample-copy-managed-disks-vhd.md) </li> </ul></ul> |

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, aprenda a [Ampliar su negocio en la nube con Azure Marketplace](https://azuremarketplace.microsoft.com/sell).

Para registrarse y empezar a trabajar en el Centro de partners, haga lo siguiente:

- [Inicie sesión en el Centro de partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) para crear o completar la oferta.
- Para obtener más información, consulte [Creación de una oferta de aplicaciones de Azure](./partner-center-portal/create-new-azure-apps-offer.md).
