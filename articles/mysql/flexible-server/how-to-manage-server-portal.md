---
title: 'Administración de Azure Database for MySQL con Azure Portal: servidor flexible'
description: 'Aprenda a administrar Azure Database for MySQL: servidor flexible desde Azure Portal.'
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 7a01863b3a0c29e94550be67ca957655cff32660
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931905"
---
# <a name="manage-an-azure-database-for-mysql---flexible-server-preview-using-azure-portal"></a>Administración de Azure Database for MySQL: servidor flexible (versión preliminar) mediante Azure Portal


> [!IMPORTANT]
> Azure Database for MySQL: servidor flexible está actualmente en versión preliminar pública.

En este artículo se muestra cómo administrar los servidores flexibles de Azure Database for MySQL (versión preliminar). Las tareas de administración incluyen el escalado de los procesos y el almacenamiento, el restablecimiento de la contraseña de administrador del servidor y la eliminación del servidor.

## <a name="sign-in"></a>Iniciar sesión
Inicie sesión en [Azure Portal](https://portal.azure.com). Vaya al recurso de servidor flexible en Azure Portal.

## <a name="scale-compute-and-storage"></a>Escalado de proceso y almacenamiento

Tras crear el servidor, puede escalar entre los distintos [planes de tarifa](https://azure.microsoft.com/pricing/details/mysql/) a medida que cambien las necesidades. También puede escalar o reducir verticalmente el proceso y la memoria mediante el aumento o la reducción de núcleos virtuales.

1. Seleccione el servidor en Azure Portal. Seleccione **Proceso y almacenamiento** en la sección **Configuración**.

2. Puede cambiar el **nivel de proceso**, el **núcleo virtual** y el **almacenamiento** para escalar verticalmente el servidor con un nivel de proceso superior, o bien hacerlo verticalmente dentro del mismo nivel al aumentar el almacenamiento o los núcleos virtuales al valor que quiera.

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

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/howto-manage-server-portal/delete-server.png" alt-text="Eliminación del servidor flexible":::

   > [!NOTE]
   > La eliminación de un servidor es irreversible.

3. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes
- [Obtenga información sobre cómo iniciar o detener un servidor](how-to-stop-start-server-portal.md).
- [Obtenga información sobre cómo restaurar un servidor](how-to-restore-server-portal.md).
- [Solución de problemas de conexión a Azure Database for PostgreSQL- Hiperescala (Citus)](how-to-troubleshoot-common-connection-issues.md)

