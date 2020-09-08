---
author: yashesvi
ms.author: banders
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 07/15/2020
ms.openlocfilehash: fb3282666362e3ef592f253405cff01a52941203
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88655280"
---
Cuando haga "commit" a una instancia reservada de VM de Azure, puede ahorrar dinero. El descuento de la reserva se aplica automáticamente el número de máquinas virtuales en ejecución que coincidan con el ámbito y los atributos de la reserva. No es necesario asignar una reserva a una máquina virtual para obtener los descuentos. Una compra de instancia reservada cubre solo la parte de proceso del uso de la máquina virtual. En el caso de las máquinas virtuales Windows, el medidor de uso se divide en dos medidores independientes. Hay un medidor de proceso, que es el mismo que el medidor de Linux, y un medidor de IP de Windows. Los cargos que verá al hacer la compra son solo por los costos de proceso. Los cargos no incluyen los costos de software de Windows. Para obtener más información sobre los costos de software, consulte los [costos de software no incluidos en Azure Reserved Virtual Machine Instances](../articles/cost-management-billing/reservations/reserved-instance-windows-software-costs.md).

## <a name="determine-the-right-vm-size-before-you-buy"></a>Determinación del tamaño adecuado de una máquina virtual antes de su adquisición

Antes de adquirir una reserva, debe determinar el tamaño de la máquina virtual que necesita. Las siguientes secciones le ayudarán a determinar el tamaño correcto de la máquina virtual.

### <a name="use-reservation-recommendations"></a>Usar recomendaciones de reserva

Puede consultar las recomendaciones de reserva para averiguar las reservas que debe adquirir.

- Se mostrarán recomendaciones de compra y la cantidad recomendada al adquirir una instancia reservada de máquina virtual en Azure Portal.
- Azure Advisor proporciona recomendaciones de compra de suscripciones individuales.  
- Puede usar las API para obtener recomendaciones de compra relativas tanto a los ámbitos de suscripción tanto compartida como única. Para más información, vea [Recommendations API de compra de instancia reservada para clientes empresariales](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation).
- En el caso de los clientes de Contrato Enterprise (EA) y del Contrato de cliente de Microsoft (MCA), las recomendaciones de compra de los ámbitos de suscripción tanto única como compartida solo están disponibles con el [paquete de contenido de Power BI de Azure Consumption Insights](/power-bi/service-connect-to-azure-consumption-insights).

### <a name="services-that-get-vm-reservation-discounts"></a>Servicios que obtienen descuentos de reserva de máquina virtual

Puede aplicar sus reservas de máquina virtual al uso de máquinas virtuales emitidas desde varios servicios, no solo a sus implementaciones de máquinas virtuales. Los recursos que obtienen descuentos de reserva cambian en función de la configuración de flexibilidad de tamaño de instancia.

#### <a name="instance-size-flexibility-setting"></a>Configuración de flexibilidad de tamaño de instancia

La configuración de flexibilidad de tamaño de instancia determina qué servicios obtienen los descuentos de la instancia reservada.

Tanto si la configuración está activada como si no, los descuentos de reserva se aplican automáticamente a cualquier uso de máquina virtual que coincida cuando *ConsumedService* sea `Microsoft.Compute`. Por lo tanto, compruebe los datos de uso del valor *ConsumedService*. Estos son algunos ejemplos:

- Máquinas virtuales
- Conjuntos de escalado de máquinas virtuales
- Servicio de contenedor
- Implementaciones de Azure Batch (en modo de suscripciones de usuario)
- Azure Kubernetes Service (AKS)
- Service Fabric

Cuando la configuración está activada, los descuentos de reserva se aplican automáticamente al uso de máquina virtual que coincida cuando *ConsumedService* sea alguno de los elementos siguientes:

- Microsoft.Compute
- Microsoft.ClassicCompute
- Microsoft.Batch
- Microsoft.MachineLearningServices
- Microsoft.Kusto

Compruebe el valor *ConsumedService* en los datos de uso para determinar si el uso puede optar a los descuentos de reserva.

Para más información sobre la flexibilidad de tamaño de instancia, vea [Flexibilidad en el tamaño de las máquinas virtuales con Azure Reserved VM Instances](../articles/virtual-machines/reserved-vm-instance-size-flexibility.md).

### <a name="analyze-your-usage-information"></a>Analizar la información de uso

