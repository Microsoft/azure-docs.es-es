---
title: Adquisición de una reserva de Azure
description: Para comprar una reserva de Azure, es importante que conozca algunos aspectos importantes.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: 211e06695d3cabef8cfeb0a7bbf581835819345c
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686214"
---
# <a name="buy-a-reservation"></a>Adquisición de una reserva

Las reservas de Azure le ayudan a ahorrar dinero, ya que se compromete a planes de uno o tres años para muchos productos de Azure. Antes de entrar en un compromiso para comprar una reserva, asegúrese de revisar las secciones siguientes para prepararse para ello.

## <a name="who-can-buy-a-reservation"></a>Quién puede comprar una reserva

Para comprar un plan, debe tener un rol de propietario de la suscripción en una suscripción Enterprise (MS-AZR-0017P o MS-AZR-0148P) o de pago por uso (MS-AZR-0003P o MS-AZR-0023P), o bien una suscripción de Microsoft Customer Agreement. Los proveedores de soluciones en la nube pueden usar Azure Portal o el  [Centro de partners](/partner-center/azure-reservations) para comprar reservas de Azure.

Los clientes de Contrato Enterprise (EA) pueden limitar sus compras a los administradores de EA mediante la desactivación de la opción **Add Reserved Instances** (Agregar instancias reservadas) en EA Portal. Para poder comprar una reserva, los administradores de Contrato Enterprise deben ser propietarios de la suscripción para al menos una suscripción de Contrato Enterprise. La opción es útil para empresas que requieren un equipo centralizado para comprar reservas para distintos centros de coste. Después de la compra, los equipos centralizados pueden agregar propietarios de centros de coste a las reservas. Los propietarios pueden entonces ampliar la reserva a sus suscripciones. El equipo central no necesita tener acceso al propietario de la suscripción donde se compra la reserva.

Los descuentos por reserva se aplican solo a los recursos asociados con las suscripciones compradas a través de un contrato Enterprise, Proveedor de soluciones en la nube (CSP), Microsoft Customer Agreement y de planes individuales con tarifas de pago por uso.

## <a name="scope-reservations"></a>Ámbito de las reservas

Puede limitar el ámbito de una reserva a una suscripción o a grupos de recursos. Al limitar el ámbito de una reserva, se selecciona el lugar en el que se aplican los ahorros de la reserva. Al limitar el ámbito de la reserva a un grupo de recursos, los descuentos por reserva solo se aplican al grupo de recursos, no a la suscripción completa.

### <a name="reservation-scoping-options"></a>Opciones de limitación del ámbito de la reserva

Tiene tres opciones para limitar el ámbito de una reserva, según sus necesidades:

- **Single resource group scope** (Ámbito de grupo de recursos único): El descuento por reserva se aplica a los recursos coincidentes solo del grupo de recursos seleccionado.
- **Single subscription scope** (Ámbito de suscripción única): El descuento por reserva se aplica a los recursos coincidentes de la suscripción seleccionada.
- **Ámbito compartido**: aplica el descuento por reserva a los recursos coincidentes en suscripciones aptas que están en el contexto de facturación.
    - Para los clientes con Contrato Enterprise, el contexto de facturación es la inscripción. El ámbito compartido de la reserva incluiría varios inquilinos de Active Directory en una inscripción.
    - En el caso de los clientes con contrato Microsoft Customer Agreement, el ámbito de facturación es el perfil de facturación.
    - En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.

Al aplicar los descuentos por reserva sobre su uso, Azure procesa la reserva en el orden siguiente:

1. Reservas cuyo ámbito es un grupo de recursos
2. Reservas de ámbito único
3. Reservas de ámbito compartido

También puede actualizar el ámbito después de comprar una reserva. Para ello, vaya a la reserva, haga clic en **Configuración** y limite de nuevo el ámbito de la reserva. Volver a limitar el ámbito de una reserva no es una transacción comercial. No se cambian las condiciones de la reserva. Para más información sobre cómo actualizar el ámbito, consulte [Update the scope after you purchase a reservation](manage-reserved-vm-instance.md#change-the-reservation-scope) (Actualización del ámbito después de comprar una reserva).

![Ejemplo que muestra un cambio en el ámbito de la reserva](./media/prepare-buy-reservation/rescope-reservation-resource-group.png)

## <a name="discounted-subscription-and-offer-types"></a>Tipos de ofertas y suscripciones con descuento

Los descuentos por reserva se aplican a las siguientes suscripciones aptas y tipos de ofertas.

- Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P)
- Suscripciones del contrato Microsoft Customer Agreement.
- Planes individuales con tarifas de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P)
- Suscripciones de CSP

Los recursos que se ejecutan en una suscripción con otros tipos de oferta no recibirán el descuento por la reserva.

## <a name="purchase-reservations"></a>Compra de reservas

