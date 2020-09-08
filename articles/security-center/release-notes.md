---
title: Notas de la versión de Azure Security Center
description: Una descripción de las novedades y los cambios en Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/12/2020
ms.author: memildin
ms.openlocfilehash: c7df035aec199953bdf9a6bd56262af70a5a77e7
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723969"
---
# <a name="whats-new-in-azure-security-center"></a>Novedades de Azure Security Center

La seguridad de Azure está en desarrollo activo y recibe mejoras continuas. Para mantenerse al día con los avances más recientes, esta página proporciona información acerca de los elementos siguientes:

- Nuevas características
- Corrección de errores
- Funciones obsoletas

Esta página se actualiza regularmente, por lo que se recomienda visitarla a menudo. Si busca elementos de más de 6 meses, puede encontrarlos en las [Novedades de Azure Security Center](release-notes-archive.md).


## <a name="august-2020"></a>Agosto de 2020

Las actualizaciones de agosto incluyen:

- [Inventario de recursos: nueva vista eficaz de la posición de seguridad de sus recursos](#asset-inventory---powerful-new-view-of-the-security-posture-of-your-assets)
- [Compatibilidad agregada con los valores predeterminados de seguridad de Azure Active Directory (para la autenticación multifactor)](#added-support-for-azure-active-directory-security-defaults-for-multi-factor-authentication)
- [Recomendación de entidades de servicio agregada](#service-principals-recommendation-added)
- [Evaluación de vulnerabilidades en máquinas virtuales: recomendaciones y directivas consolidadas](#vulnerability-assessment-on-vms---recommendations-and-policies-consolidated)
- [Se han agregado nuevas directivas de seguridad de AKS a la iniciativa de ASC_default para su uso solo por parte de clientes de versión preliminar privada](#new-aks-security-policies-added-to-asc_default-initiative--for-use-by-private-preview-customers-only)


### <a name="asset-inventory---powerful-new-view-of-the-security-posture-of-your-assets"></a>Inventario de recursos: nueva vista eficaz de la posición de seguridad de sus recursos

El inventario de recursos de Security Center (actualmente en versión preliminar) proporciona una manera de ver la posición de seguridad de los recursos que ha conectado a Security Center.

Security Center analiza periódicamente el estado de seguridad de los recursos de Azure para identificar posibles puntos vulnerables de la seguridad. A continuación, se proporcionan recomendaciones sobre cómo corregir dichos puntos vulnerables. Cuando algún recurso tenga recomendaciones pendientes, aparecerán en el inventario.

Puede usar la vista y sus filtros para explorar los datos de la posición de seguridad y realizar acciones adicionales en función de los resultados.

Obtenga más información sobre el [inventario de recursos](asset-inventory.md).


### <a name="added-support-for-azure-active-directory-security-defaults-for-multi-factor-authentication"></a>Compatibilidad agregada con los valores predeterminados de seguridad de Azure Active Directory (para la autenticación multifactor)

Security Center ha agregado compatibilidad total con los [valores predeterminados de seguridad](https://docs.microsoft.com/azure/active-directory/fundamentals/concept-fundamentals-security-defaults), las protecciones de seguridad de la identidad gratuitas de Microsoft.

Los valores predeterminados de seguridad proporcionan una configuración de seguridad de la identidad preconfigurada para proteger su organización frente a ataques habituales relacionados con la identidad. Los valores predeterminados de seguridad ya protegen más de 5 millones de inquilinos en total; 50 000 inquilinos también reciben la protección de Security Center.

Security Center ahora proporciona una recomendación de seguridad cada vez que identifica una suscripción de Azure sin valores predeterminados de seguridad habilitados. Hasta ahora, Security Center recomendaba habilitar la autenticación multifactor mediante el acceso condicional, que forma parte de la licencia Premium de Azure Active Directory (AD). Para los clientes que usen Azure AD de forma gratuita, ahora recomendamos que se habiliten los valores predeterminados de seguridad. 

Nuestro objetivo es animar a más clientes a proteger sus entornos en la nube con MFA y mitigar uno de los mayores riesgos que también es el que tiene más impacto en su [puntuación de seguridad](https://docs.microsoft.com/azure/security-center/secure-score-security-controls).

Obtenga más información sobre los [valores predeterminados de seguridad](https://docs.microsoft.com/azure/active-directory/fundamentals/concept-fundamentals-security-defaults).


### <a name="service-principals-recommendation-added"></a>Recomendación de entidades de servicio agregada

Se ha agregado una nueva recomendación para recomendar que los clientes de Security Center que usan certificados de administración para administrar sus suscripciones cambien a las entidades de servicio.

La recomendación, **Para proteger las suscripciones,deben usarse entidades de servicio en lugar de certificados de administración**, le aconseja usar entidades de servicio o Azure Resource Manager para administrar de forma más segura sus suscripciones. 

Obtenga más información sobre [Objetos de aplicación y de entidad de servicio de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals#service-principal-object).


### <a name="vulnerability-assessment-on-vms---recommendations-and-policies-consolidated"></a>Evaluación de vulnerabilidades en máquinas virtuales: recomendaciones y directivas consolidadas

Security Center inspecciona las máquinas virtuales para detectar si ejecutan alguna solución de valoración de las vulnerabilidades. Si no encuentra ninguna solución, Security Center recomienda simplificar la implementación.

Si se encuentran vulnerabilidades, Security Center realiza una recomendación en la que se resumen los resultados que se deben investigar y corregir, en caso de que sea necesario.

Para garantizar una experiencia coherente para todos los usuarios, independientemente del tipo de analizador que usen, se han unificado cuatro recomendaciones en las dos siguientes:

|Recomendación unificada|Descripción del cambio|
|----|:----|
|**Debe habilitarse una solución de evaluación de vulnerabilidades en sus máquinas virtuales**|Reemplaza las dos recomendaciones siguientes:<br> **•** Habilitar la solución de evaluación de vulnerabilidades integrada en las máquinas virtuales (con tecnología de Qualys) (ahora en desuso) (se incluye con el nivel Estándar).<br> **•** La solución de evaluación de vulnerabilidades debe instalarse en sus máquinas virtuales (ahora en desuso) (niveles gratuito y Estándar).|
|**Es necesario corregir las vulnerabilidades de las máquinas virtuales**|Reemplaza las dos recomendaciones siguientes:<br>**•** Corregir las vulnerabilidades que se encontraron en las máquinas virtuales (con tecnología de Qualys) (ahora en desuso).<br>**•** Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades (ahora en desuso).|
|||

Ahora usará la misma recomendación para implementar la extensión de evaluación de vulnerabilidades de Security Center o una solución con licencia privada ("BYOL") de un asociado como Qualys o Rapid7.

Además, cuando se detectan vulnerabilidades y se notifican a Security Center, una sola recomendación le avisará de los hallazgos, independientemente de la solución de evaluación de vulnerabilidades que los haya identificado.

#### <a name="updating-dependencies"></a>Actualización de dependencias

Si tiene scripts, consultas o automatizaciones que hacen referencia a las recomendaciones anteriores o a nombres o claves de directivas, use las tablas siguientes para actualizar las referencias:

##### <a name="before-august-2020"></a>Antes de agosto de 2020

|Recomendación|Ámbito|
|----|:----|
|**Habilitar la solución de evaluación de vulnerabilidades integrada en las máquinas virtuales (con tecnología de Qualys)**<br>Clave: 550e890b-e652-4d22-8274-60b3bdb24c63|Integrada|
|**Corregir las vulnerabilidades que se encontraron en las máquinas virtuales (con tecnología de Qualys)**<br>Clave: 1195afff-c881-495e-9bc5-1486211ae03f|Integrada|
|**La solución de evaluación de vulnerabilidades debe instalarse en sus máquinas virtuales**<br>Clave: 01b1ed4c-b733-4fee-b145-f23236e70cf3|BYOL|
|**Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades**<br>Clave: 71992a2a-d168-42e0-b10e-6b45fa2ecddb|BYOL|
||||


|Directiva|Ámbito|
|----|:----|
|**La evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales**<br>Id. de directiva: 501541f7-f7e7-4cd6-868c-4190fdad3ac9|Integrada|
|**Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades**<br>Id. de directiva: 760a85ff-6162-42b3-8d70-698e268f648c|BYOL|
||||


##### <a name="from-august-2020"></a>Desde agosto de 2020

|Recomendación|Ámbito|
|----|:----|
|**Debe habilitarse una solución de evaluación de vulnerabilidades en sus máquinas virtuales**<br>Clave: ffff0522-1e88-47fc-8382-2a80ba848f5d|Integrada + BYOL|
|**Es necesario corregir las vulnerabilidades de las máquinas virtuales**<br>Clave: 1195afff-c881-495e-9bc5-1486211ae03f|Integrada + BYOL|
||||

|Directiva|Ámbito|
|----|:----|
|[**La evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales**](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f501541f7-f7e7-4cd6-868c-4190fdad3ac9)<br>Id. de directiva: 501541f7-f7e7-4cd6-868c-4190fdad3ac9 |Integrada + BYOL|
||||


### <a name="new-aks-security-policies-added-to-asc_default-initiative--for-use-by-private-preview-customers-only"></a>Se han agregado nuevas directivas de seguridad de AKS a la iniciativa de ASC_default para su uso solo por parte de clientes de versión preliminar privada

Para asegurarse de que las cargas de trabajo de Kubernetes son seguras de forma predeterminada, Security Center está agregando directivas en el nivel de Kubernetes y recomendaciones de protección, incluidas las opciones de cumplimiento con el control de admisión de Kubernetes.

La fase temprana de este proyecto incluye una versión preliminar privada y la adición de nuevas directivas (deshabilitadas de forma predeterminada) a la iniciativa ASC_default.

Puede omitir estas directivas con seguridad y no habrá ningún impacto en el entorno. Si desea habilitarlas, regístrese para obtener la versión preliminar en https://aka.ms/SecurityPrP y seleccione una de las opciones siguientes:

1. **Versión preliminar única**: para unirse solo a esta versión preliminar privada. Mencione explícitamente "ASC Continuous SCAN" como la versión preliminar a la que le gustaría unirse.
1. **Programa continuo**: para agregarse a esta y a futuras versiones preliminares privadas. Deberá completar un perfil y un contrato de privacidad.


## <a name="july-2020"></a>Julio de 2020

Las actualizaciones de julio incluyen:
- [Evaluación de vulnerabilidades de las máquinas virtuales disponible ahora para las imágenes que no son de Marketplace](#vulnerability-assessment-for-virtual-machines-is-now-available-for-non-marketplace-images)
- [Protección contra amenazas para Azure Storage expandido con el fin de incluir Azure Files y Azure Data Lake Storage Gen2 (versión preliminar)](#threat-protection-for-azure-storage-expanded-to-include-azure-files-and-azure-data-lake-storage-gen2-preview)
- [Ocho nuevas recomendaciones para habilitar las características de protección contra amenazas](#eight-new-recommendations-to-enable-threat-protection-features)
- [Mejoras en la seguridad de los contenedores: análisis del registro más rápido y documentación actualizada](#container-security-improvements---faster-registry-scanning-and-refreshed-documentation)
- [Controles de aplicaciones adaptables actualizados con una nueva recomendación y compatibilidad para caracteres comodín en reglas de ruta de acceso](#adaptive-application-controls-updated-with-a-new-recommendation-and-support-for-wildcards-in-path-rules)
- [Novedades de Azure Security Center](#six-policies-for-sql-advanced-data-security-deprecated)




### <a name="vulnerability-assessment-for-virtual-machines-is-now-available-for-non-marketplace-images"></a>Evaluación de vulnerabilidades de las máquinas virtuales disponible ahora para las imágenes que no son de Marketplace

Anteriormente, al implementar una solución de evaluación de vulnerabilidades, Security Center realizaba una comprobación de validación antes de la implementación. La comprobación se realizaba para confirmar la SKU de Marketplace de la máquina virtual de destino. 

A partir de esta actualización, se ha eliminado esta comprobación y así ahora se pueden implementar herramientas de evaluación de vulnerabilidades en máquinas Windows y Linux personalizadas. Las imágenes personalizadas son aquellas que se han modificado a partir de los valores predeterminados de Marketplace.

Aunque ahora puede implementar la extensión de evaluación de vulnerabilidades integrada (con tecnología de Qualys) en muchas más máquinas, la compatibilidad solo está disponible si usa uno de los sistemas operativos que se indica en [Implementación del detector de vulnerabilidades integrado de Qualys](built-in-vulnerability-assessment.md#deploying-the-qualys-built-in-vulnerability-scanner).

Más información sobre [Detector de vulnerabilidades integrado para máquinas virtuales (solo nivel Estándar)](built-in-vulnerability-assessment.md).

Más información sobre el uso de su propia solución de evaluación de vulnerabilidades con licencia privada de Qualys o Rapid7 en [Implementación de una solución de examen de vulnerabilidades de asociados](partner-vulnerability-assessment.md).


### <a name="threat-protection-for-azure-storage-expanded-to-include-azure-files-and-azure-data-lake-storage-gen2-preview"></a>Protección contra amenazas para Azure Storage expandido con el fin de incluir Azure Files y Azure Data Lake Storage Gen2 (versión preliminar)

La protección contra amenazas para Azure Storage detecta actividades potencialmente dañinas en las cuentas de Azure Storage. Security Center muestra alertas cuando detecta intentos de acceder a las cuentas de almacenamiento o de aprovecharlas. 

Los datos se pueden proteger tanto si se almacenan como contenedores de blobs, recursos compartidos de archivos o lagos de datos. 

Obtenga más información sobre la [protección contra amenazas para Azure Storage](threat-protection.md#threat-protection-for-azure-storage-).




### <a name="eight-new-recommendations-to-enable-threat-protection-features"></a>Ocho nuevas recomendaciones para habilitar las características de protección contra amenazas

Se han agregado ocho nuevas recomendaciones para proporcionar una manera sencilla de habilitar las características de protección contra amenazas de Azure Security Center para los siguientes tipos de recursos: máquinas virtuales, planes de App Service, servidores de Azure SQL Database, servidores SQL Server en máquinas, cuentas de Azure Storage, clústeres de Azure Kubernetes Service, registros de Azure Container Registry y almacenes de Azure Key Vault.

Las nuevas recomendaciones son:

- **Advanced Data Security debe estar habilitado en los servidores de Azure SQL Database**
- **Advanced Data Security debe estar habilitado en los servidores SQL Server en las máquinas**
- **Advanced Threat Protection debe estar habilitado en los planes de Azure App Service**
- **Advanced Threat Protection debe estar habilitado en los registros de Azure Container Registry**
- **Advanced Threat Protection debe estar habilitado en los almacenes de Azure Key Vault**
- **Advanced Threat Protection debe estar habilitado en los clústeres de Azure Kubernetes Service**
- **Advanced Threat Protection debe estar habilitado en las cuentas de almacenamiento de Azure**
- **Advanced Threat Protection debe estar habilitado en Virtual Machines**

Estas nuevas recomendaciones pertenecen al control de seguridad **Activar Protección contra amenazas avanzada**.

Las recomendaciones también incluyen la funcionalidad de corrección rápida. 

> [!IMPORTANT]
> La corrección de cualquiera de estas recomendaciones dará lugar a cargos por la protección de los recursos pertinentes. Estos cargos se iniciarán de forma inmediata si tiene recursos relacionados en la suscripción actual. O en el futuro, si los agrega en una fecha posterior.
> 
> Por ejemplo, si no tiene ningún clúster de Azure Kubernetes Service en su suscripción y habilita la protección contra amenazas, no se aplicarán cargos. Si, en el futuro, agrega un clúster a la misma suscripción, este se protegerá automáticamente y a partir de ese momento, se iniciarán los cargos.

Más información sobre cada una de estas recomendaciones en la [página de referencia de las recomendaciones de seguridad](recommendations-reference.md).

Más información sobre la [protección contra amenazas en Azure Security Center](https://docs.microsoft.com/azure/security-center/threat-protection).




### <a name="container-security-improvements---faster-registry-scanning-and-refreshed-documentation"></a>Mejoras en la seguridad de los contenedores: análisis del registro más rápido y documentación actualizada

Como parte de las inversiones continuas en el dominio de la seguridad de los contenedores, nos complace compartir una mejora significativa del rendimiento en los análisis dinámicos de las imágenes de contenedor de Security Center que se almacenan en Azure Container Registry. Los exámenes se completan ahora en aproximadamente dos minutos. En algunos casos, pueden tardar hasta 15 minutos.

Para mejorar la claridad y la orientación con respecto a las funcionalidades de seguridad de los contenedores de Azure Security Center, también hemos actualizado las páginas de documentación sobre la seguridad de los contenedores. 

Más información sobre la seguridad de los contenedores de Security Center en los artículos siguientes:

- [Información general sobre las características de seguridad de los contenedores de Azure Security Center](https://docs.microsoft.com/azure/security-center/container-security)
- [Detalles de la integración con Azure Container Registry](https://docs.microsoft.com/azure/security-center/azure-container-registry-integration)
- [Detalles de la integración con Azure Kubernetes Service](https://docs.microsoft.com/azure/security-center/azure-kubernetes-service-integration)
- [Procedimientos para el examen de los registros y la protección de los hosts de Docker](https://docs.microsoft.com/azure/security-center/monitor-container-security)
- [Alertas de seguridad de las características de protección contra amenazas para clústeres de Azure Kubernetes Service](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-akscluster)
- [Alertas de seguridad de las características de protección contra amenazas para hosts de Azure Kubernetes Service](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-containerhost)
- [Recomendaciones de seguridad para contenedores](https://docs.microsoft.com/azure/security-center/recommendations-reference#recs-containers)



### <a name="adaptive-application-controls-updated-with-a-new-recommendation-and-support-for-wildcards-in-path-rules"></a>Controles de aplicaciones adaptables actualizados con una nueva recomendación y compatibilidad para caracteres comodín en reglas de ruta de acceso

La característica de controles de aplicaciones adaptables ha recibido dos actualizaciones importantes:

* Una nueva recomendación identifica un comportamiento potencialmente legítimo que no se ha permitido previamente. La nueva recomendación, **Se deben actualizar las reglas de la lista de permitidos de la directiva de controles de aplicaciones adaptables**, le pedirá que agregue nuevas reglas a la directiva existente para reducir el número de falsos positivos en las alertas de infracción de controles de aplicaciones adaptables.

* Las reglas de ruta de acceso ahora admiten caracteres comodín. A partir de esta actualización, puede configurar las reglas de ruta de acceso permitidas mediante caracteres comodín. Se admiten dos escenarios:

    * El uso de un carácter comodín al final de una ruta de acceso para permitir todos los ejecutables dentro de esta carpeta y subcarpetas

    * El uso de un carácter comodín en medio de una ruta de acceso para habilitar un nombre ejecutable conocido con un nombre de carpeta cambiante (por ejemplo, carpetas de usuario personales con un archivo ejecutable conocido, nombres de carpeta generados automáticamente, etc.).


[Más información acerca de los controles de aplicación adaptables](security-center-adaptive-application.md).



### <a name="six-policies-for-sql-advanced-data-security-deprecated"></a>Seis directivas para la seguridad avanzada de datos de SQL en desuso

Seis directivas relacionadas con la seguridad avanzada de datos para máquinas SQL están en desuso:

- Los tipos de Advanced Threat Protection deben definirse como "Todos" en la configuración de Advanced Data Security para SQL Managed Instance.
- Los tipos de Advanced Threat Protection deben definirse como "Todos" en la configuración de Advanced Data Security del servidor SQL Server.
- La configuración de seguridad de datos avanzada para una instancia administrada SQL debe contener una dirección de correo electrónico para recibir alertas de seguridad
- La configuración de seguridad de datos avanzada para SQL Server debe contener una dirección de correo electrónico para recibir alertas de seguridad
- Las notificaciones a los administradores y a los propietarios de la suscripción deben estar habilitadas en la configuración de seguridad avanzada de datos de la instancia administrada de SQL
- Las notificaciones por correo electrónico para administradores y propietarios de suscripciones deben estar habilitadas en la configuración de seguridad avanzada de datos de SQL Server

Obtenga más información sobre las [directivas integradas](security-center-policy-definitions.md).





## <a name="june-2020"></a>Junio de 2020

Las actualizaciones de junio incluyen:
- [API de puntuación segura (versión preliminar)](#secure-score-api-preview)
- [Advanced Data Security para máquinas SQL (Azure, otras nubes e instancias locales) (versión preliminar)](#advanced-data-security-for-sql-machines-azure-other-clouds-and-on-prem-preview)
- [Dos nuevas recomendaciones para implementar el agente de Log Analytics en máquinas de Azure Arc (versión preliminar)](#two-new-recommendations-to-deploy-the-log-analytics-agent-to-azure-arc-machines-preview)
- [Nuevas directivas para crear configuraciones de exportación continua y de automatización de flujos de trabajo a escala](#new-policies-to-create-continuous-export-and-workflow-automation-configurations-at-scale)
- [Nueva recomendación para usar NSG para proteger las máquinas virtuales sin conexión a Internet](#new-recommendation-for-using-nsgs-to-protect-non-internet-facing-virtual-machines)
- [Nuevas directivas para habilitar la protección contra amenazas y Advanced Data Security](#new-policies-for-enabling-threat-protection-and-advanced-data-security)



### <a name="secure-score-api-preview"></a>API de puntuación segura (versión preliminar)

Ahora puede acceder a la puntuación a través de la [API de puntuación segura](https://docs.microsoft.com/rest/api/securitycenter/securescores/) (actualmente en versión preliminar). Los métodos de API proporcionan la flexibilidad necesaria para consultar los datos y crear su propio mecanismo de creación de informes de las puntuaciones seguras a lo largo del tiempo. Por ejemplo, puede usar la API **Secure Scores** para obtener la puntuación de una suscripción específica. Además, puede usar la API **Secure Score Controls** para mostrar los controles de seguridad y la puntuación actual de las suscripciones.

Para ver ejemplos de herramientas externas que son posibles gracias a la API de puntuación segura, vea el [área de puntuación segura de la comunidad de GitHub](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score).

Obtenga más información sobre la [puntuación segura y los controles de seguridad en Azure Security Center](secure-score-security-controls.md).



### <a name="advanced-data-security-for-sql-machines-azure-other-clouds-and-on-prem-preview"></a>Advanced Data Security para máquinas SQL (Azure, otras nubes e instancias locales) (versión preliminar)

Ahora, Advanced Data Security de Azure Security Center para máquinas SQL protege los servidores SQL Server hospedados en Azure, en otros entornos en la nube e incluso en máquinas locales. Esto amplía las protecciones de los servidores SQL Server nativos de Azure, de modo que se admitan totalmente los entornos híbridos.

Advanced Data Security proporciona una evaluación de vulnerabilidades y protección contra amenazas avanzada para máquinas SQL dondequiera que se encuentren.

La configuración conlleva dos pasos:

1. Implementación del agente de Log Analytics en la máquina host de SQL Server para proporcionar la conexión a la cuenta de Azure.

1. Habilitación del conjunto de productos opcional en la página de precios y configuración de Security Center.

Obtenga más información sobre [Advanced Data Security para máquinas SQL](security-center-iaas-advanced-data.md).



### <a name="two-new-recommendations-to-deploy-the-log-analytics-agent-to-azure-arc-machines-preview"></a>Dos nuevas recomendaciones para implementar el agente de Log Analytics en máquinas de Azure Arc (versión preliminar)

Se han agregado dos nuevas recomendaciones para ayudar a implementar el [agente de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/log-analytics-agent) en las máquinas de Azure Arc y asegurarse de que están protegidas por Azure Security Center:

- **El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Windows (versión preliminar)** .
- **El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Linux (versión preliminar)** .

Estas nuevas recomendaciones aparecerán en los mismos cuatro controles de seguridad que la recomendación existente (relacionada) **El agente de supervisión debe estar instalado en las máquinas**: corrija las configuraciones de seguridad, aplique el control de aplicaciones adaptativo, aplique actualizaciones del sistema y habilite Endpoint Protection.

Las recomendaciones también incluyen la función de corrección rápida para ayudar a acelerar el proceso de implementación. 

Obtenga más información sobre estas dos nuevas recomendaciones en la tabla [Recomendaciones de proceso y aplicación](recommendations-reference.md#recs-computeapp).

Obtenga más información sobre la manera en que Azure Security Center usa el agente en [¿Qué es el agente de Log Analytics?](https://docs.microsoft.com/azure/security-center/faq-data-collection-agents#what-is-the-log-analytics-agent)

Obtenga más información sobre las [extensiones para máquinas de Azure Arc](https://docs.microsoft.com/azure/azure-arc/servers/manage-vm-extensions#enable-extensions-from-the-portal).



### <a name="new-policies-to-create-continuous-export-and-workflow-automation-configurations-at-scale"></a>Nuevas directivas para crear configuraciones de exportación continua y de automatización de flujos de trabajo a escala

La automatización de los procesos de supervisión y respuesta ante incidentes de la organización puede mejorar considerablemente el tiempo necesario para investigar y mitigar los incidentes de seguridad.

Para implementar las configuraciones de automatización en la organización, use estas directivas de Azure "DeployIfdNotExist" integradas para crear y configurar los procedimientos de [exportación continua](continuous-export.md) y [automatización de flujos de trabajo](workflow-automation.md):

Las directivas se pueden encontrar en la directiva de Azure:


|Objetivo  |Directiva  |Id. de directiva  |
|---------|---------|---------|
|Exportación continua al centro de eventos|[Implementar la exportación al centro de eventos para las alertas y recomendaciones de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fcdfcce10-4578-4ecd-9703-530938e4abcb)|cdfcce10-4578-4ecd-9703-530938e4abcb|
|Exportación continua a las áreas de trabajo de Log Analytics|[Implementar la exportación al área de trabajo de Log Analytics para las alertas y recomendaciones de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fffb6f416-7bd2-4488-8828-56585fef2be9)|ffb6f416-7bd2-4488-8828-56585fef2be9|
|Automatización de flujos de trabajo para alertas de seguridad|[Implementar la automatización del flujo de trabajo para las alertas de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2ff1525828-9a90-4fcf-be48-268cdd02361e)|f1525828-9a90-4fcf-be48-268cdd02361e|
|Automatización de flujos de trabajo para recomendaciones de seguridad|[Implementar la automatización del área de trabajo para las recomendaciones de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f73d6ab6c-2475-4850-afd6-43795f3492ef)|73d6ab6c-2475-4850-afd6-43795f3492ef|
||||

Empiece a usar las [plantillas de automatización de flujos de trabajo](https://github.com/Azure/Azure-Security-Center/tree/master/Workflow%20automation).

Obtenga más información sobre el uso de las dos directivas de exportación en [Exportación continua de alertas y recomendaciones de Azure Security Center alertas a través de directivas](https://techcommunity.microsoft.com/t5/azure-security-center/continuously-export-azure-security-center-alerts-and/ba-p/1440745).


### <a name="new-recommendation-for-using-nsgs-to-protect-non-internet-facing-virtual-machines"></a>Nueva recomendación para usar NSG para proteger las máquinas virtuales sin conexión a Internet

El control de seguridad "implementar prácticas recomendadas de seguridad" incluye ahora la siguiente recomendación nueva:

- **Las máquinas virtuales sin conexión a Internet deben protegerse con grupos de seguridad de red**

Una recomendación existente (**Las máquinas virtuales sin conexión a Internet deben protegerse con grupos de seguridad de red**) no distinguía entre las máquinas virtuales con y sin conexión a Internet. En ambos casos, se generaba una recomendación de alta gravedad si una máquina virtual no estaba asignada a un grupo de seguridad de red. Esta nueva recomendación separa las máquinas sin conexión a Internet para reducir los falsos positivos y evitar alertas innecesarias de alta gravedad.

Obtenga más información en la tabla [Recomendaciones de red](recommendations-reference.md#recs-network).




### <a name="new-policies-for-enabling-threat-protection-and-advanced-data-security"></a>Nuevas directivas para habilitar la protección contra amenazas y Advanced Data Security

Las nuevas directivas que se indican a continuación se agregaron a la iniciativa predeterminada de ASC y están diseñadas para ayudar a habilitar la protección contra amenazas o Advanced Data Security para los tipos de recursos pertinentes.

Las directivas se pueden encontrar en la directiva de Azure:


| Directiva                                                                                                                                                                                                                                                                | Id. de directiva                            |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
| [Advanced Data Security debe estar habilitado en los servidores de Azure SQL Database](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f7fe3b40f-802b-4cdd-8bd4-fd799c948cc2)     | 7fe3b40f-802b-4cdd-8bd4-fd799c948cc2 |
| [Advanced Data Security debe estar habilitado en los servidores SQL Server en las máquinas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f6581d072-105e-4418-827f-bd446d56421b) | 6581d072-105e-4418-827f-bd446d56421b |
| [Advanced Threat Protection debe estar habilitado en las cuentas de almacenamiento de Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f308fbb08-4ab8-4e67-9b29-592e93fb94fa)           | 308fbb08-4ab8-4e67-9b29-592e93fb94fa |
| [Advanced Threat Protection debe estar habilitado en los almacenes de Azure Key Vault](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f0e6763cc-5078-4e64-889d-ff4d9a839047)           | 0e6763cc-5078-4e64-889d-ff4d9a839047 |
| [Advanced Threat Protection debe estar habilitado en los planes de Azure App Service](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f2913021d-f2fd-4f3d-b958-22354e2bdbcb)                | 2913021d-f2fd-4f3d-b958-22354e2bdbcb |
| [Advanced Threat Protection debe estar habilitado en los registros de Azure Container Registry](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fc25d9a16-bc35-4e15-a7e5-9db606bf9ed4)   | c25d9a16-bc35-4e15-a7e5-9db606bf9ed4 |
| [Advanced Threat Protection debe estar habilitado en los clústeres de Azure Kubernetes Service](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f523b5cd1-3e23-492f-a539-13118b6d1e3a)   | 523b5cd1-3e23-492f-a539-13118b6d1e3a |
| [Advanced Threat Protection debe estar habilitado en Virtual Machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f4da35fc9-c9e7-4960-aec9-797fe7d9051d)           | 4da35fc9-c9e7-4960-aec9-797fe7d9051d |
|                                                                                                                                                                                                                                                                       |                                      |

Obtenga más información sobre la [protección contra amenazas en Azure Security Center](https://docs.microsoft.com/azure/security-center/threat-protection).





## <a name="may-2020"></a>Mayo de 2020

Las actualizaciones de mayo incluyen:
- [Reglas de supresión de alertas (versión preliminar)](#alert-suppression-rules-preview)
- [La evaluación de vulnerabilidades de la máquina virtual ya está disponible con carácter general](#virtual-machine-vulnerability-assessment-is-now-generally-available)
- [Cambios en el acceso a la máquina virtual (VM) Just-in-Time (JIT)](#changes-to-just-in-time-jit-virtual-machine-vm-access)
- [Las recomendaciones personalizadas se han migrado a un control de seguridad independiente](#custom-recommendations-have-been-moved-to-a-separate-security-control)
- [Alternancia agregada para ver las recomendaciones en controles o como una lista plana](#toggle-added-to-view-recommendations-in-controls-or-as-a-flat-list)
- [Control de seguridad expandido "implementar prácticas recomendadas de seguridad"](#expanded-security-control-implement-security-best-practices)
- [Las directivas personalizadas con metadatos personalizados ya están disponibles con carácter general](#custom-policies-with-custom-metadata-are-now-generally-available)
- [Migración de funcionalidades de análisis de volcado de memoria a detección de ataques sin archivos](#crash-dump-analysis-capabilities-migrating-to-fileless-attack-detection)


### <a name="alert-suppression-rules-preview"></a>Reglas de supresión de alertas (versión preliminar)

Esta nueva característica (actualmente en versión preliminar) ayuda a reducir la fatiga de la alerta. Use reglas para ocultar automáticamente las alertas que se sabe que son inocuas o relacionadas con las actividades normales de su organización. Esto le permite centrarse en las amenazas más importantes. 

Todavía se generarán alertas que coincidan con las reglas de supresión habilitadas, pero su estado se establecerá en descartado. Puede ver el estado en el Azure Portal o tener acceso a las alertas de seguridad de Security Center.

Las reglas de supresión definen los criterios para los que se deben descartar automáticamente las alertas. Normalmente, se usaría una regla de supresión para:

- Eliminar las alertas identificadas como falsos positivos

- suprimir las alertas que se desencadenan con demasiada frecuencia para ser útiles

Más información sobre la [supresión de alertas de protección contra amenazas de Azure Security Center](alerts-suppression-rules.md).


### <a name="virtual-machine-vulnerability-assessment-is-now-generally-available"></a>La evaluación de vulnerabilidades de la máquina virtual ya está disponible con carácter general

El nivel estándar de Security Center ahora incluye una evaluación de vulnerabilidades integrada para máquinas virtuales sin precio adicional. Esta extensión está basada en Qualys pero notifica sus hallazgos directamente de nuevo a Security Center. No se necesita ninguna licencia ni cuenta de Qualys, ya que todo se administra sin problemas en Security Center.

La nueva solución puede examinar continuamente las máquinas virtuales para encontrar vulnerabilidades y presentar las conclusiones en Security Center. 

Para implementar la solución, use la nueva recomendación de seguridad:

"Habilitar la solución de evaluación de vulnerabilidades integrada en las máquinas virtuales (con tecnología de Qualys)"

Más información sobre la [evaluación de vulnerabilidades integradas de Security Center para máquinas virtuales](built-in-vulnerability-assessment.md).



### <a name="changes-to-just-in-time-jit-virtual-machine-vm-access"></a>Cambios en el acceso a la máquina virtual (VM) Just-in-Time (JIT)

Security Center incluye una característica opcional para proteger los puertos de administración de las máquinas virtuales. Esto proporciona una defensa contra la forma más común de ataques por fuerza bruta.

Esta actualización lleva a cabo los siguientes cambios en esta característica:

- Se ha cambiado el nombre de la recomendación que le aconseja habilitar JIT en una máquina virtual. Anteriormente, "el control de acceso a red Just-in-Time se debe aplicar a las máquinas virtuales" ahora es: "Los puertos de administración de las máquinas virtuales deben protegerse con el control de acceso de red Just-In-Time".

- La recomendación se desencadena solo si hay puertos de administración abiertos.

Más información acerca de [la característica de acceso JIT](security-center-just-in-time.md).


### <a name="custom-recommendations-have-been-moved-to-a-separate-security-control"></a>Las recomendaciones personalizadas se han migrado a un control de seguridad independiente

Uno de los controles de seguridad incluidos con la puntuación segura mejorada fue "Implementación de procedimientos recomendados de seguridad". Las recomendaciones personalizadas creadas para las suscripciones se colocan automáticamente en ese control. 

Para que sea más fácil encontrar las recomendaciones personalizadas, las hemos pasado a un control de seguridad dedicado, "recomendaciones personalizadas". Este control no afecta a la puntuación segura.

Más información sobre los controles de seguridad en [puntuación de seguridad mejorada (versión preliminar) de Azure Security Center](secure-score-security-controls.md).


### <a name="toggle-added-to-view-recommendations-in-controls-or-as-a-flat-list"></a>Alternancia agregada para ver las recomendaciones en controles o como una lista plana

Los controles de seguridad son grupos lógicos de recomendaciones de seguridad relacionadas. Reflejan las superficies de ataque vulnerables. Un control es un conjunto de recomendaciones de seguridad con instrucciones que le ayudan a implementar esas recomendaciones.

Para ver de inmediato cómo protege su organización cada superficie de ataque individual, revise las puntuaciones de cada control de seguridad.

De forma predeterminada, las recomendaciones se muestran en los controles de seguridad. A partir de esta actualización también se pueden mostrar como una lista. Para verlos como una lista simple ordenada por el estado de mantenimiento de los recursos afectados, use el nuevo comando de alternancia "agrupar por controles". El comando de alternancia está encima de la lista en el portal.

Los controles de seguridad y esta alternancia forman parte de la nueva experiencia de puntuación segura. No se olvide de enviarnos sus comentarios desde dentro del portal.

Más información sobre los controles de seguridad en [puntuación de seguridad mejorada (versión preliminar) de Azure Security Center](secure-score-security-controls.md).

![Alternancia "agrupar por controles" para recomendaciones](\media\secure-score-security-controls\recommendations-group-by-toggle.gif)

### <a name="expanded-security-control-implement-security-best-practices"></a>Control de seguridad expandido "implementar prácticas recomendadas de seguridad" 

Uno de los controles de seguridad que incluye con la puntuación segura mejorada es "Implementación de procedimientos recomendados de seguridad". Cuando una recomendación está en este control, no afecta a la puntuación segura. 

Con esta actualización, se han sacado tres recomendaciones de los controles en los que se colocaron originalmente y en este control de prácticas recomendadas. Hemos realizado este paso porque hemos determinado que el riesgo de estas tres recomendaciones es menor de lo que se pensó inicialmente.

Además, se han incorporado dos nuevas recomendaciones y se han agregado a este control.

Las tres recomendaciones que se movieron son:

- **MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción** (originalmente en el control "habilitar MFA")
- **Las cuentas externas con permisos de lectura deben quitarse de la suscripción** (originalmente en el control "administrar acceso y permisos")
- **Se debe designar un máximo de 3 propietarios para la suscripción** (originalmente en el control "administrar acceso y permisos")

Las dos nuevas recomendaciones que se han agregado al control son:

- **La extensión de configuración de invitado debe instalarse en máquinas virtuales Windows (versión preliminar):** el uso de la [configuración de invitado de Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/guest-configuration) proporciona visibilidad dentro de las máquinas virtuales a la configuración de servidor y de aplicación (solo Windows).

- **Protección contra vulnerabilidades de seguridad de Windows Defender debe estar habilitada en las máquinas (versión preliminar)** : Protección contra vulnerabilidades de seguridad de Windows Defender aprovecha el agente de configuración de invitado de Azure Policy. La protección contra vulnerabilidades de seguridad tiene cuatro componentes diseñados para bloquear dispositivos en una amplia variedad de vectores de ataque y comportamientos de bloque utilizados habitualmente en ataques de malware, al tiempo que permiten a las empresas equilibrar los requisitos de productividad y riesgo de seguridad (solo Windows).

Más información sobre la protección contra vulnerabilidades de seguridad de Windows Defender en [Crear e implementar una directiva de protección contra vulnerabilidades](https://docs.microsoft.com/mem/configmgr/protect/deploy-use/create-deploy-exploit-guard-policy).

Obtenga más información sobre los controles de seguridad en [Puntuación de seguridad mejorada (versión preliminar)](secure-score-security-controls.md).



### <a name="custom-policies-with-custom-metadata-are-now-generally-available"></a>Las directivas personalizadas con metadatos personalizados ya están disponibles con carácter general

Las directivas personalizadas ahora forman parte de la experiencia de recomendaciones de Security Center, la puntuación segura y el panel de normas de cumplimiento normativo. Esta característica ya está disponible con carácter general y permite ampliar la cobertura de evaluación de seguridad de la organización en Security Center. 

Cree una iniciativa personalizada en Azure Policy, agréguele directivas e incorpore a Azure Security Center y visualice como recomendaciones.

Ahora también hemos agregado la opción para editar los metadatos de recomendación personalizados. Las opciones de metadatos incluyen gravedad, pasos de corrección, información de amenazas y mucho más.  

Más información sobre [mejora de las recomendaciones personalizadas con información detallada](custom-security-policies.md#enhancing-your-custom-recommendations-with-detailed-information).



### <a name="crash-dump-analysis-capabilities-migrating-to-fileless-attack-detection"></a>Migración de funcionalidades de análisis de volcado de memoria a detección de ataques sin archivos 

Estamos integrando las capacidades de detección del análisis de volcado de memoria de Windows (CDA) en [detección de ataques sin archivos](https://docs.microsoft.com/azure/security-center/threat-protection#windows-fileless). El análisis de detección de ataques sin archivos no proporciona versiones mejoradas de las siguientes alertas de seguridad para máquinas Windows: Inyección de código detectada, se detectó un módulo de Windows enmascarado, shellcode detectado y segmento de código sospechoso.

Algunas de las ventajas de esta transición son:

- **Detección de malware proactiva y oportuna**: el enfoque de CDA implicaba esperar a que se produjera un bloqueo y, después, ejecutar el análisis para encontrar artefactos malintencionados. El uso de la detección de ataques sin archivos proporciona una identificación proactiva de las amenazas en memoria mientras se ejecutan. 

- **Alertas enriquecidas**: las alertas de seguridad de la detección de ataques sin archivos no están disponibles en CDA, como la información de las conexiones de red activas. 

- **Agregación de alertas**: cuando CDA detectó varios patrones de ataque en un solo volcado de memoria, desencadenó varias alertas de seguridad. Detección de ataques sin archivos combina todos los patrones de ataque identificados del mismo proceso en una única alerta, lo que elimina la necesidad de poner en correlación varias alertas.

- **Requisitos reducidos en el área de trabajo de Log Analytics**: los volcados que contienen datos potencialmente confidenciales ya no se cargarán en el área de trabajo de Log Analytics.



## <a name="april-2020"></a>Abril de 2020

Las actualizaciones de abril incluyen:
- [Los paquetes de cumplimiento dinámico ya están disponibles con carácter general](#dynamic-compliance-packages-are-now-generally-available)
- [Las recomendaciones de identidad ahora están incluidas en el nivel gratuito de Azure Security Center](#identity-recommendations-now-included-in-azure-security-center-free-tier)


### <a name="dynamic-compliance-packages-are-now-generally-available"></a>Los paquetes de cumplimiento dinámico ya están disponibles con carácter general

El panel de cumplimiento normativo de Azure Security Center ahora incluye **paquetes de cumplimiento dinámico** (ya disponible con carácter general) para realizar un seguimiento de los estándares de la industria y normativas adicionales.

Los paquetes de cumplimiento dinámico se pueden agregar a su suscripción o grupo de administración desde la página de la directiva de seguridad de Security Center. Cuando haya incorporado un estándar o una prueba comparativa, el estándar aparece en el panel de cumplimiento normativo con todos los datos de cumplimiento asociados asignados como evaluaciones. Podrá descargar un informe de resumen para cualquiera de los estándares que se hayan incorporado.

Ahora, puede agregar estándares como:

- **NIST SP 800-53 R4**
- **SWIFT CSP CSCF-v2020**
- **UK Official y UK NHS**
- **Canada Federal PBMM**
- **Azure CIS 1.1.0 (nuevo)** (que es una representación más completa de Azure CIS 1.1.0)

Además, recientemente hemos agregado el **Azure Security Benchmark**, las instrucciones específicas de Azure creadas por Microsoft para las prácticas recomendadas de seguridad y cumplimiento basadas en los marcos de cumplimiento comunes. Se admitirán estándares adicionales en el panel a medida que estén disponibles.  
 
Más información sobre [cómo personalizar el conjunto de estándares en el panel de cumplimiento normativo](update-regulatory-compliance-packages.md).


### <a name="identity-recommendations-now-included-in-azure-security-center-free-tier"></a>Recomendaciones de identidad ahora incluidas en el nivel gratuito de Azure Security Center

Las recomendaciones de seguridad para la identidad y el acceso en el nivel gratuito de Azure Security Center ya están disponibles con carácter general. Esto forma parte del esfuerzo de hacer que las características de la administración de posturas de seguridad en la nube (CSPM) sean gratuitas. Hasta ahora, estas recomendaciones solo estaban disponibles en el plan de tarifa estándar.

Los ejemplos de recomendaciones de identidad y acceso incluyen:

- "La autenticación multifactor debe estar habilitada en las cuentas con permisos de propietario en la suscripción".
- "Debe designar un máximo de tres propietarios para la suscripción".
- "Las cuentas en desuso deben quitarse de la suscripción".

Si tiene suscripciones en el plan de tarifa gratuito, su puntuación de seguridad se verá afectada por este cambio ya que nunca se evaluó su identidad y seguridad de acceso.

Más información sobre [recomendaciones de identidad y acceso](recommendations-reference.md#recs-identity).

Más información acerca de la [supervisión de la identidad y el acceso](security-center-identity-access.md).


## <a name="march-2020"></a>Marzo de 2020

Las actualizaciones de marzo incluyen:
- [La automatización del flujo de trabajo ya está disponible con carácter general](#workflow-automation-is-now-generally-available)
- [Integración de Azure Security Center con Windows Admin Center](#integration-of-azure-security-center-with-windows-admin-center)
- [Protección para Azure Kubernetes Service](#protection-for-azure-kubernetes-service)
- [Experiencia Just-in-Time mejorada](#improved-just-in-time-experience)
- [Dos recomendaciones de seguridad para las aplicaciones web en desuso](#two-security-recommendations-for-web-applications-deprecated)


### <a name="workflow-automation-is-now-generally-available"></a>La automatización del flujo de trabajo ya está disponible con carácter general

La característica de automatización del flujo de trabajo de Azure Security Center ahora está disponible con carácter general. Úsela para desencadenar automáticamente una instancia de Logic Apps sobre alertas y recomendaciones de seguridad. Además, los desencadenadores manuales están disponibles para las alertas y todas las recomendaciones que tienen la opción corrección rápida disponible.

Cada programa de seguridad incluye varios flujos de trabajo para la respuesta a incidentes. Estos procesos pueden incluir notificaciones para las partes interesadas competentes, iniciar un proceso de administración de cambios y aplicar pasos de corrección específicos. Los expertos en seguridad recomiendan automatizar tantos pasos de esos procedimientos como sea posible. La automatización reduce la sobrecarga y puede mejorar la seguridad asegurándose de que los pasos del proceso se realizan de forma rápida, coherente y según sus requisitos predefinidos.

Para más información acerca de las capacidades de Security Center automática y manual para ejecutar los flujos de trabajo, consulte [ automatización de flujos de trabajo](workflow-automation.md).

Más información acerca de la [creación de Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview).


### <a name="integration-of-azure-security-center-with-windows-admin-center"></a>Integración de Azure Security Center con Windows Admin Center

Ahora es posible migrar los servidores de Windows locales desde Windows Admin Center directamente al Azure Security Center. Security Center, a continuación, se convierte en el único panel para ver la información de seguridad de todos los recursos de Windows Admin Center, incluidos los servidores locales, las máquinas virtuales y las cargas de trabajo PaaS adicionales.

Después de mover un servidor de Windows Admin Center a Azure Security Center, podrá realizar lo siguiente:

- Ver alertas y recomendaciones de seguridad en la extensión de Security Center de Windows Admin Center.
- Ver la postura de seguridad y recuperar información detallada adicional de los servidores administrados de Windows Admin Center en Security Center, mediante Azure Portal (o a través de una API).

Más información sobre [cómo integrar Azure Security Center con Windows Admin Center](windows-admin-center-integration.md).


### <a name="protection-for-azure-kubernetes-service"></a>Protección para Azure Kubernetes Service

Azure Security Center está expandiendo sus características de seguridad de contenedor para proteger Azure Kubernetes Service (AKS).

La popular plataforma de código abierto Kubernetes se ha adoptado tan ampliamente que ahora es un estándar del sector para la orquestación de contenedores. A pesar de esta implementación generalizada, todavía hay una falta de comprensión sobre cómo proteger un entorno de Kubernetes. Defender las superficies de ataque de una aplicación en contenedores requiere conocimientos para asegurarse de que la infraestructura esté configurada de forma segura y se supervise constantemente para detectar posibles amenazas.

La defensa de Security Center incluye:

- **Detección y visibilidad**: detección continua de instancias de AKS administradas dentro de las suscripciones registradas de Security Center.
- **Recomendaciones de seguridad**: recomendaciones útiles para ayudarle a cumplir los procedimientos recomendados de seguridad para AKS. Estas recomendaciones se incluyen en su puntuación segura para asegurarse de que se ven como parte de la postura de seguridad de su organización. Un ejemplo de una recomendación relacionada con AKS que podría ver es que "se debe usar el control de acceso basado en roles para restringir el acceso a un clúster de servicio de Kubernetes".
- **Protección contra amenazas**: gracias al análisis continuo de la implementación de AKS, Security Center le alerta de las amenazas y actividades malintencionadas que se detectan en el host y el nivel de clúster de AKS.

Más información acerca de [integración de Azure Kubernetes Services con Security Center](azure-kubernetes-service-integration.md).

Más información sobre [las características de seguridad de los contenedores en Security Center](container-security.md).


### <a name="improved-just-in-time-experience"></a>Experiencia Just-in-Time mejorada

Las características, la operación y la interfaz de usuario de las herramientas Just-in-Time de Azure Security Center que protegen los puertos de administración se han mejorado de la manera siguiente: 

- **Campo de justificación**: al solicitar acceso a una máquina virtual (VM) a través de la página Just-in-Time del Azure Portal, hay un nuevo campo opcional disponible para especificar una justificación para la solicitud. Se puede realizar un seguimiento de la información especificada en este campo en el registro de actividad. 
- **Limpieza automática de reglas Just-In-Time (JIT) redundantes**: siempre que se actualiza una directiva JIT, se ejecuta automáticamente una herramienta de limpieza para comprobar la validez de todo el conjunto de reglas. La herramienta busca discrepancias entre las reglas de la directiva y las reglas del grupo de seguridad de red. Si la herramienta de limpieza encuentra una discrepancia, determina la causa y, cuando es seguro hacerlo, quita las reglas integradas que ya no son necesarias. El limpiador no elimina nunca las reglas que ha creado. 

Más información acerca de [la característica de acceso JIT](security-center-just-in-time.md).


### <a name="two-security-recommendations-for-web-applications-deprecated"></a>Dos recomendaciones de seguridad para las aplicaciones web en desuso

Dos recomendaciones de seguridad relacionadas con las aplicaciones web están en desuso: 

- Se deben proteger las reglas de las aplicaciones web en los grupos de seguridad de red de IaaS.
    (Directiva relacionada: Se deben proteger las reglas de NSG para las aplicaciones web en IaaS).

- El acceso a App Services debe estar restringido.
    (Directiva relacionada: El acceso a App Services debe estar restringido [versión preliminar])

Estas recomendaciones ya no aparecerán en la lista de Security Center de recomendaciones. Las directivas relacionadas ya no se incluirán en la iniciativa denominada "valor predeterminado de Security Center".

Más información sobre las [recomendaciones de seguridad](recommendations-reference.md).

