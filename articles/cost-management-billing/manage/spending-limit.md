---
title: Límite de gasto de Azure
description: En este artículo se describe cómo funciona un límite de gasto de Azure y cómo quitarlo.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 04/30/2021
ms.author: banders
ms.openlocfilehash: f49b318cfb413aff98a37dc244d43c690c8c1698
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753660"
---
# <a name="azure-spending-limit"></a>Límite de gasto de Azure

El límite de gasto de Azure evita los gastos por encima del importe de crédito. Todos los clientes nuevos que se registren para obtener una cuenta nueva de Azure o los tipos de suscripción que incluyan créditos para varios meses tienen activado el límite de gasto de forma predeterminada. El límite de gasto es igual a la cantidad de crédito. La cantidad del límite de gasto no se puede cambiar. Por ejemplo, si se ha registrado para obtener una cuenta gratuita de Azure, el límite de gasto es 200 dólares y no se puede cambiar a 500 dólares, pero sí puede quitarlo. Por consiguiente, no tiene límite o tiene un límite igual a la cantidad de crédito. Así se evita la mayoría de los tipos de gasto.

El límite de gasto no está disponible para las suscripciones con planes de compromiso o con precios de pago por uso. Para esos tipos de suscripciones, no se muestra el límite de gasto en Azure Portal y no se puede habilitar un límite. Vea la [lista completa de los tipos de suscripción de Azure y la disponibilidad del límite de gasto](https://azure.microsoft.com/support/legal/offer-details/).

## <a name="reaching-a-spending-limit"></a>Alcanzar un límite de gasto

Cuando el uso genere cargos que agoten el límite de gasto, los servicios que ha implementado se deshabilitan durante el resto de ese período de facturación.

Por ejemplo, si gasta todo el crédito de su cuenta gratuita de Azure, los recursos de Azure que ha implementado se quitan de la producción y las máquinas virtuales de Azure se detienen y se desasignan. Los datos de las cuentas de almacenamiento están disponibles únicamente en modo de solo lectura.

Si su tipo de suscripción incluye créditos durante varios meses, la suscripción se vuelve a habilitar automáticamente al principio del siguiente período de facturación. Después, puede volver a implementar los recursos de Azure y tener acceso total a las cuentas de almacenamiento y bases de datos.

Azure envía notificaciones por correo electrónico cuando se alcanza el límite de gasto. Inicie sesión en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) para ver notificaciones acerca de las suscripciones que han alcanzado el límite de gasto.

Si se ha registrado para obtener una cuenta gratuita de Azure y alcanza el límite de gasto, puede actualizar a un precio de [pago por uso](upgrade-azure-subscription.md) para quitar el límite de gasto y que la suscripción de vuelva a habilitar automáticamente.

## <a name="remove-the-spending-limit-in-azure-portal"></a>Eliminación del límite de gasto en Azure Portal

Puede quitar el límite de gasto en cualquier momento siempre que haya un método de pago válido asociado a la suscripción de Azure. En el caso de los tipos de suscripción con crédito en varios meses, como Visual Studio Enterprise y Visual Studio Professional, también puede optar por quitar el límite de gasto indefinidamente o solo durante el período de facturación actual. Si elige solo el período de facturación actual, el límite de gasto se habilitará automáticamente al principio del siguiente período de facturación.

Si tiene una cuenta gratuita de Azure, consulte el artículo sobre cómo [actualizar la suscripción de Azure](upgrade-azure-subscription.md) para quitar el límite de gasto. En caso contrario, siga estos pasos para quitar el límite de gasto:

<a id="remove"></a>

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de la cuenta.
1. Busque **Administración de costos + facturación**.  
    :::image type="content" source="./media/spending-limit/search-bar.png" alt-text="Captura de pantalla que muestra la búsqueda de Cost Management + Billing." lightbox="./media/spending-limit/search-bar.png" :::
1. En la lista **Mis suscripciones**, seleccione su suscripción. Por ejemplo, *Visual Studio Enterprise*.  
    :::image type="content" source="./media/spending-limit/cost-management-overview-msdn-x.png" alt-text="Captura de pantalla que muestra la cuadrícula Mis suscripciones en Información general." lightbox="./media/spending-limit/cost-management-overview-msdn-x.png" :::
    > [!NOTE]
    > Si no ve aquí alguna de sus suscripciones a Visual Studio, puede deberse a que haya cambiado el directorio de suscripción en algún momento. Para estas suscripciones, debe cambiar el directorio al directorio original (el directorio en el que se registró inicialmente). A continuación, repita el paso 2.
