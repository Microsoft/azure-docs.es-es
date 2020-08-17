---
title: Optimización del entorno de Active Directory con Azure Monitor | Microsoft Docs
description: Puede usar periódicamente la solución Active Directory Health Check para evaluar el riesgo y el estado de los entornos.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/10/2019
ms.openlocfilehash: 3c86b21c5e0533ab6a1a3c64dc601eb8bb573547
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318103"
---
# <a name="optimize-your-active-directory-environment-with-the-active-directory-health-check-solution-in-azure-monitor"></a>Optimización del entorno de Active Directory con la solución Active Directory Health Check en Azure Monitor

![Símbolo de AD Health Check](./media/ad-assessment/ad-assessment-symbol.png)

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

Puede usar periódicamente la solución Active Directory Health Check para evaluar el riesgo y el estado de los entornos del servidor. Este artículo facilita la instalación y el uso de la solución para que pueda tomar acciones correctivas en caso de posibles problemas.

Esta solución proporciona una lista priorizada de recomendaciones específicas para su infraestructura de servidor implementada. Las recomendaciones se clasifican en cuatro áreas de enfoque que ayudan a comprender rápidamente el riesgo y a tomar una acción correctiva.

Las recomendaciones se basan en los conocimientos y la experiencia adquiridos por los ingenieros de Microsoft producto de miles de visitas de clientes. En cada recomendación, se explica por qué podría ser importante para usted un problema y se proporcionan instrucciones sobre cómo implementar los cambios sugeridos.

Puede elegir las áreas de enfoque que sean más importantes para su organización y realizar un seguimiento del progreso hacia la consecución de un entorno libre de riesgos y en buen estado.

Después de agregar la solución y completar una comprobación, se muestra información resumida sobre las áreas de enfoque en el panel de **AD Health Check** para la infraestructura del entorno. Las secciones siguientes describen cómo usar la información que aparece en el panel de **AD Health Check**, donde puede ver y ejecutar las acciones recomendadas para su infraestructura de servicio de Active Directory.  

![imagen del icono de AD Health Check](./media/ad-assessment/ad-healthcheck-summary-tile.png)

![imagen del panel de AD Health Check](./media/ad-assessment/ad-healthcheck-dashboard-01.png)

## <a name="prerequisites"></a>Prerrequisitos

* La solución Active Directory Health Check requiere que esté instalada una versión compatible de .NET Framework 4.6.2 o superior en todos los equipos que tengan instalado el agente de Log Analytics para Windows (también conocido como Microsoft Monitoring Agent [MMA]).  System Center 2016 - Operations Manager, Operations Manager 2012 R2 y Azure Monitor usan el agente.
* La solución es compatible con controladores de dominio que ejecutan Windows Server 2008, y 2008 R2, Windows Server 2012 y 2012 R2, Windows Server 2016 y Windows Server 2019.
* Un área de trabajo de Log Analytics para agregar la solución Active Directory Health Check desde Azure Marketplace en Azure Portal. No se necesita ninguna configuración adicional.

  > [!NOTE]
  > Después de agregar la solución, el archivo AdvisorAssessment.exe se agrega a servidores con agentes. Los datos de configuración se leen y, luego, se envían a Azure Monitor en la nube para su procesamiento. Se aplica la lógica a los datos recibidos y el servicio de nube registra los datos.
  >
  >

Para llevar a cabo la comprobación de estado en los controladores de dominio que son miembros del dominio que se va a evaluar, cada controlador de dominio necesita un agente y conectividad a Azure Monitor mediante alguno de los siguientes métodos admitidos:

