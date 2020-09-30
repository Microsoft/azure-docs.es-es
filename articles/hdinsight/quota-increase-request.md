---
title: 'Solicitud de aumento de cuota de núcleos de CPU: Azure HDInsight'
description: Obtenga información sobre el proceso para solicitar un aumento de los núcleos de la CPU asignados a su suscripción.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 05/07/2020
ms.openlocfilehash: f8411176e0979bdb894983bcf866abd0e1109e21
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291677"
---
# <a name="requesting-quota-increases-for-azure-hdinsight"></a>Solicitud de aumentos de cuota para Azure HDInsight

Las cuotas de núcleos de la CPU permiten garantizar que el uso de recursos se distribuya equitativamente entre todos los clientes de una determinada región de Azure. Sin embargo, en algunos casos, es posible que los requisitos empresariales exijan más recursos de clúster de los que permitirá la cuota actual. En tales casos, puede solicitar un aumento de la cuota de núcleos de la CPU para poder implementar clústeres, que cumplan los requisitos de procesamiento de datos.

Cuando alcance un límite de cuota, no podrá implementar clústeres nuevos ni escalar horizontalmente los existentes con la adición de más nodos de trabajo. El límite de cuota única es la cuota de núcleos de CPU que existe en el nivel de región para cada suscripción. Por ejemplo, la suscripción puede tener un límite de 30 núcleos de CPU en la región Este de EE. UU., con otros 30 núcleos de CPU permitidos en el Este de EE. UU. 2.

## <a name="gather-required-information"></a>Recopilación de la información necesaria

Si ha recibido un error que indica que ha alcanzado un límite de cuota, use el proceso descrito en esta sección para recopilar información importante y enviar una solicitud de aumento de cuota.

1. Determine el tamaño, la escala y el tipo de máquina virtual del clúster deseado.
1. Compruebe los límites de capacidad actuales de la cuota de la suscripción. Siga estos pasos para comprobar los núcleos disponibles:

    1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
    1. Vaya a la página **información general** para el clúster de HDInsight.
    1. En el menú de la izquierda, seleccione **Límites de cuota**. La página muestra el número de núcleos en uso, el número de núcleos disponibles y el total de núcleos.

Para solicitar un aumento de la cuota, siga los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Seleccione **Ayuda y soporte técnico** de la parte inferior izquierda de la página.

    ![botón de ayuda y soporte técnico](./media/quota-increase-request/help-support-button.png)

1. Seleccione **Nueva solicitud de soporte técnico**.
1. En la página **Nueva solicitud de soporte técnico**, en la pestaña **Fundamentos**, seleccione las opciones siguientes:

   - **Tipo de problema**: **Límites de servicio y suscripción (cuotas)**
   - **Suscripción**: la suscripción que desea modificar.
   - **Tipo de cuota**: **HDInsight**

     ![Creación de una solicitud de soporte técnico para aumentar la cuota de núcleos de HDInsight](./media/quota-increase-request/hdinsight-quota-support-request.png)

1. Seleccione **Siguiente: Soluciones >>** .
1. En la página **Detalles**, escriba una descripción del problema, seleccione su gravedad y método de contacto preferido, y complete los demás campos obligatorios. Use la plantilla que se muestra a continuación para asegurarse de proporcionar la información necesaria. El equipo de capacidad de Azure es quién evalúa las solicitudes de aumento de cuota, y no el equipo del producto de HDInsight. Cuanto más completa sea la información que proporcione, más probabilidades habrá de que se apruebe la solicitud.

    ```
    I would like to request [SPECIFY DESIRED AMOUNT] on [DESIRED SKU] for [SUBSCRIPTION ID].
    
    My current quota on this subscription is [CURRENT QUOTA AMOUNT].
    
    I would like to use the extra cores for [DETAIL REASON].
    ```

    ![detalles del problema](./media/quota-increase-request/problem-details.png)

1. Seleccione **Siguiente: Revisar y crear >>** .
1. En la pestaña **Revisar y crear**, seleccione **Crear**.

> [!NOTE]  
> Si necesita aumentar la cuota de núcleos de HDInsight en una región privada, [envíe una solicitud de lista aprobada](https://aka.ms/canaryintwhitelist).

Puede [ponerse en contacto con el servicio de soporte técnico para solicitar un aumento de la cuota](https://docs.microsoft.com/azure/azure-portal/supportability/resource-manager-core-quotas-request).

Hay algunos límites de cuota fijos. Por ejemplo, una sola suscripción de Azure puede tener como máximo 10 000 núcleos. Para obtener información detallada sobre estos límites, vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits).

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de clústeres en HDInsight con Hadoop, Spark, Kafka, etc.](hdinsight-hadoop-provision-linux-clusters.md): Aprenda a instalar y configurar clústeres en HDInsight.
* [Supervisión del rendimiento del clúster](hdinsight-key-scenarios-to-monitor.md): obtenga información sobre los escenarios claves para supervisar el clúster de HDInsight que podría afectar a la capacidad del clúster.
