---
title: 'Compatibilidad con mapas de origen para aplicaciones JavaScript: Azure Monitor Application Insights'
description: Aprenda a cargar mapas de origen en su propio contenedor de blobs de la cuenta de almacenamiento mediante Application Insights.
ms.topic: conceptual
author: markwolff
ms.author: marwolff
ms.date: 06/23/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: de475cbd7cbb0d8dbcda026109b0387da14bedff
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386799"
---
# <a name="source-map-support-for-javascript-applications"></a>Compatibilidad con mapas de origen para aplicaciones JavaScript

Application Insights admite la carga de mapas de origen en su propio contenedor de blobs de la cuenta de almacenamiento.
Los mapas de origen se pueden usar para desminificar las pilas de llamadas que se encuentran en la página de detalles de la transacción completa. Cualquier excepción enviada por el [SDK de JavaScript][ApplicationInsights-JS] o el [SDK de Node.js][ApplicationInsights-Node.js] se puede desminificar con mapas de origen.

![Desminifique una pila de llamadas mediante la vinculación a una cuenta de almacenamiento](./media/source-map-support/details-unminify.gif)

## <a name="create-a-new-storage-account-and-blob-container"></a>Creación de una cuenta de almacenamiento y un contenedor de blobs

Si ya tiene una cuenta de almacenamiento o un contenedor de blobs, puede omitir este paso.

1. [Creación de una cuenta de almacenamiento nueva][create storage account]
2. [Cree un contenedor de blobs][create blob container] en la cuenta de almacenamiento. Asegúrese de establecer "Nivel de acceso público" en `Private` para asegurarse de que los mapas de origen no sean accesibles públicamente.

> [!div class="mx-imgBorder"]
>![El nivel de acceso del contenedor debe estar establecido en Privado](./media/source-map-support/container-access-level.png)

## <a name="push-your-source-maps-to-your-blob-container"></a>Inserción de mapas de origen en el contenedor de blobs

Para integrar la canalización de implementación continua con la cuenta de almacenamiento, debe configurarla para cargar automáticamente los mapas de origen en el contenedor de blobs configurado.

Los mapas de origen se pueden cargar al contenedor de Blob Storage con la misma estructura de carpetas en la que se compilaron e implementaron. Un caso de uso común es prefijar una carpeta de implementación con su versión, por ejemplo, `1.2.3/static/js/main.js`. Cuando se desminifique a través de un contenedor de blobs de Azure llamado `sourcemaps`, intentará capturar un mapa de origen ubicado en `sourcemaps/1.2.3/static/js/main.js.map`.

### <a name="upload-source-maps-via-azure-pipelines-recommended"></a>Carga de mapas de origen mediante Azure Pipelines (recomendado)

Si usa Azure Pipelines para compilar e implementar de forma continua la aplicación, agregue una tarea [Copia de archivos de Azure][azure file copy] a la canalización para cargar automáticamente los mapas de origen.

> [!div class="mx-imgBorder"]
> ![Incorporación de una tarea Copia de archivos de Azure a la canalización para cargar los mapas de origen en Azure Blob Storage](./media/source-map-support/azure-file-copy.png)

## <a name="configure-your-application-insights-resource-with-a-source-map-storage-account"></a>Configuración del recurso de Application Insights con una cuenta de almacenamiento del mapa de origen

### <a name="from-the-end-to-end-transaction-details-page"></a>Página de detalles de la transacción completa

En la pestaña de detalles de la transacción completa, puede hacer clic en *Desminificar* y se mostrará un aviso de configuración si el recurso no está configurado.

1. En el portal, consulte los detalles de una excepción que está minificada.
2. Haga clic en *Desminificar*
3. Si el recurso no se ha configurado, aparecerá un mensaje que le solicitará que lo configure.

### <a name="from-the-properties-page"></a>En la página de propiedades

Si desea configurar o cambiar la cuenta de almacenamiento o el contenedor de blobs que está vinculado a su recurso de Application Insights, puede hacerlo en la pestaña *Propiedades* del recurso de Application Insights.

1. Vaya a la pestaña *Propiedades* del recurso de Application Insights.
2. Haga clic en *Cambiar el contenedor de blobs del mapa de origen*.
3. Seleccione otro contenedor de blobs como contenedor de mapas de origen.
4. Haga clic en `Apply`.

> [!div class="mx-imgBorder"]
> ![Nueva configuración del contenedor de blobs de Azure seleccionado con la hoja Propiedades](./media/source-map-support/reconfigure.png)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="required-role-based-access-control-rbac-settings-on-your-blob-container"></a>Configuración del control de acceso basado en rol (RBAC) requerida en el contenedor de blobs

Cualquier usuario del portal que use esta característica debe estar como mínimo asignado como [Lector de datos de Storage Blob][storage blob data reader] a su contenedor de blobs. Debe asignar este rol a cualquier otro usuario que vaya a usar los mapas de origen mediante esta característica.

> [!NOTE]
> En función de cómo se haya creado el contenedor, es posible que no haya sido asignado automáticamente a usted o a su equipo.

### <a name="source-map-not-found"></a>Mapa de origen no encontrado

1. Compruebe que el mapa de origen correspondiente se haya cargado en el contenedor de blobs correcto.
2. Compruebe que el archivo de mapa de origen lleve el nombre del archivo de JavaScript coincidente, con el sufijo `.map`.
    - Por ejemplo, `/static/js/main.4e2ca5fa.chunk.js` buscará el blob llamado `main.4e2ca5fa.chunk.js.map`
3. Compruebe la consola del explorador para ver si se registra algún error. Inclúyalo en una incidencia de soporte técnico.

## <a name="next-steps"></a>Pasos siguientes

* [Tarea Copia de archivos de Azure](/azure/devops/pipelines/tasks/deploy/azure-file-copy?view=azure-devops)


<!-- Remote URLs -->
[create storage account]: ../../storage/common/storage-account-create.md?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal
[create blob container]: ../../storage/blobs/storage-quickstart-blobs-portal.md
[storage blob data reader]: ../../role-based-access-control/built-in-roles.md#storage-blob-data-reader
[ApplicationInsights-JS]: https://github.com/microsoft/applicationinsights-js
[ApplicationInsights-Node.js]: https://github.com/microsoft/applicationinsights-node.js
[azure file copy]: https://aka.ms/azurefilecopyreadme
