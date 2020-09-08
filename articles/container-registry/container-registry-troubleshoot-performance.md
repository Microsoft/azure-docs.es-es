---
title: Solución de problemas de rendimiento del registro
description: Síntomas, causas y resolución de problemas comunes con el rendimiento de un registro
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 414e3a92b8ebd4ff58528fc5e9ec4794471bd775
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226980"
---
# <a name="troubleshoot-registry-performance"></a>Solución de problemas de rendimiento del registro

Este artículo le ayuda a solucionar problemas que pueden surgir con el rendimiento de un registro de contenedor de Azure. 

## <a name="symptoms"></a>Síntomas

Puede encontrarse con uno o varios de los siguientes:

* Las imágenes de extracción o inserción con la CLI de Docker tardan más de lo esperado.
* La implementación de imágenes en un servicio como Azure Kubernetes Service tarda más de lo esperado.
* No es posible realizar un gran número de operaciones simultáneas de extracción o inserción en el tiempo esperado.
* Las operaciones de extracción o inserción en un registro con replicación geográfica tardan más de lo esperado, o bien se produce el error `Error writing blob` o `Error writing manifest` en la inserción

## <a name="causes"></a>Causas

* La velocidad de conexión de la red puede ralentizar las operaciones del registro: [solución](#check-expected-network-speed)
* La compresión o extracción de la capa de imagen puede ser lenta en el cliente: [solución](#check-client-hardware)  
* Está alcanzando un límite configurado en el nivel de servicio del registro o en el entorno: [solución](#review-configured-limits)
* El registro con replicación geográfica tiene réplicas en las regiones cercanas: [solución](#configure-geo-replicated-registry)
* Va a realizar una extracción de una réplica del registro que se encuentra geográficamente distante: [solución](#configure-dns-for-geo-replicated-registry)

Si no encuentra una solución para su problema aquí, consulte [Solución avanzada de problemas](#advanced-troubleshooting) y [Pasos siguientes](#next-steps) para examinar otras opciones.

## <a name="potential-solutions"></a>Posibles soluciones

### <a name="check-expected-network-speed"></a>Comprobar la velocidad de red esperada

Compruebe la velocidad de carga y descarga de Internet, o use una herramienta como AzureSpeed para probar la [cargar](https://www.azurespeed.com/Azure/Uploadß) y [descarga](https://www.azurespeed.com/Azure/Download) desde Azure Blob Storage, que hospeda las capas de la imagen del registro.

Compare el tamaño de la imagen con el tamaño máximo admitido y el ancho de banda admitido de carga y descarga correspondiente al nivel de servicio del registro. Si el registro se encuentra en el nivel básico o estándar, considere la posibilidad de actualizarlo para mejorar el rendimiento. 

Para la implementación de imágenes en otros servicios, compruebe las regiones en las que se encuentran el registro y el destino. Para mejorar el rendimiento, considere la posibilidad de ubicar el registro y el destino de la implementación en las mismas regiones o en otras cercanas a la red.

Vínculos relacionados:

* [Niveles del servicio Azure Container Registry](container-registry-skus.md)    
* [Preguntas frecuentes sobre Container Registry](container-registry-faq.md)
* [Objetivos de escalabilidad y rendimiento de Azure Blob Storage](../storage/blobs/scalability-targets.md)

### <a name="check-client-hardware"></a>Comprobar el hardware del cliente

El tipo de disco y la CPU del cliente de Docker pueden afectar a la velocidad de extracción o compresión de las capas de imagen en el cliente como parte de las operaciones de extracción o inserción. Por ejemplo, la extracción de capas en una unidad de disco duro tardará más que en un disco de estado sólido. Compare las operaciones de extracción en imágenes equiparables desde Azure Container Registry y desde un registro público, como Docker Hub.

### <a name="review-configured-limits"></a>Revisar los límites configurados

Si va a insertar o extraer a la vez muchas imágenes de varias capas en el registro, revise los límites admitidos de operaciones de lectura y escritura en el nivel de servicio del registro. Si el registro se encuentra en el nivel básico o estándar, considere la posibilidad de actualizarlo para aumentar los límites. Hable también con su proveedor de red sobre la limitación de la red que puede producirse con muchas operaciones simultáneas. 

Revise la configuración del demonio de Docker para ver el número máximo de cargas o descargas simultáneas con cada operación de inserción o extracción en el cliente. Si es necesario, configure límites más altos.

Dado que cada capa de imagen requiere una operación de lectura o escritura del registro independiente, compruebe el número de capas de las imágenes. Para reducir el número de capas de imagen, piense en algunas estrategias.

Vínculos relacionados:

* [Niveles del servicio Azure Container Registry](container-registry-skus.md)
* [dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/)

### <a name="configure-geo-replicated-registry"></a>Configurar el registro con replicación geográfica

Es posible que un cliente de Docker que inserta una imagen en un registro con replicación geográfica no inserte todas las capas de imagen y su manifiesto en una sola región replicada. Esta situación puede darse si Azure Traffic Manager enruta las solicitudes del registro al registro replicado más cercano a la red. Si el registro tiene dos regiones de replicación cercanas, las capas de imagen y el manifiesto se pueden distribuir a los dos sitios, y se produce un error en la operación de extracción cuando se valida el manifiesto.

Para optimizar la resolución DNS para la réplica más cercana al insertar imágenes, configure un registro con replicación geográfica en las mismas regiones de Azure que el origen de las operaciones de inserción o la región más cercana cuando trabaje fuera de Azure.

Para solucionar los problemas de operaciones con los registros con replicación geográfica, también puede deshabilitar temporalmente el enrutamiento de Traffic Manager a una o más replicaciones.

Vínculos relacionados:

* [Replicación geográfica en Azure Container Registry](container-registry-geo-replication.md)

### <a name="configure-dns-for-geo-replicated-registry"></a>Configurar DNS para el registro con replicación geográfica

Si las operaciones de extracción de un registro con replicación geográfica parecen lentas, la configuración de DNS en el cliente podría resolverse en un servidor DNS geográficamente distante. En este caso, puede que Traffic Manager esté enrutando las solicitudes a una réplica que está cercana a la red del servidor DNS, pero alejada del cliente. Ejecute una herramienta como `nslookup` o `dig` (en Linux) para determinar la réplica a la que Traffic Manager enruta las solicitudes del registro. Por ejemplo:

```console
nslookup myregistry.azurecr.io
```

Una posible solución consiste en configurar un servidor DNS más cercano.

Vínculos relacionados:

* [Replicación geográfica en Azure Container Registry](container-registry-geo-replication.md)
* [Solución de problemas de operaciones de inserción con registros con replicación geográfica](container-registry-geo-replication.md#troubleshoot-push-operations-with-geo-replicated-registries)
* [Deshabilitación temporal del enrutamiento a replicaciones](container-registry-geo-replication.md#temporarily-disable-routing-to-replication)
* [Preguntas frecuentes sobre Traffic Manager](../traffic-manager/traffic-manager-faqs.md)

### <a name="advanced-troubleshooting"></a>Pasos detallados para solucionar problemas de conexión a Escritorio remoto a máquinas virtuales Windows en Azure

Si los permisos para los recursos del registro lo permiten, [compruebe el estado del entorno del registro](container-registry-check-health.md). Si se notifican errores, revise la [referencia del error](container-registry-health-error-reference.md) para ver posibles soluciones.

Si la [colección de registros de recursos](container-registry-diagnostics-audit-logs.md) está habilitada en el registro, revise el registro ContainterRegistryRepositoryEvents. Este registro almacena información de operaciones, como eventos de inserción o extracción. Consulte el registro para conocer los [errores de operación de nivel de repositorio](container-registry-diagnostics-audit-logs.md#repository-level-operation-failures). 

Vínculos relacionados:

* [Registros para la evaluación y auditoría de diagnóstico](container-registry-diagnostics-audit-logs.md)
* [Preguntas frecuentes sobre Container Registry](container-registry-faq.md)
* [Procedimientos recomendados para Azure Container Registry](container-registry-best-practices.md)

## <a name="next-steps"></a>Pasos siguientes

Si no encuentra aquí una solución para su problema, vea las siguientes opciones.

* Otros temas de solución de problemas del registro incluyen:
  * [Solución de problemas de inicio de sesión del registro](container-registry-troubleshoot-login.md)
  * [Solución de problemas de red con el registro](container-registry-troubleshoot-access.md)
* Opciones de [soporte técnico de la comunidad](https://azure.microsoft.com/support/community/)
* [Preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/products/)
* [Abrir una incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/)


