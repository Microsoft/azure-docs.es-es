---
title: Recomendaciones de seguridad
description: Sepa en qué consiste el concepto de recomendaciones de seguridad y cómo se usan en Defender para IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 0eccab6c3d59ad68ddc8f96c3d84c57dc1bbeeca
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932309"
---
# <a name="security-recommendations"></a>Recomendaciones de seguridad

Defender para IoT analiza los dispositivos IoT y los recursos de Azure y hace recomendaciones de seguridad para reducir la superficie expuesta a ataques.
Las recomendaciones de seguridad requieren una acción y tienen como objetivo ayudar a los clientes a cumplir los procedimientos recomendados de seguridad existentes.

En este artículo, encontrará una lista de recomendaciones que se pueden activar en IoT Hub o en sus dispositivos IoT.

## <a name="recommendations-for-iot-devices"></a>Recomendaciones para dispositivos IoT

Las recomendaciones de dispositivo proporcionan información y sugerencias para mejorar la posición de seguridad de los dispositivos.

| severity | Nombre                                                      | Origen de datos | Descripción                                                                                                                                                                                           |
|----------|-----------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Media   | Puertos abiertos en el dispositivo                                      | Agente       | Se encontró un punto de conexión de escucha en el dispositivo.                                                                                                                                                        |
| Media   | Se encontró una directiva de firewall permisiva en una de las cadenas. | Agente       | Se encontró una directiva de firewall permitida en las cadenas de firewall principales (ENTRADA/SALIDA). La directiva debe denegar todo el tráfico de manera predeterminada y definir reglas para permitir la comunicación necesaria hacia y desde el dispositivo.                               |
| Media   | Se encontró una regla de firewall permisiva en la cadena de entrada     | Agente       | Se encontró una regla en el firewall que contiene un patrón permisivo para una amplia gama de direcciones IP o puertos.                                                                                    |
| Media   | Se encontró una regla de firewall permisiva en la cadena de salida    | Agente       | Se encontró una regla en el firewall que contiene un patrón permisivo para una amplia gama de direcciones IP o puertos.                                                                                   |
| Media   | Error de validación de línea base de sistema operativo           | Agente       | El dispositivo no cumple las [pruebas comparativas de CIS Linux](https://www.cisecurity.org/cis-benchmarks/).                                                                                                        |

### <a name="operational-recommendations-for-iot-devices"></a>Recomendaciones operativas para dispositivos IoT

Las recomendaciones operativas proporcionan información y sugerencias para mejorar la configuración de los agentes de seguridad.

| severity | Nombre                                    | Origen de datos | Descripción                                                                       |
|----------|-----------------------------------------|-------------|-----------------------------------------------------------------------------------|
| Bajo      | El agente envía mensajes sin utilizar          | Agente       | 10 % o más de los mensajes de seguridad tenían un tamaño inferior a 4 KB durante las últimas 24 horas.  |
| Bajo      | Configuración de gemelo de seguridad no óptima | Agente       | La configuración del gemelo de seguridad no es óptima.                                        |
| Bajo      | Conflicto de configuración de gemelo de seguridad    | Agente       | Se identificaron conflictos en la configuración del gemelo de seguridad. |                          |
|

## <a name="recommendations-for-iot-hub"></a>Recomendaciones para IoT Hub

Las alertas de recomendación proporcionan información y sugerencias relativas a las acciones para mejorar la posición de seguridad del entorno.

| severity | Nombre                                                     | Origen de datos | Descripción                                                                                                                                                                                                             |
|----------|----------------------------------------------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Alto     | Credenciales de autenticación idénticas utilizadas por varios dispositivos | IoT Hub     | Hay varios dispositivos que usan las mismas credenciales de autenticación de IoT Hub. Esto puede indicar que hay un dispositivo ilegítimo que suplanta un dispositivo legítimo. El uso de credenciales duplicadas aumenta el riesgo de suplantación del dispositivo a manos de un individuo malintencionado. |
| Media   | La directiva del filtro de IP predeterminada debe ser Denegar.                  | IoT Hub     | La configuración del filtro IP debe tener reglas definidas para el tráfico permitido y debe denegar el resto del tráfico de forma predeterminada.                                                                                                     |
| Media   | La regla del filtro IP incluye un amplio intervalo de IP                   | IoT Hub     | Un intervalo IP de origen de la regla de filtro IP permitido es demasiado grande. Las reglas excesivamente permisivas podrían exponer su instancia de IoT Hub a agentes malintencionados.                                                                                       |
| Bajo      | Habilitar registros de diagnóstico en IoT Hub                       | IoT Hub     | Habilite los registros y consérvelos por hasta un año. Conservar los registros le permite volver a crear seguimientos de actividad con fines de investigación cuando se produce un incidente de seguridad o se pone en peligro la red.                                       |
|

## <a name="next-steps"></a>Pasos siguientes

- [Información general](overview.md) del servicio Defender para IoT
- Aprenda a [acceder a los datos de seguridad](how-to-security-data-access.md)
- Más información sobre cómo [investigar un dispositivo](how-to-investigate-device.md)
