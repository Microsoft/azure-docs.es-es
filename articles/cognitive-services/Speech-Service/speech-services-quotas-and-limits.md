---
title: Límites y cuotas de los servicios de voz
titleSuffix: Azure Cognitive Services
description: Referencia rápida, descripción detallada y procedimientos recomendados para las cuotas y los límites de los servicios de voz de Azure Cognitive Services.
services: cognitive-services
author: alexeyo26
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: alexeyo
ms.openlocfilehash: 554dd0967979bc2457c3a9c8371152e09535381f
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690143"
---
# <a name="speech-services-quotas-and-limits"></a>Límites y cuotas de los servicios de voz

Este artículo contiene una referencia rápida y la **descripción detallada** de las cuotas y los límites de los servicios de voz de Azure Cognitive Services para todos los [planes de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/). También contiene algunos procedimientos recomendados para evitar la limitación de solicitudes. 

## <a name="quotas-and-limits-quick-reference"></a>Referencia rápida de cuotas y límites
Ir a [Cuotas y límites de la conversión de texto a voz](#text-to-speech-quotas-and-limits-per-speech-resource)
### <a name="speech-to-text-quotas-and-limits-per-speech-resource"></a>Cuotas y límites de la conversión de voz en texto por recurso de voz
En la tabla siguiente, los parámetros sin la fila "Ajustable" **no** son ajustables para todos los planes de tarifa.

| Quota | Gratis (F0)<sup>1</sup> | Estándar (S0) |
|--|--|--|
| **Límite de solicitudes simultáneas de transcripción en línea (modelos base y personalizado)** |  |  |
| Valor predeterminado | 1 | 20 |
| Ajustable | No<sup>2</sup> | Sí<sup>2</sup> |
| **Límite de solicitudes de API REST (puntos de conexión de [API Management](../../api-management/api-management-key-concepts.md))** | 100 solicitudes por 10 segundos | 100 solicitudes por 10 segundos |
| **Tamaño máximo del archivo de conjunto de datos para la importación de datos** | 2 GB | 2 GB |
| **Tamaño máximo de blobs de entrada para Batch Transcription** | N/D | 2,5 GB |
| **Tamaño máximo del contenedor de blobs para Batch Transcription** | N/D | 5 GB |
| **Número máximo de blobs por contenedor para Batch Transcription** | N/D | 10000 |
| **Número máximo de trabajos en ejecución simultánea para Batch Transcription** | N/D | 2000  |

<sup>1</sup> Para el plan de tarifa **Gratis (F0)** , consulte también las asignaciones mensuales en la [página de precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).<br/>
<sup>2</sup> Consulte las [explicaciones adicionales](#detailed-description-quota-adjustment-and-best-practices), los [procedimientos recomendados](#general-best-practices-to-mitigate-throttling-during-autoscaling) y las [instrucciones de ajuste](#speech-to-text-increasing-online-transcription-concurrent-request-limit).<br/> 

### <a name="text-to-speech-quotas-and-limits-per-speech-resource"></a>Cuotas y límites de la conversión de texto en voz por recurso de voz
En la tabla siguiente, los parámetros sin la fila "Ajustable" **no** son ajustables para todos los planes de tarifa.

| Quota | Gratis (F0)<sup>3</sup> | Estándar (S0) |
|--|--|--|
| **Número máximo de transacciones por segundo (TPS) para las voces estándar y neuronal** | 200<sup>4</sup> | 200<sup>4</sup> |  |
| **Límite de solicitudes simultáneas para la voz personalizada** |  |  |
| Valor predeterminado | 10 | 10 |
| Ajustable | No<sup>5</sup> | Sí<sup>5</sup> |
| **Cuotas específicas de HTTP** |  |
| Longitud máxima de audio generada por solicitud | 10 min | 10 min |
| Número máximo de etiquetas `<voice>` distintivas en SSML | 50 | 50 |
| **Cuotas específicas de WebSocket** |  |  |
|Longitud máxima de audio generada por turno | 10 min | 10 min |
|Tamaño máximo de mensaje SSML por turno |64 KB |64 KB |
| **Límite de solicitudes de API REST** | 20 solicitudes por minuto | 25 solicitudes por 5 segundos |


<sup>3</sup> Para el plan de tarifa **Gratis (F0)** , consulte también las asignaciones mensuales en la [página de precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).<br/>
<sup>4</sup> Consulte las [explicaciones adicionales](#detailed-description-quota-adjustment-and-best-practices) y los [procedimientos recomendados](#general-best-practices-to-mitigate-throttling-during-autoscaling).<br/>
<sup>5</sup> Consulte las [explicaciones adicionales](#detailed-description-quota-adjustment-and-best-practices), los [procedimientos recomendados](#general-best-practices-to-mitigate-throttling-during-autoscaling) y las [instrucciones de ajuste](#text-to-speech-increasing-transcription-concurrent-request-limit-for-custom-voice).<br/> 

## <a name="detailed-description-quota-adjustment-and-best-practices"></a>Descripción detallada, ajuste de cuota y procedimientos recomendados
Antes de solicitar un aumento de cuota (si procede), asegúrese de que sea necesario. El servicio de voz emplea tecnologías de escalado automático para poner los recursos computacionales necesarios en modo "a petición" y, al mismo tiempo, mantener bajos los costos del cliente gracias a que no es necesario el mantenimiento de una cantidad excesiva de capacidad de hardware. Cada vez que la aplicación reciba un código de respuesta 429 ("Demasiadas solicitudes") mientras la carga de trabajo se encuentra dentro de los límites definidos (consulte [Referencia rápida de cuotas y límites](#quotas-and-limits-quick-reference)), la explicación más probable es que el servicio se esté escalando verticalmente según la demanda y que no haya alcanzado aún la escala necesaria, por lo que no cuenta inmediatamente con suficientes recursos para atender la solicitud. Este estado suele ser transitorio y no debería durar mucho.

### <a name="general-best-practices-to-mitigate-throttling-during-autoscaling"></a>Procedimientos recomendados generales para mitigar la limitación durante el escalado automático
Para minimizar los problemas relacionados con la limitación (código de respuesta 429), se recomienda usar las técnicas siguientes:
- Implemente lógica de reintento en la aplicación
- Evite cambios bruscos en la carga de trabajo. Aumente la carga de trabajo gradualmente. <br/>
*Ejemplo.* La aplicación usa la conversión de texto en voz y la carga de trabajo actual es de 5 TPS (transacciones por segundo). En el siguiente segundo, aumenta la carga a 20 TPS (es decir, cuatro veces más). El servicio comienza inmediatamente el escalado vertical para satisfacer la nueva carga, pero es probable que no pueda hacerlo en un segundo, por lo que algunas de las solicitudes obtendrán el código de respuesta 429.   
- Prueba de diferentes patrones de aumento de carga
  - Consulte [Ejemplo de conversión de voz en texto](#speech-to-text-example-of-a-workload-pattern-best-practice).
- Cree recursos de voz adicionales en las mismas regiones u otras diferentes y distribuya la carga de trabajo entre ellas mediante la técnica "round robin". Esto es especialmente importante para el parámetro **Text-to-Speech TPS (transactions per second)** (TPS de conversión de texto en voz [transacciones por segundo]), que se establece en 200 por recurso de voz y no se puede ajustar.  

En las secciones siguientes se describen los casos específicos de ajuste de las cuotas.<br/>
Vaya a [Conversión de texto en voz: aumento del límite de solicitudes simultáneas de transcripciones para voz personalizada](#text-to-speech-increasing-transcription-concurrent-request-limit-for-custom-voice).

### <a name="speech-to-text-increasing-online-transcription-concurrent-request-limit"></a>Conversión de voz en texto: aumento del límite de solicitudes simultáneas de transcripciones en línea
De forma predeterminada, el número de solicitudes simultáneas está limitado a 20 recursos por voz (modelo base) o por punto de conexión personalizado (modelo personalizado). En el plan de tarifa estándar, esta cantidad se puede aumentar. Antes de enviar la solicitud, asegúrese de que está familiarizado con el material de [esta sección](#detailed-description-quota-adjustment-and-best-practices) y tenga en cuenta estos [procedimientos recomendados](#general-best-practices-to-mitigate-throttling-during-autoscaling).

El aumento del límite de solicitudes simultáneas **no** afecta directamente a los costos. Los servicios de voz usan el modelo "pago por uso". El límite define hasta dónde se puede escalar el servicio antes de empezar a limitar las solicitudes.

Los límites de solicitudes simultáneas de los modelos **base** y **personalizado** se deben ajustar **por separado**.

El valor existente del parámetro de límite de solicitudes simultáneas **no** es visible en Azure Portal, en las herramientas de línea de comandos o las solicitudes de API. Para comprobar el valor existente, cree una solicitud de soporte técnico de Azure.

>[!NOTE]
>Los [contenedores de voz](speech-container-howto.md) no requieren aumentos en los límites de solicitudes simultáneas, ya que solo están restringidos por las CPU del hardware en el que se hospedan.

#### <a name="have-the-required-information-ready"></a>Tenga lista la información necesaria:
- Para el **modelo base**:
  - Identificador del recurso de voz
  - Region
- Para el **modelo personalizado**: 
  - Region
  - Identificador del punto de conexión personalizado

- **Cómo obtener información (modelo base)** :  
  - Vaya a [Azure Portal](https://portal.azure.com/).
  - Seleccione el recurso de voz para el que desea aumentar el límite de solicitudes simultáneas.
  - Seleccione *Propiedades* (grupo *Administración de recursos*) 
  - Copie y guarde los valores de los siguientes campos:
    - **Identificador del recurso**
    - **Ubicación** (la región del punto de conexión)

- **Cómo obtener información (modelo personalizado)** :
  - Vaya al portal de [Speech Studio](https://speech.microsoft.com/).
  - Inicie sesión (si es necesario).
  - Vaya a Custom Speech.
  - Seleccione su proyecto.
  - Vaya a *Deployment* (Implementación).
  - Seleccione el punto de conexión necesario.
  - Copie y guarde los valores de los siguientes campos:
    - **Service Region** (Región del servicio) (la región del punto de conexión)
    - **Endpoint ID** (Id. del punto de conexión)
  
#### <a name="create-and-submit-support-request"></a>Creación y envío de una solicitud de soporte técnico
Inicie el aumento del límite de solicitudes simultáneas para el recurso o, si es necesario, compruebe el límite actual enviando la solicitud de soporte técnico:

- Asegúrese de que tiene la [información necesaria](#have-the-required-information-ready).
- Vaya a [Azure Portal](https://portal.azure.com/).
- Seleccione el recurso de voz para el que le gustaría aumentar (o comprobar) el límite de solicitudes simultáneas
- Seleccione *Nueva solicitud de soporte técnico* (grupo *Soporte técnico y solución de problemas*). 
- Aparecerá una nueva ventana con información rellenada automáticamente sobre la suscripción y el recurso de Azure.
- Rellene el campo *Resumen* (por ejemplo, "Aumento del límite de solicitudes simultáneas de STT").
- En *Tipo de problema*, seleccione "Quota or Subscription issues" (Problemas de cuota o suscripción).
- En *Subtipo de problema*, seleccione:
  - "Quota or concurrent requests increase" (Aumento de cuota o solicitudes simultáneas) para aumentar las solicitudes
  - "Quota or usage validation" (Validación de cuota o uso) para comprobar el límite existente
- Haga clic en *Siguiente: Soluciones*
- Continúe con la creación de la solicitud.
- En la pestaña *Detalles*, rellene el campo *Descripción*:
  - Tenga en cuenta que la solicitud es sobre la cuota de **conversión de voz en texto**.
  - Modelo **base** o **personalizado**
  - La información de recursos de Azure que [recopiló antes](#have-the-required-information-ready). 
  - Termine de escribir la información necesaria y haga clic en el botón *Crear* en la pestaña *Revisar y crear*.
  - Anote el número de la solicitud de soporte técnico que aparece en las notificaciones de Azure Portal. Pronto se pondrán en contacto con usted para continuar con el procesamiento.

### <a name="speech-to-text-example-of-a-workload-pattern-best-practice"></a>Conversión de voz en texto: ejemplo de un procedimiento recomendado para un patrón de carga de trabajo
En este ejemplo se presenta la estrategia que se recomienda seguir para mitigar la posible limitación de solicitudes debido a que el [escalado automático está en curso](#detailed-description-quota-adjustment-and-best-practices). No es una receta exacta, sino una mera plantilla que le invitamos a seguir y ajustar según sea necesario.

Supongamos que un recurso de voz tiene el límite de solicitudes simultáneas establecido en 300. Inicie la carga de trabajo a partir de 20 conexiones simultáneas y aumente la carga en 20 conexiones simultáneas cada 1,5 o 2 minutos. Controle las respuestas del servicio e implemente la lógica que invierte la operación (reduce la carga) si se obtienen demasiados códigos de respuesta 429. A continuación, vuelva a intentarlo según el patrón de 1-2-4-4 minutos. (Es decir, reintentar el aumento de la carga en 1 minuto, si sigue sin funcionar en 2 minutos, y así sucesivamente).

Por lo general, se recomienda encarecidamente probar la carga de trabajo y los patrones de carga de trabajo antes de pasar a producción.

### <a name="text-to-speech-increasing-transcription-concurrent-request-limit-for-custom-voice"></a>Conversión de texto a voz: aumento del límite de solicitudes simultáneas de transcripción para la voz personalizada
De forma predeterminada, el número de solicitudes simultáneas para un punto de conexión de voz personalizada está limitado a 10. En el plan de tarifa estándar, esta cantidad se puede aumentar. Antes de enviar la solicitud, asegúrese de que está familiarizado con el material de [esta sección](#detailed-description-quota-adjustment-and-best-practices) y tenga en cuenta estos [procedimientos recomendados](#general-best-practices-to-mitigate-throttling-during-autoscaling).

El aumento del límite de solicitudes simultáneas **no** afecta directamente a los costos. Los servicios de voz usan el modelo "pago por uso". El límite define hasta dónde se puede escalar el servicio antes de empezar a limitar las solicitudes.

El valor existente del parámetro de límite de solicitudes simultáneas **no** es visible en Azure Portal, en las herramientas de línea de comandos o las solicitudes de API. Para comprobar el valor existente, cree una solicitud de soporte técnico de Azure.

>[!NOTE]
>Los [contenedores de voz](speech-container-howto.md) no requieren aumentos en los límites de solicitudes simultáneas, ya que solo están restringidos por las CPU del hardware en el que se hospedan.

#### <a name="prepare-the-required-information"></a>Prepare la información necesaria:
Para crear una solicitud de aumento, deberá proporcionar su región de implementación y el identificador del punto de conexión personalizado. Para obtenerlo, realice las acciones siguientes: 

- Vaya al portal de [Speech Studio](https://speech.microsoft.com/).
- Inicie sesión (si es necesario).
- Vaya a *Custom Voice* (Voz personalizada).
- Seleccione su proyecto.
- Vaya a *Deployment* (Implementación).
- Seleccione el punto de conexión necesario.
- Copie y guarde los valores de los siguientes campos:
    - **Service Region** (Región del servicio) (la región del punto de conexión)
    - **Endpoint ID** (Id. del punto de conexión)
  
#### <a name="create-and-submit-support-request"></a>Creación y envío de una solicitud de soporte técnico
Inicie el aumento del límite de solicitudes simultáneas para el recurso o, si es necesario, compruebe el límite actual enviando la solicitud de soporte técnico:

- Asegúrese de que tiene la [información necesaria](#prepare-the-required-information).
- Vaya a [Azure Portal](https://portal.azure.com/).
- Seleccione el recurso de voz para el que le gustaría aumentar (o comprobar) el límite de solicitudes simultáneas
- Seleccione *Nueva solicitud de soporte técnico* (grupo *Soporte técnico y solución de problemas*). 
- Aparecerá una nueva ventana con información rellenada automáticamente sobre la suscripción y el recurso de Azure.
- Rellene el campo *Resumen* (por ejemplo, "Aumento del límite de solicitudes simultáneas de los puntos de conexión personalizados de STT").
- En *Tipo de problema*, seleccione "Quota or Subscription issues" (Problemas de cuota o suscripción).
- En *Subtipo de problema*, seleccione:
  - "Quota or concurrent requests increase" (Aumento de cuota o solicitudes simultáneas) para aumentar las solicitudes
  - "Quota or usage validation" (Validación de cuota o uso) para comprobar el límite existente
- Haga clic en *Siguiente: Soluciones*
- Continúe con la creación de la solicitud.
- En la pestaña *Detalles*, rellene el campo *Descripción*:
  - Tenga en cuenta que la solicitud es sobre la cuota de **conversión de texto en voz**.
  - La información de recursos de Azure que [recopiló antes](#prepare-the-required-information). 
  - Termine de escribir la información necesaria y haga clic en el botón *Crear* en la pestaña *Revisar y crear*.
  - Anote el número de la solicitud de soporte técnico que aparece en las notificaciones de Azure Portal. Pronto se pondrán en contacto con usted para continuar con el procesamiento.

