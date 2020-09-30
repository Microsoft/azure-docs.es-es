---
title: Contenedores confidenciales en Azure Kubernetes Service (AKS)
description: Conozca la compatibilidad con contenedores sin modificar en contenedores confidenciales.
services: container-service
author: agowdamsft
ms.topic: article
ms.date: 9/22/2020
ms.author: amgowda
ms.service: container-service
ms.openlocfilehash: 528b843e24e1d63e4822c253b3636ef490e8fe8e
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90993845"
---
# <a name="confidential-containers"></a>Contenedores confidenciales

## <a name="overview"></a>Información general

Permita a los desarrolladores incorporar una **aplicación de Docker (nueva o existente)** y ejecutarla de forma segura en Azure Kubernetes Service (AKS) gracias a la compatibilidad con nodos de informática confidencial.

Los contenedores confidenciales ayudan a:

- integridad de datos 
- proteger la confidencialidad de los datos
- proteger la integridad del código
- proteger el código del contenedor mediante cifrado
- proteger las garantías basadas en hardware
- permitir la ejecución de aplicaciones existentes
- crear la raíz de confianza del hardware

Un entorno de ejecución de confianza (TEE) basado en hardware es un componente importante que se usa para proporcionar garantías sólidas mediante medidas de hardware y software de componentes de la base informática de confianza (TCB). Las comprobaciones de estas medidas ayudan en la validación del cálculo esperado y confirman las posibles manipulaciones de las aplicaciones de contenedor.

Los contenedores confidenciales admiten la ejecución de aplicaciones personalizadas desarrolladas con **Python, Java, Node.js, etc. o paquetes de aplicaciones de software, como NGINX, Redis Cache, MemCache**, etc., sin modificar en AKS.

Son el camino más rápido a la confidencialidad de los contenedores, lo que incluye la protección del contenedor mediante cifrado, al permitir la migración mediante lift-and-shift sin cambios o con cambios mínimos en la lógica de negocios.

![Conversión del contenedor confidencial](./media/confidential-containers/conf-con-deploy-process.jpg)


## <a name="confidential-container-enablers"></a>Habilitadores de contenedores confidenciales

Para ejecutar un contenedor de Docker existente, las aplicaciones de los nodos informáticos confidenciales requieren una capa de abstracción o software SGX para aprovechar el conjunto de instrucciones especiales de la CPU. El software SGX también permite proteger el código de las aplicaciones confidenciales y crear una ejecución directa en la CPU para quitar el sistema operativo invitado, el sistema operativo host o el hipervisor. Esta protección reduce las áreas de ataque de superficie generales y las vulnerabilidades con las capas de sistema operativo o del hipervisor.

Los contenedores confidenciales son completamente compatibles con AKS y se habilitan a través de los proyectos de asociados de Azure y de software de código abierto (OSS). Los desarrolladores pueden elegir proveedores de software en función de las características, la integración con los servicios de Azure y la compatibilidad con las herramientas.

## <a name="partner-enablers"></a>Habilitadores de asociados
> [!NOTE]
> Las soluciones siguientes se ofrecen a través de los asociados de Azure y pueden conllevar tarifas de licencia. Compruebe por otro lado los términos del software asociado. 

### <a name="fortanix"></a>Fortanix

