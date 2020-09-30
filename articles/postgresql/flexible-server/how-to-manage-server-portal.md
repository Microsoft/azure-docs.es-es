---
title: 'Administración de un servidor mediante Azure Portal en Azure Database for PostgreSQL: Servidor flexible'
description: Obtenga información sobre cómo administrar un Servidor flexible de Azure Database for PostgreSQL desde Azure Portal.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 13d78110f50d6ce72b8525914ed0e91dfcadd2cc
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932105"
---
# <a name="manage-an-azure-database-for-postgresql---flexible-server-using-the-azure-portal"></a>Administración de un Servidor flexible de Azure Database for PostgreSQL mediante Azure Portal

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible está en versión preliminar.

En este artículo se explica cómo administrar Azure Database for PostgreSQL: Servidor flexible. Entre las tareas de administración se incluyen el escalado de proceso y almacenamiento, el restablecimiento de contraseñas de administración y la visualización de detalles del servidor.

## <a name="sign-in"></a>Iniciar sesión

Inicie sesión en [Azure Portal](https://portal.azure.com). Vaya al recurso de servidor flexible en Azure Portal.

## <a name="scale-compute-and-storage"></a>Escalado de proceso y almacenamiento

Tras crear el servidor, puede escalar entre los distintos [planes de tarifa](https://azure.microsoft.com/pricing/details/postgresql/) a medida que cambien las necesidades. También puede escalar o reducir verticalmente el proceso y la memoria mediante el aumento o la reducción de núcleos virtuales.

> [!NOTE]
> El almacenamiento no se puede reducir verticalmente a un valor más bajo.

1. Seleccione el servidor en Azure Portal. Seleccione **Compute + Storage** (Proceso y almacenamiento) en la sección **Configuración**.
2. Puede cambiar el **nivel de proceso**, el **núcleo virtual** y el **almacenamiento** para escalar verticalmente el servidor con un nivel de proceso superior o escalar verticalmente dentro del mismo nivel al aumentar el almacenamiento o los núcleos virtuales al valor que quiera.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/howto-manage-server-portal/scale-server.png" alt-text="Escalado del almacenamiento del servidor flexible":::

> [!Important]
> - El almacenamiento no se puede reducir verticalmente.
> - El escalado de los núcleos virtuales provoca un reinicio del servidor.

3. Seleccione **Aceptar** para guardar los cambios.

## <a name="reset-admin-password"></a>Restablecimiento de la contraseña de administrador

La contraseña del rol de administrador se puede cambiar en Azure Portal.

1. Seleccione el servidor en Azure Portal. En la ventana **Información general**, seleccione **Restablecer contraseña**.
2. Escriba la contraseña nueva y confírmela. El cuadro de texto le pedirá los requisitos de complejidad de la contraseña.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/howto-manage-server-portal/reset-password.png" alt-text="Restablecimiento de la contraseña del servidor flexible":::

3. Seleccione **Guardar** para guardar la contraseña nueva.

## <a name="delete-a-server"></a>Eliminación de un servidor

Puede eliminar el servidor cuando ya no lo necesite.

1. Seleccione el servidor en Azure Portal. En la ventana **Información general**, seleccione **Eliminar**.
2. Escriba el nombre del servidor en el cuadro de entrada para confirmar que lo quiere eliminar.

   :::image type="content" source="./media/howto-manage-server-portal/delete-server.png" alt-text="Eliminación del servidor flexible":::

   > [!IMPORTANT]
   > La eliminación de un servidor es irreversible.

  > [!div class="mx-imgBorder"]
  > ![Eliminación del servidor flexible](./media/howto-manage-server-portal/delete-server.png)  

3. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de los conceptos de copia de seguridad y restauración](concepts-backup-restore.md)
- [Ajuste y supervisión del servidor](concepts-monitoring.md)
