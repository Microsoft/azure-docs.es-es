---
title: 'Configuración de parámetros de servidor de Azure Database for PostgreSQL: Servidor flexible mediante Azure Portal'
description: En este artículo se explica cómo configurar los parámetros de Postgres en el Servidor Flexible de Azure Database for PostgreSQL mediante Azure Portal.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 4f945c2c7fffb143bdb8324a330775fb072b25c1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932170"
---
# <a name="configure-server-parameters-in-azure-database-for-postgresql---flexible-server-via-the-azure-portal"></a>Configuración de parámetros de servidor en Azure Database for PostgreSQL: Servidor flexible mediante Azure Portal 

Puede enumerar, mostrar y actualizar los parámetros de configuración de un Servidor flexible de Azure Database for PostgreSQL a través de Azure Portal.

## <a name="prerequisites"></a>Prerequisites

Para seguir esta guía, necesitará:
- [Servidor flexible de Azure Database for PostgreSQL](quickstart-create-server-portal.md)

## <a name="viewing-and-editing-parameters"></a>Ver y editar parámetros

1. Abra [Azure Portal](https://portal.azure.com).

2. Seleccione el servidor flexible.

3. En la sección **CONFIGURACIÓN**, seleccione **Parámetros del servidor**. La página muestra una lista de parámetros, sus valores y descripciones.
![Página de información general de parámetros](./media/howto-configure-server-parameters-in-portal/3-overview-of-parameters.png)

4. Seleccione el botón **desplegable** para ver los posibles valores para los parámetros de tipo enumerado, como client_min_messages.
![Menú desplegable de enumeración](./media/howto-configure-server-parameters-in-portal/4-enum-drop-down.png)

5. Seleccione o mantenga el puntero sobre el botón **i** (información) para ver el rango de valores posibles para los parámetros numéricos, como cpu_index_tuple_cost.
![botón de información](./media/howto-configure-server-parameters-in-portal/4-information-button.png)

6. Si es necesario, utilice el **cuadro de búsqueda** para reducir a un parámetro específico. Se busca en el nombre y la descripción de los parámetros.
![Resultados de la búsqueda](./media/howto-configure-server-parameters-in-portal/5-search.png)

7. Cambie los valores de los parámetros que le gustaría ajustar. Todos los cambios que realiza en una sesión se resaltan en color púrpura. Una vez que haya cambiado los valores, puede seleccionar **Guardar**. O bien puede **descartar** los cambios.
![Guardar o descartar cambios](./media/howto-configure-server-parameters-in-portal/6-save-and-discard-buttons.png)

8. Si ha guardado los nuevos valores para los parámetros, siempre puede revertir todos los elementos a los valores predeterminados; para ello, seleccione **Restablecer todos los valores predeterminados**.
![Restablecer todos los valores predeterminados](./media/howto-configure-server-parameters-in-portal/7-reset-to-default-button.png)

## <a name="next-steps"></a>Pasos siguientes

Más información sobre lo siguiente:

- [Información general de los parámetros del servidor en Azure Database for PostgreSQL](concepts-servers.md)
- [Configuración de parámetros con la CLI de Azure](howto-configure-server-parameters-using-cli.md)
