---
title: Implementación de OpenShift en Azure Stack
description: Implemente OpenShift en Azure Stack.
author: haroldwongms
manager: joraio
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.openlocfilehash: 3729dbbc54d15d4641f5e514582f1437e790ef19
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374053"
---
# <a name="deploy-openshift-container-platform-or-okd-in-azure-stack"></a>Implementación de OpenShift Container Platform u OKD en Azure Stack

OpenShift pude implementarse en Azure Stack. Hay algunas diferencias clave entre Azure y Azure Stack, por lo que la implementación variará ligeramente, al igual que las funcionalidades.

Actualmente, el proveedor de soluciones en la nube de Azure no funciona en Azure Stack. Por este motivo, no podrá conectar discos para almacenamiento persistente en Azure Stack. En su lugar, puede configurar otras opciones de almacenamiento, como NFS, iSCSI, GlusterFS, etc. También puede habilitar CNS y usar GlusterFS para el almacenamiento persistente. Si CNS está habilitado, se implementarán tres nodos más con almacenamiento adicional para GlusterFS.

Puede usar uno de los diversos métodos para implementar OpenShift Container Platform u OKD en Azure Stack:

- Puede implementar manualmente los componentes necesarios de la infraestructura de Azure y seguir después la [documentación de OpenShift Container Platform](https://docs.openshift.com/container-platform) o la [documentación de OKD](https://docs.okd.io).
- También puede usar una plantilla existente de [Resource Manager](https://github.com/Microsoft/openshift-container-platform/) que simplifica la implementación del clúster de OpenShift Container Platform.
- También se puede usar una [plantilla de Resource Manager](https://github.com/Microsoft/openshift-origin) existente, lo que simplifica la implementación del clúster de OKD.

Si utiliza la plantilla de Resource Manager, seleccione la rama adecuada (azurestack-versión-3.x). Las plantillas de Azure no funcionarán, ya que las versiones de API son diferentes entre Azure y Azure Stack. Actualmente, la referencia de la imagen de RHEL está codificada de forma rígida como una variable en el archivo azuredeploy.json, por lo que deberá modificarse para que coincida con la imagen.

```json
"imageReference": {
    "publisher": "Redhat",
    "offer": "RHEL-OCP",
    "sku": "7-4",
    "version": "latest"
}
```

En todos los casos, se requiere una suscripción a Red Hat. Durante la implementación, la instancia de Red Hat Enterprise Linux está registrada en la suscripción de Red Hat y asociada al identificador de grupo que contiene los derechos para OpenShift Container Platform.
Asegúrese de que tiene un nombre de usuario, una contraseña y un identificador de grupo válidos para Red Hat Subscription Manager (RHSM). También puede usar una clave de activación, un identificador de organización y un identificador de grupo.  Puede comprobar esta información iniciando sesión en https://access.redhat.com.

## <a name="azure-stack-prerequisites"></a>Requisitos previos de Azure Stack

Para implementar un clúster de OpenShift, es necesario agregar una imagen de RHEL (OpenShift Container Platform) o una imagen de CentOS (OKD) al entorno de Azure Stack. Póngase en contacto con el administrador de Azure Stack para agregar estas imágenes. Si necesita instrucciones, las encontrará aquí:

- https://docs.microsoft.com/azure/azure-stack/azure-stack-add-vm-image
- https://docs.microsoft.com/azure/azure-stack/azure-stack-marketplace-azure-items
- https://docs.microsoft.com/azure/azure-stack/azure-stack-redhat-create-upload-vhd

## <a name="deploy-by-using-the-openshift-container-platform-or-okd-resource-manager-template"></a>Implementación con la plantilla de OpenShift Container Platform o de OKD Resource Manager

Para realizar la implementación mediante la plantilla de Resource Manager, se usa un archivo de parámetros para proporcionar los parámetros de entrada. Para personalizar aún más la implementación, puede bifurcar el repositorio de GitHub y cambiar los elementos correspondientes.

Algunas opciones de personalización comunes son, entre otras:

- Tamaño de máquina virtual de bastión (variable en azuredeploy.json)
- Convenciones de nomenclatura (variables en azuredeploy.json)
- Especificaciones del clúster de OpenShift, modificadas mediante el archivo de hosts (deployOpenShift.sh)
- Referencia de la imagen de RHEL (variable en azuredeploy.json)

Si desea seguir los pasos necesarios para realizar la implementación con la CLI de Azure, consulte la sección apropiada: [OpenShift Container Platform](./openshift-container-platform-3x.md) u [OKD](./openshift-okd.md).

## <a name="next-steps"></a>Pasos siguientes

- [Tareas posteriores a la implementación](./openshift-container-platform-3x-post-deployment.md)
- [Solución de problemas de implementación de OpenShift en Azure](./openshift-container-platform-3x-troubleshooting.md)