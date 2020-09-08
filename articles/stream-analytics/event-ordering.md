---
title: Configuración de las directivas de ordenación de eventos para Azure Stream Analytics
description: En este artículo se describe cómo establecer la configuración del orden de los eventos en Stream Analytics.
author: sidram
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 08/06/2020
ms.openlocfilehash: b4e34befbf28de2b985ff49ce17a87a25842015e
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87901698"
---
# <a name="configuring-event-ordering-policies-for-azure-stream-analytics"></a>Configuración de las directivas de ordenación de eventos para Azure Stream Analytics

En este artículo se describe cómo configurar y utilizar las directivas de eventos desordenados y de llegada tardía en Azure Stream Analytics. Estas directivas se aplican solo cuando se usa la cláusula [TIMESTAMP BY](https://docs.microsoft.com/stream-analytics-query/timestamp-by-azure-stream-analytics) en la consulta y solo se aplican para orígenes de entrada de nube.

## <a name="event-time-and-arrival-time"></a>Hora del evento y hora de llegada

El trabajo de Stream Analytics puede procesar eventos según la *hora del evento* o la *hora de llegada*. **La hora del evento/aplicación** es la marca de tiempo presente en la carga del evento (cuándo se generó el evento). **La hora de llegada** es la marca de tiempo de recepción del evento en el origen de entrada (Event Hubs/IoT Hub/Blob Storage). 

De forma predeterminada, Stream Analytics procesa eventos por la *hora de llegada*, pero puede optar por procesar los eventos por la *hora del evento* utilizando la cláusula [TIMESTAMP BY](https://docs.microsoft.com/stream-analytics-query/timestamp-by-azure-stream-analytics) en la consulta. Las directivas de desorden y llegada tardía solo son aplicables si procesa los eventos por la hora del evento. Tenga en cuenta los requisitos de latencia y la adecuación a su escenario al configurar estos ajustes. 

## <a name="what-is-late-arrival-policy"></a>¿Que es una directiva de llegada tardía?

En ocasiones, los eventos llegan tarde debido a varias razones. Por ejemplo, un evento que llega tarde 40 segundos tarde tendrá como hora del evento 00:10:00 y como hora de llegada 00:10:40. Si establece la directiva de llegada tardía en 15 segundos, todos los eventos que llegan más tarde de 15 segundos se eliminarán (no se procesan por Stream Analytics) o bien se les ajustará la hora del evento. En el ejemplo anterior, como el evento llegó 40 segundos tarde (superior a la directiva establecida), su valor para hora del evento se ajustará al máximo de la directiva de llegada tardía 00:10:25 (hora de llegada - valor de la directiva de llegada tardía). La directiva predeterminada de llegada tardía es de 5 segundos.

## <a name="what-is-out-of-order-policy"></a>¿Qué es la directiva de desorden? 

Los eventos también pueden llegar desordenados. Después ajustar la hora del evento en función de la directiva de llegada tardía, también puede elegir eliminar o ajustar automáticamente los eventos que estén desordenados. Si establece esta directiva en 8 segundos, los eventos que lleguen desordenados, pero dentro de la ventana de 8 segundos, se reordenan según la hora del evento. Los eventos que llegan más tarde se eliminan o se ajustan al valor máximo de la directiva de desorden. La directiva de desorden predeterminada es de 0 segundos. 

## <a name="adjust-or-drop-events"></a>Ajuste o eliminación de eventos

Si los eventos llegan tarde o en desorden en función de las directivas que ha configurado, puede eliminar dichos eventos (no se procesan por Stream Analytics) o ajustarles el valor de la hora del evento.

Veamos un ejemplo de estas directivas en acción.
<br> **Directiva de llegada tardía:** 15 segundos
<br> **Directiva de desorden:** 8 segundos

| Nº. de evento | Hora del evento | Hora de llegada | System.Timestamp | Explicación |
| --- | --- | --- | --- | --- |
| **1** | 00:10:00  | 00:10:40  | 00:10:25  | El evento llegó tarde y fuera del nivel de tolerancia. Por lo tanto, la hora del evento se ajusta a la tolerancia máximo para la llegada tardía.  |
| **2** | 00:10:30 | 00:10:41  | 00:10:30  | El evento llegó tarde pero dentro del nivel de tolerancia. Por lo tanto, la hora del evento no se ajusta.  |
| **3** | 00:10:42 | 00:10:42 | 00:10:42 | El evento llegó tiempo. No es necesario ningún ajuste.  |
| **4** | 00:10:38  | 00:10:43  | 00:10:38 | Evento que ha llegado en desorden, pero dentro de la tolerancia de 8 segundos. Por lo tanto, la hora del evento no se ajusta. Con fines de análisis, este evento se considera anterior al evento número 4.  |
| **5** | 00:10:35 | 00:10:45  | 00:10:37 | Evento que ha llegado en desorden y fuera de la tolerancia de 8 segundos. Por lo tanto, la hora del evento se ajusta al máximo de la tolerancia de desorden. |

## <a name="can-these-settings-delay-output-of-my-job"></a>¿Esta configuración puede retrasar la salida de mi trabajo? 

Sí. De forma predeterminada, la directiva de desorden se establece en cero (00 minutos y 00 segundos). Si cambia el valor predeterminado, la primera salida del trabajo se retrasa en función de este valor (o superior). 

Si una de las particiones de las entradas no recibe eventos, debe esperar que la salida se retrase de acuerdo con el valor de la directiva de llegada tardía. Para saber por qué, lea la sección de error InputPartition más adelante. 

## <a name="i-see-lateinputevents-messages-in-my-activity-log"></a>Veo mensajes LateInputEvents en mi registro de actividad.

Estos mensajes se muestran para informarle de que los eventos han llegado tarde y se eliminan o ajustan según la configuración. Puede omitir estos mensajes si ha configurado la directiva de llegada tardía adecuadamente. 

Un ejemplo de ese mensaje es: <br>
<code>
{"message Time":"2019-02-04 17:11:52Z","error":null,
"message":"First Occurred: 02/04/2019 17:11:48 | Resource Name: ASAjob | Message: Source 'ASAjob' had 24 data errors of kind 'LateInputEvent' between processing times '2019-02-04T17:10:49.7250696Z' and '2019-02-04T17:11:48.7563961Z'. Input event with application timestamp '2019-02-04T17:05:51.6050000' and arrival time '2019-02-04T17:10:44.3090000' was sent later than configured tolerance.","type":"DiagnosticMessage","correlation ID":"49efa148-4asd-4fe0-869d-a40ba4d7ef3b"} 
</code>

## <a name="i-see-inputpartitionnotprogressing-in-my-activity-log"></a>Veo InputPartitionNotProgressing en mi registro de actividad.

Es probable que el origen de entrada (centro de eventos o IoT Hub) tenga varias particiones. Azure Stream Analytics genera una salida para una marca de tiempo t1 solo después de que todas las particiones que se combinan lleguen al menos a la hora t1. Por ejemplo, supongamos que la consulta proviene de una partición de un centro de eventos que a su vez tiene dos particiones. Una de las particiones, P1, tiene eventos hasta la hora t1. La otra partición, P2, tiene eventos hasta la hora t1 + x. A continuación, se generan los resultados hasta la hora t1. Sin embargo, si existe una cláusula Partition by PartitionId explícita, ambas particiones progresan de manera independiente. 

Cuando se combinan varias particiones del mismo flujo de entrada, la tolerancia de llegada tardía es la cantidad máxima de tiempo que cada partición dedica a esperar nuevos datos. Si hay una partición en su centro de eventos, o si la instancia de IoT Hub no recibe las entradas, la escala de tiempo para esa partición no avanza hasta que alcance el umbral de tolerancia de llegada tardía. Esto retrasa la salida de acuerdo con el umbral de tolerancia de llegada tardía. En estos casos, podría aparecer el siguiente mensaje de error:
<br><code>
{"message Time":"2/3/2019 8:54:16 PM UTC","message":"Input Partition [2] does not have additional data for more than [5] minute(s). Partition will not progress until either events arrive or late arrival threshold is met.","type":"InputPartitionNotProgressing","correlation ID":"2328d411-52c7-4100-ba01-1e860c757fc2"} 
</code><br><br>
Este mensaje es para informarle de que al menos una partición de entrada está vacía y retrasará la salida de acuerdo con el umbral de llegada tardía. Para solucionar este problema, se recomienda que opte por una de estas opciones:  
1. Asegurarse de que todas las particiones del centro de eventos o IoT Hub reciben una entrada. 
2. Utilizar la cláusula Partition by PartitionId en la consulta. 

## <a name="why-do-i-see-a-delay-of-5-seconds-even-when-my-late-arrival-policy-is-set-to-0"></a>¿Por qué veo un retraso de 5 segundos incluso cuando mi directiva de llegada tardía está establecida en 0?
Esto sucede si hay una partición de entrada que nunca ha recibido ninguna entrada. Puede comprobar las métricas de entrada por partición para validar este comportamiento. 

Cuando una partición no tiene ningún dato durante más tiempo del umbral de llegada tardía configurado, Stream Analytics adelanta la marca de tiempo de la aplicación como se explica en la sección consideraciones de ordenación de eventos. Esto requiere una duración de llegada estimada. Si la partición nunca tenía datos, Stream Analytics calcula la hora de llegada como *hora local: 5 segundos*. Debido a las particiones que nunca tenían datos, podría mostrar un retraso de marca de agua de 5 segundos.  

## <a name="next-steps"></a>Pasos siguientes
* [Consideraciones sobre el control de tiempo](stream-analytics-time-handling.md)
* [Métricas disponibles para Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-monitoring#metrics-available-for-stream-analytics)
