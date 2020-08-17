---
title: 'Ejemplo de script de la CLI de Azure: creación de una aplicación lógica'
description: Script de ejemplo para crear una aplicación lógica a través de la extensión Logic Apps en la CLI de Azure.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.custom: mvc
ms.date: 07/30/2020
ms.openlocfilehash: ceb4a3356ef78d2129f76bd11f555a9ca5206d51
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87505572"
---
# <a name="azure-cli-script-sample---create-a-logic-app"></a>Ejemplo de script de la CLI de Azure: creación de una aplicación lógica

Este script crea una aplicación lógica de ejemplo a través de la [extensión de Logic Apps de la CLI de Azure](/cli/azure/ext/logic/logic?view=azure-cli-latest) (`az logic`). Para obtener una guía detallada sobre la creación y administración de aplicaciones lógicas mediante la CLI de Azure, consulte el [inicio rápido de Logic Apps para la CLI de Azure](quickstart-logic-apps-azure-cli.md).

> [!WARNING]
> Actualmente, la extensión Logic Apps de la CLI de Azure se encuentra en fase *experimental* y *no está incluida en el soporte al cliente*. Use esta extensión de la CLI con precaución, sobre todo en entornos de producción.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* La [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) instalada en un equipo local.
* La [extensión Logic Apps de la CLI de Azure](/cli/azure/azure-cli-extensions-list?view=azure-cli-latest) instalada en el equipo. Para instalar esta extensión, use este comando: `az extension add --name logic`
* Una [definición de flujo de trabajo](quickstart-logic-apps-azure-cli.md#workflow-definition) para la aplicación lógica. Este archivo JSON debe seguir el [esquema del lenguaje de la definición del flujo de trabajo](logic-apps-workflow-definition-language.md).
* Una conexión de API a una cuenta de correo electrónico a través de un [conector de Logic Apps](../connectors/apis-list.md) compatible en el mismo grupo de recursos que la aplicación lógica. En este ejemplo se usa el conector de [Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md), pero también puede usar otros conectores como [Outlook.com](../connectors/connectors-create-api-outlook.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

Valide el entorno antes de empezar:

* Inicie sesión en Azure Portal y compruebe que la suscripción está activa, para lo que debe ejecutar `az login`.

* Compruebe la versión de la CLI de Azure en una ventana de terminal o de comandos, para lo que debe ejecutar `az --version`. Para saber cuál es la versión más reciente, consulte las [notas de la versión más reciente](/cli/azure/release-notes-azure-cli?tabs=azure-cli&view=azure-cli-latest).

  * Si no tiene la versión más reciente, actualice la instalación, para lo que debe seguir las instrucciones que encontrará en la [guía de instalación del sistema operativo o de la plataforma](/cli/azure/install-azure-cli?view=azure-cli-latest).

### <a name="sample-workflow-explanation"></a>Explicación del flujo de trabajo de ejemplo

En este archivo de definición del flujo de trabajo de ejemplo se crea la misma aplicación lógica básica que el [inicio rápido de Logic Apps para Azure Portal](quickstart-create-first-logic-app-workflow.md). 

Este flujo de trabajo de ejemplo: 

1. Especifica el esquema `$schema`, para la aplicación lógica.

1. Define un desencadenador para la aplicación lógica en la lista de desencadenadores, `triggers`. El desencadenador se repite (`recurrence`) cada 3 horas. Las acciones se desencadenan cuando se publica un nuevo elemento de fuente (`When_a_feed_item_is_published`) para la fuente RSS especificada (`feedUrl`).

1. Define una acción para la aplicación lógica en la lista de acciones, `actions`. La acción envía un correo electrónico (`Send_an_email_(V2)`) a través de Microsoft 365 con detalles de los elementos de fuente RSS, tal y como se especifica en la sección de cuerpo (`body`) de las entradas de la acción (`inputs`).

## <a name="sample-workflow-definition"></a>Definición del flujo de trabajo de ejemplo

Antes de ejecutar el script de ejemplo, primero debe crear una [definición de flujo de trabajo](#prerequisites) de ejemplo.

1. Cree un archivo JSON `testDefinition.json` en el equipo. 

1. Copie el siguiente contenido en el archivo JSON: 
    ```json
    
    {
        "definition": {
            "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
            "actions": {
                "Send_an_email_(V2)": {
                    "inputs": {
                        "body": {
                            "Body": "<p>@{triggerBody()?['publishDate']}<br>\n@{triggerBody()?['title']}<br>\n@{triggerBody()?['primaryLink']}</p>",
                            "Subject": "@triggerBody()?['title']",
                            "To": "test@example.com"
                        },
                        "host": {
                            "connection": {
                                "name": "@parameters('$connections')['office365']['connectionId']"
                            }
                        },
                        "method": "post",
                        "path": "/v2/Mail"
                    },
                    "runAfter": {},
                    "type": "ApiConnection"
                }
            },
            "contentVersion": "1.0.0.0",
            "outputs": {},
            "parameters": {
                "$connections": {
                    "defaultValue": {},
                    "type": "Object"
                }
            },
            "triggers": {
                "When_a_feed_item_is_published": {
                    "inputs": {
                        "host": {
                            "connection": {
                                "name": "@parameters('$connections')['rss']['connectionId']"
                            }
                        },
                        "method": "get",
                        "path": "/OnNewFeed",
                        "queries": {
                            "feedUrl": "https://www.pbs.org/now/rss.xml"
                        }
                    },
                    "recurrence": {
                        "frequency": "Hour",
                        "interval": 3
                    },
                    "splitOn": "@triggerBody()?['value']",
                    "type": "ApiConnection"
                }
            }
        },
        "parameters": {
            "$connections": {
                "value": {
                    "office365": {
                        "connectionId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testResourceGroup/providers/Microsoft.Web/connections/office365",
                        "connectionName": "office365",
                        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Web/locations/westus/managedApis/office365"
                    },
                    "rss": {
                        "connectionId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testResourceGroup/providers/Microsoft.Web/connections/rss",
                        "connectionName": "rss",
                        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Web/locations/westus/managedApis/rss"
                    }
                }
            }
        }
    }
    
    ```

1. Actualice los valores del marcador de posición con su propia información:

    1. Reemplace la dirección de correo electrónico del marcador de posición (`"To": "test@example.com"`). Debe usar una dirección de correo electrónico compatible con los conectores de Logic Apps. Para más información, consulte la sección los [requisitos previos](#prerequisites).

    1. Reemplace los detalles adicionales del conector si usa un conector de correo electrónico que no sea el conector de Office 365 Outlook.

    1. Reemplace los valores de suscripción del marcador de posición (`00000000-0000-0000-0000-000000000000`) para los identificadores de conexión (`connectionId` y `id`) en el parámetro de conexiones (`$connections`) con sus propios valores de suscripción.

1. Guarde los cambios.

## <a name="sample-script"></a>Script de ejemplo

> [!NOTE]
> Este ejemplo está escrito para el shell de `bash`. Si quiere ejecutar este ejemplo en otro shell, como Windows PowerShell o el símbolo del sistema, es posible que deba realizar modificaciones en el script.

Antes de ejecutar este script de ejemplo, ejecute este comando para conectarse a Azure:

```azurecli-interactive

az login

```

A continuación, navegue hasta el directorio en el que creó la definición del flujo de trabajo. Por ejemplo, si ha creado el archivo JSON de definición del flujo de trabajo en el escritorio:

```azurecli

cd ~/Desktop

```

A continuación, ejecute este script para crear una aplicación lógica. 

```azurecli-interactive

#!/bin/bash

# Create a resource group

az group create --name testResourceGroup --location westus

# Create your logic app

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "testDefinition.json"

```

### <a name="clean-up-deployment"></a>Limpieza de la implementación

Una vez que haya terminado de usar el script de ejemplo, ejecute el siguiente comando para quitar el grupo de recursos y todos sus recursos anidados, incluida la aplicación lógica.

```azurecli-interactive

az group delete --name testResourceGroup --yes

```

## <a name="script-explanation"></a>Explicación del script

Este script de ejemplo usa los siguientes comandos para crear un nuevo grupo de recursos y la aplicación lógica.

| Get-Help | Notas |
| ------- | ----- |
| [`az group create`](/cli/azure/group?view=azure-cli-latest#az-group-create) | Crea un grupo de recursos en el que se almacenan los recursos de la aplicación lógica. |
| [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-create) | Crea una aplicación lógica basada en el flujo de trabajo definido en el parámetro `--definition`. |
| [`az group delete`](/cli/azure/vm/extension?view=azure-cli-latest) | Elimina un grupo de recursos y todos sus recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure/?view=azure-cli-latest).

Puede encontrar más ejemplos de scripts de la CLI de Logic Apps en el [explorador de ejemplos de código de Microsoft](/samples/browse/?products=azure-logic-apps).
