---
title: Cancelación de su suscripción de Azure
description: Describe cómo cancelar la suscripción de Azure, como la suscripción a la evaluación gratuita
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 16ba70717d04c44e950ce2ed2c47e54859c4be47
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684888"
---
# <a name="cancel-your-azure-subscription"></a>Cancelación de su suscripción de Azure

Puede cancelar la suscripción de Azure en Azure Portal si ya no necesita la suscripción.

Aunque no es obligatorio, Microsoft *recomienda* realizar las siguientes acciones antes de cancelar una suscripción:

* Realice una copia de seguridad de los datos. Por ejemplo, si está almacenando datos en Azure Storage o SQL, descargue una copia. Si tiene una máquina virtual, guarde una imagen de la misma localmente.
* Cierre los servicios. Vaya a la [página de recursos en el portal de administración](https://ms.portal.azure.com/?flight=1#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fresources) y **detenga** todas las máquinas virtuales, aplicaciones u otros servicios en ejecución.
* Considere la posibilidad de migrar los datos. Consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md).
* Elimine todos los recursos y todos los grupos de recursos.
* Si tiene algún rol personalizado que haga referencia a esta suscripción en `AssignableScopes`, debe actualizar este rol personalizado para quitar la suscripción. Si intenta actualizar un rol personalizado después de cancelar una suscripción, es posible que obtenga un error. Para más información, consulte [Solución de problemas con los roles personalizados](../../role-based-access-control/troubleshooting.md#problems-with-custom-roles) y [Roles personalizados de Azure](../../role-based-access-control/custom-roles.md).

Si cancela un plan de soporte técnico de Azure de pago, se le facturará el resto del período de la suscripción. Para más información, vea [Soporte técnico de Azure](https://azure.microsoft.com/support/plans/).

## <a name="cancel-subscription-in-the-azure-portal"></a>Cancelación de la suscripción en Azure Portal

1. Seleccione su suscripción en la [página Suscripciones de Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).
1. Seleccione la suscripción que quiere cancelar.
1. Seleccione **Introducción** y, luego, seleccione **Cancelar suscripción**.
    ![Captura de pantalla que muestra el botón Cancelar](./media/cancel-azure-subscription/cancel_ibiza.png)
1. Siga las indicaciones y finalice la cancelación.

## <a name="who-can-cancel-a-subscription"></a>¿Quién puede cancelar una suscripción?

En la tabla siguiente se describe el permiso necesario para cancelar una suscripción.

|Tipo de suscripción     |Quién puede cancelar  |
|---------|---------|
|Suscripciones creadas al registrarse en Azure a través del sitio web de Azure. Por ejemplo, cuando se registra para obtener una [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), como una [cuenta con tarifas de pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o como un [suscriptor de Visual Studio](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/). |  Administrador de cuenta y propietarios de la suscripción  |
|[Contrato Enterprise de Microsoft](https://azure.microsoft.com/pricing/enterprise-agreement/) y [Desarrollo/pruebas - Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/)     |  Propietario de la cuenta y propietarios de la suscripción       |
|[Plan de Azure](https://azure.microsoft.com/offers/ms-azr-0017g/) y [Azure Plan for DevTest](https://azure.microsoft.com/offers/ms-azr-0148g/)     |  Propietarios de la suscripción      |

## <a name="what-happens-after-i-cancel-my-subscription"></a>¿Qué ocurre después de cancelar la suscripción?

Una vez que se cancela, la facturación se detiene inmediatamente. Sin embargo, pueden transcurrir hasta 10 minutos hasta que la cancelación se muestre en el portal. Si cancela en medio de un período de facturación, enviamos la factura final en la fecha de factura típica una vez finalizado el período.

Después de cancelar, los servicios se deshabilitan. Esto significa que las máquinas virtuales se desasignan, las direcciones IP temporales se liberan y el almacenamiento es de solo lectura.

Tras la cancelación de la suscripción, Microsoft espera entre 30 y 90 días para eliminar de forma permanente los datos, por si el usuario necesitará acceder a ellos o cambiara de opinión. No se le cobrará por conservar los datos. Para más información, consulte el [Microsoft Trust Center: How we manage your data](https://go.microsoft.com/fwLink/p/?LinkID=822930&clcid=0x409) (Centro de confianza de Microsoft: Administración de los datos).

## <a name="delete-free-trial-subscription"></a>Eliminación de una suscripción de evaluación gratuita

Si tiene una suscripción de evaluación gratuita, no es preciso que espere 30 días para que la suscripción se elimine automáticamente. Puede eliminarla *tres días* después de cancelarla. La opción **Eliminar suscripción** no está disponible hasta tres días después de cancelar la suscripción.

1. Espere tres días después de la fecha de cancelación de la suscripción.
1. Seleccione la suscripción en la página [Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) de Azure Portal.
1. Seleccione la suscripción que quiere eliminar.
1. Seleccione **Información general** y, después, **Eliminar suscripción**.

## <a name="reactivate-subscription"></a>Reactivación de la suscripción

Si cancela su suscripción de pago por uso accidentalmente, puede [volver a activarla en el Centro de cuentas](subscription-disabled.md).

Si su suscripción no es de pago por uso, para reactivar la suscripción, póngase en contacto con el soporte técnico en los 90 días posteriores a la cancelación.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- Si fuera necesario, las suscripciones de pago por uso se pueden reactivar en el [Centro de cuentas](subscription-disabled.md).