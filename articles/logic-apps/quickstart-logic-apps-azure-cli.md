---
title: Creación y administración de aplicaciones lógicas con la CLI de Azure
description: Use la CLI de Azure para crear una aplicación lógica y, luego, adminístrela mediante operaciones como list, show (get), update y delete.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: quickstart
ms.custom: mvc
ms.date: 07/30/2020
ms.openlocfilehash: e492a5f0afdfc2087e5719df65221d08db0a2e77
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499558"
---
# <a name="quickstart-create-and-manage-logic-apps-using-the-azure-cli"></a>Inicio rápido: Creación y administración de aplicaciones lógicas mediante la CLI de Azure

En este inicio rápido se muestra cómo crear y administrar aplicaciones lógicas mediante la [extensión Logic Apps de la CLI de Azure](/cli/azure/ext/logic/logic?view=azure-cli-latest) (`az logic`). Desde la línea de comandos, puede crear una aplicación lógica mediante el archivo JSON en una definición del flujo de trabajo de una aplicación lógica. Después, para administrar la aplicación lógica, es preciso ejecutar operaciones como `list`, `show` (`get`), `update` y `delete` también desde la línea de comandos.

> [!WARNING]
> Actualmente, la extensión Logic Apps de la CLI de Azure se encuentra en fase *experimental* y *no está incluida en el soporte al cliente*. Use esta extensión de la CLI con precaución, sobre todo en entornos de producción.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* La [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) instalada en un equipo local.
* La [extensión Logic Apps de la CLI de Azure](/cli/azure/azure-cli-extensions-list?view=azure-cli-latest) instalada en el equipo. Para instalar esta extensión, use este comando: `az extension add --name logic`
* [Grupo de recursos de Azure](#example---create-resource-group) en el que se crea la aplicación lógica.

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

Valide el entorno antes de empezar:

* Inicie sesión en Azure Portal y compruebe que la suscripción está activa, para lo que debe ejecutar `az login`.
* Compruebe la versión de la CLI de Azure en una ventana de terminal o de comandos, para lo que debe ejecutar `az --version`. Para saber cuál es la versión más reciente, consulte las [notas de la versión más reciente](/cli/azure/release-notes-azure-cli?tabs=azure-cli&view=azure-cli-latest).
  * Si no tiene la versión más reciente, actualice la instalación, para lo que debe seguir las instrucciones que encontrará en la [guía de instalación del sistema operativo o de la plataforma](/cli/azure/install-azure-cli?view=azure-cli-latest).

### <a name="example---create-resource-group"></a>Ejemplo: creación de un grupo de recursos

Si aún no tiene un grupo de recursos para la aplicación lógica, créelo con el comando `az group create`. Por ejemplo, el siguiente comando crea un grupo de recursos denominado `testResourceGroup` en la ubicación `westus`.

```azurecli-interactive

az group create --name testResourceGroup --location westus

```

La salida muestra `provisioningState` como `Succeeded` cuando el grupo de recursos se crea correctamente:

```output

<...>
  "name": "testResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
<...>

```

## <a name="workflow-definition"></a>Definición del flujo de trabajo

Antes de [crear una aplicación lógica](#create-logic-apps-from-cli) o [actualizar una aplicación lógica ya existente](#update-logic-apps-from-cli) mediante la CLI de Azure, se necesita una definición del flujo de trabajo de la misma. En Azure Portal, para ver la definición del flujo de trabajo subyacente de una aplicación lógica en formato JSON es preciso cambiar de la vista **Diseñador** a la **vista Código**.

Cuando se ejecutan los comandos para crear o actualizar una aplicación lógica, la definición del flujo de trabajo se carga como un parámetro requerido (`--definition`). La definición del flujo de trabajo se debe crear en forma de archivo JSON que siga el [esquema del lenguaje de definición de flujo de trabajo](./logic-apps-workflow-definition-language.md).

## <a name="create-logic-apps-from-cli"></a>Creación de aplicaciones lógicas desde la CLI

Para crear un flujo de trabajo de aplicaciones lógicas desde la CLI de Azure, use el comando [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-create) con un archivo JSON para la definición.

```azurecli

az logic workflow create --definition
                         --location
                         --name
                         --resource-group
                         [--access-control]
                         [--endpoints-configuration]
                         [--integration-account]
                         [--integration-service-environment]
                         [--state {Completed, Deleted, Disabled, Enabled, NotSpecified, Suspended}]
                         [--tags]

```

El comando debe incluir los siguientes [parámetros requeridos](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-create-required-parameters):

| Parámetro | Value | Descripción |
| --------- | ----- | ----------- |
| Definición del flujo de trabajo | `--definition` | Archivo JSON con la [definición del flujo de trabajo](#workflow-definition) de la aplicación lógica. |
| Ubicación | `--location -l` | La región de Azure en la que se encuentra la aplicación lógica. |
| Nombre | `--name -n` | Nombre de la aplicación lógica. El nombre puede contener solo letras, números, guiones (`-`), caracteres de subrayado (`_`), paréntesis (`()`) y puntos (`.`). El nombre también debe ser único entre las regiones. |
| Definición de un nombre de grupo de recursos | `--resource-group -g` | [Grupo de recursos de Azure](../azure-resource-manager/management/overview.md) en el que desea crear la aplicación lógica. Si no tiene un grupo de recursos para su aplicación lógica, [créelo](#example---create-resource-group) antes de empezar. |

También puede incluir [parámetros opcionales](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-create-optional-parameters) adicionales para configurar los controles de acceso, los puntos de conexión, la cuenta de integración, el entorno del servicio de integración, el estado y las etiquetas de recursos de la aplicación lógica.

### <a name="example---create-logic-app"></a>Ejemplo: creación de una aplicación lógica

En este ejemplo, se crea un flujo de trabajo denominado `testLogicApp` en el grupo de recursos `testResourceGroup` de la ubicación `westus`. El archivo JSON `testDefinition.json` contiene la definición del flujo de trabajo.

```azurecli-interactive

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "testDefinition.json"

```

Cuando el flujo de trabajo se crea correctamente, la CLI muestra el código JSON de la definición del nuevo flujo de trabajo. Si se produce algún error en la creación del flujo de trabajo, consulte la [lista de posibles errores](#errors).

## <a name="update-logic-apps-from-cli"></a>Actualización de aplicaciones lógicas desde la CLI

Desde la CLI de Azure también se puede actualizar el flujo de trabajo de una aplicación lógica. Para ello, es preciso usar el comando [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-create).

Este comando debe incluir los mismos [parámetros requeridos](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-create-required-parameters) que cuando se [crea una aplicación lógica](#create-logic-apps-from-cli). También se pueden agregar los mismos [parámetros opcionales](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-create-optional-parameters) que cuando se crea una aplicación lógica.

```azurecli

az logic workflow create --definition
                         --location
                         --name
                         --resource-group
                         [--access-control]
                         [--endpoints-configuration]
                         [--integration-account]
                         [--integration-service-environment]
                         [--state {Completed, Deleted, Disabled, Enabled, NotSpecified, Suspended}]
                         [--tags]

```

### <a name="example---update-logic-app"></a>Ejemplo: actualización de una aplicación lógica

En este ejemplo, el [flujo de trabajo creado en la sección anterior](#example---create-logic-app) se actualiza para que use otro archivo de definición de JSON, `newTestDefinition.json`, y se agregan dos etiquetas de recursos, `testTag1` y `testTag2`, con valores de descripción.

```azurecli-interactive

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "newTestDefinition.json" --tags "testTag1=testTagValue1" "testTag2=testTagValue"

```

Cuando el flujo de trabajo se actualiza correctamente, la CLI muestra la definición del flujo de trabajo actualizada de la aplicación lógica. Si se produce algún error en la actualización, consulte la [lista de posibles errores](#errors).

## <a name="delete-logic-apps-from-cli"></a>Eliminación de aplicaciones lógicas desde la CLI

Desde la CLI de Azure se puede eliminar el flujo de trabajo de una aplicación lógica. Para ello, es preciso usar el comando [`az logic workflow delete`](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-delete).

El comando debe incluir los siguientes [parámetros requeridos](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-delete-required-parameters):

| Parámetro | Value | Descripción |
| --------- | ----- | ----------- |
| Nombre | `--name -n` | Nombre de la aplicación lógica. |
| Definición de un nombre de grupo de recursos | `-resource-group -g` | El grupo de recursos en que se encuentra la aplicación lógica. |

También puede incluir un [parámetro opcional](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-delete-optional-parameters) para omitir los mensajes de confirmación, `--yes -y`.

```azurecli

az logic workflow delete --name
                         --resource-group
                         [--yes]

```

La CLI le pide que confirme la eliminación de la aplicación lógica. Para omitir el mensaje de confirmación, use el parámetro opcional `--yes -y` con el comando.

```azurecli

Are you sure you want to perform this operation? (y/n):

```

Para confirmar la eliminación de una aplicación lógica [enumere las aplicaciones lógicas de la CLI](#list-logic-apps-in-cli) o vea las aplicaciones lógicas en Azure Portal.

### <a name="example---delete-logic-app"></a>Ejemplo: eliminación de una aplicación lógica

En este ejemplo, se elimina el [flujo de trabajo creado en una sección anterior](#example---create-logic-app).

```azurecli-interactive

az logic workflow delete --resource-group "testResourceGroup" --name "testLogicApp"

```

Después de responder al aviso de confirmación con `y`, se elimina la aplicación lógica.

## <a name="show-logic-apps-in-cli"></a>Visualización de aplicaciones lógicas en la CLI

Puede obtener un flujo de trabajo de aplicación lógica específico mediante el comando [`az logic workflow show`](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-show).

```azurecli

az logic workflow show --name
                       --resource-group

```

Este comando debe incluir los siguientes [parámetros requeridos](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-show-required-parameters)

| Parámetro | Value | Descripción |
| --------- | ----- | ----------- |
| Nombre | `--name -n` | Nombre de la aplicación lógica. |
| Nombre de grupo de recursos | `--resource-group -g` | Nombre del grupo de recursos en que se encuentra la aplicación lógica. |

### <a name="example---get-logic-app"></a>Ejemplo: obtener una aplicación lógica

En este ejemplo, la aplicación lógica `testLogicApp` del grupo de recursos `testResourceGroup` se devuelve con registros completos para la depuración.

```azurecli-interactive

az logic workflow show --resource-group "testResourceGroup" --name "testLogicApp" --debug

```

## <a name="list-logic-apps-in-cli"></a>Enumeración de aplicaciones lógicas en la CLI

Mediante el comando [`az logic workflow list`](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-list) puede enumerar sus aplicaciones lógicas por suscripción. Este comando devuelve el código JSON de los flujos de trabajo de sus aplicaciones lógicas.

Puede filtrar los resultados por los siguientes [parámetros opcionales](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-list-optional-parameters):

| Parámetro | Value | Descripción |
| --------- | ----- | ----------- |
| Nombre de grupo de recursos | `--resource-group -g` | Nombre del grupo de recursos por el que desea filtrar los resultados. |
| Número de elementos | `--top` | Número de elementos que se incluyen en los resultados. |
| Filtro | `--filter` | Tipo de filtro que se usa en la lista. Puede filtrar por estado (`State`), desencadenador (`Trigger`) y el identificador del recurso al que se hace referencia (`ReferencedResourceId`). |

```azurecli

az logic workflow list [--filter]
                       [--resource-group]
                       [--top]

```

### <a name="example---list-logic-apps"></a>Ejemplo: enumeración de aplicaciones lógicas

En este ejemplo, todos los flujos de trabajo habilitados del grupo de recursos `testResourceGroup` se devuelven en formato de tabla ASCII.

```azurecli-interactive

az logic workflow list --resource-group "testResourceGroup" --filter "(State eq 'Enabled')" --output "table"

```

## <a name="errors"></a>Errors

El siguiente error indica que la extensión de la CLI de Azure Logic Apps no está instalada. Siga los pasos de los requisitos previos para [instalar la extensión de Logic Apps](#prerequisites) en el equipo.

```output

az: 'logic' is not in the 'az' command group. See 'az --help'. If the command is from an extension, please make sure the corresponding extension is installed. To learn more about extensions, please visit https://docs.microsoft.com/cli/azure/azure-cli-extensions-overview

```

El siguiente error puede indicar que la ruta de acceso del archivo para cargar la definición del flujo de trabajo no es correcta.

```output

Expecting value: line 1 column 1 (char 0)

```

## <a name="global-parameters"></a>Parámetros globales

Puede usar los siguientes parámetros globales opcionales de la CLI de Azure con sus comandos de `az logic`:

| Parámetro | Value | Descripción |
| --------- | ----- | ----------- |
| Formato de salida | `--output -o` | Cambia el [formato de salida](/cli/azure/format-output-azure-cli?view=azure-cli-latest) para que no sea el JSON predeterminado. |
| Mostrar solo errores | `--only-show-errors` | Suprime las advertencias y muestra solo los errores. |
| Verbose | `--verbose` | Muestra registros detallados. |
| Depurar | `--debug` | Muestra todos los registros de depuración. |
| Mensaje de ayuda | `--help -h` | Muestra el cuadro de diálogo de ayuda. |
| Consultar | `--query` | Establece una cadena de consulta de JMESPath para la salida JSON. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure/?view=azure-cli-latest).

Para ver un ejemplo de cómo crear una aplicación lógica básica a través de la CLI, consulte el [script de ejemplo y la definición del flujo de trabajo](sample-logic-apps-cli-script.md).

Puede encontrar más ejemplos de scripts de la CLI de Logic Apps en el [explorador de ejemplos de código de Microsoft](/samples/browse/?products=azure-logic-apps).
