---
title: 'Inicio rápido: Creación de alertas personalizadas'
description: Reconozca, cree y asigne alertas de dispositivo personalizadas para el servicio de seguridad Azure Defender para IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/04/2020
ms.author: mlottner
ms.openlocfilehash: 48682e465374c1a0e1fb74fc6627016696ff6d2c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946243"
---
# <a name="quickstart-create-custom-alerts"></a>Inicio rápido: Creación de alertas personalizadas

Mediante el uso de alertas y grupos de seguridad personalizados, saque el máximo provecho de la información de seguridad de un extremo a otro y del conocimiento de las categorías de dispositivos para garantizar una mejora de la seguridad en la solución de IoT.

## <a name="why-use-custom-alerts"></a>¿Por qué se usan las alertas personalizadas?

Usted es quien mejor conoce sus dispositivos IoT.

Para los clientes que conocen a la perfección el comportamiento esperado de su dispositivo, Defender para IoT permite convertir este conocimiento en una directiva de comportamiento del dispositivo y en una alerta ante cualquier desviación con respecto al comportamiento normal y esperado.

## <a name="security-groups"></a>Grupos de seguridad

Los grupos de seguridad permiten definir grupos lógicos de dispositivos y administran su estado de seguridad de forma centralizada.

Estos grupos pueden representar los dispositivos con un hardware concreto, los dispositivos implementados en una ubicación determinada o cualquier otro grupo adecuado para sus necesidades específicas.

Los grupos de seguridad los define una propiedad de la etiqueta del dispositivo gemelo denominada **SecurityGroup**. De manera predeterminada, en IoT Hub cada solución de IoT de tiene un grupo de seguridad denominado **predeterminado**. Para cambiar el grupo de seguridad de un dispositivo, cambie el valor de la propiedad **SecurityGroup**.

Por ejemplo:

```
{
  "deviceId": "VM-Contoso12",
  "etag": "AAAAAAAAAAM=",
  "deviceEtag": "ODA1BzA5QjM2",
  "status": "enabled",
  "statusUpdateTime": "0001-01-01T00:00:00",
  "connectionState": "Disconnected",
  "lastActivityTime": "0001-01-01T00:00:00",
  "cloudToDeviceMessageCount": 0,
  "authenticationType": "sas",
  "x509Thumbprint": {
    "primaryThumbprint": null,
    "secondaryThumbprint": null
  },
  "version": 4,
  "tags": {
    "SecurityGroup": "default"
  },
```

Use los grupos de seguridad para agrupar los dispositivos en categorías lógicas. Después de crear los grupos asígnelos a las alertas personalizadas que prefiera para la solución de un extremo a otro de seguridad de IoT más eficaz.

## <a name="customize-an-alert"></a>Personalización de una alerta

1. Abra IoT Hub y seleccione **Configuración** en el menú **Seguridad**. 
1. Haga clic en **Alertas personalizadas**.
1. Elija un grupo de seguridad al que desee aplicar la personalización.
1. Haga clic en **Add a custom alert** (Agregar una alerta personalizada).
1. Seleccione una alerta personalizada en la lista desplegable.
1. Edite las propiedades necesarias y haga clic en **OK** (Aceptar).
1. Asegúrese de hacer clic en **SAVE** (Guardar). Si no se guarda la nueva alerta, la alerta se eliminará la próxima vez que se cierra IoT Hub.

## <a name="alerts-available-for-customization"></a>Alertas disponibles para la personalización

Defender para IoT ofrece un gran número de alertas que se pueden personalizar en función de las necesidades específicas de cada persona. Revise la [tabla de alertas personalizable](concept-customizable-security-alerts.md) para conocer la gravedad de las alertas, el origen de datos, la descripción y los pasos de corrección que sugerimos dar si se recibe alguna alerta.

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para aprender a implementar a un agente de seguridad...

> [!div class="nextstepaction"]
> [Implementación de un agente de seguridad](how-to-deploy-agent.md)
