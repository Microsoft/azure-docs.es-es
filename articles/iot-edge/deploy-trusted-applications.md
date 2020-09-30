---
title: Aplicaciones de confianza como módulos de Azure IoT Edge
description: Use la API y el SDK de Open Enclave para escribir aplicaciones de confianza e implementarlas como módulos de IoT Edge para la computación confidencial
author: kgremban
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 09/15/2020
ms.author: kgremban
ms.openlocfilehash: 3a43d7e3e406e4a248e0ac2c27fb74a2f1b036cf
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932641"
---
# <a name="confidential-computing-at-the-edge"></a>Computación confidencial en el perímetro

Azure IoT Edge admite aplicaciones de confianza que se ejecutan dentro de enclaves seguros en el dispositivo. El cifrado proporciona seguridad para los datos mientras están en tránsito o en reposo, pero los enclaves proporcionan seguridad para los datos y las cargas de trabajo mientras están en uso. IoT Edge admite Open Enclave como estándar para el desarrollo de aplicaciones de confianza.

La seguridad siempre ha sido un aspecto importante del Internet de las cosas (IoT) porque los dispositivos IoT rara vez se encuentran protegidos dentro de una instalación privada. Esta exposición de los dispositivos los pone en riesgo de sufrir alteraciones y falsificaciones, porque son físicamente accesibles para actores malintencionados. Los dispositivos IoT Edge necesitan incluso más confianza e integridad, porque permiten que las cargas de trabajo confidenciales se ejecuten en el perímetro. A diferencia de lo que ocurre con sensores y accionadores comunes, estos dispositivos de inteligencia perimetral exponen potencialmente cargas de trabajo confidenciales que anteriormente se ejecutaban solo dentro de entornos protegidos locales o de nube.

El [administrador de seguridad de IoT Edge](iot-edge-security-manager.md) aborda una parte del desafío de la computación confidencial. El administrador de seguridad usa un módulo de seguridad de hardware (HSM) para proteger las cargas de trabajo de identidad y los procesos en curso de un dispositivo IoT Edge.

Otro aspecto de la computación confidencial es proteger los datos que están en uso en el perímetro. Un *entorno de ejecución de confianza* (TEE) es un entorno aislado seguro en un procesador y a veces se le llama *enclave*. Una *aplicación de confianza* es una aplicación que se ejecuta en un enclave. Debido a la naturaleza de los enclaves, las aplicaciones de confianza están protegidas de otras aplicaciones que se ejecutan en el procesador principal o en el entorno de ejecución de confianza.

## <a name="trusted-applications-on-iot-edge"></a>Aplicaciones de confianza en IoT Edge

Las aplicaciones de confianza están cifradas en tránsito y en reposo y solo se descifran para su ejecución dentro de un entorno de ejecución de confianza. Este estándar es válido para las aplicaciones de confianza implementadas como módulos de IoT Edge.

El desarrollador crea la aplicación de confianza y la empaqueta como un módulo de IoT Edge. La aplicación se cifra antes de insertarse en el registro de contenedor. La aplicación permanece cifrada durante todo el proceso de implementación de IoT Edge hasta que el módulo se inicie en el dispositivo IoT Edge. Una vez que la aplicación de confianza se encuentra dentro del entorno de ejecución de confianza, se descifra y puede empezar a ejecutarse.

![Diagrama: Las aplicaciones de confianza se cifran dentro de los módulos de IoT Edge hasta que se implementan en el enclave seguro.](./media/deploy-trusted-applications/trusted-applications-encrypted.png)

Las aplicaciones de confianza en IoT Edge son una extensión lógica de la [computación confidencial de Azure](../confidential-computing/overview.md). Las cargas de trabajo que se ejecutan dentro de enclaves seguros en la nube también se pueden implementar para ejecutarse dentro de enclaves seguros en el perímetro.

## <a name="open-enclave"></a>Open Enclave

El [SDK de Open Enclave](https://openenclave.io/sdk/) es un proyecto de código abierto que ayuda a los desarrolladores a crear aplicaciones de confianza para varias plataformas y entornos. El SDK de Open Enclave funciona dentro del entorno de ejecución de confianza de un dispositivo, mientras que la API de Open Enclave sirve como interfaz entre el entorno TEE y el entorno de procesamiento no TEE.

Open Enclave admite varias plataformas de hardware. Actualmente, la compatibilidad de IoT Edge con los enclaves requiere el sistema operativo Open Portable TEE (SO OP-TEE). Para más información, consulte el artículo sobre el [SDK de Open Enclave para OP-TEE OS](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/OP-TEE/Introduction.md).

El repositorio de Open Enclave también incluye ejemplos que ayudan a los desarrolladores a empezar a trabajar. Para más información, elija uno de los artículos introductorios:

* [Compilación de ejemplos del SDK de Open Enclave en Linux](https://github.com/openenclave/openenclave/blob/master/samples/README_Linux.md)
* [Compilación de ejemplos del SDK de Open Enclave en Windows](https://github.com/openenclave/openenclave/blob/master/samples/README_Windows.md)

## <a name="hardware"></a>Hardware

Actualmente, [TrustBox de Scalys](https://scalys.com/trustbox-industrial/) es el único dispositivo compatible con acuerdos de servicio del fabricante para implementar aplicaciones de confianza como módulos de IoT Edge. TrustBox se basa en los dispositivos TrustBox Edge y TrustBox EdgeXL en los que están cargados previamente el SDK de Open Enclave y Azure IoT Edge.

Para más información, consulte el artículo de [introducción a Open Enclave para TrustBox de Scalys](https://aka.ms/scalys-trustbox-edge-get-started).

## <a name="develop-and-deploy"></a>Desarrollo e implementación

Cuando esté listo para desarrollar e implementar una aplicación de confianza, la extensión [Microsoft Open Enclave](https://marketplace.visualstudio.com/items?itemName=ms-iot.msiot-vscode-openenclave) para Visual Studio Code puede ser útil. Puede usar Linux o Windows como máquina de desarrollo para desarrollar módulos para TrustBox.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo empezar a desarrollar aplicaciones de confianza como módulos de IoT Edge con la [extensión de Open Enclave para Visual Studio Code](https://github.com/openenclave/openenclave/tree/master/devex/vscode-extension).
