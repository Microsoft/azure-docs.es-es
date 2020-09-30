---
title: Grupos compartidos
titleSuffix: Azure Data Science Virtual Machine
description: Aprenda a crear e implementar un grupo compartido de instancias de Data Science Virtual Machine (DSVM) como recurso compartido para un equipo.
keywords: aprendizaje profundo, AI, herramientas de ciencia de datos, data science virtual machine, análisis geoespacial, proceso de ciencia de datos en equipo
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: conceptual
ms.date: 12/10/2018
ms.openlocfilehash: 46572853cf882df4d6909297dd058d5ccaf782e1
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89144777"
---
# <a name="create-a-shared-pool-of-data-science-virtual-machines"></a>Crear un grupo compartido de máquinas virtuales de Data Science Virtual Machine

En este artículo, aprenderá a crear un grupo compartido de instancias de Data Science Virtual Machine (DSVM) para un equipo. Las ventajas de usar un grupo compartido son mejor uso de los recursos, colaboración y el uso compartido más fáciles y administración más eficaz de los recursos de DSVM.

Puede utilizar muchos métodos y tecnologías para crear un grupo de DSVM. Este artículo se centra en los grupos de máquinas virtuales (VM) interactivas. Una infraestructura de proceso administrado alternativa es Proceso de Azure Machine Learning. Para más información, consulte [Creación de destinos de proceso con el SDK para Python](../how-to-create-attach-compute-sdk.md).

## <a name="interactive-vm-pool"></a>Grupo de máquinas virtuales interactivas

Un grupo de máquinas virtuales interactivas que comparte todo el equipo de AI o ciencia de datos permite a los usuarios iniciar sesión en una instancia disponible de DSVM en lugar de tener una instancia específica para cada conjunto de usuarios. Esta configuración permite una mejor disponibilidad y un uso de recursos más efectivo.

Para crear un grupo de máquinas virtuales interactivo, usará [conjuntos de escalado de máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/). Los conjuntos de escalado se pueden usar para crear y administrar un grupo de máquinas virtuales idénticas, con equilibrio de carga y escalado automático.

El usuario inicia sesión en la dirección IP o DNS del grupo principal. El conjunto de escalado enruta automáticamente la sesión a una de sus máquinas virtuales DSVM. Dado que los usuarios quieren un entorno coherente y familiar con independencia de la máquina virtual donde inicien sesión, todas las instancias de la máquina virtual del conjunto de escalado montan una unidad de red compartida, como un recurso compartido de Azure Files o del Sistema de archivos de red (NFS). El área de trabajo compartida del usuario se suele conservar en el almacén de archivos compartidos montado en todas las instancias.

En [GitHub](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.json) se puede encontrar una plantilla de Azure Resource Manager de ejemplo que crea un conjunto de escalado con instancias de DSVM de Ubuntu. En la misma ubicación encontrará un ejemplo del [archivo de parámetros](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.parameters.json) para la plantilla de Azure Resource Manager.

Para crear el conjunto de escalado a partir de la plantilla de Azure Resource Manager, especifique los valores del archivo de parámetros en la CLI de Azure:

```azurecli-interactive
az group create --name [[NAME OF RESOURCE GROUP]] --location [[ Data center. For eg: "West US 2"]
az group deployment create --resource-group  [[NAME OF RESOURCE GROUP ABOVE]]  --template-uri https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.json --parameters @[[PARAMETER JSON FILE]]
```

Para los comandos anteriores se supone que tiene:

* Una copia del archivo de parámetros con los valores especificados para su instancia del conjunto de escalado.
* El número de instancias de VM.
* Punteros al recurso compartido de Azure Files.
* Credenciales de la cuenta de almacenamiento que se montará en cada máquina virtual.

Se hace referencia al archivo de parámetros localmente en los comandos. También puede pasar parámetros alineados o solicitarlos en el script.  

La plantilla anterior habilita el puerto SSH y JupyterHub del conjunto de escalado de front-end en el grupo de back-end de las máquinas virtuales DSVM de Ubuntu. Como usuario, inicie sesión en la máquina virtual en Secure Shell (SSH) o en JupyterHub como lo haría normalmente. Puesto que las instancias de máquina virtual se pueden escalar o reducir verticalmente de forma dinámica, es necesario guardar los estados en el recurso compartido montado de Azure Files. Puede usar el mismo enfoque para crear un grupo de máquinas virtuales de DSVM de Windows.

El [script que monta el recurso compartido de Azure Files](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Extensions/General/mountazurefiles.sh) también está disponible en el repositorio Azure DataScienceVM en Github. El script monta el recurso compartido de Azure Files en el punto de montaje especificado en el archivo de parámetros. El script también crea vínculos simbólicos a la unidad montada en el directorio principal del usuario inicial. Se establece un vínculo simbólico de un directorio de bloc de notas específico del usuario en el recurso compartido de archivos de Azure al directorio `$HOME/notebooks/remote` para que los usuarios pueden acceder a sus blocs de notas de Jupyter, así como ejecutarlos y guardarlos. La misma convención puede usarse al crear usuarios adicionales en la máquina virtual para dirigir el área de trabajo de Jupyter de cada usuario al recurso compartido de Azure Files.

Virtual Machine Scale Sets es compatible con el escalado automático. Puede establecer reglas sobre cuándo crear instancias adicionales y cuándo reducir verticalmente las instancias. Por ejemplo, puede reducir verticalmente las instancias a cero para ahorrar en costos de uso de hardware de nube si nunca se utilizan máquinas virtuales. Las páginas de documentación de Virtual Machine Scale Sets proporcionan pasos detallados para realizar el [escalado automático](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview).

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de una identidad común](dsvm-common-identity.md)
* [Almacenamiento seguro de las credenciales para acceder a recursos en la nube](dsvm-secure-access-keys.md)
