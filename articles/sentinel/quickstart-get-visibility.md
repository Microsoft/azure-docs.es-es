---
title: 'Inicio rápido: Introducción a Azure Sentinel'
description: Use este inicio rápido para aprender a ver y supervisar rápidamente lo que sucede en su entorno mediante Azure Sentinel. 
services: sentinel
author: yelevin
manager: rkarlin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: quickstart
ms.custom: mvc, fasttrack-edit
ms.date: 09/16/2020
ms.author: yelevin
ms.openlocfilehash: fe2a632901b32993d46e723ab73fce045bfa9f84
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90889133"
---
# <a name="quickstart-get-started-with-azure-sentinel"></a>Inicio rápido: Introducción a Azure Sentinel




En este inicio rápido, aprenderá a ver y supervisar rápidamente lo que sucede en su entorno mediante Azure Sentinel. Después de conectar los orígenes de datos a Azure Sentinel, obtendrá una visualización y un análisis instantáneos de los datos, así sabrá lo que sucede en todos los orígenes de datos conectados. Azure Sentinel ofrece libros que le proporcionan toda la potencia de las herramientas que ya están disponibles en Azure, así como tablas y gráficos que están integrados para ofrecerle análisis de sus registros y consultas. Puede usar los libros integrados o crear un libro fácilmente desde cero o a partir de un libro existente. 

## <a name="get-visualization"></a>Obtener visualización

Para visualizar y obtener análisis de lo que sucede en su entorno, primero eche un vistazo al panel de información general para hacerse una idea de la posición de seguridad de su organización. Puede hacer clic en cada elemento de estos iconos para explorar en profundidad los datos sin procesar desde los que se crean. Para ayudar a reducir el ruido y minimizar el número de alertas que tiene que revisar e investigar, Azure Sentinel emplea una técnica de fusión para correlacionar alertas con incidentes. Los **incidentes** son grupos de alertas relacionadas que, juntas, crean un incidente procesable que se puede investigar y resolver.

- En Azure Portal, seleccione Azure Sentinel y, luego, seleccione el área de trabajo que quiere supervisar.

  ![Introducción a Azure Sentinel](./media/qs-get-visibility/overview.png)

- La barra de herramientas de la parte superior le indica cuántos eventos obtuvo durante el período de tiempo seleccionado y lo compara con las 24 horas anteriores. A partir de estos eventos, la barra de herramientas le indica las alertas que se desencadenaron (el número pequeño representa los cambios en las últimas 24 horas) y, luego, le indica cuántos de esos eventos están abiertos, en curso y cerrados. Compruebe que no haya un ascenso o descenso destacables en el número de eventos. Si se produce un descenso, es posible que una de las conexiones haya dejado de informar a Azure Sentinel. Si hay un ascenso, puede que haya ocurrido algo sospechoso. Vea si tiene nuevas alertas.

   ![Contadores de Azure Sentinel](./media/qs-get-visibility/funnel.png)

El cuerpo principal de la página de información general ofrece conclusiones de un vistazo sobre el estado de seguridad del área de trabajo:

- **Eventos y alertas con el tiempo**: muestra el número de eventos y cuántas alertas se crearon a partir de ellos. Si observa un pico inusual, examine las alertas al respecto; si hay algo raro donde hay un pico en los eventos, pero no hay alertas, podría ser motivo de preocupación.

- **Posibles eventos malintencionados**: cuando se detecta tráfico desde orígenes que se sabe que son malintencionados, Azure Sentinel le avisa en el mapa. Si la alerta es naranja, se trata del tráfico entrante: alguien está intentando acceder a su organización desde una dirección IP malintencionada conocida. Si observa actividad saliente (rojo), significa que los datos de la red se están transmitiendo fuera de la organización a una dirección IP malintencionada conocida.

   ![Mapa de tráfico malintencionado](./media/qs-get-visibility/map.png)

- **Incidentes recientes**: para ver los incidentes recientes, su gravedad y el número de alertas asociadas con el incidente. Si observa como un pico repentino en un tipo de alerta específico, podría significar que hay un ataque activado actualmente en ejecución. Por ejemplo, si tiene un pico repentino de 20 eventos Pass-the-hash procedentes de Microsoft Defender for Identity (anteriormente, Azure ATP), es posible que alguien esté intentando atacarle.

