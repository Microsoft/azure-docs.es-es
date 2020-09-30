---
title: 'Copia de seguridad y restauración de los grupos de servidores de Azure Database for PostgreSQL: hiperescala'
description: 'Copia de seguridad y restauración de los grupos de servidores de Azure Database for PostgreSQL: hiperescala'
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: d300f3e02d2a1a83410d5b7d981298a4743fb223
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932354"
---
# <a name="backup-and-restore-for-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Copia de seguridad y restauración de los grupos de servidores de PostgreSQL: hiperescala habilitados para Azure Arc

Puede hacer una copia de seguridad completa o una restauración del grupo de servidores de hiperescala de PostgreSQL habilitado para Azure Arc. Al hacerlo, se hace la copia de seguridad o la restauración del conjunto completo de bases de datos en todos los nodos del grupo de servidores de hiperescala de PostgreSQL habilitado para Azure Arc.
Para hacer una copia de seguridad y restaurarla, debe asegurarse de que haya una clase de almacenamiento de copia de seguridad configurada para el grupo de servidores. Por ahora, debe indicar una clase de almacenamiento de copia de seguridad en el momento de crear el grupo de servidores. Todavía no es posible configurar el grupo de servidores para utilizar una clase de almacenamiento de copia de seguridad una vez que se ha creado.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

> [!CAUTION]
> La versión preliminar no admite la copia de seguridad y restauración de la versión 11 del motor Postgres. Solo admite la copia de seguridad y restauración de la versión 12 de Postgres.

## <a name="verify-configuration"></a>Comprobación de la configuración

En primer lugar, compruebe si el grupo de servidores existente se ha configurado para utilizar la clase de almacenamiento de copia de seguridad.

Ejecute el siguiente comando después de establecer el nombre del grupo de servidores:
```console
 azdata arc postgres server show -n postgres01
```
Examine la sección de almacenamiento de la salida:
```console
...
"storage": {
      "backups": {
        "className": "local-storage"
      },
      "data": {
        "className": "local-storage",
        "size": "5Gi"
      },
      "logs": {
        "className": "local-storage",
        "size": "5Gi"
      }
    }
...
```
Si ve una sección de copias de seguridad, significa que el grupo de servidores se ha configurado para usar una clase de almacenamiento de copia de seguridad y está listo para que haga copias de seguridad y restauraciones. Si no ve una sección de copias de seguridad, debe eliminar y volver a crear el grupo de servidores para configurar la clase de almacenamiento de copias de seguridad. En este momento, todavía no es posible configurar una clase de almacenamiento de copias de seguridad después de que se haya creado el grupo de servidores.

>[!IMPORTANT]
>Si el grupo de servidores ya está configurado para usar una clase de almacenamiento de copia de seguridad, omita el paso siguiente y vaya directamente al paso "Copia de seguridad completa manual".

## <a name="create-a-server-group"></a>Crear un grupo de servidores 

A continuación, cree un grupo de servidores configurado para copias de seguridad y restauraciones.

Para hacer copias de seguridad y restaurarlas, debe crear un servidor que esté configurado con una clase de almacenamiento.

Para obtener una lista de las clases de almacenamiento disponibles en el clúster de Kubernetes, ejecute el siguiente comando:

```console
kubectl get sc
```

<!--The general format of create server group command is documented [here](create-postgresql-instances.md)-->

```console
azdata arc postgres server create -n <name> --workers 2 --storage-class-backups <storage class name> [--storage-class-data <storage class name>] [--storage-class-logs <storage class name>]
```

Por ejemplo, si ha creado un entorno simple basado en kubeadm:
```console
azdata arc postgres server create -n postgres01 --workers 2 --storage-class-backups local-storage
```

## <a name="take-manual-full-backup"></a>Copia de seguridad completa manual

A continuación, haga una copia de seguridad completa manual.

Para hacer una copia de seguridad completa de todas las carpetas de datos y de registro del grupo de servidores, ejecute el siguiente comando:

```console
azdata arc postgres backup create [--name <backup name>] --server-name <server group name> [--no-wait] 
```
Donde:
- __name__ indica el nombre de una copia de seguridad
- __server-name__ indica un grupo de servidores
- __no-wait__ indica que la línea de comandos no esperará a que se complete la copia de seguridad para que pueda seguir usando esta ventana de línea de comandos.

>**Nota**: El comando que le permite enumerar las copias de seguridad que están disponibles para la restauración no muestra todavía la fecha ni la hora en que se hizo la copia de seguridad. Por lo tanto, se recomienda asignar un nombre a la copia de seguridad (mediante el parámetro --name) que incluye la información de fecha y hora.

Este comando coordinará una copia de seguridad completa distribuida en todos los nodos que constituyen el grupo de servidores de hiperescala de PostgreSQL habilitado para Azure Arc. En otras palabras, hará una copia de seguridad de todos los datos de los nodos Coordinador y Trabajador.

