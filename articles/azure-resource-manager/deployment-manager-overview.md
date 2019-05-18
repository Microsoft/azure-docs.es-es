---
title: Prácticas de implementación segura entre regiones con Azure Deployment Manager
description: Describe cómo implementar un servicio en varias regiones con Azure Deployment Manager. Muestra las prácticas de implementación segura para comprobar la estabilidad de la implementación antes de proceder en todas las regiones.
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/13/2019
ms.author: tomfitz
ms.custom: seodec18
ms.openlocfilehash: d93d9999c407e64658b88025feda48d33e1a5ad1
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595790"
---
# <a name="enable-safe-deployment-practices-with-azure-deployment-manager-public-preview"></a>Habilite prácticas de desarrollo seguro con Azure Deployment Manager (versión preliminar)

Para implementar un servicio en varias regiones y asegurarse de que se ejecuta según lo previsto en todas ellas, puede usar Azure Deployment Manager para coordinar su lanzamiento por fases. Igual que haría en cualquier implementación de Azure, los recursos del servicio se definen en las [plantillas de Resource Manager](resource-group-authoring-templates.md). Después de crear las plantillas, use Deployment Manager para describir la topología del servicio y cómo se debe lanzar.

Deployment Manager es una característica de Resource Manager. Amplía sus funcionalidades durante la implementación. Use Deployment Manager cuando tenga un implementar un servicio complejo en varias regiones. Mediante el establecimiento de fases en el lanzamiento del servicio, puede encontrar posibles problemas antes de que se haya implementado en todas las regiones. Si no necesita las precauciones adicionales de un lanzamiento por fases, use las [opciones de implementación](resource-group-template-deploy-portal.md) estándar para Resource Manager. Deployment Manager se integra perfectamente con todas las herramientas de terceros existentes que admiten las implementaciones de Resource Manager, como las ofertas de integración continua y entrega continua (CI/CD). 

