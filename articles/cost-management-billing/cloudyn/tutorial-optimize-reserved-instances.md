---
title: 'Tutorial: Optimización de los costos de las instancias reservadas con Cloudyn en Azure'
description: En este tutorial, aprenderá a optimizar los costos de las instancias reservadas para Azure y Amazon Web Services (AWS).
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ms.reviewer: benshy
ROBOTS: NOINDEX
ms.openlocfilehash: 8488b1f511a3b269f36f3dfe7a351765a4e312aa
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88687882"
---
<!-- Intent: As a cloud-consuming administrator, I need to ensure that my reserved instances are optimized for cost and usage
-->

# <a name="tutorial-optimize-reserved-instances"></a>Tutorial: Optimización de instancias reservadas

En este tutorial, aprenderá cómo Cloudyn le ayuda a optimizar los costos de las instancias reservadas para Azure y Amazon Web Services (AWS) y su uso. Una instancia reservada con cualquier proveedor de servicios en la nube es un contrato a largo plazo de compromiso por adelantado por el uso futuro de la máquina virtual. Además, puede ofrecer ahorros considerables en comparación con el modelo de precios de máquina virtual de pago por uso estándar. Los ahorros solo se aprecian al usar toda la capacidad de las instancias reservadas.

Este tutorial explica cómo Cloudyn mantiene las instancias reservadas para Azure y AWS. También describe cómo se optimizan los costos de las instancias reservadas. Principalmente, al garantizar que las reservas funcionan a la máxima capacidad. En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Descripción de los costos de las instancias reservadas para Azure
> * Más información sobre las ventajas de las instancias reservadas
> * Optimización de los costos de las instancias reservadas para Azure
> * Visualización de los costos de las instancias reservadas
> * Evaluación de la rentabilidad de las instancias reservadas para Azure
> * Optimización de los costos de las instancias reservadas para AWS
> * Compra de instancias reservadas recomendadas
> * Modificación de las reservas sin usar

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="prerequisites"></a>Prerequisites

- Debe tener una cuenta de Azure.
- Debe tener un registro de prueba o una suscripción de pago en Cloudyn.
- Debe haber comprado instancias reservadas en Azure o AWS.

## <a name="understand-azure-ri-costs"></a>Descripción de los costos de las instancias reservadas para Azure

Al comprar Azure Reserved VM Instances, se paga por adelantado para el uso futuro. El pago por adelantado cubre el costo del uso futuro de las máquinas virtuales:

- de un tipo específico
- en una región específica
- durante un plazo de uno o tres años
- del máximo de máquinas virtuales compradas

Puede ver las instancias de Azure Reserved VM Instances adquiridas en Azure Portal, en [Reservations](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).

El término _instancia reservada de máquina virtual de Azure_ solo se aplica a un modelo de precios. No cambia las máquinas virtuales en ejecución en absoluto. El término es específico de Azure y se suele denominar _instancia reservada_ o _reserva_. Las instancias reservadas que haya comprado no se aplican a máquinas virtuales específicas, sino a cualquier máquina virtual que cumplan las condiciones. Por ejemplo, una reserva para un tipo de máquina virtual que se ejecuta en la región que eligió para la reserva que adquirió.

Las instancias reservadas adquiridas solo se aplican al hardware básico. No cubren las licencias de software de las máquinas virtuales. Por ejemplo, puede reservar una instancia y tiene una máquina virtual que cumple las condiciones y se ejecuta con Windows. La instancia reservada solo cubre el costo base de la máquina virtual. En este ejemplo, usted paga el precio total de las licencias de Windows necesarias. Para obtener un descuento en el sistema operativo u otro software que se ejecute en las máquinas virtuales, debe considerar el uso de [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit). Ventaja híbrida ofrece un tipo de descuento para las licencias de software similar al de las instancias reservadas para las máquinas virtuales base.

El uso de las instancias reservadas no afecta al costo directamente. En otras palabras, la ejecución de una máquina virtual al 100 % o al 0 % de capacidad de CPU tiene el mismo efecto: paga previamente la asignación de la máquina virtual, no su uso.