Analice su información de uso para averiguar qué reservas debe adquirir. Los datos de uso están disponibles en el archivo de uso y en las API. Úselos de manera conjunta para determinar qué reserva adquirir. Para determinar la cantidad de reservas que necesita adquirir, compruebe si hay instancias de máquina virtual que tengan un uso elevado diario. No tenga en cuenta la subcategoría `Meter` ni los campos `Product` de los datos de uso, ya que no distinguen los distintos tamaños de máquina virtual que usan Premium Storage. Si usa estos campos para determinar el tamaño de máquina virtual al adquirir la reserva, puede correr el riesgo de adquirir un tamaño equivocado, y no obtendrá el descuento de reserva que espera. En su lugar, consulte el campo `AdditionalInfo` del archivo o la API de uso para determinar el tamaño apropiado de la máquina virtual.

El archivo de uso proporciona los cargos por período de facturación y el uso diario. Para más información sobre cómo descargar el archivo de uso, vea [Visualización y descarga de los datos de uso y los cargos de Azure](../articles/cost-management-billing/understand/download-azure-daily-usage.md). Tras ello, una vez posea la información del archivo de uso, podrá [determinar qué reserva comprar](../articles/cost-management-billing/reservations/determine-reservation-purchase.md).

### <a name="purchase-restriction-considerations"></a>Consideraciones sobre restricciones de compra

Las instancias reservadas de máquinas virtuales están disponibles para la mayoría de los tamaños de máquina virtual, a excepción de los indicados aquí. El descuento de la reserva no se aplica en las siguientes máquinas virtuales:

- **Series de máquinas virtuales**: serie A, serie Av2 o serie G.

- **Máquinas virtuales de versión preliminar o promocionales**: cualquier serie o tamaño de máquina virtual que se encuentra en versión preliminar o usa el medidor promocional.

- **Nubes**: las reservas no están disponibles para la compra en las regiones de Alemania o China.

- **Cuota insuficiente**: una reserva cuyo ámbito sea de una sola suscripción debe tener cuota de vCPU disponible en la suscripción para la nueva instancia reservada. Por ejemplo, si la suscripción de destino tiene un límite de cuota de 10 vCPU para la serie D, no podrá comprar una reserva para 11 instancias Standard_D1. La comprobación de cuota para las reservas incluye las máquinas virtuales ya implementadas en la suscripción. Por ejemplo, si la suscripción tiene una cuota de 10 vCPU para la serie D y tiene implementadas dos instancias standard_D1, podrá comprar una reserva para 10 instancias standard_D1 en esta suscripción. También puede [cursar una solicitud de aumento de la cuota](../articles/azure-portal/supportability/resource-manager-core-quotas-request.md) para resolver este problema.

- **Restricciones de capacidad**: en algunas circunstancias poco frecuentes, Azure limita la compra de nuevas reservas a un subconjunto de tamaños de máquina virtual debido a que la capacidad en una región es baja.

## <a name="buy-a-reserved-vm-instance"></a>Compra de una instancia reservada de máquina virtual

Puede comprar una instancia reservada de máquina virtual en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22VirtualMachines%22%7D). Pague la reserva [por adelantado o mensualmente](../articles/cost-management-billing/reservations/monthly-payments-reservations.md).
Estos requisitos se aplican a la compra de una instancia reservada de máquina virtual:

- Debe tener un rol de propietario en al menos una suscripción de EA o en una suscripción con una tarifa de pago por uso.
- En el caso de las suscripciones de EA, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.
- En el caso del programa Proveedor de soluciones en la nube (CSP), solo los agentes de administración o de ventas pueden adquirir reservas.

Para comprar una instancia:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** > **Reservations**.
1. Seleccione **Agregar** para comprar una nueva reserva y, luego, haga clic en **Máquina virtual**.
1. Rellene los campos obligatorios. Las instancias de máquina virtual en ejecución que coinciden con los atributos seleccionados cumplen los requisitos para obtener el descuento de la reserva. El número real de las instancias de máquina virtual que obtienen el descuento depende el ámbito y la cantidad seleccionada.

Si tiene un Contrato Enterprise, puede usar la opción **Agregar más** para agregar rápidamente instancias adicionales. La opción no está disponible para otros tipos de suscripciones.