Azure Deployment Manager está en versión preliminar privada. Para usar Azure Deployment Manager, complete el [formulario de registro](https://aka.ms/admsignup). Sus [comentarios](https://aka.ms/admfeedback) pueden ayudarnos a mejorar la característica.

Para usar Deployment Manager, es preciso crear cuatro archivos:

* Plantilla de topología
* Plantilla de lanzamiento
* Archivo de parámetros para topología
* Archivo de parámetros para lanzamiento

La plantilla de topología se implementar antes que la de lanzamiento.

Puede encontrar la referencia de la API REST de Azure Deployment Manager [aquí](https://docs.microsoft.com/rest/api/deploymentmanager/).

## <a name="identity-and-access"></a>Identidad y acceso

Con Deployment Manager, una [identidad administrada asignada por el usuario](../active-directory/managed-identities-azure-resources/overview.md) realiza las acciones de implementación. Cree dicha identidad antes de iniciar la implementación. Debe tener acceso a la suscripción de Azure en la que va a implementar el servicio y debe tener permisos suficientes para completar dicha implementación. Para obtener información acerca de las acciones que se otorgan a través de los roles, consulte [Roles integrados en los recursos de Azure](../role-based-access-control/built-in-roles.md).

La identidad debe residir en la misma ubicación que la implementación.

## <a name="topology-template"></a>Plantilla de topología

La plantilla de topología describe los recursos de Azure que componen el servicio y dónde implementarlos. La siguiente imagen muestra la topología de un servicio de ejemplo:

![Jerarquía de topología del servicio a los servicios a las unidades de servicio](./media/deployment-manager-overview/service-topology.png)

La plantilla de topología incluye los siguientes recursos:

* Origen del artefacto: donde se almacenan los parámetros y las plantillas de Resource Manager
* Topología del servicio: apunta al origen del artefacto
  * Servicios: especifica la ubicación y el identificador de la suscripción de Azure
    * Unidades de servicio: especifica el grupo de recursos, el modo de implementación y ruta de acceso al archivo de plantillas y parámetros

Para saber lo que sucede en cada nivel, es útil ver los valores que proporciona.

![Valores de cada nivel](./media/deployment-manager-overview/topology-values.png)

### <a name="artifact-source-for-templates"></a>Origen del artefacto para plantillas

En la plantilla de la topología se crea un origen de artefacto que contiene los archivos de plantillas y de parámetros. El origen del artefacto es una forma de extraer los archivos para la implementación. En este mismo artículo verá otro origen de artefacto para archivos binarios.

En el siguiente ejemplo se muestra el formato general del origen del artefacto.

```json
{
    "type": "Microsoft.DeploymentManager/artifactSources",
    "name": "<artifact-source-name>",
    "location": "<artifact-source-location>",
    "apiVersion": "2018-09-01-preview",
    "properties": {
        "sourceType": "AzureStorage",
        "artifactRoot": "<root-folder-for-templates>",
        "authentication": {
            "type": "SAS",
            "properties": {
                "sasUri": "<SAS-URI-for-storage-container>"
            }
        }
    }
}
```

Para más información, consulte la [referencia de la plantilla artifactSources](/azure/templates/Microsoft.DeploymentManager/artifactSources).

### <a name="service-topology"></a>Topología del servicio

En el siguiente ejemplo se muestra el formato general del recurso de la topología del servicio. Especifique el identificador de recurso del origen de artefacto que contiene los archivos de plantillas y de parámetros. La topología del servicio incluye todos los recursos del servicio. Para asegurarse de que el origen del artefacto está disponible, la topología del servicio depende de él.

```json
{
    "type": "Microsoft.DeploymentManager/serviceTopologies",
    "name": "<topology-name>",
    "location": "<topology-location>",
    "apiVersion": "2018-09-01-preview",
    "properties": {
        "artifactSourceId": "<resource-ID-artifact-source>"
    },
    "dependsOn": [
        "<artifact-source>"
    ],
    "resources": [
        {
            "type": "services",
            ...
        }
    ]
}
```

Para más información, consulte la [referencia de la plantilla serviceTopologies](/azure/templates/Microsoft.DeploymentManager/serviceTopologies).

### <a name="services"></a>Servicios

En el siguiente ejemplo se muestra el formato general del recurso de los servicios. En cada servicio, especifique la ubicación y el identificador de la suscripción de Azure que se usan para implementar el servicio. Para realizar la implementación en varias regiones, defina un servicio para cada una de ellas. El servicio depende de la topología del servicio.

```json
{
    "type": "services",
    "name": "<service-name>",
    "location": "<service-location>",
    "apiVersion": "2018-09-01-preview",
    "dependsOn": [
        "<service-topology>"
    ],
    "properties": {
        "targetSubscriptionId": "<subscription-ID>",
        "targetLocation": "<location-of-deployed-service>"
    },
    "resources": [
        {
            "type": "serviceUnits",
            ...
        }
    ]
}
```

Para más información, consulte la [referencia de la plantilla de servicios](/azure/templates/Microsoft.DeploymentManager/serviceTopologies/services).

### <a name="service-units"></a>Unidades de servicio

En el siguiente ejemplo se muestra el formato general del recurso de las unidades de servicio. En cada unidad de servicio, especifique el grupo de recursos, el [modo de implementación](deployment-modes.md) que se usa para la implementación y la ruta de acceso al archivo de plantillas y parámetros. Si especifica una ruta de acceso relativa para la plantilla y los parámetros, la ruta de acceso completa se construye desde la carpeta raíz del origen de artefactos. Puede especificar una ruta de acceso absoluta para la plantilla y los parámetros, pero perderá la capacidad de controlar las versiones. La unidad servicio depende del servicio.

```json
{
    "type": "serviceUnits",
    "name": "<service-unit-name>",
    "location": "<service-unit-location>",
    "apiVersion": "2018-09-01-preview",
    "dependsOn": [
        "<service>"
    ],
    "tags": {
        "serviceType": "Service West US Web App"
    },
    "properties": {
        "targetResourceGroup": "<resource-group-name>",
        "deploymentMode": "Incremental",
        "artifacts": {
            "templateArtifactSourceRelativePath": "<relative-path-to-template>",
            "parametersArtifactSourceRelativePath": "<relative-path-to-parameter-file>"
        }
    }
}
```

Cada plantilla debe incluir los recursos relacionados que desea implementar en un paso. Por ejemplo, una unidad de servicio puede tener una plantilla que implemente todos los recursos del front-end de un servicio.

Para más información, consulte la [referencia de la plantilla serviceUnits](/azure/templates/Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits).

## <a name="rollout-template"></a>Plantilla de lanzamiento

La plantilla de lanzamiento describe los pasos que se deben seguir al implementar un servicio. Especifique la topología del servicio que va a usar y defina el orden de implementación de las unidades de servicio. Incluye un origen de artefacto para almacenar los archivos binarios de la implementación. En la plantilla de lanzamiento, se define la siguiente jerarquía:

* Origen de artefacto
* Paso
* Lanzamiento
  * Grupos de pasos
    * Operaciones de implementación

La siguiente imagen muestra la jerarquía de la plantilla de lanzamiento:

![Jerarquía del lanzamiento a los pasos](./media/deployment-manager-overview/Rollout.png)

Cada lanzamiento puede tener muchos grupos de pasos. Cada grupo de pasos tiene una operación de implementación que apunta a una unidad de servicio de la topología del servicio.

### <a name="artifact-source-for-binaries"></a>Origen de artefacto para binarios

En la plantilla de lanzamiento, cree un origen de artefacto para los archivos binarios que necesita implementar en el servicio. Este origen de artefacto es similar al [origen de artefacto para plantillas](#artifact-source-for-templates), salvo que contiene los scripts, las páginas web, el código compilado u otros archivos que necesita el servicio.

### <a name="steps"></a>Pasos

Puede definir que se realice un paso antes o después de la operación de implementación. Actualmente, solo el `wait` paso y el 'healthCheck' están disponibles. 

El paso de espera detiene la implementación antes de continuar. Permite comprobar que el servicio se ejecuta según lo esperado antes de implementar la siguiente unidad de servicio. En el siguiente ejemplo se muestra el formato general de un paso de espera.

```json
{
    "apiVersion": "2018-09-01-preview",
    "type": "Microsoft.DeploymentManager/steps",
    "name": "waitStep",
        "location": "<step-location>",
    "properties": {
        "stepType": "wait",
        "attributes": {
          "duration": "PT1M"
        }
    }
},
```

La propiedad duration usa el [estándar ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations). El ejemplo anterior especifica una espera un minuto.

Para obtener más información sobre el paso de comprobación de mantenimiento, consulte [introducir el lanzamiento de la integración de estado a Azure Deployment Manager](./deployment-manager-health-check.md) y [Tutorial: Usar comprobación de estado en Azure Deployment Manager](./deployment-manager-tutorial-health-check.md).

Para más información, consulte la [referencia de la plantilla de pasos](/azure/templates/Microsoft.DeploymentManager/steps).

### <a name="rollouts"></a>Lanzamientos

Para asegurarse de que el origen del artefacto está disponible, el lanzamiento depende de él. El lanzamiento define los grupos de pasos de cada unidad de servicio que se implementa. Puede definir las acciones que se realizan antes o después de la implementación. Por ejemplo, puede especificar que la implementación espere después de que la unidad de servicio se haya implementado. Puede definir el orden de los grupos de pasos.

El objeto de identidad especifica la [identidad administrada asignada por el usuario](#identity-and-access) que realiza las acciones de implementación.

En el siguiente ejemplo se muestra el formato general del lanzamiento.

```json
{
    "type": "Microsoft.DeploymentManager/rollouts",
    "name": "<rollout-name>",
    "location": "<rollout-location>",
    "apiVersion": "2018-09-01-preview",
    "Identity": {
        "type": "userAssigned",
        "identityIds": [
            "<managed-identity-ID>"
        ]
    },
    "dependsOn": [
        "<artifact-source>"
    ],
    "properties": {
        "buildVersion": "1.0.0.0",
        "artifactSourceId": "<artifact-source-ID>",
        "targetServiceTopologyId": "<service-topology-ID>",
        "stepGroups": [
            {
                "name": "stepGroup1",
                "dependsOnStepGroups": ["<step-group-name>"],
                "preDeploymentSteps": ["<step-ID>"],
                "deploymentTargetId":
                    "<service-unit-ID>",
                "postDeploymentSteps": ["<step-ID>"]
            },
            ...
        ]
    }
}
```

Para más información, consulte la [referencia de la plantilla de lanzamientos](/azure/templates/Microsoft.DeploymentManager/rollouts).

## <a name="parameter-file"></a>Archivo de parámetros

Cree dos archivos de parámetros. Uno de los archivos de parámetros se utiliza al implementar la topología del servicio, mientras que el otro se usa para la implementación del lanzamiento. Hay algunos valores que es preciso estar seguro de que son iguales en ambos archivos de parámetros.  

## <a name="containerroot-variable"></a>Variable containerRoot

Con las implementaciones con control de versiones, la ruta de acceso a los artefactos cambia con cada nueva versión. La primera vez que se ejecuta una implementación la ruta de acceso puede ser `https://<base-uri-blob-container>/binaries/1.0.0.0`. La segunda vez puede ser `https://<base-uri-blob-container>/binaries/1.0.0.1`. Deployment Manager simplifica la obtención de la ruta de acceso raíz correcta para la implementación actual mediante el uso de la variable `$containerRoot`. Este valor cambia con cada versión y no se conoce antes de la implementación.

Use la variable `$containerRoot` en el archivo de parámetros de la plantilla para implementar los recursos de Azure. En el momento de la implementación, esta variable se reemplaza por los valores reales del lanzamiento. 

Por ejemplo, durante el lanzamiento crea un origen de artefacto para los artefactos de binarios.

```json
{
    "type": "Microsoft.DeploymentManager/artifactSources",
    "name": "[variables('rolloutArtifactSource').name]",
    "location": "[parameters('azureResourceLocation')]",
    "apiVersion": "2018-09-01-preview",
    "properties": {
        "sourceType": "AzureStorage",
        "artifactRoot": "[parameters('binaryArtifactRoot')]",
        "authentication" :
        {
            "type": "SAS",
            "properties": {
                "sasUri": "[parameters('artifactSourceSASLocation')]"
            }
        }
    }
},
```

Observe las propiedades `artifactRoot` y `sasUri`. Para la raíz del artefacto se puede seleccionar un valor como `binaries/1.0.0.0`. El identificador URI de SAS es el identificador URI para el contenedor de almacenamiento con un token de SAS para el acceso. Deployment Manager construye automáticamente el valor de la variable `$containerRoot`. Combina estos valores con el formato `<container>/<artifactRoot>`.

La plantilla y el archivo de parámetros necesitan conocer la ruta de acceso correcta para obtener los archivos binarios con control de versiones. Por ejemplo, para implementar archivos para una aplicación web, cree el siguiente archivo de parámetros con la variable $containerRoot. Debe usar dos barras diagonales inversas (`\\`) en la ruta de acceso, ya que la primera es un carácter de escape.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "deployPackageUri": {
            "value": "$containerRoot\\helloWorldWebAppWUS.zip"
        }
    }
}
```

Luego, use dicho parámetro en la plantilla:

```json
{
    "name": "MSDeploy",
    "type": "extensions",
    "location": "[parameters('location')]",
    "apiVersion": "2015-08-01",
    "dependsOn": [
        "[concat('Microsoft.Web/sites/', parameters('WebAppName'))]"
    ],
    "tags": {
        "displayName": "WebAppMSDeploy"
    },
    "properties": {
        "packageUri": "[parameters('deployPackageURI')]"
    }
}
```

Para administrar las implementaciones con control de versiones, cree carpetas nuevas y use esa raíz durante el lanzamiento. La ruta de acceso llega hasta la plantilla que implementa los recursos.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información acerca de Deployment Manager. Pase al siguiente artículo, donde aprenderá a realizar implementaciones con Deployment Manager.

> [!div class="nextstepaction"]
> [Tutorial: Uso de Azure Deployment Manager con plantillas de Resource Manager](./deployment-manager-tutorial.md)