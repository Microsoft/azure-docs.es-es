---
title: Optimización del entorno de SQL Server con Azure Monitor | Microsoft Docs
description: Con Azure Monitor, puede usar periódicamente la solución SQL Health Check para evaluar el riesgo y el estado de los entornos.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/05/2020
ms.openlocfilehash: 98caca31e172f54c3e37f33c5a463790d9d27032
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325991"
---
# <a name="optimize-your-sql-environment-with-the-sql-server-health-check-solution-in-azure-monitor"></a>Optimización del entorno de SQL con la solución SQL Server Health Check en Azure Monitor

![Símbolo de SQL Health Check](./media/sql-assessment/sql-assessment-symbol.png)

Puede usar periódicamente la solución SQL Health Check para evaluar el riesgo y el estado de los entornos de servidor. Este artículo le ayudará a instalar la solución para que puede tomar acciones correctivas para posibles problemas.

Esta solución proporciona una lista priorizada de recomendaciones específicas para su infraestructura de servidor implementada. Las recomendaciones se clasifican en seis áreas de enfoque que ayudan a comprender rápidamente el riesgo y a tomar la acción correctiva.

Las recomendaciones efectuadas se basan en los conocimientos y la experiencia adquiridos por los ingenieros de Microsoft fruto de miles de visitas de clientes. En cada recomendación, se explica por qué podría ser importante para usted un problema y se proporcionan instrucciones sobre cómo implementar los cambios sugeridos.

Puede elegir las áreas de enfoque que sean más importantes para su organización y realizar un seguimiento del progreso hacia la consecución de un entorno libre de riesgos y en buen estado.

Después de haber agregado la solución y completar una evaluación, se muestra información resumida sobre las áreas de enfoque en el panel de **SQL Health Check** para la infraestructura del entorno. Las secciones siguientes describen cómo usar la información que aparece en el panel de **SQL Health Check**, donde puede ver y ejecutar las acciones recomendadas para su infraestructura de SQL Server.

![imagen del icono de SQL Health Check](./media/sql-assessment/sql-healthcheck-summary-tile.png)

![imagen del panel de SQL Health Check](./media/sql-assessment/sql-healthcheck-dashboard-01.png)

## <a name="prerequisites"></a>Prerrequisitos

* La solución SQL Health Check requiere que esté instalada una versión compatible de .NET Framework 4.6.2 en todos los equipos que tengan Microsoft Monitoring Agent (MMA) instalado.  System Center 2016 - Operations Manager y Operations Manager 2012 R2, y Azure Monitor, usan el agente de MMA.  
* La solución admite las versiones de SQL Server 2012, 2014, 2016, 2017 y 2019.
* Un área de trabajo de Log Analytics para agregar la solución SQL Health Check desde Azure Marketplace en Azure Portal. Para instalar la solución, debe ser administrador o colaborador en la suscripción de Azure.

  > [!NOTE]
  > Después de agregar la solución, el archivo AdvisorAssessment.exe se agrega a servidores con agentes. Los datos de configuración se leen y, luego, se envían a Azure Monitor en la nube para su procesamiento. Se aplica la lógica a los datos recibidos y el servicio de nube registra los datos.
  >
  >

Para llevar a cabo la comprobación de estado en los servidores de SQL Server, necesitan un agente y conectividad a Azure Monitor mediante alguno de los siguientes métodos admitidos:

