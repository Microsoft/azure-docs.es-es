---
title: Información general sobre la privacidad y el aislamiento de la red virtual
titleSuffix: Azure Machine Learning
description: Use una instancia de Azure Virtual Network aislada con Azure Machine Learning para proteger los recursos del área de trabajo y los entornos de proceso.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 07/07/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, references_regions
ms.openlocfilehash: 36d3d84949e44719474656d07da9c7b7c46a4e98
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893180"
---
# <a name="virtual-network-isolation-and-privacy-overview"></a>Información general sobre la privacidad y el aislamiento de la red virtual

En este artículo, aprenderá a usar redes virtuales (VNET) para proteger la comunicación de red en Azure Machine Learning. En este artículo se usa un escenario de ejemplo para mostrar cómo configurar una red virtual completa.

Este artículo forma parte de una serie de cinco capítulos que le guía a través de la protección de un flujo de trabajo de Azure Machine Learning. Se recomienda encarecidamente leer este artículo de información general para comprender los conceptos en primer lugar. 

Los demás artículos de esta serie son:

**1. Introducción a las redes virtuales** > [2. Protección del área de trabajo](how-to-secure-workspace-vnet.md) > [3. Protección del entorno de entrenamiento](how-to-secure-training-vnet.md) > [4. Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md) > [5. Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por hecho que está familiarizado con los siguientes temas:
+ [Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)
+ [Redes de IP](https://docs.microsoft.com/azure/virtual-network/virtual-network-ip-addresses-overview-arm)
+ [Azure Private Link](how-to-configure-private-link.md)
+ [Grupos de seguridad de red (NSG)](../virtual-network/security-overview.md)
+ [Firewalls de red](../firewall/overview.md)

## <a name="example-scenario"></a>Escenario de ejemplo

En esta sección, aprenderá cómo se configura un escenario de red común para proteger la comunicación de Azure Machine Learning con las direcciones IP privadas.

En la tabla siguiente se compara el modo en que los servicios acceden a diferentes partes de una red de Azure Machine Learning con y sin una red virtual.

| Escenario | Área de trabajo |  Recursos asociados | Entrenamiento del entorno de proceso | Inferencia del entorno de proceso |
|-|-|-|-|-|-|
|**Sin red virtual**| Dirección IP pública | Dirección IP pública | Dirección IP pública | Dirección IP pública |
|**Protección de recursos en una red virtual**| IP privada (punto de conexión privado) | IP pública (punto de conexión de servicio) <br> **- o -** <br> IP privada (punto de conexión privado) | Dirección IP privada | Dirección IP privada  | 

* **Área de trabajo**: cree un punto de conexión privado desde la red virtual para conectarse a Private Link en el área de trabajo. El punto de conexión privado conecta el área de trabajo a la red virtual a través de varias direcciones IP privadas.
* **Recurso asociado**: use puntos de conexión de servicio o puntos de conexión privados para conectarse a recursos del área de trabajo como Azure Storage, Azure Key Vault y Azure Container Services.
    * Los **puntos de conexión de servicio** proporcionan la identidad de la red virtual al servicio de Azure. Una vez que habilita puntos de conexión de servicio en su red virtual, puede agregar una regla de red virtual para proteger los recursos de los servicios de Azure en la red virtual. Los puntos de conexión de servicio usan direcciones IP públicas.
    * Los **puntos de conexión privados** son interfaces de red que le permiten conectarse de forma segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual, y coloca el servicio de manera eficaz en su red virtual.
* **Acceso al proceso de entrenamiento**: acceda a destinos de proceso de entrenamiento como la instancia de proceso y los clústeres de proceso de Azure Machine Learning de forma segura con direcciones IP privadas. 
* **Acceso al proceso de inferencia**: acceda a los clústeres de proceso de Azure Kubernetes Services (AKS) con direcciones IP privadas.


En las cinco secciones siguientes se muestra cómo proteger el escenario de red descrito anteriormente. Para proteger la red, debe hacer lo siguiente:

1. Proteger el [**área de trabajo y los recursos asociados**](#secure-the-workspace-and-associated-resources).
1. Proteger el [**entorno de entrenamiento**](#secure-the-training-environment).
1. Proteger el [**entorno de inferencia**](#secure-the-inferencing-environment).
1. Opcional: [**habilitar la funcionalidad de Studio**](#optional-enable-studio-functionality).
1. [**Configurar el firewall**](#configure-firewall-settings).

## <a name="secure-the-workspace-and-associated-resources"></a>Proteger el área de trabajo y los recursos asociados

Realice los pasos siguientes para proteger el área de trabajo y los recursos asociados. Estos pasos permiten que los servicios se comuniquen en la red virtual.

1. Cree una [área de trabajo habilitada para Private Link](how-to-secure-workspace-vnet.md#secure-the-workspace-with-private-endpoint) para habilitar la comunicación entre la red virtual y el área de trabajo.
1. Agregue Azure Key Vault a la red virtual con un [punto de conexión de servicio](../key-vault/general/overview-vnet-service-endpoints.md) o un [punto de conexión privado](../key-vault/general/private-link-service.md). Establezca Key Vault para ["permitir que los servicios de confianza de Microsoft puedan omitir este firewall"](how-to-secure-workspace-vnet.md#secure-azure-key-vault).
1. Agregue una cuenta de Azure Storage a la red virtual con un [punto de conexión de servicio](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts) o un [punto de conexión privado](../storage/common/storage-private-endpoints.md).
1. [Configure Azure Container Registry para usar un punto de conexión privado](how-to-secure-workspace-vnet.md#enable-azure-container-registry-acr) y [habilite la delegación de subred en Azure Container Instances](how-to-secure-inferencing-vnet.md#enable-azure-container-instances-aci).

![Diagrama de arquitectura que muestra cómo el área de trabajo y los recursos asociados se comunican entre sí a través de puntos de conexión de servicio o puntos de conexión privados en una red virtual](./media/how-to-network-security-overview/secure-workspace-resources.png)

Para obtener instrucciones detalladas sobre cómo completar estos pasos, consulte [Protección de un área de trabajo de Azure Machine Learning](how-to-secure-workspace-vnet.md). 

### <a name="limitations"></a>Limitaciones

La protección del área de trabajo y los recursos asociados en una red virtual presenta las siguientes limitaciones:
- La instancia de Private Link del área de trabajo solo está disponible en las siguientes regiones: eastus, westus2, southcentralus
    - Esta limitación no se aplica a los recursos asociados. Por ejemplo, puede habilitar la red virtual para el almacenamiento en cualquier región de Azure Machine Learning.
- Todos los recursos deben estar detrás de la misma red virtual. Sin embargo, se permiten subredes en la misma red virtual.
- Algunas características de Studio como el diseñador, AutoML, etiquetado y generación de perfiles de datos no se pueden usar con cuentas de almacenamiento configuradas para usar un punto de conexión privado. Si necesita usar estas características de Studio, utilice puntos de conexión de servicio en su lugar.

## <a name="secure-the-training-environment"></a>Protección del entorno de entrenamiento

En esta sección, aprenderá a proteger el entorno de entrenamiento en Azure Machine Learning. También aprenderá cómo Azure Machine Learning completa un trabajo de entrenamiento para comprender el funcionamiento conjunto de las configuraciones de red.

Para proteger el entorno de entrenamiento, siga estos pasos:

1. Cree una [instancia de proceso y un clúster de proceso](how-to-secure-training-vnet.md#compute-instance) de Azure Machine Learning en la red virtual para ejecutar el trabajo de entrenamiento.
1. [Permita la comunicación entrante desde el servicio de Azure Batch](how-to-secure-training-vnet.md#mlcports) para que el servicio de Batch pueda enviar trabajos a los recursos de proceso. 

![Diagrama de arquitectura que muestra cómo proteger las instancias y los clústeres de proceso administrados](./media/how-to-network-security-overview/secure-training-environment.png)

Para obtener instrucciones detalladas sobre cómo completar estos pasos, consulte [Protección de un entorno de entrenamiento](how-to-secure-training-vnet.md). 

### <a name="example-training-job-submission"></a>Ejemplo de envío de trabajo de entrenamiento 

En esta sección, aprenderá cómo Azure Machine Learning se comunica de forma segura entre los servicios para enviar un trabajo de entrenamiento. Muestra cómo funcionan todas las configuraciones de forma conjunta para proteger la comunicación.

1. El cliente carga scripts y datos de entrenamiento en las cuentas de almacenamiento protegidas con un punto de conexión de servicio o privado.

1. El cliente envía un trabajo de entrenamiento al área de trabajo de Azure Machine Learning a través del punto de conexión privado.

1. El servicio de Azure Batch recibe el trabajo del área de trabajo y envía el trabajo de entrenamiento al entorno de proceso a través del equilibrador de carga público que se aprovisiona con el recurso de proceso. 

1. El recurso de proceso recibe el trabajo e inicia el entrenamiento. El recurso de proceso accede a cuentas de almacenamiento seguras para descargar archivos de entrenamiento y cargar la salida. 

![Diagrama de arquitectura que muestra cómo se envía un trabajo de entrenamiento de Azure Machine Learning mientras se usa una red virtual](./media/how-to-network-security-overview/secure-training-job-submission.png)


### <a name="limitations"></a>Limitaciones

- La instancia de proceso de Azure y los clústeres de proceso de Azure deben estar en la misma red virtual, región y suscripción que el área de trabajo y sus recursos asociados. 

## <a name="secure-the-inferencing-environment"></a>Protección del entorno de inferencia

En esta sección, aprenderá las opciones disponibles para proteger un entorno de inferencia. Se recomienda usar los clústeres de Azure Kubernetes Services (AKS) para las implementaciones de producción a gran escala.

Tiene dos opciones para los clústeres de AKS en una red virtual:

- Implementar o conectar un clúster de AKS predeterminado a la red virtual.
- Conectar un clúster de AKS privado a la red virtual.

Los **clústeres de AKS predeterminados** tienen un plano de control con las direcciones IP públicas. Puede agregar un clúster de AKS predeterminado a la red virtual durante la implementación o conectar un clúster después de su creación.

Los **clústeres de AKS privados** tienen un plano de control, al que solo se puede acceder a través de direcciones IP privadas. Los clústeres de AKS privados se deben conectar una vez creado el clúster.

Para obtener instrucciones detalladas sobre cómo agregar clústeres predeterminados y privados, consulte [Protección de un entorno de inferencia](how-to-secure-inferencing-vnet.md). 

En el diagrama de red siguiente se muestra un área de trabajo de Azure Machine Learning protegida con un clúster de AKS privado conectado a la red virtual.

![Diagrama de arquitectura que muestra cómo conectar un clúster de AKS privado a la red virtual. El plano de control de AKS se coloca fuera de la red virtual del cliente.](./media/how-to-network-security-overview/secure-inferencing-environment.png)

### <a name="limitations"></a>Limitaciones
- Los clústeres de AKS deben pertenecer a la misma red virtual que el área de trabajo y sus recursos asociados. 

## <a name="optional-enable-studio-functionality"></a>Opcional: habilitación de la funcionalidad de Studio

[Proteger el área de trabajo](#secure-the-workspace-and-associated-resources) > [Proteger el entorno de entrenamiento](#secure-the-training-environment) > [Proteger el entorno de inferencia](#secure-the-inferencing-environment) > **Habilitar la funcionalidad de Studio** > [Configurar el firewall](#configure-firewall-settings)

Aunque Studio puede acceder a los datos de una cuenta de almacenamiento configurada con un punto de conexión de servicio, algunas características están deshabilitadas de forma predeterminada:

* Vista previa de los datos en Studio.
* Visualización de los datos en el diseñador.
* Envío de un experimento de AutoML.
* Inicio de un proyecto de etiquetado.

Para habilitar la funcionalidad completa mientras se usa un punto de conexión de servicio de almacenamiento, consulte [Uso de Azure Machine Learning Studio en una red virtual](how-to-enable-studio-virtual-network.md#access-data-using-the-studio). Actualmente, Studio no admite puntos de conexión privados de almacenamiento.

### <a name="limitations"></a>Limitaciones
- Studio no puede acceder a los datos de las cuentas de almacenamiento configuradas para usar puntos de conexión privados. Para obtener la funcionalidad completa, debe usar puntos de conexión de servicio para el almacenamiento y utilizar la identidad administrada.

## <a name="configure-firewall-settings"></a>Configuración del firewall

Configure el firewall para controlar el acceso a los recursos del área de trabajo de Azure Machine Learning y a la red pública de Internet. Aunque se recomienda Azure Firewall, debería poder usar otros productos de firewall para proteger la red. Si tiene alguna pregunta sobre cómo permitir la comunicación a través del firewall, consulte la documentación del firewall que usa.

Para obtener más información sobre la configuración del firewall, consulte [Uso de áreas de trabajo detrás de un firewall](how-to-access-azureml-behind-firewall.md).

## <a name="next-steps"></a>Pasos siguientes

Este artículo forma parte de una serie sobre redes virtuales de cuatro capítulos. Vea el resto de los artículos para obtener información sobre cómo proteger una red virtual:

* [Parte 2: Introducción a las redes virtuales](how-to-secure-workspace-vnet.md)
* [Parte 3: Protección del entorno de entrenamiento](how-to-secure-training-vnet.md)
* [Parte 4: Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md)
* [Parte 5: Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)
