---
title: Visualización de las tendencias de costos mensuales estimados de laboratorio en Azure DevTest Labs
description: En este artículo se proporciona información sobre cómo realizar un seguimiento del costo del laboratorio (gráfico de tendencias de costo estimadas mensuales) en Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 220cbd1bc793e4bcfdf03a184d7463114506e6e0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283833"
---
# <a name="track-costs-associated-with-a-lab-in-azure-devtest-labs"></a>Seguimiento de los costos asociados con un laboratorio en Azure DevTest Labs
En este artículo se proporciona información sobre cómo seguir el costo del laboratorio. Muestra cómo ver la tendencia de costo estimado durante el mes actual para el laboratorio. El artículo también muestra cómo ver el costo por recurso del mes hasta la fecha en el laboratorio.

## <a name="view-the-monthly-estimated-lab-cost-trend"></a>Visualización de la tendencia de costos mensuales estimados de laboratorio 
En esta sección, se muestra el uso del gráfico **Monthly Estimated Cost Trend** (Tendencia de costos estimados mensuales) para ver el costo estimado hasta la fecha del mes de calendario actual, así como el costo a fin de mes previsto para el mes del calendario actual. También se muestra cómo administrar mejor los costos de laboratorio estableciendo objetivos y umbrales de gastos que, al alcanzarlos, desencadenan que DevTest Labs le informe de los resultados.

