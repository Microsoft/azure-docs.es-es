---
title: 'Libro Conditional Access insights and reporting: Azure Active Directory'
description: Uso del libro Conditional Access insights and reporting (Información detallada e informes del acceso condicional) de Azure AD para solucionar problemas de directivas
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 08/27/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3c2364eae0d04da8f8e6fe38ae80db7adb8666ce
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89049424"
---
# <a name="conditional-access-insights-and-reporting"></a>Información detallada e informes del acceso condicional

El libro Conditional Access insights and reporting (Información detallada e informes del acceso condicional) le permite comprender el impacto de las directivas de acceso condicional en su organización a lo largo del tiempo. Durante el inicio de sesión, se pueden aplicar una o varias directivas de acceso condicional, de modo que se conceda acceso si se cumplen determinados controles de concesión o se deniegue de lo contrario. Dado que se pueden evaluar varias directivas de acceso condicional durante cada inicio de sesión, el libro de información detallada e informes le permite examinar el impacto de una directiva individual o un subconjunto de todas las directivas.  

## <a name="prerequisites"></a>Requisitos previos

Para habilitar el libro de información detallada e informes, el inquilino debe tener un área de trabajo de Log Analytics para conservar los datos de los registros de inicio de sesión. Los usuarios deben tener licencias de Azure AD Premium P1 o P2 para usar el acceso condicional.

Los siguientes roles pueden acceder a información detallada e informes:  

- Administrador de acceso condicional 
- Lector de seguridad 
- Administrador de seguridad 
- Lector global 
- Administrador global 

Los usuarios también necesitan uno de los siguientes roles de área de trabajo de Log Analytics:  

- Colaborador  
- Propietario 

### <a name="stream-sign-in-logs-from-azure-ad-to-azure-monitor-logs"></a>Transmisión de los registros de inicio de sesión de Azure AD a los registros de Azure Monitor 

Si no ha integrado los registros de Azure AD con los registros de Azure Monitor, deberá realizar los pasos siguientes para que se cargue el libro:  

1. [Crear un área de trabajo de Log Analytics en Azure Monitor](../../azure-monitor/learn/quick-create-workspace.md).
1. [Integrar los registros de Azure AD con los registros de Azure Monitor](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md).

## <a name="how-it-works"></a>Funcionamiento 

Para acceder al libro de información detallada e informes:  

1. Inicie sesión en **Azure Portal**.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional** > **Insights and reporting** (Información detallada e informes).

### <a name="get-started-select-parameters"></a>Primeros pasos: Selección de parámetros 

El panel Insights and reporting (Información detallada e informes) le permite ver el impacto de una o varias directivas de acceso condicional durante un período especificado. Empiece por establecer cada uno de los parámetros en la parte superior del libro. 

![Panel Conditional Access Insights and Reporting de Azure Portal](./media/howto-conditional-access-insights-reporting/conditional-access-insights-and-reporting-dashboard.png)

**Directiva de acceso condicional**: seleccione una o varias directivas de acceso condicional para ver su impacto combinado. Las directivas se dividen en dos grupos: directivas habilitadas y de solo informe. De forma predeterminada, todas las directivas habilitadas están seleccionadas. Estas directivas habilitadas son las directivas que se aplican actualmente en su inquilino.  

**Intervalo de tiempo**: seleccione un intervalo de tiempo de 4 horas a un máximo de 90 días. Si selecciona un intervalo de tiempo anterior a la integración de los registros de Azure AD con Azure Monitor, solo se mostrarán los inicios de sesión posteriores a la integración.  

**Usuario**: de forma predeterminada, el panel muestra el impacto de las directivas seleccionadas para todos los usuarios. Para filtrar por un usuario individual, escriba el nombre del usuario en el campo de texto. Para filtrar por todos los usuarios, escriba "todos los usuarios" en el campo de texto o deje el parámetro vacío. 

**Aplicación**: de forma predeterminada, el panel muestra el impacto de las directivas seleccionadas para todas las aplicaciones. Para filtrar por una aplicación individual, escriba el nombre de la aplicación en el campo de texto. Para filtrar por todas las aplicaciones, escriba "todas las aplicaciones" en el campo de texto o deje el parámetro vacío. 

