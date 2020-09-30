---
title: 'Configuración de una alta disponibilidad en Azure Database for PostgreSQL: Hiperescala (Citus)'
description: Cómo habilitar o deshabilitar la alta disponibilidad
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 07/27/2020
ms.openlocfilehash: 46b842994cbcf7efe66d5992c79246d77626e268
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907390"
---
# <a name="configure-hyperscale-citus-high-availability"></a>Configurar la alta disponibilidad de Hiperescala (Citus)

Azure Database for PostgreSQL - Hiperescala (Citus) proporciona alta disponibilidad (HA) para evitar tiempo de inactividad en la base de datos. Con la alta disponibilidad habilitada, todos los nodos de un grupo de servidores obtendrán un estado de espera. Si el nodo original pasa a ser incorrecto, se promoverá su modo de espera para reemplazarlo.

> [!IMPORTANT]
> Dado que la alta disponibilidad duplica el número de servidores del grupo, también duplicará el coste.

La habilitación de la alta disponibilidad es posible durante la creación del grupo de servidores o después en la pestaña **Compute + Storage** (Proceso y almacenamiento) para su grupo de servidores en Azure Portal. La interfaz de usuario es similar en cualquier caso. Arrastre el control deslizante de **Alta disponibilidad** de NO a SÍ:

:::image type="content" source="./media/howto-hyperscale-high-availability/01-ha-slider.png" alt-text="control de alta disponibilidad":::

Haga clic en el botón **Guardar** para aplicar la selección. La habilitación de la alta disponibilidad puede llevar algún tiempo, ya que el grupo de servidores aprovisiona los datos en espera y los transmite por secuencias.

En la pestaña **Introducción** del grupo de servidores se enumerarán todos los nodos y sus esperas, junto con una columna **Alta disponibilidad** que indica si la alta disponibilidad está habilitada correctamente para cada nodo.

:::image type="content" source="./media/howto-hyperscale-high-availability/02-ha-column.png" alt-text="Columna de alta disponibilidad en la información general del grupo de servidores":::

### <a name="next-steps"></a>Pasos siguientes

Para más información sobre la [alta disponibilidad](concepts-hyperscale-high-availability.md).
