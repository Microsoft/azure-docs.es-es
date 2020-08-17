---
title: 'Incorporación de una cuadrícula a la interfaz de usuario de la solución de supervisión remota: Azure | Microsoft Docs'
description: En este artículo se muestra cómo agregar una nueva cuadrícula en una pagina de la interfaz de usuario web del acelerador de soluciones de supervisión remota.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 10/04/2018
ms.topic: conceptual
ms.custom: devx-track-javascript
ms.openlocfilehash: 0eab7d743823109ed138ce19f35cea0ffb143422
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422732"
---
# <a name="add-a-custom-grid-to-the-remote-monitoring-solution-accelerator-web-ui"></a>Adición de una cuadrícula personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota

En este artículo se muestra cómo agregar una nueva cuadrícula en una pagina de la interfaz de usuario web del acelerador de soluciones de supervisión remota. El artículo describe:

- Cómo preparar un entorno de desarrollo local.
- Cómo agregar una nueva cuadrícula a una página de la interfaz de usuario web.

La cuadrícula de ejemplo de este artículo muestra los datos del servicio que el artículo de procedimientos [Adición de un servicio personalizado a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-service.md) muestra cómo agregar.

## <a name="prerequisites"></a>Prerequisites

Para completar los pasos de esta guía, necesita que el software siguiente esté instalado en la máquina de desarrollo local:

- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/download/)

## <a name="before-you-start"></a>Antes de comenzar

Debe completar los pasos descritos en los artículos siguientes antes de continuar:

- [Adición de una página personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-page.md).
- [Adición de un servicio personalizado a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-service.md)

## <a name="add-a-grid"></a>Agregar una cuadrícula

Para agregar una cuadrícula a la interfaz de usuario web, deberá agregar los archivos de origen que definen la cuadrícula y modificar algunos archivos existentes para hacer que la interfaz de usuario web sea consciente del nuevo componente.

### <a name="add-the-new-files-that-define-the-grid"></a>Adición de los nuevos archivos que definen la cuadrícula

Para comenzar, la carpeta **src/walkthrough/components/pages/pageWithGrid/exampleGrid** contiene los archivos que definen una cuadrícula:

**exampleGrid.js**

[!code-javascript[Example grid](~/remote-monitoring-webui/src/walkthrough/components/pages/pageWithGrid/exampleGrid/exampleGrid.js?name=grid "Example grid")]

**exampleGridConfig.js**

[!code-javascript[Example grid configuration](~/remote-monitoring-webui/src/walkthrough/components/pages/pageWithGrid/exampleGrid/exampleGridConfig.js?name=gridconfig "Example grid configuration")]

Copie la carpeta **src/walkthrough/components/pages/pageWithGrid/exampleGrid** en la carpeta **src/components/pages/example**.

### <a name="add-the-grid-to-the-page"></a>Adición de la cuadrícula a la página

Modifique el archivo **src/components/pages/example/basicPage.container.js** como se indica a continuación para importar las definiciones del servicio:

```js
import { connect } from 'react-redux';
import { translate } from 'react-i18next';

import {
  epics as exampleEpics,
  getExamples,
  getExamplesError,
  getExamplesLastUpdated,
  getExamplesPendingStatus
} from 'store/reducers/exampleReducer';
import { BasicPage } from './basicPage';

// Pass the data
const mapStateToProps = state => ({
  data: getExamples(state),
  error: getExamplesError(state),
  isPending: getExamplesPendingStatus(state),
  lastUpdated: getExamplesLastUpdated(state)
});

// Wrap the dispatch method
const mapDispatchToProps = dispatch => ({
  fetchData: () => dispatch(exampleEpics.actions.fetchExamples())
});

export const BasicPageContainer = translate()(connect(mapStateToProps, mapDispatchToProps)(BasicPage));
```

Modifique el archivo **src/components/pages/example/basicPage.js** como se indica a continuación para agregar la cuadrícula:

```js
// Copyright (c) Microsoft. All rights reserved.

import React, { Component } from 'react';

import {
  AjaxError,
  ContextMenu,
  PageContent,
  RefreshBar
} from 'components/shared';
import { ExampleGrid } from './exampleGrid';

import './basicPage.css';

export class BasicPage extends Component {
  constructor(props) {
    super(props);
    this.state = { contextBtns: null };
  }

  componentDidMount() {
    const { isPending, lastUpdated, fetchData } = this.props;
    if (!lastUpdated && !isPending) fetchData();
  }

  onGridReady = gridReadyEvent => this.gridApi = gridReadyEvent.api;

  onContextMenuChange = contextBtns => this.setState({ contextBtns });

  render() {
    const { t, data, error, isPending, lastUpdated, fetchData } = this.props;
    const gridProps = {
      onGridReady: this.onGridReady,
      rowData: isPending ? undefined : data || [],
      onContextMenuChange: this.onContextMenuChange,
      t: this.props.t
    };

    return [
      <ContextMenu key="context-menu">
        {this.state.contextBtns}
      </ContextMenu>,
      <PageContent className="basic-page-container" key="page-content">
        <RefreshBar refresh={fetchData} time={lastUpdated} isPending={isPending} t={t} />
        {!!error && <AjaxError t={t} error={error} />}
        {!error && <ExampleGrid {...gridProps} />}
      </PageContent>
    ];
  }
}
```

