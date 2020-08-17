---
title: Implementación de OKD en Azure
description: Implementación de OKD en Azure.
author: haroldwongms
manager: joraio
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/15/2019
ms.author: haroldw
ms.openlocfilehash: a7524ea80906f9df5f63af9e7d672bbef500b0d9
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373341"
---
# <a name="deploy-okd-in-azure"></a>Implementación de OKD en Azure

Existen dos maneras de implementar OKD (antes OpenShift Origin) en Azure:

- Puede implementar manualmente todos los componentes de la infraestructura de Azure necesarios y seguir después la [documentación](https://docs.okd.io) de OKD.
- También se puede usar una [plantilla de Resource Manager](https://github.com/Microsoft/openshift-origin) existente, lo que simplifica la implementación del clúster de OKD.

## <a name="deploy-using-the-okd-template"></a>Implementación mediante la plantilla de OKD

Para realizar la implementación mediante la plantilla de Resource Manager, se usa un archivo de parámetros para proporcionar los parámetros de entrada. Para personalizar aún más la implementación, puede bifurcar el repositorio de GitHub y cambiar los elementos correspondientes.

Algunas opciones de personalización comunes son, entre otras:

- Tamaño de máquina virtual de bastión (variable en azuredeploy.json)
- Convenciones de nomenclatura (variables en azuredeploy.json)
- Especificaciones del clúster de OpenShift, modificadas mediante el archivo de hosts (deployOpenShift.sh)

La [plantilla de OKD](https://github.com/Microsoft/openshift-origin) dispone de varias ramas para las diferentes versiones de OKD.  En función de sus necesidades, puede realizar la implementación directamente desde el repositorio o puede bifurcar el repositorio y realizar cambios personalizados antes de proceder con la implementación.

Use el valor `appId` de la entidad de servicio que creó anteriormente para el parámetro `aadClientId`.

A continuación, se muestra un archivo de parámetros de ejemplo llamado azuredeploy.parameters.json con todas las entradas necesarias.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "masterVmSize": {
            "value": "Standard_E2s_v3"
        },
        "infraVmSize": {
            "value": "Standard_E2s_v3"
        },
        "nodeVmSize": {
            "value": "Standard_E2s_v3"
        },
        "storageKind": {
            "value": "managed"
        },
        "openshiftClusterPrefix": {
            "value": "mycluster"
        },
        "masterInstanceCount": {
            "value": 3
        },
        "infraInstanceCount": {
            "value": 2
        },
        "nodeInstanceCount": {
            "value": 2
        },
        "dataDiskSize": {
            "value": 128
        },
        "adminUsername": {
            "value": "clusteradmin"
        },
        "openshiftPassword": {
            "value": "{Strong Password}"
        },
        "sshPublicKey": {
            "value": "{SSH Public Key}"
        },
        "enableMetrics": {
            "value": "true"
        },
        "enableLogging": {
            "value": "false"
        },
        "keyVaultResourceGroup": {
            "value": "keyvaultrg"
        },
        "keyVaultName": {
            "value": "keyvault"
        },
        "keyVaultSecret": {
            "value": "keysecret"
        },
        "enableAzure": {
            "value": "true"
        },
        "aadClientId": {
            "value": "11111111-abcd-1234-efgh-111111111111"
        },
        "aadClientSecret": {
            "value": "{Strong Password}"
        },
        "defaultSubDomainType": {
            "value": "nipio"
        }
    }
}
```

Reemplace los parámetros por su información específica.

Los parámetros pueden ser diferentes en las distintas versiones, así que compruebe los parámetros correspondientes a la rama utilizada.

### <a name="deploy-using-azure-cli"></a>Implementación con la CLI de Azure


> [!NOTE] 
> El comando siguiente requiere la CLI de Azure 2.0.8 o una versión posterior. Puede comprobar la versión de la CLI con el comando `az --version`. Para actualizar la versión de la CLI, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

En el ejemplo siguiente, el clúster de OKD y todos los recursos relacionados se implementan en un grupo de recursos llamado openshiftrg con el nombre de implementación myOpenShiftCluster. El repositorio de GitHub contiene una referencia directa a la plantilla y utiliza un archivo de parámetros local llamado azuredeploy.parameters.json.

```azurecli 
az group deployment create -g openshiftrg --name myOpenShiftCluster \
      --template-uri https://raw.githubusercontent.com/Microsoft/openshift-origin/master/azuredeploy.json \
      --parameters @./azuredeploy.parameters.json
```

La implementación tarda como mínimo 30 minutos en completarse, en función del número total de nodos implementados. La dirección URL de la consola de OpenShift y el nombre DNS del maestro de OpenShift se imprime en el terminal cuando concluye la implementación. También puede consultar la sección de salida de la implementación en Azure Portal.

```json
{
  "OpenShift Console Url": "http://openshiftlb.cloudapp.azure.com/console",
  "OpenShift Master SSH": "ssh -p 2200 clusteradmin@myopenshiftmaster.cloudapp.azure.com"
}
```

Si no desea que la línea de comandos espere hasta que finalice la implementación, agregue `--no-wait` como una de las opciones para la implementación del grupo. La salida de la implementación se puede recuperar en Azure Portal, en la sección de implementación del grupo de recursos.

## <a name="connect-to-the-okd-cluster"></a>Conexión al clúster de OKD

Cuando finalice la implementación, conéctese a la consola de OpenShift con el explorador mediante `OpenShift Console Url`. También puede acceder mediante SSH al maestro de OKD. En el ejemplo siguiente, se utiliza la salida de la implementación:

```bash
$ ssh -p 2200 clusteradmin@myopenshiftmaster.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, el clúster de OpenShift y todos los recursos relacionados.

```azurecli 
az group delete --name openshiftrg
```

## <a name="next-steps"></a>Pasos siguientes

- [Tareas posteriores a la implementación](./openshift-container-platform-3x-post-deployment.md)
- [Solución de problemas de implementación de OpenShift](./openshift-container-platform-3x-troubleshooting.md)
- [Introducción a OKD](https://docs.okd.io)
