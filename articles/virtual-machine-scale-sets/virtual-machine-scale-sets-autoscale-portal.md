---
title: Escalado automático de conjuntos de escalado de máquinas virtuales en Azure Portal
description: Creación de reglas de escalado automático de conjuntos de escalado de máquinas virtuales en Azure Portal
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms:service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 05/29/2018
ms.reviewer: avverma
ms.custom: avverma
ms.openlocfilehash: ac42fe3265163a5a967524fe11063803c9ca91d3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080597"
---
# <a name="automatically-scale-a-virtual-machine-scale-set-in-the-azure-portal"></a>Creación de reglas de escalado automático de conjuntos de escalado de máquinas virtuales en Azure Portal
Al crear un conjunto de escalado, puede definir el número de instancias de máquina virtual que quiere ejecutar. A medida que cambia la demanda de las aplicaciones, puede aumentar o reducir automáticamente el número de estas instancias. La posibilidad de realizar el escalado automático le permite satisfacer la demanda del cliente o responder a los cambios de rendimiento de la aplicación a lo largo del ciclo de vida de esta.

Este artículo muestra cómo crear reglas de escalado automático en Azure Portal que supervisan el rendimiento de las instancias de máquina virtual del conjunto de escalado. Estas reglas de escalado automático permiten aumentar o reducir el número de instancias de máquina virtual en respuesta a estas métricas de rendimiento. También puede seguir estos pasos con [Azure PowerShell](tutorial-autoscale-powershell.md) o la [CLI de Azure](tutorial-autoscale-cli.md).


## <a name="prerequisites"></a>Requisitos previos
Para crear reglas de escalado automático, necesita un conjunto de escalado de máquinas virtuales ya existente. Puede crear un conjunto de escalado con [Azure Portal](quick-create-portal.md), [Azure PowerShell](quick-create-powershell.md) o la [CLI de Azure](quick-create-cli.md).


## <a name="create-a-rule-to-automatically-scale-out"></a>Creación de una regla para realizar un escalado horizontal de forma automática
Si aumenta la demanda de la aplicación, la carga de las instancias de máquina virtual del conjunto de escalado aumenta. Si este aumento de la carga es continuado, en lugar de ser algo puntual, puede configurar reglas de escalado automático para aumentar el número de instancias de máquina virtual en el conjunto de escalado. Cuando se crean estas instancias de máquina virtual y se implementan las aplicaciones, el conjunto de escalado empieza a distribuir el tráfico entre ellas mediante el equilibrador de carga. Puede controlar qué métricas se deben supervisar como, por ejemplo, la CPU o el disco, cuánto tiempo debe cumplir la carga de la aplicación un límite determinado y cuántas instancias de máquina virtual se deben agregar al conjunto de escalado.

1. Abra Azure Portal y seleccione **Grupos de recursos** en el menú de la izquierda del panel de información.
2. Seleccione el grupo de recursos que contenga el conjunto de escalado y, después, elija el conjunto de escalado en la lista de recursos.
3. Elija **Escalado** en el menú de la izquierda de la ventana del conjunto de escalado. Haga clic en el botón **Habilitar escalado automático**:

    ![Habilitar escalado automático en Azure Portal](media/virtual-machine-scale-sets-autoscale-portal/enable-autoscale.png)

4. Escriba el nombre de la configuración, como por ejemplo *escalado automático*, y seleccione la opción **Agregar una regla**.

