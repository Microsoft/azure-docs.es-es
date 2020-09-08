---
title: Qué es Azure Data Science Virtual Machine
titleSuffix: Azure Data Science Virtual Machine
description: 'Información general de Azure Data Science Virtual Machine: una máquina virtual fácil de usar en la plataforma de la nube de Azure, con herramientas y bibliotecas preinstaladas y configuradas para la práctica de la ciencia de datos.'
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: overview
ms.date: 04/02/2020
ms.openlocfilehash: 2bfcdfcb01e7908c199054e793d82cdfa1b726c7
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816344"
---
# <a name="what-is-the-azure-data-science-virtual-machine-for-linux-and-windows"></a>¿Qué es Azure Data Science Virtual Machine de Azure para Linux y Windows?

Data Science Virtual Machine (DSVM) es una imagen de máquina virtual personalizada en la plataforma en la nube de Azure diseñada específicamente para realizar ciencia de datos. Tiene muchas herramientas de ciencias de datos populares preinstaladas y preconfiguradas para impulsar la creación de aplicaciones inteligentes para análisis avanzado.

El DSVM está disponible en:

+ Windows Server 2019
+ Ubuntu 18.04 LTS

## <a name="comparison-with-azure-machine-learning"></a>Comparación con Azure Machine Learning

DSVM es una imagen de máquina virtual personalizada para la ciencia de datos, en tanto que [Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/overview-what-is-azure-ml) (AzureML) es una plataforma de un extremo a otro que abarca:

+ Proceso totalmente administrado
  + Compute Instances (Instancias de proceso)
  + Clústeres de proceso para tareas distribuidas de aprendizaje automático
  + Clústeres de inferencia para puntuaciones en tiempo real
+ Almacenes de datos (por ejemplo, Blob, ADLS Gen2, base de datos SQL)
+ Seguimiento de experimentos
+ Administración de modelos
+ Cuaderno
+ Entornos (administración de las dependencias de Conda y R)
+ Etiquetado
+ Canalizaciones (automatización de flujos de trabajo de ciencia de datos de un extremo a otro)

### <a name="comparison-with-azureml-compute-instances"></a>Comparación con instancias de proceso de AzureML

Las [instancias de Proceso de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-compute-instance) son una imagen de máquina virtual totalmente configurada y __administrada__, mientras que DSVM es una máquina virtual __no administrada__.

A continuación se detallan las diferencias principales entre estas dos ofertas de productos:


|Característica |Ciencia de datos<br>máquina virtual |AzureML<br>Instancia de proceso  | 
|---------|---------|---------|
| Totalmente administrada | No        | Sí        |
|Compatibilidad con idiomas     |  Python, R, Julia, SQL, C#,<br> Java, Node.js, F#       | Python y R        |
|Sistema operativo     | Ubuntu<br>Windows         |    Ubuntu     |
|Opción de GPU preconfigurada     |  Sí       |    Sí     |
|Opción de escalado vertical | Sí | Sí |
|Acceso SSH    | Sí        |    Sí     |
|Acceso RDP    | Sí        |     No    |
|Integrada<br>Notebooks hospedados     |   No<br>(requiere configuración adicional)      |      Sí   |
|SSO integrado     | No <br>(requiere configuración adicional)         |    Sí     |
|Colaboración integrada     | No         | Sí        |
|Herramientas preinstaladas     |  Jupyter (lab), RStudio Server, VSCode,<br> Visual Studio, PyCharm, Juno,<br>Power BI Desktop, SSMS, <br>Microsoft Office 365, Apache Drill       |     Jupyter(lab)<br> RStudio Server   |

## <a name="sample-use-cases"></a>Casos de uso de ejemplo

A continuación, se muestran algunos casos de uso comunes para los clientes de DSVM.

### <a name="short-term-experimentation-and-evaluation"></a>Evaluación y experimentación a corto plazo

Puede usar DSVM para evaluar o aprender sobre las nuevas [herramientas](./tools-included.md) de ciencia de datos, especialmente en algunos de nuestros [ejemplos y tutoriales](./dsvm-samples-and-walkthroughs.md) publicados.

### <a name="deep-learning-with-gpus"></a>Aprendizaje profundo con GPU

En Data Science Virtual Machine, los modelos de entrenamiento pueden usar algoritmos de aprendizaje profundo en hardware basado en unidades de procesamiento de gráficos (GPU). Aprovechando las funcionalidades de escalado de máquinas virtuales de la plataforma Azure, DSVM le ayuda a usar hardware basado en GPU en la nube según sus necesidades. Puede cambiar a una máquina virtual basada en GPU cuando vaya a entrenar modelos grandes o cuando necesite cálculos a alta velocidad y desee mantener el mismo disco del SO. Puede elegir cualquiera de las SKU de máquina virtual habilitada para GPU de la serie N con DSVM. Tenga en cuenta que las cuentas gratuitas de Azure no admiten las SKU de máquinas virtuales habilitadas para GPU.

Las ediciones de Windows de DSVM vienen preinstaladas con controladores de GPU, marcos y versiones de GPU de los marcos de aprendizaje profundo. En las ediciones de Linux, el aprendizaje profundo en las GPU está habilitado en las DSVM con Ubuntu. 

También puede implementar las ediciones de Ubuntu o Windows de DSVM en una máquina virtual de Azure no basada en las GPU. En este caso, todos los marcos de aprendizaje profundo se revertirán al modo CPU.

[Más información sobre los marcos de aprendizaje profundo y de inteligencia artificial](dsvm-tools-deep-learning-frameworks.md).

### <a name="data-science-training-and-education"></a>Educación y formación de ciencia de datos

Los instructores y educadores que imparten clases de ciencia de datos proporcionan normalmente una imagen de máquina virtual. La imagen garantiza que los alumnos tienen una configuración coherente y que los ejemplos funcionan según lo previsto.

Data Science Virtual Machine crea un entorno a petición con una configuración coherente que simplifica los desafíos de incompatibilidad y soporte técnico. En los casos en donde estos entornos tienen que crearse con frecuencia, especialmente para las clases de aprendizaje más cortas, estos entornos se benefician sustancialmente.


## <a name="whats-included-on-the-dsvm"></a>¿Qué incluye DSVM?

Consulte [aquí](tools-included.md) una lista completa de herramientas en las DSVM con Windows y Linux.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información en estos artículos:

+ Windows:
  + [Configuración de una instancia de DSVM de Windows](provision-vm.md)
  + [Ciencia de datos en una DSVM con Windows](vm-do-ten-things.md)

+ Linux:
  + [Configuración de una instancia de DSVM de Linux (Ubuntu)](dsvm-ubuntu-intro.md)
  + [Ciencia de datos en una DSVM de Linux](linux-dsvm-walkthrough.md)
