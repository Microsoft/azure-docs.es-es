---
title: 'Personalización de la interfaz de usuario de la solución de supervisión remota: Azure | Microsoft Docs'
description: En este artículo se ofrece información sobre cómo se puede configurar el acceso al código fuente de la interfaz de usuario del acelerador de la solución de supervisión remota y realizar algunas personalizaciones.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 11/09/2018
ms.topic: conceptual
ms.custom: devx-track-javascript
ms.openlocfilehash: 412254f47a2700f77003f7db661f0ade214a1e23
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423208"
---
# <a name="customize-the-remote-monitoring-solution-accelerator"></a>Personalización del acelerador de la solución de supervisión remota

En este artículo se proporciona información sobre cómo puede acceder al código fuente y personalizar la interfaz de usuario del acelerador de la solución de supervisión remota.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prepare-a-local-development-environment-for-the-ui"></a>Preparación de un entorno de desarrollo local para la interfaz de usuario

El código de la interfaz de usuario del acelerador de la solución de supervisión remota se implementa mediante el marco de trabajo React.js. Puede encontrar el código fuente en el repositorio de GitHub [azure-iot-pcs-remote-monitoring-webui](https://github.com/Azure/azure-iot-pcs-remote-monitoring-webui).

Para realizar cambios de la interfaz de usuario, puede realizar una copia de ella localmente. Para realizar acciones, como recuperar la telemetría, la copia local se conecta a una instancia implementada de la solución.

Los siguientes pasos describen el proceso para configurar un entorno local para el desarrollo de la interfaz de usuario:

1. Implemente una instancia **básica** del acelerador de la solución con la CLI **pcs**. Tome nota del nombre de la implementación y las credenciales proporcionadas para la máquina virtual. Para más información, vea [Implementación mediante la CLI](iot-accelerators-remote-monitoring-deploy-cli.md).

1. Para habilitar el acceso SSH a la máquina virtual que hospeda los microservicios de la solución, use Azure Portal o Azure Cloud Shell. Por ejemplo:

    ```azurecli-interactive
    az network nsg rule update --name SSH --nsg-name {your solution name}-nsg --resource-group {your solution name} --access Allow
    ```

    Solo se debe habilitar el acceso de SSH durante el desarrollo y las pruebas. Si habilita SSH, [debe deshabilitarlo tan pronto como haya terminado de usarlo](../security/fundamentals/network-best-practices.md#disable-rdpssh-access-to-virtual-machines).

1. Use Azure Portal o Azure Cloud Shell para encontrar el nombre y la dirección IP pública de la máquina virtual. Por ejemplo:

    ```azurecli-interactive
    az resource list --resource-group {your solution name} -o table
    az vm list-ip-addresses --name {your vm name from previous command} --resource-group {your solution name} -o table
    ```

1. Use SSH para conectarse a la máquina virtual. Use la dirección IP del paso anterior y las credenciales proporcionadas cuando ejecutó **pcs** para implementar la solución. El comando `ssh` está disponible en Azure Cloud Shell.

1. Para permitir la conexión de la UX local, ejecute los siguientes comandos en el shell de bash en la máquina virtual:

    ```sh
    cd /app
    sudo ./start.sh --unsafe
    ```

1. Una vez completado el comando e iniciado el sitio web, puede desconectarse de la máquina virtual.

1. En la copia local del repositorio [azure-iot-pcs-remote-monitoring-webui](https://github.com/Azure/azure-iot-pcs-remote-monitoring-webui), edite el archivo **.env** para agregar la dirección URL de la solución implementada:

    ```config
    NODE_PATH = src/
    REACT_APP_BASE_SERVICE_URL=https://{your solution name}.azurewebsites.net/
    ```

1. En un símbolo del sistema, vaya a la copia local de la carpeta `azure-iot-pcs-remote-monitoring-webui`.

1. Para instalar las bibliotecas necesarias y ejecutar la interfaz de usuario localmente, ejecute los siguientes comandos:

    ```cmd/sh
    npm install
    npm start
    ```

1. El comando anterior ejecuta la interfaz de usuario localmente en http:\//localhost:3000/dashboard. Puede modificar el código mientras se ejecuta el sitio y ver su actualización de forma dinámica.

## <a name="customize-the-layout"></a>Personalización del diseño

Cada página de la solución de supervisión remota se compone de un conjunto de controles, que se conocen como *paneles* en el código fuente. La página **Panel** se compone de cinco paneles: Información general, Mapa, Alertas, Telemetría y Análisis. Puede encontrar el código fuente que define cada página y sus paneles en el repositorio de GitHub [pcs-remote-monitoring-webui](https://github.com/Azure/pcs-remote-monitoring-webui). Por ejemplo, el código que define la página **Dashboard**, su diseño y los paneles de la página se encuentra en la carpeta [src/components/pages/dashboard](https://github.com/Azure/pcs-remote-monitoring-webui/tree/master/src/components/pages/dashboard).

Dado que los paneles administran su propio diseño y tamaño, puede modificar con facilidad el diseño de una página. Realice los cambios siguientes en el elemento **PageContent** del archivo `src/components/pages/dashboard/dashboard.js`:

* Intercambie las posiciones de los paneles de mapa y telemetría.
* Cambie los anchos relativos de los paneles de mapa y análisis.

```javascript
<PageContent className="dashboard-container">
  <Grid>
    <Cell className="col-1 devices-overview-cell">
      <OverviewPanel
        activeDeviceGroup={activeDeviceGroup}
        openWarningCount={openWarningCount}
        openCriticalCount={openCriticalCount}
        onlineDeviceCount={onlineDeviceCount}
        offlineDeviceCount={offlineDeviceCount}
        isPending={analyticsIsPending || devicesIsPending}
        error={deviceGroupError || devicesError || analyticsError}
        t={t} />
    </Cell>
    <Cell className="col-6">
      <TelemetryPanel
        timeSeriesExplorerUrl={timeSeriesParamUrl}
        telemetry={telemetry}
        isPending={telemetryIsPending}
        lastRefreshed={lastRefreshed}
        error={deviceGroupError || telemetryError}
        theme={theme}
        colors={chartColorObjects}
        t={t} />
    </Cell>
    <Cell className="col-3">
      <AlertsPanel
        alerts={currentActiveAlertsWithName}
        isPending={analyticsIsPending || rulesIsPending}
        error={rulesError || analyticsError}
        t={t}
        deviceGroups={deviceGroups} />
    </Cell>
    <Cell className="col-4">
      <PanelErrorBoundary msg={t('dashboard.panels.map.runtimeError')}>
        <MapPanel
          analyticsVersion={analyticsVersion}
          azureMapsKey={azureMapsKey}
          devices={devices}
          devicesInAlert={devicesInAlert}
          mapKeyIsPending={azureMapsKeyIsPending}
          isPending={devicesIsPending || analyticsIsPending}
          error={azureMapsKeyError || devicesError || analyticsError}
          t={t} />
      </PanelErrorBoundary>
    </Cell>
    <Cell className="col-6">
      <AnalyticsPanel
        timeSeriesExplorerUrl={timeSeriesParamUrl}
        topAlerts={topAlertsWithName}
        alertsPerDeviceId={alertsPerDeviceType}
        criticalAlertsChange={criticalAlertsChange}
        isPending={analyticsIsPending || rulesIsPending || devicesIsPending}
        error={devicesError || rulesError || analyticsError}
        theme={theme}
        colors={chartColorObjects}
        t={t} />
    </Cell>
    {
      Config.showWalkthroughExamples &&
      <Cell className="col-4">
        <ExamplePanel t={t} />
      </Cell>
    }
  </Grid>
</PageContent>
```

![Cambio del diseño del panel](./media/iot-accelerators-remote-monitoring-customize/layout.png)

También puede agregar varias instancias del mismo panel o varias versiones si [duplica y personaliza un panel](#duplicate-and-customize-an-existing-control). En el ejemplo siguiente se muestra cómo agregar dos instancias del panel de telemetría. Para realizar estos cambios, edite el archivo `src/components/pages/dashboard/dashboard.js`:

```javascript
<PageContent className="dashboard-container">
  <Grid>
    <Cell className="col-1 devices-overview-cell">
      <OverviewPanel
        activeDeviceGroup={activeDeviceGroup}
        openWarningCount={openWarningCount}
        openCriticalCount={openCriticalCount}
        onlineDeviceCount={onlineDeviceCount}
        offlineDeviceCount={offlineDeviceCount}
        isPending={analyticsIsPending || devicesIsPending}
        error={deviceGroupError || devicesError || analyticsError}
        t={t} />
    </Cell>
    <Cell className="col-3">
      <TelemetryPanel
        timeSeriesExplorerUrl={timeSeriesParamUrl}
        telemetry={telemetry}
        isPending={telemetryIsPending}
        lastRefreshed={lastRefreshed}
        error={deviceGroupError || telemetryError}
        theme={theme}
        colors={chartColorObjects}
        t={t} />
    </Cell>
    <Cell className="col-3">
      <TelemetryPanel
        timeSeriesExplorerUrl={timeSeriesParamUrl}
        telemetry={telemetry}
        isPending={telemetryIsPending}
        lastRefreshed={lastRefreshed}
        error={deviceGroupError || telemetryError}
        theme={theme}
        colors={chartColorObjects}
        t={t} />
    </Cell>
    <Cell className="col-3">
      <AlertsPanel
        alerts={currentActiveAlertsWithName}
        isPending={analyticsIsPending || rulesIsPending}
        error={rulesError || analyticsError}
        t={t}
        deviceGroups={deviceGroups} />
    </Cell>
    <Cell className="col-4">
      <PanelErrorBoundary msg={t('dashboard.panels.map.runtimeError')}>
        <MapPanel
          analyticsVersion={analyticsVersion}
          azureMapsKey={azureMapsKey}
          devices={devices}
          devicesInAlert={devicesInAlert}
          mapKeyIsPending={azureMapsKeyIsPending}
          isPending={devicesIsPending || analyticsIsPending}
          error={azureMapsKeyError || devicesError || analyticsError}
          t={t} />
      </PanelErrorBoundary>
    </Cell>
    <Cell className="col-6">
      <AnalyticsPanel
        timeSeriesExplorerUrl={timeSeriesParamUrl}
        topAlerts={topAlertsWithName}
        alertsPerDeviceId={alertsPerDeviceType}
        criticalAlertsChange={criticalAlertsChange}
        isPending={analyticsIsPending || rulesIsPending || devicesIsPending}
        error={devicesError || rulesError || analyticsError}
        theme={theme}
        colors={chartColorObjects}
        t={t} />
    </Cell>
    {
      Config.showWalkthroughExamples &&
      <Cell className="col-4">
        <ExamplePanel t={t} />
      </Cell>
    }
  </Grid>
</PageContent>
```

Puede ver distintas telemetrías en cada panel:

![Varios paneles de telemetría](./media/iot-accelerators-remote-monitoring-customize/multiple-telemetry.png)

## <a name="duplicate-and-customize-an-existing-control"></a>Duplicación y personalización de un control existente

En los siguientes pasos se describe cómo duplicar un panel existente, modificarlo y, luego, usar la versión modificada. En los pasos se usa el panel de **alertas** como ejemplo:

1. En la copia local del repositorio, realice una copia de la carpeta **alerts** en la carpeta `src/components/pages/dashboard/panels`. Asigne a la nueva copia el nombre **cust_alerts**.

1. En el archivo **alertsPanel.js** de la carpeta **cust_alerts**, cambie el nombre de la clase a **CustAlertsPanel**:

    ```javascript
    export class CustAlertsPanel extends Component {
    ```

1. Agregue la siguiente línea al archivo `src/components/pages/dashboard/panels/index.js` :

    ```javascript
    export * from './cust_alerts';
    ```

1. Reemplace `alertsPanel` con `CustAlertsPanel` en el archivo `src/components/pages/dashboard/dashboard.js`:

    ```javascript
    import {
      OverviewPanel,
      CustAlertsPanel,
      TelemetryPanel,
      KpisPanel,
      MapPanel,
      transformTelemetryResponse,
      chartColors
    } from './panels';

    ...

    <Cell className="col-3">
      <CustAlertsPanel
        alerts={currentActivealertsWithName}
        isPending={kpisIsPending || rulesIsPending}
        error={rulesError || kpisError}
        t={t} />
    </Cell>
    ```

Ya ha reemplazado el panel de **alertas** original por una copia llamada **CustAlerts**. Esta copia es igual que el original. Ahora puede modificar la copia. Por ejemplo, para cambiar el orden de las columnas en el panel de **alertas**:

1. Abra el archivo `src/components/pages/dashboard/panels/cust_alerts/alertsPanel.js` .

1. Modifique las definiciones de columna tal y como se muestra en el siguiente fragmento de código:

    ```javascript
    this.columnDefs = [
      rulesColumnDefs.severity,
      {
        headerName: 'rules.grid.count',
        field: 'count'
      },
      {
        ...rulesColumnDefs.ruleName,
        minWidth: 200
      },
      rulesColumnDefs.explore
    ];
    ```

En la siguiente captura de pantalla se muestra la nueva versión del panel de **alertas**:

![Panel de alertas actualizado](./media/iot-accelerators-remote-monitoring-customize/reorder-columns.png)

## <a name="customize-the-telemetry-chart"></a>Personalización del gráfico de telemetría

Los archivos de la carpeta `src/components/pages/dashboard/panels/telemtry` definen el gráfico de telemetría de la página **Dashboard** (Panel). La interfaz de usuario recupera la telemetría del back-end de la solución del archivo `src/services/telemetryService.js`. En los pasos siguientes se muestra cómo cambiar el período de tiempo mostrado en el gráfico de telemetría de 15 a 5 minutos:

1. En el archivo `src/services/telemetryService.js`, localice la función llamada **getTelemetryByDeviceIdP15M**. Realice una copia de esta función y modifique la copia de la manera siguiente:

    ```javascript
    static getTelemetryByDeviceIdP5M(devices = []) {
      return TelemetryService.getTelemetryByMessages({
        from: 'NOW-PT5M',
        to: 'NOW',
        order: 'desc',
        devices
      });
    }
    ```

1. Para usar esta nueva función para rellenar el gráfico de telemetría, abra el archivo `src/components/pages/dashboard/dashboard.js`. Busque la línea que inicializa el flujo de telemetría y modifíquela como se indica a continuación:

    ```javascript
    const getTelemetryStream = ({ deviceIds = [] }) => TelemetryService.getTelemetryByDeviceIdP5M(deviceIds)
    ```

El gráfico de telemetría ahora muestra los cinco minutos de datos de telemetría:

![Gráfico de telemetría que muestra un día](./media/iot-accelerators-remote-monitoring-customize/telemetry-period.png)

## <a name="add-a-new-kpi"></a>Adición de un nuevo KPI

En la página **Dashboard** (Panel) se muestran los KPI en el panel **Analytics** (Análisis). Estos KPI se calculan en el archivo `src/components/pages/dashboard/dashboard.js`. Los KPI se representan mediante el archivo `src/components/pages/dashboard/panels/analytics/analyticsPanel.js`. Los siguientes pasos describen cómo calcular y representar un nuevo valor de KPI en la página **Dashboard** (Panel). El ejemplo mostrado refleja cómo agregar un nuevo cambio de porcentaje en el KPI de alertas de advertencia:

1. Abra el archivo `src/components/pages/dashboard/dashboard.js` . Modifique el objeto **initialState** para que incluya una propiedad **warningAlertsChange** de la manera siguiente:

    ```javascript
    const initialState = {
      ...

      // Analytics data
      analyticsVersion: 0,
      currentActiveAlerts: [],
      topAlerts: [],
      alertsPerDeviceId: {},
      criticalAlertsChange: 0,
      warningAlertsChange: 0,
      analyticsIsPending: true,
      analyticsError: null

      ...
    };
    ```

1. Modifique el objeto **currentAlertsStats** para que incluya **totalWarningCount** como propiedad:

    ```javascript
    return {
      openWarningCount: (acc.openWarningCount || 0) + (isWarning && isOpen ? 1 : 0),
      openCriticalCount: (acc.openCriticalCount || 0) + (isCritical && isOpen ? 1 : 0),
      totalWarningCount: (acc.totalWarningCount || 0) + (isWarning ? 1 : 0),
      totalCriticalCount: (acc.totalCriticalCount || 0) + (isCritical ? 1 : 0),
      alertsPerDeviceId: updatedAlertsPerDeviceId
    };
    ```

1. Calcule el nuevo KPI. Busque el cálculo del recuento de alertas críticas. Duplique el código y modifique la copia como sigue:

    ```javascript
    // ================== Warning Alerts Count - START
    const currentWarningAlerts = currentAlertsStats.totalWarningCount;
    const previousWarningAlerts = previousAlerts.reduce(
      (cnt, { severity }) => severity === Config.ruleSeverity.warning ? cnt + 1 : cnt,
      0
    );
    const warningAlertsChange = ((currentWarningAlerts - previousWarningAlerts) / currentWarningAlerts * 100).toFixed(2);
    // ================== Warning Alerts Count - END
    ```

1. Incluya el nuevo KPI **warningAlertsChange** en el flujo de KPI:

    ```javascript
    return ({
      analyticsIsPending: false,
      analyticsVersion: this.state.analyticsVersion + 1,

      // Analytics data
      currentActiveAlerts,
      topAlerts,
      criticalAlertsChange,
      warningAlertsChange,
      alertsPerDeviceId: currentAlertsStats.alertsPerDeviceId,

      ...
    });
    ```

1. Incluya el nuevo KPI **warningAlertsChange** en los datos de estado usados para representar la interfaz de usuario:

    ```javascript
    const {
      ...

      analyticsVersion,
      currentActiveAlerts,
      topAlerts,
      alertsPerDeviceId,
      criticalAlertsChange,
      warningAlertsChange,
      analyticsIsPending,
      analyticsError,

      ...
    } = this.state;
    ```

1. Actualice los datos pasados al panel de KPI:

    ```javascript
    <AnalyticsPanel
      timeSeriesExplorerUrl={timeSeriesParamUrl}
      topAlerts={topAlertsWithName}
      alertsPerDeviceId={alertsPerDeviceType}
      criticalAlertsChange={criticalAlertsChange}
      warningAlertsChange={warningAlertsChange}
      isPending={analyticsIsPending || rulesIsPending || devicesIsPending}
      error={devicesError || rulesError || analyticsError}
      theme={theme}
      colors={chartColorObjects}
      t={t} />
    ```

Ya ha terminado de realizar los cambios en el archivo `src/components/pages/dashboard/dashboard.js`. Los siguientes pasos describen los cambios realizados en el archivo `src/components/pages/dashboard/panels/analytics/analyticsPanel.js` para mostrar el nuevo KPI:

1. Modifique la siguiente línea de código para recuperar los nuevos valores de KPI como sigue:

    ```javascript
    const { t, isPending, criticalAlertsChange, warningAlertsChange, alertsPerDeviceId, topAlerts, timeSeriesExplorerUrl, error } = this.props;
    ```

1. Modifique el incremento para mostrar el nuevo valor de KPI como sigue:

    ```javascript
    <div className="analytics-cell">
      <div className="analytics-header">{t('dashboard.panels.analytics.criticalAlerts')}</div>
      <div className="critical-alerts">
        {
          !showOverlay &&
            <div className="analytics-percentage-container">
              <div className="analytics-value">{ !isNaN(criticalAlertsChange) ? criticalAlertsChange : 0 }</div>
              <div className="analytics-percentage-sign">%</div>
            </div>
        }
      </div>
      <div className="critical-alerts">
        {
          !showOverlay &&
            <div className="analytics-percentage-container">
              <div className="analytics-value">{ !isNaN(warningAlertsChange) ? warningAlertsChange : 0 }</div>
              <div className="analytics-percentage-sign">%</div>
            </div>
        }
      </div>
    </div>
    ```

La página **Dashboard** (Panel) ahora muestra el nuevo valor de KPI:

![KPI de advertencia](./media/iot-accelerators-remote-monitoring-customize/new-kpi.png)

## <a name="customize-the-map"></a>Personalización del mapa

Consulte la página de [personalización del mapa](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Reference-Guide#upgrade-map-key-to-see-devices-on-a-dynamic-map) en GitHub para detalles de los componentes del mapa en la solución.

<!--
### Connect an external visualization tool

See the [Connect an external visualization tool](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/) page in GitHub for details of how to connect an external visualization tool.

-->

## <a name="other-customization-options"></a>Otras opciones de personalización

Para modificar aún más la capa de presentación y visualizaciones en la solución de supervisión remota, puede editar el código. Los repositorios GitHub pertinentes son:

* [Microservicio de configuración de soluciones de Azure IoT (.NET)](https://github.com/Azure/remote-monitoring-services-dotnet/tree/master/config)
* [Microservicio de configuración de soluciones de Azure IoT (Java)](https://github.com/Azure/remote-monitoring-services-java/tree/master/config)
* [Interfaz de usuario web de supervisión remota de Azure IoT PCS](https://github.com/Azure/pcs-remote-monitoring-webui)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió sobre los recursos que tiene a su disposición para ayudarlo a personalizar la interfaz de usuario web en el acelerador de la solución de supervisión remota. Para más información acerca de la personalización de la IU, consulte los siguientes artículos:

* [Adición de una página personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-page.md)
* [Adición de un servicio personalizado a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-service.md)
* [Adición de una cuadrícula personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-grid.md)
* [Adición de un control flotante personalizado a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-flyout.md)
* [Adición de un panel personalizado al panel de la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-panel.md)

Para información más conceptual sobre el acelerador de la solución de supervisión remota, vea [Arquitectura de supervisión remota](iot-accelerators-remote-monitoring-sample-walkthrough.md).

Para más información sobre cómo personalizar los microservicios de la solución de supervisión remota, consulte [Personalización y reimplementación de un microservicio](iot-accelerators-microservices-example.md).
<!-- Next tutorials in the sequence -->
