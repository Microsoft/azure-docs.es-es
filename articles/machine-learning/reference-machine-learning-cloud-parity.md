---
title: Paridad entre regiones públicas y soberanas
titleSuffix: Azure Machine Learning
description: Algunas características de Azure Machine Learning, como las características en vista previa pública (gb), puede que solo estén disponibles en regiones de la nube pública. En este artículo se enumeran las características que también están disponibles en las regiones Azure Government, Azure Alemania y Azure China 21Vianet.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.reviewer: larryfr
ms.author: andzha
author: Anurzeuii
ms.date: 08/24/2020
ms.custom: references_regions
ms.openlocfilehash: 4a6f09fdff82b8e86c7fe75018c5267dba3c1b4a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90892984"
---
# <a name="azure-machine-learning-sovereign-cloud-parity"></a>Paridad de nube soberana de Azure Machine Learning

Obtenga información sobre qué características de Azure Machine Learning están disponibles en las regiones de nube soberanas. 

En la lista de regiones globales de Azure, hay varias regiones "soberanas" que sirven a mercados específicos. Por ejemplo, las regiones Azure Government y Azure China 21Vianet. Actualmente Azure Machine Learning se implementa en las siguientes regiones de nube soberanas:

* **EE. UU.: Arizona** y **EE. UU.: Virginia** de la región Azure Government.
* **Este de China 2** de la región Azure China 21Vianet.

> [!TIP]
> Para diferenciar entre regiones soberanas y no soberanas, este artículo usará el término __nube pública__ para hacer referencia a regiones no soberanas.

Nos esforzamos por proporcionar la máxima paridad entre nuestra nube pública y las regiones soberanas. Todas las características de Azure Machine Learning estarán disponibles en estas regiones dentro de **30 días de GA** (disponibilidad general) en nuestra nube pública. También se habilita un número seleccionado de características en vista previa (gb) en estas regiones. A continuación, se muestran las diferencias de paridad actuales entre nuestras nubes soberanas y públicas.

## <a name="azure-government"></a>Azure Government 

