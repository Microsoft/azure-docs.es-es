---
title: 'Inicio rápido: Creación de un módulo gemelo de seguridad'
description: En esta guía de inicio rápido, aprenderá a crear un módulo gemelo de Azure Security Center para IoT para su uso con Azure Security Center para IoT.
services: asc-for-iot
ms.service: asc-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: c782692e-1284-4c54-9d76-567bc13787cc
ms.subservice: asc-for-iot
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/08/2019
ms.author: mlottner
ms.openlocfilehash: 7b867f5bcbbdc5deab2d67f356df69ee870c21f3
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142254"
---
# <a name="quickstart-create-an-azureiotsecurity-module-twin"></a>Inicio rápido: Creación de un módulo gemelo de azureiotsecurity

Este inicio rápido explica cómo crear módulos gemelos de _azureiotsecurity_ individuales para los nuevos dispositivos o para crear módulos gemelos por lotes para todos los dispositivos de IoT Hub.

## <a name="understanding-azureiotsecurity-module-twins"></a>Descripción de los módulos gemelos de azureiotsecurity

En el caso de las soluciones de IoT integradas en Azure, los dispositivos gemelos desempeñan un rol clave tanto en la administración de dispositivos como en la automatización de procesos.

Azure Security Center para IoT ofrece integración completa con la plataforma de administración de dispositivos de IoT existente, lo que permite no solo administrar el estado de seguridad de los dispositivos, sino también hacer uso de las funcionalidades de control de dispositivos existentes.
Para lograr La integración de Azure Security Center para IoT, es preciso usar el mecanismo gemelo de IoT Hub.

Para más información acerca del concepto general de los módulos gemelos en Azure IoT Hub, consulte [Módulos gemelos de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-module-twins).

Azure Security Center para IoT usa el mecanismo de módulos gemelos y mantiene un módulo gemelo de seguridad denominado _azureiotsecurity_ para cada uno de los dispositivos.

Este módulo contiene toda la información apropiada para la seguridad del dispositivo en cuestión.

Para usar las características de Azure Security Center para IoT, estos módulos gemelos de seguridad deben crearse, configurarse y usarse en todos los dispositivos del servicio.

## <a name="create-azureiotsecurity-module-twin"></a>Creación de un módulo gemelo de azureiotsecurity

Los módulos gemelos _azureiotsecurity_ se pueden crear de dos maneras:

1. Mediante [script por lotes de módulos](https://aka.ms/iot-security-github-create-module): esto crea automáticamente el módulo gemelo para los nuevos dispositivos o para aquellos dispositivos sin un módulo gemelo mediante la configuración predeterminada.
1. Editando de forma manual cada módulo gemelo individualmente con configuraciones específicas para cada dispositivo.

>[!NOTE]
> El método por lotes no sobrescribirá los módulos gemelos azureiotsecurity existentes. El método por lotes SOLO crea módulos gemelos para aquellos dispositivos que no tienen ningún módulo de seguridad.

Para aprender a modificar o cambiar la configuración de un módulo gemelo existente, consulte [Configuración de agente](how-to-agent-configuration.md).

Para crear manualmente un módulo gemelo _azureiotsecurity_ para un dispositivo siga estas instrucciones:

1. En IoT Hub, busque y seleccione el dispositivo para el que desea crear el módulo gemelo de seguridad.
1. Haga clic en el dispositivo y, a continuación, en **Agregar identidad de módulo**.
1. En **Nombre de identidad de módulo**, escriba **azureiotsecurity**.

1. Haga clic en **Save**(Guardar).

## <a name="verify-creation-of-a-module-twin"></a>Comprobación de la creación de un módulo gemelo

Para comprobar si existe un módulo gemelo de seguridad en un dispositivo concreto:

1. En Azure IoT Hub, seleccione **IoT devices** (Dispositivos de IoT) en el menú **Explorers** (Exploradores).
1. Escriba el identificador del dispositivo o seleccione una opción en el **campo Query device field** (Consulta dispositivo) y haga clic en **Query devices** (Consultar dispositivos).
    ![Consultar dispositivos](./media/quickstart/verify-security-module-twin.png)
1. Seleccione el dispositivo o haga doble clic en él para abrir la página Device details (Detalles del dispositivo).
1. Seleccione el menú **Identidades de módulo** y confirme la existencia del módulo **azureiotsecurity** en la lista de identidades de módulo asociadas con el dispositivo.
    ![Módulos asociados con un dispositivo](./media/quickstart/verify-security-module-twin-3.png)

Para más información acerca de cómo personalizar las propiedades de módulos gemelos de Azure Security Center para IoT, consulte [Configuración del agente](how-to-agent-configuration.md).

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo para aprender a investigar las recomendaciones de seguridad...

> [!div class="nextstepaction"]
> [Investigación de recomendaciones de seguridad](quickstart-investigate-security-recommendations.md)
