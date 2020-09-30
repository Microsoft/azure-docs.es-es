---
title: Entorno de desarrollo de Azure IoT Edge | Microsoft Docs
description: Obtenga información sobre los sistemas admitidos y las herramientas de desarrollo propias que le ayudarán a crear módulos IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 01/04/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: bee1f95e1f56b65912895fec8af2512c6cd7b85a
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89300381"
---
# <a name="prepare-your-development-and-test-environment-for-iot-edge"></a>Preparación del entorno de desarrollo y prueba para IoT Edge

Azure IoT Edge traslada la lógica de negocios existente a los dispositivos que funcionan en el perímetro. Con el fin de preparar las aplicaciones y cargas de trabajo para que se ejecuten como [módulos IoT Edge](iot-edge-modules.md), tendrá que compilarlas como contenedores. En este artículo se proporcionan instrucciones sobre cómo configurar el entorno de desarrollo, para poder crear correctamente una solución de IoT Edge. Una vez configurado el entorno de desarrollo, puede aprender cómo [desarrollar módulos IoT Edge propios](module-development.md).

En cualquier solución de IoT Edge, hay que tener en cuenta al menos dos equipos. Uno es el propio dispositivo (o dispositivos) de IoT Edge que ejecuta el módulo IoT Edge. El otro es el equipo de desarrollo que se usa para compilar, probar e implementar los módulos. Este artículo se centra principalmente en el equipo de desarrollo. Con fines de prueba, los dos equipos pueden ser el mismo. Puede ejecutar IoT Edge en el equipo de desarrollo e implementar módulos en él.

## <a name="operating-system"></a>Sistema operativo

Azure IoT Edge se ejecuta en un conjunto específico de [sistemas operativos compatibles](support.md). Para desarrollar para IoT Edge, puede usar la mayoría de los sistemas operativos en los que se puede ejecutar un motor de contenedor. El motor de contenedor es un requisito en el equipo de desarrollo para compilar los módulos como contenedores e insertarlos en un registro de contenedor.

