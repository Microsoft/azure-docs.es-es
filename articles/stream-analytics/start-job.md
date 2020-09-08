---
title: Inicio de un trabajo de Azure Stream Analytics
description: En este artículo se describe cómo iniciar un trabajo de Stream Analytics desde Azure Portal, PowerShell y Visual Studio.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 04/03/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: bce1f00603d9304fc544fb92d4cc5f1a429504ed
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89069007"
---
# <a name="how-to-start-an-azure-stream-analytics-job"></a>Inicio de un trabajo de Azure Stream Analytics

Puede iniciar el trabajo de Azure Stream Analytics mediante Azure Portal, Visual Studio y PowerShell. Cuando inicie un trabajo, seleccione una hora para que el trabajo comience a crear la salida. Azure Portal, Visual Studio y PowerShell tienen diferentes métodos para establecer la hora de inicio. Esos métodos se describen a continuación.

## <a name="start-options"></a>Opciones de inicio
Las tres opciones siguientes están disponibles para iniciar un trabajo. Tenga en cuenta que todas las horas que se mencionan a continuación son las especificadas en [TIMESTAMP BY](https://docs.microsoft.com/stream-analytics-query/timestamp-by-azure-stream-analytics). Si TIMESTAMP BY no se ha especificado, se usará la hora de llegada.
* **Ahora**: hace que el punto de inicio de la secuencia del evento de salida coincida con el de inicio del trabajo. Si se usa un operador temporal (por ejemplo, una ventana de tiempo, LAG o JOIN), Azure Stream Analytics volverá a consultar los datos en el origen de entrada automáticamente. Por ejemplo, si inicia un trabajo "Ahora" y la consulta utiliza una ventana de saltos de tamaño constante de 5 minutos, Azure Stream Analytics buscará datos de hace 5 minutos en la entrada.
El primer evento de salida posible tendría una marca de tiempo igual o mayor que la hora actual y ASA garantiza que todos los eventos de entrada que pueden contribuir lógicamente a la salida se tengan en cuenta. Por ejemplo, no se generan agregados en ventanas parciales. Siempre es el valor agregado completo.

* **Personalizado**: puede elegir el punto de partida de la salida. Del mismo modo que la opción **Ahora**, Azure Stream Analytics leerá automáticamente los datos antes de esta hora si se usa un operador temporal. 

* **Última detención**. Esta opción está disponible cuando el trabajo se inició previamente, pero se detuvo manualmente o no se realizó correctamente. Si se elige esta opción, Azure Stream Analytics usará la última hora de salida para reiniciar el trabajo, de modo que no se pierdan datos. Del mismo modo que las opciones anteriores, Azure Stream Analytics leerá automáticamente los datos antes de esta hora si se usa un operador temporal. Dado que varias particiones de entrada pueden tener una hora diferente, se usa la primera hora de detención de todas las particiones. Como resultado, se pueden ver algunos duplicados en la salida. Encontrará más información sobre el procesamiento de tipo “exactamente una vez” en la página [Event Delivery Guarantees](https://docs.microsoft.com/stream-analytics-query/event-delivery-guarantees-azure-stream-analytics) (Garantías de entrega de eventos).


## <a name="azure-portal"></a>Portal de Azure

Vaya a su trabajo en Azure Portal y seleccione **Iniciar** en la página de información general. Seleccione **Hora de inicio de la salida del trabajo** y, a continuación, seleccione **Iniciar**.

Elija una de las opciones de **Hora de inicio de la salida del trabajo**. Las opciones son *Ahora*, *personalizado* y, si el trabajo ya se ejecutó previamente, *Última detención*. Consulte arriba más información sobre estas opciones.

## <a name="visual-studio"></a>Visual Studio

En la vista de trabajos, seleccione el botón de la flecha verde para iniciar el trabajo. Establezca **Modo de inicio de salida del trabajo** y seleccione **Iniciar**. El estado del trabajo cambiará a **En ejecución**.

Existen tres opciones para **Modo de inicio de salida del trabajo**: *JobStartTime*, *CustomTime* y *LastOutputEventTime*. Si esta propiedad no está presente, el valor predeterminado es *JobStartTime*. Consulte arriba más información sobre estas opciones.


## <a name="powershell"></a>PowerShell

Use el siguiente cmdlet para iniciar el trabajo con PowerShell:

```powershell
Start-AzStreamAnalyticsJob `
  -ResourceGroupName $resourceGroup `
  -Name $jobName `
  -OutputStartMode 'JobStartTime'
```

Existen tres opciones para **OutputStartMode**: *JobStartTime*, *CustomTime* y *LastOutputEventTime*. Si esta propiedad no está presente, el valor predeterminado es *JobStartTime*. Consulte arriba más información sobre estas opciones.

Para obtener más información sobre el cmdlet `Start-AzStreamAnalyitcsJob`, consulte la [referencia de Start-AzStreamAnalyticsJob](/powershell/module/az.streamanalytics/start-azstreamanalyticsjob).

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante las herramientas de Azure Stream Analytics para Visual Studio](stream-analytics-quick-create-vs.md)
