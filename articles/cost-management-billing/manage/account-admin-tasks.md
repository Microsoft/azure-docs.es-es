---
title: Tareas de administrador de cuenta en Azure Portal
description: Describe cómo realizar operaciones de pago en Azure Portal
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: ad39b9980562c345aff87f4e0771f265011886b3
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683307"
---
# <a name="account-administrator-tasks-in-the-azure-portal"></a>Tareas de administrador de cuenta en Azure Portal

En este artículo se explica cómo realizar las siguientes tareas en Azure Portal:
- Administrar los métodos de pago de la suscripción
- Eliminar el límite de gasto en su suscripción
- Agregar créditos a una suscripción de Azure bajo licencia Open

Debe ser el administrador de cuenta para realizar cualquiera de estas tareas.

## <a name="navigate-to-your-subscriptions-payment-methods"></a>Ir a los métodos de pago de la suscripción

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de la cuenta.

1. Busque **Administración de costos + facturación**.

    ![Captura de pantalla que muestra la búsqueda para la administración de costos y facturación ](./media/account-admin-tasks/search-bar.png)

1. En **Mis suscripciones**, seleccione la suscripción a la que quiere agregar la tarjeta de crédito.

   ![Captura de pantalla que muestra la cuadrícula Mis suscripciones en información general](./media/account-admin-tasks/cost-management-billing-overview-x.png)

   > [!NOTE]
   > Si no ve aquí alguna de sus suscripciones, puede deberse a que haya cambiado el directorio de suscripción en algún momento. Para estas suscripciones, debe cambiar el directorio al directorio original (el directorio en el que se registró inicialmente). A continuación, repita el paso 2.

1. Seleccione **Métodos de pago**.

    ![Captura de pantalla que muestra la hoja de métodos de pago seleccionada.](./media/account-admin-tasks/subscription-payment-methods-blade.png)

Aquí puede agregar una nueva tarjeta de crédito, cambiar el método de pago activo, editar los detalles de la tarjeta de crédito y eliminar las tarjetas de crédito.

### <a name="change-active-payment-method"></a>Cambio del método de pago activo

Puede cambiar el método de pago activo si agrega una nueva tarjeta de crédito o elije una que ya esté guardada. Para cambiar el método de pago activo a una nueva tarjeta de crédito:

1. En la esquina superior izquierda, seleccione "+" para agregar una tarjeta de crédito.

    ![Captura de pantalla que muestra el signo más](./media/account-admin-tasks/subscription-payment-methods-plus.png)

1. Escriba los detalles de la tarjeta de crédito en el formulario de la derecha.

    ![Captura de pantalla que muestra el formulario de la tarjeta de crédito.](./media/account-admin-tasks/subscription-add-payment-method-x.png)

1. Para que esta tarjeta sea su forma de pago activa, active la casilla situada junto a **Convertir en mi método de pago activo** en la parte superior del formulario. Esta tarjeta se convertirá en el instrumento de pago activo para todas las suscripciones que utilicen la misma tarjeta que la suscripción seleccionada.

    ![Captura de pantalla que muestra la casilla para activar el método de pago activo de la tarjeta.](./media/account-admin-tasks/subscription-make-active-payment-method-x.png)

1. Seleccione **Next** (Siguiente).

Para cambiar el método de pago activo a una tarjeta de crédito que haya guardado antes:

1. Seleccione la casilla junto a la tarjeta que quiere convertir en el método de pago activo.

    ![Captura de pantalla que muestra el cuadro activado junto a la tarjeta de crédito](./media/account-admin-tasks/subscription-checked-payment-method-x.png)

1. Haga clic en **Establecer como activo** en la barra de comandos.

    ![Captura de pantalla que muestra el botón Establecer como activo](./media/account-admin-tasks/subscription-checked-payment-method-set-active.png)

### <a name="edit-credit-card-details"></a>Edición de los detalles de la tarjeta de crédito

Para editar los detalles de la tarjeta de crédito, como la fecha de expiración o la dirección, haga clic en la tarjeta de crédito que desea editar. Aparecerá un formulario de tarjeta de crédito a la derecha.

![Captura de pantalla que muestra la tarjeta de crédito seleccionada](./media/account-admin-tasks/subscription-edit-payment-method-x.png)

Actualice los detalles de la tarjeta de crédito y haga clic en **Guardar**.

### <a name="remove-a-credit-card-from-the-account"></a>Eliminación de una tarjeta de crédito de la cuenta

1. Seleccione la casilla junto a la tarjeta que quiere eliminar.

    ![Captura de pantalla que muestra el cuadro activado junto a la tarjeta de crédito](./media/account-admin-tasks/subscription-checked-payment-method-x.png)

1. En la barra de comandos, haga clic en **Eliminar**.

    ![Captura de pantalla que muestra el botón Eliminar](./media/account-admin-tasks/subscription-checked-payment-method-delete.png)

Si la tarjeta de crédito es el método de pago activo de otras suscripciones de Microsoft, no podrá eliminarla de la cuenta de Azure. Cambie el método de pago activo en todas las suscripciones vinculadas a esta tarjeta de crédito y pruebe de nuevo.

### <a name="switch-to-invoice-payment"></a>Cambio al pago de la factura

Si tiene la opción de pagar mediante factura (cheque o transferencia bancaria), puede cambiar la suscripción para el pago de facturas (cheque/transferencia bancaria) en Azure Portal.

