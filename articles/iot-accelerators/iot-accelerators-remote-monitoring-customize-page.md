---
title: 'Adición de una página a la interfaz de usuario de la solución de supervisión remota: Azure | Microsoft Docs'
description: En este artículo se muestra cómo agregar una nueva página a la interfaz de usuario web del acelerador de soluciones de supervisión remota.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 10/02/2018
ms.topic: conceptual
ms.custom: devx-track-javascript
ms.openlocfilehash: 8101c01762749c2ceb085516936fa0c421fdb5f3
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422715"
---
# <a name="add-a-custom-page-to-the-remote-monitoring-solution-accelerator-web-ui"></a>Adición de una página personalizada a la interfaz de usuario web del acelerador de soluciones de supervisión remota

En este artículo se muestra cómo agregar una nueva página a la interfaz de usuario web del acelerador de soluciones de supervisión remota. El artículo describe:

- Cómo preparar un entorno de desarrollo local.
- Cómo agregar una nueva página a la interfaz de usuario web.

Otras guías paso a paso amplían este escenario para añadir más características a la página que se agrega.

## <a name="prerequisites"></a>Prerequisites

Para completar los pasos de esta guía, necesita que el software siguiente esté instalado en la máquina de desarrollo local:

- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/download/)

## <a name="prepare-a-local-development-environment-for-the-ui"></a>Preparación de un entorno de desarrollo local para la interfaz de usuario

