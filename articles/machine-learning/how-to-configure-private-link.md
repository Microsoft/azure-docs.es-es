---
title: Configuración de un punto de conexión privado (versión preliminar)
titleSuffix: Azure Machine Learning
description: Use Azure Private Link para acceder de forma segura al área de trabajo de Azure Machine Learning desde una red virtual.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 07/28/2020
ms.openlocfilehash: bdb7ba30d9fa2d0bd1eff9368d6e30e516b53895
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192727"
---
# <a name="configure-azure-private-link-for-an-azure-machine-learning-workspace-preview"></a>Configuración de Azure Private Link para un área de trabajo de Azure Machine Learning (versión preliminar)

En este documento, aprenderá a usar Azure Private Link con el área de trabajo de Azure Machine Learning. 

> [!IMPORTANT]
> El uso de Azure Private Link con áreas de trabajo de Azure Machine Learning se encuentra actualmente en versión preliminar pública. Esta funcionalidad solo está disponible en las regiones **Este de EE. UU.** y **Oeste de EE. UU. 2**. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Private Link le permite conectarse a su área de trabajo mediante un punto de conexión privado. El punto de conexión privado es un conjunto de direcciones IP privadas dentro de la red virtual. Después, puede limitar el acceso al área de trabajo para que solo se produzca en las direcciones IP privadas. Private Link ayuda a reducir el riesgo de una filtración de datos. Para más información sobre los puntos de conexión privados, consulte el artículo [Azure Private Link](/azure/private-link/private-link-overview).

> [!IMPORTANT]
> Azure Private Link no afecta al plano de control de Azure (operaciones de administración), como la eliminación del área de trabajo o la administración de los recursos de proceso. Por ejemplo, crear, actualizar o eliminar un destino de proceso. Estas operaciones se realizan sobre la red pública de Internet como de costumbre.
>
> No se admite la vista previa de instancias de proceso de Azure Machine Learning en un área de trabajo en la que Private Link está habilitado.
>
> Si usa Mozilla Firefox, puede encontrar problemas al intentar acceder al punto de conexión privado del área de trabajo. Este problema puede estar relacionado con DNS a través de HTTPS en Mozilla. Como solución alternativa, se recomienda usar Microsoft Edge de Google Chrome.

## <a name="create-a-workspace-that-uses-a-private-endpoint"></a>Creación de un área de trabajo que usa un punto de conexión privado

> [!IMPORTANT]
> Actualmente, solo se admite la habilitación de un punto de conexión privado al crear una nueva área de trabajo de Azure Machine Learning.

Puede usar la plantilla que se encuentra en [https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-advanced](https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-advanced) para crear un área de trabajo con un punto de conexión privado.

Para información sobre el uso de esta plantilla, que incluye los puntos de conexión privados, consulte [Uso de una plantilla de Azure Resource Manager para crear un área de trabajo para Azure Machine Learning](how-to-create-workspace-template.md).

## <a name="using-a-workspace-over-a-private-endpoint"></a>Uso de un área de trabajo mediante un punto de conexión privado

Dado que la comunicación con el área de trabajo solo se permite desde la red virtual, cualquier entorno de desarrollo que use el área de trabajo debe ser miembro de la red virtual. Por ejemplo, una máquina virtual en la red virtual.

> [!IMPORTANT]
> Para evitar la interrupción temporal de la conectividad, Microsoft recomienda vaciar la memoria caché de DNS en las máquinas que se conectan al área de trabajo después de habilitar Private Link. 

Para más información sobre Azure Virtual Machines, consulte [Documentación sobre Virtual Machines](/azure/virtual-machines/).


## <a name="using-azure-storage"></a>Uso de Azure Storage

Para proteger la cuenta de Azure Storage usada por el área de trabajo, colóquela dentro de la red virtual.

Para obtener información sobre cómo colocar la cuenta de almacenamiento en la red virtual, consulte [Uso de una cuenta de almacenamiento para el área de trabajo](how-to-enable-virtual-network.md#use-a-storage-account-for-your-workspace).

> [!WARNING]
> Azure Machine Learning no admite el uso de una cuenta de Azure Storage que tenga habilitado Private Link.

## <a name="using-azure-key-vault"></a>Uso de Azure Key Vault

Para proteger la instancia de Azure Key Vault que usa el área de trabajo, puede colocarla dentro de la red virtual o habilitar Private Link para ella.

Para obtener información sobre cómo colocar la instancia de Key Vault en la red virtual, consulte [Uso de una instancia de Key Vault con el área de trabajo](how-to-enable-virtual-network.md#key-vault-instance).

Para obtener información sobre cómo habilitar Private Link para el almacén de claves, consulte [Integración de Key Vault con Azure Private Link](/azure/key-vault/private-link-service).

## <a name="using-azure-kubernetes-services"></a>Uso de Azure Kubernetes Service

Para proteger la instancia de Azure Kubernetes Service usada por el área de trabajo, colóquela dentro de la red virtual. Para más información, consulte [Uso de Azure Kubernetes Service con el área de trabajo](how-to-enable-virtual-network.md#aksvnet).

Azure Machine Learning ya admite el uso de un servicio Azure Kubernetes Service que tenga habilitado Private Link.
Para crear un clúster de AKS privado, siga los documentos que se muestran [aquí](https://docs.microsoft.com/azure/aks/private-clusters).

## <a name="azure-container-registry"></a>Azure Container Registry

Para obtener información sobre cómo proteger Azure Container Registry dentro de la red virtual, consulte [Uso de Azure Container Registry](how-to-enable-virtual-network.md#azure-container-registry).

> [!IMPORTANT]
> Si usa Private Link para el área de trabajo de Azure Machine Learning y coloca la instancia de Azure Container Registry del área de trabajo en una red virtual, también debe aplicar la siguiente plantilla de Azure Resource Manager. Esta plantilla permite que el área de trabajo se comunique con ACR mediante Private Link.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "keyVaultArmId": {
      "type": "string"
      },
      "workspaceName": {
      "type": "string"
      },
      "containerRegistryArmId": {
      "type": "string"
      },
      "applicationInsightsArmId": {
      "type": "string"
      },
      "storageAccountArmId": {
      "type": "string"
      },
      "location": {
      "type": "string"
      }
  },
  "resources": [
      {
      "type": "Microsoft.MachineLearningServices/workspaces",
      "apiVersion": "2019-11-01",
      "name": "[parameters('workspaceName')]",
      "location": "[parameters('location')]",
      "identity": {
          "type": "SystemAssigned"
      },
      "sku": {
          "tier": "enterprise",
          "name": "enterprise"
      },
      "properties": {
          "sharedPrivateLinkResources":
  [{"Name":"Acr","Properties":{"PrivateLinkResourceId":"[concat(parameters('containerRegistryArmId'), '/privateLinkResources/registry')]","GroupId":"registry","RequestMessage":"Approve","Status":"Pending"}}],
          "keyVault": "[parameters('keyVaultArmId')]",
          "containerRegistry": "[parameters('containerRegistryArmId')]",
          "applicationInsights": "[parameters('applicationInsightsArmId')]",
          "storageAccount": "[parameters('storageAccountArmId')]"
      }
      }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo proteger el área de trabajo de Azure Machine Learning, consulte el artículo [Seguridad de la empresa](concept-enterprise-security.md).