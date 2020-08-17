---
title: 'Servicio de detección de anomalías para la facturación de uso medido: Microsoft Azure Marketplace'
description: Describe cómo funciona la detección de anomalías, cuándo se envían notificaciones y qué hacer con ellas, y las opciones de soporte técnico.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.author: mingshen
author: mingshen-ms
ms.date: 06/10/2020
ms.openlocfilehash: cb134c82f231eb8b6329b1acafb181032edd6936
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320092"
---
# <a name="anomaly-detection-service-for-metered-billing"></a>Servicio de detección de anomalías para la facturación de uso medido

El [servicio de medición de Marketplace](marketplace-metering-service-apis-faq.md), le permite crear ofertas en el programa comercial de Marketplace que se cobran conforme a unidades no estándar. Con la facturación de uso medido, puede enviar eventos de uso del cliente a Microsoft y nosotros prepararemos la facturación según ese uso.

Unos datos de uso incorrectos pueden deberse a diversas causas como, por ejemplo, errores, configuraciones incorrectas en el seguimiento del consumo o fraude. Los datos de uso incorrectos pueden producir cargos incorrectos al cliente y conflictos de facturación.

Para reducir este riesgo, nuestro servicio de detección de anomalías aplica algoritmos de aprendizaje automático para determinar el comportamiento normal de la facturación de uso medido, analizar el uso de facturación medida y detectar anomalías con una intervención mínima del usuario.

Se le notificará si se detectan anomalías en el uso de facturación medida. Esto le ofrece la oportunidad de investigar y enviarnos una notificación si se confirma que una anomalía es un problema real, momento en el cual se pueden realizar acciones para solucionar el problema de facturación del cliente de forma proactiva.

Además de picos repentinos, interrupciones y cambios de tendencia en el uso de facturación medida, nuestro modelo también tiene en cuenta los efectos estacionales. Dado que la facturación de uso medido se comunica a través de datos de uso por encima del límite, nuestro modelo también es capaz de tratar correctamente con períodos largos de datos que faltan.

A continuación se muestran ejemplos de resultados de detección de anomalías. El intervalo esperable aparece como una banda amarilla. El intervalo aceptable del uso de facturación medida se indica mediante estrellas verdes en la banda. El uso para facturación fuera de la banda se muestra como un punto rojo.  

Anomalías detectadas fuera de una tendencia predecible:

![Muestra las anomalías que se detectan fuera de una tendencia predecible.](media/anomaly-1.png)

Anomalías detectadas fuera de una tendencia cíclica periódica:

![Muestra las anomalías que se detectan fuera de una tendencia cíclica periódica.](media/anomaly-2.png)

Anomalías detectadas en una tendencia al alza:

![Muestra las anomalías detectadas en una tendencia al alza.](media/anomaly-3.png)

## <a name="how-anomaly-detection-service-works"></a>Funcionamiento del servicio de detección de anomalías

La detección de anomalías se habilita automáticamente para todos los usos de facturación medida. Cuando se envían los eventos de uso a Microsoft, el servicio de detección de anomalías crea un modelo de los valores esperables basado en los datos de uso del pasado. Este modelo se ejecuta semanalmente.

La detección de anomalías funciona en un nivel por medidor y por cliente. Esto significa que cada medidor de cada cliente tendrá un modelo entrenado basado en el patrón de uso en el pasado de este cliente para este valor de medición.

El modelo funciona mediante la generación de intervalos de confianza retrospectiva. La previsión de serie temporal es un modelo de suma generalizado que se compone de una parte de predicción de tendencias y una parte de estacionalidad. Dado que el modelo se formula como una tarea de regresión, puede controlar correctamente períodos largos de datos que faltan. Si una observación está fuera de los intervalos de confianza predichos, significa que la observación no se puede explicar según los patrones históricos de la facturación de uso medido y, por lo tanto, puede constituir una anomalía.

## <a name="anomaly-detection-notification"></a>Notificaciones de detección de anomalías

Se enviarán por correo electrónico avisos de detección de anomalías semanalmente. Estos avisos incluyen todas las anomalías detectadas esa semana para todos los medidores y clientes. Este correo electrónico se envía a los contactos de **ingeniería** y **soporte técnico** que se proporcionaron cuando creó la oferta.

Se espera que compruebe si las anomalías detectadas son problemas reales y, si es así, se ponga en contacto con Microsoft para informar del uso incorrecto (consulte la sección de soporte técnico a continuación).

Si confirma que las anomalías detectadas constituyen un uso normal, no es necesario realizar ninguna otra acción. Sin embargo, si una anomalía representa un riesgo financiero potencialmente alto, podremos ponernos en contacto con usted para confirmar el uso.  

## <a name="when-and-how-to-get-support"></a>Obtención de soporte técnico

Si ha enviado un uso incorrecto a Microsoft y esto ha dado o dará lugar a un cargo menor al cliente, Microsoft no iniciará una factura al cliente por la utilización no informada o no le pagará por dicha utilización. Tendrá que asumir la pérdida de ingresos debido a la falta de información.

Si se aplica uno de los casos siguientes, puede abrir una incidencia de soporte técnico para solicitar el reembolso o el ajuste de facturación de los clientes:

- Ha confirmado que una de las anomalías encontradas es un problema real y la utilización incorrecta daría como resultado **cargos extra** al cliente.
- Descubre que nos ha enviado una utilización incorrecta y que esto daría como resultado **cargos extra** al cliente.
- Le gustaría solicitar un reembolso por el cargo del uso de facturación medida de su cliente.

Para enviar una incidencia:

1. Vaya a la página de soporte técnico. En el **cuadro de información sobre la incidencia**, escriba "uso incorrecto".
2. En los temas de soporte técnico, en la lista desplegable de resultados de la búsqueda, seleccione una de las siguientes opciones:
    - **Marketplace comercial** > **Facturación de uso medido** > **Se ha enviado un uso incorrecto para la oferta de aplicaciones de Azure**, o
    - **Marketplace comercial** > **Facturación de uso medido** > **Se ha enviado un uso incorrecto para la oferta de SaaS**
3. En **Siguiente paso**, seleccione el botón **Revisar soluciones** para que se le redirija para iniciar sesión en el Centro de partners y poder enviar una incidencia de soporte técnico.

Para ver más opciones de soporte técnico para editores, consulte [Soporte técnico para el programa Marketplace comercial en el Centro de partners](support.md).

## <a name="next-step"></a>Paso siguiente

- Más información sobre la [API del servicio de medición de Marketplace](marketplace-metering-service-apis.md).
