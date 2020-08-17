---
title: Niveles y características del servicio de registro
description: Obtenga información sobre las características y los límites de los niveles de servicio Básico, Estándar y Premium (SKU) de Azure Container Registry.
ms.topic: article
ms.date: 05/18/2020
ms.openlocfilehash: 66cb5d7c3cdca45f7a44f0f23bfa449d5ade6dbf
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86512626"
---
# <a name="azure-container-registry-service-tiers"></a>Niveles del servicio Azure Container Registry

Azure Container Registry está disponible en varios niveles de servicio (también conocidos como SKU). Estos niveles ofrecen precios predecibles y varias opciones para alinearse con la capacidad y los patrones de uso de su registro de Docker privado en Azure.

| Nivel | Descripción |
| --- | ----------- |
| **Basic** | Un punto de entrada optimizado para los costos para que los desarrolladores aprendan sobre Azure Container Registry. Los registros básicos tienen las mismas funcionalidades de programación que Estándar y Premium (como [integración de la autenticación](container-registry-authentication.md#individual-login-with-azure-ad) de Azure Active Directory, la [eliminación de imágenes][container-registry-delete] y [webhooks][container-registry-webhook]). Sin embargo, el almacenamiento incluido y el rendimiento de las imágenes son más adecuadas para escenarios de uso inferior. |
| **Estándar** | Los registros estándar ofrecen las mismas funcionalidades que los básicos, pero con más almacenamiento y un mayor rendimiento de las imágenes. Los registros estándar deberían satisfacer las necesidades de la mayoría de los escenarios de producción. |
| **Premium** | Los registros premium proporcionan la mayor cantidad de almacenamiento incluido y operaciones simultáneas, por lo que permiten trabajar con escenarios de mayor volumen. Además de la mayor capacidad de rendimiento de las imágenes, el nivel Premium agrega características tales como [replicación geográfica][container-registry-geo-replication] para la administración de un único registro en varias regiones, [confianza del contenido](container-registry-content-trust.md) para la firma de etiquetas de imagen, y [vínculo privado con puntos de conexión privados](container-registry-private-link.md) para restringir el acceso al Registro. |

Los niveles de servicio Básico, Estándar y Premium ofrecen las mismas funcionalidades de programación. También aprovechan el [almacenamiento de imágenes][container-registry-storage] administrado totalmente por Azure. La elección de un nivel de servicio superior aporta mayor rendimiento y escalabilidad. Con varios niveles de servicio, puede empezar a trabajar con el nivel Básico y, después, cambiar a Estándar y a Premium a medida que aumente su uso del registro.

## <a name="service-tier-features-and-limits"></a>Características y límites del nivel de servicio

En la tabla siguiente se detallan las características y los límites de registro de los niveles de servicio Básico, Estándar y Premium.

[!INCLUDE [container-instances-limits](../../includes/container-registry-limits.md)]

## <a name="changing-tiers"></a>Cambio de niveles de servicio

Puede cambiar el nivel de servicio de un registro con la CLI de Azure o en Azure Portal. Puede moverse libremente de un nivel de servicio a otro siempre que el nivel al que se vaya a cambiar tenga la capacidad de almacenamiento máximo necesaria. 

### <a name="azure-cli"></a>Azure CLI

Para moverse entre las los niveles de servicio en la CLI de Azure, use el comando [az acr update][az-acr-update]. Por ejemplo, para cambiar a Premium:

```azurecli
az acr update --name myregistry --sku Premium
```

### <a name="azure-portal"></a>Azure Portal

En la **información general** del registro de contenedor en Azure Portal, seleccione **Actualizar** y después seleccione una **SKU** nueva en la lista desplegable de SKU.

![Actualización de la SKU del registro de contenedor en Azure Portal][update-registry-sku]

## <a name="pricing"></a>Precios

Para más información sobre los precios de cada uno los niveles de servicio de Azure Container Registry, consulte [Precios de Container Registry][container-registry-pricing].

Para obtener más información acerca de los precios de las transferencias de datos, consulte [Detalles de precios de Bandwidth](https://azure.microsoft.com/pricing/details/bandwidth/). 

## <a name="next-steps"></a>Pasos siguientes

**Mapa de ruta de Azure Container Registry**

Visite el [mapa de ruta de ACR][acr-roadmap] en GitHub para obtener información sobre las próximas características del servicio.

**UserVoice en Azure Container Registry**

Envíe y vote las sugerencias de nuevas características de [UserVoice en ACR][container-registry-uservoice].

<!-- IMAGES -->
[update-registry-sku]: ./media/container-registry-skus/update-registry-sku.png

<!-- LINKS - External -->
[acr-roadmap]: https://aka.ms/acr/roadmap
[container-registry-pricing]: https://azure.microsoft.com/pricing/details/container-registry/
[container-registry-uservoice]: https://feedback.azure.com/forums/903958-azure-container-registry

<!-- LINKS - Internal -->
[az-acr-update]: /cli/azure/acr#az-acr-update
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-storage]: container-registry-storage.md
[container-registry-delete]: container-registry-delete.md
[container-registry-webhook]: container-registry-webhook.md
