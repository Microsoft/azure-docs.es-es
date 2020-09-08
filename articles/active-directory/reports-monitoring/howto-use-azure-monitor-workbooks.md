---
title: Libros de Azure Monitor para informes | Microsoft Docs
description: Aprenda a usar los libros de Azure Monitor en informes de Azure Active Directory.
services: active-directory
author: MarkusVi
manager: daveba
ms.assetid: 4066725c-c430-42b8-a75b-fe2360699b82
ms.service: active-directory
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: ''
ms.workload: identity
ms.subservice: report-monitor
ms.date: 10/30/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.openlocfilehash: 510991c1afba9a5ffbfd77e855ee25d8b641bdd0
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226988"
---
# <a name="how-to-use-azure-monitor-workbooks-for-azure-active-directory-reports"></a>Uso de los libros de Azure Monitor en informes de Azure Active Directory

> [!IMPORTANT]
> Para optimizar las consultas subyacentes de este libro, haga clic en "Editar", haga clic en el icono Configuración y seleccione el área de trabajo en la que desea ejecutar estas consultas. De forma predeterminada, los libros seleccionarán todas las áreas de trabajo en las que se enrutan los registros de Azure AD. 

Quiere:

- Comprender el efecto de las [directivas de acceso condicional](../conditional-access/overview.md) en la experiencia de inicio de sesión de los usuarios.

- Solucionar problemas de inicio de sesión para obtener una mejor visión del estado de inicio de sesión de la organización y resolver problemas rápidamente.

- Saber quién está utilizando las autenticaciones heredadas para iniciar sesión en su entorno. (Al [bloquear la autenticación heredada](../conditional-access/block-legacy-authentication.md), puede mejorar la protección del inquilino).

- ¿Necesita comprender el impacto de las directivas de acceso condicional en el inquilino?

- ¿Desea tener la posibilidad de revisar las consultas del registro de inicio de sesión? El libro informa del número de usuarios a los que se ha concedido o denegado el acceso, así como de cuántos usuarios han omitido las directivas de acceso condicional al acceder a los recursos.

- ¿Está interesado en desarrollar una comprensión más profunda? El libro proporciona detalles por condición, para que el impacto de una directiva se pueda contextualizar por condición, incluidas la plataforma del dispositivo, el estado del dispositivo, la aplicación cliente, el riesgo de inicio de sesión, la ubicación y la aplicación.

- Obtenga información más detallada sobre las consultas del registro de inicio de sesión, el libro informa del número de usuarios a los que se ha concedido o denegado el acceso, así como de cuántos usuarios han omitido las directivas de acceso condicional al acceder a los recursos.

- Para ayudarle a resolver estas cuestiones, Azure Active Directory proporciona libros para la supervisión. Los [libros de Azure Monitor](../../azure-monitor/platform/workbooks-overview.md) combinan texto, consultas de análisis, métricas de Azure y parámetros en informes interactivos avanzados.



Este artículo:

- Supone que está familiarizado con cómo [crear informes interactivos con libros de Monitor](../../azure-monitor/platform/workbooks-overview.md).

- Explica cómo utilizar los libros de Monitor para comprender el efecto de las directivas de acceso condicional, para solucionar los problemas de inicio de sesión e identificar las autenticaciones heredadas.
 


## <a name="prerequisites"></a>Prerrequisitos

Para utilizar los libros de Monitor, necesita:

- Un inquilino de Azure Active Directory con una licencia Premium (P1 o P2). Aprenda cómo [obtener una licencia prémium](../fundamentals/active-directory-get-started-premium.md).

- Un [área de trabajo de Log Analytics.](../../azure-monitor/learn/quick-create-workspace.md)

