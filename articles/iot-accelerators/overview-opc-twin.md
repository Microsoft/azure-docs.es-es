---
title: '¿Qué es OPC Twin: Azure | Microsoft Docs'
description: En este artículo se proporciona información general de OPC Twin. OPC Twin proporciona la detección, el registro y el control remoto de dispositivos industriales mediante las API REST.
author: dominicbetts
ms.author: dobett
ms.date: 11/26/2018
ms.topic: overview
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: 169a3149a2d8d02d486bc126071b17e9a4f7ed72
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167477"
---
# <a name="what-is-opc-twin"></a>¿Qué es OPC Twin?

OPC Twin consta de microservicios que usan Azure IoT Edge e IoT Hub para conectar la nube y la red de la fábrica. OPC Twin proporciona la detección, el registro y el control remoto de dispositivos industriales mediante las API REST. OPC Twin no necesita un SDK de arquitectura unificada de OPC (OPC UA), es independiente del lenguaje de programación y puede incluirse en un flujo de trabajo sin servidor. En este artículo se describen varios casos de uso de OPC Twin.

## <a name="discovery-and-control"></a>Detección y control
Puede usar OPC Twin para simplificar la detección y el registro.

### <a name="simple-discovery-and-registration"></a>Registro y detección simples
OPC Twin permite a los operadores de fábrica examinar la red de fábrica, de forma que se puedan detectar y registrar los servidores OPC UA. Como alternativa, los operadores de fábrica también pueden registrar manualmente los dispositivos OPC UA mediante una dirección URL de detección conocida. Por ejemplo, para conectarse a todos los dispositivos OPC UA después de haber instalado la puerta de enlace de IoT Edge con un módulo de OPC Twin en la planta de producción, el operador de fábrica puede desencadenar un examen de la red de forma remota y ver todos los servidores OPC UA. 

### <a name="simple-control"></a>Control sencillo
OPC Twin permite a los operadores de fábrica reaccionar a eventos y volver a configurar sus máquinas de la planta de producción desde la nube bien de forma automática o manual sobre la marcha. OPC Twin proporciona API REST para invocar servicios en el servidor OPC UA, examinar su espacio de direcciones, así como para leer o escribir variables y ejecutar métodos. Por ejemplo, una caldera usa KPI de temperatura para controlar la línea de producción. El sensor de temperatura publica el cambio en los datos mediante OPC Publisher. El operador de fábrica recibe la alerta de que la temperatura ha alcanzado el umbral. La línea de producción se enfría automáticamente mediante OPC Twin. El operador de fábrica recibe la notificación del enfriamiento.

## <a name="authentication"></a>Authentication
OPC Twin se puede usar para simplificar la autenticación y la experiencia de desarrollo.

### <a name="simple-authentication"></a>Autenticación sencilla 
OPC Twin usa autenticación basada en Azure Active Directory (AAD) y auditoría de un extremo a otro. Por ejemplo, OPC Twin permite que la aplicación se cree encima de OPC Twin para determinar lo que ha realizado un operador en una máquina. En el lado de la máquina, es mediante la auditoría de OPC UA. En la nube, es mediante el almacenamiento de un registro de auditoría de cliente inmutable y la autenticación de AAD en la API REST.

### <a name="simple-developer-experience"></a>Experiencia sencilla para el desarrollador 
OPC Twin puede usarse con aplicaciones escritas en cualquier lenguaje de programación gracias a las API REST. Como los desarrolladores integran un cliente OPC UA en una solución, no es necesario el conocimiento del SDK de OPC UA. OPC Twin puede integrarse sin problemas en arquitecturas sin estado y sin servidor. Por ejemplo, un desarrollador web de pila completa que desarrolle una aplicación para un panel de alarmas y eventos puede escribir la lógica para responder a los eventos en JavaScript o TypeScript mediante OPC Twin sin tener conocimientos de C o C# o sin la implementación de la pila completa de OPC UA. 

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido sobre OPC Twin y sus usos, este es el siguiente paso sugerido:

> [!div class="nextstepaction"]
> [¿Qué es OPC Vault?](overview-opc-vault.md)
