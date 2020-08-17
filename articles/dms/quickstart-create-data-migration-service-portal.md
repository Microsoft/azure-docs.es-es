---
title: 'Inicio rápido: Creación de una instancia mediante Azure Portal'
titleSuffix: Azure Database Migration Service
description: Utilice Azure Portal para crear una instancia de Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: quickstart
ms.date: 07/21/2020
ms.openlocfilehash: 0dd150909a56aafdd2005f2d68a12d32a1e8fae4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087788"
---
# <a name="quickstart-create-an-instance-of-the-azure-database-migration-service-by-using-the-azure-portal"></a>Inicio rápido: Creación de una instancia de Azure Database Migration Service mediante Azure Portal

En este inicio rápido, puede usar Azure Portal para crear una instancia de Azure Database Migration Service.  Después de crear la instancia, puede usarla para migrar datos de SQL Server a Azure SQL Database.

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Abra el explorador web, vaya a [Microsoft Azure Portal](https://portal.azure.com/) y, a continuación, escriba sus credenciales para iniciar sesión en el portal.

La vista predeterminada es el panel del servicio.

> [!NOTE]
> Puede crear hasta 10 instancias de DMS por suscripción. Si necesita un número mayor de instancias, cree una incidencia de soporte técnico.

## <a name="register-the-resource-provider"></a>Registrar el proveedor de recursos

Registre el proveedor de recursos Microsoft.DataMigration antes de crear su primera instancia de Database Migration Service.

1. En Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/quickstart-create-data-migration-service-portal/dms-register-provider.png)

## <a name="create-an-instance-of-the-service"></a>Creación de una instancia del servicio

1. Seleccione +**Crear un recurso** para crear una instancia de Azure Database Migration Service.

2. Busque "migration" en Marketplace, seleccione **Azure Database Migration Service** y, en la pantalla **Azure Database Migration Service**, seleccione **Crear**.

3. En la pantalla **Crear el servicio de migración**:

    - Elija un **nombre del servicio** que sea fácil de recordar y único para identificar la instancia de Azure Database Migration Service.
    - Seleccione la **suscripción** de Azure donde desea crear la instancia.
    - Seleccione un **Grupo de recursos** existente o cree uno.
    - Elija la **ubicación** más cercana a su servidor de origen o de destino.
    - Seleccione una **red virtual** existente o cree una.

        La red virtual proporciona a Azure Database Migration Service acceso a la base de datos de origen y al entorno de destino.

        Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual con Azure Portal](https://aka.ms/vnet).

    - Seleccione Básico: 1 núcleo virtual para el **plan de tarifa**.

        ![Creación del servicio de migración](media/quickstart-create-data-migration-service-portal/dms-create-service1.png)

4. Seleccione **Crear**.

    Transcurridos unos instantes, se crea la instancia de Azure Database Migration Service y estará lista para usarse. Azure Database Migration Service se muestra tal como aparece en la imagen siguiente:

    ![Servicio de migración creado](media/quickstart-create-data-migration-service-portal/dms-service-created.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede limpiar los recursos creados en esta guía de inicio rápido mediante la eliminación del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md). Para eliminar el grupo de recursos, navegue a la instancia de Azure Database Migration Service que creó. Seleccione el nombre del **grupo de recursos** y, luego, seleccione **Eliminar grupo de recursos**. Esta acción elimina todos los activos del grupo de recursos, así como el mismo grupo.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Migración de SQL Server a Azure SQL Database](tutorial-sql-server-to-azure-sql.md)
