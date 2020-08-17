---
title: Creación de una red virtual
titleSuffix: Azure SQL Managed Instance
description: En este artículo se describe cómo crear una red virtual configurada para admitir la implementación de Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, carlrab
ms.date: 09/12/2019
ms.openlocfilehash: 9ec6b7747ad22df96fd8ab5f9b5b5bc1e27b0993
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543748"
---
# <a name="create-a-virtual-network-for-azure-sql-managed-instance"></a>Creación de una red virtual para Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se explica cómo crear una red virtual y una subred válidas en las que puede implementar una Instancia administrada de Azure SQL.

Instancia administrada de Azure SQL debe implementarse dentro de una [red virtual](../../virtual-network/virtual-networks-overview.md) de Azure. Esta implementación permite los siguientes escenarios:

- Protección de una dirección IP privada
- Conexión a SQL Managed Instance directamente desde una red local
- Conexión de SQL Managed Instance con un servidor vinculado o con otro almacén de datos local
- Conexión de SQL Managed Instance a los recursos de Azure  

> [!NOTE]
> Debería [determinar el tamaño de la subred para la Instancia administrada de SQL](vnet-subnet-determine-size.md) antes de implementar la primera instancia. No se puede cambiar el tamaño de la subred después de colocar los recursos en su interior.
>
> Si piensa usar una red virtual existente, tiene que modificar la configuración de red para dar cabida a SQL Managed Instance. Para más información, consulte [Modificación de una red virtual existente para Instancia administrada de SQL](vnet-existing-add-subnet.md).
>
> Después de crear una instancia administrada, no se admite el traslado de la instancia administrada o la red virtual a otro grupo de recursos o suscripción.  Tampoco se admite el traslado de la instancia administrada a otra subred.
>

## <a name="create-a-virtual-network"></a>Creación de una red virtual

La manera más fácil de crear y configurar una red virtual es usar una plantilla de implementación de Azure Resource Manager.

1. Inicie sesión en Azure Portal.

2. Seleccione el botón **Implementar en Azure**:

   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-sql-managed-instance-azure-environment%2Fazuredeploy.json)

   Este botón abre un formulario que puede usar para configurar el entorno de red, donde puede implementar SQL Managed Instance.

   > [!Note]
   > Esta plantilla de Azure Resource Manager implementará una red virtual con dos subredes. Una subred, llamada **ManagedInstances**, está reservada para Instancia administrada de SQL y tiene una tabla de rutas configuradas previamente. La otra subred, denominada **Default**, se usa para otros recursos que deberían acceder a Instancia administrada de SQL (por ejemplo, Azure Virtual Machines).

3. Configure el entorno de red. En el formulario siguiente, puede configurar parámetros de su entorno de red:

   ![Plantilla de Resource Manager para configurar la red de Azure](./media/virtual-network-subnet-create-arm-template/create-mi-network-arm.png)

   Puede cambiar los nombres de la red virtual y las subredes, y ajustar los intervalos IP asociados a sus recursos de red. Una vez que presione el botón **Comprar**, se creará el formulario y configurará el entorno. Si no necesita dos subredes, puede eliminar la predeterminada.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general, vea [¿Qué es Instancia administrada de SQL?](sql-managed-instance-paas-overview.md)
- Obtenga información sobre la [arquitectura de conectividad en Instancia administrada de SQL](connectivity-architecture-overview.md).
- Aprenda a [modificar una red virtual existente para Instancia administrada de SQL](vnet-existing-add-subnet.md).
- Para obtener un tutorial en el que se muestra cómo crear una red virtual y una instancia administrada, y restaurar una base de datos a partir de una copia de seguridad de base de datos, vea [Creación de una instancia administrada](instance-create-quickstart.md).
- Para incidencias de DNS, consulte [Configuración de un DNS personalizado](custom-dns-configure.md).
