---
title: Informes de actividad de inicio de sesión en el portal de Azure Active Directory | Microsoft Docs
description: Introducción a los informes de actividad de inicio de sesión en el portal de Azure Active Directory
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 03/24/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: e84fb04d967311fede3be7f20b45f01fb442e69e
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89228773"
---
# <a name="sign-in-activity-reports-in-the-azure-active-directory-portal"></a>Informes de actividad de inicio de sesión en el portal de Azure Active Directory

La arquitectura de los informes de Azure Active Directory (Azure AD) consta de los siguientes componentes:

- **Actividad** 
    - **Inicios de sesión**: información sobre el uso de aplicaciones administradas y actividades de inicio de sesión de usuario.
    - **Registros de auditoría** - [: los ](concept-audit-logs.md)registros de auditoría proporcionan información de la actividad del sistema sobre la administración de usuarios y grupos, aplicaciones administradas y actividades de directorio.
- **Seguridad** 
    - **Inicios de sesión de riesgo**: un [inicio de sesión de riesgo](../identity-protection/overview-identity-protection.md) es un indicador de un intento de inicio de sesión de alguien que no es el propietario legítimo de una cuenta de usuario.
    - **Usuarios marcados en riesgo**: un [usuario en riesgo](../identity-protection/overview-identity-protection.md) es un indicador de una cuenta de usuario que puede haber estado en peligro.

En este artículo se ofrece una visión general del reporte de inicios de sesión.

## <a name="prerequisites"></a>Prerrequisitos

### <a name="who-can-access-the-data"></a>¿Quién puede acceder a los datos?

* Usuarios de los roles Administrador de seguridad, Lector de seguridad, Lector global y Lector de informes
* Administradores globales
* Cualquier usuario (no administradores) puede acceder a sus propios inicios de sesión 

### <a name="what-azure-ad-license-do-you-need-to-access-sign-in-activity"></a>¿Qué licencia de Azure AD se necesita para acceder a la actividad de inicio de sesión?

