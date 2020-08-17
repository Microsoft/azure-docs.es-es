---
title: Supervisión de la seguridad de los contenedores en Azure Security Center
description: Aprenda a comprobar la postura de seguridad de los contenedores desde Azure Security Center
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 06/30/2020
ms.author: memildin
ms.openlocfilehash: 5f558aaa367d156a8a4fa8633f113c6480236f5b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87038621"
---
# <a name="monitor-the-security-of-your-containers"></a>Supervisión de la seguridad de los contenedores

En esta página se explica cómo usar las características de seguridad del contenedor descritas en el [artículo sobre seguridad del contenedor](container-security.md) en nuestra sección de conceptos.


## <a name="scanning-your-arm-based-container-registries-for-vulnerabilities"></a>Examen de los registros de contenedor basados en ARM para detectar vulnerabilidades 

1. Para habilitar los análisis de vulnerabilidades de las imágenes de Azure Container Registry:

    1. Suscríbase a un plan de tarifa estándar de Azure Security Center.

    1. En la página **Precios y configuración**, habilite el paquete opcional de registros de contenedor para su suscripción: ![Habilitación del paquete de registros de contenedor](media/monitor-container-security/enabling-container-registries-bundle.png)

        Security Center ya está listo para examinar las imágenes que se insertan en el registro. 

        >[!NOTE]
        >Esta característica se cobra por imagen.


1. Para desencadenar el examen de una imagen, insértela en el repositorio. 

    Cuando finaliza el examen (al cabo de 2 minutos aproximadamente, aunque puede tardar hasta 15 minutos), los resultados están disponibles en recomendaciones de Security Center como esta:

1. Para ver los resultados, vaya a la página **Recomendaciones**. Si se detectan problemas, verá la siguiente recomendación:

    ![Recomendación para corregir problemas ](media/monitor-container-security/acr-finding.png)

1. Seleccione la recomendación. 
    Se abre la página de detalles de la recomendación con información adicional. Esta información incluye la lista de registros con imágenes vulnerables ("recursos afectados") y los pasos de corrección. 

1. Seleccione un registro específico para ver los repositorios que contienen repositorios vulnerables.

    ![Seleccione un registro.](media/monitor-container-security/acr-finding-select-registry.png)

    Se abre la página Detalles del registro con la lista de repositorios afectados.

1. Seleccione un repositorio específico para ver los repositorios que contienen imágenes vulnerables.

    ![Selección de un repositorio](media/monitor-container-security/acr-finding-select-repository.png)

    Se abre la página Detalles del repositorio. Muestra las imágenes vulnerables junto con una evaluación de la gravedad de las conclusiones.

1. Seleccione una imagen específica para ver las vulnerabilidades.

    ![Selección de imágenes](media/monitor-container-security/acr-finding-select-image.png)

    Se abre la lista de resultados de la imagen seleccionada.

    ![Lista de resultados](media/monitor-container-security/acr-findings.png)

1. Para más información sobre un resultado, selecciónelo. 

    Se abre el panel de detalles de los resultados.

    [![Panel de detalles de los resultados](media/monitor-container-security/acr-finding-details-pane.png)](media/monitor-container-security/acr-finding-details-pane.png#lightbox)

    En este panel se incluye una descripción detallada del problema y vínculos a recursos externos para ayudar a mitigar las amenazas.

1. Siga los pasos de la sección de corrección de este panel.

1. Cuando haya realizado los pasos necesarios para corregir el problema de seguridad, reemplace la imagen en el registro:

    1. Inserte la imagen actualizada. Esto desencadenará un examen. 
    
    1. Consulte en la página de recomendaciones la recomendación "Las vulnerabilidades de las imágenes de Azure Container Registry deben corregirse". 
    
        Si la recomendación sigue apareciendo y la imagen que ha corregido todavía aparece en la lista de imágenes vulnerables, vuelva a comprobar los pasos de corrección.

    1. Cuando esté seguro de que la imagen actualizada se ha insertado y examinado y que ya no aparece en la recomendación, elimine la imagen vulnerable "antigua" del registro.


## <a name="hardening-your-containers-docker-hosts"></a>Protección de los hosts de Docker de los contenedores

Security Center supervisa constantemente la configuración de los hosts de Docker y genera recomendaciones de seguridad que reflejan los estándares del sector.

Para ver las recomendaciones de seguridad de Azure Security Center para los hosts de Docker de los contenedores:

1. En la barra de navegación de Security Center, abra  **Proceso y aplicaciones** y seleccione la pestaña **Contenedores**.

1. Opcionalmente, filtre la lista de recursos de contenedor a los hosts del contenedor.

    ![Filtro de recursos de contenedor](media/monitor-container-security/container-resources-filter.png)

1. En la lista de las máquinas de host de contenedor, seleccione una para seguir investigando.

    ![Recomendaciones del host de contenedor](media/monitor-container-security/container-resources-filtered-to-hosts.png)

    Se abre la **página de información del host de contenedor** con los detalles del host y una lista de recomendaciones.

1. En la lista de recomendaciones, seleccione para seguir investigando.

    ![Lista de recomendaciones de host de contenedor](media/monitor-container-security/container-host-rec.png)

1. Opcionalmente, lea la descripción, la información, las amenazas y los pasos de corrección. 

1. Seleccione **Realizar acción** en la parte inferior de la página.

    [![Botón Realizar acción](media/monitor-container-security/host-security-take-action-button.png)](media/monitor-container-security/host-security-take-action.png#lightbox)

    Log Analytics se abre con una operación personalizada lista para ejecutarse. La consulta personalizada predeterminada incluye una lista de todas las reglas con errores que se han evaluado, junto con instrucciones para ayudarle a resolver los problemas.

    [![Acción de Log Analytics](media/monitor-container-security/log-analytics-for-action-small.png)](media/monitor-container-security/log-analytics-for-action.png#lightbox)

1. Retoque los parámetros de consulta si es necesario.

1. Cuando esté seguro de que el comando es adecuado y está listo para el host, seleccione **Ejecutar**.



## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a usar las características de seguridad de contenedor de Security Center. 

Para obtener material relacionado, consulte las páginas siguientes: 

- [Recomendaciones de Security Center para contenedores](recommendations-reference.md#recs-containers)
- [Alertas del nivel de clúster de AKS](alerts-reference.md#alerts-akscluster)
- [Alertas del nivel de host de contenedor](alerts-reference.md#alerts-containerhost)
