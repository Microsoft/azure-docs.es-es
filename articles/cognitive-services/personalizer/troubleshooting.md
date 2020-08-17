---
title: 'Solución de problemas: Personalizer'
description: En este artículo se incluyen respuestas a preguntas para solucionar problemas sobre Personalizer.
ms.topic: troubleshooting
ms.date: 02/26/2020
ms.openlocfilehash: 55b9068dbd7bfbb3ea3972c3381fda6d9cde5076
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87126115"
---
# <a name="personalizer-troubleshooting"></a>Solución de problemas de Personalizer

En este artículo se incluyen respuestas a preguntas para solucionar problemas sobre Personalizer.

## <a name="configuration-issues"></a>Problemas de configuración

### <a name="i-changed-a-configuration-setting-and-now-my-loop-isnt-performing-at-the-same-learning-level-what-happened"></a>He cambiado un valor de configuración y ahora mi bucle no funciona en el mismo nivel de aprendizaje. ¿Qué ha ocurrido?

Algunos valores de configuración [restablecen el modelo](how-to-settings.md#settings-that-include-resetting-the-model). Los cambios de configuración deben planearse con cuidado.

### <a name="when-configuring-personalizer-with-the-api-i-received-an-error-what-happened"></a>Al configurar Personalizer con la API, se ha recibido un error. ¿Qué ha ocurrido?

Si usa una única solicitud de API para configurar el servicio y cambiar el comportamiento de aprendizaje, recibirá un error. Debe realizar dos llamadas API independientes: en primer lugar, para configurar el servicio y, a continuación, para cambiar el comportamiento de aprendizaje.

## <a name="transaction-errors"></a>Errores de transacción

### <a name="i-get-an-http-429-too-many-requests-response-from-the-service-what-can-i-do"></a>Obtengo una respuesta HTTP 429 (demasiadas solicitudes) del servicio. ¿Qué puedo hacer?

Si seleccionó un plan de tarifa gratis al crear la instancia de Personalizer, existe un límite de cuota para el número permitido de solicitudes de Rank. Revise la tasa de llamadas API para Rank API (en el panel Métricas de Azure Portal del recurso de Personalizer) y ajuste el plan de tarifa (en el panel Plan de tarifa) si tiene previsto que el volumen de llamadas aumente hasta superar el umbral del plan de tarifa seleccionado.

### <a name="im-getting-a-5xx-error-on-rank-or-reward-apis-what-should-i-do"></a>Obtengo un error 5xx en las API Rank o Reward. ¿Cuál debo hacer?

Estos problemas deben ser fáciles de solucionar. Si persisten, póngase en contacto con el servicio de soporte técnico al seleccionar **Nueva solicitud de soporte técnico** en la sección **Soporte técnico y solución de problemas**en Azure Portal del recurso de Personalizer.

## <a name="learning-loop"></a>Bucle de aprendizaje

### <a name="the-learning-loop-doesnt-attain-a-100-match-to-the-system-without-personalizer-how-do-i-fix-this"></a>El bucle de aprendizaje no logra una coincidencia del 100 % con el sistema sin Personalizer. ¿Cómo puedo corregirlo?

Motivos por los que no alcanza su objetivo con el bucle de aprendizaje:
* No hay suficientes características enviadas con la llamada API Rank.
* Errores en las características enviadas, como, por ejemplo, el envío de datos de características no agregados, como marcas de tiempo a la API Rank.
* Errores con el procesamiento de bucles, como no enviar datos de recompensa a la API Reward para eventos.

Para corregir este problema, debe cambiar el procesamiento mediante la modificación de las características enviadas al bucle o asegurarse de que la recompensa es una evaluación correcta de la calidad de la respuesta de Rank.

### <a name="the-learning-loop-doesnt-seem-to-learn-how-do-i-fix-this"></a>El bucle de aprendizaje parece que no aprende. ¿Cómo puedo corregirlo?

El bucle de aprendizaje necesita unas pocas miles de llamadas a Reward antes de que las llamadas a Rank se prioricen eficazmente.

Si no está seguro de cómo se está comportando el bucle de aprendizaje actualmente, ejecute una [evaluación sin conexión](concepts-offline-evaluation.md) y aplique la directiva de aprendizaje corregido.

### <a name="i-keep-getting-rank-results-with-all-the-same-probabilities-for-all-items-how-do-i-know-personalizer-is-learning"></a>Sigo obteniendo resultados de clasificación con las mismas probabilidades para todos los elementos. ¿Cómo puedo saber si Personalizer está aprendiendo?

Personalizer devuelve las mismas probabilidades en un resultado de la Rank API cuando acaba de iniciar y tiene un modelo _vacío_, o cuando se restablece el bucle de Personalizer y el modelo todavía está dentro del período establecido en **Frecuencia de actualización del modelo**.

Cuando comience el nuevo período de actualización, se usará el modelo actualizado y verá el cambio de probabilidades.

### <a name="the-learning-loop-was-learning-but-seems-to-not-learn-anymore-and-the-quality-of-the-rank-results-isnt-that-good-what-should-i-do"></a>El bucle de aprendizaje estaba aprendiendo, pero parece que ya no es así, y la calidad de los resultados de Rank no es tan buena. ¿Cuál debo hacer?

* Asegúrese de haber completado y aplicado una evaluación en Azure Portal para ese recurso de Personalizer (bucle de aprendizaje).
* Asegúrese de que se hayan enviado (a través de Reward API) y procesado todas las recompensas.

### <a name="how-do-i-know-that-the-learning-loop-is-getting-updated-regularly-and-is-used-to-score-my-data"></a>¿Cómo puedo saber si el bucle de aprendizaje se está actualizando periódicamente y usándose para puntuar los datos?

Puede encontrar la hora de la última actualización del modelo en la página **Configuración de modelo y aprendizaje** de Azure Portal. Si ve una marca de tiempo antigua, es probable que no esté enviando las llamadas de Rank y Reward. Si el servicio no tiene datos entrantes, no se actualiza el aprendizaje. Si nota que el bucle de aprendizaje no se actualiza con suficiente frecuencia, puede modificar la **Frecuencia de actualización del modelo** del bucle.

## <a name="offline-evaluations"></a>Evaluaciones sin conexión

### <a name="an-offline-evaluations-feature-importance-returns-a-long-list-with-hundreds-or-thousands-of-items-what-happened"></a>La importancia de las características de una evaluación sin conexión devuelve una lista larga con cientos o miles de elementos. ¿Qué ha ocurrido?

Esto suele deberse a que se enviaron marcas de tiempo, Id. de usuario u otras características específicas.

### <a name="i-created-an-offline-evaluation-and-it-succeeded-almost-instantly-why-is-that-i-dont-see-any-results"></a>He creado una evaluación sin conexión y se realizó con éxito casi al instante. ¿Por qué ocurre esto? ¿No se ve ningún resultado?

La evaluación sin conexión utiliza los datos de modelo entrenado de los eventos en ese período de tiempo. Si no ha enviado ningún dato en el período entre las horas inicial y final de la evaluación, esta se completará sin ningún resultado. Envíe una nueva evaluación sin conexión seleccionando un intervalo de tiempo con los eventos que sabe que se enviaron a Personalizer.

## <a name="learning-policy"></a>Directiva de aprendizaje

### <a name="how-do-i-import-a-learning-policy"></a>¿Cómo puedo importar una directiva de aprendizaje?

Obtenga más información sobre los [conceptos de la directiva de aprendizaje](concept-active-learning.md#understand-learning-policy-settings) y [cómo aplicar](how-to-manage-model.md) una nueva directiva de aprendizaje. Si no quiere seleccionar una directiva de aprendizaje, puede usar [la evaluación sin conexión](how-to-offline-evaluation.md) para sugerir una directiva de aprendizaje en función de los eventos actuales.


## <a name="security"></a>Seguridad

### <a name="the-api-key-for-my-loop-has-been-compromised-what-can-i-do"></a>La clave de API para mi bucle está en peligro. ¿Qué puedo hacer?

Puede regenerar una clave después de intercambiarla con los clientes para que usen otra clave. Tener dos claves le permite propagar la clave de manera diferida sin producir tiempo de inactividad. Se recomienda hacerlo de forma periódica como medida de seguridad.


## <a name="next-steps"></a>Pasos siguientes

[Configuración de la frecuencia de actualización del modelo](how-to-settings.md#model-update-frequency)