El código de la interfaz de usuario del acelerador de soluciones de supervisión remota se implementa mediante el marco de trabajo de JavaScript [React](https://reactjs.org/). Puede encontrar el código fuente en el repositorio de GitHub [WebUI de supervisión remota](https://github.com/Azure/pcs-remote-monitoring-webui).

Para realizar cambios en la interfaz de usuario y probarlos, puede ejecutarla en el equipo de desarrollo local. Opcionalmente, la copia local puede conectarse a una instancia implementada del acelerador de soluciones para poder interactuar con dispositivos reales o simulados.

Para preparar el entorno de desarrollo local, use Git para clonar el repositorio [WebUI de supervisión remota](https://github.com/Azure/pcs-remote-monitoring-webui) en el equipo local:

```cmd/sh
git clone https://github.com/Azure/pcs-remote-monitoring-webui.git
```

## <a name="add-a-page"></a>Agregar una página

Para agregar una página a la interfaz de usuario web, deberá agregar los archivos de origen que definen la página y modificar algunos archivos existentes para hacer que la interfaz de usuario web sea consciente de la nueva página.

### <a name="add-the-new-files-that-define-the-page"></a>Adición de nuevos archivos que definen la página

Para comenzar, la carpeta **src/walkthrough/components/pages/basicPage** contiene cuatro archivos que definen una página sencilla:

**basicPage.container.js**

[!code-javascript[Page container source](~/remote-monitoring-webui/src/walkthrough/components/pages/basicPage/basicPage.container.js?name=container "Page container source")]

**basicPage.js**

[!code-javascript[Basic page](~/remote-monitoring-webui/src/walkthrough/components/pages/basicPage/basicPage.js?name=page "Basic page")]

**basicPage.scss**

[!code-javascript[Page styling](~/remote-monitoring-webui/src/walkthrough/components/pages/basicPage/basicPage.scss?name=styles "Page styling")]

**basicPage.test.js**

[!code-javascript[Test code for basic page](~/remote-monitoring-webui/src/walkthrough/components/pages/basicPage/basicPage.test.js?name=test "Test code for basic page")]

Cree una nueva carpeta **src/components/pages/example** y copie estos cuatro archivos en ella.

### <a name="add-the-new-page-to-the-web-ui"></a>Adición de una nueva página a la interfaz de usuario web

Para agregar la nueva página a la interfaz de usuario web, realice los siguientes cambios a los archivos existentes:

1. Agregue el contenedor de la nueva página al archivo **src/components/pages/index.js**:

    ```js
    export * from './example/basicPage.container';
    ```

1. (Opcional) Agregue un icono SVG para la nueva página. Para obtener más información, consulte [webui/src/utilities/README.md](https://github.com/Azure/pcs-remote-monitoring-webui/blob/master/src/utilities/README.md). Puede usar un archivo SVG existente.

1. Agregue el nombre de página al archivo de traducciones **public/locales/en/translations.json**. La interfaz de usuario web usa [i18next](https://www.i18next.com/) para la internacionalización.

    ```json
    "tabs": {
      "basicPage": "Example",
    },
    ```

1. Abra el archivo **src/components/app.js** que define la página de aplicación de nivel superior. Agregue la nueva página a la lista de importaciones:

    ```javascript
    // Page Components
    import  {
      //...
      BasicPageContainer
    } from './pages';
    ```

1. En el mismo archivo, agregue la nueva página a la matriz `pagesConfig`. Establezca la dirección `to` para la ruta, haga referencia al icono SVG y a las traducciones que se agregaron anteriormente y establezca `component` en el contenedor de la página:

    ```js
    const pagesConfig = [
      //...
      {
        to: '/basicpage',
        exact: true,
        svg: svgs.tabs.example,
        labelId: 'tabs.basicPage',
        component: BasicPageContainer
      },
      //...
    ];
    ```

1. Agregue rutas de navegación nuevas para la matriz `crumbsConfig`:

    ```js
    const crumbsConfig = [
      //...
      {
        path: '/basicpage', crumbs: [
          { to: '/basicpage', labelId: 'tabs.basicPage' }
        ]
      },
      //...
    ];
    ```

    Esta página de ejemplo solo tiene una ruta de navegación, pero algunas páginas pueden tener más.

Guarde todos los cambios. Ya está todo listo para ejecutar la interfaz de usuario web con la nueva página agregada.

### <a name="test-the-new-page"></a>Probar la nueva página

En un símbolo del sistema, navegue a la raíz de la copia local del repositorio y, después, ejecute los siguientes comandos para instalar las bibliotecas necesarias y ejecute la interfaz de usuario web localmente:

```cmd/sh
npm install
npm start
```

El comando anterior ejecuta la interfaz de usuario localmente en `http://localhost:3000/dashboard`.

Sin conectar su instancia local de la interfaz de usuario web a una instancia implementada del acelerador de soluciones, verá errores en el panel. Estos errores no afectan a su capacidad para probar la nueva página.

Puede modificar el código mientras el sitio se ejecuta localmente y ve como la interfaz de usuario web se actualiza dinámicamente.

## <a name="optional-connect-to-deployed-instance"></a>[Opcional] Conectarse a la instancia implementada

Si lo desea, puede conectar su copia local de ejecución de la interfaz de usuario web al acelerador de soluciones de supervisión remota en la nube:

1. Implemente una instancia **básica** del acelerador de la solución con la CLI **pcs**. Tome nota del nombre de la implementación y las credenciales proporcionadas para la máquina virtual. Para más información, vea [Implementación mediante la CLI](iot-accelerators-remote-monitoring-deploy-cli.md).

1. Use Azure Portal o la [CLI az](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) para habilitar el acceso SSH a la máquina virtual que hospeda los microservicios en la solución. Por ejemplo:

    ```azurecli
    az network nsg rule update --name SSH --nsg-name {your solution name}-nsg --resource-group {your solution name} --access Allow
    ```

    Sólo se debe habilitar el acceso SSH durante el desarrollo y pruebas. Si habilita SSH, [debe deshabilitarlo nuevamente tan pronto como sea posible](../security/fundamentals/network-best-practices.md).

1. Use Azure Portal o la [CLI az](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) para encontrar el nombre y la dirección IP pública de la máquina virtual. Por ejemplo:

    ```azurecli
    az resource list --resource-group {your solution name} -o table
    az vm list-ip-addresses --name {your vm name from previous command} --resource-group {your solution name} -o table
    ```

1. Use SSH para conectarse a la máquina virtual con la dirección IP del paso anterior y las credenciales proporcionadas cuando ejecutó **pcs** para implementar la solución.

1. Para permitir la conexión de la UX local, ejecute los siguientes comandos en el shell de bash en la máquina virtual:

    ```sh
    cd /app
    sudo ./start.sh --unsafe
    ```

1. Una vez completado el comando e iniciado el sitio web, puede desconectarse de la máquina virtual.

1. En la copia local del repositorio [WebUI de supervisión remota](https://github.com/Azure/pcs-remote-monitoring-webui), edite el archivo **.env** para agregar la dirección URL de la solución implementada:

    ```config
    NODE_PATH = src/
    REACT_APP_BASE_SERVICE_URL=https://{your solution name}.azurewebsites.net/
    ```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió sobre los recursos que tiene a su disposición para ayudarlo a personalizar la interfaz de usuario web en el acelerador de la solución de supervisión remota.

Ya ha definido una página, el paso siguiente consiste en [agregar un servicio personalizado a la interfaz de usuario web del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-customize-service.md) que recupera los datos que se muestran en la interfaz de usuario.

Para información más conceptual sobre el acelerador de soluciones de supervisión remota, vea [Arquitectura de supervisión remota](iot-accelerators-remote-monitoring-sample-walkthrough.md).
