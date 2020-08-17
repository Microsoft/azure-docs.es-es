---
title: 'Adición de un panel a la interfaz de usuario de la solución de supervisión remota: Azure | Microsoft Docs'
description: En este artículo se muestra cómo agregar un nuevo panel a la interfaz de usuario web del acelerador de soluciones de supervisión remota.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 10/05/2018
ms.topic: conceptual
ms.custom: devx-track-javascript
ms.openlocfilehash: 207e5b8d9735004b6dc1a9662a3f66fbed8c2307
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422698"
---
# <a name="add-a-custom-panel-to-the-dashboard-in-the-remote-monitoring-solution-accelerator-web-ui"></a>Adición de un panel personalizado al panel de la interfaz de usuario web del acelerador de soluciones de supervisión remota

En este artículo se muestra cómo agregar un nuevo panel en la página de panel de la interfaz de usuario web del acelerador de soluciones de supervisión remota. El artículo describe:

- Cómo preparar un entorno de desarrollo local.
- Cómo agregar un nuevo panel a una página de panel de la interfaz de usuario web.

El panel de ejemplo de este artículo se muestra en la página de panel existente.

## <a name="prerequisites"></a>Prerequisites

Para completar los pasos de esta guía, necesita que el software siguiente esté instalado en la máquina de desarrollo local:

- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/download/)

## <a name="before-you-start"></a>Antes de comenzar

Debe completar los pasos descritos en el artículo [Adición de una página personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-page.md) antes de continuar.

## <a name="add-a-panel"></a>Agregar un panel

Para agregar un panel a la interfaz de usuario web, deberá agregar los archivos de origen que definen el panel y, a continuación, modificar la página de panel para mostrar el panel.

### <a name="add-the-new-files-that-define-the-panel"></a>Adición de los nuevos archivos que definen el panel

Para comenzar, la carpeta **src/walkthrough/components/pages/dashboard/panels/examplePanel** contiene los archivos que definen un panel, incluyendo:

**examplePanel.js**

[!code-javascript[Example panel](~/remote-monitoring-webui/src/walkthrough/components/pages/dashboard/panels/examplePanel/examplePanel.js?name=panel "Example panel")]

Copie la carpeta **src/walkthrough/components/pages/dashboard/panels/examplePanel** en la carpeta **src/components/pages/dashboard/panels**.

Agregue la siguiente exportación al archivo **src/walkthrough/components/pages/dashboard/panels/index.js**:

```js
export * from './examplePanel';
```

### <a name="add-the-panel-to-the-dashboard"></a>Adición del panel a la página de panel

Modifique el archivo **src/components/pages/dashboard/dashboard.js** para agregar el panel.

Agregue el panel de ejemplo a la lista de importaciones de paneles:

```js
import {
  OverviewPanel,
  AlertsPanel,
  TelemetryPanel,
  AnalyticsPanel,
  MapPanel,
  transformTelemetryResponse,
  chartColorObjects,
  ExamplePanel
} from './panels';
```

Agregue la siguiente definición de celda a la cuadrícula en el contenido de la página:

```js
          <Cell className="col-2">
            <ExamplePanel t={t} />
          </Cell>
```

## <a name="test-the-flyout"></a>Prueba del control flotante

Si la interfaz de usuario web no se está ejecutando localmente, ejecute el siguiente comando en la raíz de la copia local del repositorio:

```cmd/sh
npm start
```

El comando anterior ejecuta la interfaz de usuario localmente en `http://localhost:3000/dashboard`. Para ver el nuevo panel, vaya a la página **Panel**.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió sobre los recursos que tiene a su disposición para ayudarle a agregar o personalizar paneles en la interfaz de usuario web del acelerador de la solución de supervisión remota.

Para información más conceptual sobre el acelerador de soluciones de supervisión remota, vea [Arquitectura de supervisión remota](iot-accelerators-remote-monitoring-sample-walkthrough.md).