5. Vamos a crear una regla que aumenta el número de instancias de máquina virtual de un conjunto de escalado cuando la carga promedio de la CPU es superior al 70 % durante un período de más de 10 minutos. Cuando se desencadena la regla, aumenta el número de instancias de máquina virtual en un 20 %. En conjuntos de escalado con un pequeño número de instancias de máquina virtual, en **Operación** se puede elegir *Aumentar el número en* y, después, especificar *1* o *2* en *Número de instancias*. En conjuntos de escalado con un gran número de instancias de máquina virtual, un aumento del 10 % o 20 % de instancias puede resultar más adecuado.

    Especifique la configuración siguiente para la regla:
    
    | Parámetro              | Explicación                                                                                                         | Value          |
    |------------------------|---------------------------------------------------------------------------------------------------------------------|----------------|
    | *Agregación de tiempo*     | Define cómo se deben agregar las métricas recopiladas para el análisis.                                                | Average        |
    | *Nombre de métrica*          | La métrica de rendimiento del conjunto de escalado sobre el que realizar las acciones de supervisión y aplicación.                                                   | Porcentaje de CPU |
    | *Estadísticas de intervalo de agregación* | Define cómo se deben agregar las métricas recopiladas en cada intervalo de agregación para el análisis.                             | Average        |
    | *Operador*             | El operador que se utiliza para comparar los datos de las métricas con los umbrales.                                                     | Mayor que   |
    | *Umbral*            | El porcentaje que hace que la regla de escalado automático desencadene una acción.                                                 | 70             |
    | *Duration*             | El periodo durante el que se realiza la supervisión antes de que se comparen los valores de métricas y umbrales. No incluye el período de enfriamiento.                                   | 10 minutos     |
    | *operación*            | Define si el conjunto de escalado debe escalarse o reducirse verticalmente cuando se aplica la regla y en qué incremento debe hacerse.                        | Aumentar porcentaje en |
    | *Recuento de instancias*       | El porcentaje de instancias de máquina virtual se debe cambiar al desencadenarse la regla.                                            | 20             |
    | *Tiempo de finalización (minutos)*  | El periodo que hay que esperar hasta que la regla se vuelva a aplicar, para que las acciones de escalado automático tengan tiempo de surtir efecto. | 5 minutos      |

    Los ejemplos siguientes muestran una regla creada en Azure Portal que se ajusta a esta configuración:

    ![Crear una regla de escalado automático para aumentar el número de instancias de máquina virtual](media/virtual-machine-scale-sets-autoscale-portal/rule-increase.png)

6. Para crear la regla, seleccione **Agregar**


## <a name="create-a-rule-to-automatically-scale-in"></a>Creación de una regla para realizar una reducción horizontal de forma automática
La demanda de la aplicación puede reducirse por las tardes o durante los fines de semana. Si esta reducción es constante a lo largo de un período, puede configurar reglas de escalado automático para reducir el número de instancias de máquina virtual del conjunto de escalado. Esta acción de reducción horizontal permite rebajar el costo de ejecutar el conjunto de escalado ya que solo se ejecuta el número de instancias necesario para satisfacer la demanda actual.

1. Elija **Agregar una regla** otra vez.
2. Cree una regla que reduzca el número de instancias de máquina virtual de un conjunto de escalado cuando la carga promedio de la CPU es inferior al 30 % durante un período de más de 10 minutos. Cuando se desencadena la regla, reduzca el número de instancias de máquina virtual en un 20 %.

    Use el mismo enfoque que con la regla anterior. Ajuste la configuración siguiente para la regla:
    
    | Parámetro              | Explicación                                                                                                          | Value          |
    |------------------------|----------------------------------------------------------------------------------------------------------------------|----------------|
    | *Operador*             | El operador que se utiliza para comparar los datos de las métricas con los umbrales.                                                      | Menor que   |
    | *Umbral*            | El porcentaje que hace que la regla de escalado automático desencadene una acción.                                                 | 30             |
    | *operación*            | Define si el conjunto de escalado debe escalarse o reducirse verticalmente cuando se aplica la regla y en qué incremento debe hacerse                         | Reducir porcentaje en |
    | *Recuento de instancias*       | El porcentaje de instancias de máquina virtual se debe cambiar al desencadenarse la regla.                                             | 20             |

3. Para crear la regla, seleccione **Agregar**


## <a name="define-autoscale-instance-limits"></a>Definir los límites de la instancia de escalado automático
El perfil de escalado automático debe definir un número mínimo, máximo y predeterminado de instancias de máquina virtual. Cuando se aplican las reglas de escalado automático, estos límites de instancia garantizan que no se realiza un escalado horizontal que supere el número máximo de instancias ni se realiza una reducción horizontal que no llegue al mínimo de instancias.

1. Establezca los siguientes límites de instancia:

    | Mínima | Máxima | Valor predeterminado|
    |---------|---------|--------|
    | 2       | 10      | 2      |

2. Para aplicar las reglas de escalado automático y los límites de instancia, seleccione **Guardar**.


