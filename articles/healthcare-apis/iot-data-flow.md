---
title: 'Conceptos: Flujo de datos en la característica de conector de Azure IoT para FHIR (versión preliminar) de Azure API for FHIR'
description: Información sobre el flujo de datos del conector de Azure IoT para FHIR (versión preliminar). El conector de Azure IoT para FHIR (versión preliminar) ingiere, normaliza, agrupa, transforma y conserva los datos de IoMT en Azure API for FHIR.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: conceptual
ms.date: 07/31/2020
ms.author: punagpal
ms.openlocfilehash: 43b7bcba97617d6931fd5c191e62e833a25bf89d
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513387"
---
# <a name="azure-iot-connector-for-fhir-preview-data-flow"></a>Flujo de datos del conector de Azure IoT para FHIR (versión preliminar)

En este artículo se proporciona información general sobre el flujo de datos en el conector de Azure IoT para FHIR*. Aprenderá las diferentes fases de procesamiento de datos en el conector de Azure IoT para FHIR que transforman los datos del dispositivo en recursos [Observación](https://www.hl7.org/fhir/observation.html) basados en FHIR.

![Flujo de datos del conector de Azure IoT para FHIR](media/concepts-iot-data-flow/iot-connector-data-flow.png)

En el diagrama anterior se muestran los flujos de datos comunes que usan el conector de Azure IoT para FHIR. 

A continuación se muestran diferentes fases por las que pasan los datos una vez recibidos por el conector de Azure IoT para FHIR.

## <a name="ingest"></a>Ingesta
La ingesta es la primera fase en la que los datos del dispositivo se reciben en el conector de Azure IoT para FHIR. El punto de conexión de ingesta de los datos del dispositivo se hospeda en un [centro de eventos de Azure](https://docs.microsoft.com/azure/event-hubs/). La plataforma Azure Event Hubs admite gran escala y rendimiento, y tiene capacidad para recibir y procesar millones de mensajes por segundo. También permite que el conector de Azure IoT para FHIR consuma mensajes de forma asincrónica, lo que elimina la necesidad de que los dispositivos esperen mientras se procesan los datos del dispositivo.

> [!NOTE]
> En este momento el único formato admitido para los datos del dispositivo es JSON.

## <a name="normalize"></a>Normalizar
La normalización es la siguiente fase, en la que se recuperan los datos del dispositivo del centro de eventos de Azure anterior y se procesan mediante plantillas de asignación de dispositivos. Este proceso de asignación da como resultado la transformación de los datos del dispositivo en un esquema normalizado. 

El proceso de normalización no solo simplifica el procesamiento de datos en fases posteriores, sino que también proporciona la capacidad de proyectar un mensaje de entrada en varios mensajes normalizados. Por ejemplo, un dispositivo puede enviar varias constantes vitales con la temperatura del cuerpo, la frecuencia cardíaca, la presión arterial y la frecuencia respiratoria en un solo mensaje. Este mensaje de entrada creará cuatro recursos FHIR independientes. Cada recurso representaría una constante vital diferente, con el mensaje de entrada proyectado en cuatro mensajes normalizados diferentes.

## <a name="group"></a>Grupo
La agrupación es la siguiente fase, en la que los mensajes normalizados disponibles en la fase anterior se agrupan mediante el uso de tres parámetros diferentes: identidad de dispositivo, tipo de medida y período de tiempo.

La agrupación de la identidad del dispositivo y el tipo de medida habilita el uso del tipo de medida [SampledData](https://www.hl7.org/fhir/datatypes.html#SampledData). Este tipo proporciona una manera concisa de representar una serie de medidas basadas en el tiempo desde un dispositivo en FHIR. Y el período de tiempo controla la latencia a la que se escriben los recursos de observación generados por el conector de Azure IoT para FHIR en Azure API for FHIR.

> [!NOTE]
> El valor de período de tiempo se establece de manera predeterminada en 15 minutos y no se puede configurar para la versión preliminar.

## <a name="transform"></a>Transformación
En la fase de transformación, los mensajes agrupados y normalizados se procesan mediante las plantillas de asignación de FHIR. Los mensajes que coinciden con un tipo de plantilla se transforman en recursos de observación basados en FHIR, tal y como se especifica mediante la asignación.

En este punto, el recurso [Dispositivo](https://www.hl7.org/fhir/device.html), junto con su recurso asociado [Paciente](https://www.hl7.org/fhir/patient.html), también se recupera del servidor FHIR con el identificador de dispositivo presente en el mensaje. Estos recursos se agregan como una referencia al recurso Observación que se va a crear.

> [!NOTE]
> Todas las búsquedas de identidad se almacenan en caché una vez que se resuelven para reducir la carga en el servidor de FHIR. Si planea reutilizar dispositivos con varios pacientes, se recomienda crear un recurso de dispositivo virtual específico para el paciente y enviar el identificador de dispositivo virtual en la carga del mensaje. El dispositivo virtual se puede vincular al recurso de dispositivo real como primario.

Si no existe ningún recurso Dispositivo para un identificador de dispositivo determinado en el servidor de FHIR, el resultado depende del valor de `Resolution Type` establecido en el momento de la creación. Cuando se establece en `Lookup`, se omite el mensaje específico y la canalización seguirá procesando otros mensajes entrantes. Si se establece en `Create`, el conector de Azure IoT para FHIR creará unos recursos Dispositivo y Paciente esenciales en el servidor de FHIR.  

## <a name="persist"></a>Persist
Una vez que el recurso FHIR Observación se genera en la fase de transformación, el recurso se guarda en Azure API for FHIR. Si el recurso FHIR es nuevo, se creará en el servidor. Si el recurso FHIR ya existe, se actualizará.

## <a name="next-steps"></a>Pasos siguientes

Haga clic en el siguiente paso para aprender a crear plantillas de asignación de dispositivos y FHIR.

>[!div class="nextstepaction"]
>[Plantillas de asignación del conector de Azure IoT para FHIR (versión preliminar)](iot-mapping-templates.md)

*En Azure Portal, el conector de Azure IoT para FHIR se conoce como conector de IoT (versión preliminar).

FHIR es la marca registrada de HL7 y se usa con su permiso.