Si el equipo de desarrollo no puede ejecutar Azure IoT Edge, continúe en este artículo para obtener información sobre [las herramientas de prueba](#testing-tools) que le ayudarán a realizar pruebas y depurar de forma local.

No es necesario que el sistema operativo del equipo de desarrollo coincida con el del dispositivo IoT Edge. Pero el sistema operativo del contenedor debe ser coherente entre el equipo de desarrollo y el dispositivo IoT Edge. Por ejemplo, puede desarrollar módulos en un equipo Windows e implementarlos en un dispositivo de Linux. El equipo con Windows debe ejecutar contenedores de Linux para compilar los módulos para el dispositivo de Linux.

## <a name="container-engine"></a>Motor de contenedor

El concepto central de IoT Edge es que se puede implementar de forma remota la lógica de negocios y en la nube en los dispositivos si se empaqueta en contenedores. Para crear contenedores, necesita un motor de contenedor en el equipo de desarrollo.

El único motor de contenedor compatible para los dispositivos IoT Edge en producción es Moby. Pero cualquier motor de contenedor compatible con la iniciativa Open Container, como Docker, es capaz de crear imágenes de módulos IoT Edge.

## <a name="development-tools"></a>Herramientas de desarrollo

Visual Studio y Visual Studio Code tienen extensiones de complemento para ayudar a desarrollar soluciones de IoT Edge. Estas extensiones proporcionan plantillas específicas del idioma para ayudar a crear e implementar nuevos escenarios de IoT Edge. Las extensiones de Azure IoT Edge para Visual Studio y Visual Studio Code ayudan a crear el código, compilar, implementar y depurar las soluciones de IoT Edge. Puede crear una solución completa de IoT Edge que contenga varios módulos, y las extensiones actualizan de forma automática una plantilla de manifiesto de implementación con cada nuevo módulo que se agregue. Con las extensiones, también puede administrar los dispositivos IoT desde dentro de Visual Studio o Visual Studio Code. Implemente los módulos en un dispositivo, supervise el estado y vea los mensajes cuando llegan a IoT Hub. Las dos extensiones también usan la [herramienta de desarrollo de IoT EdgeHub](#iot-edgehub-dev-tool) para habilitar la ejecución local y la depuración de los módulos en el equipo de desarrollo.

Si prefiere desarrollar con otros editores o desde la CLI, la herramienta de desarrollo de Azure IoT Edge proporciona comandos para que pueda desarrollar y probar desde la línea de comandos. Puede crear escenarios de IoT Edge, compilar imágenes de módulos, ejecutar módulos en un simulador y supervisar los mensajes enviados a IoT Hub.

### <a name="visual-studio-code-extension"></a>Extensión de Visual Studio Code

La extensión de Azure IoT Edge para Visual Studio Code proporciona plantillas de módulos IoT Edge basadas en lenguajes de programación como C, C#, Java, Node.js y Python, así como Azure Functions en C#.

Para obtener más información y descargar, consulte [Herramientas de Azure IoT para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

Además de las extensiones de IoT Edge, le resultará útil instalar extensiones adicionales para el desarrollo. Por ejemplo, puede usar [compatibilidad con Docker para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker) para administrar imágenes, contenedores y registros. Además, todos los principales lenguajes compatibles tienen extensiones para Visual Studio Code que pueden ayudar cuando desarrolle módulos.

#### <a name="prerequisites"></a>Requisitos previos

Las plantillas de módulo para algunos lenguajes y servicios tienen requisitos previos que son necesarios para crear las carpetas de proyecto en el equipo de desarrollo con Visual Studio Code.

| Plantilla de módulo | Requisito previo |
| --------------- | ------------ |
| Azure Functions | [SDK de .NET Core 2.1](https://www.microsoft.com/net/download). |
| C | [Git](https://git-scm.com/) |
| C# | [SDK de .NET Core 2.1](https://www.microsoft.com/net/download). |
| Java | <ul><li>[Java SE Development Kit 10](https://aka.ms/azure-jdks) <li> [Establecer la variable de entorno JAVA_HOME](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/) <li> [Maven](https://maven.apache.org/)</ul> |
| Node.js | <ul><li>[Node.js](https://nodejs.org/) <li> [Yeoman](https://www.npmjs.com/package/yo) <li> [Generador de módulos Node.js de Azure IoT Edge](https://www.npmjs.com/package/generator-azure-iot-edge-module)</ul> |
| Python |<ul><li> [Python](https://www.python.org/downloads/) <li> [Pip](https://pip.pypa.io/en/stable/installing/#installation) <li> [Git](https://git-scm.com/) </ul> |

### <a name="visual-studio-20172019-extension"></a>Extensión de Visual Studio 2017/2019

Las herramientas de Azure IoT Edge para Visual Studio proporcionan una plantilla de módulo IoT Edge basada en C# y C.

Para obtener más información y descargar, consulte [Herramientas de Azure IoT Edge para Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools) o [Herramientas de Azure IoT Edge para Visual Studio 2019](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools).

### <a name="iot-edge-dev-tool"></a>Herramienta de desarrollo de IoT Edge

La herramienta de desarrollo de Azure IoT Edge simplifica el desarrollo de IoT Edge con funciones de línea de comandos. Esta herramienta proporciona comandos de la CLI para desarrollar, depurar y probar módulos. La herramienta de desarrollo de IoT Edge funciona con el sistema de desarrollo, con independencia de que haya instalado manualmente las dependencias en el equipo o use el contenedor de desarrollo de IoT Edge.

Para obtener más información y para comenzar, vea la [wiki de la herramienta de desarrollo de IoT Edge](https://github.com/Azure/iotedgedev/wiki).

## <a name="testing-tools"></a>Herramientas de pruebas

Existen varias herramientas de pruebas para ayudar a simular dispositivos IoT Edge o depurar módulos de forma más eficaz. En la tabla siguiente se muestra una comparación general entre las herramientas y, después, en las secciones individuales, se describe cada herramienta de forma más específica.

El runtime de IoT Edge solo se admite para las implementaciones de producción, pero las herramientas siguientes permiten simular o crear fácilmente dispositivos IoT Edge para fines de prueba y desarrollo. Estas herramientas no son mutuamente excluyentes, pero pueden funcionar de forma conjunta para proporcionar una experiencia de desarrollo completa.

| Herramienta | También conocida como | Plataformas compatibles | Más adecuado para |
| ---- | ------------- | ------------------- | --------- |
| Herramienta de desarrollo de IoT EdgeHub  | iotedgehubdev | Windows, Linux, macOS | Simular un dispositivo para depurar módulos. |
| Contenedor de desarrollo de IoT Edge | microsoft/iotedgedev | Windows, Linux, macOS | Desarrollar sin instalar dependencias. |
| Runtime de IoT Edge en un contenedor | iotedgec | Windows, Linux, macOS, ARM | Realizar pruebas en un dispositivo que no admita el runtime. |
| Contenedor de dispositivos IoT Edge | toolboc/azure-iot-edge-device-container | Windows, Linux, macOS, ARM | Probar un escenario con muchos dispositivos IoT Edge a escala. |

### <a name="iot-edgehub-dev-tool"></a>Herramienta de desarrollo de IoT EdgeHub

La herramienta de desarrollo de Azure IoT EdgeHub proporciona una experiencia de desarrollo y depuración local. La herramienta ayuda a iniciar los módulos IoT Edge sin el runtime de IoT Edge para que pueda crear, desarrollar, probar, ejecutar y depurar módulos y soluciones de IoT Edge de forma local. Para realizar las pruebas, no es necesario insertar imágenes en un registro de contenedor e implementarlas en un dispositivo.

La herramienta de desarrollo IoT EdgeHub se ha diseñado para funcionar conjuntamente con las extensiones de Visual Studio y Visual Studio Code, así como con la herramienta de desarrollo de IoT Edge. Admite el desarrollo de bucle interno así como las pruebas de bucle externo, por lo que también se integra con las herramientas de DevOps.

Para obtener más información y para la instalación, vea [Herramienta de desarrollo de Azure IoT EdgeHub](https://pypi.org/project/iotedgehubdev/).

### <a name="iot-edge-dev-container"></a>Contenedor de desarrollo de IoT Edge

El contenedor de desarrollo de Azure IoT Edge es un contenedor de Docker que tiene todas las dependencias que necesita para el desarrollo de IoT Edge. Este contenedor facilita empezar a trabajar con cualquier lenguaje en el que quiera desarrollar, incluidos C#, Python, Node.js y Java. Todo lo que necesita instalar es un motor de contenedor, como Docker o Moby, para extraer el contenedor en el equipo de desarrollo.

Para obtener más información, vea el artículo sobre el [contenedor de desarrollo de Azure IoT Edge](https://hub.docker.com/r/microsoft/iotedgedev/).

### <a name="iot-edge-runtime-in-a-container"></a>Runtime de IoT Edge en un contenedor

El runtime de IoT Edge en un contenedor proporciona un runtime completo que toma la cadena de conexión del dispositivo como una variable de entorno. Este contenedor permite probar módulos y escenarios de IoT Edge en un sistema que es posible que no admita el entorno en tiempo de ejecución de forma nativa, como macOS. Todos los módulos que implemente se iniciarán fuera del contenedor del runtime. Si quiere que el runtime y los módulos implementados existan dentro del mismo contenedor, considere en su lugar el contenedor de dispositivos de IoT Edge.

Para obtener más información, vea [Running Azure IoT Edge in a container](https://github.com/Azure/iotedgedev/tree/master/docker/runtime) (Ejecución de Azure IoT Edge en un contenedor).

### <a name="iot-edge-device-container"></a>Contenedor de dispositivos IoT Edge

El contenedor de dispositivos de IoT Edge es un dispositivo IoT Edge completo, listo para iniciarse en cualquier equipo con un motor de contenedor. El contenedor de dispositivos incluye el runtime de IoT Edge y un motor de contenedor propio. Cada instancia del contenedor es un dispositivo IoT Edge autoaprovisionado totalmente funcional. El contenedor de dispositivos admite la depuración remota de los módulos, siempre y cuando haya una ruta de red para el módulo. El contenedor de dispositivos es adecuado para crear de forma rápida grandes cantidades de dispositivos IoT Edge para probar escenarios a escala o Azure Pipelines. También admite la implementación en Kubernetes a través de Helm.

Para obtener más información, vea el artículo sobre el [contenedor de dispositivos de Azure IoT Edge](https://github.com/toolboc/azure-iot-edge-device-container).

## <a name="devops-tools"></a>Herramientas de DevOps

Cuando esté listo para desarrollar soluciones a escala para escenarios de producción de gran tamaño, aproveche las ventajas de los principios modernos de DevOps incluida la automatización, la supervisión y los procesos de ingeniería de software simplificados. IoT Edge tiene extensiones para admitir herramientas de DevOps, incluidos Azure DevOps, Azure DevOps Projects y Jenkins. Si quiere personalizar una canalización existente o usar otra herramienta de DevOps como CircleCI o TravisCI, puede hacerlo con las características de la CLI que se incluyen en la herramienta de desarrollo de IoT Edge.

Para obtener más información, instrucciones y ejemplos, vea las páginas siguientes:

* [Integración continua e implementación continua en Azure IoT Edge](how-to-continuous-integration-continuous-deployment.md)
* [Creación de una canalización de CI/CD para IoT Edge con Azure DevOps Starter](how-to-devops-starter.md)
* [Complemento de Jenkins para Azure IoT Edge](https://plugins.jenkins.io/azure-iot-edge)
* [Repositorio de GitHub de DevOps para IoT Edge](https://github.com/toolboc/IoTEdge-DevOps)
