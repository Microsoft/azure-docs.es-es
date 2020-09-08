---
title: 'Tutorial: Reducción de los costos de Azure con recomendaciones'
description: Este tutorial le ayudará a reducir los costos de Azure cuando siga las recomendaciones de optimización.
author: bandersmsft
ms.author: banders
ms.date: 03/24/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.custom: seodec18
ms.openlocfilehash: 28f9369d044cf9b2713cb2f2ad42f744e517918b
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683290"
---
# <a name="tutorial-optimize-costs-from-recommendations"></a>Tutorial: Optimización de los costos a partir de las recomendaciones

Azure Cost Management funciona con Azure Advisor para proporcionar recomendaciones de optimización de costos. Azure Advisor le ayuda a optimizar y mejorar la eficiencia mediante la identificación de recursos inactivos e infrautilizados. Este tutorial le guiará a través de un ejemplo donde identificará los recursos de Azure infrautilizados y, a continuación, adoptará medidas para reducir los costos.

Vea el vídeo [Optimización de las inversiones en la nube en Azure Cost Management](https://www.youtube.com/watch?v=cSNPoAb-TNc) para aprender a usar Advisor para optimizar los costos. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/cSNPoAb-TNc]

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Ver las recomendaciones de optimización de costos para conocer las posibles deficiencias de uso
> * Actuar en función de una recomendación para cambiar el tamaño de una máquina virtual a una opción más rentable
> * Comprobar la acción para asegurarse de que la máquina virtual cambió el tamaño correctamente

## <a name="prerequisites"></a>Prerrequisitos
Hay recomendaciones disponibles para varios tipos de cuenta y ámbitos de Azure. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](understand-cost-mgt-data.md) (Información sobre los datos de Cost Management). Debe tener al menos acceso de lectura a uno o varios de los siguientes ámbitos para ver datos de costos. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

- Subscription
- Resource group

Si su suscripción es nueva, no podrá usar inmediatamente las características de Cost Management. Para poder hacerlo deberán transcurrir un máximo de 48 horas. También debe tener máquinas virtuales activas con un mínimo de 14 días de actividad.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com/).

## <a name="view-cost-optimization-recommendations"></a>Visualización de las recomendaciones de optimización de costos

Para ver las recomendaciones de optimización de costos para una suscripción, abra el ámbito de su elección en Azure Portal y seleccione **Recomendaciones de Advisor**.

Con el fin de ver las recomendaciones para un grupo de administración, abra el ámbito de su elección en Azure Portal y seleccione **Análisis de costos** en el menú. Use la píldora **Ámbito** para cambiar a un ámbito distinto como, por ejemplo, un grupo de administración. Seleccione **Recomendaciones de Advisor** en el menú. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

![Recomendaciones de Advisor para Cost Management en Azure Portal](./media/tutorial-acm-opt-recommendations/advisor-recommendations.png)

La lista de recomendaciones identifica las deficiencias de uso o muestra las recomendaciones de compra que pueden ayudarle a ahorrar más dinero. El **Ahorro potencial anual** totalizado muestra el importe total que se puede ahorrar si apaga o desasigna todas las VM que cumplan las reglas de recomendación. Si no quiere apagarlas, considere la posibilidad de cambiar su tamaño a una SKU de VM menos costosa.

La categoría **Impacto**, junto con el **Ahorro potencial anual**, está diseñada para ayudar a identificar las recomendaciones con el máximo potencial de ahorro.

