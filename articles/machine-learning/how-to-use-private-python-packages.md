---
title: Uso de paquetes privados de Python
titleSuffix: Azure Machine Learning
description: Obtenga acceso a los paquetes privados de Python de forma segura desde entornos de Azure Machine Learning.
services: machine-learning
author: rastala
ms.author: roastala
ms.reviewer: laobri
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 07/10/2020
ms.openlocfilehash: 0f6f5d0ca757b10a16b31864124f1bcf1190674a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90896928"
---
# <a name="use-private-python-packages-with-azure-machine-learning"></a>Uso de paquetes privados de Python con Azure Machine Learning


En este artículo, aprenderá a usar paquetes privados de Python de forma segura dentro de Azure Machine Learning. Los casos de uso para los paquetes privados de Python incluyen:

 * Desarrolló un paquete privado que no quiere compartir públicamente.
 * Quiere usar un repositorio protegido de paquetes que están almacenados en un firewall empresarial.

El enfoque recomendado depende de si tiene pocos paquetes para una sola área de trabajo de Azure Machine Learning o un repositorio completo de paquetes para todas las áreas de trabajo de una organización.

Los paquetes privados se usan a través de la clase [Environment](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.environment). Dentro de un entorno, se declaran los paquetes de Python que se van a usar, incluidos los privados. Para obtener información sobre entornos en Azure Machine Learning en general, consulte [Uso de entornos](how-to-use-environments.md). 

## <a name="prerequisites"></a>Requisitos previos

 * El [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true)
 * Un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="use-small-number-of-packages-for-development-and-testing"></a>Uso de un número pequeño de paquetes para desarrollo y pruebas

Para usar un número pequeño de paquetes privados para una sola área de trabajo, use el método estático [`Environment.add_private_pip_wheel()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.environment?view=azure-ml-py#&preserve-view=trueadd-private-pip-wheel-workspace--file-path--exist-ok-false-). Este enfoque permite agregar rápidamente un paquete privado al área de trabajo y resulta adecuado para fines de desarrollo y pruebas.

Apunte el argumento de ruta de acceso de archivo a un archivo wheel local y ejecute el comando ```add_private_pip_wheel```. El comando devuelve una dirección URL que se usa para realizar el seguimiento de la ubicación del paquete en el área de trabajo. Capture la dirección URL de almacenamiento y pásela al método `add_pip_package()`.

```python
whl_url = Environment.add_private_pip_wheel(workspace=ws,file_path = "my-custom.whl")
myenv = Environment(name="myenv")
conda_dep = CondaDependencies()
conda_dep.add_pip_package(whl_url)
myenv.python.conda_dependencies=conda_dep
```

De forma interna, Azure Machine Learning Service reemplaza la dirección URL por una dirección URL de SAS segura, por lo que el archivo wheel se mantiene privado y seguro.

## <a name="use-a-repository-of-packages-from-azure-devops-feed"></a>Uso de un repositorio de paquetes desde la fuente de Azure DevOps

Si está desarrollando activamente paquetes de Python para su aplicación de aprendizaje automático, puede hospedarlos en un repositorio de Azure DevOps como artefactos y publicarlos como una fuente. Este enfoque permite integrar el flujo de trabajo de DevOps para compilar paquetes con el área de trabajo de Azure Machine Learning. Para obtener información sobre cómo configurar fuentes de Python con Azure DevOps, consulte [Introducción a los paquetes de Python en Azure Artifacts](https://docs.microsoft.com/azure/devops/artifacts/quickstarts/python-packages?view=azure-devops)

Este enfoque usa el token de acceso personal para la autenticación en el repositorio. El mismo enfoque se aplica a otros repositorios con autenticación basada en tokens, como los repositorios privados de GitHub. 

 1. [Cree un token de acceso personal (PAT)](https://docs.microsoft.com/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page#create-a-pat) para su instancia de Azure DevOps. Establezca el ámbito del token en __Empaquetado > Lectura__. 

 2. Agregue la dirección URL y PAT de Azure DevOps como propiedades del área de trabajo con el método [Workspace.set_connection](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py#&preserve-view=trueset-connection-name--category--target--authtype--value-).

     ```python
    from azureml.core import Workspace
    
    pat_token = input("Enter secret token")
    ws = Workspace.from_config()
    ws.set_connection(name="connection-1", 
        category = "PythonFeed",
        target = "https://<my-org>.pkgs.visualstudio.com", 
        authType = "PAT", 
        value = pat_token) 
     ```

 3. Cree un entorno de Azure Machine Learning y agregue paquetes de Python desde la fuente.
    
    ```python
    from azureml.core import Environment
    from azureml.core.conda_dependencies import CondaDependencies
    
    env = Environment(name="my-env")
    cd = CondaDependencies()
    cd.add_pip_package("<my-package>")
    cd.set_pip_option("--extra-index-url https://<my-org>.pkgs.visualstudio.com/<my-project>/_packaging/<my-feed>/pypi/simple")
    env.python.conda_dependencies=cd
    ```

El entorno ya está listo para usarse en las ejecuciones de entrenamiento o en las implementaciones de punto de conexión de servicio web. Al compilar el entorno, Azure Machine Learning Service usa PAT para autenticarse en la fuente con la dirección URL base coincidente.

## <a name="use-a-repository-of-packages-from-private-storage"></a>Uso de un repositorio de paquetes desde un almacenamiento privado

Puede consumir paquetes de una cuenta de Azure Storage en el firewall de su organización. La cuenta de almacenamiento puede contener un conjunto de paquetes protegido o un reflejo interno de paquetes disponibles públicamente.

Para configurar este tipo de almacenamiento privado, consulte [Protección de un área de trabajo de Azure Machine Learning y los recursos asociados](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts). Además, tiene que [colocar Azure Container Registry (ACR) detrás de la red virtual](how-to-secure-workspace-vnet.md#enable-azure-container-registry-acr).

> [!IMPORTANT]
> Debe completar este paso para poder entrenar o implementar modelos mediante el repositorio de paquetes privados.

Después de completar estas configuraciones, puede hacer referencia a los paquetes en la definición del entorno de Azure Machine Learning a través de su dirección URL completa en Azure Blob Storage.

## <a name="next-steps"></a>Pasos siguientes

 * Más información sobre [seguridad de empresa en Azure Machine Learning](concept-enterprise-security.md)
