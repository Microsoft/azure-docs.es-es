---
title: Administración de reservas de Azure
description: Obtenga información sobre cómo administrar las reservas de Azure. Consulte los pasos para cambiar el ámbito de la reserva, dividirla y optimizar su uso.
ms.service: cost-management-billing
author: bandersmsft
ms.reviewer: yashesvi
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: f3e4a772382606178d6cd5b0dcb92b0d1bc28695
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461823"
---
# <a name="manage-reservations-for-azure-resources"></a>Administración de reservas para los recursos de Azure

Después de comprar una reserva de Azure, es posible que deba aplicar la reserva a otra suscripción, cambiar quién puede administrar la reserva o cambiar el ámbito de la reserva. También puede dividir una reserva en dos para aplicar algunas de las instancias que compró a otra suscripción.

Si ha adquirido Azure Reserved Virtual Machine Instances, puede cambiar la configuración de optimización de la reserva. Puede aplicar el descuento de reserva a las máquinas virtuales de la misma serie o puede reservar la capacidad del centro de datos para un tamaño específico de máquina virtual. Además, debe intentar optimizar las reservas para que se usen totalmente.

*El permiso necesario para administrar una reserva es independiente del permiso de suscripción.*

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="reservation-order-and-reservation"></a>Pedido de reserva y reserva

Cuando compra una reserva, se crean dos objetos: **Pedido de reserva** y **Reserva**.

En el momento de la compra, un pedido de reserva tiene una sola reserva. Acciones como la división, combinación, reembolso parcial o intercambio crean nuevas reservas en el **Pedido de reserva**.

Para ver un pedido de reserva, vaya a **Reservas** > seleccione la reserva y, a continuación, haga clic en el **Id. de pedido de reserva**.

![Ejemplo de los detalles de pedido de reserva que muestra el Id. de pedido de reserva ](./media/manage-reserved-vm-instance/reservation-order-details.png)

Una reserva hereda los permisos de su pedido de reserva.

## <a name="change-the-reservation-scope"></a>Cambio del ámbito de reserva

 El descuento de la reserva se aplica a las máquinas virtuales, SQL Database, Azure Cosmos DB u otros recursos que se correspondan con la reserva y que se ejecuten dentro del ámbito de esta. El contexto de facturación depende de la suscripción que se usó para comprar la reserva.

Para cambiar el ámbito de una reserva:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** > **Reservations**.
3. Seleccione la reserva.
4. Seleccione **Configuración** > **Configuración**.
5. Cambie el ámbito.

Si cambia el ámbito para que pase de ser compartido a único, solo podrá seleccionar suscripciones de las que sea el propietario. Únicamente se pueden seleccionar las suscripciones que pertenezcan al mismo contexto de facturación que la reserva.

El ámbito solo se aplica a suscripciones individuales con tarifas de pago por uso (ofertas MS-AZR-0003P o MS-AZR-0023P), la oferta Enterprise MS-AZR-0017P o MS-AZR-0148P o los tipos de suscripción CSP.

## <a name="add-or-change-users-who-can-manage-a-reservation"></a>Agregar o cambiar los usuarios que pueden administrar una reserva

Para delegar la administración de una reserva, agregue usuarios a roles en el pedido de reserva o en la reserva. De forma predeterminada, la persona que crea el pedido de reserva y el administrador de cuenta tienen el rol de propietario en el pedido de reserva y en la reserva.

Puede administrar el acceso a los pedidos de reserva y a las reservas *de forma independiente de las suscripciones* que obtienen el descuento de reserva. El hecho de que a un usuario se le concedan permisos para administrar un pedido de reserva o la reserva misma, no implica que también se le otorguen derechos para administrar la suscripción. Igualmente, si a un usuario se le conceden permisos para administrar una suscripción dentro del ámbito de la reserva, no significa que se le otorguen derechos para administrar el pedido de reserva o la reserva.

Para llevar a cabo un intercambio o un reembolso, el usuario tiene que tener acceso al pedido de reserva. Al conceder permisos, es mejor concederlos para el pedido de reserva y no para la reserva.