- [Acceso](../../azure-monitor/platform/manage-access.md#manage-access-using-workspace-permissions) al área de trabajo de Log Analytics
- Los siguientes roles en Azure Active Directory (si obtiene acceso a Log Analytics a través del portal de Azure Active Directory)
    - Administrador de seguridad
    - Lector de seguridad
    - Lector de informes
    - Administrador global

## <a name="roles"></a>Roles
Debe estar en uno de los roles siguientes y tener [acceso al área de trabajo subyacente de Log Analytics](../../azure-monitor/platform/manage-access.md#manage-access-using-azure-permissions) para administrar los libros:
-   Administrador global
-   Administrador de seguridad
-   Lector de seguridad
-   Lector de informes
-   Administrador de aplicaciones

## <a name="workbook-access"></a>Acceso a los libros 

Para acceder a los libros:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Vaya a **Azure Active Directory** > **Supervisión** > **Libros**. 

1. Seleccione un informe o una plantilla o, en la barra de herramientas, seleccione **Abrir**. 

![Búsqueda de los libros de Azure Monitor en Azure AD](./media/howto-use-azure-monitor-workbooks/azure-monitor-workbooks-in-azure-ad.png)

## <a name="sign-in-analysis"></a>Análisis de inicio de sesión

Para tener acceso al libro de análisis de inicio de sesión, en la sección **Uso**, seleccione **Inicios de sesión**. 

Este libro muestra las siguientes tendencias de inicio de sesión:

- Todos los inicios de sesión

- Correcto

- Acción de usuario pendiente

- Error

Puede filtrar cada tendencia por las siguientes categorías:

- Intervalo de horas

- Aplicaciones

- Usuarios

![Análisis de inicio de sesión](./media/howto-use-azure-monitor-workbooks/43.png)


Para cada tendencia, puede obtener un desglose por las siguientes categorías:

- Location

    ![Inicios de sesión por ubicación](./media/howto-use-azure-monitor-workbooks/45.png)

- Dispositivo

    ![Inicios de sesión por dispositivo](./media/howto-use-azure-monitor-workbooks/46.png)


## <a name="sign-ins-using-legacy-authentication"></a>Inicios de sesión que utilizan una autenticación heredada 


Para acceder al libro para los inicios de sesión que utilizan la [autenticación heredada](../conditional-access/block-legacy-authentication.md), en la sección **Usage** (Uso), seleccione **Sign-ins using Legacy Authentication** (Inicios de sesión que utilizan una autenticación heredada). 

Este libro muestra las siguientes tendencias de inicio de sesión:

- Todos los inicios de sesión

- Correcto


Puede filtrar cada tendencia por las siguientes categorías:

- Intervalo de horas

- Aplicaciones

- Usuarios

- Protocolos

![Inicios de sesión por autenticación heredada](./media/howto-use-azure-monitor-workbooks/47.png)


Para cada tendencia, va a obtener un desglose por aplicación y protocolo.

![Inicios de sesión de autenticación heredada por aplicación y protocolo](./media/howto-use-azure-monitor-workbooks/48.png)



## <a name="sign-ins-by-conditional-access"></a>Inicios de sesión por acceso condicional 


Para tener acceso al libro para los inicios de sesión por [directivas de acceso condicional](../conditional-access/overview.md), en la sección **Conditional Access** (Acceso condicional), seleccione **Sign-ins by Conditional Access** (Inicios de sesión por acceso condicional). 

Este libro muestra las tendencias para los inicios de sesión deshabilitados. Puede filtrar cada tendencia por las siguientes categorías:

- Intervalo de horas

- Aplicaciones

- Usuarios

![Inicios de sesión con acceso condicional](./media/howto-use-azure-monitor-workbooks/49.png)


Para los inicios de sesión deshabilitados, obtendrá un desglose por el estado de acceso condicional.

![Estado de acceso condicional](./media/howto-use-azure-monitor-workbooks/conditional-access-status.png)


## <a name="conditional-access-insights"></a>Conditional Access Insights

### <a name="overview"></a>Información general

Los libros contienen consultas del registro de inicio de sesión que pueden ayudar a los administradores de TI a supervisar el impacto de las directivas de acceso condicional en el inquilino. Tiene la capacidad de informar sobre el número de usuarios a los que se les habría concedido o denegado el acceso. El libro contiene información sobre cuántos usuarios habrían omitido las directivas de acceso condicional en función de los atributos de los usuarios en el momento del inicio de sesión. Contiene detalles por condición, para que el impacto de una directiva se pueda contextualizar por condición, incluidas la plataforma del dispositivo, el estado del dispositivo, la aplicación cliente, el riesgo de inicio de sesión, la ubicación y la aplicación.

### <a name="instructions"></a>Instructions 
Para acceder al libro con la información de acceso condicional, seleccione el libro **Información de acceso condicional** en la sección Acceso condicional. Este libro muestra el impacto esperado de cada directiva de acceso condicional en el inquilino. Seleccione una o varias directivas de acceso condicional en la lista desplegable y restrinja el ámbito del libro; para ello, aplique los siguientes filtros: 

- **Intervalo de tiempo**

- **User**

- **Aplicaciones**

- **Vista de datos**

![Estado de acceso condicional](./media/howto-use-azure-monitor-workbooks/access-insights.png)


El Resumen de impacto muestra el número de usuarios o inicios de sesión para los que las directivas seleccionadas tenían un resultado determinado. Total es el número de usuarios o inicios de sesión para los que se evaluaron las directivas seleccionadas en el intervalo de tiempo seleccionado. Haga clic en un icono para filtrar los datos del libro por ese tipo de resultado. 

![Estado de acceso condicional](./media/howto-use-azure-monitor-workbooks/impact-summary.png)

Este libro también muestra el impacto de las directivas seleccionadas divididas por cada una de estas seis condiciones: 
- **Estado del dispositivo**
- **Plataforma del dispositivo**
- **Aplicaciones cliente**
- **Riesgo de inicio de sesión**
- **Ubicación**
- **Aplicaciones**

![Estado de acceso condicional](./media/howto-use-azure-monitor-workbooks/device-platform.png)

También puede investigar inicios de sesión individuales, filtrados por los parámetros seleccionados en el libro. Busque usuarios individuales, ordenados por frecuencia de inicio de sesión y consulte los eventos de inicio de sesión correspondientes. 

![Estado de acceso condicional](./media/howto-use-azure-monitor-workbooks/filtered.png)





## <a name="sign-ins-by-grant-controls"></a>Inicios de sesión por controles de concesión

Para tener acceso al libro para los inicios de sesión por [controles de concesión](../conditional-access/controls.md), en la sección **Conditional Access** (Acceso condicional), seleccione **Sign-ins by Grant Controls** (Inicios de sesión por controles de concesión). 

Este libro muestra las siguientes tendencias de inicios de sesión deshabilitados:

- Requerir MFA
 
- Requerir condiciones de uso

- Requerir la declaración de privacidad

- Otros


Puede filtrar cada tendencia por las siguientes categorías:

- Intervalo de horas

- Aplicaciones

- Usuarios

![Inicios de sesión por controles de concesión](./media/howto-use-azure-monitor-workbooks/50.png)


Para cada tendencia, va a obtener un desglose por aplicación y protocolo.

![Desglose de los inicios de sesión recientes](./media/howto-use-azure-monitor-workbooks/51.png)




## <a name="sign-ins-failure-analysis"></a>Análisis de errores de inicio de sesión

Use el libro **Análisis de errores de inicios de sesión** para solucionar errores en:

- Inicios de sesión
- Directivas de acceso condicional
- Autenticación heredada 


Para acceder a los inicios de sesión por datos de acceso condicional, en la sección **Solución de problemas**, seleccione **Sign-ins using Legacy Authentication** (Inicios de sesión que utilizan una autenticación heredada). 

Este libro muestra las siguientes tendencias de inicio de sesión:

- Todos los inicios de sesión

- Correcto

- Acción pendiente

- Error


Puede filtrar cada tendencia por las siguientes categorías:

- Intervalo de horas

- Aplicaciones

- Usuarios

![Solución de problemas de inicios de sesión](./media/howto-use-azure-monitor-workbooks/52.png)


Para ayudarle a solucionar problemas de inicios de sesión, Azure Monitor proporciona un desglose por las siguientes categorías:

- Principales errores

    ![Resumen de los errores principales](./media/howto-use-azure-monitor-workbooks/53.png)

- Inicios de sesión a la espera de una acción del usuario

    ![Resumen de los inicios de sesión a la espera de una acción del usuario](./media/howto-use-azure-monitor-workbooks/54.png)






## <a name="next-steps"></a>Pasos siguientes

[Creación de informes interactivos con los libros de Azure Monitor](../../azure-monitor/platform/workbooks-overview.md).