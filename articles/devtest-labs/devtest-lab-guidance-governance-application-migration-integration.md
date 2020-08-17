---
title: Migración e integración de aplicaciones en Azure DevTest Labs
description: En este artículo se proporcionan instrucciones para la gobernanza de la infraestructura de Azure DevTest Labs en el contexto de la migración y la integración de aplicaciones.
ms.topic: article
ms.date: 06/26/2020
ms.reviewer: christianreddington,anthdela,juselph
ms.openlocfilehash: 8d86728eefc46c74b49ac610e2207ce5e7ae6a9d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289350"
---
# <a name="governance-of-azure-devtest-labs-infrastructure---application-migration-and-integration"></a>Gobernanza de la infraestructura de Azure DevTest Labs: migración e integración de aplicaciones
Una vez establecido el entorno de laboratorio de desarrollo y pruebas, necesita pensar en las siguientes preguntas:

- ¿Cómo usar el entorno en el equipo del proyecto?
- ¿Cómo se garantiza el cumplimiento de las directivas organizativas necesarias y el mantenimiento de la agilidad para aportar valor a la aplicación?

## <a name="azure-marketplace-images-vs-custom-images"></a>Imágenes de Azure Marketplace frente a imágenes personalizadas

### <a name="question"></a>Pregunta
¿Cuándo debo usar una imagen de Azure Marketplace en lugar de una imagen de la organización personalizada propia?

### <a name="answer"></a>Respuesta
Azure Marketplace se debe usar de forma predeterminada a menos que tenga requisitos organizativos o preocupaciones específicos. Algunos ejemplos frecuentes son:

- Instalación compleja del software que requiere que una aplicación se incluya como parte de la imagen base.
- La instalación y configuración de una aplicación podrían tardar muchas horas, lo que no constituye un uso eficaz del tiempo de proceso que se agregará a una imagen de Azure Marketplace.
- Los desarrolladores y evaluadores necesitan tener acceso a una máquina virtual rápidamente y desean minimizar el tiempo de configuración de una nueva máquina virtual.
- Existencia de condiciones de cumplimiento o normativas (por ejemplo, las directivas de seguridad) para todas las máquinas.

El uso de imágenes personalizadas no se debe considerar a la ligera. Generan una complejidad adicional, ya que ahora necesita administrar archivos de disco duro virtual para las imágenes base subyacentes. También debe revisar periódicamente esas imágenes base con las actualizaciones de software. Estas actualizaciones incluyen nuevas actualizaciones del sistema operativo (SO) y las actualizaciones o los cambios de configuración necesarios para el paquete de software.

## <a name="formula-vs-custom-image"></a>Fórmula frente imágenes personalizadas

### <a name="question"></a>Pregunta
¿Cuándo debe usar una fórmula en lugar de una imagen personalizada?

### <a name="answer"></a>Respuesta
Normalmente, los factores decisivos en este escenario son el costo y la reutilización.

Si tiene un escenario donde muchos usuarios o laboratorios requieren una imagen con una gran cantidad de software sobre la imagen base, podría reducir el costo mediante la creación de una imagen personalizada. Esto significa que la imagen se crea una vez. Reduce el tiempo de configuración de la máquina virtual y el costo en que se incurre por la ejecución de la máquina virtual durante la configuración.

Sin embargo, otro factor que se debe tener en cuenta es la frecuencia de los cambios realizados en el paquete de software. Si ejecuta compilaciones diariamente y requiere que el software se encuentre en las máquinas virtuales de los usuarios, considere el uso de una fórmula en lugar de una imagen personalizada.

## <a name="use-custom-organizational-images"></a>Uso de imágenes de la organización personalizadas

### <a name="question"></a>Pregunta
¿Cómo se configura un proceso repetible fácilmente para incorporar imágenes de la organización personalizadas en un entorno de DevTest Labs?