| Característica | Estado de la nube pública  | EE.UU.: Virginia | EE.UU.: Arizona| 
|----------------------------------------------------------------------------|:----------------------:|:--------------------:|:-------------:|
| **Aprendizaje automático automatizado** | | | |
| Creación y ejecución de experimentos en cuadernos                                    | Disponibilidad general                   | SÍ                | SÍ         |
| Creación y ejecución de experimentos en la experiencia web de Studio                        | Vista previa pública       | SÍ                | SÍ         |
| Capacidades de previsión de líderes del sector                                  | Disponibilidad general                   | SÍ                | SÍ         |
| Compatibilidad con aprendizaje profundo y otros aprendizajes avanzados                      | Disponibilidad general                   | SÍ                | SÍ         |
| Compatibilidad con datos de gran tamaño (hasta 100 GB)                                          | Vista previa pública       | SÍ                | SÍ         |
| Integración con Azure Databricks                                              | Disponibilidad general                   | No                 | No          |
| Integraciones con SQL, CosmosDB y HDInsight                                   | Disponibilidad general                   | SÍ                | SÍ         |
| **Canalizaciones de Machine Learning** |   |  | | 
| Creación, ejecución y publicación de canalizaciones con el SDK de Azure Machine Learning                   | Disponibilidad general                   | SÍ                | SÍ         |
| Creación de puntos de conexión de canalización con el SDK de Azure Machine Learning                           | Disponibilidad general                   | SÍ                | SÍ         |
| Creación, edición y eliminación de ejecuciones programadas de canalizaciones con el SDK de Azure Machine Learning | Disponibilidad general                   | SÍ*               | SÍ*        |
| Visualización de los detalles de la ejecución de la canalización en Studio                                        | Disponibilidad general                   | SÍ                | SÍ         |
| Creación, ejecución, visualización y publicación de canalizaciones en el diseñador de Azure Machine Learning          | Disponibilidad general      | SÍ                | SÍ         |
| Integración de Azure Databricks con canalización de Machine Learning                             | Disponibilidad general                   | No                 | No          |
| Creación de puntos de conexión de canalización en el diseñador de Azure Machine Learning                             | Disponibilidad general      | SÍ                | SÍ         |
| **Cuadernos integrados** |   |  | | 
| Uso compartido de archivos y cuadernos del área de trabajo                                        | Disponibilidad general                   | SÍ                | SÍ         |
| Compatibilidad con R y Python                                                       | Disponibilidad general                   | SÍ                | SÍ         |
| Compatibilidad con redes virtuales                                                    | Vista previa pública       | No                 | No          |
| **Instancia de proceso** |   |  | | 
| Instancias de proceso administradas para cuadernos integrados                         | Disponibilidad general                   | SÍ                | SÍ         |
| Jupyter, integración con JupyterLab                                            | Disponibilidad general                   | SÍ                | SÍ         |
| Compatibilidad con redes virtuales (VNet)                                             | Vista previa pública       | SÍ                | SÍ         |
| **Compatibilidad con SDK** |  |  | | 
| Compatibilidad con el SDK de R                                                              | Vista previa pública       | SÍ                | SÍ         |
| Compatibilidad con el SDK de Python                                                         | Disponibilidad general                   | SÍ                | SÍ         |
| **Seguridad** |   | | | 
| Compatibilidad con Virtual Network (VNet) para aprendizaje                                | Disponibilidad general                   | SÍ                | SÍ         |
| Compatibilidad con Virtual Network (VNet) para inferencia                               | Disponibilidad general                   | SÍ                | SÍ         |
| Autenticación de puntos de conexión de puntuación                                            | Vista previa pública       | SÍ                | SÍ         |
| Private Link de área de trabajo                                                     | Vista previa pública       | No                 | No          |
| ACI detrás de VNet                                                            | Vista previa pública       | No                 | No          |
| ACR detrás de VNet                                                            | Vista previa pública       | No                 | No          |
| IP privada de clúster de AKS                                                  | Vista previa pública       | No                 | No          |
| **Proceso** |   | | |
| Administración de cuotas entre áreas de trabajo                                         | Disponibilidad general                   | SÍ                | SÍ         |
| **Datos para el aprendizaje automático** |   | | |
| Creación, visualización o edición de conjuntos de datos y almacenes de datos del SDK                  | Disponibilidad general                   | SÍ                | SÍ         |
| Creación, visualización o edición de conjuntos de datos y almacenes de datos de la interfaz de usuario                   | Disponibilidad general                   | SÍ                | SÍ         |
| Visualización, edición o eliminación de monitores de desfase de conjuntos de datos del SDK                   | Vista previa pública       | SÍ                | SÍ         |
| Visualización, edición o eliminación de monitores de desfase de conjunto de datos de la interfaz de usuario                    | Vista previa pública       | SÍ                | SÍ         |
| **Ciclo de vida de Machine Learning** |   | | |
| Generación de perfiles de modelos                                                            | Disponibilidad general                   | SÍ                | PARTIAL     |
| La extensión Azure DevOps para Machine Learning y la CLI de Azure ML         | Disponibilidad general                   | SÍ                | SÍ         |
| Modelos acelerados mediante hardware basados en FPGA                                     | Disponibilidad general                   | No                 | No          |
| Integración con Visual Studio Code                                             | Vista previa pública       | No                 | No          |
| Integración con Event Grid                                                     | Vista previa pública       | No                 | No          |
| Integración de Azure Stream Analytics con Azure Machine Learning               | Vista previa pública       | No                 | No          |
| **Etiquetado** |   | | |
| Etiquetado del Portal de administración de proyectos                                        | Disponibilidad general                   | SÍ                | SÍ         |
| Portal de etiquetador                                                            | Disponibilidad general                   | SÍ                | SÍ         |
| Etiquetado mediante recursos privados                                          | Disponibilidad general                   | SÍ                | SÍ         |
| Etiquetado asistido por ML (clasificación de imágenes y detección de objetos)           | Vista previa pública       | SÍ                | SÍ         |
| **Aprendizaje automático responsable** |   | | |
| Capacidad de explicación en la interfaz de usuario                                                       | Vista previa pública       | No                 | No          |
| Kit de herramientas de privacidad diferencial WhiteNoise                                    | OSS                  | No                 | No          |
| Etiquetas personalizadas en Azure Machine Learning para implementar hojas de datos              | Disponibilidad general                   | No                 | No          |
| Integración con AzureML de equidad                                               | Vista previa pública       | No                 | No          |
| SDK de interpretabilidad                                                      | Disponibilidad general                   | SÍ                | SÍ         |
| **Cursos** |   | | |
| Streaming de registro de experimentación                                              | Disponibilidad general                   | SÍ                | SÍ         |
| Aprendizaje de refuerzo                                                     | Vista previa pública       | No                 | No          |
| Interfaz de usuario de experimentación                                                         | Disponibilidad general                   | SÍ                | SÍ         |
| Integración de .NET ML.NET 1.0                                                | Disponibilidad general                   | SÍ                | SÍ         |
| **Inferencia** |   | | |
| Inferencia por lotes                                                          | Disponibilidad general                   | SÍ                | SÍ         |
| Data Box Edge con FPGA                                                    | Vista previa pública       | No                 | No          |
| **Otros** |   | | |
| Open Datasets                                                              | Vista previa pública       | SÍ                | SÍ         |
| Cognitive Search personalizado                                                    | Vista previa pública       | SÍ                | SÍ         |
| Muchos modelos                                                                | Vista previa pública       | No                 | No          |