| Campo      | Descripción|
|------------|--------------|
|Subscription|Suscripción que se usa para pagar la reserva. Los costos de la reserva se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Contrato de cliente de Microsoft o una suscripción individual con tarifas de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P). Los cargos se deducen del saldo de compromiso monetario, si está disponible, o se cobran como uso por encima del límite. En una suscripción con tarifas de pago por uso, los cargos se cobran con el método de pago de factura o la tarjeta de crédito de la suscripción.|    
|Ámbito       |El ámbito de la reserva puede cubrir una o varias suscripciones (ámbito compartido). Si selecciona: <ul><li>**Single resource group scope** (Ámbito de grupo de recursos único): aplica el descuento por reserva a los recursos coincidentes solo en el grupo de recursos seleccionado.</li><li>**Single subscription scope** (Ámbito de suscripción única): aplica el descuento por reserva a los recursos coincidentes de la suscripción seleccionada.</li><li>**Ámbito compartido**: aplica el descuento por reserva a los recursos coincidentes en suscripciones aptas que están en el contexto de facturación. Para los clientes de EA, el contexto de facturación es la inscripción. En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.</li></ul>|
|Region    |Región de Azure que está cubierta por la reserva.|    
|Tamaño de VM     |Tamaño de las instancias de máquina virtual.|
|Optimizar para     |La flexibilidad de tamaño de instancia de máquina virtual está seleccionada de forma predeterminada. Haga clic en **Configuración avanzada** para cambiar el valor de flexibilidad de tamaño de instancia, con el fin de aplicar el descuento de reserva a otras VM del mismo [grupo de tamaño de VM](../articles/virtual-machines/reserved-vm-instance-size-flexibility.md). La prioridad de capacidad da preferencia a la capacidad del centro de datos para las implementaciones. Esto ofrece una mayor confianza en su capacidad para iniciar instancias de máquinas virtuales cuando las necesite. La prioridad de capacidad solo está disponible si el ámbito de la reserva es de suscripción única. |
|Término        |Un año o tres años. También hay un período de 5 años disponible solo para las máquinas virtuales HBv2.|
|Cantidad    |Número de instancias que se compran dentro de la reserva. La cantidad es el número de instancias de máquina virtual en ejecución a las que se aplica el descuento de facturación. Por ejemplo, si ejecuta 10 máquinas virtuales Standard_D2 en la región Este de EE. UU., debería especificar 10 como cantidad para maximizar el beneficio de todas las máquinas virtuales en ejecución. |

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2PjmT]

## <a name="usage-data-and-reservation-utilization"></a>Datos de uso y utilización de la reserva

Los datos de uso que obtienen un descuento de reserva tienen un precio efectivo de cero. Puede ver qué instancia de máquina virtual recibió el descuento de reserva para cada reserva.

Para obtener más información sobre cómo aparecen los descuentos de reserva en los datos de uso, consulte [Obtención del uso y los costos de reservas de Contrato Enterprise](../articles/cost-management-billing/reservations/understand-reserved-instance-usage-ea.md) si es un cliente de EA. Si tiene una suscripción individual, consulte [Descripción del uso de reservas para su suscripción individual de pago por uso](../articles/cost-management-billing/reservations/understand-reserved-instance-usage.md).

## <a name="change-a-reservation-after-purchase"></a>Cambiar una reserva después de la compra

Puede realizar los siguientes tipos de cambios en una reserva después de haberla comprado:

- Actualización del ámbito de la reserva
- Flexibilidad del tamaño de instancia (si procede)
- Propiedad

También puede dividir una reserva en fragmentos más pequeños y combinar reservas ya divididas. Ninguno de estos cambios conlleva una nueva transacción comercial ni un cambio en la fecha de finalización de la reserva.

No se pueden realizar los siguientes tipos de cambios tras la compra:

- La región de una reserva existente
- SKU
- Cantidad
- Duration

Pero sí se puede *intercambiar* una reserva si quiere realizar cambios.

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](../articles/cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo administrar una reserva, consulte [Administración de Azure Reservations](../articles/cost-management-billing/reservations/manage-reserved-vm-instance.md).
- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
    - [¿Qué es Azure Reservations?](../articles/cost-management-billing/reservations/save-compute-costs-reservations.md)
    - [Administración de reservas en Azure](../articles/cost-management-billing/reservations/manage-reserved-vm-instance.md)
    - [Información sobre cómo se aplica el descuento por la reserva](../articles/cost-management-billing/manage/understand-vm-reservation-charges.md)
    - [Información sobre el uso de reservas de Azure para suscripciones de pago por uso](../articles/cost-management-billing/reservations/understand-reserved-instance-usage.md)
    - [Información sobre el uso de reservas para la inscripción Enterprise](../articles/cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)
    - [Costos de software de Windows no incluidos con reservas](../articles/cost-management-billing/reservations/reserved-instance-windows-software-costs.md)
    - [Azure Reservations en el programa del Proveedor de soluciones en la nube (CSP) del Centro de partners](https://docs.microsoft.com/partner-center/azure-reservations)