**Vista de datos**: seleccione si desea que el panel muestre los resultados en función del número de usuarios o del número de inicios de sesión. Un usuario individual puede tener cientos de inicios de sesión en muchas aplicaciones con muchos resultados diferentes durante un intervalo de tiempo determinado. Si selecciona la vista de datos de usuarios, un usuario se puede incluir en los recuentos correctos y erróneos (por ejemplo, si hay 10 usuarios, 8 de ellos podrían haber tenido el resultado correcto los últimos 30 días y 9 de ellos podrían haber obtenido un resultado erróneo los últimos 30 días).

## <a name="impact-summary"></a>Resumen de impacto 

Una vez establecidos los parámetros, se carga el resumen de impacto. En el resumen se muestra el número de usuarios o inicios de sesión durante el intervalo de tiempo tuvieron el resultado "Correcto", "Error", "Se requiere una acción del usuario" o "No aplicado" cuando se evaluaron las directivas seleccionadas.  

![Resumen de impacto en el libro Acceso condicional](./media/howto-conditional-access-insights-reporting/workbook-impact-summary.png)

**Total**: número de usuarios o inicios de sesión durante el período de tiempo en que se evaluó al menos una de las directivas seleccionadas.

**Correcto**: número de usuarios o inicios de sesión durante el período de tiempo en que el resultado combinado de las directivas seleccionadas fue "Correcto" o "Solo informe": Correcto".

**Error**: número de usuarios o inicios de sesión durante el período de tiempo en que el resultado de al menos una de las directivas seleccionadas fue "Error" o "Solo informe": Error".

**Se requiere una acción del usuario**: número de usuarios o inicios de sesión durante el período de tiempo en que el resultado combinado de las directivas seleccionadas fue "Solo informe": Se requiere una acción del usuario". La acción del usuario es necesaria cuando una directiva de acceso condicional de solo informe requiere un control de concesión interactivo, como la autenticación multifactor. Dado que las directivas de solo informe no aplican los controles de concesión interactivos, no se puede determinar si el resultado es correcto o erróneo.  

**No aplicado**: número de usuarios o inicios de sesión durante el período de tiempo en que ninguna de las directivas seleccionadas estaba aplicada.

### <a name="understanding-the-impact"></a>Explicación del impacto 

![Desglose del libro por condición y estado](./media/howto-conditional-access-insights-reporting/workbook-breakdown-condition-and-status.png)

Vea el desglose de usuarios e inicios de sesión para cada una de las condiciones. Puede filtrar los inicios de sesión de un resultado determinado (por ejemplo, Correcto o Error). Para ello, seleccione los iconos de resumen en la parte superior del libro. Puede ver el desglose de inicios de sesión para cada una de las condiciones de acceso condicional: estado del dispositivo, plataforma del dispositivo, aplicación cliente, ubicación, aplicación y riesgo de inicio de sesión.  

## <a name="sign-in-details"></a>Detalles de inicio de sesión 

![Detalles de inicio de sesión del libro](./media/howto-conditional-access-insights-reporting/workbook-sign-in-details.png)

También puede investigar los inicios de sesión de un usuario específico si busca inicios de sesión en la parte inferior del panel. La consulta de la izquierda muestra los usuarios más frecuentes. Al seleccionar un usuario se filtrará la consulta a la derecha.  

> [!NOTE]
> Al descargar los registros de inicios de sesión, seleccione formato JSON para incluir datos de resultados de solo informe de acceso condicional.

## <a name="configure-a-conditional-access-policy-in-report-only-mode"></a>Configuración de una directiva de acceso condicional en modo de solo informe

Para configurar una directiva de acceso condicional en modo de solo informe:

1. Inicie sesión en **Azure Portal** como administrador de acceso condicional, administrador de seguridad o administrador global.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione una directiva existente o cree una nueva.
1. En **Habilitar directiva**, establezca la opción en modo de **solo informe**.
1. Seleccione **Guardar**.

> [!TIP]
> La edición del estado **Habilitar directiva** de una directiva existente de **Activado** a **Solo informe** deshabilita la aplicación de la directiva existente. 

## <a name="troubleshooting"></a>Solución de problemas

### <a name="why-are-queries-failing-due-to-a-permissions-error"></a>¿Por qué se producen errores en las consultas debido a un error de permisos?