Para delegar la administración de acceso en una reserva:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** > **Reservations** para enumerar las reservas a las que tiene acceso.
3. Seleccione la reserva cuyo acceso quiere delegar a otros usuarios.
4. Seleccione **Access Control (IAM)** .
5. Seleccione **Agregar asignación de roles** > **Rol** > **Propietario**. U otro rol si quiere conceder acceso limitado.
6. Escriba la dirección de correo electrónico del usuario al que desea agregar como propietario.
7. Seleccione el usuario y, después, **Guardar**.

## <a name="split-a-single-reservation-into-two-reservations"></a>División de una reserva única en dos

 Si se compran varias instancias de recursos en una reserva, se pueden asignar algunas de las instancias de la reserva a distintas suscripciones. De forma predeterminada, todas las instancias tienen un ámbito, que puede ser de suscripción única o compartida. Pongamos por ejemplo que se han comprado diez instancias de reserva y como ámbito se ha especificado la suscripción A. En caso de que le interese, se puede establecer el ámbito de siete reservas en la suscripción A y el de las tres restantes en la suscripción B. Dividir una reserva le permite distribuir las instancias para lograr una administración más pormenorizada del ámbito. Si quiere simplificar la asignación a las suscripciones, elija el ámbito compartido. Sin embargo, para la administración de costos o fines presupuestarios, puede asignar las cantidades a suscripciones concretas.

 Una reserva se puede dividir en dos mediante PowerShell, CLI o la API.

### <a name="split-a-reservation-by-using-powershell"></a>División de una reserva con PowerShell

1. Ejecute el comando siguiente para obtener el identificador de pedido de reserva:

    ```powershell
    # Get the reservation orders you have access to
    Get-AzReservationOrder
    ```

2. Obtenga los detalles de una reserva:

    ```powershell
    Get-AzReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId b8be062a-fb0a-46c1-808a-5a844714965a
    ```

3. Divida la reserva en dos y distribuya las instancias:

    ```powershell
    # Split the reservation. The sum of the reservations, the quantity, must equal the total number of instances in the reservation that you're splitting.
    Split-AzReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId b8be062a-fb0a-46c1-808a-5a844714965a -Quantity 3,2
    ```
4. Si quiere actualizar el ámbito, ejecute el siguiente comando:

    ```powershell
    Update-AzReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId 5257501b-d3e8-449d-a1ab-4879b1863aca -AppliedScopeType Single -AppliedScope /subscriptions/15bb3be0-76d5-491c-8078-61fe3468d414
    ```

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](exchange-and-refund-azure-reservations.md).

