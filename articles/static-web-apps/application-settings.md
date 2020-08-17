---
title: Configuración de las opciones de la aplicación para Azure Static Web Apps
description: Obtenga información sobre cómo configurar las opciones de la aplicación para Azure Static Web Apps
services: static-web-apps
author: burkeholland
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/08/2020
ms.author: buhollan
ms.custom: devx-track-javascript
ms.openlocfilehash: 33a40e7a0e7e3be4433b3cbd4ba7a8e09a84e689
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87428055"
---
# <a name="configure-application-settings-for-azure-static-web-apps-preview"></a>Configuración de las opciones de la aplicación para la versión preliminar de Azure Static Web Apps

La configuración de la aplicación almacena las opciones de configuración para los valores que pueden cambiar, como las cadenas de conexión de base de datos. Al agregar la configuración de la aplicación, puede modificar las entradas de la configuración en la aplicación sin necesidad de cambiar el código de la aplicación.

Las opciones de configuración de la aplicación:

- Se cifran en reposo
- Se copian a los entornos de [ensayo](review-publish-pull-requests.md) y producción

A las opciones de configuración de la aplicación también se les conoce a veces como variables de entorno.

> [!IMPORTANT]
> La configuración de la aplicación descrita en este artículo solo se aplica a la API de back-end de una aplicación de Azure Static Web App.
>
> Para obtener información sobre el uso de variables de entorno con la aplicación web de front-end, consulte la documentación de su [marco de JavaScript](#javascript-frameworks-and-libraries) o [generador de sitios estáticos](#static-site-generators).

## <a name="prerequisites"></a>Requisitos previos

- Una aplicación de Azure Static Web Apps
- [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest)

## <a name="types-of-application-settings"></a>Tipos de configuraciones de la aplicación

Normalmente hay dos aspectos en una aplicación de Azure Static Web Apps. El primero es la aplicación web (o contenido estático), que se representa mediante HTML, CSS, JavaScript e imágenes. El segundo es la API de back-end, que está basada en una aplicación de Azure Functions.

En este artículo se muestra cómo administrar la configuración de la aplicación para la API de back-end en Azure Functions.

No puede usar ni hacer referencia a la configuración de la aplicación que se describe en este artículo en ninguna aplicación web estática. Sin embargo, muchos marcos front-end y generadores de sitios estáticos permiten usar variables de entorno durante el desarrollo. En el momento de la compilación, estas variables se reemplazan por los valores correspondientes en el código HTML o JavaScript generado. Dado que los visitantes del sitio pueden reconocer con facilidad los datos de HTML y JavaScript, evite incluir información confidencial en la aplicación front-end. Es preferible colocar las opciones de configuración que contienen datos confidenciales en la parte de la API de la aplicación.

Para obtener información sobre cómo usar las variables de entorno con la biblioteca o el marco de trabajo de JavaScript, consulte los siguientes artículos.

### <a name="javascript-frameworks-and-libraries"></a>Bibliotecas y marcos de JavaScript

- [Angular](https://angular.io/guide/build#configuring-application-environments)
- [React](https://create-react-app.dev/docs/adding-custom-environment-variables/)
- [Svelte](https://linguinecode.com/post/how-to-add-environment-variables-to-your-svelte-js-app)
- [Vue](https://cli.vuejs.org/guide/mode-and-env.html)

### <a name="static-site-generators"></a>Generadores de sitios estáticos

- [Gatsby](https://www.gatsbyjs.org/docs/environment-variables/)
- [Hugo](https://gohugo.io/getting-started/configuration/)
- [Jekyll](https://jekyllrb.com/docs/configuration/environments/)

## <a name="about-api-app-settings"></a>Acerca de la configuración de la aplicación de API

Las API de Azure Static Web Apps están basadas en Azure Functions, lo que permite definir la configuración de la aplicación en el archivo _local.settings.json_. Este archivo define la configuración de la aplicación en la propiedad `Values` de la configuración.

En el ejemplo siguiente, _local.settings.json_ muestra cómo agregar un valor para `DATABASE_CONNECTION_STRING`.

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "DATABASE_CONNECTION_STRING": "<YOUR_DATABASE_CONNECTION_STRING>"
  }
}
```

Se puede hacer referencia a las opciones de configuración que se hayan definido en la propiedad `Values` desde el código mediante variables de entorno, que están disponibles en el objeto `process.env`.

```js
const connectionString = process.env.DATABASE_CONNECTION_STRING;
```

El repositorio de GitHub no realiza el seguimiento del archivo `local.settings.json` porque la información confidencial, como las cadenas de conexión de base de datos, a menudo se incluye en dicho archivo. Dado que la configuración local se conserva en el equipo, debe cargar la configuración en Azure de forma manual.

Por lo general, la carga de la configuración se realiza con poca frecuencia y no es necesaria en cada compilación.

## <a name="uploading-application-settings"></a>Carga de la configuración de la aplicación

Puede configurar las opciones de la aplicación mediante Azure Portal o con la CLI de Azure.

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

Azure Portal proporciona una interfaz para crear, actualizar y eliminar la configuración de la aplicación.

1. Acceda a [Azure Portal](https://portal.azure.com).

1. En la barra de búsqueda, busque y seleccione **Static Web Apps**.

1. Haga clic en la opción **Configuración** de la barra lateral.

1. Seleccione el entorno al que quiere aplicar la configuración de la aplicación. Los entornos de ensayo se crean automáticamente cuando se genera una solicitud de incorporación de cambios y se promueven a producción cuando la solicitud de incorporación de cambios se fusiona mediante combinación. Puede establecer una configuración de la aplicación por cada entorno.

1. Haga clic en el botón **Agregar** para agregar una nueva configuración de la aplicación.

    :::image type="content" source="media/application-settings/configuration.png" alt-text="Vista de configuración de Azure Static Web Apps":::

1. Proporcione un **nombre** y un **valor**.

1. Haga clic en **OK**.

1. Haga clic en **Save**(Guardar).

### <a name="using-the-azure-cli"></a>Uso de la CLI de Azure

Puede usar el comando `az rest` para realizar cargas masivas de opciones de configuración en Azure. El comando acepta las opciones de configuración de la aplicación como objetos JSON en una propiedad primaria denominada `properties`.

La forma más fácil de crear un archivo JSON con los valores adecuados consiste en crear una versión modificada del archivo _local.settings.json_.

1. Para asegurarse de que el nuevo archivo con datos confidenciales no se exponga públicamente, agregue la entrada siguiente al archivo _.gitignore_.

   ```bash
   local.settings*.json
   ```

2. Después, realice una copia del archivo _local.settings.json_ y asígnele el nombre _local.settings.properties.json_.

3. Dentro del nuevo archivo, quite todos los demás datos, excepto la configuración de la aplicación, y cambie el nombre de `Values` por `properties`.

   Ahora el archivo debería ser similar al del siguiente ejemplo:

   ```json
   {
     "properties": {
       "DATABASE_CONNECTION_STRING": "<YOUR_DATABASE_CONNECTION_STRING>"
     }
   }
   ```

El comando de la CLI de Azure requiere una serie de valores específicos de la cuenta para ejecutar la carga. En la ventana _Información general_ del recurso de Static Web Apps, puede acceder a la siguiente información:

1. Nombre del sitio estático
2. Definición de un nombre de grupo de recursos
3. Id. de suscripción

:::image type="content" source="media/application-settings/overview.png" alt-text="Información general de Azure Static Web Apps":::

4. Desde un terminal o línea de comandos, ejecute el siguiente comando. Asegúrese de reemplazar los marcadores de posición `<YOUR_STATIC_SITE_NAME>`, `<YOUR_RESOURCE_GROUP_NAME>` y `<YOUR_SUBSCRIPTION_ID>` por los valores de la ventana _Información general_.

   ```bash
   az rest --method put --headers "Content-Type=application/json" --uri "/subscriptions/<YOUR_SUBSCRIPTION_ID>/resourceGroups/<YOUR_RESOURCE_GROUP_NAME>/providers/Microsoft.Web/staticSites/<YOUR_STATIC_SITE_NAME>/config/functionappsettings?api-version=2019-12-01-preview" --body @local.settings.properties.json
   ```

> [!IMPORTANT]
> El archivo "local.settings.properties.json" debe estar en el mismo directorio en el que se ejecuta este comando. Este archivo puede tener cualquier nombre que quiera, ya que no es importante.

### <a name="view-application-settings-with-the-azure-cli"></a>Visualización de la configuración de la aplicación con la CLI de Azure

La configuración de la aplicación está disponible para visualizarse a través de la CLI de Azure.

- Desde un terminal o línea de comandos, ejecute el siguiente comando. Asegúrese de reemplazar los marcadores de posición `<YOUR_SUBSCRIPTION_ID>`, `<YOUR_RESOURCE_GROUP_NAME>` y `<YOUR_STATIC_SITE_NAME>` por sus valores.

   ```bash
   az rest --method post --uri "/subscriptions/<YOUR_SUBSCRIPTION_ID>/resourceGroups/<YOUR_RESOURCE_GROUP_NAME>/providers/Microsoft.Web/staticSites/<YOUR_STATIC_SITE_NAME>/listFunctionAppSettings?api-version=2019-12-01-preview"
   ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración del desarrollo local](local-development.md)
