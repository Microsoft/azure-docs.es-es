---
title: 'Inicio rápido: Uso de Synapse Studio'
description: En este artículo de inicio rápido, verá y aprenderá lo fácil que es consultar diversos tipos de archivos mediante Synapse Studio.
services: synapse-analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql
ms.date: 04/15/2020
ms.author: jovanpop
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: a45c98192eccdb320814f88f7f4823603357f528
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90031762"
---
# <a name="quickstart-use-synapse-studio-preview"></a>Inicio rápido: Uso de Synapse Studio (versión preliminar)

En este artículo de inicio rápido, aprenderá a consultar archivos con Synapse Studio.

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="prerequisites"></a>Requisitos previos

[Cree un área de trabajo de Azure Synapse y una cuenta de almacenamiento asociada](quickstart-create-workspace.md).

## <a name="launch-synapse-studio"></a>Inicio de Synapse Studio

En el área de trabajo de Azure Synapse en Azure Portal, haga clic en **Iniciar Synapse Studio**.

![Inicio de Synapse Studio](./media/quickstart-synapse-studio/launch-synapse-workspace.png)

Como alternativa, puede iniciar Synapse Studio haciendo clic en [Azure Synapse Analytics](https://web.azuresynapse.net) y proporcionando los valores de inquilino, suscripción y área de trabajo adecuados.

## <a name="browse-storage-accounts"></a>Examen de las cuentas de almacenamiento

Cuando abra Synapse Studio, vaya a **Datos** y, a continuación, expanda **Cuentas de almacenamiento** para ver la cuenta de almacenamiento en el área de trabajo.

![Examen de archivos en el almacenamiento](./media/quickstart-synapse-studio/browse-files-on-storage.png)

Puede crear nuevas carpetas y cargar archivos mediante los vínculos de la barra de herramientas para organizar los archivos.

## <a name="query-files-on-storage-account"></a>Consulta de archivos en la cuenta de almacenamiento

> [!IMPORTANT]
> Debe ser miembro del rol `Storage Blob Reader` en el almacenamiento subyacente para poder realizar consultas en los archivos. Obtenga información sobre cómo [asignar los permisos de RBAC **Lector de datos de Storage Blob** o **Colaborador de datos de Storage Blob** en Azure Storage](../storage/common/storage-auth-aad-rbac-portal.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#assign-an-azure-built-in-role).

1. Cargue algunos archivos `PARQUET`.
2. Seleccione uno o varios archivos y, a continuación, cree un nuevo script SQL o un cuaderno de Spark para ver el contenido de los archivos. Si desea crear un cuaderno, debe crear el [grupo de Apache Spark en las áreas de trabajo de Synapse](quickstart-create-apache-spark-pool-studio.md).

   ![Consulta de archivos en el almacenamiento](./media/quickstart-synapse-studio/query-files-on-storage.png)

3. Ejecute la consulta o cuaderno generados para ver el contenido del archivo.

   ![Visualización del contenido del archivo](./media/quickstart-synapse-studio/query-files-on-storage-result.png)

4. Puede cambiar la consulta para filtrar y ordenar los resultados. Busque características de lenguaje que están disponibles en SQL a petición en [Introducción a las características de SQL](sql/overview-features.md).

## <a name="next-steps"></a>Pasos siguientes

- Habilite a los usuarios de Azure AD consultar archivos [mediante la asignación de los permisos de RBAC **Lector de datos de Storage Blob** o **Colaborador de datos de Storage Blob** en Azure Storage](../storage/common/storage-auth-aad-rbac-portal.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#assign-an-azure-built-in-role).
- [Consulta de archivos en Azure Storage mediante SQL a petición (versión preliminar)](sql/on-demand-workspace-overview.md)
- [Inicio rápido: Creación de un nuevo grupo de Apache Spark (versión preliminar)](quickstart-create-apache-spark-pool-portal.md)
- [Tutorial: Conexión de SQL a petición (versión preliminar) con Power BI Desktop y creación de informes](sql/tutorial-connect-power-bi-desktop.md)