Para ver el gráfico de tendencias de costos mensuales estimados, siga estos pasos: 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
3. En la lista de laboratorios, seleccione el suyo.  
4. Seleccione **Configuración y directivas** en el menú de la izquierda.  
4. Seleccione **Tendencia de costos** en la sección **Seguimiento de costos**, en el menú de la izquierda. La siguiente captura de pantalla muestra un ejemplo de gráfico de costos. 
   
    ![Gráfico de costos](./media/devtest-lab-configure-cost-management/graph.png)

    El valor **Costo estimado** es el costo estimado hasta la fecha del mes de calendario actual. El valor de **Projected cost** (Costo previsto) es el costo estimado para el mes de calendario actual completo, calculado con el costo de laboratorio para los cinco días anteriores.

    Los importes de los costos se redondean al siguiente número entero. Por ejemplo: 

   * 5,01 se redondea a 6. 
   * 5,50 se redondea a 6.
   * 5,99 se redondea a 6.

     Como indica anteriormente el gráfico, los costos que se ven de forma predeterminada en el gráfico son costos *estimados* con tarifas de oferta de [Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/). También puede establecer sus propios objetivos de gastos que se muestran en los gráficos mediante la [administración de los objetivos de costos para un laboratorio](#managing-cost-targets-for-your-lab).

     Los siguientes costos *no* se incluyen en el cálculo de costos:

   * Actualmente no se admiten suscripciones de DreamSpark y CSP, puesto que Azure DevTest Labs emplea las [API de facturación de Azure](../cost-management-billing/manage/usage-rate-card-overview.md) para calcular el costo de laboratorio, y estas suscripciones no las admiten.
   * Las tarifas de su oferta. Actualmente, no puede usar las tarifas de oferta (que aparecen en su suscripción) negociadas con Microsoft o con asociados de Microsoft. Solo se usan tarifas de pago por uso.
   * Los impuestos
   * Los descuentos
   * La divisa de facturación. Actualmente, el costo de laboratorio solo se muestra en divisa USD.

### <a name="managing-cost-targets-for-your-lab"></a>Administración de objetivos de costos para un laboratorio
DevTest Labs le permite administrar mejor los costos de su laboratorio estableciendo un objetivo de gasto que puede ver en el gráfico de tendencias de costos mensuales estimados. DevTest Labs también puede enviarle una notificación cuando se alcanza el umbral o el gasto objetivo especificado. 

1. En la página **Tendencia de costos**, seleccione **Administrar objetivo**.

    ![Botón Administrar objetivo](./media/devtest-lab-configure-cost-management/cost-trend-manage-target.png)
2. En el panel **Administrar objetivo**, especifique el objetivo y los umbrales de gastos. También puede establecer si se informa de cada umbral seleccionado en el gráfico de tendencias de costos o mediante una notificación de webhook.

    ![Panel Administrar objetivo](./media/devtest-lab-configure-cost-management/cost-trend-manage-target-pane.png)

   - Seleccione un período de tiempo durante el que desea que se realice el seguimiento de los objetivos de costos.
      - **Mensual**: se realiza un seguimiento de los objetivos de costos mensual.
      - **Fijo**: se realiza un seguimiento de los objetivos de costos para el intervalo de fechas que especifique en las fechas de inicio y de finalización. Por lo general, estos valores pueden corresponder a cuánto tiempo está programada la ejecución del proyecto.
   - Especifique un **Costo objetivo**. Por ejemplo, puede tratarse de cuánto planea gastar en este laboratorio en el período de tiempo definido.
   - Seleccione habilitar o deshabilitar cualquier umbral del que desee que le informen, en incrementos del 25 %, hasta un 125 % del **Costo objetivo** que ha especificado.
      - **Notificar**: cuando se alcanza este umbral, se le notificará mediante la dirección URL de webhook que especifique.
      - **Trazado en gráfico**: cuando se alcanza este umbral, los resultados se trazan en el gráfico de tendencias de costos que puede ver, tal como se describe en Visualización del gráfico de tendencias de costo estimado mensual.
   - Si opta por **Notificar** cuando se alcanza el umbral, debe especificar una dirección URL de webhook. En el área de integraciones de costos, seleccione **Haga clic aquí para agregar una integración**. Escriba una **dirección URL de webhook** en el panel Configurar notificación y seleccione **Aceptar**.

       ![Panel Configurar notificación](./media/devtest-lab-configure-cost-management/configure-notification-new.png)

     - Si especifica **Notificar**, debe definir una dirección URL de webhook.
     - Del mismo modo, si define una dirección URL de webhook, debe establecer **Notificación** en **Activado** en el panel Umbral de costo.
     - Tiene que crear un webhook antes de especificarlo aquí.  

       Para obtener más información sobre los webhooks, consulte [Creación de un webhook o una función de API de Azure](../azure-functions/functions-bindings-http-webhook.md). 

## <a name="view-cost-by-resource"></a>Visualización del costo por recurso 
La característica de tendencia de costo mensual en los laboratorios permite ver cuánto ha empleado en el mes en curso. También muestra la proyección de los gastos hasta el final del mes, en función de sus gastos en los últimos siete días. Para ayudarle a entender por qué el gasto en el laboratorio está cumpliendo los umbrales, puede usar la característica de **costo por recurso** que muestra el costo del mes hasta la fecha **por recurso** en una tabla.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
3. En la lista de laboratorios, seleccione el laboratorio que desee.  
4. Seleccione **Configuración y directivas** en el menú de la izquierda.
5. Seleccione **Costo por recurso** en la sección **Seguimiento de costos**, en el menú de la izquierda. Verá los costos asociados con cada recurso asociado a un laboratorio. 

    ![Costo por recurso](./media/devtest-lab-configure-cost-management/cost-by-resource.png)

Esta característica le ayuda a identificar fácilmente los recursos que cuestan más para que pueda tomar acciones destinadas a reducir el gasto de laboratorio. Por ejemplo, el costo de una máquina virtual se basa en el tamaño de la máquina virtual. Cuanto mayor sea el tamaño de la máquina virtual, más es el costo. Puede encontrar fácilmente el tamaño de una máquina virtual y el propietario, por lo que puede hablar con el propietario de la máquina virtual para saber por qué es necesario este tamaño de máquina virtual y si hay una posibilidad de reducir el tamaño.

La [directiva de apagado automático](devtest-lab-set-lab-policy.md?#set-auto-shutdown-policy) le ayuda a reducir el costo de apagar las máquinas virtuales de laboratorio en un momento determinado del día. Sin embargo, un usuario de laboratorio puede rechazar la directiva de apagado, lo que aumenta el costo de la ejecución de la máquina virtual. Puede seleccionar una máquina virtual en la tabla para ver si se ha dado de baja de la directiva de apagado automático. Si es así, puede hablar con el propietario de la máquina virtual para averiguar por qué la máquina virtual se ha dado de baja de la directiva.
 
## <a name="next-steps"></a>Pasos siguientes
Vea algunas acciones que puede probar a continuación:

* [Definición de directivas de laboratorio](devtest-lab-set-lab-policy.md): obtenga información sobre cómo establecer las distintas directivas que se emplean para controlar el uso del laboratorio y sus máquinas virtuales. 
* [Creación de una imagen personalizada](devtest-lab-create-template.md): cuando cree una máquina virtual, tendrá que especificar una base, que puede ser una imagen personalizada o una imagen de Marketplace. Este artículo muestra cómo crear una imagen personalizada desde un archivo VHD.
* [Configuración de imágenes de Marketplace](devtest-lab-configure-marketplace-images.md): DevTest Labs admite la creación de máquinas virtuales basadas en imágenes de Azure Marketplace. En este artículo se muestra cómo especificar las imágenes de Azure Marketplace, si corresponde, que se pueden usar en el momento de crear máquinas virtuales en un laboratorio.
* [Creación de una máquina virtual en un laboratorio](devtest-lab-add-vm.md): se muestra cómo crear una máquina virtual desde una imagen base (ya sea personalizada o de Marketplace) y cómo trabajar con artefactos en la máquina virtual.