En la siguiente imagen veremos la asociación del uso estándar de máquinas virtuales a petición con los costos en lo relativo a las instancias reservadas:

![Costos a demanda frente a costos con instancias reservadas](./media/tutorial-optimize-reserved-instances/azure01.png)



Las barras de color rojo muestran el costo acumulado de la compra de instancias reservadas. El precio se paga solo una vez. El uso de las máquinas virtuales es gratuito. Las barras azules muestran el costo acumulado de la misma máquina virtual al ejecutarla con el modelo de precios de pago por uso y a demanda. En algún lugar entre el séptimo y el octavo mes de uso de la máquina virtual existe un *punto de equilibrio*. En este ejemplo, se empieza a ahorrar a partir del octavo mes.

## <a name="benefits-of-ris"></a>Ventajas de las instancias reservadas

Cada compra de instancias reservadas se aplica a una máquina virtual de tamaño y ubicación específicos. Por ejemplo, D2s\_v3, que se ejecuta en la ubicación del Oeste de EE. UU. como se muestra en la siguiente imagen:

![Detalles de las instancias reservadas de Azure](./media/tutorial-optimize-reserved-instances/azure02.png)

La compra de instancias reservadas pasa a ser beneficiosa cuando una máquina virtual se ejecuta durante las horas mínimas para alcanzar el punto de equilibrio de la reserva. La máquina virtual debe coincidir con el tamaño y la ubicación de la instancia reservada. Por ejemplo, en el gráfico anterior, el punto de equilibrio se encuentra aproximadamente a la mitad del séptimo mes. Por lo tanto, la compra es beneficiosa cuando la máquina virtual de la reserva se ejecuta al menos durante 7,5 meses \* 30 días \* 24 horas = 5400 horas. Si la máquina virtual correspondiente se ejecuta menos 5400 horas, la reserva cuesta más que el pago por uso.

El punto de equilibrio puede variar en función del tamaño y la ubicación de las máquinas virtuales. También depende de los precios de pago por uso negociados para la máquina virtual. Antes de realizar una compra, compruebe el punto de equilibrio aplicable a su caso.

Otro aspecto que considerar al adquirir la reserva es el ámbito de las instancias reservadas. El ámbito determina si se comparte el beneficio de la reserva o si se aplica a una suscripción específica. Las instancias reservadas compartidas se aplican aleatoriamente en todas las suscripciones a las primeras máquinas virtuales que cumplan las condiciones.

El ámbito de la adquisición compartida es más flexible y se recomienda siempre que sea posible. Las posibilidades de usar todas las instancias reservadas son mucho mayores con el ámbito compartido. Sin embargo, al pagar la instancia reservada, puede que el propietario de la suscripción solo pueda comprarla para una suscripción única.

## <a name="optimize-azure-ri-costs"></a>Optimización de los costos de las instancias reservadas para Azure

Cloudyn mantiene las instancias reservadas y Ventaja híbrida al:

- Mostrar los costos asociados con los modelos de precios
- Realizar el seguimiento de las instancias reservadas
- Evaluar el impacto de las instancias reservadas
- Asignar los costos de las instancias reservadas según las directivas

La primera medida que debe tomar antes de adquirir una instancia reservada es evaluar el impacto de su compra:

- ¿Cuánto costará?
- ¿Cuánto se ahorrará?
- ¿Cuál es el punto de equilibrio?

El informe de impacto de la compra de instancias reservadas puede ayudar a responder esas preguntas.

## <a name="assess-azure-ri-cost-effectiveness"></a>Evaluación de la rentabilidad de las instancias reservadas para Azure

En el portal de Cloudyn, vaya a **Optimizer** > **RI Comparison** (Optimizador > Comparación de instancias reservadas) y seleccione **Reserved Instance Purchase Impact** (Impacto de la compra de instancias reservadas).

En el informe de impacto de la compra de instancias reservadas, seleccione el tamaño de máquina virtual (tipo de instancia), la ubicación (región), el plazo de reserva, la cantidad y el tiempo de ejecución previsto. A continuación, evalúe si compra permite el ahorro.

