---
title: 'Administración de un servidor de Azure Database for PostgreSQL: Azure Portal'
description: Obtenga información sobre cómo administrar un servidor de Azure Database for PostgreSQL desde Azure Portal.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: how-to
ms.date: 11/20/2019
ms.openlocfilehash: 02a50a94b0b07d1755abe78c567df7ff5c7eda92
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907453"
---
# <a name="manage-an-azure-database-for-postgresql-server-using-the-azure-portal"></a>Administración de un servidor de Azure Database for PostgreSQL mediante Azure Portal

En este artículo se explica cómo administrar los servidores de Azure Database for PostgreSQL. Entre las tareas de administración se incluyen el escalado de proceso y almacenamiento, el restablecimiento de contraseñas de administración y la visualización de detalles del servidor.

## <a name="sign-in"></a>Iniciar sesión

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-a-server"></a>Creación de un servidor

Visite el [inicio rápido](quickstart-create-server-database-portal.md) para más información sobre cómo crear un servidor de Azure Database for PostgreSQL y empezar a trabajar con él.

## <a name="scale-compute-and-storage"></a>Escalado de proceso y almacenamiento

Tras crear el servidor, puede escalar entre los niveles De uso general y Con optimización para memoria a medida que cambien sus necesidades. También puede escalar el proceso y la memoria aumentando o reduciendo núcleos virtuales. El almacenamiento se puede escalar verticalmente (pero no reducir).

### <a name="scale-between-general-purpose-and-memory-optimized-tiers"></a>Escalado entre niveles De uso general y Con optimización de memoria

Puede escalar desde De uso general a Optimizada para memoria y viceversa. No se permite el cambio desde un plan Básico después de haber creado el servidor.

1. Seleccione el servidor en Azure Portal. Seleccione **Plan de tarifa**, que se encuentra en la sección **Configuración**.

2. Seleccione **De uso general** o **Con optimización de memoria**, dependiendo de lo que esté escalando.

   :::image type="content" source="./media/howto-create-manage-server-portal/change-pricing-tier.png" alt-text="Captura de pantalla de Azure Portal para elegir el nivel Básico, De uso general o Memoria optimizada en Azure Database for PostgreSQL":::

   > [!NOTE]
   > El cambio de los niveles provoca un reinicio del servidor.

3. Seleccione **Aceptar** para guardar los cambios.

### <a name="scale-vcores-up-or-down"></a>Escalado o reducción vertical de los núcleos virtuales

1. Seleccione el servidor en Azure Portal. Seleccione **Plan de tarifa**, que se encuentra en la sección **Configuración**.

2. Cambie el valor de **vCore** moviendo el control deslizante hasta alcanzar el valor deseado.

   :::image type="content" source="./media/howto-create-manage-server-portal/scaling-compute.png" alt-text="Captura de pantalla de Azure Portal para elegir la opción de núcleo virtual en Azure Database for PostgreSQL":::

   > [!NOTE]
   > El escalado de los núcleos virtuales provoca un reinicio del servidor.

3. Seleccione **Aceptar** para guardar los cambios.

### <a name="scale-storage-up"></a>Escalado vertical del almacenamiento

1. Seleccione el servidor en Azure Portal. Seleccione **Plan de tarifa**, que se encuentra en la sección **Configuración**.

2. Cambie el valor de **Almacenamiento** moviendo el control deslizante hacia arriba hasta alcanzar el valor que se quiera.

   :::image type="content" source="./media/howto-create-manage-server-portal/scaling-storage.png" alt-text="Captura de pantalla de Azure Portal para elegir la escala de almacenamiento en Azure Database for PostgreSQL":::

   > [!NOTE]
   > El almacenamiento no se puede reducir verticalmente.

3. Seleccione **Aceptar** para guardar los cambios.

## <a name="update-admin-password"></a>Actualización de la contraseña del administrador

La contraseña del rol de administrador se puede cambiar en Azure Portal.

1. Seleccione el servidor en Azure Portal. En la ventana **Información general**, seleccione **Restablecer contraseña**.

   :::image type="content" source="./media/howto-create-manage-server-portal/overview-reset-password.png" alt-text="Captura de pantalla de Azure Portal para restablecer la contraseña en Azure Database for PostgreSQL":::

2. Escriba la contraseña nueva y confírmela. El cuadro de texto le pedirá los requisitos de complejidad de la contraseña.

   :::image type="content" source="./media/howto-create-manage-server-portal/reset-password.png" alt-text="Captura de pantalla de Azure Portal para restablecer la contraseña y guardarla en Azure Database for PostgreSQL":::

3. Seleccione **Aceptar** para guardar la nueva contraseña.

## <a name="delete-a-server"></a>Eliminación de un servidor

Puede eliminar el servidor cuando ya no lo necesite. 

1. Seleccione el servidor en Azure Portal. En la ventana **Información general**, seleccione **Eliminar**.

   :::image type="content" source="./media/howto-create-manage-server-portal/overview-delete.png" alt-text="Captura de pantalla de Azure Portal para eliminar el servidor en Azure Database for PostgreSQL":::

2. Escriba el nombre del servidor en el cuadro de entrada para confirmar que es el servidor que quiere eliminar.

   :::image type="content" source="./media/howto-create-manage-server-portal/confirm-delete.png" alt-text="Captura de pantalla de Azure Portal para confirmar la eliminación del servidor en Azure Database for PostgreSQL":::

   > [!NOTE]
   > La eliminación de un servidor es irreversible.

3. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre [copias de seguridad y restauración del servidor](howto-restore-server-portal.md).
- Obtenga más información sobre las opciones de [supervisión y ajuste en Azure Database for PostgreSQL](concepts-monitoring.md).
