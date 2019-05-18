---
title: Tutorial para usar Azure App Configuration para administrar las marcas de características | Microsoft Docs
description: En este tutorial, obtendrá información sobre cómo administrar las marcas de características por separado desde la aplicación mediante Azure App Configuration.
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 04/19/2019
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: d995a2e9f0d05dc3b0853036e8fb3c04ccdfab96
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412294"
---
# <a name="tutorial-manage-feature-flags-in-azure-app-configuration"></a>Tutorial: Administración de las marcas de características en Azure App Configuration

Puede almacenar todas las marcas de características en Azure App Configuration y administrarlas desde un único lugar. Tiene una interfaz de usuario de portal, denominada **Administrador de características**, que se ha diseñado específicamente para las marcas de características. Además, App Configuration admite el esquema de datos de las marcas de características de .NET Core de forma nativa.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Definir y administrar las marcas de características en App Configuration.
> * Obtener acceso a las marcas de características desde la aplicación.

## <a name="create-feature-flags"></a>Creación de marcas de características

El **Administrador de características** de Azure Portal para App Configuration proporciona una interfaz de usuario para crear y administrar las marcas de características que usa en la aplicación. Siga estos pasos para agregar una nueva marca de característica.

1. Seleccione **Administrador de características** > **+ Crear** para agregar una marca de características.

    ![Lista de marcas de características](./media/azure-app-configuration-feature-flags.png)

2. Escriba un nombre de clave único para la marca de características. Necesita este nombre para hacer referencia a la marca en el código.

3. Opcionalmente, asigne a la marca de características una descripción más identificable.

4. Establezca el estado inicial de la marca de características. Normalmente suele ser *Activado* o *Desactivado*.

    ![Creación de la marca de características](./media/azure-app-configuration-feature-flag-create.png)

5. Cuando el estado es *Activado*, puede especificar cualquier condición adicional para calificarla con **Agregar filtro**. Escriba una clave de filtro integrada o personalizada y asocie los parámetros. Entre los filtros integrados están los siguientes:

    | Clave | Parámetros JSON |
    |---|---|
    | Microsoft.Percentage | {"Value": 0-100 percent} |
    | Microsoft.TimeWindow | {"Start": UTC time, "End": UTC time} |

    ![Filtro de la marca de características](./media/azure-app-configuration-feature-flag-filter.png)

## <a name="update-feature-flag-states"></a>Actualización de los estados de las marcas de características

Siga estos pasos para cambiar el valor de estado de una marca de características.

1. Seleccione **Administrador de características**.

2. Haga clic en **...**  >  **Editar** a la derecha de una marca de características que desee modificar.

3. Establezca un nuevo estado para la marca de características.

## <a name="access-feature-flags"></a>Acceso a las marcas de características

Las marcas de características creadas por el **Administrador de características** se almacenan y recuperan como pares clave-valor regulares. Se mantienen en un prefijo de espacio de nombres especial *. appconfig.featureflag*. Puede ver los pares clave-valor subyacentes mediante el **Explorador de configuración**. La aplicación puede recuperarlos mediante los proveedores de configuración de App Configuration, los SDK, las extensiones de la línea de comandos y las API REST.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a administrar las marcas de características y sus estados mediante App Configuration. Consulte los siguientes recursos para obtener más información sobre la compatibilidad de administración de características en App Configuration y ASP.NET Core.

* [Use feature flags in an ASP.NET Core app](./use-feature-flags-dotnet-core.md) (Uso de marcas de características en una aplicación ASP.NET Core)