- **Anomalías del origen de datos**: los analistas de datos de Microsoft crean modelos que examinan constantemente los datos de los orígenes de datos en busca de anomalías. Si no hay ninguna anomalía, no se muestra nada. Si se detectan anomalías, debe analizarlas a fondo para ver lo que ha sucedido. Por ejemplo, haga clic en el pico de actividad de Azure. Puede hacer clic en **Chart** (Gráfico) para ver el pico que ha ocurrido y, luego, filtrar por las actividades que se han producido durante ese período de tiempo para ver la causa.

   ![Orígenes de datos anómalos](./media/qs-get-visibility/anomolies.png)

## <a name="use-built-in-workbooks"></a>Uso de libros integrados<a name="dashboards"></a>

Los libros integrados proporcionan datos integrados procedentes de orígenes de datos conectados y permiten analizar en profundidad los eventos generados en dichos servicios. Los libros integrados incluyen Azure AD, eventos de actividad de Azure y locales, que pueden ser datos de eventos de Windows de servidores, de alertas de primera entidad o de terceros, incluidos los registros de tráfico del firewall, Office 365 y protocolos poco seguros basados en eventos de Windows. Los libros se basan en Azure Monitor Workbooks para proporcionar una mayor flexibilidad y capacidad de personalización al diseñar su propio libro. Para más información, consulte [Libros](../azure-monitor/platform/workbooks-overview.md).

1. En **Configuración**, seleccione **Libros**. En **Instalados**, puede ver todos los libros instalados. En **Todos**, puede ver toda la galería entera de libros integrados disponibles para su instalación. 
2. Busque un libro específico ver la lista completa y una descripción de lo que ofrece cada uno. 
3. Suponiendo que usa Azure AD, para comenzar a trabajar con Azure Sentinel, se recomienda que instale al menos los libros siguientes:
   - **Azure AD**: use uno de estos paneles o ambos:
       - **Inicios de sesión de Azure AD**: analiza los inicios de sesión con el tiempo para ver si hay anomalías. Este libro proporciona los errores de inicio de sesión de aplicaciones, dispositivos y ubicaciones de forma que pueda advertir de un vistazo si sucede algo inusual. Preste atención cuando se producen varios errores de inicio de sesión. 
       - **Registros de auditoría de Azure AD**: analiza las actividades de administración, como los cambios en los usuarios (agregar, quitar, etc.), la creación de grupos y las modificaciones.  

   - Agregue un libro para el firewall. Por ejemplo, agregue el libro Palo Alto. El libro analiza el tráfico del firewall, lo que proporciona correlaciones entre los datos del firewall y los eventos de amenazas, y resalta los eventos sospechosos de las distintas entidades. Los libros proporcionan información sobre las tendencias en el tráfico y permite explorar en profundidad los resultados y filtrarlos. 

      ![Panel Palo Alto](./media/qs-get-visibility/palo-alto-week-query.png)


Puede personalizar los libros mediante la edición de la consulta principal ![botón Editar consulta](./media/qs-get-visibility/edit-query-button.png). Puede hacer clic en el botón ![botón Log Analytics](./media/qs-get-visibility/go-to-la-button.png) para ir a [Log Analytics y editar ahí la consulta](../azure-monitor/log-query/get-started-portal.md), y puede seleccionar el botón de puntos suspensivos (...) y, después, **Personalizar los datos del icono**, que le permite editar el filtro de tiempo principal o quitar iconos concretos del libro.

Para más información sobre cómo trabajar con consultas, consulte [Tutorial: Visualización de datos en Log Analytics](../azure-monitor/learn/tutorial-logs-dashboards.md)

### <a name="add-a-new-tile"></a>Agregar un nuevo icono

Si desea agregar un icono, puede agregarlo a un libro existente (uno que cree o uno integrado en Azure Sentinel). 
1. En Log Analytics, cree un icono mediante las instrucciones que encontrará en [Tutorial: Visualización de datos en Log Analytics](../azure-monitor/learn/tutorial-logs-dashboards.md). 
2. Después de crear el icono, en **Anclar**, seleccione el libro en el que desee que aparezca.