1. Instale [Microsoft Monitoring Agent (MMA)](../platform/agent-windows.md) si el servidor no está ya supervisado por System Center 2016 - Operations Manager u Operations Manager 2012 R2.
2. Si está supervisado por System Center 2016 - Operations Manager u Operations Manager 2012 R2, y el grupo de administración no está integrado con el servicio Azure Monitor, el servidor se puede hospedar en varios hosts junto con Log Analytics para recopilar datos y enviarlos al servicio y mantener, no obstante, la supervisión de Operations Manager.  
3. En caso contrario, si el grupo de administración de Operations Manager está integrado con el servicio, tendrá que agregar los controladores de dominio para la recopilación de datos del servicio siguiendo los pasos descritos en [Adición de equipos administrados por agente](../platform/om-agents.md#connecting-operations-manager-to-azure-monitor) una vez que habilite la solución en el área de trabajo.  

El agente de su instancia de SQL Server que envía informes a un grupo de administración de Operations Manager, recopila datos, los reenvía a su servidor de administración asignado y, a continuación, los envía directamente desde un servidor de administración a Azure Monitor.  Los datos no se escriben en las bases de datos de Operations Manager.  

Si Operations Manager supervisa SQL Server, deberá configurar una cuenta de ejecución de Operations Manager. Para más información, consulte [Cuentas de ejecución de Operations Manager para Azure Monitor](#operations-manager-run-as-accounts-for-log-analytics) más adelante.

## <a name="sql-health-check-data-collection-details"></a>Detalles de la recopilación de datos de SQL Health Check
SQL Health Check recopila datos de los siguientes orígenes mediante el agente habilitado:

* Instrumental de administración de Windows (WMI)
* Registro
* Contadores de rendimiento
* Resultados de la vista de administración dinámica de SQL Server

Los datos se recopilan en el servidor de SQL Server y se reenvían a Log Analytics cada siete días.

## <a name="operations-manager-run-as-accounts-for-log-analytics"></a>Cuentas de ejecución de Operations Manager para Log Analytics
Log Analytics usa el agente y el grupo de administración de Operations Manager para recopilar y enviar datos al servicio de Log Analytics. Log Analytics se basa en paquetes de administración de cargas de trabajo para proporcionar servicios de valor añadido. Cada carga de trabajo requiere privilegios específicos de la carga de trabajo para ejecutar paquetes de administración en un contexto de seguridad diferente, como una cuenta de usuario de dominio. Debe proporcionar información de credenciales mediante la configuración de una cuenta de ejecución de Operations Manager.

Utilice la siguiente información para establecer la cuenta de ejecución de Operations Manager para SQL Health Check.

### <a name="set-the-run-as-account-for-sql-health-check"></a>Establecimiento de la cuenta de ejecución para SQL Health Check
 Si ya está usando el paquete de administración de SQL Server, debe utilizar esa configuración de ejecución.

#### <a name="to-configure-the-sql-run-as-account-in-the-operations-console"></a>Para configurar la cuenta de ejecución de SQL en la consola de Operations, siga estos pasos:
> [!NOTE]
> De forma predeterminada los flujos de trabajo del módulo de administración se ejecutan en el contexto de seguridad de la cuenta del sistema Local. Si va a usar Microsoft Monitoring Agent conectado directamente al servicio en lugar de enviar los informes a un grupo de administración de Operations Manager, omita los pasos 1 a 5 que aparecen a continuación y ejecute el ejemplo de T-SQL o el de PowerShell especificando NT AUTHORITY\SYSTEM como nombre de usuario.
>
>

1. En Operations Manager, abra la Consola del operador y haga clic en **Administración**.
2. En **Configuración de ejecución**, haga clic en **Perfiles** y abra el **Perfil de identificación de SQL Assessment**.
3. En la página **Cuentas de ejecución**, haga clic en **Agregar**.
4. Seleccione una cuenta de ejecución de Windows que contenga las credenciales necesarias para SQL Server o haga clic en **Nuevo** para crear una.

   > [!NOTE]
   > El tipo de cuenta de ejecución debe ser Windows. La cuenta de ejecución también debe ser parte del grupo de administradores locales en todos los servidores de Windows que hospedan instancias de SQL Server.
   >
   >
5. Haga clic en **Save**(Guardar).
6. Modifique y luego ejecute el siguiente ejemplo de T-SQL en cada instancia de SQL Server para conceder los permisos mínimos que necesita la cuenta de ejecución para realizar la comprobación de estado. Sin embargo, este paso no es necesario si una cuenta de ejecución ya forma parte del rol de servidor sysadmin en las instancias de SQL Server.

```
    ---
    -- Replace <UserName> with the actual user name being used as Run As Account.
    USE master

    -- Create login for the user, comment this line if login is already created.
    CREATE LOGIN [<UserName>] FROM WINDOWS

    -- Grant permissions to user.
    GRANT VIEW SERVER STATE TO [<UserName>]
    GRANT VIEW ANY DEFINITION TO [<UserName>]
    GRANT VIEW ANY DATABASE TO [<UserName>]

    -- Add database user for all the databases on SQL Server Instance, this is required for connecting to individual databases.
    -- NOTE: This command must be run anytime new databases are added to SQL Server instances.
    EXEC sp_msforeachdb N'USE [?]; CREATE USER [<UserName>] FOR LOGIN [<UserName>];'

```

#### <a name="to-configure-the-sql-run-as-account-using-windows-powershell"></a>Para configurar la cuenta de ejecución de SQL mediante Windows PowerShell, siga estos pasos:
Abra una ventana de PowerShell y ejecute el siguiente script después de actualizarlo con su información:

```
    import-module OperationsManager
    New-SCOMManagementGroupConnection "<your management group name>"

    $profile = Get-SCOMRunAsProfile -DisplayName "SQL Assessment Run As Profile"
    $account = Get-SCOMrunAsAccount | Where-Object {$_.Name -eq "<your run as account name>"}
    Set-SCOMRunAsProfile -Action "Add" -Profile $Profile -Account $Account
```

## <a name="understanding-how-recommendations-are-prioritized"></a>Cómo se establecen prioridades entre las recomendaciones
A cada recomendación efectuada se le asigna un valor de ponderación que identifica su importancia relativa. Se muestran solo las diez recomendaciones más importantes.

### <a name="how-weights-are-calculated"></a>Cálculo de las ponderaciones
Las ponderaciones son valores agregados en función de tres factores principales:

* La *probabilidad* de que un asunto identificado pueda causar problemas. Una probabilidad más alta equivale a una puntuación total mayor para la recomendación.
* El *impacto* del asunto en su organización en caso de que se produzca un problema. Un mayor impacto equivale a una puntuación total mayor para la recomendación.
* El *esfuerzo* necesario para implementar la recomendación. Un mayor esfuerzo equivale a una puntuación total menor para la recomendación.

La ponderación de cada recomendación se expresa como un porcentaje de la puntuación total disponible para cada área de enfoque. Por ejemplo, si una recomendación en el área de enfoque de seguridad y cumplimiento tiene una puntuación del 5 %, la implementación de esa recomendación aumentará la puntuación total de seguridad y cumplimiento en un 5 %.

### <a name="focus-areas"></a>Áreas de enfoque
**Seguridad y cumplimiento** : este apartado muestra recomendaciones en caso de posibles amenazas e infracciones de seguridad, directivas corporativas y requisitos de cumplimiento técnico, legal y reglamentario.

**Disponibilidad y continuidad empresarial** : este apartado muestra recomendaciones relacionadas con la disponibilidad de servicio, la resistencia de la infraestructura y la protección del negocio.

**Rendimiento y escalabilidad** : este apartado muestra recomendaciones que ayudarán a crecer a la infraestructura de TI de su organización y garantizarán que el entorno de TI cumple los requisitos de rendimiento vigentes y que, además, puede responder a las cambiantes necesidades de infraestructura.

**Actualización, migración e implementación**. Esta área de enfoque muestra recomendaciones que lo ayudarán a actualizar, migrar e implementar SQL Server en la infraestructura existente.

**Operaciones y supervisión**. Esta área de enfoque muestra recomendaciones para ayudar a simplificar las operaciones de TI, implementar el mantenimiento preventivo y maximizar el rendimiento.

**Administración de cambios y configuración**. Esta área de enfoque muestra recomendaciones para ayudar a proteger las operaciones diarias, garantizar que los cambios no afectan de manera negativa a su infraestructura, establecer procedimientos de control de cambios y realizar un seguimiento y auditar las configuraciones del sistema.

### <a name="should-you-aim-to-score-100-in-every-focus-area"></a>¿Debe tratar de conseguir una puntuación del 100 % en cada área de enfoque?
No necesariamente. Las recomendaciones se basan en los conocimientos y las experiencias adquiridos por los ingenieros de Microsoft producto de miles de visitas de clientes. Sin embargo, no hay dos infraestructuras de servidores que sean iguales, y es posible que determinadas recomendaciones puedan ser más o menos relevantes para usted. Por ejemplo, algunas recomendaciones de seguridad pueden ser menos pertinentes si las máquinas virtuales no están expuestas a Internet. Algunas recomendaciones de disponibilidad pueden ser menos relevantes para los servicios que proporcionan informes y recopilaciones de datos ad hoc de baja prioridad. Los problemas que son importantes para un negocio maduro pueden no serlo para otro que esté en sus inicios. Puede que desee identificar qué áreas de enfoque son prioritarias para usted y, posteriormente, observar cómo cambian las puntuaciones con el tiempo.

Cada recomendación incluye pautas que indican por qué es importante. Debe utilizar estas directrices para evaluar si es adecuado o no para usted implementar la recomendación, en función de la naturaleza de los servicios de TI y las necesidades empresariales de su organización.

## <a name="use-health-check-focus-area-recommendations"></a>Uso de las recomendaciones del área centradas en la comprobación de estado
Antes de que pueda usar una solución de evaluación en Azure Monitor, debe tener instalada la solución.  Una vez instalada, puede ver el resumen de las recomendaciones mediante el icono de SQL Health Check en la página **Información general** de Azure Monitor en Azure Portal.

Consulte un resumen de las evaluaciones de cumplimiento para su infraestructura y, a continuación, profundice las recomendaciones.

### <a name="to-view-recommendations-for-a-focus-area-and-take-corrective-action"></a>Visualización de las recomendaciones para un área de enfoque y adopción de las medidas correctivas
1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En Azure Portal, haga clic en **Más servicios**, en la esquina inferior izquierda. En la lista de recursos, escriba **Monitor**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Monitor**.
3. En la sección **Conclusiones** del menú, seleccione **Más**.  
4. En la página **Información general**, haga clic en el icono de **SQL Health Check**.
5. En la página **Comprobación de estado**, revise la información de resumen de una de las hojas de las áreas de enfoque y, a continuación, haga clic en una de ellas para ver las recomendaciones para esa área de enfoque.
6. En cualquiera de las páginas de área de enfoque, puede ver las recomendaciones priorizadas que se han efectuado para su entorno. Haga clic en una recomendación en **Objetos afectados** para ver los detalles sobre por qué se realiza la recomendación.<br><br> ![imagen de las recomendaciones de SQL Health Check](./media/sql-assessment/sql-healthcheck-dashboard-02.png)<br>
7. Puede tomar las medidas correctivas que se sugieren en **Acciones sugeridas**. Cuando se haya ocupado del asunto, las evaluaciones posteriores registrarán las acciones recomendadas que se han realizado y aumentará su calificación de cumplimiento normativo. Los asuntos que se hayan corregido aparecerán en **Objetos superados**.

## <a name="ignore-recommendations"></a>Omisión de las recomendaciones
Si desea omitir ciertas recomendaciones, puede crear un archivo de texto que Azure Monitor usará para evitar que aparezcan las recomendaciones en los resultados de la evaluación.

### <a name="to-identify-recommendations-that-you-will-ignore"></a>Para identificar las recomendaciones que omitirá
1. En el menú de Azure Monitor, haga clic en **Registros**.
2. Use la siguiente consulta para mostrar las recomendaciones para los equipos que presentan errores en el entorno.

    ```
    SQLAssessmentRecommendation | where RecommendationResult == "Failed" | sort by Computer asc | project Computer, RecommendationId, Recommendation
    ```
    Esta es una captura de pantalla que muestra la consulta de registro:<br><br> ![recomendaciones con error](./media/sql-assessment/sql-assess-failed-recommendations.png)<br>

3. Elija las recomendaciones que desea omitir. Usará los valores para RecommendationId en el procedimiento siguiente.

### <a name="to-create-and-use-an-ignorerecommendationstxt-text-file"></a>Para crear y usar un archivo de texto IgnoreRecommendations.txt
1. Cree un archivo llamado IgnoreRecommendations.txt.
2. Pegue o escriba cada RecommendationId para cada recomendación que desee que Azure Monitor omita en una línea independiente y, luego, guarde y cierre el archivo.
3. Coloque el archivo en la carpeta siguiente en cada equipo donde desea que Azure Monitor omita las recomendaciones.
   * En equipos con Microsoft Monitoring Agent (conectado directamente o a través de Operations Manager): *UnidadDelSistema*:\Archivos de programa\Microsoft Monitoring Agent\Agent
   * En el servidor de administración de Operations Manager: *UnidadDelSistema*:\Archivos de programa\Microsoft System Center 2012 R2\Operations Manager\Server
   * En el servidor de administración de Operations Manager 2016: *UnidadDelSistema*:\Archivos de programa\Microsoft System Center 2016\Operations Manager\Server

### <a name="to-verify-that-recommendations-are-ignored"></a>Para comprobar que se omiten las recomendaciones
1. Después de que se ejecute la siguiente evaluación programada, de forma predeterminada cada 7 días, las recomendaciones especificadas se marcan como omitidas y no aparecerán en el panel de evaluación.
2. Puede usar las consultas de búsqueda de registros siguientes para enumerar todas las recomendaciones omitidas.

    ```
    SQLAssessmentRecommendation | where RecommendationResult == "Ignored" | sort by Computer asc | project Computer, RecommendationId, Recommendation
    ```
3. Si posteriormente decide que desea ver las recomendaciones omitidas, quite todos los archivos IgnoreRecommendations.txt. También puede quitar RecommendationID de ellos.

## <a name="sql-health-check-solution-faq"></a>Preguntas más frecuentes sobre la solución de SQL Health Check

*¿Qué comprobaciones lleva a cabo la solución SQL Assessment?*

* La consulta siguiente muestra una descripción de todas las comprobaciones que se llevan a cabo actualmente:

```Kusto
SQLAssessmentRecommendation
| distinct RecommendationId, FocusArea, ActionArea, Recommendation, Description
| sort by FocusArea,ActionArea, Recommendation
```
Los resultados pueden exportarse a Excel para su revisión.


*¿Con qué frecuencia se ejecuta una comprobación de estado?*

* La comprobación se ejecuta cada siete días.

*¿Se puede configurar la frecuencia con la que se realiza la comprobación?*

* De momento, no.

*Si se detecta otro servidor después de haber agregado la solución SQL Health Check, ¿se comprobará?*

* Sí, una vez que se detecte, se comprobará cada siete días a partir de entonces.

*Si se retira un servidor, ¿cuándo dejará de incluirse en la comprobación de estado?*

* Si un servidor no envía datos durante 3 semanas, se quitará.

*¿Cuál es el nombre del proceso que realiza la recopilación de datos?*

* AdvisorAssessment.exe

*¿Cuánto tiempo tarda en recopilar datos?*

* La recopilación de datos reales en el servidor tarda aproximadamente 1 hora. Puede tardar más en servidores que tengan un gran número de bases de datos o instancias de SQL.

*¿Qué tipo de datos se recopila?*

* Se recopilan los siguientes tipos de datos:
  * WMI
  * Registro
  * Contadores de rendimiento
  * Vistas de administración dinámica (DMV) de SQL

*¿Se puede configurar el momento en que se recopilan los datos?*

* De momento, no.

*¿Por qué es necesario configurar una cuenta "ejecutar como"?*

* Para SQL Server, se ejecuta un número reducido de consultas SQL. Para que se ejecuten, hay que utilizar una cuenta "ejecutar como" que disponga de los permisos VIEW SERVER STATE para SQL.  Además, para consultar WMI, se necesitan credenciales de administrador local.

*¿Por qué se muestran solo las 10 recomendaciones principales?*

* En lugar de darle una lista exhaustiva y abrumadora de tareas, se recomienda centrarse primero en las recomendaciones prioritarias. Después de aplicarlas, se mostrarán más recomendaciones. Si prefiere ver una lista detallada, puede ver todas las recomendaciones mediante la búsqueda de registros de Log Analytics.

*¿Se puede hacer caso omiso de una recomendación?*

* Sí, consulte la sección [Omisión de las recomendaciones](#ignore-recommendations) anterior.

## <a name="next-steps"></a>Pasos siguientes
* [Registre consultas](../log-query/log-query-overview.md) para aprender a analizar recomendaciones y datos de SQL Health Check.

