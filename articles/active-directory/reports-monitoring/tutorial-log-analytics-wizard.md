---
title: Configuración del asistente de Log Analytics en Azure AD | Microsoft Docs
description: Aprenda a configurar Log Analytics.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 08/05/2020
ms.author: markvi
author: MarkusVi
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 93caf52d8b4a11f9843ad5f18ebf968d1d0730cd
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226214"
---
# <a name="tutorial-configure-the-log-analytics-wizard"></a>Tutorial: Configuración del asistente de Log Analytics


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar un área de trabajo de Log Analytics para los registros de auditoría e inicio de sesión
> * Ejecutar consultas mediante el lenguaje de consulta de Kusto (KQL)
> * Crear una regla de alertas que envíe alertas cuando se use una cuenta específica
> * Crear un libro personalizado mediante la plantilla de inicio rápido
> * Agregar una consulta a una plantilla de libro existente

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción de Azure con al menos un administrador con licencia P1. Si no tiene ninguna suscripción de Azure, puede [registrarse para obtener una evaluación gratuita](https://azure.microsoft.com/free/).

- Un inquilino de Azure AD.

- Un usuario que sea administrador global o administrador de seguridad para el inquilino de Azure AD.


Familiarícese con estos artículos:

- [Tutorial: Recopilación y análisis de registros de recurso desde un recurso de Azure](../../azure-monitor/learn/tutorial-resource-logs.md)

- [Integración de registros de actividad con Log Analytics](./howto-integrate-activity-logs-with-log-analytics.md)

- [Administración de cuentas de acceso de emergencia en Azure AD](../users-groups-roles/directory-emergency-access.md)

- [Referencia rápida de KQL](/azure/data-explorer/kql-quick-reference)

- [Libros de Azure Monitor](../../azure-monitor/platform/workbooks-overview.md)



## <a name="configure-a-workspace"></a>Configuración de un área de trabajo 

En este procedimiento se indica cómo configurar un área de trabajo de Log Analytics para los registros de auditoría e inicio de sesión.
La configuración de un área de trabajo de Log Analytics consta de dos pasos principales:
 
1. Creación de un área de trabajo de Log Analytics
2. Actualización de la configuración de diagnóstico

**Para configurar un área de trabajo:** 


1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.

2. Busque **áreas de trabajo de Log Analytics**.

    ![Buscar recursos, servicios y documentos](./media/tutorial-log-analytics-wizard/search-services.png)

3. En la página Áreas de trabajo de Log Analytics, haga clic en **Agregar**.

    ![Sumar](./media/tutorial-log-analytics-wizard/add.png)

4.  En la página **Crear un área de trabajo de Log Analytics**, siga estos pasos:

    ![Creación de un área de trabajo de análisis de registros](./media/tutorial-log-analytics-wizard/create-log-analytics-workspace.png)

    1. Seleccione su suscripción.

    2. Seleccione un grupo de recursos.
 
    3. En el cuadro de texto **Nombre**, escriba un nombre (por ejemplo: MiÁreaDeTrabajoDePrueba1).

    4. Seleccione la región.

5. Haga clic en **Revisar y crear**.

    ![Revisar y crear](./media/tutorial-log-analytics-wizard/review-create.png)

6. Haga clic en **Crear** y espere a que la implementación se complete correctamente. Puede que tenga que actualizar la página para ver la nueva área de trabajo.

    ![Crear](./media/tutorial-log-analytics-wizard/create-workspace.png)

7. Busque **Azure Active Directory**.

    ![Azure Active Directory](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

8. En la sección **Supervisión**, haga clic en **Configuración de diagnóstico**.

    ![Azure Active Directory](./media/tutorial-log-analytics-wizard/diagnostic-settings.png)

9. En la página **Configuración de diagnóstico**, haga clic en **Agregar configuración de diagnóstico**.

    ![Incorporación de la configuración de diagnóstico](./media/tutorial-log-analytics-wizard/add-diagnostic-setting.png)

10. En la página **Configuración de diagnóstico**, realice los pasos siguientes:

    ![Seleccionar Configuración de diagnóstico](./media/tutorial-log-analytics-wizard/select-diagnostics-settings.png)

    1. En **Detalles de la categoría**, seleccione **AuditLogs** y **SigninLogs**.

    2. En **Detalles de destino**, seleccione **Enviar a Log Analytics** y, a continuación, seleccione la nueva área de trabajo de Log Analytics. 
   
    3. Haga clic en **Save**(Guardar). 

## <a name="run-queries"></a>Ejecutar consultas  

En este procedimiento se describe cómo ejecutar consultas mediante el **lenguaje de consulta de Kusto (KQL)** .


**Para ejecutar una consulta:**


1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.

2. Busque **Azure Active Directory**.

    ![Azure Active Directory](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. En la sección **Supervisión**, haga clic en **Registros**.

4. En la página **Registros**, haga clic en **Iniciar**.

5. En el cuadro de texto **Buscar*, escriba la consulta.

6. Haga clic en **Ejecutar**.  


### <a name="kql-query-examples"></a>Ejemplos de consultas de KQL

Tome 10 entradas aleatorias de los datos de entrada:

`SigninLogs | take 10`

Consulte los inicios de sesión en los que el acceso condicional fue correcto

`SigninLogs | where ConditionalAccessStatus == "success" | project UserDisplayName, ConditionalAccessStatus` 


Cuente el número de accesos correctos que se han producido

`SigninLogs | where ConditionalAccessStatus == "success" | project UserDisplayName, ConditionalAccessStatus | count`


Agregue el recuento de inicios de sesión correctos por usuario por día:

`SigninLogs | where ConditionalAccessStatus == "success" | summarize SuccessfulSign-ins = count() by UserDisplayName, bin(TimeGenerated, 1d)` 


Vea el número de veces que un usuario realiza una determinada operación en un período de tiempo específico:

`AuditLogs | where TimeGenerated > ago(30d) | where OperationName contains "Add member to role" | summarize count() by OperationName, Identity`


Dinamice los resultados sobre el nombre de la operación

`AuditLogs | where TimeGenerated > ago(30d) | where OperationName contains "Add member to role" | project OperationName, Identity | evaluate pivot(OperationName)`


Combine los registros de auditoría e inicio de sesión mediante una combinación interna:

`AuditLogs |where OperationName contains "Add User" |extend UserPrincipalName = tostring(TargetResources[0].userPrincipalName) | |project TimeGenerated , UserPrincipalName |join kind = inner (SigninLogs) on UserPrincipalName |summarize arg_min(TimeGenerated, *) by UserPrincipalName |extend SigninDate = TimeGenerated` 


Vea el número de inicios de sesión por tipo de aplicación cliente:

`SigninLogs | summarize count() by ClientAppUsed`

Cuente los inicios de sesión por día:

`SigninLogs | summarize NumberOfEntries=count() by bin(TimeGenerated, 1d)`

Tome 5 entradas aleatorias e incluya las columnas que desea ver en los resultados:

`SigninLogs | take 5 | project ClientAppUsed, Identity, ConditionalAccessStatus, Status, TimeGenerated `


Tome las 5 entradas principales en orden descendente e incluya las columnas que desea ver

`SigninLogs | take 5 | project ClientAppUsed, Identity, ConditionalAccessStatus, Status, TimeGenerated `

Cree una nueva columna combinando los valores de otras dos columnas:

`SigninLogs | limit 10 | extend RiskUser = strcat(RiskDetail, "-", Identity) | project RiskUser, ClientAppUsed`




## <a name="create-an-alert-rule"></a>Crear una regla de alerta  

Este procedimiento muestra cómo enviar alertas cuando se usa la cuenta de BreakGlass.

**Para crear una regla de alertas:**


1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.

2. Busque **Azure Active Directory**.

    ![Azure Active Directory](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. En la sección **Supervisión**, haga clic en **Registros**.

4. En la página **Registros**, haga clic en **Iniciar**.

5. En el cuadro de texto **Buscar**, escriba: `SigninLogs |where UserDisplayName contains "BreakGlass" | project UserDisplayName`

6. Haga clic en **Ejecutar**.  

7. En la barra de herramientas, haga clic en **Nueva regla de alertas**.

    ![Nueva alerta de reglas](./media/tutorial-log-analytics-wizard/new-alert-rule.png)

8. En la página **Crear regla de alertas**, compruebe que el ámbito es correcto.

9. En **Condición**, haga clic en: **Siempre que la búsqueda de registros media personalizada sea mayor que el recuento de <logic undefined>**

    ![Condición predeterminada](./media/tutorial-log-analytics-wizard/default-condition.png)

10. En la página **Configurar lógica de señal**, en la sección **Lógica de alerta**, realice los pasos siguientes:

    ![Lógica de alerta](./media/tutorial-log-analytics-wizard/alert-logic.png)

    1. En **Basado en**, seleccione **Número de resultados**.

    2. Como **Operador**, seleccione **Mayor que**.

    3. En **Valor del umbral**, seleccione **0**. 

11. En la página **Configurar lógica de señal**, en la sección **Se evaluó basándose en**, realice los pasos siguientes:

    ![Se evaluó basándose en](./media/tutorial-log-analytics-wizard/evaluated-based-on.png)

    1. En **Período (en minutos)** , seleccione **5**.

    2. En **Frecuencia (en minutos)** , seleccione **5**.

    3. Haga clic en **Done**(Listo). 

12. En **Grupo de acciones**, haga clic en **Seleccionar grupo de acciones**. 

    ![Grupo de acciones](./media/tutorial-log-analytics-wizard/action-group.png)

13. En **Seleccionar un grupo de acciones para asociar a esta regla de alerta**, haga clic en **Crear grupo de acciones**. 

    ![Creación de un grupo de acciones](./media/tutorial-log-analytics-wizard/create-action-group.png)

14. En la página **Crear grupo de acciones**, siga estos pasos:

    ![Detalles de instancia](./media/tutorial-log-analytics-wizard/instance-details.png)

    1. En el cuadro de texto **Nombre del grupo de acciones**, escriba **Mi grupo de acciones**.

    2. En el cuadro de texto **Nombre para mostrar**, escriba **Mi acción**.

    3. Haga clic en **Revisar + crear**. 

    4. Haga clic en **Crear**.


15. En **Personalizar acción**, realice los siguientes pasos:

    ![Personalizar acciones](./media/tutorial-log-analytics-wizard/customize-actions.png)

    1. Seleccione **Asunto del correo electrónico**.

    2. En el cuadro de texto **Línea de asunto**, escriba: `Breakglass account has been used`

16. En **Detalles de la regla de alertas**, lleve a cabo los pasos siguientes:

    ![Detalles de la regla de alertas](./media/tutorial-log-analytics-wizard/alert-rule-details.png)

    1. En el cuadro de texto **Nombre de la regla de alertas**, escriba: `Breakglass account`

    2. En el cuadro de texto **Descripción**, escriba: `Your emergency access account has been used`

17. Haga clic en **Crear regla de alertas**.   


## <a name="create-a-custom-workbook"></a>Creación de un libro personalizado

En este procedimiento se muestra cómo crear un nuevo libro mediante la plantilla de inicio rápido.




1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.

2. Busque **Azure Active Directory**.

    ![Azure Active Directory](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. En la sección **Supervisión**, haga clic en **Libros**.

    ![Workbooks](./media/tutorial-log-analytics-wizard/workbooks.png)

4. En la sección **Inicio rápido**, haga clic en **Vacío**.

    ![Inicio rápido](./media/tutorial-log-analytics-wizard/quick-start.png)

5. Haga clic en **Agregar**.

    ![Agregar libro](./media/tutorial-log-analytics-wizard/add-workbook.png)

6. Haga clic en **Agregar texto**.

    ![Agregar texto](./media/tutorial-log-analytics-wizard/add-text.png)


7. En el cuadro de texto, escriba: `# Client apps used in the past week` y, a continuación, haga clic en **Edición finalizada**.

    ![Texto del libro](./media/tutorial-log-analytics-wizard/workbook-text.png)

8. En el nuevo libro, haga clic en **Agregar** y, a continuación, haga clic en **Agregar consulta**.

    ![Agregar consulta](./media/tutorial-log-analytics-wizard/add-query.png)

9. En el cuadro de texto de consulta, escriba: `SigninLogs | where TimeGenerated > ago(7d) | project TimeGenerated, UserDisplayName, ClientAppUsed | summarize count() by ClientAppUsed`

10. Haga clic en **Ejecutar consulta**.

    ![Ejecutar consulta](./media/tutorial-log-analytics-wizard/run-workbook-query.png)

11. En la barra de herramientas, en **Visualización**, haga clic en **Gráfico circular**.

    ![Gráfico circular](./media/tutorial-log-analytics-wizard/pie-chart.png)

12. Haga clic en **Edición finalizada**.

    ![Edición finalizada](./media/tutorial-log-analytics-wizard/done-workbook-editing.png)



## <a name="add-a-query-to-a-workbook-template"></a>Incorporación de una consulta a una plantilla de libro  

Este procedimiento muestra cómo agregar una consulta a una plantilla de libro existente. El ejemplo se basa en una consulta que muestra la distribución de los errores de acceso condicional.


1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.

2. Busque **Azure Active Directory**.

    ![Azure Active Directory](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. En la sección **Supervisión**, haga clic en **Libros**.

    ![Workbooks](./media/tutorial-log-analytics-wizard/workbooks.png)

4. En la sección **acceso condicional**, haga clic en **Información detallada e informes del acceso condicional**.

    ![Plantilla de acceso condicional](./media/tutorial-log-analytics-wizard/conditional-access-template.png)

5. En la barra de herramientas, haga clic en **Editar**.

    ![Plantilla de acceso condicional](./media/tutorial-log-analytics-wizard/edit-workbook-template.png)

6. En la barra de herramientas, haga clic en los tres puntos, luego, en **Agregar** y, finalmente, en **Agregar consulta**.

    ![Agregar consulta del libro](./media/tutorial-log-analytics-wizard/add-custom-workbook-query.png)

7. En el cuadro de texto de consulta, escriba: `SigninLogs | where TimeGenerated > ago(20d) | where ConditionalAccessPolicies != "[]" | summarize dcount(UserDisplayName) by bin(TimeGenerated, 1d), ConditionalAccessStatus`

8. Haga clic en **Ejecutar consulta**.

    ![Ejecutar consulta](./media/tutorial-log-analytics-wizard/run-workbook-insights-query.png)

9. Haga clic en **Intervalo de tiempo** y, después, elija **Establecer en la consulta**.

10. Haga clic en **Visualización** y, a continuación, seleccione **Gráfico de barras**. 

11. Haga clic en **Configuración avanzada**, como título del gráfico, escriba `Conditional Access status over the last 20 days` y, a continuación, haga clic en **Edición finalizada**. 

    ![Establecer título del gráfico](./media/tutorial-log-analytics-wizard/set-chart-title.png)








## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo para aprender a administrar las identidades de dispositivo mediante Azure Portal.
> [!div class="nextstepaction"]
> [Supervisión](overview-monitoring.md)