Por ejemplo, si compra una reserva para una máquina virtual de tipo DS1\_v2 en el Este de EE. UU. y se ejecuta permanentemente durante un año entero, ahorraría 369,48 $ anuales. El punto de equilibrio es de cinco meses. Consulte la siguiente imagen:

![Punto de equilibrio de instancia reservada de Azure](./media/tutorial-optimize-reserved-instances/azure03.png)

Sin embargo, si se ejecuta solo el 50 % del tiempo, el punto de equilibrio se encontrará en los 10 meses y el ahorro será solo de 49,74 $ anuales. En este ejemplo, la compra de la reserva para ese tipo de instancia puede no ser beneficioso. Consulte la siguiente imagen:

![Ejemplo del punto de equilibrio para VM de Azure](./media/tutorial-optimize-reserved-instances/azure04.png)

## <a name="view-ri-costs"></a>Visualización de los costos de las instancias reservadas

Al comprar una reserva, el pago es único. Hay dos maneras de ver el pago en Cloudyn:

- Costo real
- Costo amortizado

### <a name="actual-reserved-instance-cost"></a>Costo real de las instancias reservadas

Los informes de análisis del costo real y a lo largo del tiempo muestran el importe total que ha pagado por la reserva, a partir del mes de compra. Además, ayudan a ver los gastos reales durante un período.

Vaya a **Costs** > **Cost Analysis** (Costos > Análisis del costo) en el portal de Cloudyn y seleccione **Actual Cost Analysis** (Análisis del costo real) o **Actual Cost Over Time** (Costo real a lo largo del tiempo). A continuación, establezca los filtros. Por ejemplo, filtre solo por el servicio Azure/máquina virtual y agrupe por tipo de recursos y modelo de precios. Consulte la siguiente imagen:

![Ejemplo de costo real de las instancias reservadas](./media/tutorial-optimize-reserved-instances/azure05.png)

Puede filtrar por servicio, **Azure/VM** en este ejemplo y agrupar por **modelo de precios** y **tipo de recurso** tal como se muestra en la siguiente imagen:

![Ejemplo de grupos y filtros del informe de costo real agrupados por modelo de precios y tipo de recurso](./media/tutorial-optimize-reserved-instances/azure06.png)

También puede analizar el tipo de pagos realizados, como los precios únicos, las tarifas por uso y los precios de licencia.

### <a name="amortized-reserved-instance-cost"></a>Costo amortizado de las instancias reservadas

Se paga un precio por adelantado, visible el mes de la compra de las instancias reservadas. No se ve en las facturas posteriores. Por lo tanto, el uso mensual puede resultar confuso. El mes cuesta realmente el uso mensual más la parte proporcional (amortizada) de los precios únicos realizados previamente. El informe de costo amortizado ayuda a ver la situación global.

Al calcular el costo amortizado de las instancias reservadas se considera el precio único y su amortización durante el plazo de la reserva. En los informes de costo real, los precios únicos se ven el mes de la compra de la reserva. Los gastos diarios y mensuales no aparecen en el costo real de implementación. Los informes de costo amortizado muestran el costo real de la implementación a lo largo del tiempo.  El informe de costo amortizado es la única manera de ver las tendencias de costo reales. También es la única manera de proyectar los gastos futuros.

En el informe de costo real, ha visto un pico por la compra de una instancia reservada el 16 de noviembre de 747 $. En el informe de costo amortizado (vea la imagen siguiente) hay un costo parcial del 16 de noviembre. A partir del 17 de noviembre verá el costo amortizado de la instancia reservada de 747/365 $ = 2,05 $. En consecuencia, también observará que la reserva adquirida no está en uso, puede cambiarla a un tamaño de máquina virtual distinto para optimizarla.

Para verlo, vaya a **Costs** > **Cost Analysis** (Costo > Análisis del costo), seleccione **Amortized Cost Analysis** (Análisis del costo amortizado) o **Amortized Cost Over Time** (Costo amortizado a lo largo del tiempo).

![Informe de ejemplo que muestra el costo amortizado de las instancias reservadas](./media/tutorial-optimize-reserved-instances/azure07.png)