### <a name="answer"></a>Respuesta
Vea [este vídeo sobre el patrón de una fábrica de imágenes](./devtest-lab-faq.md#blog-post). Este escenario es un escenario avanzado, y los scripts se proporcionan solo como ejemplo. Si se requieren cambios, deberá administrar y mantener los scripts usados en su entorno.

Uso de DevTest Labs para crear una canalización de la imagen personalizada en Azure Pipelines:

- [Introduction: Get VMs ready in minutes by setting up an image factory in Azure DevTest Labs](./devtest-lab-faq.md#blog-post) (Introducción: Preparación de máquinas virtuales en cuestión de minutos con la configuración de una fábrica de imágenes en Azure DevTest Labs)
- [Image Factory – Part 2! Setup Azure Pipelines and Factory Lab to Create VMs](./devtest-lab-faq.md#blog-post) (Fábrica de imágenes – Parte 2: Configuración de Azure Pipelines y del laboratorio de fábrica para crear máquinas virtuales)
- [Image Factory – Part 3: Save Custom Images and Distribute to Multiple Labs](./devtest-lab-faq.md#blog-post) (Fábrica de imágenes – Parte 3: Personalización de imágenes y distribución a varios laboratorios)
- [Vídeo: Custom Image Factory with Azure DevTest Labs](./devtest-lab-faq.md#blog-post) (Fábrica de imágenes personalizadas con Azure DevTest Labs)

## <a name="patterns-to-set-up-network-configuration"></a>Patrones para configurar la red

### <a name="question"></a>Pregunta
¿Cómo garantizar que las máquinas virtuales de desarrollo y pruebas no sean accesibles a través de la red pública de Internet? ¿Existen patrones recomendados para configurar la red?

### <a name="answer"></a>Respuesta
Sí. Hay dos aspectos que se deben tener en cuenta: el tráfico entrante y saliente.

**Tráfico entrante**: si la máquina virtual no tiene una dirección IP pública, no se puede acceder a ella a través de Internet. Un enfoque común consiste en asegurarse de que se establece una directiva de nivel de suscripción, para que ningún usuario pueda crear una dirección IP pública.

**Tráfico saliente**: si desea evitar que las máquinas virtuales sean accesibles directamente a través de la red pública de Internet y forzar el tráfico a través de un firewall corporativo, puede enrutar el tráfico local a través de ExpressRoute o VPN, mediante el uso del enrutamiento forzado.

> [!NOTE]
> Si tiene un servidor proxy que bloquea el tráfico sin configuración del proxy, no olvide agregar excepciones a la cuenta de almacenamiento de artefactos del laboratorio.

También puede usar grupos de seguridad de red para máquinas virtuales o subredes. Este paso agrega una capa adicional de protección para permitir o bloquear el tráfico.

## <a name="new-vs-existing-virtual-network"></a>Red virtual nueva frente a una existente

### <a name="question"></a>Pregunta
¿Cuándo se debe crear una red virtual para un entorno de DevTest Labs en lugar de usar una red virtual existente?

### <a name="answer"></a>Respuesta
Si las máquinas virtuales necesitan interactuar con la infraestructura existente, debe considerar la posibilidad de utilizar una red virtual existente dentro de su entorno de DevTest Labs. Además, si usa ExpressRoute, puede minimizar la cantidad de redes virtuales y subredes para no fragmentar el espacio de direcciones IP que se asigna para su uso en las suscripciones. También debe considerar la posibilidad de usar aquí el patrón de emparejamiento de la red virtual (modelo tipo hub-and-spoke). Este enfoque permite la comunicación de la red virtual o la subred entre las suscripciones dentro de una región determinada, aunque el emparejamiento entre regiones es una característica que se incorporará pronto a las redes de Azure.

En caso contrario, cada entorno de DevTest Labs podría tener su propia red virtual. Sin embargo, tenga en cuenta que hay [límites](../azure-resource-manager/management/azure-subscription-service-limits.md) en el número de redes virtuales por suscripción. La cantidad predeterminada es 50, aunque este límite puede aumentarse a 100.

## <a name="shared-public-or-private-ip"></a>Dirección IP compartida, pública o privada

### <a name="question"></a>Pregunta
¿Cuándo se debe usar una dirección IP compartida en lugar de una pública o privada?

### <a name="answer"></a>Respuesta
Si usa una VPN de sitio a sitio o ExpressRoute, considere la posibilidad de usar direcciones IP privadas para que se pueda acceder a las máquinas a través de la red interna, pero no a través de Internet.

> [!NOTE]
> Los propietarios de laboratorios pueden cambiar esta directiva de subred para asegurarse de que nadie puede crear accidentalmente direcciones IP públicas para sus máquinas virtuales. El propietario de la suscripción debe crear una directiva de suscripción que impida la creación de direcciones IP públicas.

Si se usan direcciones IP compartidas, las máquinas virtuales del laboratorio comparten una dirección IP pública. Este enfoque puede resultar útil si necesita impedir que se traspasen los límites de las direcciones IP públicas de una suscripción concreta.

## <a name="limits-of-number-of-virtual-machines-per-user-or-lab"></a>Límites del número de máquinas virtuales por usuario o laboratorio

### <a name="question"></a>Pregunta
¿Existe alguna regla que determine cuántas máquinas virtuales se pueden configurar por usuario o por laboratorio?

### <a name="answer"></a>Respuesta
Para determinar el número de máquinas virtuales por usuario o laboratorio, existen tres inquietudes principales:

- El **costo total** en que el equipo puede incurrir por los recursos del laboratorio. Es fácil poner en marcha varias máquinas. Para controlar los costos, un mecanismo consiste en limitar el número de máquinas virtuales por usuario o laboratorio.
- Las [cuotas a nivel de suscripción](../azure-resource-manager/management/azure-subscription-service-limits.md) disponibles repercuten en el número total de máquinas virtuales de un laboratorio. Uno de los límites máximos es 800 grupos de recursos por suscripción. DevTest Labs actualmente crea un grupo de recursos para cada máquina virtual (a menos que se usen direcciones IP públicas compartidas). Si hay 10 laboratorios en una suscripción, los laboratorios podrían incluir aproximadamente 79 máquinas en cada laboratorio (Límite máximo de 800 – 10 grupos de recursos para los 10 laboratorios) = 79 máquinas virtuales por laboratorio.
- Si el laboratorio está conectado a un entorno local a través de ExpressRoute, por ejemplo, hay **espacios de direcciones IP definidos disponibles** para la red virtual o subred. Para garantizar que no se produzcan errores al crear máquinas virtuales en el laboratorio (error: no se puede obtener la dirección IP), los propietarios del laboratorio pueden especificar el número máximo de máquinas virtuales por laboratorio de tal forma que esté en consonancia con el espacio de direcciones IP disponible.

## <a name="use-resource-manager-templates"></a>Uso de plantillas de Resource Manager

### <a name="question"></a>Pregunta
¿Cómo se pueden usar plantillas de Resource Manager en un entorno de DevTest Labs?

### <a name="answer"></a>Respuesta
Para implementar las plantillas de Resource Manager en un entorno de DevTest Labs, aplique los pasos mencionados en el artículo sobre las [características de entornos de DevTest Labs](devtest-lab-test-env.md). Básicamente, consulte las plantillas de Resource Manager en un repositorio de GIT (Azure Repos o GitHub) y agregue un [repositorio privado para las plantillas](devtest-lab-test-env.md) al laboratorio.

Este escenario puede no ser útil si usa DevTest Labs para hospedar máquinas de desarrollo, pero sí puede serlo si va a crear un entorno de ensayo, que es representativo de producción.

También merece la pena tener en cuenta que el número de máquinas virtuales por laboratorio o por usuario solo limita el número de máquinas creadas de forma nativa en el laboratorio y no según el entorno (plantillas de Resource Manager).

## <a name="next-steps"></a>Pasos siguientes
Consulte [Uso de entornos en DevTest Labs](devtest-lab-test-env.md).
