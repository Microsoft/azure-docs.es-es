---
title: Eliminar red virtual después de eliminar la base de datos de SQL Azure instancia administrada | Microsoft Docs
description: Obtenga información sobre cómo eliminar la red virtual después de eliminar la base de datos de SQL Azure instancia administrada.
services: sql-database
ms.service: sql-database
ms.subservice: management
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: danil
ms.reviewer: douglas, carlrab, sstein
manager: craigg
ms.date: 05/07/2019
ms.openlocfilehash: 61f6c25031c4906e65c2f75a7679600741e8311a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791373"
---
# <a name="delete-subnet-after-deleting-azure-sql-database-managed-instance"></a>Eliminar la subred después de eliminar la base de datos de SQL Azure instancia administrada

En este artículo proporciona instrucciones sobre cómo eliminar manualmente la subred después de eliminar el último de Azure SQL Database administre la instancia que residen en él.

El [clúster virtual](sql-database-managed-instance-connectivity-architecture.md#virtual-cluster-connectivity-architecture) que ha incluido la eliminó una instancia administrada se conservará durante 12 horas desde la eliminación de la instancia. El clúster virtual se mantiene activo por cuestiones de diseño para habilitar la creación más rápida de las instancias administradas en la misma subred. Mantener un clúster virtual vacío es de forma gratuita. Durante este período, no se puede eliminar la subred asociada con el clúster virtual.

Es posible a través de la eliminación manual del clúster virtual liberación inmediata de la subred usada por un clúster virtual vacío. Eliminación del clúster virtual se puede lograr a través de Azure portal o la API de clústeres virtuales.

> [!NOTE]
> El clúster virtual no debe contener ninguna instancia administrada para la eliminación se realice correctamente.

## <a name="delete-virtual-cluster-from-azure-portal"></a>Eliminar clúster virtual de Azure portal

Para eliminar un clúster virtual mediante Azure portal, busque los recursos de clúster virtual mediante la búsqueda integrada.

![Busque el clúster virtual.](./media/sql-database-managed-instance-delete-virtual-cluster/virtual-clusters-search.png)

Una vez que localice el clúster virtual que desea eliminar, seleccione este recurso y seleccione la opción de eliminación. Se le pedirá que confirme la eliminación de clúster virtual.

![Eliminar clúster virtual.](./media/sql-database-managed-instance-delete-virtual-cluster/virtual-clusters-delete.png)

Confirmación de que se ha eliminado el clúster virtual se proporciona en notificaciones de Azure portal. Eliminación correcta del clúster virtual libera inmediatamente la subred para volver a usarlo.

## <a name="delete-virtual-cluster-using-api"></a>Eliminar clúster virtual mediante la API

Para eliminar un virtual del clúster a través del uso de API los parámetros de URI especificados en el [clústeres virtuales eliminación método](https://docs.microsoft.com/rest/api/sql/virtualclusters/delete).

## <a name="next-steps"></a>Pasos siguientes

- Para información general, consulte [¿Qué es Instancia administrada?](sql-database-managed-instance.md)
- Obtenga información sobre la [arquitectura de conectividad en Instancia administrada](sql-database-managed-instance-connectivity-architecture.md).
- Aprenda a [modificar una red virtual existente para Instancia administrada](sql-database-managed-instance-configure-vnet-subnet.md).
- Para obtener un tutorial que muestre cómo crear una red virtual, crear una instancia administrada y restaurar una base de datos desde una copia de seguridad de base de datos, consulte [Creación de una Instancia administrada de Azure SQL Database](sql-database-managed-instance-get-started.md).
- Para ver información relativa a los distintos problemas de DNS, consulte [Configuración de un DNS personalizado](sql-database-managed-instance-custom-dns.md).