## <a name="optimize-aws-ri-costs"></a>Optimización de los costos de las instancias reservadas para AWS

Las instancias reservadas son un compromiso abierto. Son útiles para el uso continuado de las máquinas virtuales, ya que las instancias reservadas son más económicas que las instancias a demanda. No obstante, deben utilizarse lo suficiente. El compromiso consiste en usar recursos, por lo general, las máquinas virtuales, durante un período definido, de uno o tres años. Al realizar el compromiso de compra, se pagan por adelantado los recursos con reserva. Sin embargo, puede que no siempre use todo para lo que se ha comprometido con la reserva.

Por ejemplo, puede evaluar su entorno y determinar que tenía 20 instancias D2 estándares que se ejecutaban constantemente durante el último año. Puede comprar una reserva para ellas y ahorrar considerablemente. En otro ejemplo, podría haberse comprometido al uso de diez instancias MA4 durante el año. Pero que solo haya usado cinco hasta el momento. Ambos ejemplos ilustran el uso ineficaz de instancias reservadas. Hay dos maneras de optimizar los costos de las instancias reservadas con los informes de optimización de Cloudyn:

- Revisión de las recomendaciones de la compra en función del uso histórico
- Modificación de las reservas sin usar

Con los informes _EC2 RI Buying Recommendations_ y _EC2 Currently Unused Reservations_ se mejoran el uso de las instancias reservadas y los costos.

## <a name="buy-recommended-ris"></a>Compra de instancias reservadas recomendadas

Cloudyn compara el uso de instancias a demanda con posibles instancias reservadas. Donde encuentra ahorro posible, las recomendaciones se muestran en el informe EC2 Buying Recommendations.

En el menú de informes de la parte superior del portal, haga clic en **Optimizer** > **Pricing Optimization** > **EC2 RI Buying Recommendations** (Optimizador > Optimización de precios > EC2 RI Buying Recommendations).

En la siguiente imagen se muestran las recomendaciones de compra del informe.

![Ejemplo que muestra recomendaciones de compra en el informe EC2 Buying Recommendations](./media/tutorial-optimize-reserved-instances/aws01.png)

En este ejemplo, la cuenta Cloudyn\_A tiene 32 recomendaciones de compra de instancia reservada. Si sigue todas las recomendaciones de compra, podría ahorrar 137 770 $ al año. Tenga en cuenta que las recomendaciones de compra que ofrece Cloudyn presuponen el uso continuo y coherente de las cargas de trabajo en ejecución.

Para ver los detalles de por qué se recomienda cada compra, haga clic en el signo más ( **+** ) de **Justifications** (Justificaciones). Este es un ejemplo de la primera recomendación de la lista.

![Ejemplo que muestra detalles de justificación de compra](./media/tutorial-optimize-reserved-instances/aws02.png)

En el ejemplo anterior se muestra que ejecutar la carga de trabajo a demanda costaría 90 456 $ al año. Sin embargo, si adquiere la reserva de antemano, la misma carga de trabajo costaría 56,592 $ y ahorraría 33.864 $ al año.

Haga clic en el signo más junto a **EC2 RI Purchase Impact** (Impacto de compra de instancia reservada para EC2) para ver el punto de equilibrio en un año desde la fecha aproximada desde que se realizara la compra. En el ejemplo siguiente, unos ocho meses después de realizar la compra, el costo acumulado a demanda empieza a superar el de la instancia reservada:

![Ejemplo que muestra detalles de impacto de compra](./media/tutorial-optimize-reserved-instances/aws03.png)

El ahorro comenzaría en ese momento.

Puede revisar **Instances over Time** (Instancias a lo largo del tiempo) para verificar la exactitud de la recomendación de compra sugerida. En este ejemplo, puede ver que, de media, se utilizaron seis instancias para la carga de trabajo en el último período de 30 días.

![Ejemplo que muestra el uso histórico de las instancias con el tiempo](./media/tutorial-optimize-reserved-instances/aws04.png)

## <a name="modify-unused-reservations"></a>Modificación de las reservas sin usar

