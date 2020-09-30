---
title: Vídeo de aplicación SQL de SaaS
description: En este artículo se indexan diversos puntos en nuestro vídeo de 81 minutos sobre el diseño de aplicaciones de inquilinos de base de datos de SaaS, de la conferencia de Ignite celebrada el 11 de octubre de 2017. Puede avanzar directamente a la parte que le interese. Se describen al menos 3 patrones. Se describen las características de Azure que simplifican el desarrollo y la administración.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.custom: sqldbrb=1
ms.date: 12/18/2018
ms.topic: conceptual
ms.openlocfilehash: 8cceeef7e41e8fc5c48b4e67cdb1adf3aa4f02a1
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89483871"
---
# <a name="video-indexed-and-annotated-for-multi-tenant-saas-app-using-azure-sql-database"></a>Vídeo indexado y anotado para la aplicación SaaS multiinquilinos mediante Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Este artículo es un índice anotado en las ubicaciones de tiempo de un vídeo de 81 minutos sobre modelos o patrones de inquilinos de SaaS. Este artículo le permite avanzar o retroceder en el vídeo hasta la parte que le interese. El vídeo explica las opciones de diseño principales de una aplicación de base de datos multiinquilino en Azure SQL Database. El vídeo incluye demostraciones, tutoriales de código de administración y, a veces, una cantidad mayor de detalles fundamentados en la experiencia que los que pueden encontrarse en nuestra documentación escrita.

El vídeo amplifica la información de nuestra documentación escrita, que se encuentra en: 
- *Conceptual:* [Patrones de inquilinato de base de datos SaaS multiinquilino][saas-concept-design-patterns-563e]
- *Tutoriales:* [Aplicación SaaS Wingtip Tickets][saas-how-welcome-wingtip-app-679t]

En el vídeo y los artículos se describen las diferentes fases de creación de una aplicación multiinquilino en Azure SQL Database en la nube. Las características especiales de Azure SQL Database facilitan el desarrollo y la implementación de aplicaciones multiinquilino que son fáciles de administrar y de rendimiento confiable.

Actualizamos habitualmente nuestra documentación escrita. El vídeo no se modifica ni actualiza, por lo que es posible que al final su contenido se quede obsoleto.



## <a name="sequence-of-38-time-indexed-screenshots"></a>Secuencia de 38 capturas de pantallas indexadas por tiempo

En esta sección se indexa la ubicación temporal de las 38 situaciones en todo el vídeo de 81 minutos. Cada índice de tiempo se anotará con una captura de pantalla del vídeo y, a veces, con información adicional.

Cada índice de tiempo se presenta en el formato de *h:mm:ss*. Por ejemplo, la segunda ubicación temporal indizada, con la etiqueta **Objetivos de la sesión**, comienza en la ubicación temporal aproximada de **0:03:11**.


### <a name="compact-links-to-video-indexed-time-locations"></a>Vínculos compactos a las ubicaciones de tiempo indexadas por vídeo

Los títulos siguientes son vínculos a sus correspondientes secciones anotadas más adelante en este artículo:

- [1. **(Inicio)** Diapositiva de bienvenida, 0:00:03](#anchor-image-wtip-min00001)
- [2. Objetivos de la sesión, 0:03:11](#anchor-image-wtip-min00311)
- [3. Agenda, 0:04:17](#anchor-image-wtip-min00417)
- [4. Aplicación web multiinquilino, 0:05:05](#anchor-image-wtip-min00505)
- [5. Formulario web de aplicaciones en acción, 0:05:55](#anchor-image-wtip-min00555)
- [6. Costo por inquilino (escala, aislamiento, recuperación), 0:09:31](#anchor-image-wtip-min00931)
- [7. Modelos de base de datos multiinquilino: ventajas y desventajas, 0:11:59](#anchor-image-wtip-min01159)
- [8. Modelo híbrido que combina las ventajas de MT/ST, 0:13:01](#anchor-image-wtip-min01301)
- [9. Inquilino único o multiinquilino: ventajas y desventajas, 0:16:44](#anchor-image-wtip-min01644)
- [10. Los grupos son rentables para cargas de trabajo impredecibles, 0:19:36](#anchor-image-wtip-min01936)
- [11. Demostración de la base de datos por inquilino e híbridos ST/MT, 0:20:08](#anchor-image-wtip-min02008)
- [12. Formulario de aplicación en directo que muestra Dojo, 0:20:29](#anchor-image-wtip-min02029)
- [13. MYOB y no un DBA a la vista, 0:28:54](#anchor-image-wtip-min02854)
- [14. Ejemplo de uso de grupo elástico MYOB, 0:29:40](#anchor-image-wtip-min02940)
- [15. Aprendizaje de MYOB y otros ISV, 0:31:36](#anchor-image-wtip-min03136)
- [16. Creación de patrones en el escenario de E2E SaaS, 0:43:15](#anchor-image-wtip-min04315)
- [17. Aplicación de SaaS híbrida canónica multiinquilino, 0:33:47](#anchor-image-wtip-min04733)
- [18. Aplicación de ejemplo de SaaS de Wingtip, 0:48:10](#anchor-image-wtip-min04810)
- [19. Escenarios y patrones explorados en los tutoriales, 0:49:10](#anchor-image-wtip-min04910)
- [20. Demostración de tutoriales y del repositorio de GitHub, 0:50:18](#anchor-image-wtip-min05018)
- [21. Repositorio de Github Microsoft/WingtipSaaS, 0:50:38](#anchor-image-wtip-min05038)
- [22. Exploración de los patrones, 0:56:20](#anchor-image-wtip-min05620)
- [23. Aprovisionamiento de los inquilinos e incorporación, 0:57:44](#anchor-image-wtip-min05744)
- [24. Aprovisionamiento de los inquilinos y conexión de la aplicación, 0:58:58](#anchor-image-wtip-min05858)
- [25. Demostración de scripts de administración que aprovisionan un único inquilino, 0:59:43](#anchor-image-wtip-min05943)
- [26. PowerShell para aprovisionar y catalogar, 1:00:02](#anchor-image-wtip-min10002)
- [27. T-SQL SELECT * FROM TenantsExtended, 1:03:30](#anchor-image-wtip-min10330)
- [28. Administración de cargas de trabajo de inquilino imprevisibles, 1:04:36](#anchor-image-wtip-min10436)
- [29. Supervisión de grupo elástico, 1:06:39](#anchor-image-wtip-min10639)
- [30. Generación de carga y supervisión de rendimiento, 1:09:42](#anchor-image-wtip-min10942)
- [31. Administración de esquemas a escala, 1:10:33](#anchor-image-wtip-min11033)
- [32. Consulta distribuida entre bases de datos de inquilino, 1:12:21](#anchor-image-wtip-min11221)
- [33. Demostración de generación de vales, 1:12:32](#anchor-image-wtip-min11232)
- [34. Análisis ad hoc de SSMS, 1:12:46](#anchor-image-wtip-min11246)
- [35. Extracción de datos del inquilino en Azure Synapse Analytics, 1:16:32](#anchor-image-wtip-min11632)
- [36. Gráfico de distribución de ventas diaria, 1:16:48](#anchor-image-wtip-min11648)
- [37. Resumen y llamada a la acción, 1:19:52](#anchor-image-wtip-min11952)
- [38. Recursos para obtener más información, 1:20:42](#anchor-image-wtip-min12042)


&nbsp;

### <a name="annotated-index-time-locations-in-the-video"></a>Ubicaciones temporales de índice anotadas en el vídeo

Al hacer clic en cualquier imagen de captura de pantalla se le dirigirá a la ubicación temporal exacta en el vídeo.


&nbsp; <a name="anchor-image-wtip-min00001"></a>
#### <a name="1-start-welcome-slide-00001"></a>1. *(Inicio)* Diapositiva de bienvenida, 0:00:01

*Aprendizaje de MYOB: Diseño de modelos para aplicaciones de SaaS en Azure SQL Database - BRK3120*

[![Diapositiva de bienvenida][image-wtip-min00003-brk3120-whole-welcome]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1)

- Título: Aprendizaje de MYOB: Diseño de modelos para aplicaciones de SaaS en Azure SQL Database
- Bill.Gibson@microsoft.com
- Jefe de programas principal, Microsoft Azure SQL Database
- Sesión de Microsoft Ignite BRK3120, Orlando, Florida (Estados Unidos), 11 de octubre de 2017


&nbsp; <a name="anchor-image-wtip-min00311"></a>
#### <a name="2-session-objectives-00153"></a>2. Objetivos de la sesión, 0:01:53
[![Objetivos de la sesión][image-wtip-min00311-session]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=113)

- Modelos alternativos para las aplicaciones multiinquilino, con las ventajas y desventajas.
- Modelos de SaaS para reducir los costos de desarrollo, administración y recursos.
- Una aplicación de ejemplo y scripts.
- Las características de PaaS y los patrones de SaaS hacen que SQL Database sea una plataforma de datos altamente escalable y rentable para SaaS multiinquilino.


&nbsp; <a name="anchor-image-wtip-min00417"></a>
#### <a name="3-agenda-00409"></a>3. Agenda, 0:04:09
[![Agenda][image-wtip-min00417-agenda]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=249)


&nbsp; <a name="anchor-image-wtip-min00505"></a>
#### <a name="4-multi-tenant-web-app-00500"></a>4. Aplicación web multiinquilino, 0:05:00
[![Aplicación SaaS de Wingtip: Aplicación web multiinquilino][image-wtip-min00505-web-app]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=300)


&nbsp; <a name="anchor-image-wtip-min00555"></a>
#### <a name="5-app-web-form-in-action-00539"></a>5. Formulario web de aplicaciones en acción, 0:05:39
[![Formulario web de aplicaciones en acción][image-wtip-min00555-app-web-form]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=339)


&nbsp; <a name="anchor-image-wtip-min00931"></a>
#### <a name="6-per-tenant-cost-scale-isolation-recovery-00658"></a>6. Costo por inquilino (escala, aislamiento, recuperación), 0:06:58
[![Costo por inquilino (escala, aislamiento, recuperación)][image-wtip-min00931-per-tenant-cost]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=418)


&nbsp; <a name="anchor-image-wtip-min01159"></a>
#### <a name="7-database-models-for-multi-tenant-pros-and-cons-00952"></a>7. Modelos de base de datos multiinquilino: ventajas y desventajas, 0:09:52
[![Modelos de base de datos multiinquilino: ventajas y desventajas][image-wtip-min01159-db-models-pros-cons]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=592)


&nbsp; <a name="anchor-image-wtip-min01301"></a>
#### <a name="8-hybrid-model-blends-benefits-of-mtst-01229"></a>8. Modelo híbrido que combina las ventajas de MT/ST, 0:12:29
[![Modelo híbrido que combina las ventajas de MT/ST][image-wtip-min01301-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=749)


&nbsp; <a name="anchor-image-wtip-min01644"></a>
#### <a name="9-single-tenant-vs-multi-tenant-pros-and-cons-01311"></a>9. Inquilino único o multiinquilino: ventajas y desventajas, 0:13:11
[![Inquilino único o multiinquilino: ventajas y desventajas][image-wtip-min01644-st-vs-mt]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=791)


&nbsp; <a name="anchor-image-wtip-min01936"></a>
#### <a name="10-pools-are-cost-effective-for-unpredictable-workloads-01749"></a>10. Los grupos son rentables para cargas de trabajo impredecibles, 0:17:49
[![Los grupos son rentables para cargas de trabajo impredecibles][image-wtip-min01936-pools-cost]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1069)


&nbsp; <a name="anchor-image-wtip-min02008"></a>
#### <a name="11-demo-of-database-per-tenant-and-hybrid-stmt-01959"></a>11. Demostración de la base de datos por inquilino e híbridos ST/MT, 0:19:59
[![Demostración de la base de datos por inquilino e híbridos ST/MT][image-wtip-min02008-demo-st-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1199)


&nbsp; <a name="anchor-image-wtip-min02029"></a>
#### <a name="12-live-app-form-showing-dojo-02010"></a>12. Formulario de aplicación en directo que muestra Dojo, 0:20:10
[![Formulario de aplicación en directo que muestra Dojo][image-wtip-min02029-live-app-form-dojo]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1210)

&nbsp; <a name="anchor-image-wtip-min02854"></a>
#### <a name="13-myob-and-not-a-dba-in-sight-02506"></a>13. MYOB y no un DBA a la vista, 0:25:06
[![MYOB y no un DBA a la vista][image-wtip-min02854-myob-no-dba]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1506)


&nbsp; <a name="anchor-image-wtip-min02940"></a>
#### <a name="14-myob-elastic-pool-usage-example-02930"></a>14. Ejemplo de uso de grupo elástico MYOB, 0:29:30
[![Ejemplo de uso de grupo elástico MYOB, 0:29:40][image-wtip-min02940-myob-elastic]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1770)


&nbsp; <a name="anchor-image-wtip-min03136"></a>
#### <a name="15-learning-from-myob-and-other-isvs-03125"></a>15. Aprendizaje de MYOB y otros ISV, 0:31:25
[![Aprendizaje de MYOB y otros ISV, 0:31:36][image-wtip-min03136-learning-isvs]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1885)


&nbsp; <a name="anchor-image-wtip-min04315"></a>
#### <a name="16-patterns-compose-into-e2e-saas-scenario-03142"></a>16. Creación de patrones en el escenario de E2E SaaS, 0:31:42
[![Creación de patrones en el escenario de E2E SaaS][image-wtip-min04315-patterns-compose]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1902)


&nbsp; <a name="anchor-image-wtip-min04733"></a>
#### <a name="17-canonical-hybrid-multi-tenant-saas-app-04604"></a>17. Aplicación de SaaS híbrida canónica multiinquilino, 0:04:46
[![Aplicación de SaaS híbrida canónica multiinquilino][image-wtip-min04733-canonical-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2764)


&nbsp; <a name="anchor-image-wtip-min04810"></a>
#### <a name="18-wingtip-saas-sample-app-04801"></a>18. Aplicación de ejemplo de SaaS de Wingtip, 0:48:01
[![Aplicación de ejemplo de SaaS de Wingtip, 0:48:10][image-wtip-min04810-wingtip-saas-app]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2881)


&nbsp; <a name="anchor-image-wtip-min04910"></a>
#### <a name="19-scenarios-and-patterns-explored-in-the-tutorials-04900"></a>19. Escenarios y patrones explorados en los tutoriales, 0:49:00
[![Escenarios y patrones explorados en los tutoriales][image-wtip-min04910-scenarios-tutorials]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2940)


&nbsp; <a name="anchor-image-wtip-min05018"></a>
#### <a name="20-demo-of-tutorials-and-github-repository-05012"></a>20. Demostración de tutoriales y del repositorio de GitHub, 0:50:12
[![Demostración de tutoriales y del repositorio de GitHub][image-wtip-min05018-demo-tutorials-github]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3012)


&nbsp; <a name="anchor-image-wtip-min05038"></a>
#### <a name="21-github-repo-microsoftwingtipsaas-05032"></a>21. Repositorio de GitHub Microsoft/WingtipSaaS, 0:50:32
[![Repositorio de GitHub Microsoft/WingtipSaaS][image-wtip-min05038-github-wingtipsaas]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3032)


&nbsp; <a name="anchor-image-wtip-min05620"></a>
#### <a name="22-exploring-the-patterns-05615"></a>22. Exploración de los patrones, 0:56:15
[![Exploración de los patrones][image-wtip-min05620-exploring-patterns]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3375)


&nbsp; <a name="anchor-image-wtip-min05744"></a>
#### <a name="23-provisioning-tenants-and-onboarding-05619"></a>23. Aprovisionamiento de los inquilinos e incorporación, 0:56:19
[![Aprovisionamiento de los inquilinos e incorporación][image-wtip-min05744-provisioning-tenants-onboarding-1]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3379)


&nbsp; <a name="anchor-image-wtip-min05858"></a>
#### <a name="24-provisioning-tenants-and-application-connection-05752"></a>24. Aprovisionamiento de los inquilinos y conexión de la aplicación, 0:57:52
[![Aprovisionamiento de los inquilinos y conexión de la aplicación][image-wtip-min05858-provisioning-tenants-app-connection-2]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3472)


&nbsp; <a name="anchor-image-wtip-min05943"></a>
#### <a name="25-demo-of-management-scripts-provisioning-a-single-tenant-05936"></a>25. Demostración de scripts de administración que aprovisionan un único inquilino, 0:59:36
[![Demostración de scripts de administración que aprovisionan un único inquilino][image-wtip-min05943-demo-management-scripts-st]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3576)


&nbsp; <a name="anchor-image-wtip-min10002"></a>
#### <a name="26-powershell-to-provision-and-catalog-05956"></a>26. PowerShell para aprovisionar y catalogar, 0:59:56
[![PowerShell para aprovisionar y catalogar][image-wtip-min10002-powershell-provision-catalog]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3596)


&nbsp; <a name="anchor-image-wtip-min10330"></a>
#### <a name="27-t-sql-select--from-tenantsextended-10325"></a>27. T-SQL SELECT * FROM TenantsExtended, 1:03:25
[![T-SQL SELECT * FROM TenantsExtended][image-wtip-min10330-sql-select-tenantsextended]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3805)


&nbsp; <a name="anchor-image-wtip-min10436"></a>
#### <a name="28-managing-unpredictable-tenant-workloads-10334"></a>28. Administración de cargas de trabajo de inquilino imprevisibles, 1:03:34
[![Administración de cargas de trabajo de inquilino imprevisibles, 1:04:36][image-wtip-min10436-managing-unpredictable-workloads]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3814)


&nbsp; <a name="anchor-image-wtip-min10639"></a>
#### <a name="29-elastic-pool-monitoring-10632"></a>29. Supervisión de grupo elástico, 1:06:32
[![Supervisión de grupo elástico][image-wtip-min10639-elastic-pool-monitoring]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3992)


&nbsp; <a name="anchor-image-wtip-min10942"></a>
#### <a name="30-load-generation-and-performance-monitoring-10937"></a>30. Generación de carga y supervisión de rendimiento, 1:09:37
[![Generación de carga y supervisión de rendimiento][image-wtip-min10942-load-generation]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4117)


&nbsp; <a name="anchor-image-wtip-min11033"></a>
#### <a name="31-schema-management-at-scale-10940"></a>31. Administración de esquemas a escala, 1:09:40
[![Administración de esquemas a escala][image-wtip-min11033-schema-management-scale]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=34120)


&nbsp; <a name="anchor-image-wtip-min11221"></a>
#### <a name="32-distributed-query-across-tenant-databases-11118"></a>32. Consulta distribuida entre bases de datos de inquilino, 1:11:18
[![Consulta distribuida entre bases de datos de inquilino][image-wtip-min11221-distributed-query]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4278)


&nbsp; <a name="anchor-image-wtip-min11232"></a>
#### <a name="33-demo-of-ticket-generation-11228"></a>33. Demostración de generación de vales, 1:12:28
[![Demostración de generación de vales, 1:12:32][image-wtip-min11232-demo-ticket-generation]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4348)


&nbsp; <a name="anchor-image-wtip-min11246"></a>
#### <a name="34-ssms-adhoc-analytics-11235"></a>34. Análisis ad hoc de SSMS, 1:12:35
[![Análisis ad hoc de SSMS][image-wtip-min11246-ssms-adhoc-analytics]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4355)


&nbsp; <a name="anchor-image-wtip-min11632"></a>
#### <a name="35-extract-tenant-data-into-azure-synapse-analytics-11546"></a>35. Extracción de datos del inquilino en Azure Synapse Analytics, 1:15:46
[![Extracción de datos del inquilino en Azure Synapse Analytics][image-wtip-min11632-extract-tenant-data-sql-dw]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4546)


&nbsp; <a name="anchor-image-wtip-min11648"></a>
#### <a name="36-graph-of-daily-sale-distribution-11638"></a>36. Gráfico de distribución de ventas diaria, 1:16:38
[![Gráfico de distribución de ventas diaria, 1:16:48][image-wtip-min11648-graph-daily-sale-distribution]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4598)


&nbsp; <a name="anchor-image-wtip-min11952"></a>
#### <a name="37-wrap-up-and-call-to-action-11743"></a>37. Resumen y llamada a la acción, 1:17:43
[![Resumen y llamada a la acción][image-wtip-min11952-wrap-up-call-action]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4663)


&nbsp; <a name="anchor-image-wtip-min12042"></a>
#### <a name="38-resources-for-more-information-12035"></a>38. Recursos para obtener más información, 1:20:35
[![Recursos para obtener más información][image-wtip-min12042-resources-more-info]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4835)

- [Publicación de blog, 22 de mayo de 2017][resource-blog-saas-patterns-app-dev-sql-db-768h]

- *Conceptual:* [Patrones de inquilinato de base de datos SaaS multiinquilino][saas-concept-design-patterns-563e]

- *Tutoriales:* [Aplicación SaaS Wingtip Tickets][saas-how-welcome-wingtip-app-679t]

- Repositorios de GitHub para tipos de aplicación de inquilinos SaaS de Wingtip Tickets:
    - [Repositorio de GitHub para el modelo de aplicación independiente][github-wingtip-standaloneapp].
    - [Repositorio de GitHub para el modelo de base de datos por inquilino][github-wingtip-dbpertenant].
    - [Repositorio de GitHub para el modelo de base de datos multiinquilino][github-wingtip-multitenantdb].





## <a name="next-steps"></a>Pasos siguientes

- [Artículo del primer tutorial][saas-how-welcome-wingtip-app-679t]




<!-- Image link reference IDs. -->

[image-wtip-min00003-brk3120-whole-welcome]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00003-brk3120-welcome-myob-design-saas-app-sql-db.png "Diapositiva de bienvenida"

[image-wtip-min00311-session]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00311-session-objectives-takeaway.png "Objetivos de la sesión"

[image-wtip-min00417-agenda]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00417-agenda-app-management-models-patterns.png "Agenda"

[image-wtip-min00505-web-app]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00505-wingtip-saas-app-mt-web.png "Aplicación SaaS de Wingtip: Aplicación web multiinquilino"

[image-wtip-min00555-app-web-form]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00555-app-form-contoso-concert-hall-night-opera.png "Formulario web de aplicaciones en acción"

[image-wtip-min00931-per-tenant-cost]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00931-saas-data-management-concerns.png "Costo por inquilino (escala, aislamiento, recuperación)"

[image-wtip-min01159-db-models-pros-cons]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01159-db-models-multi-tenant-saas-apps.png "Modelos de base de datos multiinquilino: ventajas y desventajas"

[image-wtip-min01301-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01301-hybrib-model-blends-benefits-mt-st.png "Modelo híbrido que combina las ventajas de MT/ST"

[image-wtip-min01644-st-vs-mt]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01644-st-mt-pros-cons.png "Inquilino único o multiinquilino: ventajas y desventajas"

[image-wtip-min01936-pools-cost]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01936-pools-cost-effective-unpredictable-workloads.png "Los grupos son rentables para cargas de trabajo impredecibles"

[image-wtip-min02008-demo-st-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02008-demo-st-hybrid.png "Demostración de la base de datos por inquilino e híbridos ST/MT"

[image-wtip-min02029-live-app-form-dojo]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02029-app-form-dogwwod-dojo.png "Formulario de aplicación en directo que muestra Dojo"

[image-wtip-min02854-myob-no-dba]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02854-myob-no-dba.png "MYOB y no un DBA a la vista"

[image-wtip-min02940-myob-elastic]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02940-myob-elastic-pool-usage-example.png "Ejemplo de uso de grupo elástico MYOB"

[image-wtip-min03136-learning-isvs]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min03136-myob-isv-saas-patterns-design-scale.png "Aprendizaje de MYOB y otros ISV"

[image-wtip-min04315-patterns-compose]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04315-patterns-compose-into-e2e-saas-scenario-st-mt.png "Creación de patrones en el escenario de E2E SaaS"

[image-wtip-min04733-canonical-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04733-canonical-hybrid-mt-saas-app.png "Aplicación de SaaS híbrida canónica multiinquilino"

[image-wtip-min04810-wingtip-saas-app]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04810-saas-sample-app-descr-of-modules-links.png "Aplicación de ejemplo de SaaS de Wingtip"

[image-wtip-min04910-scenarios-tutorials]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04910-scenarios-patterns-explored-tutorials.png "Escenarios y patrones explorados en los tutoriales"

[image-wtip-min05018-demo-tutorials-github]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05018-demo-saas-tutorials-github-repo.png "Demostración de tutoriales y del repositorio de GitHub"

[image-wtip-min05038-github-wingtipsaas]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05038-github-repo-wingtipsaas.png "Repositorio de GitHub Microsoft/WingtipSaaS"

[image-wtip-min05620-exploring-patterns]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05620-exploring-patterns-tutorials.png "Exploración de los patrones"

[image-wtip-min05744-provisioning-tenants-onboarding-1]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05744-provisioning-tenants-connecting-run-time-1.png "Aprovisionamiento de los inquilinos e incorporación"

[image-wtip-min05858-provisioning-tenants-app-connection-2]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05858-provisioning-tenants-connecting-run-time-2.png "Aprovisionamiento de los inquilinos y conexión de la aplicación"

[image-wtip-min05943-demo-management-scripts-st]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05943-demo-management-scripts-provisioning-st.png "Demostración de scripts de administración que aprovisionan un único inquilino"

[image-wtip-min10002-powershell-provision-catalog]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10002-powershell-code.png "PowerShell para aprovisionar y catalogar"

[image-wtip-min10330-sql-select-tenantsextended]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10330-ssms-tenantcatalog.png "T-SQL SELECT * FROM TenantsExtended"

[image-wtip-min10436-managing-unpredictable-workloads]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10436-managing-unpredictable-tenant-workloads.png "Administración de cargas de trabajo de inquilino imprevisibles"

[image-wtip-min10639-elastic-pool-monitoring]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10639-elastic-pool-monitoring.png "Supervisión de grupo elástico"

[image-wtip-min10942-load-generation]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10942-schema-management-scale.png "Generación de carga y supervisión de rendimiento"

[image-wtip-min11033-schema-management-scale]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11033-schema-manage-1000s-dbs-one.png "Administración de esquemas a escala"

[image-wtip-min11221-distributed-query]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11221-distributed-query-all-tenants-asif-single-db.png "Consulta distribuida entre bases de datos de inquilino"

[image-wtip-min11232-demo-ticket-generation]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11232-demo-ticket-generation-distributed-query.png "Demostración de generación de vales"

[image-wtip-min11246-ssms-adhoc-analytics]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11246-tsql-adhoc-analystics-db-elastic-query.png "Análisis ad hoc de SSMS"

[image-wtip-min11632-extract-tenant-data-sql-dw]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11632-extract-tenant-data-analytics-db-dw.png "Extracción de datos del inquilino en Azure Synapse Analytics"

[image-wtip-min11648-graph-daily-sale-distribution]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11648-graph-daily-sale-contoso-concert-hall.png "Gráfico de distribución de ventas diaria"

[image-wtip-min11952-wrap-up-call-action]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11952-wrap-call-action-saasfeedback.png "Resumen y llamada a la acción"

[image-wtip-min12042-resources-more-info]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min12042-resources-blog-github-tutorials-get-started.png "Recursos para obtener más información"




<!-- Article link reference IDs. -->

[saas-concept-design-patterns-563e]: saas-tenancy-app-design-patterns.md

[saas-how-welcome-wingtip-app-679t]: saas-tenancy-welcome-wingtip-tickets-app.md


[video-on-youtube-com-478y]: https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1

[video-on-channel9-479c]: https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK3120


[resource-blog-saas-patterns-app-dev-sql-db-768h]: https://azure.microsoft.com/blog/saas-patterns-accelerate-saas-application-development-on-sql-database/


[github-wingtip-standaloneapp]: https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp/

[github-wingtip-dbpertenant]: https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/

[github-wingtip-multitenantdb]: https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDB/