## <a name="create-new-workbooks"></a>Creación de libros
Puede crear un libro desde cero o usar un libro integrado como base para un nuevo libro.

1. Para crear un libro desde cero, seleccione **Libros** y, después, **+Nuevo libro**.
2. Seleccione la suscripción en la que se crea el libro y asígnele un nombre descriptivo. Cada libro es un recurso de Azure como cualquier otro y puede asignarle roles (RBAC), con el fin de definir y limitar quién puede acceder. 
3. Para habilitarlo y mostrarlo en los libros a los que se anclan las visualizaciones, tiene que compartirlo. Haga clic en **Share** (Compartir) y, luego, en **Manage users** (Administrar usuarios). 
 
1. Use **Check access** (Comprobar acceso) y **Role assignments** (Asignaciones de roles) como haría con cualquier otro recurso de Azure. Para más información, consulte [Uso compartido de libros de Azure mediante el control de acceso basado en rol](../azure-portal/azure-portal-dashboard-share-access.md).


## <a name="new-workbook-examples"></a>Ejemplos de libros nuevos

La siguiente consulta de ejemplo le permite comparar las tendencias del tráfico entre semanas. Puede cambiar fácilmente en el proveedor de dispositivos y el origen de datos en que se ejecutará la consulta. En este ejemplo se usa SecurityEvent de Windows, pero puede cambiarlo por AzureActivity, CommonSecurityLog o cualquier otro firewall.

```console
 |where DeviceVendor == "Palo Alto Networks":
  // week over week query
  SecurityEvent
  | where TimeGenerated > ago(14d)
  | summarize count() by bin(TimeGenerated, 1d)
  | extend Week = iff(TimeGenerated>ago(7d), "This Week", "Last Week"), TimeGenerated = iff(TimeGenerated>ago(7d), TimeGenerated, TimeGenerated + 7d)
```

Puede que quiera crear una consulta que incorpore datos de varios orígenes. Puede crear una consulta que examine los registros de auditoría de Azure Active Directory de los nuevos usuarios que acaba de crear y que luego compruebe los registros de Azure para ver si el usuario comenzó a realizar cambios en la asignación de roles al cabo de 24 horas de la creación. En este panel aparecería esa actividad sospechosa:

```console
AuditLogs
| where OperationName == "Add user"
| project AddedTime = TimeGenerated, user = tostring(TargetResources[0].userPrincipalName)
| join (AzureActivity
| where OperationName == "Create role assignment"
| project OperationName, RoleAssignmentTime = TimeGenerated, user = Caller) on user
| project-away user1
```

Puede crear distintos libros en función del rol de la persona que examina los datos y lo que busca. Por ejemplo, puede crear un libro para el administrador de red que incluya los datos del firewall. También puede crear libros en función de la frecuencia con la que quiera verlos, si hay cosas que quiera revisar a diario y otros elementos que quiera comprobar cada hora; por ejemplo, podría querer examinar sus inicios de sesión de Azure AD cada hora para detectar anomalías. 

## <a name="create-new-detections"></a>Creación de nuevas detecciones

Genere detecciones en los [orígenes de datos que ha conectado a Azure Sentinel](connect-data-sources.md) para investigar las amenazas de su organización.

Cuando cree una nueva detección, aproveche las detecciones integradas diseñadas por los investigadores de seguridad de Microsoft que se adapten a los orígenes de datos que ha conectado.

Para ver todas las detecciones estándar, vaya a **Análisis** y, después, a **Rule templates** (Plantillas de reglas). Esta pestaña contiene todas las reglas integradas de Azure Sentinel.

   ![Usar las detecciones integradas para encontrar amenazas con Azure Sentinel](media/tutorial-detect-built-in/view-oob-detections.png)

Para más información sobre cómo obtener detecciones estándar, consulte [Tutorial: Obtención de análisis integrado](tutorial-detect-threats-built-in.md).
 
## <a name="next-steps"></a>Pasos siguientes
En este inicio rápido, ha aprendido cómo empezar a usar Azure Sentinel. Siga con el tutorial sobre [cómo detectar amenazas](tutorial-detect-threats-built-in.md).
> [!div class="nextstepaction"]
> [Cree reglas de detección de amenazas personalizadas](tutorial-detect-threats-custom.md) para automatizar las respuestas a las amenazas.

