---
title: 'Azure Defender para registros de contenedor: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para registros de contenedor.
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: conceptual
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 742d230d6ed1535bada9c35219d2ddf68fd64ae1
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90977164"
---
# <a name="introduction-to-azure-defender-for-container-registries"></a>Introducción a Azure Defender para registros de contenedor

Azure Container Registry (ACR) es un servicio de registro de Docker privado y administrado que almacena y administra las imágenes de contenedor de las implementaciones de Azure en un registro central. Se basa en el registro 2.0 de Docker de código abierto.

Para proteger todos los registros basados en Azure Resource Manager de su suscripción, habilite **Azure Defender para registros de contenedor** en el nivel de suscripción. A continuación, Security Center examinará las imágenes insertadas en el registro o importadas en el registro, así como las imágenes extraídas en los últimos 30 días. Esta característica se cobra por imagen.

## <a name="what-are-the-benefits-of-azure-defender-for-container-registries"></a>¿Cuáles son las ventajas de Azure Defender para registros de contenedor?

Security Center identifica registros de ACR basados Azure Resource Manager en la suscripción y brinda una función fluida de evaluación y administración de vulnerabilidades nativa de Azure para las imágenes del registro.

**Azure Defender para registros de contenedor** incluye un detector de vulnerabilidades para examinar las imágenes de los registros de Azure Container Registry basados en Azure Resource Manager y proporciona una mayor visibilidad de las vulnerabilidades de las imágenes. El detector integrado lo proporciona el proveedor de análisis de vulnerabilidades líder del sector, Qualys.

Si Qualys o Security Center encuentran incidencias, recibirá una notificación en el panel de Security Center. Por cada vulnerabilidad, Security Center proporciona recomendaciones útiles, junto con una clasificación de gravedad e información sobre cómo corregir el problema. Para más información sobre las recomendaciones de Security Center para los contenedores, consulte la [lista de recomendaciones de referencia](recommendations-reference.md#recs-containers).

Security Center filtra y clasifica los resultados del análisis. Cuando una imagen es correcta, Security Center la marca como tal. Security Center solo genera recomendaciones de seguridad para las imágenes que tienen incidencias sin resolver. Security Center proporciona detalles de cada vulnerabilidad detectada y una clasificación de gravedad. Asimismo, ofrece instrucciones sobre cómo corregir las vulnerabilidades específicas encontradas en cada imagen.

Al enviar notificaciones solo cuando hay problemas, Security Center reduce las alertas informativas no deseadas.


## <a name="when-are-images-scanned"></a>¿Cuándo se examinan las imágenes?

Hay tres desencadenadores para un examen de imagen:

- **Al insertar**: cada vez que se inserta una imagen en el registro, Security Center la examina de forma automática. Para desencadenar el examen de una imagen, insértela en el repositorio.

- **Extraído recientemente**: dado que se detectan nuevas vulnerabilidades cada día, **Azure Defender para registros de contenedor** también examina cualquier imagen que se haya extraído en los últimos 30 días. No hay ningún cargo adicional por un nuevo examen; como se mencionó anteriormente, se le facturará una vez por imagen.

- **Al importar**: Azure Container Registry tiene herramientas de importación para incorporar imágenes al registro desde Docker Hub, Microsoft Container Registry u otro registro de contenedor de Azure. **Azure Defender para registros de contenedor** examina las imágenes admitidas que importe. Obtenga más información en [Importación de imágenes de contenedor en un registro de contenedor](../container-registry/container-registry-import-images.md).
 
El análisis se completa normalmente en 2 minutos, pero puede tardar hasta 15 minutos. Los resultados se publican como recomendaciones de Security Center, como en este caso:

[![Ejemplo de recomendación Azure Security Center sobre puntos vulnerables detectados en una imagen hospedada de Azure Container Registry (ACR)](media/azure-container-registry-integration/container-security-acr-page.png)](media/azure-container-registry-integration/container-security-acr-page.png#lightbox)


## <a name="how-does-security-center-work-with-azure-container-registry"></a>Funcionamiento de Security Center con Azure Container Registry

A continuación se muestra un diagrama detallado de los componentes y las ventajas de proteger los registros con Security Center.

![Introducción de alto nivel de Azure Security Center y Azure Container Registry (ACR)](./media/azure-container-registry-integration/aks-acr-integration-detailed.png)




## <a name="faq-for-azure-container-registry-image-scanning"></a>Preguntas más frecuentes sobre el análisis de imágenes de Azure Container Registry

### <a name="how-does-security-center-scan-an-image"></a>¿De qué forma analiza Security Center las imágenes?
La imagen se extrae del registro. Luego, se ejecuta en un espacio aislado con el escáner de Qualys, que extrae una lista de puntos vulnerables conocidos.

Security Center filtra y clasifica los resultados del análisis. Cuando una imagen es correcta, Security Center la marca como tal. Security Center solo genera recomendaciones de seguridad para las imágenes que tienen incidencias sin resolver. Al enviar notificaciones solo cuando hay problemas, Security Center reduce las alertas informativas no deseadas.

### <a name="how-often-does-security-center-scan-my-images"></a>¿Con qué frecuencia examina Security Center mis imágenes?
Los recorridos de imagen se desencadenan en cada una de las inserciones o importaciones, y si la imagen se ha extraído en los últimos 30 días. 

### <a name="can-i-get-the-scan-results-via-rest-api"></a>¿Se pueden obtener los resultados del examen mediante la API REST?
Sí. Los resultados se encuentran en [API REST de valoración secundaria](/rest/api/securitycenter/subassessments/list/). Asimismo, puede usar Azure Resource Graph (ARG), la API similar a Kusto para todos los recursos: una consulta puede recuperar un análisis específico.
 
### <a name="what-registry-types-are-scanned-what-types-are-billed"></a>¿Qué tipos de registros se analizan? ¿Qué tipos se facturan?
En la sección de disponibilidad se enumeran los tipos de registros de contenedor que admite Azure Defender para registros de contenedor. 

Si conecta registros no compatibles con su suscripción de Azure, no se examinarán y no se le facturarán.


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las características de seguridad de los contenedores de Security Center, consulte:

- [Seguridad de los contenedores y Azure Security Center](container-security.md)

- [Introducción a Azure Defender para Kubernetes](defender-for-kubernetes-introduction.md)


