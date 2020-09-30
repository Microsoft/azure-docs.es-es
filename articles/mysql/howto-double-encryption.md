---
title: 'Cifrado doble de infraestructura: Azure Portal: Azure Database for MySQL'
description: Obtenga información sobre cómo configurar y administrar el cifrado doble de infraestructura para Azure Database for MySQL.
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: how-to
ms.date: 06/30/2020
ms.openlocfilehash: eafad5edf9dcac5745986d09060baf7e4278762d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903976"
---
# <a name="infrastructure-double-encryption-for-azure-database-for-mysql"></a>Cifrado doble de infraestructura para Azure Database for MySQL

Obtenga información sobre cómo configurar y administrar el cifrado doble de infraestructura para Azure Database for MySQL.

## <a name="prerequisites"></a>Requisitos previos

* Debe tener una suscripción de Azure y ser un administrador en esa suscripción.

## <a name="create-an-azure-database-for-mysql-server-with-infrastructure-double-encryption---portal"></a>Creación de un servidor de Azure Database for MySQL con el cifrado doble de infraestructura: portal

Siga estos pasos para crear un servidor de Azure Database for MySQL con el cifrado doble de infraestructura desde Azure Portal:

1. En la esquina superior izquierda del portal, seleccione **Crear un recurso** (+).

2. Seleccione **Bases de datos** > **Azure Database for MySQL**. También puede escribir **MySQL** en el cuadro de búsqueda para encontrar el servicio.

   :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/2_navigate-to-mysql.png" alt-text="Opción Azure Database for MySQL":::

3. Proporcione la información básica del servidor. Seleccione **Configuración adicional** y marque la casilla **Cifrado doble de infraestructura** para establecer el parámetro.

    :::image type="content" source="./media/howto-double-encryption/infrastructure-encryption-selected.png" alt-text="Selecciones de Azure Database for MySQL":::

4. Seleccione **Revisar y crear** para realizar el aprovisionamiento del servidor.

    :::image type="content" source="./media/howto-double-encryption/infrastructure-encryption-summary.png" alt-text="Resumen de Azure Database for MySQL":::

5. Una vez creado el servidor, puede validar el cifrado doble de infraestructura comprobando el estado en la hoja del servidor **Cifrado de datos**.

    :::image type="content" source="./media/howto-double-encryption/infrastructure-encryption-validation.png" alt-text="Validación de Azure Database for MySQL":::

## <a name="create-an-azure-database-for-mysql-server-with-infrastructure-double-encryption---cli"></a>Creación de un servidor de Azure Database for MySQL con el cifrado doble de infraestructura: CLI

Siga estos pasos para crear un servidor de Azure Database for MySQL con el cifrado doble de infraestructura desde la CLI:

En este ejemplo se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```
En el ejemplo siguiente se crea un servidor MySQL 5.7 llamado `mydemoserver` en la región Oeste de EE. UU., en el grupo de recursos `myresourcegroup` y con el inicio de sesión de administrador de servidor `myadmin`. Se trata de un servidor **Gen 4** de **uso general** con **dos núcleos virtuales**. Asimismo, habilitará el cifrado doble de infraestructura para el servidor creado. Sustituya `<server_admin_password>` por su propio valor.

```azurecli-interactive
az mysql server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 5.7 --infrastructure-encryption <Enabled/Disabled>
```

## <a name="next-steps"></a>Pasos siguientes

 Para más información sobre el cifrado de datos, consulte [Cifrado doble de infraestructura de datos de Azure Database for MySQL](concepts-Infrastructure-double-encryption.md).
