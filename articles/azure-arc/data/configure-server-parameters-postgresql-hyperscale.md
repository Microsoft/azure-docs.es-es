---
title: Configuración de los parámetros del servidor del motor de Postgres para el grupo de servidores de Hiperescala de PostgreSQL en Azure Arc
titleSuffix: Azure Arc enabled data services
description: Configuración de los parámetros del servidor del motor de Postgres para el grupo de servidores de Hiperescala de PostgreSQL en Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 4e8813647211e0adbfe43a45ae0d19dc12a4a165
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932246"
---
# <a name="set-the-database-engine-settings-for-azure-arc-enabled-postgresql-hyperscale"></a>Establecimiento de la configuración del motor de base de datos para Hiperescala de PostgreSQL habilitada para Azure Arc

En este documento se describen los pasos para establecer la configuración del motor de base de datos del grupo de servidores de Hiperescala de PostgreSQL en valores personalizados (no predeterminados). Para más información sobre los parámetros del motor de base de datos que se pueden establecer y cuál es su valor predeterminado, consulte la documentación de PostgreSQL [aquí](https://www.postgresql.org/docs/current/runtime-config.html).

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

> [!NOTE]
> La versión preliminar no admite la configuración de los parámetros siguientes: 
>
> - `archive_command`
> - `archive_timeout`
> - `log_directory`
> - `log_file_mode`
> - `log_filename`
> - `restore_command`
> - `shared_preload_libraries`
> - `synchronous_commit`
> - `ssl`
> - `wal_level`

## <a name="syntax"></a>Syntax

El formato general del comando para establecer la configuración del motor de base de datos es:

```console
azdata arc postgres server edit -n <server group name>, [{--engine-settings, -e}] [{--replace-engine-settings, --re}] {'<parameter name>=<parameter value>, ...'}
```

## <a name="show-the-current-custom-values-of-the-parameters-settings"></a>Mostrar los valores personalizados actuales de la configuración de parámetros

## <a name="with-azdata-cli-command"></a>Con el comando de la CLI azdata

```console
azdata arc postgres server show -n <server group name>
```

Por ejemplo:

```console
azdata arc postgres server show -n postgres01
```

Este comando devuelve las especificaciones del grupo de servidores, en las que puede ver los parámetros que ha establecido. Si no existe la sección engine\settings, significa que todos los parámetros se ejecutan en su valor predeterminado:

```console
 "
...
engine": {
      "settings": {
        "default": {
          "autovacuum_vacuum_threshold": "65"
        }
      }
    },
...
```

## <a name="with-kubectl-command"></a>Con el comando kubectl

Siga los pasos que se indican a continuación.

### <a name="1-retrieve-the-kind-of-custom-resource-definition-for-your-server-group"></a>1. Recuperar el tipo de definición de recurso personalizado del grupo de servidores

Ejecute:

```console
azdata arc postgres server show -n <server group name>
```

Por ejemplo:

```console
azdata arc postgres server show -n postgres01
```

Este comando devuelve las especificaciones del grupo de servidores, en las que puede ver los parámetros que ha establecido. Si no existe la sección engine\settings, significa que todos los parámetros se ejecutan en su valor predeterminado:

```
> {
  >"apiVersion": "arcdata.microsoft.com/v1alpha1",
  >"**kind**": "**postgresql-12**",
  >"metadata": {
    >"creationTimestamp": "2020-08-25T14:32:23Z",
    >"generation": 1,
    >"name": "postgres01",
    >"namespace": "arc",
```

En ella, busque el campo "kind" (tipo) y anote el valor, por ejemplo: `postgresql-12`.

### <a name="2-describe-the-kubernetes-custom-resource-corresponding-to-your-server-group"></a>2. Describir el recurso personalizado de Kubernetes correspondiente al grupo de servidores 

El formato general del comando es:

```console
kubectl describe <kind of the custom resource> <server group name> -n <namespace name>
```

Por ejemplo:

```console
kubectl describe postgresql-12 postgres01
```

Si hay valores personalizados establecidos para la configuración del motor, los devolverá. Por ejemplo:

```console
Engine:
...
    Settings:
      Default:
        autovacuum_vacuum_threshold:  65
```

Si no ha establecido valores personalizados para ninguno de los valores del motor, la sección de configuración del motor del conjunto de resultados estará vacía, como:

```console
Engine:
...
    Settings:
      Default:
```

## <a name="set-custom-values-for-the-engine-settings"></a>Establecimiento de valores personalizados para la configuración del motor

Los comandos siguientes establecen los parámetros del nodo de coordinación y los nodos de trabajo de Hiperescala de PostgreSQL en los mismos valores. Todavía no es posible establecer parámetros por rol en el grupo de servidores. Es decir, todavía no es posible configurar un parámetro determinado con un valor específico en el nodo de coordinación y con otro valor para los nodos de trabajo.

## <a name="set-a-single-parameter"></a>Establecimiento de un único parámetro

```console
azdata arc server edit -n <server group name> -e <parameter name>=<parameter value>
```

Por ejemplo:

```console
azdata arc postgres server edit -n postgres01 -e shared_buffers=8MB
```

## <a name="set-multiple-parameters-with-a-single-command"></a>Establecimiento de varios parámetros con un solo comando

```console
azdata arc postgres server edit -n <server group name> -e '<parameter name>=<parameter value>, <parameter name>=<parameter value>,...'
```

Por ejemplo:

```console
azdata arc postgres server edit -n postgres01 -e 'shared_buffers=8MB, max_connections=50'
```

## <a name="reset-a-parameter-to-its-default-value"></a>Restablecimiento de un parámetro a su valor predeterminado

Para restablecer un parámetro a su valor predeterminado, establézcalo sin indicar un valor. 

Por ejemplo:

```console
azdata arc postgres server edit -n postgres01 -e shared_buffers=
```

## <a name="reset-all-parameters-to-their-default-values"></a>Restablecimiento de todos los parámetros a sus valores predeterminados

```console
azdata arc postgres server edit -n <server group name> -e '' -re
```

Por ejemplo:

```console
azdata arc postgres server edit -n postgres01 -e '' -re
```

## <a name="special-considerations"></a>Consideraciones especiales

### <a name="set-a-parameter-which-value-contains-a-comma-space-or-special-character"></a>Establecimiento de un parámetro que contiene una coma, un espacio o un carácter especial

```console
azdata arc postgres server edit -n <server group name> -e '<parameter name>="<parameter value>"'
```

Por ejemplo:

```console
azdata arc postgres server edit -n postgres01 -e 'custom_variable_classes = "plpgsql,plperl"'
```

### <a name="pass-an-environment-variable-in-a-parameter-value"></a>Paso de una variable de entorno en un valor de parámetro

La variable de entorno se debe encapsular dentro de "''" para que no se resuelva antes de establecerse.

Por ejemplo: 

```console
azdata arc postgres server edit -n postgres01 -e 'search_path = "$user"'
```



## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre el [escalado horizontal (adición de nodos de trabajo)](scale-out-postgresql-hyperscale-server-group.md) del grupo de servidores.
- Obtenga información sobre [cómo escalar o reducir verticalmente (aumentar o reducir la memoria o los núcleos virtuales)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) el grupo de servidores.