Modifique el archivo **src/components/pages/example/basicPage.test.js** como se indica a continuación para actualizar las pruebas:

```js
// Copyright (c) Microsoft. All rights reserved.

import React from 'react';
import { shallow } from 'enzyme';
import 'polyfills';

import { BasicPage } from './basicPage';

describe('BasicPage Component', () => {
  it('Renders without crashing', () => {

    const fakeProps = {
      data: undefined,
      error: undefined,
      isPending: false,
      lastUpdated: undefined,
      fetchData: () => { },
      t: () => { },
    };

    const wrapper = shallow(
      <BasicPage {...fakeProps} />
    );
  });
});
```

## <a name="test-the-grid"></a>Prueba de la cuadrícula

Si la interfaz de usuario web no se está ejecutando localmente, ejecute el siguiente comando en la raíz de la copia local del repositorio:

```cmd/sh
npm start
```

El comando anterior ejecuta la interfaz de usuario localmente en `http://localhost:3000/dashboard`. Vaya a la pagina de **Ejemplo** para ver que la cuadrícula muestra los datos del servicio.

## <a name="select-rows"></a>Selección de filas

Hay dos opciones para habilitar a un usuario para que seleccione las filas de la cuadrícula:

### <a name="hard-select-rows"></a>Filas de selección fija

Si un usuario necesita actuar en varias filas al mismo tiempo, use las casillas de verificación de las filas:

1. Puede habilitar la selección fija de filas mediante la adición de un elemento **checkboxColumn** al elemento **columnDefs** que se proporciona a la cuadrícula. **checkboxColumn** se define en **/src/components/shared/pcsGrid/pcsGrid.js**:

    ```js
    this.columnDefs = [
      checkboxColumn,
      exampleColumnDefs.id,
      exampleColumnDefs.description
    ];
    ```

1. Para acceder a los elementos seleccionados, debe obtener una referencia a la API interna de la cuadrícula:

    ```js
    onGridReady = gridReadyEvent => {
      this.gridApi = gridReadyEvent.api;
      // Call the onReady props if it exists
      if (isFunc(this.props.onGridReady)) {
        this.props.onGridReady(gridReadyEvent);
      }
    };
    ```

1. Puede proporcionar botones de contexto a la página cuando se realiza una selección fija de una fila de la cuadrícula:

    ```js
    this.contextBtns = [
      <Btn key="context-btn-1" svg={svgs.reconfigure} onClick={this.doSomething()}>Button 1</Btn>,
      <Btn key="context-btn-2" svg={svgs.trash} onClick={this.doSomethingElse()}>Button 2</Btn>
    ];
    ```

    ```js
    onHardSelectChange = (selectedObjs) => {
      const { onContextMenuChange, onHardSelectChange } = this.props;
      // Show the context buttons when there are rows checked.
      if (isFunc(onContextMenuChange)) {
        onContextMenuChange(selectedObjs.length > 0 ? this.contextBtns : null);
      }
      //...
    }
    ```

1. Cuando se hace clic en un botón de contexto, puede obtener los elementos de la selección fija para realizar el trabajo:

    ```js
    doSomething = () => {
      //Just for demo purposes. Don't console log in a real grid.
      console.log('Hard selected rows', this.gridApi.getSelectedRows());
    };
    ```

### <a name="soft-select-rows"></a>Filas de selección temporal

Si el usuario necesita actuar en una sola fila, puede configurar un vínculo de selección temporal para una o varias columnas del elemento **columnDefs**.

1. En el archivo **exampleGridConfig.js**, agregue **SoftSelectLinkRenderer** como el elemento **cellRendererFramework** de un elemento **columnDef**.

    ```js
    export const exampleColumnDefs = {
      id: {
        headerName: 'examples.grid.name',
        field: 'id',
        sort: 'asc',
        cellRendererFramework: SoftSelectLinkRenderer
      }
    };
    ```

1. Cuando se hace clic en un vínculo de selección temporal, se desencadena el evento **onSoftSelectChange**. Puede realizar cualquier acción que desee para esa fila, como abrir un control flotante con detalles. En este ejemplo sencillamente se escribe en la consola:

    ```js
    onSoftSelectChange = (rowId, rowData) => {
      //Note: only the Id is reliable, rowData may be out of date
      const { onSoftSelectChange } = this.props;
      if (rowId) {
        //Just for demo purposes. Don't console log a real grid.
        console.log('Soft selected', rowId);
        this.setState({ softSelectedId: rowId });
      }
      if (isFunc(onSoftSelectChange)) {
        onSoftSelectChange(rowId, rowData);
      }
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió sobre los recursos que tiene a su disposición para ayudarle a agregar o personalizar páginas en la interfaz de usuario web del acelerador de la solución de supervisión remota.

Ya ha definido una cuadrícula, el paso siguiente consiste en la [Adición de un control flotante personalizado a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-flyout.md) que se muestra en la página de ejemplo.

Para información más conceptual sobre el acelerador de soluciones de supervisión remota, vea [Arquitectura de supervisión remota](iot-accelerators-remote-monitoring-sample-walkthrough.md).
