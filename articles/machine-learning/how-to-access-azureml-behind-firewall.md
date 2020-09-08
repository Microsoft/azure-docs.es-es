---
title: Uso de un firewall
titleSuffix: Azure Machine Learning
description: Controle el acceso a las áreas de trabajo de Azure Machine Learning con instancias de Azure Firewall. Obtenga información acerca de los hosts que debe permitir a través del firewall para que Azure Machine Learning funcione correctamente.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 07/17/2020
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 581feff516e0f0cd820c94290d4aaa729cc4d3a4
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88889947"
---
# <a name="use-workspace-behind-a-firewall-for-azure-machine-learning"></a>Uso de áreas de trabajo detrás de un firewall en Azure Machine Learning

En este artículo, aprenderá a configurar Azure Firewall para controlar el acceso al área de trabajo de Azure Machine Learning y a la red pública de Internet.   Para obtener más información sobre la protección de Azure Machine Learning, consulte [Seguridad de empresa para Azure Machine Learning](concept-enterprise-security.md).

Aunque la información de este documento se basa en el uso de [Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md), sirve también para otros productos de firewall. Si tiene alguna pregunta sobre cómo permitir la comunicación a través del firewall, consulte la documentación del firewall que usa.

## <a name="application-rules"></a>Reglas de aplicación

En el firewall, cree una _regla de aplicación_ que permita el tráfico entrante y saliente de las direcciones de este artículo.

> [!TIP]
> Al agregar la regla de red, establezca __Protocolo__ en cualquiera y los puertos en `*`.
>
> Para más información sobre la configuración de Azure Firewall, consulte [Implementación y configuración de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md#configure-an-application-rule).

## <a name="microsoft-hosts"></a>Hosts de Microsoft

Si no está configurado correctamente, el firewall puede provocar problemas al usar el área de trabajo. Hay una serie de nombres de host que se usan en el área de trabajo de Azure Machine Learning.

Los hosts de esta sección son propiedad de Microsoft y proporcionan servicios necesarios para que el área de trabajo funcione correctamente.

| **Nombre de host** | **Propósito** |
| ---- | ---- |
| **\*.batchai.core.windows.net** | Clústeres de entrenamiento |
| **ml.azure.com** | Azure Machine Learning Studio |
| **default.exp-tas.com** | Se usa por Azure Machine Learning Studio. |
| **\*.azureml.ms** | Usado por las API de Azure Machine Learning |
| **\*.experiments.azureml.net** | Usado por experimentos que se ejecutan en Azure Machine Learning |
| **\*.modelmanagement.azureml.net** | Usado para registrar e implementar modelos|
| **mlworkspace.azure.ai** | Usado por Azure Portal al visualizar un área de trabajo |
| **\*.aether.ms** | Usado al ejecutar canalizaciones de Azure Machine Learning |
| **\*.instances.azureml.net** | Instancias de proceso de Azure Machine Learning |
| **\*.instances.azureml.ms** | Instancias de proceso de Azure Machine Learning cuando el área de trabajo tiene habilitado Private Link. |
| **windows.net** | Azure Blob Storage |
| **vault.azure.net** | Azure Key Vault |
| **azurecr.io** | Azure Container Registry |
| **mcr.microsoft.com** | Microsoft Container Registry para imágenes de Docker base |
| **your-acr-server-name.azurecr.io** | Solo es necesario si Azure Container Registry está detrás de la red virtual. En esta configuración, se crea un vínculo privado desde el entorno de Microsoft hasta la instancia de ACR de la suscripción. Use el nombre del servidor de ACR para el área de trabajo de Azure Machine Learning. |
| **\*.notebooks.azure.net** | Se necesita en los cuadernos de Azure Machine Learning Studio. |
| **graph.windows.net** | Necesario para cuadernos |

## <a name="python-hosts"></a>Hosts de Python

Los hosts de esta sección se usan para instalar paquetes de Python. Son necesarios durante el desarrollo, el entrenamiento y la implementación. 

| **Nombre de host** | **Propósito** |
| ---- | ---- |
| **anaconda.com** | Se usa para instalar paquetes predeterminados. |
| **\*.anaconda.org** | Se usa para obtener datos del repositorio. |
| **pypi.org** | Se usa para enumerar las dependencias del índice predeterminado, si hay alguna, y el índice no se sobrescribe con la configuración del usuario. Si el índice se sobrescribe, también debe permitir **\*.pythonhosted.org**. |

## <a name="r-hosts"></a>Hosts de R

Los hosts de esta sección se usan para instalar paquetes de R. Son necesarios durante el desarrollo, el entrenamiento y la implementación.

> [!IMPORTANT]
> Internamente, el SDK de R para Azure Machine Learning usa paquetes de Python. Por lo tanto, también debe permitir los hosts de Python a través del firewall.

| **Nombre de host** | **Propósito** |
| ---- | ---- |
| **cloud.r-project.org** | Usado al instalar paquetes CRAN |

## <a name="azure-government-region"></a>Región de Azure Government

Direcciones URL necesarias para regiones Azure Government.

| **Nombre de host** | **Propósito** |
| ---- | ---- |
| **usgovarizona.api.ml.azure.us** | Región EE.UU.: Arizona |
| **usgovvirginia.api.ml.azure.us** | Región EE.UU.: Virginia |

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal](../firewall/tutorial-firewall-deploy-portal.md)
* [Protección de los trabajos de experimentación e inferencia de ML en una instancia de Azure Virtual Network](how-to-enable-virtual-network.md)