### <a name="azure-government-scenarios"></a>Escenarios de Azure Government

| Escenario                                                    | EE.UU.: Virginia | EE.UU.: Arizona| Limitaciones  |
|----------------------------------------------------------------------------|:----------------------:|:--------------------:|-------------|
| **Configuración de la seguridad general** |   | | |
| Comunicación de red privada entre servicios                                     | No | No | No hay Private Link actualmente | 
| Deshabilitación o control del acceso a Internet (entrante y saliente) y una red virtual específica | PARTIAL| PARTIAL   | ACR detrás de VNet no está disponible en Azure Government (doble comprobación en ACI) | 
| Selección de ubicación de todos los recursos o servicios asociados  | SÍ | SÍ |  |
| Cifrado en reposo y en tránsito                                                 | SÍ | SÍ |  |
| Acceso de SSH y raíz a los recursos de proceso                                          | SÍ | SÍ |  |
| Mantenimiento de la seguridad de los sistemas implementados (instancias, puntos de conexión, etc.), incluida la protección de puntos de conexión, la revisión y el registro |  PARTIAL|  PARTIAL |ACI detrás de VNet y punto de conexión privado no disponibles actualmente |                                  
| Control (deshabilitación, limitación o restricción) del uso de la integración de ACI/AKS                    | PARTIAL| PARTIAL |ACI detrás de VNet y punto de conexión privado no disponibles actualmente|
| Control de acceso basado en rol (RBAC): creaciones de roles personalizados                           | SÍ | SÍ |  |
| Control de acceso a imágenes de ACR utilizadas por el servicio de ML (proporcionado o mantenido por Azure frente a personalizado)  |PARTIAL|  PARTIAL | ACR detrás del punto de conexión privado y la red virtual no se admiten en Azure Government |
| **Uso general de Machine Learning Service** |  | | |
| Capacidad de tener un entorno de desarrollo para crear un modelo, entrenar ese modelo, hospedarlo como punto de conexión y consumirlo a través de una aplicación web     | SÍ | SÍ |  |
| Capacidad de extraer datos de ADLS (Data Lake Storage)                                 |SÍ | SÍ |  |
| Capacidad de extraer datos de Azure Blob Storage                                       |SÍ | SÍ |  |



### <a name="additional-azure-government-limitations"></a>Limitaciones adicionales de Azure Government