Para tener acceso al libro, necesita los permisos de Azure AD adecuados, así como permisos del área de trabajo de Log Analytics. Para probar si tiene los permisos adecuados del área de trabajo, ejecute una consulta de Log Analytics de ejemplo:

1. Inicie sesión en **Azure Portal**.
1. Vaya a **Azure Active Directory** > **Registros**.
1. Escriba `SigninLogs` en el cuadro consulta y seleccione **Ejecutar**.
1. Si la consulta no devuelve ningún resultado, es posible que el área de trabajo no se haya configurado correctamente. 

![Solución de problemas de consultas con errores](./media/howto-conditional-access-insights-reporting/query-troubleshoot-sign-in-logs.png)

Para obtener más información sobre cómo transmitir los registros de inicio de sesión de Azure AD a un área de trabajo de Log Analytics, consulte el artículo [Integración de registros de Azure AD con registros de Azure Monitor](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md).

### <a name="why-are-the-queries-in-the-workbook-failing"></a>¿Por qué se producen errores en las consultas del libro?

Los clientes han observado que, en ocasiones, se producen errores en las consultas si varias áreas de trabajo o áreas de trabajo erróneas se asocian al libro. Para corregir este problema, haga clic en **Editar** en la parte superior del libro y, a continuación, en el icono con forma de engranaje de Configuración. Seleccione y, a continuación, quite las áreas de trabajo no asociadas al libro. Solo debe haber un área de trabajo asociada a cada libro.

### <a name="why-is-the-conditional-access-policies-parameter-is-empty"></a>¿Por qué está vacío el parámetro de las directivas de acceso condicional?

La lista de directivas se genera examinando las directivas evaluadas para el evento de inicio de sesión más reciente. Si no hay inicios de sesión recientes en el inquilino, es posible que tenga que esperar unos minutos para que el libro cargue la lista de directivas de acceso condicional. Esto puede ocurrir inmediatamente después de configurar Log Analytics o puede tardar más tiempo si un inquilino no tiene actividad reciente de inicio de sesión.

### <a name="why-is-the-workbook-taking-a-long-time-to-load"></a>¿Por qué el libro tarda tanto tiempo en cargarse?  

Según el intervalo de tiempo seleccionado y el tamaño del inquilino, es posible que el libro esté evaluando un número extraordinariamente elevado de eventos de inicio de sesión. En el caso de inquilinos grandes, el volumen de inicios de sesión puede superar la capacidad de consulta de Log Analytics. Intente acortar el intervalo de tiempo a 4 horas para ver si se carga el libro.  

### <a name="after-loading-for-a-few-minutes-why-is-the-workbook-returning-zero-results"></a>Después de la carga durante unos minutos, ¿por qué el libro devuelve cero resultados? 

Cuando el volumen de inicios de sesión supera la capacidad de consulta de Log Analytics, el libro no devuelve ningún resultado. Intente acortar el intervalo de tiempo a 4 horas para ver si se carga el libro.  

### <a name="can-i-save-my-parameter-selections"></a>¿Se pueden guardar las selecciones de parámetros?  

Puede guardar las selecciones de parámetros en la parte superior del libro; para ello, vaya a **Azure Active Directory** > **Libros** > **Conditional Access Insights and reporting**. Aquí encontrará la plantilla de libro, donde puede editar el libro y guardar una copia en el área de trabajo, incluidas las selecciones de parámetros, en **Mis informes** o **Informes compartidos**. 

### <a name="can-i-edit-and-customize-the-workbook-with-additional-queries"></a>¿Puedo editar y personalizar el libro con consultas adicionales? 

Para editar y personalizar el libro, puede dirigirse a **Azure Active Directory** > **Libros** > **Conditional Access Insights and reporting**. Aquí encontrará la plantilla de libro, donde puede editar el libro y guardar una copia en el área de trabajo, incluidas las selecciones de parámetros, en **Mis informes** o **Informes compartidos**. Para empezar a editar las consultas, haga clic en **Editar** en la parte superior del libro.  
 
## <a name="next-steps"></a>Pasos siguientes

- [Modo de solo informe de acceso condicional](concept-conditional-access-report-only.md)

- Para obtener más información acerca de los libros de Azure AD, consulte [Cómo usar los libros de Azure Monitor en informes de Azure Active Directory](../reports-monitoring/howto-use-azure-monitor-workbooks.md).

- [Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)