[Fortanix](https://www.fortanix.com/) ofrece a los desarrolladores la opción de un portal y una experiencia basada en la CLI para incorporar sus aplicaciones en contenedores y convertirlas en contenedores confidenciales con capacidad SGX sin necesidad de modificar o volver a compilar la aplicación. Fortanix ofrece la flexibilidad para ejecutar y administrar el conjunto más amplio de aplicaciones, como las aplicaciones existentes, las nuevas aplicaciones nativas de enclave y las aplicaciones ya empaquetadas. Los usuarios pueden empezar con la interfaz de usuario de [Enclave Manager](https://em.fortanix.com/) o las [API REST](https://www.fortanix.com/api/em/) para crear contenedores confidenciales mediante la guía [Inicio rápido](https://support.fortanix.com/hc/en-us/articles/360049658291-Fortanix-Confidential-Container-on-Azure-Kubernetes-Service) para Azure Kubernetes Service.

![Proceso de implementación de Fortanix](./media/confidential-containers/fortanix-confidential-containers-flow.png)

### <a name="scone-scontain"></a>Scone (Scontain)

[SCONE](https://scontain.com/index.html?lang=en) admite directivas de seguridad que pueden generar certificados, claves y secretos, y garantiza que solo son visibles para los servicios atestiguados de una aplicación. De esta manera, los servicios de una aplicación se atestiguan automáticamente entre sí a través de TLS, sin necesidad de modificar las aplicaciones ni TLS. Esto se explica aquí con la ayuda de una sencilla aplicación de Flask: https://sconedocs.github.io/flask_demo/  

SCONE puede convertir los archivos binarios actuales en aplicaciones que se ejecutan dentro de enclaves sin necesidad de cambiar la aplicación o de volver a compilarla. SCONE también protege los lenguajes interpretados como Python mediante el cifrado de los archivos de datos y de los archivos de código de Python. Con la ayuda de una directiva de seguridad de SCONE, uno puede proteger los archivos cifrados contra los accesos, las modificaciones y las reversiones no autorizados. Puede encontrar información sobre cómo aplicar SCONE a una aplicación de Python existente [aquí](https://sconedocs.github.io/sconify_image/).

![Flujo de Scontain](./media/confidential-containers/scone-workflow.png)

Las implementaciones de Scone en nodos de informática confidencial con AKS se admiten e integran por completo. Puede comenzar con una aplicación de ejemplo aquí https://sconedocs.github.io/aks/.

### <a name="anjuna"></a>Anjuna

[Anjuna](https://www.anjuna.io/) proporciona software de plataforma SGX que permite ejecutar contenedores sin modificar en AKS. Puede encontrar más información sobre la **próxima** funcionalidad y el flujo de usuario [aquí](https://www.anjuna.io/microsoft-azure-confidential-computing-aks-lp).

Puede comenzar con un ejemplo de Redis Cache y de una aplicación personalizada de Python [aquí](https://www.anjuna.io/microsoft-azure-confidential-computing-aks-lp).

![Proceso de Anjuna](./media/confidential-containers/anjuna-process-flow.png)

## <a name="oss-enablers"></a>Habilitadores de OSS 
> [!NOTE]
> Las soluciones siguientes se ofrecen a través de proyectos de código abierto y no están directamente afiliadas a Azure Confidential Computing (ACC) o Microsoft.  

### <a name="graphene"></a>Graphene

[Graphene](https://grapheneproject.io/) es un sistema operativo invitado ligero diseñado para ejecutar una única aplicación Linux con requisitos mínimos de host. Graphene puede ejecutar aplicaciones en un entorno aislado con ventajas comparables a las de ejecutar un sistema operativo completo y tiene una buena compatibilidad con herramientas para convertir la aplicación de contenedor de Docker existente en Graphene Shielded Containers (GSC).

Puede comenzar con una aplicación de ejemplo y la implementación en AKS [aquí](https://graphene.readthedocs.io/en/latest/cloud-deployment.html#azure-kubernetes-service-aks).

### <a name="occlum"></a>Occlum
[Occlum](https://occlum.io/) es un sistema operativo de biblioteca (LibOS) de varios procesos y seguro para memoria para Intel SGX. Permite que las aplicaciones heredadas se ejecuten en SGX con pocas modificaciones en el código fuente, o incluso ninguna. Occlum protege de forma transparente la confidencialidad de las cargas de trabajo de los usuarios, a la vez que permite una migración mediante lift-and-shift sencilla a aplicaciones de Docker existentes.

Occlum admite implementaciones de AKS. Siga las instrucciones de implementación con varias aplicaciones de ejemplo [aquí](https://github.com/occlum/occlum/blob/master/docs/azure_aks_deployment_guide.md).


## <a name="confidential-containers-demo"></a>Demostración de contenedores confidenciales
Vea la demostración de contenedores confidenciales en el ámbito sanitario. El ejemplo está disponible [aquí](https://github.com/Azure-Samples/confidential-container-samples/blob/main/confidential-healthcare-scone-confinf-onnx/README.md). 

> [!VIDEO https://www.youtube.com/embed/PiYCQmOh0EI]


## <a name="get-in-touch"></a>Contacto

¿Tiene alguna pregunta sobre su implementación o desea convertirse en habilitador? Enviar un correo electrónico a acconaks@microsoft.com

## <a name="reference-links"></a>Vínculos de referencia

[Microsoft Azure Attestation](../attestation/overview.md)

[Máquinas virtuales DCsv2](virtual-machine-solutions.md)

[Azure Kubernetes Service (AKS)](../aks/intro-kubernetes.md)