1. Seleccione **Pago mediante factura** en la barra de comandos.

    ![Captura de pantalla que muestra la hoja de métodos de pago seleccionada.](./media/account-admin-tasks/subscription-payment-methods-pay-by-invoice.png)

1. Escriba la dirección del método de pago mediante factura.
1. Haga clic en **Next**.

Si quiere que se le apruebe el pago mediante factura, consulte [Pago mediante factura](pay-by-invoice.md).

### <a name="edit-invoice-payment-address"></a>Edición de la dirección de pago mediante factura

Para editar la dirección de su método de pago mediante factura, haga clic en **Factura** en la lista de métodos de pago de la suscripción. El formulario de dirección se abrirá a la derecha.

## <a name="remove-spending-limit"></a>Eliminación del límite de gasto

El límite de gasto de Azure evita los gastos por encima del importe de crédito. Puede quitar el límite de gasto en cualquier momento siempre que haya un método de pago válido asociado a la suscripción de Azure. En el caso de los tipos de suscripción con crédito en varios meses, como Visual Studio Enterprise y Visual Studio Professional, también puede optar por habilitar el límite de gasto al principio del siguiente período de facturación.

El límite de gasto no está disponible para las suscripciones con planes de compromiso o con precios de pago por uso. Vea la [lista completa de los tipos de suscripción de Azure y la disponibilidad del límite de gasto](https://azure.microsoft.com/support/legal/offer-details/).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de la cuenta.
1. Busque **Administración de costos + facturación**.

    ![Captura de pantalla que muestra la búsqueda para la administración de costos y facturación ](./media/account-admin-tasks/search-bar.png)

1. En la lista **Mis suscripciones**, seleccione su suscripción de Visual Studio Enterprise.

   ![Captura de pantalla que muestra la cuadrícula Mis suscripciones en información general](./media/account-admin-tasks/cost-management-overview-msdn-x.png)

    > [!NOTE]
    > Si no ve aquí alguna de sus suscripciones a Visual Studio, puede deberse a que haya cambiado el directorio de suscripción en algún momento. Para estas suscripciones, debe cambiar el directorio al directorio original (el directorio en el que se registró inicialmente). A continuación, repita el paso 2.

1. En la información general de la suscripción, haga clic en el banner naranja para quitar el límite de gasto.

    ![Captura de pantalla que muestra el banner quitar el límite de gasto](./media/account-admin-tasks/msdn-remove-spending-limit-banner-x.png)

1. Elija si desea quitar el límite de gasto indefinidamente o solo durante el período de facturación actual.

   ![Captura de pantalla que muestra la hoja para quitar el límite de gasto](./media/account-admin-tasks/remove-spending-limit-blade-x.png)

1. Haga clic en **Seleccionar método de pago** a fin de elegir un método de pago para la suscripción. Se convertirá en el método de pago activo de su suscripción.

1. Haga clic en **Finalizar**

## <a name="add-credits-to-azure-in-open-subscription"></a>Incorporación de créditos una suscripción a Azure bajo licencia Open

Si tiene una suscripción a Azure bajo licencia Open, puede agregar créditos a su suscripción en Azure Portal mediante el canje de una clave de producto o la compra de créditos con una tarjeta de crédito.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de la cuenta.
1. Busque **Administración de costos + facturación**.

    ![Captura de pantalla que muestra la búsqueda para la administración de costos y facturación ](./media/account-admin-tasks/search-bar.png)

1. En la lista **Mis suscripciones**, seleccione su suscripción de Azure bajo licencia Open.

    ![Captura de pantalla que muestra la cuadrícula Mis suscripciones en información general](./media/account-admin-tasks/cost-management-overview-aio-x.png)

   > [!NOTE]
   > Si no ve aquí su suscripción, puede deberse a que haya cambiado el directorio en algún momento. Debe cambiar el directorio de la suscripción al directorio original (el directorio en el que se registró inicialmente). A continuación, repita el paso 2.

1. Seleccione **Historial de crédito**.

    ![Captura de pantalla que muestra el historial de crédito](./media/account-admin-tasks/aio-credit-history-blade.png)

1. En la esquina superior izquierda, seleccione "+" para agregar más créditos.

    ![Captura de pantalla que muestra el botón de agregar crédito](./media/account-admin-tasks/aio-credit-history-plus.png)

1. Seleccione un tipo de método de pago en el menú desplegable. Puede agregar una clave de producto o comprar créditos con una tarjeta de crédito.

    ![Captura de pantalla de la lista desplegable del método de pago en la hoja Agregar créditos](./media/account-admin-tasks/add-credits-select-payment-method.png)

1. Si seleccionó la clave de producto:
    - Especificación de la clave de producto
    - Haga clic en **Validar**.

1. Si eligió la tarjeta de crédito:
    - Haga clic en **Seleccionar método de pago** para agregar una tarjeta de crédito o seleccionar una existente.
    - Especifique la cantidad de créditos que desea agregar.

1. Haga clic en **Aplicar**.

## <a name="troubleshooting"></a>Solución de problemas
No se admiten tarjetas virtuales ni prepago. Si aparecen errores al agregar o actualizar una tarjeta de crédito válida, pruebe a abrir el explorador en modo privado.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre [analizar y evitar costos inesperados en Azure Portal](getting-started.md)