1. Instale el [agente de Log Analytics para Windows](../platform/agent-windows.md) si el controlador de dominio no está ya supervisado por System Center 2016 - Operations Manager u Operations Manager 2012 R2.
2. Si está supervisado por System Center 2016 - Operations Manager u Operations Manager 2012 R2, y el grupo de administración no está integrado con Azure Monitor, el controlador de dominio se puede hospedar en varios hosts junto con Azure Monitor para recopilar datos y enviarlos al servicio y mantener, no obstante, la supervisión de Operations Manager.  
3. En caso contrario, si el grupo de administración de Operations Manager está integrado con el servicio, tendrá que agregar los controladores de dominio para la recopilación de datos del servicio siguiendo los pasos descritos en [Adición de equipos administrados por agente](../platform/om-agents.md#connecting-operations-manager-to-azure-monitor) una vez que habilite la solución en el área de trabajo.  

El agente del controlador de dominio que envía informes a un grupo de administración de Operations Manager, recopila datos, los reenvía a su servidor de administración asignado y, a continuación, los envía directamente desde un servidor de administración a Azure Monitor.  Los datos no se escriben en las bases de datos de Operations Manager.  

## <a name="active-directory-health-check-data-collection-details"></a>Detalles de la recopilación de datos de Active Directory Health Check

Active Directory Health Check recopila datos de los siguientes orígenes mediante el agente habilitado:

- Registro
- LDAP
- .NET Framework
- Registro de eventos
- Interfaces ADSI
- Windows PowerShell
- Datos de archivo
- Instrumental de administración de Windows (WMI)
- API de la herramienta DCDIAG
- API del servicio de replicación de archivos (NTFRS)
- Código personalizado C#

Los datos se recopilan en el controlador de dominio y se reenvían a Azure Monitor cada siete días.  

## <a name="understanding-how-recommendations-are-prioritized"></a>Cómo se establecen prioridades entre las recomendaciones

A cada recomendación efectuada se le asigna un valor de ponderación que identifica su importancia relativa. Se muestran solo las diez recomendaciones más importantes.

### <a name="how-weights-are-calculated"></a>Cálculo de las ponderaciones

Las ponderaciones son valores agregados en función de tres factores principales:

* La *probabilidad* de que un asunto identificado cause problemas. Una probabilidad más alta equivale a una puntuación total mayor para la recomendación.
* El *impacto* del asunto en su organización en caso de que se produzca un problema. Un mayor impacto equivale a una puntuación total mayor para la recomendación.
* El *esfuerzo* necesario para implementar la recomendación. Un mayor esfuerzo equivale a una puntuación total menor para la recomendación.

La ponderación de cada recomendación se expresa como un porcentaje de la puntuación total disponible para cada área de enfoque. Por ejemplo, si una recomendación en el área de enfoque de seguridad y cumplimiento tiene una puntuación del 5 %, la implementación de esa recomendación aumenta la puntuación total de seguridad y cumplimiento en un 5 %.

### <a name="focus-areas"></a>Áreas de enfoque

**Seguridad y cumplimiento** : este apartado muestra recomendaciones en caso de posibles amenazas e infracciones de seguridad, directivas corporativas y requisitos de cumplimiento técnico, legal y reglamentario.

**Disponibilidad y continuidad empresarial** : este apartado muestra recomendaciones relacionadas con la disponibilidad de servicio, la resistencia de la infraestructura y la protección del negocio.

**Rendimiento y escalabilidad** : este apartado muestra recomendaciones que ayudarán a crecer a la infraestructura de TI de su organización y garantizarán que el entorno de TI cumple los requisitos de rendimiento vigentes y que, además, puede responder a las cambiantes necesidades de infraestructura.

**Actualización, migración e implementación** : este apartado muestra recomendaciones que le ayudarán a actualizar, migrar e implementar Active Directory en la infraestructura existente.

### <a name="should-you-aim-to-score-100-in-every-focus-area"></a>¿Debe tratar de conseguir una puntuación del 100 % en cada área de enfoque?

No necesariamente. Las recomendaciones se basan en los conocimientos y las experiencias adquiridos por los ingenieros de Microsoft producto de miles de visitas de clientes. Sin embargo, no hay dos infraestructuras de servidores que sean iguales, y es posible que determinadas recomendaciones puedan ser más o menos relevantes para usted. Por ejemplo, algunas recomendaciones de seguridad pueden ser menos pertinentes si las máquinas virtuales no están expuestas a Internet. Algunas recomendaciones de disponibilidad pueden ser menos relevantes para los servicios que proporcionan informes y recopilaciones de datos ad hoc de baja prioridad. Los problemas que son importantes para un negocio maduro pueden no serlo para otro que esté en sus inicios. Puede que desee identificar qué áreas de enfoque son prioritarias para usted y, posteriormente, observar cómo cambian las puntuaciones con el tiempo.

Cada recomendación incluye pautas que indican por qué es importante. Debe utilizar estas directrices para evaluar si es adecuado o no para usted implementar la recomendación, en función de la naturaleza de los servicios de TI y las necesidades empresariales de su organización.

## <a name="use-health-check-focus-area-recommendations"></a>Uso de las recomendaciones del área centradas en la comprobación de estado

Una vez instalada, puede ver el resumen de las recomendaciones mediante el icono Comprobación de estado en la página de soluciones de Azure Portal.

Consulte un resumen de las evaluaciones de cumplimiento para su infraestructura y, a continuación, profundice las recomendaciones.

### <a name="to-view-recommendations-for-a-focus-area-and-take-corrective-action"></a>Visualización de las recomendaciones para un área de enfoque y adopción de las medidas correctivas

[!INCLUDE [azure-monitor-solutions-overview-page](../../../includes/azure-monitor-solutions-overview-page.md)]

1. En la página de **información general**, haga clic en el icono de **Active Directory Health Check**.

2. En la página **Comprobación de estado**, revise la información de resumen de una de las hojas de las áreas de enfoque y, a continuación, haga clic en una de ellas para ver las recomendaciones para esa área de enfoque.

3. En cualquiera de las páginas de área de enfoque, puede ver las recomendaciones priorizadas que se han efectuado para su entorno. Haga clic en una recomendación en **Objetos afectados** para ver los detalles sobre por qué se realiza la recomendación.

    ![imagen de las recomendaciones de la comprobación de estado](./media/ad-assessment/ad-healthcheck-dashboard-02.png)

4. Puede tomar las medidas correctivas que se sugieren en **Acciones sugeridas**. Cuando se haya ocupado del asunto, las evaluaciones posteriores registran las acciones recomendadas que se han realizado y aumenta su puntuación de cumplimiento normativo. Los asuntos que se hayan corregido aparecerán en **Objetos superados**.

## <a name="ignore-recommendations"></a>Omisión de las recomendaciones

Si desea omitir ciertas recomendaciones, puede crear un archivo de texto que Azure Monitor usará para evitar que aparezcan las recomendaciones en los resultados de la evaluación.

### <a name="to-identify-recommendations-that-you-will-ignore"></a>Para identificar las recomendaciones que omitirá

[!INCLUDE [azure-monitor-log-queries](../../../includes/azure-monitor-log-queries.md)]

Use la siguiente consulta para mostrar las recomendaciones para los equipos que presentan errores en el entorno.

```
ADAssessmentRecommendation | where RecommendationResult == "Failed" | sort by Computer asc | project Computer, RecommendationId, Recommendation
```

Esta es una captura de pantalla que muestra la consulta de registro:

![recomendaciones con error](media/ad-assessment/ad-failed-recommendations.png)

Elija las recomendaciones que desea omitir. Usará los valores para RecommendationId en el procedimiento siguiente.

### <a name="to-create-and-use-an-ignorerecommendationstxt-text-file"></a>Para crear y usar un archivo de texto IgnoreRecommendations.txt

1. Cree un archivo llamado IgnoreRecommendations.txt.

2. Pegue o escriba cada RecommendationId para cada recomendación que desee que Azure Monitor omita en una línea independiente y, luego, guarde y cierre el archivo.

3. Coloque el archivo en la carpeta siguiente en cada equipo donde desea que Azure Monitor omita las recomendaciones.

   * En equipos con Microsoft Monitoring Agent (conectado directamente o a través de Operations Manager): *UnidadDelSistema*:\Archivos de programa\Microsoft Monitoring Agent\Agent
   * En el servidor de administración de Operations Manager 2012 R2: *UnidadDelSistema*:\Archivos de programa\Microsoft System Center 2012 R2\Operations Manager\Server
   * En el servidor de administración de Operations Manager 2016: *UnidadDelSistema*:\Archivos de programa\Microsoft System Center 2016\Operations Manager\Server

### <a name="to-verify-that-recommendations-are-ignored"></a>Para comprobar que se omiten las recomendaciones

Después de que se ejecute la siguiente comprobación de estado programada, de forma predeterminada cada siete días, las recomendaciones especificadas se marcan como *omitidas* y no aparecerán en el panel.

1. Puede usar las consultas de registros siguientes para enumerar todas las recomendaciones omitidas.

    ```
    ADAssessmentRecommendation | where RecommendationResult == "Ignored" | sort by Computer asc | project Computer, RecommendationId, Recommendation
    ```

2. Si posteriormente decide que desea ver las recomendaciones omitidas, quite todos los archivos IgnoreRecommendations.txt. También puede quitar RecommendationID de ellos.

## <a name="ad-health-check-solutions-faq"></a>Preguntas más frecuentes sobre las soluciones de AD Health Check

*¿Qué comprobaciones lleva a cabo la solución AD Assessment?*

* La consulta siguiente muestra una descripción de todas las comprobaciones que se llevan a cabo actualmente:

```Kusto
ADAssessmentRecommendation
| distinct RecommendationId, FocusArea, ActionArea, Recommendation, Description
| sort by FocusArea,ActionArea, Recommendation
```
Los resultados pueden exportarse a Excel para su revisión.

*¿Con qué frecuencia se ejecuta una comprobación de estado?*

* La comprobación se ejecuta cada siete días.

*¿Se puede configurar la frecuencia con la que se realiza la comprobación?*

* De momento, no.

*Si se detecta otro servidor después de haber agregado una solución de comprobación de estado, ¿se comprobará?*

* Sí, una vez que se detecte, se comprobará cada siete días a partir de entonces.

*Si se retira un servidor, ¿cuándo dejará de incluirse en la comprobación de estado?*

* Si un servidor no envía datos durante 3 semanas, se quitará.

*¿Cuál es el nombre del proceso que realiza la recopilación de datos?*

* AdvisorAssessment.exe

*¿Cuánto tiempo tarda en recopilar datos?*

* La recopilación de datos reales en el servidor tarda aproximadamente 1 hora. Puede demorar más en servidores que tengan una gran cantidad de servidores de Active Directory.

*¿Se puede configurar el momento en que se recopilan los datos?*

* De momento, no.

*¿Por qué se muestran solo las 10 recomendaciones principales?*

* En lugar de darle una lista exhaustiva y abrumadora de tareas, se recomienda centrarse primero en las recomendaciones prioritarias. Después de aplicarlas, se mostrarán más recomendaciones. Si prefiere ver una lista detallada, puede ver todas las recomendaciones mediante una consulta de registros.

*¿Se puede hacer caso omiso de una recomendación?*

* Sí, consulte la sección [Omisión de las recomendaciones](#ignore-recommendations) anterior.

## <a name="next-steps"></a>Pasos siguientes

Use [consultas de registros de Azure Monitor](../log-query/log-query-overview.md) para aprender a analizar recomendaciones y datos detallados de AD Health Check.