El informe de actividad de inicio de sesión está disponible en [todas las ediciones de Azure AD](reference-reports-data-retention.md#how-long-does-azure-ad-store-the-data) y también se puede acceder a ella mediante Microsoft Graph API.

## <a name="sign-ins-report"></a>Informe de inicios de sesión

El informe de inicios de sesión de usuario proporciona respuestas a las preguntas siguientes:

* ¿Cuál es el patrón de inicio de sesión de un usuario?
* ¿Cuántos usuarios han iniciado sesión en una semana?
* ¿Cuál es el estado de estos inicios de sesión?

En el menú de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory** o busque y seleccione **Azure Active Directory** desde cualquier página.

![Seleccionar Azure Active Directory](./media/concept-sign-ins/select-azure-active-directory.png "Azure Active Directory")

En **Supervisión**, seleccione **Inicios de sesión** para abrir el [informe de inicios de sesión](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/SignIns).

![Actividad de inicio de sesión](./media/concept-sign-ins/monitoring-sign-ins-in-azure-active-directory.png "Actividad de inicio de sesión")

Algunos registros de inicio de sesión pueden tardar hasta dos horas antes de aparecer en el portal.

> [!IMPORTANT]
> El informe de inicios de sesión solo muestra los inicios de sesión **interactivos**, es decir, los inicios de sesión donde un usuario inicia sesión manualmente con su nombre de usuario y contraseña. Los inicios de sesión no interactivos, como la autenticación de servicio a servicio, no se muestran en el informe de inicios de sesión. 

Un registro de inicios de sesión tiene una vista de lista predeterminada que muestra:

- La fecha de inicio de sesión
- El usuario relacionado
- La aplicación en la que el usuario ha iniciado sesión
- El estado de inicio de sesión
- El estado de la detección de riesgos
- El estado del requisito de la autenticación multifactor (MFA)

![Actividad de inicio de sesión](./media/concept-sign-ins/sign-in-activity.png "Actividad de inicio de sesión")

Puede personalizar la vista de lista, haga clic en **Columnas** en la barra de herramientas.

![Actividad de inicio de sesión](./media/concept-sign-ins/19.png "Actividad de inicio de sesión")

En el cuadro de diálogo **Columnas** se proporciona acceso a los atributos seleccionables. En un informe de inicio de sesión, no puede tener campos que tengan más de un valor para una solicitud de inicio de sesión determinada como columna. Esto es así, por ejemplo, para los detalles de autenticación, los datos de acceso condicional y la ubicación de red.   

![Actividad de inicio de sesión](./media/concept-sign-ins/columns.png "Actividad de inicio de sesión")

Seleccione un elemento de la vista de lista para obtener información más detallada.

![Actividad de inicio de sesión](./media/concept-sign-ins/basic-sign-in.png "Actividad de inicio de sesión")

> [!NOTE]
> Los clientes ahora pueden solucionar problemas de directivas de acceso condicional con todos los informes de inicios de sesión. Al hacer clic en la pestaña **Acceso condicional** de un registro de inicio de sesión, los clientes pueden revisar el estado de acceso condicional y profundizar en los detalles de las directivas que se aplican al inicio de sesión y del resultado de cada directiva.
> Para más información, vea las [preguntas más frecuentes sobre la información de acceso condicional en todos los inicios de sesión](reports-faq.md#conditional-access).



## <a name="filter-sign-in-activities"></a>Filtrado de las actividades de inicio de sesión

En primer lugar, limite los datos informados a un nivel que le resulte más adecuado. En segundo lugar, filtre los datos de inicios de sesión mediante el campo de fecha como filtro predeterminado. Azure AD proporciona una amplia variedad de filtros adicionales que puede establecer:

![Actividad de inicio de sesión](./media/concept-sign-ins/04.png "Actividad de inicio de sesión")

**Identificador de solicitud**: el identificador de la solicitud que le interesa.

**Usuario**: el nombre o el nombre principal de usuario (UPN) del usuario que le interesa.

**Aplicación**: el nombre de la aplicación de destino.
 
**Estado**: el estado de inicio de sesión que le interesa:

- Correcto

- Error

- Interrumpido


**Dirección IP**: la dirección IP del dispositivo que se utiliza para conectarse al inquilino.

**Ubicación**: la ubicación desde la que se inició la conexión:

- City

- Estado/provincia

- País/región


**Recurso**: el nombre del servicio que se usa para el inicio de sesión.


**Identificador de recurso**: el identificador del servicio que se utiliza para el inicio de sesión.


**Aplicación cliente**: el tipo de aplicación cliente que se usa para conectarse al inquilino:

![Filtro de aplicación cliente](./media/concept-sign-ins/client-app-filter.png)


|Nombre|Autenticación moderna|Descripción|
|---|:-:|---|
|SMTP autenticado| |Utilizado por clientes POP e IMAP para enviar mensajes de correo electrónico.|
|Detección automática| |Usada por clientes Outlook y EAS para buscar y conectarse a buzones en Exchange Online.|
|Exchange ActiveSync| |Este filtro muestra todos los intentos de inicio de sesión en los que se ha intentado el protocolo EAS.|
|Browser|![Comprobar](./media/concept-sign-ins/check.png)|Muestra todos los intentos de inicio de sesión de los usuarios mediante exploradores web.|
|Exchange ActiveSync| | Muestra todos los intentos de inicio de sesión de los usuarios con aplicaciones cliente que usan Exchange ActiceSync para conectarse a Exchange Online.|
|PowerShell de Exchange Online| |Se usa para conectarse a Exchange Online con PowerShell remoto. Si bloquea la autenticación básica para PowerShell de Exchange Online, debe usar el módulo de PowerShell de Exchange Online para conectarse. Para obtener instrucciones, consulte [Conexión a Exchange Online PowerShell con autenticación multifactor](/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/mfa-connect-to-exchange-online-powershell).|
|Servicios web de Exchange| |Una interfaz de programación que se usa en Outlook, Outlook para Mac y aplicaciones de terceros.|
|IMAP4| |Un cliente de correo heredado que utiliza IMAP para recuperar el correo electrónico.|
|MAPI sobre HTTP| |Utilizado por Outlook 2010 y versiones posteriores.|
|Aplicaciones móviles y aplicaciones de escritorio|![Comprobar](./media/concept-sign-ins/check.png)|Muestra todos los intentos de inicio de sesión de los usuarios que usan aplicaciones móviles y clientes de escritorio.|
|Libreta de direcciones sin conexión| |Una copia de las colecciones de listas de direcciones que Outlook descarga y usa.|
|Outlook en cualquier lugar (RPC sobre HTTP)| |Utilizado por Outlook 2016 y versiones anteriores.|
|Servicio Outlook| |Usado por la aplicación de correo electrónico y calendario de Windows 10.|
|POP3| |Un cliente de correo heredado que utiliza POP3 para recuperar el correo electrónico.|
|Servicios web de creación de informes| |Se usan para recuperar datos de informes en Exchange Online.|
|Otros clientes| |Muestra todos los intentos de inicio de sesión de los usuarios en los que la aplicación cliente no está incluida o es desconocida.|



**Sistema operativo**: el sistema operativo que se ejecuta en el dispositivo utilizado para iniciar sesión en el inquilino. 


**Explorador de dispositivos**: si la conexión se inició desde un explorador, este campo le permite filtrar por nombre de explorador.


**Id. de correlación**: el identificador de correlación de la actividad.




**Acceso condicional**: el estado de las reglas de acceso condicional aplicadas

- **No aplicado**: No se aplica ninguna directiva al usuario y a la aplicación durante el inicio de sesión.

- **Correcto**: Una o varias directivas de acceso condicional aplicadas al usuario y a la aplicación (pero no necesariamente a las demás condiciones) durante el inicio de sesión. 

- **Error**: El inicio de sesión cumplió la condición de usuario y aplicación de al menos una directiva de acceso condicional, y los controles de concesión no se han cumplido o se han establecido para bloquear el acceso.









## <a name="download-sign-in-activities"></a>Descarga de actividades de inicio de sesión

Haga clic en **Descargar** para crear un archivo CSV o JSON de los 250 000 registros más recientes. Empiece por [descargar los datos de inicios de sesión](quickstart-download-sign-in-report.md) si quiere trabajar con ellos fuera de Azure Portal.  

![Descargar](./media/concept-sign-ins/71.png "Descargar")

> [!IMPORTANT]
> El número de registros que se puede descargar también está restringido por las [directivas de retención de informes de Azure Active Directory](reference-reports-data-retention.md).  


## <a name="sign-ins-data-shortcuts"></a>Accesos directos a los datos de inicios de sesión

Tanto Azure AD como Azure Portal proporcionan puntos de entrada adicionales para los datos de inicios de sesión:

- Información general sobre la protección de la seguridad de la identidad
- Usuarios
- Grupos
- Aplicaciones empresariales

### <a name="users-sign-ins-data-in-identity-security-protection"></a>Datos de inicio de sesión de usuarios en la protección de la identidad de la seguridad

El gráfico de inicio de sesión de usuario en la página de información general de la **protección de la seguridad de identidad** muestra agregaciones semanales de inicios de sesión. El valor predeterminado para el período es 30 días.

![Actividad de inicio de sesión](./media/concept-sign-ins/06.png "Actividad de inicio de sesión")

Al hacer clic en un día del gráfico de inicio de sesión, obtiene una vista general de las actividades de inicio de sesión de ese día.

Cada fila de la lista de actividades de inicio de sesión muestra:

* ¿Quién ha iniciado sesión?
* ¿Qué aplicación era el destino del inicio de sesión?
* ¿Cuál es el estado del inicio de sesión?
* ¿Cuál es el estado de MFA del inicio de sesión?

Si hace clic en un elemento, obtendrá más detalles sobre la operación de inicio de sesión:

- Id. de usuario
- Usuario
- Nombre de usuario
- Identificador de aplicación
- Application
- Remoto
- Location
- Dirección IP
- Date
- Se requiere MFA
- Estado de inicio de sesión

> [!NOTE]
> Las direcciones IP se emiten de forma que no haya ninguna conexión definitiva entre una dirección IP y donde se encuentre físicamente el equipo con esa dirección. La asignación de direcciones IP se complica por el hecho de que los proveedores de dispositivos móviles y las VPN emiten direcciones IP desde grupos centrales que, a menudo, están muy lejos de donde el dispositivo cliente se usa realmente. Actualmente, la conversión de la dirección IP en una ubicación física en los informes de Azure AD es un esfuerzo notable basado en seguimientos, datos del Registro, búsquedas inversas y otra información.

En la página **Usuarios**, puede obtener una vista general completa de todos los inicios de sesión del usuario haciendo clic en **Inicios de sesión** en la sección **Actividad**.

![Actividad de inicio de sesión](./media/concept-sign-ins/08.png "Actividad de inicio de sesión")

## <a name="usage-of-managed-applications"></a>Uso de las aplicaciones administradas

Con una vista centrada en la aplicación de los datos de inicio de sesión, puede responder a preguntas tales como:

* ¿Quién está usando mis aplicaciones?
* ¿Cuáles son las tres aplicaciones principales en su organización?
* ¿Cómo es el rendimiento de la aplicación más reciente?

El punto de entrada a estos datos son las tres aplicaciones principales de su organización. Los datos se incluyen en el informe de los últimos 30 días en la sección **Información general** en **Aplicaciones empresariales**.

![Actividad de inicio de sesión](./media/concept-sign-ins/10.png "Actividad de inicio de sesión")

Agregaciones semanales de los gráficos de uso de la aplicación de inicios de sesión para las tres aplicaciones principales en un período determinado. El valor predeterminado para el período es 30 días.

![Actividad de inicio de sesión](./media/concept-sign-ins/graph-chart.png "Actividad de inicio de sesión")

Si lo desea, puede establecer el foco en una aplicación específica.

![Reporting](./media/concept-sign-ins/single-app-usage-graph.png "Notificación")

Al hacer clic en un día del gráfico de uso de la aplicación, obtendrá una lista detallada de las actividades de inicio de sesión.

La opción **Inicios de sesión** ofrece una descripción completa de todos los eventos de inicio de sesión para sus aplicaciones.

## <a name="office-365-activity-logs"></a>Registros de actividad de Office 365

Puede ver los registros de actividad de Office 365 desde el [Centro de administración de Microsoft 365](/office365/admin/admin-overview/about-the-admin-center). Tenga en cuenta que la actividad de Office 365 y los registros de actividad de Azure AD comparten una cantidad significativa de los recursos del directorio. Solo el centro de administración de Microsoft 365 proporciona una vista completa de los registros de actividad de Office 365. 

También puede tener acceso a los registros de actividad de Office 365 mediante programación con las [API de administración de Office 365](/office/office-365-management-api/office-365-management-apis-overview).

## <a name="next-steps"></a>Pasos siguientes

* [Códigos de error de los informes de actividad de inicio de sesión](reference-sign-ins-error-codes.md)
* [Directivas de retención de datos de Azure AD](reference-reports-data-retention.md)
* [Latencias de informes de Azure AD](reference-reports-latencies.md)