Entre las recomendaciones de gran impacto se incluyen:
- [Compra de instancias reservadas de máquina virtual para ahorrar dinero en los costos de pago por uso](../../advisor/advisor-cost-recommendations.md#buy-reserved-virtual-machine-instances-to-save-money-over-pay-as-you-go-costs).
- [Optimización del gasto en máquinas virtuales mediante la adecuación del tamaño o el apagado en instancias infrautilizadas](../../advisor/advisor-cost-recommendations.md#optimize-virtual-machine-spend-by-resizing-or-shutting-down-underutilized-instances).
- [Uso de Standard Storage para almacenar instantáneas de Managed Disks](../../advisor/advisor-cost-recommendations.md#use-standard-snapshots-for-managed-disks).

Entre las recomendaciones de impacto medio se incluyen:
- [Eliminación de las canalizaciones de Azure Data Factory que dan error](../../advisor/advisor-cost-recommendations.md#delete-azure-data-factory-pipelines-that-are-failing).
- [Reducción de los costos mediante la eliminación de circuitos ExpressRoute no aprovisionados](../../advisor/advisor-cost-recommendations.md#reduce-costs-by-eliminating-unprovisioned-expressroute-circuits).
- [Reducción de costos mediante la eliminación o reconfiguración de puertas de enlace de red virtual inactivas](../../advisor/advisor-cost-recommendations.md#reduce-costs-by-deleting-or-reconfiguring-idle-virtual-network-gateways).

## <a name="act-on-a-recommendation"></a>Actuar en función de una recomendación

Azure Advisor supervisa el uso de las máquinas virtuales durante siete días e identifica las que están infrautilizadas. Se considera que una máquina virtual tiene una utilización escasa si su uso de la CPU es del 5 % o menos y el de la red es de 7 MB o menos durante cuatro o más días.

La configuración de uso de la CPU del 5 % o menos es el valor predeterminado, pero se puede ajustar. Para más información sobre cómo ajustar la configuración, consulte [Recomendación de la configuración de la regla de uso promedio de CPU para una máquina virtual con poco uso](../../advisor/advisor-get-started.md#configure-low-usage-vm-recommendation).

Si bien determinados escenarios pueden dar lugar a un uso escaso debido al diseño, a menudo puede ahorrar dinero al cambiar el tamaño de las máquinas virtuales a otros menos costosos. Los ahorros reales pueden variar si elige una acción de cambio de tamaño. Veamos un ejemplo de cambio de tamaño de una máquina virtual.

En la lista de recomendaciones, haga clic en la recomendación **Apague las máquinas virtuales infrautilizadas o cambie su tamaño**. En la lista de candidatos de máquina virtual, elija una máquina virtual para cambiar su tamaño y, luego, haga clic en la máquina virtual. Se muestran los detalles de la máquina virtual para que pueda comprobar las métricas de uso. El valor **Ahorro potencial anual** es lo que puede ahorrar si apaga o quita la VM. Con el cambio de tamaño de una VM, probablemente, ahorre dinero, pero no ahorrará el importe total del ahorro potencial anual.

![Ejemplo de los detalles de recomendación](./media/tutorial-acm-opt-recommendations/recommendation-details.png)

En los detalles de la VM, compruebe el uso de la máquina virtual para confirmar que es un candidato adecuado para cambiar el tamaño.

![Ejemplo de detalles de la máquina virtual que muestran el uso histórico](./media/tutorial-acm-opt-recommendations/vm-details.png)

Anote el tamaño actual de la máquina virtual. Después de comprobar que debe cambiarse el tamaño de la máquina virtual, cierre los detalles de la VM para ver la lista de máquinas virtuales.

En la lista de candidatos para apagar o cambiar el tamaño, seleccione **Resize _&lt;FromVirtualMachineSKU&gt;_ to _&lt;ToVirtualMachineSKU&gt;_** (Cambiar tamaño de FromVirtualMachineSKU a ToVirtualMachineSKU).
![Recomendaciones de ejemplo con la opción de cambiar el tamaño de la máquina virtual](./media/tutorial-acm-opt-recommendations/resize-vm.png)

A continuación, se le presentará una lista de opciones de cambio de tamaño disponibles. Elija la que ofrezca el mejor rendimiento y rentabilidad para su escenario. En el ejemplo siguiente, la opción elegida cambia el tamaño de **Standard_D8s_v3** a **Standard_D2s_v3**.

![Lista de ejemplo de los tamaños de máquina virtual disponibles donde puede elegir un tamaño](./media/tutorial-acm-opt-recommendations/choose-size.png)

Después de elegir un tamaño adecuado, haga clic en **Cambiar tamaño** para iniciar la acción de cambio de tamaño.

El cambio de tamaño requiere que se reinicie la máquina virtual en ejecución activa. Si la máquina virtual está en un entorno de producción, se recomienda ejecutar la operación de cambio de tamaño después del horario comercial. Programar el reinicio puede reducir las interrupciones causadas por la falta de disponibilidad momentánea.

## <a name="verify-the-action"></a>Comprobación de la acción

Cuando el cambio de tamaño de la VM finalice correctamente, se mostrará una notificación de Azure.

![Notificación de que el tamaño de la máquina virtual se cambió correctamente](./media/tutorial-acm-opt-recommendations/resized-notification.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Ver las recomendaciones de optimización de costos para conocer las posibles deficiencias de uso
> * Actuar en función de una recomendación para cambiar el tamaño de una máquina virtual a una opción más rentable
> * Comprobar la acción para asegurarse de que la máquina virtual cambió el tamaño correctamente

Si aún no ha leído el artículo de prácticas recomendadas de Cost Management, allí encontrará orientación de alto nivel y principios a tener en cuenta para ayudar a administrar los costos.

> [!div class="nextstepaction"]
> [Prácticas recomendadas de Cost Management](cost-mgt-best-practices.md)
