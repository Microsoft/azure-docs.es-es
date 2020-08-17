---
title: Escalado vertical de conjuntos de escalado de máquinas virtuales de Azure
description: Cómo escalar verticalmente una máquina virtual en respuesta a las alertas de supervisión con Azure Automation
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 04/18/2019
ms.reviewer: avverma
ms.custom: avverma
ms.openlocfilehash: 37602f7b9a8669ce0e8db984f7f7617cffdd431c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029287"
---
# <a name="vertical-autoscale-with-virtual-machine-scale-sets"></a>Escalado automático vertical con conjuntos de escalado de máquinas virtuales

Este artículo describe cómo escalar verticalmente [conjuntos de escalado de máquinas virtuales](https://azure.microsoft.com/services/virtual-machine-scale-sets/) de Azure con o sin reaprovisionamiento. 

El escalado vertical, también conocido como *ampliación vertical* y *reducción vertical*, significa aumentar o disminuir el tamaño de las máquinas virtuales (VM) en respuesta a una carga de trabajo. Compare este comportamiento con el [escalado horizontal](virtual-machine-scale-sets-autoscale-overview.md), también denominado *Escalabilidad horizontal* y *Reducción horizontal*, donde se modifica el número de máquinas virtuales según la carga de trabajo.

Reaprovisionar significa quitar una máquina virtual existente y reemplazarla por una nueva. Al aumentar o disminuir el tamaño de las máquinas virtuales en un conjunto de escalado de máquinas virtuales, en algunos casos puede que quiera cambiar el tamaño de las máquinas virtuales existentes y conservar los datos, aunque en otros casos puede que necesite implementar nuevas máquinas virtuales con el nuevo tamaño. Este documento describe ambos casos.

El escalado vertical puede resultar útil cuando:

* Un servicio integrado en máquinas virtuales se está infrautilizando (por ejemplo, los fines de semana). Reducir el tamaño de la máquina virtual puede reducir los costos mensuales.
* Aumentar el tamaño de la máquina virtual para hacer frente a una mayor demanda sin crear máquinas virtuales adicionales.

Puede configurar el escalado vertical para que se desencadene en función de las alertas basadas en métricas de su conjunto de escalado de máquinas virtuales. Cuando la alerta se activa, inicia un webhook que desencadena un runbook que puede ampliar o reducir verticalmente el conjunto de escalado. El escalado vertical puede configurarse siguiendo estos pasos:

1. Cree una cuenta de Azure Automation con funciones de ejecución.
2. Importe a la suscripción los runbooks de escalado vertical de Azure Automation para los conjuntos de escalado de máquinas virtuales.
3. Agregue un webhook al Runbook.
4. Agregue una alerta a su conjunto de escalado de máquinas virtuales mediante una notificación de webhook.

> [!NOTE]
> Debido al tamaño de la primera máquina virtual, los tamaños a los que se puede escalar pueden estar limitados en virtud de la disponibilidad de los demás tamaños en el clúster donde actualmente está implementada la máquina virtual. En los runbooks de automatización publicados que se usan en este artículo nos hacemos cargo de esta situación y solo escalamos dentro de los siguientes pares de tamaños de máquina virtual. Esto significa que una máquina virtual Standard_D1v2 no se aumentará de manera repentina a Standard_G5 ni se reducirá a Basic_A0. Además, no se admite el escalado y la reducción verticales de tamaños de máquina virtual restringidos. Puede elegir escalar entre los siguientes pares de tamaños:
> 
> | Miembro de pares de escalado de tamaños de VM | Member |
> | --- | --- |
> | Basic_A0 |Basic_A4 |
> | Standard_A0 |Standard_A4 |
> | Standard_A5 |Standard_A7 |
> | Standard_A8 |Standard_A9 |
> | Standard_A10 |Standard_A11 |
> | Standard_A1_v2 |Standard_A8_v2 |
> | Standard_A2m_v2 |Standard_A8m_v2  |
> | Standard_B1s |Standard_B2s |
> | Standard_B1ms |Standard_B8ms |
> | Standard_D1 |Standard_D4 |
> | Standard_D11 |Standard_D14 |
> | Standard_DS1 |Standard_DS4 |
> | Standard_DS11 |Standard_DS14 |
> | Standard_D1_v2 |Standard_D5_v2 |
> | Standard_D11_v2 |Standard_D14_v2 |
> | Standard_DS1_v2 |Standard_DS5_v2 |
> | Standard_DS11_v2 |Standard_DS14_v2 |
> | Standard_D2_v3 |Standard_D64_v3 |
> | Standard_D2s_v3 |Standard_D64s_v3 |
> | Standard_DC2s |Standard_DC4s |
> | Standard_E2_v3 |Standard_E64_v3 |
> | Standard_E2s_v3 |Standard_E64s_v3 |
> | Standard_F1 |Standard_F16 |
> | Standard_F1s |Standard_F16s |
> | Standard_F2sv2 |Standard_F72sv2 |
> | Standard_G1 |Standard_G5 |
> | Standard_GS1 |Standard_GS5 |
> | Standard_H8 |Standard_H16 |
> | Standard_H8m |Standard_H16m |
> | Standard_L4s |Standard_L32s |
> | Standard_L8s_v2 |Standard_L80s_v2 |
> | Standard_M8ms  |Standard_M128ms |
> | Standard_M32ls  |Standard_M64ls |
> | Standard_M64s  |Standard_M128s |
> | Standard_M64  |Standard_M128 |
> | Standard_M64m  |Standard_M128m |
> | Standard_NC6 |Standard_NC24 |
> | Standard_NC6s_v2 |Standard_NC24s_v2 |
> | Standard_NC6s_v3 |Standard_NC24s_v3 |
> | Standard_ND6s |Standard_ND24s |
> | Standard_NV6 |Standard_NV24 |
> | Standard_NV6s_v2 |Standard_NV24s_v2 |
> | Standard_NV12s_v3 |Standard_NV48s_v3 |
> 

## <a name="create-an-azure-automation-account-with-run-as-capability"></a>Creación de una cuenta de Azure Automation con funciones de ejecución
Lo primero que debe hacer es crear una cuenta de Azure Automation que hospedará los runbooks que se usan para escalar las instancias del conjunto de escalado de máquinas virtuales. Recientemente, [Azure Automation](https://azure.microsoft.com/services/automation/) presentó la característica "Cuenta de ejecución", que permite realizar la configuración de la entidad de servicio para ejecutar los runbooks automáticamente en nombre de un usuario. Para más información, consulte:

* [Autenticación de Runbooks con una cuenta de ejecución de Azure](../automation/manage-runas-account.md)

## <a name="import-azure-automation-vertical-scale-runbooks-into-your-subscription"></a>Importación de runbooks de escalado vertical de Azure Automation a la suscripción

Los runbooks necesarios para el escalado vertical de los conjuntos de escalado de máquinas virtuales están publicados en la galería de runbooks de Azure Automation. Para importarlos a su suscripción, siga los pasos de este artículo:

* [Galerías de runbooks y módulos para Azure Automation](../automation/automation-runbook-gallery.md)

Elija la opción Examinar la galería en el menú Runbooks:

![Runbooks que desea importar][runbooks]

Se muestran los Runbooks que es necesario importar. Seleccione el Runbook en función de si desea un escalado vertical con o sin reaprovisionamiento:

![Galería de Runbooks][gallery]

## <a name="add-a-webhook-to-your-runbook"></a>Agregar un webhook al runbook.

Una vez que importe los runbooks, agregue un webhook al runbook para que, de este modo, una alerta de un conjunto de escalado de máquinas virtuales pueda desencadenarlo. En este artículo puede leer los detalles sobre cómo crear un webhook para el Runbook:

* [Webhooks de Azure Automation](../automation/automation-webhooks.md)

> [!NOTE]
> Asegúrese de copiar el URI del webhook antes de cerrar el cuadro de diálogo del webhook, porque necesitará esta dirección en la siguiente sección.
> 
> 

## <a name="add-an-alert-to-your-virtual-machine-scale-set"></a>Adición de una alerta al conjunto de escalado de máquinas virtuales

A continuación, se presenta un script de PowerShell que muestra cómo agregar una alerta a un conjunto de escalado de máquinas virtuales. Consulte el siguiente artículo para obtener el nombre de la métrica para desencadenar la alerta en: [Métricas comunes de escalado automático de Azure Monitor](../azure-monitor/platform/autoscale-common-metrics.md).

```powershell
$actionEmail = New-AzAlertRuleEmail -CustomEmail user@contoso.com
$actionWebhook = New-AzAlertRuleWebhook -ServiceUri <uri-of-the-webhook>
$threshold = <value-of-the-threshold>
$rg = <resource-group-name>
$id = <resource-id-to-add-the-alert-to>
$location = <location-of-the-resource>
$alertName = <name-of-the-resource>
$metricName = <metric-to-fire-the-alert-on>
$timeWindow = <time-window-in-hh:mm:ss-format>
$condition = <condition-for-the-threshold> # Other valid values are LessThanOrEqual, GreaterThan, GreaterThanOrEqual
$description = <description-for-the-alert>

Add-AzMetricAlertRule  -Name  $alertName `
                            -Location  $location `
                            -ResourceGroup $rg `
                            -TargetResourceId $id `
                            -MetricName $metricName `
                            -Operator  $condition `
                            -Threshold $threshold `
                            -WindowSize  $timeWindow `
                            -TimeAggregationOperator Average `
                            -Actions $actionEmail, $actionWebhook `
                            -Description $description
```

> [!NOTE]
> Se recomienda configurar un período de tiempo razonable para la alerta con el fin de evitar activar el escalado vertical, y las interrupciones de servicio asociadas, con demasiada frecuencia. Considere un intervalo mínimo de 20 a 30 minutos. Considere la posibilidad de usar escalado horizontal si necesita evitar cualquier interrupción.
> 
> 

Para más información sobre cómo crear alertas, consulte los artículos siguientes:

* [Ejemplos de PowerShell de Azure Monitor](../azure-monitor/samples/powershell-samples.md)
* [Ejemplos de la CLI multiplataforma de Azure Monitor](../azure-monitor/samples/cli-samples.md)

## <a name="summary"></a>Resumen

En este artículo se mostraron ejemplos sencillos de escalado vertical. Con estos bloques de creación (cuenta de Automation, runbooks, webhooks, alertas) puede conectar una gran variedad de eventos con un conjunto personalizado de acciones.

[runbooks]: ./media/virtual-machine-scale-sets-vertical-scale-reprovision/runbooks.png
[gallery]: ./media/virtual-machine-scale-sets-vertical-scale-reprovision/runbooks-gallery.png
