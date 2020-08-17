---
title: Renovación automática de las reservas de Azure
description: Obtenga información acerca de cómo renovar automáticamente las reservas de Azure para seguir obteniendo descuentos en las reservas.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: 644156c29ec084bbc3050aa1d12b779b53084524
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286553"
---
# <a name="automatically-renew-reservations"></a>Renovación automática de reservas

Puede renovar las reservas para adquirir automáticamente un reemplazo cuando expire una reserva existente. La renovación automática es una forma sencilla de seguir obteniendo descuentos en las reservas. También evita tener que supervisar de cerca las expiraciones de las reservas. Con la renovación automática, no tendrá que renovar manualmente y evitará perder las ventajas en cuanto al ahorro. El valor de renovación está desactivado de forma predeterminada. El valor de renovación se puede habilitar o deshabilitar en cualquier momento, hasta la expiración de la reserva existente.

La renovación de una reserva no extiende el plazo de la existente cuando esta expira, sino que crea otra.

Opte por recibir la renovación automática en cualquier momento. El precio de la renovación está disponible 30 días antes de la expiración de la reserva existente. Si habilita la renovación más de 30 días antes de que la reserva expire, se le enviará un correo electrónico donde se detallarán los costos de renovación de 30 días antes de la expiración. El precio de la reserva podría cambiar entre el momento en que bloquea el precio de renovación y el momento en que hay se realiza. Si eso sucede, el costo de la renovación es el menor de los dos costos. Puede realizar los cambios que desee en la cantidad de reservas. Si lo hace, la renovación se actualiza para usar el precio de mercado establecido en el momento del cambio de la cantidad.

La renovación no es obligatoria y puede optar por no recibirla en cualquier momento antes de que expire la reserva existente.

## <a name="set-up-renewal"></a>Configuración de la renovación

Vaya a Azure Portal > **Reservations**.

1. Seleccione la reserva.
2. Haga clic en **Renovación**.
3. Seleccione **Comprar automáticamente una nueva reserva en la fecha de expiración** .  
  ![Ejemplo que muestra la renovación de una reserva](./media/reservation-renew/reservation-renewal.png)

## <a name="if-you-dont-renew"></a>Si no se realiza la renovación

Los servicios siguen funcionando con normalidad. Cuando la reserva expira, se le cobran las tarifas de pago por uso.

## <a name="required-renewal-permissions"></a>Permisos necesarios para la renovación

Para renovar una reserva es preciso que cumplir las siguientes condiciones:

- Debe ser el propietario de la reserva existente.
- Debe ser propietario de la suscripción si la reserva está en el ámbito de una suscripción individual o de un grupo de recursos.
- Debe ser el propietario de la suscripción si tiene un ámbito compartido.

## <a name="default-renewal-settings"></a>Configuración predeterminada de la renovación

De forma predeterminada, la renovación hereda todas las propiedades de la reserva que expira. La compra de la renovación de una reserva tiene la misma SKU, región, ámbito, suscripción de facturación, término y cantidad.

Sin embargo, si desea ahorrar aún más, puede ajustar la cantidad de reservas compradas que desea renovar.

## <a name="when-the-new-reservation-is-purchased"></a>Cuando se adquiere la nueva reserva

Cuando expira la reserva existente, se compra una nueva. Intentamos evitar cualquier retraso entre las dos reservas. La continuidad garantiza la predictibilidad de los costos y que se sigan obteniendo descuentos.

## <a name="changing-parent-reservation-after-setting-renewal"></a>Cambios en la reserva primaria después de establecer la renovación

Si realiza cualquiera de los siguientes cambios en la reserva que va a expirar, se cancelará la renovación de la reserva:

- Dividir
- Merge
- Transferir la reserva de una cuenta a otra
- Transferir la reserva de una suscripción WebDirect a una suscripción con Contrato Enterprise (EA) o cualquier otro método de compra
- Renovación de la inscripción

La nueva reserva hereda el ámbito y el valor de flexibilidad del tamaño de la instancia de la reserva que expira durante la renovación.

## <a name="new-reservation-permissions"></a>Permisos de la nueva reserva

Azure copia los permisos de la reserva que expira a la nueva. Además, el administrador de la cuenta de la suscripción de la compra de la reserva tiene acceso a la nueva reserva.

## <a name="potential-renewal-problems"></a>Posibles problemas en la renovación

Es posible que Azure no procese una renovación si:

- No se puede cobrar el pago
- Se produce un error del sistema durante la renovación
- La SKU que expira no está activa durante la renovación
- El Contrato Enterprise se renueva a otro Contrato Enterprise

Recibirá una notificación por correo electrónico si se da cualquiera de las condiciones anteriores y la renovación se desactiva.

## <a name="renewal-notification"></a>Notificación de la renovación

Se envían correos electrónicos a diferentes personas según en función del método de compra:

- Clientes con Contrato Enterprise: los correos electrónicos se envían al conjunto de contactos de notificaciones en el portal de EA.
- Clientes de suscripciones individuales con tarifas de pago por uso: los correos electrónicos se envían a los usuarios que están configurados como administradores de cuenta.
- Clientes con el programa Proveedor de soluciones en la nube: los correos electrónicos se envían al contacto de notificaciones del asociado.

## <a name="next-steps"></a>Pasos siguientes
- Para más información acerca de Azure Reservations, consulte [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
