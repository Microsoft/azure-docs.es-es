---
title: Evaluación de System Center Operations Manager con Azure Monitor
description: Puede usar la solución System Center Operations Manager Health Check para evaluar el riesgo y el estado de los entornos periódicamente.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/25/2018
ms.openlocfilehash: 97d7d21374062462248e1b86f2bde2fef2d25331
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326059"
---
# <a name="optimize-your-environment-with-the-system-center-operations-manager-health-check-preview-solution"></a>Optimización del entorno con la solución System Center Operations Manager Health Check (versión preliminar)

![Símbolo de System Center Operations Manager Health Check](./media/scom-assessment/scom-assessment-symbol.png)

Puede usar la solución System Center Operations Manager Health Check para evaluar el riesgo y el estado del grupo de administración de System Center Operations Manager periódicamente. Este artículo le ayudará a instalar, configurar y usar la solución para que pueda tomar acciones correctivas en caso de posibles problemas.

Esta solución proporciona una lista priorizada de recomendaciones específicas para su infraestructura de servidor implementada. Las recomendaciones se clasifican en cuatro áreas de enfoque que ayudan a comprender rápidamente el riesgo y a tomar la acción correctiva.

Las recomendaciones efectuadas se basan en los conocimientos y la experiencia adquiridos por los ingenieros de Microsoft fruto de miles de visitas de clientes. En cada recomendación, se explica por qué podría ser importante para usted un problema y se proporcionan instrucciones sobre cómo implementar los cambios sugeridos.

Puede elegir las áreas de enfoque que sean más importantes para su organización y realizar un seguimiento del progreso hacia la consecución de un entorno libre de riesgos y en buen estado.

Después de agregar la solución y realizar una evaluación, se muestra información resumida sobre las áreas más importantes en el panel de **System Center Operations Manager Health Check** para la infraestructura. Las secciones siguientes describen cómo usar la información que aparece en el panel de **System Center Operations Manager Health Check**, donde puede ver y ejecutar las acciones recomendadas para su entorno de Operations Manager.

![icono de la solución System Center Operations Manager](./media/scom-assessment/log-analytics-scom-healthcheck-tile.png)

![panel de System Center Operations Manager Health Check](./media/scom-assessment/log-analytics-scom-healthcheck-dashboard-01.png)

## <a name="installing-and-configuring-the-solution"></a>Instalación y configuración de la solución

La solución funciona con Microsoft System Center 2012 Operations Manager Service Pack 1, Microsoft System Center 2012 R2 Operations Manager, Microsoft System Center 2016 Operations Manager, Microsoft System Center 2016 Operations Manager y Microsoft System Center Operations Manager 1807. Se debe instalar una versión compatible de .NET Framework 4.6.2 en cada servidor de administración.

Utilice la siguiente información para instalar y configurar la solución.

- Para poder usar una solución Health Check en Log Analytics, debe tenerla instalada. Instale la solución desde [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.SCOMAssessmentOMS?tab=Overview).

- Después de agregar la solución al área de trabajo, el icono de **System Center Operations Manager Health Check** en el panel muestra un mensaje que indica que es necesario realizar una configuración adicional. Haga clic en el icono y siga los pasos de configuración mencionados en la página.

  ![Icono del panel de System Center Operations Manager](./media/scom-assessment/scom-configrequired-tile.png)

> [!NOTE]
> La configuración de System Center Operations Manager se puede realizar a través del script siguiendo los pasos mencionados en la página de configuración de la solución en Log Analytics.

 Para configurar la evaluación a través de la consola del operador de Operations Manager, realice los pasos siguientes por orden:
