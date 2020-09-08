---
title: Complemento Native React para el SDK de JavaScript de Application Insights
description: Instalación y uso del complemento Native React para el SDK de JavaScript de Application Insights
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 3dac47aa7cf93ca882c4c1d0d191dabf0eb7178c
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226973"
---
# <a name="native-react-plugin-for-application-insights-javascript-sdk"></a>Complemento Native React para el SDK de JavaScript de Application Insights

El complemento Native React para el SDK de JavaScript de Application Insights recopila información del dispositivo; de forma predeterminada, este complemento recopila automáticamente:

- **Identificador de dispositivo único** (también conocido como identificador de instalación).
- **Nombre del modelo de dispositivo** (como iPhone X, Samsung Galaxy Fold, Huawei P30 Pro, etc.)
- **Tipo de dispositivo** (por ejemplo, auricular, tableta, etc.)

## <a name="requirements"></a>Requisitos

Debe usar una versión mayor o igual que 2.0.0 de `@microsoft/applicationinsights-web`. Este complemento solo funcionará en aplicaciones react-native. No funcionará con [aplicaciones que usan el marco Expo](https://docs.expo.io/), por lo que no funcionará con Create React Native App.

## <a name="getting-started"></a>Introducción

Instale y vincule el paquete [react-native-device-info](https://www.npmjs.com/package/react-native-device-info). Mantenga el paquete `react-native-device-info` actualizado para recopilar los nombres de dispositivo más recientes con la aplicación.

```zsh

npm install --save @microsoft/applicationinsights-react-native @microsoft/applicationinsights-web
npm install --save react-native-device-info
react-native link react-native-device-info

```

## <a name="initializing-the-plugin"></a>Inicialización del complemento

Para usar este complemento, debe construir el complemento y agregarlo como `extension` a la instancia de Application Insights existente.

```typescript
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ReactNativePlugin } from '@microsoft/applicationinsights-react-native';

var RNPlugin = new ReactNativePlugin();
var appInsights = new ApplicationInsights({
    config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [RNPlugin]
    }
});
appInsights.loadAppInsights();

```

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre el SDK de JavaScript, consulte la [documentación del SDK de JavaScript de Application Insights](javascript.md).
- Para obtener información sobre el lenguaje de consulta de Kusto y la consulta de datos en Log Analytics, consulte la [información general sobre la consulta de registro](../../azure-monitor/log-query/log-query-overview.md).