## <a name="monitor-number-of-instances-in-a-scale-set"></a>Supervisión del número de instancias de un conjunto de escalado
Para ver tanto el número como el estado de las instancias de máquina virtual, seleccione **Instancias** en el menú de la izquierda de la ventana del conjunto de escalado. El estado indica si la instancia de la máquina virtual se *crea* a medida que el conjunto de escalado se escala horizontalmente de forma automática o se *elimina* a medida que la escala se reduce horizontalmente de forma automática.

![Ver una lista de instancias de máquina virtual del conjunto de escalado](media/virtual-machine-scale-sets-autoscale-portal/view-instances.png)


## <a name="autoscale-based-on-a-schedule"></a>Escalado automático según una programación
En los ejemplos anteriores se escaló o redujo horizontalmente de forma automática un conjunto de escalado con métricas de host básicas como el uso de la CPU. También se pueden crear reglas de escalado automático basadas en programación. Estas reglas basadas en programación le permiten escalar horizontalmente de forma automática el número de instancias de máquina virtual anticipándose a un aumento de la demanda de la aplicación como, por ejemplo, durante las horas punta del trabajo para, a continuación, reducir horizontalmente de forma automática el número de instancias a la vez si anticipa una menor demanda como durante los fines de semana.

1. Elija **Escalado** en el menú de la izquierda de la ventana del conjunto de escalado. Para eliminar las reglas de escalado automático existentes creadas en los ejemplos anteriores, elija el icono de la papelera.

    ![Eliminar las reglas de escalado automático existente](media/virtual-machine-scale-sets-autoscale-portal/delete-rules.png)

2. Elija **Add a scale condition** (Agregar una condición de escalado). Seleccione el icono de lápiz situado junto al nombre de la regla y especifique un nombre, como por ejemplo *Scale out during each work day*.

    ![Cambiar el nombre de la regla de escalado automático predeterminada](media/virtual-machine-scale-sets-autoscale-portal/rename-rule.png)

3. Seleccione el botón de radio **Escalar a un número específico de instancias**.
4. Para escalar verticalmente el número de instancias, escriba *10* como número de instancias.
5. Elija **Repetir en días específicos** en el tipo de **Programación**.
6. Seleccione todos los días laborables, de lunes a viernes.
7. Elija la zona horaria adecuada y en **Hora de inicio** especifique *09:00*.
8. Elija **Add a scale condition** (Agregar una condición de escalado) otra vez. Repita el proceso para crear una programación denominada *Scale in during the evening* que escale hasta *3* instancias, se repite todos los día de la semana y comienza a las *18:00*.
9. Para aplicar las reglas de escalado automático basadas en programación, seleccione **Guardar**.

    ![Crear reglas de escalado automático que realicen el escalado según una programación](media/virtual-machine-scale-sets-autoscale-portal/schedule-autoscale.PNG)

Para ver cómo se aplican las reglas de escalado automático, seleccione **Historial de ejecución** en la parte superior de la ventana **Escala**. El gráfico y la lista de eventos muestran cuándo se desencadenan las reglas de escalado automático y el número de instancias de máquina virtual del conjunto de escalado aumenta o disminuye.


## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha aprendido a utilizar reglas de escalado automático para escalar horizontalmente y aumentar o reducir el *número* de instancias de máquina virtual del conjunto de escalado. También puede escalar verticalmente para aumentar o reducir el *tamaño* de la instancia de máquina virtual. Para más información, consulte [Escalado automático vertical con conjuntos de escalado de máquinas virtuales](virtual-machine-scale-sets-vertical-scale-reprovision.md).

Para más información acerca de cómo administrar las instancias de máquina virtual, consulte [Manage virtual machine scale sets with Azure PowerShell](./virtual-machine-scale-sets-manage-powershell.md) (Administración de conjuntos de escalado de máquinas virtuales con Azure PowerShell).

Para más información acerca de cómo generar alertas cuando la regla de escalado automático se desencadena, consulte [Uso de acciones de escalado automático para enviar notificaciones de alerta por correo electrónico y Webhook en Azure Monitor](../azure-monitor/platform/autoscale-webhook-email.md). También puede consultar [Llamada a un webhook cuando se activan alertas del registro de actividades de Azure](../azure-monitor/platform/alerts-log-webhook.md).