Puede comprar reservas en Azure Portal, las API, PowerShell o la CLI. Lea los siguientes artículos que correspondan a su caso cuando esté preparado para hacer una compra de reservas:

- [App Service](prepay-app-service-isolated-stamp.md)
- [Azure Cache for Redis](../../azure-cache-for-redis/cache-reserved-pricing.md)
- [Cosmos DB](../../cosmos-db/cosmos-db-reserved-capacity.md)
- [Databricks](prepay-databricks-reserved-capacity.md)
- [Data Explorer](/azure/data-explorer/pricing-reserved-capacity)
- [Disk Storage](../../virtual-machines/disks-reserved-capacity.md)
- [Host dedicado](../../virtual-machines/prepay-dedicated-hosts-reserved-instances.md)
- [Planes de software](../../virtual-machines/linux/prepay-suse-software-charges.md)
- [Storage](../../storage/blobs/storage-blob-reserved-capacity.md)
- [SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
- [Azure Database para PostgreSQL](../../postgresql/concept-reserved-pricing.md)
- [Azure Database for MySQL](../../mysql/concept-reserved-pricing.md)
- [Azure Database for MariaDB](../../mariadb/concept-reserved-pricing.md)
- [Azure Synapse Analytics](prepay-sql-data-warehouse-charges.md)
- [Máquinas virtuales](../../virtual-machines/windows/prepay-reserved-vm-instances.md)

## <a name="buy-reservations-with-monthly-payments"></a>Compra de reservas con pagos mensuales

Ahora puede pagar las reservas mensualmente. A diferencia de la compra por adelantado (importe total), la opción de pago mensual divide el costo de la reserva en partes iguales cada mes del período. El costo total de las reservas por adelantado y mensuales es el mismo y no se pagan cargos adicionales por elegir el pago mensual.

Si la reserva se adquiere mediante un contrato de cliente de Microsoft (MCA), el importe del pago mensual puede variar en función de la tasa de cambio actual del mercado correspondiente a su moneda local.

Los pagos mensuales no están disponibles para: Databricks, reservas de SUSE Linux, planes de Red Hat y licencias de Red Hat OpenShift en Azure.

### <a name="view-payments-made"></a>Visualización de los pagos realizados

Puede ver los pagos realizados mediante las API, los datos de uso y el análisis de costos. En el caso de las reservas de pago mensual, el valor de la frecuencia aparece como **periódico** en los datos de uso y en Reservation Charges API. En el caso de las reservas pagadas por adelantado, el valor se muestra como **OneTime**.

El análisis de costos muestra las compras mensuales en la vista predeterminada. Aplique el filtro **compra** en **Charge type** (Tipo de cargo) y **periódico** en **Frecuencia** para ver todas las compras. Para ver solo las reservas, aplique un filtro de **Reserva**.

![Ejemplo que muestra los costos de las reservas en el análisis de costos](./media/prepare-buy-reservation/cost-analysis.png)

### <a name="exchange-and-refunds"></a>Cambios y reembolsos

Al igual que otras reservas, con la facturación mensual se pueden reembolsar o cambiar. 

Al cambiar una reserva pagada mensualmente, el costo total de la nueva compra debe ser mayor que los pagos restantes que se cancelan de la reserva devuelta. No hay ningún otro límite ni tarifa por los cambios. Puede cambiar una reserva pagada por adelantado para comprar una nueva que se facture mensualmente. Sin embargo, el valor de la duración de la nueva reserva debe ser mayor que el prorrateado de la reserva que se va a devolver.

Si cancela una reserva que se paga mensualmente, los pagos futuros cancelados se acumularán hasta el límite de reembolso de 50 000 USD.

Para más información sobre los cambios y los reembolsos, consulte [Autoservicio de cambios y reembolsos de reservas de Azure](exchange-and-refund-azure-reservations.md).

## <a name="reservation-notifications"></a>Notificaciones de reserva

En función de cómo paga la suscripción de Azure, se enviarán notificaciones de reservas por correo electrónico a los siguientes usuarios de su organización. Se envían notificaciones para varios eventos, entre los que se incluyen: 

- Purchase
- Próxima expiración de la reserva
- Expiry
- Renovación
- Cancelación
- Cambio de ámbito

Para clientes con suscripciones de EA:

- Las notificaciones solo se envían a los contactos de notificación de EA.
- Los usuarios que se agregan a una reserva a través del permiso de RBAC (IAM) no reciben ninguna notificación por correo electrónico.

Para clientes con suscripciones individuales:

- El comprador recibe una notificación de compra.
- En el momento de la compra, el propietario de la cuenta de facturación de suscripción recibe una notificación de compra.
- El propietario de la cuenta recibe todas las demás notificaciones.

## <a name="next-steps"></a>Pasos siguientes

- [Administración de reservas para los recursos de Azure](manage-reserved-vm-instance.md)
