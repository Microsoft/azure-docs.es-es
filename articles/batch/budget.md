---
title: Análisis de costos y presupuestos
description: Obtenga información acerca de cómo obtener un análisis de costos y establecer un presupuesto para los recursos de proceso subyacentes y las licencias de software que se usan para ejecutar las cargas de trabajo de Batch.
ms.topic: how-to
ms.date: 07/19/2019
ms.openlocfilehash: 50ca1ecfd0a973ff39dabfcb62175ce820d0a0d6
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654249"
---
# <a name="cost-analysis-and-budgets-for-azure-batch"></a>Análisis de costos y presupuestos para Azure Batch

No se aplica ningún cargo por Azure Batch, solo los recursos de proceso subyacentes y las licencias de software que se usan para ejecutar las cargas de trabajo de Batch. En un nivel alto, se incurre en costos desde las máquinas virtuales (VM) en un grupo, la transferencia de datos desde la máquina virtual o cualquier dato de entrada o salida almacenado en la nube. Examinemos algunos componentes clave de Batch para entender de dónde vienen los costos, cómo establecer un presupuesto para un grupo o una cuenta y algunas técnicas para hacer que las cargas de trabajo de Batch sean más rentables.

## <a name="batch-resources"></a>Recursos de Batch

Las máquinas virtuales son el recurso más importante que se usa para el procesamiento por lotes. El costo de usar máquinas virtuales para Batch se calcula en función del tipo, la cantidad y la duración del uso. Las opciones de facturación de las máquinas virtuales incluyen [Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o [reserva](../cost-management-billing/reservations/save-compute-costs-reservations.md) (pago por adelantado). Ambas opciones de pago tienen distintas ventajas en función de la carga de trabajo del proceso y ambos modelos de pago afectarán la factura de manera distinta.

Cuando se implementan aplicaciones en los nodos de Batch (máquinas virtuales) mediante [paquetes de aplicación](batch-application-packages.md), se le cobrarán los recursos de Azure Storage que los paquetes de aplicación consumen. También se le cobrará el almacenamiento de cualquier archivo de entrada o salida, como los archivos de recursos u otros datos de registro. En general, el costo de los datos de almacenamiento asociados con Batch es mucho menor que el costo de los recursos de proceso. Cada máquina virtual de un grupo que se crea con **VirtualMachineConfiguration** tiene un disco de sistema operativo asociado que usa discos administrados de Azure. Los discos administrados de Azure tienen un costo adicional y otros niveles de rendimiento de discos tienen también distintos costos.

Los grupos de Batch usan recursos de red. En concreto, para los grupos de **VirtualMachineConfiguration** se usan equilibradores de carga estándar, los que requieren direcciones IP estáticas. Los equilibradores de carga que Batch usa son visibles para las cuentas de **Suscripción del usuario**, pero no para las cuentas de **Servicio de Batch**. Los equilibradores de carga estándar incurren en gastos por todos los datos transmitidos desde y hacia las máquinas virtuales de los grupos de Batch. Las API de Batch selectas que recuperan datos desde los nodos de grupo (como Get Task/Node File), los paquetes de aplicación de tareas, los archivos de recursos o de salida y las imágenes de contenedor incurrirán en cargos.

### <a name="additional-services"></a>Servicios adicionales

Los servicios que no incluyen máquinas virtuales ni almacenamiento se pueden tener en cuenta para el costo de la cuenta de Batch.

Otros servicios que se usan habitualmente con Batch pueden incluir:

- Application Insights
- Data Factory
- Azure Monitor
- Virtual Network
- Máquinas virtuales con aplicaciones de gráficos

En función de los servicios que usa con la solución de Batch, es posible que incurra en tarifas adicionales. Consulte la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para determinar el costo de cada servicio adicional.

## <a name="cost-analysis-and-budget-for-a-pool"></a>Análisis de costos y presupuesto para un grupo

Mediante Azure Portal, puede crear presupuestos y alertas de gastos para la cuenta de Batch o los grupos de Batch. Los presupuestos y las alertas resultan útiles para notificar a las partes interesadas ante cualquier riesgo de gastos excesivos. Es posible que haya un retraso en las alertas de gastos y que se supere ligeramente un presupuesto. En este ejemplo, veremos el análisis de costos de un grupo de Batch individual.

1. En Azure Portal, seleccione **Administración de costos + facturación** en la barra de navegación de la izquierda.
1. Seleccione su suscripción en la sección **Mis suscripciones**.
1. Vaya a **Análisis de costos** en la sección **Administración de costos** de la barra de navegación de la izquierda, con lo que se mostrará una vista similar a la siguiente:
1. Seleccione **Agregar filtro**. En la primera lista desplegable, seleccione **Recurso** ![Selección del filtro de recurso](./media/batch-budget/resource-filter.png)
1. En la segunda lista desplegable, seleccione el grupo de Batch. Una vez que se seleccione el grupo, el análisis de costos se verá similar al análisis siguiente.
    ![Análisis de costos de un grupo](./media/batch-budget/pool-cost-analysis.png)

El análisis de costos resultante muestra el costo del grupo, así como los recursos que contribuyen a este costo. En este ejemplo, las máquinas virtuales que se usan en el grupo son el recurso más costoso.

Para crear un presupuesto para el grupo, seleccione **Budget: none** (Presupuesto: ninguno) y, luego, seleccione **Crear presupuesto >** . Ahora use la ventana para configurar un presupuesto específicamente para el grupo.

Para más información sobre cómo configurar un presupuesto, consulte [Creación y administración de presupuestos de Azure](../cost-management-billing/costs/tutorial-acm-create-budgets.md).

> [!NOTE]
> El servicio Azure Batch se basa en la tecnología de Azure Cloud Services y Azure Virtual Machines. Cuando elija **Configuración de Cloud Services**, se le cobrará en función de la estructura de precios de Cloud Services. Cuando elija **Configuración de la máquina virtual**, se le cobrará en función de la estructura de precios de Virtual Machines. En el ejemplo de esta página se usa la **configuración de la máquina virtual**.

## <a name="minimize-cost"></a>Minimización del costo

Usar varias máquinas virtuales y servicios de Azure durante largos períodos de tiempo puede ser costoso. Afortunadamente, hay disponibles servicios que permiten disminuir los gastos, así como estrategias para maximizar la eficacia de la carga de trabajo.

### <a name="low-priority-virtual-machines"></a>Máquinas virtuales de prioridad baja

Las máquinas virtuales de prioridad baja disminuyen el costo de las cargas de trabajo de Batch al usar la capacidad informática sobrante en Azure. Cuando se especifican máquinas virtuales de prioridad baja en los grupos, Batch usa este excedente para ejecutar la carga de trabajo. Usar máquinas virtuales de prioridad baja en lugar de máquinas virtuales dedicadas implica un ahorro de costos sustancial.

Obtenga más información sobre cómo configurar máquinas virtuales de prioridad baja para la carga de trabajo en [Uso de máquinas virtuales de prioridad baja con Batch](batch-low-pri-vms.md).

### <a name="virtual-machine-os-disk-type"></a>Tipo de disco de sistema operativo de máquina virtual

Existen varios [tipos de discos de sistema operativo de máquina virtual](../virtual-machines/disks-types.md). Muchas de las series de máquinas virtuales tienen tamaños que permiten almacenamiento premium y estándar. Cuando se selecciona un tamaño de máquina virtual "s" para un grupo, Batch configura discos de sistema operativo SSD Premium. Cuando se selecciona un tamaño de máquina virtual "distinto de s", se usa el tipo de disco HDD estándar más barato. Por ejemplo, los discos de SO SSD Premium se usan para `Standard_D2s_v3` y los discos de SO HDD estándar se usan para `Standard_D2_v3`.

Los discos de SO SSD Premium son más costosos, pero tienen un rendimiento más alto y las máquinas virtuales con discos premium pueden iniciarse ligeramente más rápido que las máquinas virtuales con discos OS HDD estándar. Con Batch, el disco de SO generalmente no se usa tanto como los archivos de tareas y las aplicaciones que se encuentran en el disco SSD temporal de las máquinas virtuales. Por lo tanto, en muchos casos, no es necesario pagar el aumento del costo por el disco SSD Premium que se aprovisiona cuando se especifica un tamaño de máquina virtual "s".

### <a name="reserved-virtual-machine-instances"></a>Instancias reservadas de máquina virtual

Si piensa usar Batch durante largo tiempo, puede ahorrar en el costo de las máquinas virtuales si usa [Azure Reservations](../cost-management-billing/reservations/save-compute-costs-reservations.md) para las cargas de trabajo. Una tarifa de reserva es considerablemente menor que una tarifa de pago por uso. Las instancias de máquina virtual que se usan sin una reserva se cargan según la tarifa de pago por uso. Si compra una reserva, se aplica el descuento de reserva y ya no se le cargará según las tarifas de pago por uso.

### <a name="automatic-scaling"></a>Escalado automático

El [escalado automático](batch-automatic-scaling.md) escala de manera dinámica el número de máquinas virtuales en el grupo de Batch en función de las demandas del trabajo actual. Al escalar el grupo según la duración de un trabajo, el escalado automático garantizar que las máquinas virtuales se escalen verticalmente y se usen solo cuando haya que realizar un trabajo. Cuando el trabajo se complete o no haya trabajos, las máquinas virtuales se reducen verticalmente de manera automáticamente para ahorrar recursos de proceso. El escalamiento permite disminuir los costos generales de la solución de Batch mediante el uso solo de los recursos que necesita.

Para más información sobre el escalado automático, consulte [Escalado automático de los nodos de ejecución en un grupo de Azure Batch](batch-automatic-scaling.md).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre las [herramientas y API de Batch](batch-apis-tools.md) disponibles para compilar y supervisar las soluciones de Batch.  

- Aprenda sobre las [máquinas virtuales de prioridad baja con Batch](batch-low-pri-vms.md).