* En el caso de instancias de proceso de Azure Machine Learning, la capacidad de actualizar un token que dura más de 24 horas no está disponible en Azure Government.
* La generación de perfiles de modelos no admite 4 CPU en la región EE.UU.: Arizona.   
* Es posible que los cuadernos de ejemplo no funcionen en Azure Government si necesitan tener acceso a datos públicos.
* Direcciones IP: El comando de la CLI que se usa en las instrucciones de [red virtual y tunelización forzada](how-to-secure-training-vnet.md#forced-tunneling) no devuelve intervalos IP. Use los [intervalos IP y las etiquetas de servicio de Azure para Azure Government](https://www.microsoft.com/download/details.aspx?id=57063) en su lugar.
* En el caso de las canalizaciones programadas, también proporcionamos un mecanismo de desencadenador basado en blobs. Este mecanismo no es compatible con las áreas de trabajo de CMK. Para habilitar un desencadenador basado en blobs para áreas de trabajo de CMK, tiene que realizar una configuración adicional. Para más información, consulte [Desencadenamiento de una ejecución de una canalización de Machine Learning desde una aplicación lógica](how-to-trigger-published-pipeline.md).
* Firewalls: Cuando use una región Azure Government, agregue los siguientes hosts adicionales a la configuración del firewall:

    * Para el uso de Arizona: `usgovarizona.api.ml.azure.us`
    * Para el uso de Virginia: `usgovvirginia.api.ml.azure.us`
    * Para ambos: `graph.windows.net` 


## <a name="azure-china-21vianet"></a>Azure China 21Vianet 

| Característica                                       | Estado de la nube pública | CH-East-2 | CH-North-3 |
|----------------------------------------------------------------------------|:------------------:|:--------------------:|:-------------:|
| **Aprendizaje automático automatizado** |    | | |
| Creación y ejecución de experimentos en cuadernos                                    | Disponibilidad general               | SÍ       | N/D        |
| Creación y ejecución de experimentos en la experiencia web de Studio                        | Vista previa pública   | SÍ       | N/D        |
| Capacidades de previsión de líderes del sector                                  | Disponibilidad general               | SÍ       | N/D        |
| Compatibilidad con aprendizaje profundo y otros aprendizajes avanzados                      | Disponibilidad general               | SÍ       | N/D        |
| Compatibilidad con datos de gran tamaño (hasta 100 GB)                                          | Vista previa pública   | SÍ       | N/D        |
| Integración con Azure Databricks                                              | Disponibilidad general               | No        | N/D        |
| Integraciones con SQL, CosmosDB y HDInsight                                   | Disponibilidad general               | SÍ       | N/D        |
| **Canalizaciones de Machine Learning** |    | | |
| Creación, ejecución y publicación de canalizaciones con el SDK de Azure Machine Learning                   | Disponibilidad general               | SÍ       | N/D        |
| Creación de puntos de conexión de canalización con el SDK de Azure Machine Learning                           | Disponibilidad general               | SÍ       | N/D        |
| Creación, edición y eliminación de ejecuciones programadas de canalizaciones con el SDK de Azure Machine Learning | Disponibilidad general               | SÍ       | N/D        |
| Visualización de los detalles de la ejecución de la canalización en Studio                                        | Disponibilidad general               | SÍ       | N/D        |
| Creación, ejecución, visualización y publicación de canalizaciones en el diseñador de Azure Machine Learning          | Disponibilidad general  | SÍ       | N/D        |
| Integración de Azure Databricks con canalización de Machine Learning                             | Disponibilidad general               | No        | N/D        |
| Creación de puntos de conexión de canalización en el diseñador de Azure Machine Learning                             | Disponibilidad general   | SÍ       | N/D        |
| **Cuadernos integrados** |   | | |
| Uso compartido de archivos y cuadernos del área de trabajo                                        | Disponibilidad general               | SÍ       | N/D        |
| Compatibilidad con R y Python                                                       | Disponibilidad general               | SÍ       | N/D        |
| Compatibilidad con redes virtuales                                                    | Vista previa pública   | No        | N/D        |
| **Instancia de proceso** |    | | |
| Instancias de proceso administradas para cuadernos integrados                         | Disponibilidad general               | No        | N/D        |
| Jupyter, integración con JupyterLab                                            | Disponibilidad general               | SÍ       | N/D        |
| Compatibilidad con redes virtuales (VNet)                                             | Vista previa pública   | SÍ       | N/D        |
| **Compatibilidad con SDK** |    | | |
| Compatibilidad con el SDK de R                                                              | Vista previa pública   | SÍ       | N/D        |
| Compatibilidad con el SDK de Python                                                         | Disponibilidad general               | SÍ       | N/D        |
| **Seguridad** |   | | |
| Compatibilidad con Virtual Network (VNet) para aprendizaje                                | Disponibilidad general               | SÍ       | N/D        |
| Compatibilidad con Virtual Network (VNet) para inferencia                               | Disponibilidad general               | SÍ       | N/D        |
| Autenticación de puntos de conexión de puntuación                                            | Vista previa pública   | SÍ       | N/D        |
| Private Link de área de trabajo                                                     | Vista previa pública   | No        | N/D        |
| ACI detrás de VNet                                                            | Vista previa pública   | No        | N/D        |
| ACR detrás de VNet                                                            | Vista previa pública   | No        | N/D        |
| IP privada de clúster de AKS                                                  | Vista previa pública   | No        | N/D        |
| **Proceso** |   | | |
| Administración de cuotas entre áreas de trabajo                                         | Disponibilidad general               | SÍ       | N/D        |
| **Datos para el aprendizaje automático** | | | |
| Creación, visualización o edición de conjuntos de datos y almacenes de datos del SDK                  | Disponibilidad general               | SÍ       | N/D        |
| Creación, visualización o edición de conjuntos de datos y almacenes de datos de la interfaz de usuario                   | Disponibilidad general               | SÍ       | N/D        |
| Visualización, edición o eliminación de monitores de desfase de conjuntos de datos del SDK                   | Vista previa pública   | SÍ       | N/D        |
| Visualización, edición o eliminación de monitores de desfase de conjunto de datos de la interfaz de usuario                    | Vista previa pública   | SÍ       | N/D        |
| **Ciclo de vida de Machine Learning** |    | | |
| Generación de perfiles de modelos                                                            | Disponibilidad general               | PARTIAL   | N/D        |
| La extensión Azure DevOps para Machine Learning y la CLI de Azure ML         | Disponibilidad general               | SÍ       | N/D        |
| Modelos acelerados mediante hardware basados en FPGA                                     | Disponibilidad general               | No        | N/D        |
| Integración con Visual Studio Code                                             | Vista previa pública   | No        | N/D        |
| Integración con Event Grid                                                     | Vista previa pública   | SÍ       | N/D        |
| Integración de Azure Stream Analytics con Azure Machine Learning               | Vista previa pública   | No        | N/D        |
| **Etiquetado** |    | | |
| Etiquetado del Portal de administración de proyectos                                        | Disponibilidad general               | SÍ       | N/D        |
| Portal de etiquetador                                                            | Disponibilidad general               | SÍ       | N/D        |
| Etiquetado mediante recursos privados                                          | Disponibilidad general               | SÍ       | N/D        |
| Etiquetado asistido por ML (clasificación de imágenes y detección de objetos)           | Vista previa pública   | SÍ       | N/D        |
| **Aprendizaje automático responsable** |    | | |
| Capacidad de explicación en la interfaz de usuario                                                       | Vista previa pública   | No        | N/D        |
| Kit de herramientas de privacidad diferencial WhiteNoise                                    | OSS              | No        | N/D        |
| Etiquetas personalizadas en Azure Machine Learning para implementar hojas de datos              | Disponibilidad general               | No        | N/D        |
| Integración con AzureML de equidad                                               | Vista previa pública   | No        | N/D        |
| SDK de interpretabilidad                                                      | Disponibilidad general               | SÍ       | N/D        |
| **Cursos** |    | | |
| Streaming de registro de experimentación                                              | Disponibilidad general               | SÍ       | N/D        |
| Aprendizaje de refuerzo                                                     | Vista previa pública   | No        | N/D        |
| Interfaz de usuario de experimentación                                                         | Disponibilidad general               | SÍ       | N/D        |
| Integración de .NET ML.NET 1.0                                                | Disponibilidad general               | SÍ       | N/D        |
| **Inferencia** |   | | |
| Inferencia por lotes                                                          | Disponibilidad general               | SÍ       | N/D        |
| Data Box Edge con FPGA                                                    | Vista previa pública   | No        | N/D        |
| **Otros** |    | | |
| Open Datasets                                                              | Vista previa pública   | SÍ       | N/D        |
| Cognitive Search personalizado                                                    | Vista previa pública   | SÍ       | N/D        |
| Muchos modelos                                                                | Vista previa pública   | No        | N/D        |



### <a name="additional-azure-china-limitations"></a>Limitaciones adicionales de Azure China

* Azure China tiene una SKU de máquina virtual limitada, especialmente para la SKU de GPU. Solo tiene la familia NCv3 (V100).
* Los puntos de conexión de la API de REST son diferentes de Azure global. Use la tabla siguiente para buscar el punto de conexión de API de REST para regiones Azure China:

    | Punto de conexión de REST                 | Azure global                                 | Gobierno de China                           |
    |------------------|--------------------------------------------|--------------------------------------------|
    | Plano de administración | `https://management.azure.com/`              | `https://management.chinacloudapi.cn/`       |
    | Plano de datos       | `https://{location}.experiments.azureml.net` | `https://{location}.experiments.ml.azure.cn` |
    | Azure Active Directory              | `https://login.microsoftonline.com`          | `https://login.chinacloudapi.cn`             |

* Es posible que los cuadernos de ejemplo no funcionen, si necesitan tener acceso a datos públicos.
* Intervalos de direcciones IP: El comando de la CLI que se usa en las instrucciones de [tunelización forzada de red virtual](how-to-secure-training-vnet.md#forced-tunneling) no devuelve intervalos IP. Use los [intervalos IP y las etiquetas de servicio de Azure para Azure China](https://www.microsoft.com//download/details.aspx?id=57062) en su lugar.
* No se admite la versión preliminar de las instancias de proceso de Azure Machine Learning en un área de trabajo en la que Private Link está habilitado, pero se admitirá CI en la siguiente implementación de la expansión del servicio a todas las regiones AML.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las regiones en las que está disponible Azure Machine Learning, consulte [Productos por región](https://azure.microsoft.com/global-infrastructure/services/).
