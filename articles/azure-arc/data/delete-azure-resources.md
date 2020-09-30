---
title: Eliminación de recursos de Azure
description: Eliminación de recursos de Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: c07a0d8279c8d2f8792f5df3e35d2d630b1a86bd
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932373"
---
# <a name="delete-resources-from-azure"></a>Eliminación de recursos de Azure

> [!NOTE]
>  Las opciones para eliminar recursos que se describen en este artículo son irreversibles.

> [!NOTE]
>  Dado que el único modo de conectividad que se ofrece actualmente para los servicios de datos habilitados para Azure Arc es el modo Conectado indirectamente, la eliminación de una instancia de Kubernetes no la elimina de Azure y la eliminación de una instancia de Azure no la elimina de Kubernetes.  Por ahora, la eliminación de un recurso es un proceso de dos pasos y se mejorará en el futuro.  En el futuro, Kubernetes será el origen de verdad y Azure se actualizará para reflejarlo.

En algunos casos, puede que tenga que eliminar manualmente los recursos de los servicios de datos habilitados para Azure Arc en Azure Resource Manager (ARM).  Puede eliminar estos recursos con cualquiera de las siguientes opciones.

- [Eliminación de recursos de Azure](#delete-resources-from-azure)
  - [Eliminación de un grupo de recursos completo](#delete-an-entire-resource-group)
  - [Eliminación de recursos específicos del grupo de recursos](#delete-specific-resources-in-the-resource-group)
  - [Eliminación de recursos mediante la CLI de Azure](#delete-resources-using-the-azure-cli)
    - [Eliminación de recursos de SQL Managed Instance mediante la CLI de Azure](#delete-sql-managed-instance-resources-using-the-azure-cli)
    - [Eliminación de los recursos del grupo de servidores de Hiperescala de PostgreSQL mediante la CLI de Azure](#delete-postgresql-hyperscale-server-group-resources-using-the-azure-cli)
    - [Eliminación de los recursos del controlador de datos de Azure Arc mediante la CLI de Azure](#delete-azure-arc-data-controller-resources-using-the-azure-cli)
    - [Eliminación de un grupo de recursos mediante la CLI de Azure](#delete-a-resource-group-using-the-azure-cli)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="delete-an-entire-resource-group"></a>Eliminación de un grupo de recursos completo
Si ha utilizado un grupo de recursos específico y dedicado para los servicios de datos habilitados para Azure Arc y desea eliminar *todo* lo que hay dentro del grupo de recursos, puede eliminar el grupo de recursos, lo que eliminará todo lo que contiene.  

Para eliminar un grupo de recursos en Azure Portal, realice los pasos siguientes:

- En Azure Portal, vaya al grupo de recursos en el que se han creado los recursos de los servicios de datos habilitados para Azure Arc.
- Haga clic en el botón **Eliminar grupo de recursos**.
- Para confirmar la eliminación, escriba el nombre del grupo de recursos y haga clic en **Eliminar**.

## <a name="delete-specific-resources-in-the-resource-group"></a>Eliminación de recursos específicos del grupo de recursos

Para eliminar recursos específicos de los servicios de datos habilitados para Azure Arc de un grupo de recursos en Azure Portal, realice los siguientes pasos:

- En Azure Portal, vaya al grupo de recursos en el que se han creado los recursos de los servicios de datos habilitados para Azure Arc.
- Seleccione todos los recursos que se van a eliminar.
- Haga clic en el botón Eliminar.
- Escriba "Sí" para confirmar la eliminación y haga clic en **Eliminar**.

## <a name="delete-resources-using-the-azure-cli"></a>Eliminación de recursos mediante la CLI de Azure

Puede eliminar recursos específicos de los servicios de datos habilitados para Azure Arc mediante la CLI de Azure.

### <a name="delete-sql-managed-instance-resources-using-the-azure-cli"></a>Eliminación de recursos de SQL Managed Instance mediante la CLI de Azure

Para eliminar de Azure los recursos de SQL Managed Instance mediante la CLI de Azure, reemplace los valores de marcador de posición en el comando siguiente y ejecútelo.

```console
az resource delete --name <sql instance name> --resource-type Microsoft.AzureData/sqlManagedInstances --resource-group <resource group name>

#Example
#az resource delete --name sql1 --resource-type Microsoft.AzureData/sqlManagedInstances --resource-group rg1
```

### <a name="delete-postgresql-hyperscale-server-group-resources-using-the-azure-cli"></a>Eliminación de los recursos del grupo de servidores de Hiperescala de PostgreSQL mediante la CLI de Azure

Para eliminar de Azure un recurso del grupo de servidores de Hiperescala de PostgreSQL mediante la CLI de Azure, reemplace los valores de marcador de posición en el comando siguiente y ejecútelo.

```console
az resource delete --name <postgresql instance name> --resource-type Microsoft.AzureData/postgresInstances --resource-group <resource group name>

#Example
#az resource delete --name pg1 --resource-type Microsoft.AzureData/postgresInstances --resource-group rg1
```

### <a name="delete-azure-arc-data-controller-resources-using-the-azure-cli"></a>Eliminación de los recursos del controlador de datos de Azure Arc mediante la CLI de Azure

> [!NOTE]
> Antes de eliminar un controlador de datos de Azure Arc, debe eliminar todos los recursos de instancia de base de datos que administra.

Para eliminar de Azure un controlador de datos de Azure Arc mediante la CLI de Azure, reemplace los valores de marcador de posición en el comando siguiente y ejecútelo.

```console
az resource delete --name <data controller name> --resource-type Microsoft.AzureData/dataControllers --resource-group <resource group name>

#Example
#az resource delete --name dc1 --resource-type Microsoft.AzureData/dataControllers --resource-group rg1
```

### <a name="delete-a-resource-group-using-the-azure-cli"></a>Eliminación de un grupo de recursos mediante la CLI de Azure

También puede usar la CLI de Azure para [eliminar un grupo de recursos](/azure-resource-manager/management/delete-resource-group).
