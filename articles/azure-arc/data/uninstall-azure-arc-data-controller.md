---
title: Eliminación del controlador de datos de Azure Arc
description: Eliminación del controlador de datos de Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 2bae661218989d49b74ed8ca3f694ccb912ef912
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932227"
---
# <a name="delete-azure-arc-data-controller"></a>Eliminación del controlador de datos de Azure Arc

En el siguiente artículo se describe cómo eliminar un controlador de datos de Azure Arc.

Antes de continuar, asegúrese de que todos los servicios de datos que se han creado en el controlador de datos se quitan de la siguiente manera:

## <a name="log-in-to-the-data-controller"></a>Inicio de sesión en el controlador de datos

Inicie sesión en el controlador de datos que quiere eliminar:

```
azdata login
```

## <a name="list--delete-existing-data-services"></a>Enumeración y eliminación de los servicios de datos existentes

Ejecute el comando siguiente para comprobar si hay alguna instancia administrada de SQL creada:

```
azdata arc sql mi list
```

Para cada instancia administrada de SQL de la lista anterior, ejecute el comando delete de la siguiente manera:

```
azdata arc sql mi delete -n <name>
# for example: azdata arc sql mi delete -n sqlinstance1
```

Del mismo modo, para comprobar las instancias de Hiperescala de PostgreSQL, ejecute lo siguiente:

```
azdata arc postgres server list
```

Y para cada instancia de Hiperescala de PostgreSQL, ejecute el comando delete de la siguiente manera:
```
azdata arc postgres server delete -n <name>
# for example: azdata arc postgres server delete -n pg1
```

## <a name="delete-controller"></a>Eliminación del controlador

Una vez eliminadas todas las instancias administradas de SQL y las de Hiperescala de PostgreSQL, el controlador de datos se puede eliminar de la siguiente manera:

```
azdata arc dc delete -n <name> -ns <namespace>
# for example: azdata arc dc delete -ns arc -n arcdc
```

### <a name="remove-sccs-red-hat-openshift-only"></a>Quite los SCC (solo Red Hat OpenShift).

```console
oc adm policy remove-scc-from-user privileged -z default -n arc
oc adm policy remove-scc-from-user anyuid     -z default -n arc
```

### <a name="optionally-delete-the-azure-arc-data-controller-namespace"></a>Opcionalmente, elimine el espacio de nombres del controlador de datos de Azure Arc.


```console
kubectl delete ns <nameSpecifiedDuringCreation>
# for example kubectl delete ns arc
```

## <a name="next-steps"></a>Pasos siguientes

[¿Qué son los servicios de datos habilitados para Azure Arc?](overview.md)