1. [Establecimiento de la cuenta de ejecución de System Center Operations Manager Health Check](#operations-manager-run-as-accounts-for-log-analytics)  
2. Configuración de la regla de System Center Operations Manager Health Check

## <a name="system-center-operations-manager-health-check-data-collection-details"></a>Información detallada acerca de la recopilación de datos de System Center Operations Manager Health Check

La solución System Center Operations Manager Health Check recopila datos de los orígenes siguientes:

* Registro
* Instrumental de administración de Windows (WMI)
* Registro de eventos
* Datos de archivo
* Directamente desde Operations Manager con las consultas de PowerShell y SQL desde un servidor de administración que haya especificado.  

Los datos se recopilan en el servidor de administración y se reenvían a Log Analytics cada siete días.  

## <a name="operations-manager-run-as-accounts-for-log-analytics"></a>Cuentas de ejecución de Operations Manager para Log Analytics

Log Analytics se basa en paquetes de administración de cargas de trabajo para proporcionar servicios de valor añadido. Cada carga de trabajo requiere privilegios específicos de la carga de trabajo para ejecutar paquetes de administración en un contexto de seguridad diferente, como una cuenta de usuario de dominio. Configure una cuenta de ejecución de Operations Manager con credenciales con privilegios. Para obtener más información, consulte [Cómo crear una Cuenta de ejecución](/previous-versions/system-center/system-center-2012-R2/hh321655(v=sc.12)) en la documentación de Operations Manager.

Use la siguiente información para establecer la cuenta de ejecución de Operations Manager para System Center Operations Manager Health Check.

### <a name="set-the-run-as-account"></a>Establecimiento de la cuenta de ejecución

La cuenta de ejecución debe cumplir los siguientes requisitos antes de continuar:

* Una cuenta de usuario de dominio que sea miembro del grupo Administradores local en todos los servidores que admitan cualquier rol de Operations Manager: servidor de administración, SQL Server que hospeda la base de datos de ACS, el almacén de datos y el operativo, notificación, consola web y servidor de puerta de enlace.
* Rol de administrador de Operation Manager para el grupo de administración que se va a evaluar
* Si la cuenta no tiene derechos de administrador del sistema SQL, ejecute el [script](#sql-script-to-grant-granular-permissions-to-the-run-as-account) para conceder permisos específicos a la cuenta en cada instancia de SQL Server que hospede una o todas las bases de datos de Operations Manager.

1. En la consola de Operations Manager, seleccione el botón de navegación de **Administración**.
2. En **Run As Configuration** (Configuración de ejecución), haga clic en **Cuentas**.
3. En el Asistente **Crear cuenta de ejecución**, en la página **Introducción**, haga clic en **Siguiente**.
4. En la página **Propiedades generales**, seleccione **Windows** en la lista **Tipo de cuenta de ejecución:** .
5. Escriba un nombre para mostrar en el cuadro de texto **Nombre para mostrar** y, opcionalmente, escriba una descripción en el cuadro **Descripción**; a continuación, haga clic en **Siguiente**.
6. En **Seguridad de distribución**, seleccione **Más seguro**.
7. Haga clic en **Crear**.  

Ahora que se creó la cuenta de ejecución, es necesario asignar servidores de administración de destino en el grupo de administración y asociarlos a un perfil de ejecución predefinido para que los flujos de trabajo se ejecuten con las credenciales.  

1. En **Run As Configuration** (Configuración de ejecución), en **Cuentas**, en el panel de resultados, haga doble clic en la cuenta que creó anteriormente.
2. En la pestaña **Distribución**, haga clic en **Agregar** para el cuadro **Equipos seleccionados** y agregue el servidor de administración al que distribuir la cuenta.  Haga clic en **Aceptar** dos veces para guardar los cambios.
3. En **Run As Configuration** (Configuración de ejecución), haga clic en **Perfiles**.
4. Busque *SCOM Assessment Profile* (Perfil de evaluación de SCOM).
5. El nombre de perfil debe ser: *Microsoft System Center Operations Manager Health Check Run As Profile* (Perfil de ejecución de Microsoft System Center Operations Manager Health Check).
6. Haga clic con el botón derecho, actualice sus propiedades y agregue la cuenta de ejecución que creó antes.

### <a name="sql-script-to-grant-granular-permissions-to-the-run-as-account"></a>Concesión de permisos específicos a la cuenta de ejecución mediante un script SQL

Ejecute el siguiente script SQL para conceder los permisos necesarios a la cuenta de ejecución en la instancia de SQL Server usada por Operations Manager que hospeda la base de datos de ACS, de almacén de datos y operativa.

```
-- Replace <UserName> with the actual user name being used as Run As Account.
USE master

-- Create login for the user, comment this line if login is already created.
CREATE LOGIN [UserName] FROM WINDOWS


--GRANT permissions to user.
GRANT VIEW SERVER STATE TO [UserName]
GRANT VIEW ANY DEFINITION TO [UserName]
GRANT VIEW ANY DATABASE TO [UserName]

-- Add database user for all the databases on SQL Server Instance, this is required for connecting to individual databases.
-- NOTE: This command must be run anytime new databases are added to SQL Server instances.
EXEC sp_msforeachdb N'USE [?]; CREATE USER [UserName] FOR LOGIN [UserName];'

Use msdb
GRANT SELECT To [UserName]
Go

--Give SELECT permission on all Operations Manager related Databases

--Replace the Operations Manager database name with the one in your environment
Use [OperationsManager];
GRANT SELECT To [UserName]
GO

--Replace the Operations Manager DatawareHouse database name with the one in your environment
Use [OperationsManagerDW];
GRANT SELECT To [UserName]
GO

--Replace the Operations Manager Audit Collection database name with the one in your environment
Use [OperationsManagerAC];
GRANT SELECT To [UserName]
GO

--Give db_owner on [OperationsManager] DB
--Replace the Operations Manager database name with the one in your environment
USE [OperationsManager]
GO
ALTER ROLE [db_owner] ADD MEMBER [UserName]

```

### <a name="configure-the-health-check-rule"></a>Configuración de la regla de comprobación de estado

El paquete de administración de la solución System Center Operations Manager Health Check incluye una regla denominada *Microsoft System Center Operations Manager Run Health Check Rule* (Regla de comprobación de estado de Microsoft System Center Operations Manager). Esta regla es responsable de ejecutar la comprobación de estado. Para habilitar la regla y configurar la frecuencia, use los procedimientos siguientes.

De forma predeterminada, la regla de comprobación de estado de Microsoft System Center Operations Manager está deshabilitada. Para ejecutar la comprobación de estado, debe habilitar la regla en un servidor de administración. Para ello, siga los pasos que se describen a continuación.

#### <a name="enable-the-rule-for-a-specific-management-server"></a>Habilitación de la regla para un servidor de administración específico

1. En el área de trabajo **Creación** de la consola del operador de Operations Manager, busque la regla *Microsoft System Center Operations Manager Run Health Check Rule* (Regla de comprobación de estado de Microsoft System Center Operations Manager) en el panel **Reglas**.
2. En los resultados de búsqueda, seleccione el que incluye el texto *Tipo: Servidor de administración*.
3. Haga clic con el botón derecho en la regla y, a continuación, haga clic en **Invalidaciones** > **Para un objeto de clase específico: Servidor de administración**.
4.  En la lista de servidores de administración disponibles, seleccione el servidor de administración donde se debe ejecutar la regla.  Debe ser el mismo servidor de administración que configuró anteriormente al que asociar la cuenta de ejecución.
5.  Asegúrese de cambiar el valor de la invalidación a **True** para el valor del parámetro **Habilitado**.<br><br> ![invalidar parámetro](./media/scom-assessment/rule.png)

    Mientras sigue en esta ventana, configure la frecuencia de la ejecución con el procedimiento siguiente.

#### <a name="configure-the-run-frequency"></a>Configuración de la frecuencia de ejecución

La evaluación está configurada para ejecutarse cada 10.080 minutos (o siete días) de forma predeterminada. Puede cambiar el valor por el valor mínimo de 1440 minutos (o un día). El valor representa el intervalo de tiempo mínimo necesario entre ejecuciones de evaluación sucesivas. Para invalidar el intervalo, siga estos pasos.

1. En el área de trabajo **Creación** de la consola de Operations Manager, busque la regla *Microsoft System Center Operations Manager Run Health Check Rule* (Regla de comprobación de estado de Microsoft System Center Operations Manager) en la sección **Reglas**.
2. En los resultados de búsqueda, seleccione el que incluye el texto *Tipo: Servidor de administración*.
3. Haga clic con el botón derecho en la regla y, a continuación, haga clic en **Invalidar la regla** > **Para todos los objetos de clase: Servidor de administración**.
4. Cambie el valor del parámetro **Intervalo** por el valor de intervalo deseado. En el ejemplo siguiente, el valor se establece en 1440 minutos (un día).<br><br> ![parámetro de intervalo](./media/scom-assessment/interval.png)<br>  

    Si el valor se establece en menos de 1440 minutos, la regla se ejecuta en un intervalo de un día. En este ejemplo, la regla omite el valor del intervalo y se ejecuta con una frecuencia de un día.


## <a name="understanding-how-recommendations-are-prioritized"></a>Cómo se establecen prioridades entre las recomendaciones

A cada recomendación efectuada se le asigna un valor de ponderación que identifica su importancia relativa. Se muestran solo las diez recomendaciones más importantes.

### <a name="how-weights-are-calculated"></a>Cálculo de las ponderaciones

Las ponderaciones son valores agregados en función de tres factores principales:

- La *probabilidad* de que un asunto identificado pueda causar problemas. Una probabilidad más alta equivale a una puntuación total mayor para la recomendación.
- El *impacto* del asunto en su organización en caso de que se produzca un problema. Un mayor impacto equivale a una puntuación total mayor para la recomendación.
- El *esfuerzo* necesario para implementar la recomendación. Un mayor esfuerzo equivale a una puntuación total menor para la recomendación.

La ponderación de cada recomendación se expresa como un porcentaje de la puntuación total disponible para cada área de enfoque. Por ejemplo, si una recomendación en el área de enfoque Disponibilidad y continuidad empresarial tiene una puntuación del 5 %, implementar esa recomendación aumenta la puntuación total de Disponibilidad y continuidad empresarial un 5 %.

### <a name="focus-areas"></a>Áreas de enfoque

**Disponibilidad y continuidad empresarial** : este apartado muestra recomendaciones relacionadas con la disponibilidad de servicio, la resistencia de la infraestructura y la protección del negocio.

**Rendimiento y escalabilidad** : este apartado muestra recomendaciones que ayudarán a crecer a la infraestructura de TI de su organización y garantizarán que el entorno de TI cumple los requisitos de rendimiento vigentes y que, además, puede responder a las cambiantes necesidades de infraestructura.

**Actualización, migración e implementación**. Esta área de enfoque muestra recomendaciones que lo ayudarán a actualizar, migrar e implementar SQL Server en la infraestructura existente.

**Operaciones y supervisión**. Esta área de enfoque muestra recomendaciones para ayudar a simplificar las operaciones de TI, implementar el mantenimiento preventivo y maximizar el rendimiento.

### <a name="should-you-aim-to-score-100-in-every-focus-area"></a>¿Debe tratar de conseguir una puntuación del 100 % en cada área de enfoque?

No necesariamente. Las recomendaciones se basan en los conocimientos y las experiencias adquiridos por los ingenieros de Microsoft producto de miles de visitas de clientes. Sin embargo, no hay dos infraestructuras de servidores que sean iguales, y es posible que determinadas recomendaciones puedan ser más o menos relevantes para usted. Por ejemplo, algunas recomendaciones de seguridad pueden ser menos pertinentes si las máquinas virtuales no están expuestas a Internet. Algunas recomendaciones de disponibilidad pueden ser menos relevantes para los servicios que proporcionan informes y recopilaciones de datos ad hoc de baja prioridad. Los problemas que son importantes para un negocio maduro pueden no serlo para otro que esté en sus inicios. Puede que desee identificar qué áreas de enfoque son prioritarias para usted y, posteriormente, observar cómo cambian las puntuaciones con el tiempo.

Cada recomendación incluye pautas que indican por qué es importante. Use estas directrices para evaluar si es adecuado o no implementar la recomendación, en función de la naturaleza de los servicios de TI y las necesidades empresariales de su organización.

## <a name="use-health-check-focus-area-recommendations"></a>Uso de las recomendaciones del área centradas en la comprobación de estado

Para poder usar una solución de comprobación de estado en Log Analytics, debe tenerla instalada. Para obtener más información sobre cómo instalar soluciones, consulte [Install a management solution](./solutions.md) (Instalar una solución de administración). Una vez instalada, puede ver el resumen de las recomendaciones mediante el icono de System Center Operations Manager Health Check en la página **Información general** del área de trabajo en Azure Portal.

Consulte un resumen de las evaluaciones de cumplimiento para su infraestructura y, a continuación, profundice las recomendaciones.

### <a name="to-view-recommendations-for-a-focus-area-and-take-corrective-action"></a>Visualización de las recomendaciones para un área de enfoque y adopción de las medidas correctivas
1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En Azure Portal, haga clic en **Más servicios**, en la esquina inferior izquierda. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Log Analytics**.
3. En el panel de suscripciones de Log Analytics, seleccione un área de trabajo y haga clic en el elemento de menú **Resumen del área de trabajo**.  
4. En la página **Información general**, haga clic en el icono de **System Center Operations Manager Health Check**.
5. En la página **System Center Operations Manager Health Check**, revise la información resumida de una de las hojas de las áreas de interés y, a continuación, haga clic en una de ellas para ver las recomendaciones para dicha área.
6. En cualquiera de las páginas de área de enfoque, puede ver las recomendaciones priorizadas que se han efectuado para su entorno. Haga clic en una recomendación en **Objetos afectados** para ver los detalles sobre por qué se realiza la recomendación.<br><br> ![área de enfoque](./media/scom-assessment/log-analytics-scom-healthcheck-dashboard-02.png)<br>
7. Puede tomar las medidas correctivas que se sugieren en **Acciones sugeridas**. Cuando se haya ocupado del asunto, las evaluaciones posteriores registrarán las acciones recomendadas que se han realizado y aumentará su calificación de cumplimiento normativo. Los asuntos que se hayan corregido aparecerán en **Objetos superados**.

## <a name="ignore-recommendations"></a>Omisión de las recomendaciones

Si desea omitir ciertas recomendaciones, puede crear un archivo de texto que Log Analytics usa para evitar que aparezcan las recomendaciones en los resultados de la evaluación.

### <a name="to-identify-recommendations-that-you-want-to-ignore"></a>Para identificar las recomendaciones que desea omitir
1. En Azure Portal, en la página del área de trabajo de Log Analytics correspondiente al área de trabajo seleccionada, haga clic en el elemento de menú **Búsqueda de registros**.
2. Use la siguiente consulta para mostrar las recomendaciones para los equipos que presentan errores en el entorno.

    ```
    Type=SCOMAssessmentRecommendationRecommendationResult=Failed | select Computer, RecommendationId, Recommendation | sort Computer
    ```

    >[!NOTE]
    > Si el área de trabajo se ha actualizado al [nuevo lenguaje de consulta de Log Analytics](../log-query/log-query-overview.md), la consulta anterior cambiaría como sigue.
    >
    > `SCOMAssessmentRecommendationRecommendation | where RecommendationResult == "Failed" | sort by Computer asc | project Computer, RecommendationId, Recommendation`

    En esta captura de pantalla se muestra la consulta Búsqueda de registros:<br><br> ![búsqueda de registros](./media/scom-assessment/scom-log-search.png)<br>

3. Elija las recomendaciones que desea omitir. Usará los valores para RecommendationId en el procedimiento siguiente.

### <a name="to-create-and-use-an-ignorerecommendationstxt-text-file"></a>Para crear y usar un archivo de texto IgnoreRecommendations.txt

1. Cree un archivo llamado IgnoreRecommendations.txt.
2. Pegue o escriba cada RecommendationId para cada recomendación que desee que Log Analytics omita en una línea independiente y, luego, guarde y cierre el archivo.
3. Coloque el archivo en la carpeta siguiente en cada equipo donde desea que Log Analytics omita las recomendaciones.
4. En el servidor de administración de Operations Manager: *UnidadDelSistema*:\Archivos de programa\Microsoft System Center 2012 R2\Operations Manager\Server.

### <a name="to-verify-that-recommendations-are-ignored"></a>Para comprobar que se omiten las recomendaciones

1. Después de que se ejecute la siguiente evaluación programada, de forma predeterminada cada siete días, las recomendaciones especificadas se marcan como omitidas y no aparecerán en el panel de comprobación de estado.
2. Puede usar las consultas de búsqueda de registros siguientes para enumerar todas las recomendaciones omitidas.

    ```
    Type=SCOMAssessmentRecommendationRecommendationResult=Ignored | select  Computer, RecommendationId, Recommendation | sort  Computer
    ```

    >[!NOTE]
    > Si el área de trabajo se ha actualizado al [nuevo lenguaje de consulta de Log Analytics](../log-query/log-query-overview.md), la consulta anterior cambiaría como sigue.
    >
    > `SCOMAssessmentRecommendationRecommendation | where RecommendationResult == "Ignore" | sort by Computer asc | project Computer, RecommendationId, Recommendation`

3. Si posteriormente decide que desea ver las recomendaciones omitidas, quite todos los archivos IgnoreRecommendations.txt. También puede quitar RecommendationID de ellos.

## <a name="system-center-operations-manager-health-check-solution-faq"></a>P+F de la solución System Center Operations Manager Health Check

*Agregué la solución de comprobación de estado a mi área de trabajo de Log Analytics. Pero no veo las recomendaciones. ¿Por qué no?* Después, utilice los pasos siguiente para ver las recomendaciones del panel de Log Analytics.  

- [Establecimiento de la cuenta de ejecución de System Center Operations Manager Health Check](#operations-manager-run-as-accounts-for-log-analytics)  
- [Configuración de la regla de System Center Operations Manager Health Check](#configure-the-health-check-rule)


*¿Se puede configurar la frecuencia con la que se realiza la comprobación?* Sí. Consulte [Configuración de la frecuencia de ejecución](#configure-the-run-frequency).

*Si se detecta otro servidor después de haber agregado la solución System Center Operations Manager Health Check, ¿se comprobará?* Sí, después de la detección, se comprobará cada siete días de forma predeterminada.

*¿Cuál es el nombre del proceso que realiza la recopilación de datos?* AdvisorAssessment.exe

*¿Dónde se ejecuta el proceso AdvisorAssessment.exe?* AdvisorAssessment.exe se ejecuta en el proceso HealthService del servidor de administración donde está habilitada la regla de comprobación de estado. Con ese proceso, la detección de todo el entorno se realiza mediante la recopilación de datos remotos.

*¿Cuánto tiempo tarda en recopilar datos?* La recopilación de datos en el servidor tarda aproximadamente una hora. Puede tardar más en entornos que tienen muchas instancias o bases de datos de Operations Manager.

*¿Qué ocurre si establezco el intervalo de la evaluación en menos de 1440 minutos?* La evaluación está preconfigurada para que se ejecute un máximo de una vez al día. Si invalida el valor de intervalo en un valor inferior a 1440 minutos, la evaluación usa 1440 minutos como valor de intervalo.

*¿Cómo se sabe si se cumplen los requisitos previos?* Si la comprobación de estado se ejecutó y no ve los resultados, es probable que no cumpla algunos de sus requisitos previos. Puede ejecutar consultas: `Operation Solution=SCOMAssessment` y `SCOMAssessmentRecommendation FocusArea=Prerequisites` en la búsqueda de registros para ver qué requisitos previos no se cumplen.

*Los requisitos previos que no se cumplen muestran el mensaje `Failed to connect to the SQL Instance (….).`. ¿Cuál es el problema?* AdvisorAssessment.exe, el proceso que recopila los datos, se ejecuta en el proceso HealthService del servidor de administración. Como parte de la comprobación de estado, el proceso intenta conectarse a la instancia de SQL Server donde se encuentra la base de datos de Operations Manager. Este error puede producirse cuando las reglas de firewall bloquean la conexión a la instancia de SQL Server.

*¿Qué tipo de datos se recopila?* Se recopilan los siguientes tipos de datos: datos de WMI, datos del Registro, datos del registro de eventos, datos de Operations Manager mediante Windows PowerShell, consultas SQL y el recopilador de información de consultas y archivos.

*¿Por qué es necesario configurar una cuenta "ejecutar como"?* Con Operations Manager se ejecutan varias consultas SQL. Para que se ejecuten, debe usar una cuenta de ejecución con los permisos necesarios. Además, para consultar WMI, se necesitan credenciales de administrador local.

*¿Por qué se muestran solo las 10 recomendaciones principales?* En lugar de darle una lista exhaustiva y abrumadora de tareas, se recomienda centrarse primero en las recomendaciones prioritarias. Después de aplicarlas, se mostrarán más recomendaciones. Si prefiere ver una lista detallada, puede ver todas las recomendaciones mediante Búsqueda de registros.

*¿Se puede hacer caso omiso de una recomendación?* Sí, consulte [Omisión de las recomendaciones](#ignore-recommendations).


## <a name="next-steps"></a>Pasos siguientes

- [Búsquedas de registros](../log-query/log-query-overview.md) para obtener más información sobre cómo analizar recomendaciones y datos de System Center Operations Manager Health Check.