1. En la información general de la suscripción, haga clic en el banner para quitar el límite de gasto.  
    :::image type="content" source="./media/spending-limit/msdn-remove-spending-limit-banner-x.png" alt-text="Captura de pantalla que muestra el banner para quitar el límite de gasto." lightbox="./media/spending-limit/msdn-remove-spending-limit-banner-x.png" :::
1. Elija si desea quitar el límite de gasto indefinidamente o solo durante el período de facturación actual.  
    :::image type="content" source="./media/spending-limit/remove-spending-limit-blade-x.png" alt-text="Captura de pantalla que muestra la página para quitar el límite de gasto." lightbox="./media/spending-limit/remove-spending-limit-blade-x.png" :::
    - Al seleccionar la opción **Quitar el límite de gasto indefinidamente**, se impide que el límite de gasto se habilite automáticamente al principio del siguiente período de facturación. Sin embargo, puede volver a activarla en cualquier momento.
    - Al seleccionar **Quitar el límite de gasto para el período de facturación actual**, se quitará el límite de gasto para que vuelva a activarse automáticamente al principio del siguiente período de facturación.
1. Haga clic en **Seleccionar método de pago** a fin de elegir un método de pago para la suscripción. Se convertirá en el método de pago activo de su suscripción.
1. Haga clic en **Finalizar**


## <a name="why-you-might-want-to-remove-the-spending-limit"></a>¿Por qué querría quitar el límite de gasto?

El límite de gasto podría impedirle implementar o usar ciertos servicios de Microsoft o de terceros. A continuación, se indican las situaciones donde debería quitar el límite de gasto en su suscripción.

-  Planea implementar imágenes de terceros como Oracle o servicios como Azure DevOps Services. Esta situación hace que alcance su límite de gasto casi inmediatamente y que la suscripción se deshabilite.
- Tiene servicios que no quiere que se interrumpan. Al alcanzar el límite de gasto, los recursos de Azure que implementó se eliminan de producción y las máquinas virtuales de Azure se detienen y se desasignan. Si tiene servicios que no desea que se interrumpan, debe quitar el límite de gasto.
- Tiene servicios y recursos con configuraciones como direcciones IP virtuales que no quiere perder. Estos valores se pierden cuando se alcanza el límite de gasto y se desasignan los servicios y recursos.

## <a name="turn-on-the-spending-limit-after-removing"></a>Activación del límite de gasto después de quitarlo

Esta característica solo está disponible cuando el límite de gasto se ha eliminado de forma indefinida en los tipos de suscripción que incluyen créditos en varios meses. Puede usar esta característica para activar el límite de gasto automáticamente al principio del siguiente período de facturación.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de la cuenta.
1. Busque **Administración de costos + facturación**.  
    :::image type="content" source="./media/spending-limit/search-bar.png" alt-text="Captura de pantalla que muestra la búsqueda de Cost Management + Billing." lightbox="./media/spending-limit/search-bar.png" :::
1. En la lista **Mis suscripciones**, seleccione su suscripción. Por ejemplo, *Visual Studio Enterprise*.  
    :::image type="content" source="./media/spending-limit/cost-management-overview-msdn-x.png" alt-text="Captura de pantalla que muestra la cuadrícula Mis suscripciones en Información general." lightbox="./media/spending-limit/cost-management-overview-msdn-x.png" :::
    > [!NOTE]
    > Si no ve aquí alguna de sus suscripciones a Visual Studio, puede deberse a que haya cambiado el directorio de suscripción en algún momento. Para estas suscripciones, debe cambiar el directorio al directorio original (el directorio en el que se registró inicialmente). A continuación, repita el paso 2.
1. En la información general de la suscripción, haga clic en el banner situado en la parte superior de la página para volver a activar el límite de gasto.

## <a name="custom-spending-limit"></a>Límite de gasto personalizado

Los límites de gasto personalizados no están disponibles.

## <a name="a-spending-limit-doesnt-prevent-all-charges"></a>Un límite de gasto no evita todos los cargos

[Algunos servicios externos publicados en Azure Marketplace](../understand/understand-azure-marketplace-charges.md) no se pueden usar con los créditos de una suscripción y pueden generar cargos independientes incluso si se ha establecido un límite de gasto. Como ejemplos se pueden citar las licencias de Visual Studio, Azure Active Directory Premium, los planes de soporte técnico y la mayoría de los servicios de marca de terceros. Al aprovisionar un nuevo servicio externo, se muestra una advertencia en la que se le informa de que los servicios se facturan por separado:

![Advertencia de compra de Marketplace](./media/spending-limit/marketplace-warning01.png)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes
- Actualización a un plan con precios de [pago por uso](upgrade-azure-subscription.md).
