---
title: Supervisión del estado de una aplicación de Azure IoT Central | Microsoft Docs
description: Como operador o administrador, supervise el estado general de los dispositivos conectados a su aplicación de IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 05/14/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: d246adbf8e87d5503bc19bcd497dd00370c0a867
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001928"
---
# <a name="monitor-the-overall-health-of-the-devices-connected-to-an-iot-central-application"></a>Supervisión del estado general de los dispositivos conectados a una aplicación de IoT Central

> [!NOTE]
> Las métricas solo están disponibles para las aplicaciones de IoT Central, versión 3. Para obtener información sobre cómo comprobar la versión de la aplicación, vea [Acerca de la aplicación](./howto-get-app-info.md).

*Este artículo se aplica a operadores y administradores.*

En este artículo, se explica cómo usar el conjunto de métricas que proporciona IoT Central para evaluar el estado general de los dispositivos conectados a su aplicación de IoT Central.

De forma predeterminada, las métricas están habilitadas para la aplicación de IoT Central y se accede a estas desde [Azure Portal](https://portal.azure.com/). La [plataforma de datos de Azure Monitor expone estas métricas](../../azure-monitor/platform/data-platform-metrics.md) y proporciona varias formas de interactuar con ellas. Por ejemplo, puede usar los gráficos de Azure Portal, una API de REST o consultas de PowerShell o la CLI de Azure.

### <a name="trial-applications"></a>Aplicaciones de prueba

Las aplicaciones que usan el plan de evaluación gratuita no tienen una suscripción de Azure asociada, por lo que no son compatibles con las métricas de Azure Monitor. Puede [convertir una aplicación a un plan de precios estándar](./howto-view-bill.md#move-from-free-to-standard-pricing-plan) y obtener acceso a estas métricas.

## <a name="view-metrics-in-the-azure-portal"></a>Visualización de métricas en Azure Portal

En los pasos siguientes se supone que tiene una [aplicación de IoT Central](./quick-deploy-iot-central.md) con algunos [dispositivos conectados](./tutorial-connect-device-nodejs.md).

Para ver las métricas de IoT Central en el portal:

1. Navegue hasta el recurso de la aplicación de IoT Central en el portal. De forma predeterminada, los recursos de IoT Central se encuentran en un grupo de recursos denominado **IOTC**.
1. Para crear un gráfico a partir de las métricas de la aplicación, seleccione **Métricas** en la sección **Supervisión**.

![Métricas de Azure](media/howto-monitor-application-health/metrics.png)

### <a name="azure-portal-permissions"></a>Permisos de Azure Portal

El acceso a las métricas en Azure Portal se administra mediante el [control de acceso basado en rol de Azure](../../role-based-access-control/overview.md). Use Azure Portal para agregar usuarios a la aplicación de IoT Central, el grupo de recursos o la suscripción para concederles acceso. Debe agregar un usuario en el portal, aunque ya se haya agregado a la aplicación de IoT Central. Use [roles integrados de Azure](../../role-based-access-control/built-in-roles.md) para lograr un control de acceso más detallado.

## <a name="iot-central-metrics"></a>Métricas de IoT Central

Para obtener una lista de las métricas que están disponibles actualmente en IoT Central, consulte [Métricas compatibles con Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-supported#microsoftiotcentraliotapps).

### <a name="metrics-and-invoices"></a>Métricas y facturas

Las métricas pueden diferir de los números que se muestran en la factura de Azure IoT Central. Esta situación puede producirse por diversas razones, como:

- Los [planes de precios estándar](https://azure.microsoft.com/pricing/details/iot-central/) de IoT Central incluyen dos dispositivos y distintas cuotas de mensajes de forma gratuita. Aunque los elementos gratuitos se excluyen de la facturación, todavía se cuentan en las métricas.

- IoT Central genera automáticamente un identificador de dispositivo de prueba para cada plantilla de dispositivo en la aplicación. Este identificador de dispositivo es visible en la página **Administrar el dispositivo de prueba** para una plantilla de dispositivo. Los generadores de soluciones pueden optar por [validar sus plantillas de dispositivo](./overview-iot-central.md#create-device-templates) antes de publicarlas mediante la generación del código que usa estos identificadores de dispositivo de prueba. Aunque estos dispositivos se excluyen de la facturación, todavía se cuentan en las métricas.

- Aunque las métricas pueden mostrar un subconjunto de las comunicaciones del dispositivo a la nube, todas las comunicaciones entre el dispositivo y la nube [cuentan como un mensaje para la facturación](https://azure.microsoft.com/pricing/details/iot-central/).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a usar las plantillas de aplicaciones, le sugerimos el paso siguiente para aprender a [administrar IoT Central desde Azure Portal](howto-manage-iot-central-from-portal.md).