Por ejemplo:
```console
azdata arc postgres backup create --name MyBackup_Aug31_0730amPST --server-name postgres01
```

Cuando se complete la copia de seguridad, se devolverá el identificador, el nombre y el estado de la copia de seguridad. Por ejemplo:
```console
{
  "ID": "d134f51aa87f4044b5fb07cf95cf797f",
  "name": "MyBackup_Aug31_0730amPS",
  "state": "Done"
}
```

> [!NOTE]
> Todavía no es posible:
> - Programación de copias de seguridad automáticas
> - Mostrar el progreso de una copia de seguridad mientras se está llevando a cabo

## <a name="list-backups"></a>Enumeración de copias de seguridad

Permite enumerar las copias de seguridad que están disponibles para restaurar.

Para enumerar las copias de seguridad que están disponibles para restaurar, ejecute el siguiente comando:

```console
azdata arc postgres backup list --server-name <servergroup name>
```

Por ejemplo:
```console
azdata arc postgres backup list --server-name postgres01
```

Devuelve un resultado como el siguiente:
```console
ID                                Name                      State
--------------------------------  ------------------------  -------
d134f51aa87f4044b5fb07cf95cf797f  MyBackup_Aug31_0730amPST  Done
```

## <a name="restore-a-backup"></a>Restaurar una copia de seguridad

Para restaurar la copia de seguridad de un grupo de servidores completo, ejecute el comando:

```console
azdata arc postgres backup restore --server-name <server group name> --backup-id <backup id>
```

Donde:
- __backup-id__ es el identificador de la copia de seguridad que se muestra en el comando list backup (consulte el paso 3).
De este modo se coordinará una restauración completa distribuida en todos los nodos que constituyen el grupo de servidores de hiperescala de PostgreSQL habilitado para Azure Arc. En otras palabras, hará una restauración de todos los datos de los nodos Coordinador y Trabajador.

Por ejemplo:
```console
azdata arc postgres backup restore --server-name postgres01 --backup-id d134f51aa87f4044b5fb07cf95cf797f
```

Una vez completada la operación de restauración, devolverá una salida similar a la siguiente en la línea de comandos:
```console
{
  "ID": "d134f51aa87f4044b5fb07cf95cf797f",
  "state": "Done"
}
```
> [!NOTE]
> Todavía no es posible:
> - Restaurar una copia de seguridad indicando su nombre
> - Restaurar un grupo de servidores con un nombre diferente o en un grupo de servidores diferente
> - Mostrar el progreso de una operación de restauración

## <a name="delete-backups"></a>Eliminar copias de seguridad
No se puede establecer la retención de copias de seguridad en la versión preliminar. Sin embargo, puede eliminar manualmente las copias de seguridad que no necesite.
El comando general para eliminar copias de seguridad es:
```console
azdata arc postgres backup delete  [--server-name, -sn] {[--name, -n], -id}
```
donde:
- `--server-name` es el nombre del grupo de servidores del que el usuario desea eliminar una copia de seguridad.
- `--name` es el nombre de la copia de seguridad que se va a eliminar.
- `-id` es el identificador de la copia de seguridad que se va a eliminar.

> [!NOTE]
> `--name` y `-id` se excluyen mutuamente.

Puede recuperar el nombre y el identificador de las copias de seguridad ejecutando el comando de enumeración de copias de seguridad como se explicó en el párrafo anterior.

Por ejemplo, suponga que tiene las siguientes copias de seguridad enumeradas:
```console
azdata arc postgres backup list -sn postgres01
ID                                Name                    State
--------------------------------  ----------------------  -------
5b0481dfc1c94b4cac79dd56a1bb21f4  MyBackup091720200110am  Done
0cf39f1e92344e6db4cfa285d36c7b14  MyBackup091720200111am  Done
```
y desea eliminar la primera de ellas; en ese caso, ejecutaría el siguiente comando:
```console
azdata arc postgres backup delete -sn postgres01 -n MyBackup091720200110am
{
  "ID": "5b0481dfc1c94b4cac79dd56a1bb21f4",
  "name": "MyBackup091720200110am",
  "state": "Done"
}
```
Si tuviera que enumerar las copias de seguridad en ese momento, obtendría el siguiente resultado:
```console
azdata arc postgres backup list -sn postgres01
ID                                Name                    State
--------------------------------  ----------------------  -------
0cf39f1e92344e6db4cfa285d36c7b14  MyBackup091720200111am  Done
```

Para más detalles sobre el comando de eliminación, ejecute:
```console
azdata arc postgres backup delete --help
```

## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre el [escalado horizontal (adición de nodos de trabajo)](scale-out-postgresql-hyperscale-server-group.md) del grupo de servidores.
- Obtenga información sobre [cómo escalar o reducir verticalmente (aumentar o reducir la memoria o los núcleos virtuales)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) el grupo de servidores.