Las reservas sin usar son habituales en muchos entornos informáticos de consumidor de recursos en la nube. Garantizar que las reservas sin usar se usan totalmente puede suponer un ahorro al modificar las reservas para que respondan a sus necesidades actuales. Por ejemplo, podría tener una suscripción con instancias D3 estándar que se ejecutan en Linux. Si no se utiliza completamente la reserva, puede cambiar el tipo de instancia. O bien, puede mover los recursos sin usar a una reserva diferente o a otra cuenta.

AWS vende instancias reservadas para zonas de disponibilidad y regiones específicas. Si ha adquirido instancias reservadas para una zona de disponibilidad concreta, no puede mover las reservas a otra zona. Sin embargo, se pueden mover fácilmente las instancias reservadas regionales de una zona que utilice el informe **EC2 Currently Unused Reservations** a otra. Como alternativa, puede modificarlas a un ámbito regional o y que se apliquen las instancias correspondientes a todas las zonas de disponibilidad.

En el menú de informes de la parte superior del portal, haga clic en **Optimizer** > **Inefficiencies** > **EC2 Currently Unused Reservations** (Optimizador > Ineficiencias > Reservas de EC2 sin usar actualmente).

En las siguientes imágenes se muestra el informe con las instancias reservadas sin usar.

![Ejemplo que muestra información resumida sobre las reservas sin usar](./media/tutorial-optimize-reserved-instances/unused-ri01.png)

Haga clic en el signo más de **Details** (Detalles) para ver la reserva de detalles para una reserva específica.

![Ejemplo que muestra detalles de las reservas sin usar](./media/tutorial-optimize-reserved-instances/unused-ri02.png)

En el ejemplo anterior, hay 77 reservas sin usar en distintas zonas de disponibilidad. La primera reserva tiene 51 instancias sin usar. Al mirar hacia abajo en la lista, hay posibles modificaciones de instancia de reserva que puede realizar con el tipo de instancia **m3.2xlarge** en la zona de disponibilidad **us-east-1c**.

Haga clic en **Modify** (Modificar) para que la primera reserva de la lista abra la página **Modify RI** (Modificar instancia reservada) con los datos de la reserva.

![Ejemplo que muestra las reservas que se pueden modificar](./media/tutorial-optimize-reserved-instances/unused-ri03.png)

Las instancias reservas que se pueden modificar se encuentran en una lista. En la imagen del ejemplo siguiente hay 51 reservas sin usar que se pueden modificar, pero entre las dos reservas se necesitan 54. Si modifica las reservas sin usar para usarlas todas, cuatro instancias continuarán ejecutándose a demanda. Para este ejemplo, se dividen las reservas sin usar de manera que la primera use 30 y la segunda, 21.

Haga clic en el signo más para la primera entrada de reserva y establezca el valor de **Reservation quantity** (Cantidad de reserva) en **30**. Para la segunda entrada, establezca la cantidad de reserva en **21** y haga clic en **Apply** (Aplicar).

![Ejemplo que muestra los cambios efectuados en la cantidad de reservas](./media/tutorial-optimize-reserved-instances/unused-ri04.png)

Todas las instancias sin usar de la reserva funcionan a máxima capacidad y 51 instancias ya no se ejecutan a demanda. En este ejemplo, la organización ahorra al reducir el uso a demanda considerablemente y emplear en su lugar las reservas que ya se han pagado.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha realizado correctamente las tareas siguientes:

> [!div class="checklist"]
> * Comprender los costos de las instancias reservadas para Azure
> * Aprender sobre las ventajas de las instancias reservadas
> * Optimizar los costos de las instancias reservadas para Azure
> * Visualizar los costos de las instancias reservadas
> * Evaluar la rentabilidad de las instancias reservadas para Azure
> * Optimizar los costos de las instancias reservadas para AWS
> * Comprar instancias reservadas recomendadas
> * Modificar las reservas sin usar


Vaya al siguiente tutorial para más información sobre el control del acceso a datos.

> [!div class="nextstepaction"]
> [Control del acceso a los datos](tutorial-user-access.md)
