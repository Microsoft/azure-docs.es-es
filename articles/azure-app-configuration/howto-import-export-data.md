---
title: Importación o exportación de datos con Azure App Configuration
description: Obtenga información sobre cómo importar o exportar datos de configuración a o desde Azure App Configuration. Intercambie datos entre el almacén de App Configuration y el proyecto de código.
services: azure-app-configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 02/25/2020
ms.author: lcozzens
ms.openlocfilehash: 21eba653bcd853db9550d0d3781aacd281884605
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588082"
---
# <a name="import-or-export-configuration-data"></a>Importación o exportación de datos de configuración

Azure App Configuration admite las operaciones de importación y exportación. Use estas operaciones para trabajar con datos de configuración de forma masiva e intercambiar datos entre el almacén de App Configuration y el proyecto de código. Por ejemplo, puede configurar un almacén de App Configuration para pruebas y otro para producción. Puede copiar la configuración de la aplicación de uno a otro para no tener que escribir los datos dos veces.

En este artículo se proporciona una guía para importar y exportar los datos con App Configuration. Si desea configurar una sincronización continua con el repositorio de GitHub, eche un vistazo a nuestra [acción de GitHub](https://aka.ms/azconfig-gha1).

## <a name="import-data"></a>Importar datos

La importación trae los datos de configuración de un almacén de App Configuration desde un origen existente. Use la función de importación para migrar los datos a un almacén de App Configuration o para agregar datos de varios orígenes. App Configuration admite la importación desde un archivo JSON, YAML o de propiedades.

Importe los datos con [Azure Portal](https://portal.azure.com) o con la [CLI de Azure](./scripts/cli-import.md). En Azure Portal, haga lo siguiente:

1. Vaya a su almacén de App Configuration y seleccione **Importar/Exportar** del menú **Operaciones**.

1. En la pestaña **Importar**, seleccione **Servicio de origen** > **Archivo de configuración**.

1. Seleccione **Para el idioma** y seleccione el tipo de entrada que desee.

1. Seleccione el icono **Carpeta** y busque el archivo para importar.

    ![Importación del archivo](./media/import-file.png)

1. Seleccione un **Separador** y, si lo desea, escriba un **Prefijo** para usarlo con los nombres de clave importados.

1. Si lo desea, seleccione una **Etiqueta**.

1. Seleccione **Aplicar** para finalizar la importación.

    ![Importación de archivo finalizada](./media/import-file-complete.png)

## <a name="export-data"></a>Exportar datos

La exportación escribe los datos de configuración almacenados en App Configuration a otro destino. Utilice la función de exportación, por ejemplo, para guardar los datos de un almacén de App Configuration en un archivo que está insertado en el código de aplicación durante la implementación.

Para exportar los datos, use [Azure Portal](https://portal.azure.com) o la [CLI de Azure](./scripts/cli-export.md). En Azure Portal, haga lo siguiente:

1. Vaya a su almacén de App Configuration y seleccione **Importar/Exportar**.

1. En la pestaña **Exportar**, seleccione **Servicio de destino** > **Archivo de configuración**.

1. Si lo desea, escriba un **Prefijo** y seleccione una **Etiqueta** y un momento dado para las claves que se exportarán.

1. Seleccione un **Tipo de archivo** > **Separador**.

1. Seleccione **Aplicar** para finalizar la exportación.

    ![Exportación de archivo finalizada](./media/export-file-complete.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cree una aplicación web ASP.NET Core](./quickstart-aspnet-core-app.md)  