## <a name="change-optimize-setting-for-reserved-vm-instances"></a>Cambiar la configuración de optimización para instancias reservadas de máquina virtual

 Al comprar una instancia reservada de máquina virtual, puede elegir entre la flexibilidad de tamaño de instancia o la prioridad de capacidad. En la opción de flexibilidad del tamaño de instancia se aplica el descuento por la reserva a otras máquinas virtuales del mismo [grupo](https://aka.ms/RIVMGroups). La prioridad de capacidad da preferencia a la capacidad del centro de datos para las implementaciones. Esta opción le ofrece una mayor confianza en su capacidad para iniciar instancias de máquinas virtuales cuando las necesite.

De forma predeterminada, cuando se comparte el ámbito de la reserva, la flexibilidad de tamaño de instancia está activada. La capacidad del centro de datos no tiene prioridad en las implementaciones de máquina virtual.

En las reservas donde el ámbito es único, puede optimizar la reserva para la prioridad de capacidad en lugar de la flexibilidad del tamaño de instancia de máquina virtual.

Para actualizar la configuración de optimización de la reserva:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** > **Reservations**.
3. Seleccione la reserva.
4. Seleccione **Configuración** > **Configuración**.  
  ![Ejemplo que muestra el elemento Configuración](./media/manage-reserved-vm-instance/add-product03.png)
5. Cambie la configuración de la **optimización**.  
  ![Ejemplo que muestra la configuración Optimizar para](./media/manage-reserved-vm-instance/instance-size-flexibility-option.png)

## <a name="optimize-reservation-use"></a>Optimización del uso de reservas

El ahorro de reservas de Azure solo se produce al usar recursos sostenidos. Cuando realiza una compra de reserva, paga por lo que es esencialmente el 100 % de uso de recursos durante un período de uno o tres años. Trate de maximizar la reserva para obtener el mayor uso y ahorros posibles. En las secciones siguientes se explica cómo supervisar una reserva y optimizar su uso.

### <a name="view-reservation-use-in-the-azure-portal"></a>Visualización del uso de las reservas en Azure Portal

Una manera de ver el uso de las reservas es mediante Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Todos los servicios** > [**Reservations**](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade) y anote el valor de **Uso (%)** de una reserva.  
  ![Imagen que muestra la lista de reservas](./media/manage-reserved-vm-instance/reservation-list.png)
3. Seleccione una reserva.
4. Revise la tendencia de uso de la reserva a lo largo del tiempo.  
  ![Imagen que muestra el uso de la reserva ](./media/manage-reserved-vm-instance/reservation-utilization-trend.png)

### <a name="view-reservation-use-with-api"></a>Visualización del uso de las reservas con la API

Si es un cliente con Contrato Enterprise (EA), puede ver mediante programación cómo se usan las reservas en su organización. Obtiene las reservas sin usar mediante los datos de uso. Al revisar los cargos de la reserva, tenga en cuenta que los datos se dividen entre el costo real y los costos amortizados. El costo real proporciona datos para conciliar la factura mensual. También tiene los costos de compra de reserva y los detalles de solicitud de reservas. El costo amortizado es como el costo real, salvo que se prorratea el precio efectivo para el uso de reserva. Las horas de reserva sin usar se muestran en datos de costos amortizados. Para más información sobre los datos de uso para los clientes de Contrato Enterprise, consulte [Obtención del uso y los costos de reservas de Contrato Enterprise](understand-reserved-instance-usage-ea.md).

Para otros tipos de suscripciones, use [Reservations Summaries - List By Reservation Order And Reservation](/rest/api/consumption/reservationssummaries/listbyreservationorderandreservation) API.

### <a name="optimize-your-reservation"></a>Optimización de la reserva

Si ve que las reservas de su organización se están utilizando por debajo de su capacidad:

- Asegúrese de que las máquinas virtuales que crea su organización coincidan con el tamaño de la máquina virtual que se encuentra en la reserva.
- Asegúrese de que la flexibilidad de tamaño de instancia esté activada. Para obtener más información, consulte [Administrar reservas: cambiar la configuración de optimización para instancias reservadas de máquina virtual](#change-optimize-setting-for-reserved-vm-instances).
- Cambie el ámbito de la reserva para que sea _compartido_ y así poder aplicarlo de manera más amplia. Para más información, consulte [Cambio del ámbito en una reserva](#change-the-reservation-scope).
- Considere la posibilidad de intercambiar la cantidad no utilizada. Para más información, consulte [Cancelaciones y cambios](#cancel-exchange-or-refund-reservations).


## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

- [¿Qué son las reservas para Azure?](save-compute-costs-reservations.md)

Compra de un plan de servicio:
- [Pago por adelantado de máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Pago por adelantado de los recursos de proceso de SQL Database con capacidad reservada de Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
- [Pago por adelantado para recursos de Azure Cosmos DB con capacidad reservada de Azure Cosmos DB](../../cosmos-db/cosmos-db-reserved-capacity.md)

Compra de un plan de software:
- [Pago por adelantado para planes de software de Red Hat con reservas de Azure](../../virtual-machines/linux/prepay-rhel-software-charges.md)
- [Pago por adelantado para planes de software SUSE con Azure Reservations](../../virtual-machines/linux/prepay-suse-software-charges.md)

Comprender descuentos y uso:
- [Información sobre cómo se aplica el descuento por la reserva de máquinas virtuales](../manage/understand-vm-reservation-charges.md)
- [Información sobre cómo se aplica el descuento del plan de reserva de software de Red Hat Linux Enterprise para Azure](understand-rhel-reservation-charges.md)
- [Descubra cómo se aplica el descuento del plan de software SUSE Linux Enterprise](understand-suse-reservation-charges.md)
- [Descubra cómo se aplican otros descuentos por reservas](understand-reservation-charges.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
- [Costos de software de Windows no incluidos con Reservations](reserved-instance-windows-software-costs.md)
