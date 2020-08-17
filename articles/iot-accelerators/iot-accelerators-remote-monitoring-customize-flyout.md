---
title: 'Adición de un control flotante a la interfaz de usuario de la solución de supervisión remota: Azure | Microsoft Docs'
description: En este artículo se muestra cómo agregar un nuevo control flotante en una pagina de la interfaz de usuario web del acelerador de soluciones de supervisión remota.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 10/05/2018
ms.topic: conceptual
ms.custom: devx-track-javascript
ms.openlocfilehash: f10be59501368012b0e4269f402fe258da552e25
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422749"
---
# <a name="add-a-custom-flyout-to-the-remote-monitoring-solution-accelerator-web-ui"></a>Adición de un control flotante personalizado a la interfaz de usuario web del acelerador de soluciones de supervisión remota

En este artículo se muestra cómo agregar un nuevo control flotante en una pagina de la interfaz de usuario web del acelerador de soluciones de supervisión remota. El artículo describe:

- Cómo preparar un entorno de desarrollo local.
- Cómo agregar un nuevo control flotante a una página de la interfaz de usuario web.

El control flotante de ejemplo de este artículo se muestra en la página con la cuadrícula que el artículo de procedimientos [Adición de una cuadrícula personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-grid.md) muestra cómo agregar.

## <a name="prerequisites"></a>Prerequisites

Para completar los pasos de esta guía, necesita que el software siguiente esté instalado en la máquina de desarrollo local:

- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/download/)

## <a name="before-you-start"></a>Antes de comenzar

Debe completar los pasos descritos en los artículos siguientes antes de continuar:

- [Adición de una página personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-page.md).
- [Adición de un servicio personalizado a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-service.md)
- [Adición de una cuadrícula personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-grid.md)

## <a name="add-a-flyout"></a>Agregar un control flotante

Para agregar un control flotante a la interfaz de usuario web, deberá agregar los archivos de origen que definen el control flotante y modificar algunos archivos existentes para hacer que la interfaz de usuario web sea consciente del nuevo componente.

### <a name="add-the-new-files-that-define-the-flyout"></a>Adición de los nuevos archivos que definen el control flotante

Para comenzar, la carpeta **src/walkthrough/components/pages/pageWithFlyout/flyouts/exampleFlyout** contiene los archivos que definen un control flotante:

**exampleFlyout.container.js**

[!code-javascript[Example flyout container](~/remote-monitoring-webui/src/walkthrough/components/pages/pageWithFlyout/flyouts/exampleFlyout/exampleFlyout.container.js?name=flyoutcontainer "Example flyout container")]

**exampleFlyout.js**

[!code-javascript[Example flyout](~/remote-monitoring-webui/src/walkthrough/components/pages/pageWithFlyout/flyouts/exampleFlyout/exampleFlyout.js?name=flyout "Example flyout")]

Copie la carpeta **src/walkthrough/components/pages/pageWithFlyout/flyouts** a la carpeta **src/components/pages/example**.

### <a name="add-the-flyout-to-the-page"></a>Incorporación del control flotante a la página

Modifique el archivo **src/components/pages/example/basicPage.js** para agregar el control flotante.

Agregue **Btn** a las importaciones desde **components/shared** y agregue las importaciones para **svgs** y **ExampleFlyoutContainer**:

```js
import {
  AjaxError,
  ContextMenu,
  PageContent,
  RefreshBar,
  Btn
} from 'components/shared';
import { ExampleGrid } from './exampleGrid';
import { svgs } from 'utilities';
import { ExampleFlyoutContainer } from './flyouts/exampleFlyout';
```

Agregue una definición de **const** para **closedFlyoutState** y agréguelo al estado en el constructor:

```js
const closedFlyoutState = { openFlyoutName: undefined };

export class BasicPage extends Component {
  constructor(props) {
    super(props);
    this.state = { contextBtns: null, closedFlyoutState };
  }
```

Agregue las siguientes funciones a la clase **BasicPage**:

```js
  closeFlyout = () => this.setState(closedFlyoutState);

  openFlyout = (name) => () => this.setState({ openFlyoutName: name });
```

Agregue las siguientes definiciones **const** a la función **render**:

```js
    const { openFlyoutName } = this.state;

    const isExampleFlyoutOpen = openFlyoutName === 'example';
```

Agregue un botón para abrir el control flotante para el menú contextual:

```js
      <ContextMenu key="context-menu">
        {this.state.contextBtns}
        <Btn svg={svgs.reconfigure} onClick={this.openFlyout('example')}>{t('walkthrough.pageWithFlyout.open')}</Btn>
      </ContextMenu>,
```

Agregue texto y el contenedor del control flotante para el contenido de la página:

```js
      <PageContent className="basic-page-container" key="page-content">
        {t('walkthrough.pageWithFlyout.pageBody')}
        { isExampleFlyoutOpen && <ExampleFlyoutContainer onClose={this.closeFlyout} /> }
        <RefreshBar refresh={fetchData} time={lastUpdated} isPending={isPending} t={t} />
        {!!error && <AjaxError t={t} error={error} />}
        {!error && <ExampleGrid {...gridProps} />}
      </PageContent>
```

## <a name="test-the-flyout"></a>Prueba del control flotante

Si la interfaz de usuario web no se está ejecutando localmente, ejecute el siguiente comando en la raíz de la copia local del repositorio:

```cmd/sh
npm start
```

El comando anterior ejecuta la interfaz de usuario localmente en `http://localhost:3000/dashboard`. Vaya a la página **ejemplo** y haga clic en **Open Flyout** (Abrir control flotante).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió sobre los recursos que tiene a su disposición para ayudarle a agregar o personalizar páginas en la interfaz de usuario web del acelerador de la solución de supervisión remota.

Ya ha definido un control flotante en una página, el paso siguiente consiste en [agregar un panel al panel personalizado en la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-panel.md).

Para información más conceptual sobre el acelerador de soluciones de supervisión remota, vea [Arquitectura de supervisión remota](iot-accelerators-remote-monitoring-sample-walkthrough.md).
