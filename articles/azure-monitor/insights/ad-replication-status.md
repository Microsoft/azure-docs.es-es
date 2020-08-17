---
title: Supervisión de Active Directory Replication Status
description: El paquete de la solución Active Directory Replication Status supervisa con regularidad el entorno de Active Directory para comprobar si existen errores de replicación.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/24/2018
ms.openlocfilehash: c33e9105be1eb080025922ff9e612771a4f021cd
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318086"
---
# <a name="monitor-active-directory-replication-status-with-azure-monitor"></a>Supervisión de Active Directory Replication Status con Azure Monitor

![Símbolo de AD Replication Status](./media/ad-replication-status/ad-replication-status-symbol.png)

Active Directory es un componente clave de un entorno de TI empresarial. Para garantizar la alta disponibilidad y el alto rendimiento, cada controlador de dominio tiene su propia copia de la base de datos de Active Directory. Los controladores de dominio se replican entre sí con el fin de propagar los cambios en toda la empresa. Los errores en este proceso de replicación pueden provocar una serie de problemas en toda la empresa.

La solución AD Replication Status supervisa con regularidad el entorno de Active Directory para comprobar si existen errores de replicación.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand-solution.md)]

## <a name="installing-and-configuring-the-solution"></a>Instalación y configuración de la solución
Utilice la siguiente información para instalar y configurar la solución.

### <a name="prerequisites"></a>Requisitos previos

* La solución AD Replication Status requiere que esté instalada una versión compatible de .NET Framework 4.6.2 o superior en todos los equipos que tengan instalado el agente de Log Analytics para Windows (también conocido como Microsoft Monitoring Agent [MMA]).  System Center 2016 - Operations Manager, Operations Manager 2012 R2 y Azure Monitor usan el agente.
* La solución es compatible con controladores de dominio que ejecutan Windows Server 2008, y 2008 R2, Windows Server 2012, y 2012 R2 y Windows Server 2016.
* Un área de trabajo de Log Analytics para agregar la solución Active Directory Health Check desde Azure Marketplace en Azure Portal. No se necesita ninguna configuración adicional.


### <a name="install-agents-on-domain-controllers"></a>Instalación de agentes en controladores de dominio
Debe instalar los agentes en controladores del dominio que se vaya a evaluar. Como alternativa, instale los agentes en servidores miembros y configure los agentes para que envíen datos de replicación de AD a Azure Monitor. Para conocer el proceso de conexión de equipos Windows a Azure Monitor, consulte [Conexión de equipos Windows a Azure Monitor](../platform/agent-windows.md). Si el controlador de dominio ya forma parte de un entorno de System Center Operations Manager que le gustaría conectar a Azure Monitor, consulte [Conexión de Operations Manager con Azure Monitor](../platform/om-agents.md).

### <a name="enable-non-domain-controller"></a>Habilitación de controladores que no son de dominio
Si no desea conectar ninguno de los controladores de dominio directamente a Azure Monitor, puede usar cualquier otro equipo en el dominio conectado a Azure Monitor para recopilar datos para el paquete de solución de AD Replication Status y hacer que este envíe los datos.

1. Compruebe que el equipo es miembro del dominio que desea supervisar mediante la solución de Estado de replicación de AD.
2. [Conecte el equipo Windows a Azure Monitor](../platform/om-agents.md) o [conéctelo con su entorno existente de Operations Manager a Azure Monitor](../platform/om-agents.md), si no está conectado aún.
3. En el equipo, configure la siguiente clave del Registro:<br>Clave: **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HealthService\Parameters\Management Groups\<ManagementGroupName>\Solutions\ADReplication**<br>Valor: **IsTarget**<br>Datos del valor: **true**

   > [!NOTE]
   > Estos cambios no surten efecto hasta que reinicia el servicio Microsoft Monitoring Agent (HealthService.exe).
   > ### <a name="install-solution"></a>Instalación de una solución
   > Siga el proceso descrito en [Instalación de una solución de supervisión](solutions.md#install-a-monitoring-solution) para agregar la solución **Active Directory Replication Status** al área de trabajo de Log Analytics. No es necesario realizar ninguna configuración más.


## <a name="ad-replication-status-data-collection-details"></a>Detalles de recopilación de datos de Estado de replicación de AD
En la tabla siguiente se muestran los métodos de recolección de datos y otros detalles sobre cómo se recopilan los datos para el Estado de replicación de AD.

| plataforma | Agente directo | Agente de SCOM | Azure Storage | ¿Se necesita SCOM? | Datos del agente de SCOM enviados a través del grupo de administración | Frecuencia de recopilación |
| --- | --- | --- | --- | --- | --- | --- |
| Windows |&#8226; |&#8226; |  |  |&#8226; |cada cinco días |



## <a name="understanding-replication-errors"></a>Descripción de los errores de replicación

[!INCLUDE [azure-monitor-solutions-overview-page](../../../includes/azure-monitor-solutions-overview-page.md)]

El icono de AD Replication Status muestra cuántos errores de replicación tiene actualmente. Los **errores críticos de replicación** son aquellos que están al 75 % de la [vigencia del marcador de exclusión](/previous-versions/windows/it-pro/windows-server-2003/cc784932(v=ws.10)) o por encima para el bosque de Active Directory.

![Icono del Estado de replicación de AD](./media/ad-replication-status/oms-ad-replication-tile.png)

Al hacer clic en el icono, se ve más información sobre los errores.
![Panel del estado de replicación de AD](./media/ad-replication-status/oms-ad-replication-dash.png)

### <a name="destination-server-status-and-source-server-status"></a>Estado del servidor de destino y estado del servidor de origen
Estas columnas muestran el estado de los servidores de destino y los servidores de origen que experimentan errores de replicación. El número después de cada nombre de controlador de dominio indica el número de errores de replicación en ese controlador de dominio.

Dado que algunos problemas son más fáciles de solucionar desde la perspectiva del servidor de origen y otros desde la perspectiva del servidor de destino, se muestran los errores tanto de los servidores de destino como de los servidores de origen.

En este ejemplo, verá que muchos servidores de destino tienen aproximadamente el mismo número de errores, pero hay un servidor de origen (ADDC35) que tiene muchos más errores que los demás. Es probable que haya algún problema en ADDC35 que esté provocando un error al enviar datos a sus asociados de replicación. Con la solución de los problemas de ADDC35 se podrían resolver muchos de los errores que aparecen en el área del servidor de destino.

### <a name="replication-error-types"></a>Tipos de errores de replicación
Esta área proporciona información sobre los tipos de errores detectados en toda la empresa. Cada error posee un código numérico único y un mensaje que puede ayudarle a determinar la causa del error.

El anillo de la parte superior le permite hacerse una idea de los errores que aparecen con mayor y menor frecuencia en su entorno.

Muestra si hay varios controladores de dominio que experimentan el mismo error de replicación. En este caso, es posible que pueda detectar o identificar una solución en un controlador de dominio y repetirla en los demás controladores de dominio afectados.

### <a name="tombstone-lifetime"></a>vigencia de objetos de desecho
La vigencia de objetos de desecho determina el tiempo que un objeto eliminado permanece en la base de datos de Active Directory. Cuando un objeto eliminado supera la vigencia de objetos de desecho, el proceso de recopilación de elementos no utilizados lo quita automáticamente de la base de datos de Active Directory.

La vigencia predeterminada de objetos de desecho es de 180 días para las versiones más recientes de Windows (60 días en versiones anteriores) y puede cambiarse explícitamente por un administrador de Active Directory.

Es importante saber si tiene errores de replicación próximos al fin de la vigencia del marcador de exclusión o la hayan superado. Si dos controladores de dominio experimentan un error de replicación que se mantiene una vez superada la vigencia del marcador de exclusión, se deshabilita la replicación entre esos dos controladores de dominio, aunque se corrija el error de replicación subyacente.

El área de la vigencia de marcadores de exclusión le ayuda a identificar los lugares donde existe el riesgo de que se deshabilite la replicación. Cada error en la categoría **Más del 100 % de la vigencia de objetos de desecho** representa una partición que no se ha replicado entre su servidor de origen y destino durante al menos la vigencia de objetos de desecho para el bosque.

En este caso, no bastará con corregir el error de replicación. Como mínimo, debe investigar manualmente para identificar y limpiar los objetos persistentes antes de reiniciar la replicación. Puede incluso que deba dar de baja un controlador de dominio.

Además de identificar los errores de replicación que se hayan mantenido una vez expirada la vigencia del marcador de exclusión, deberá prestar atención a los errores de las categorías **50-75 % de la vigencia del marcador de exclusión** o **75-100 % de la vigencia del marcador de exclusión**.

Estos son errores que son claramente persistentes, no transitorios, por lo que es probable que necesiten la intervención del usuario para resolverse. Lo bueno es que no han alcanzado aún el fin de la vigencia de objetos de desecho. Si soluciona estos problemas con prontitud y *antes* de que lleguen al límite de la vigencia de objetos de desecho, puede reiniciar la replicación con la mínima intervención manual.

Como se indicó anteriormente, el icono del panel de la solución de Estado de replicación de AD muestra el número de errores de replicación *críticos* en su entorno, que se definen como errores que están por encima del 75 % de la vigencia de objetos de desecho (incluidos los errores que están en el 100 % de la vigencia de objetos de desecho). El objetivo es mantener este número en 0.

> [!NOTE]
> Todos los cálculos de porcentaje de la vigencia de objetos de desecho se basan en la vigencia de objetos de desecho real para el bosque de Active Directory, por lo que puede confiar en que los porcentajes son precisos, incluso si ha configurado un valor personalizado para la vigencia de objetos de desecho.
>
>

### <a name="ad-replication-status-details"></a>Detalles del Estado de replicación de AD
Al hacer clic en cualquier elemento de las listas con una consulta de registros verá detalles adicionales sobre este. Los resultados se filtran para mostrar solo los errores relacionados con ese elemento. Por ejemplo, al hacer clic en el primer controlador de dominio que aparece en **Destination Server Status (ADDC02)** (Estado del servidor de destino [ADDC02]), verá los resultados de la consulta filtrados para mostrar errores con ese controlador de dominio cuando figura como servidor de destino:

![Errores de AD Replication Status en los resultados de la consulta](./media/ad-replication-status/oms-ad-replication-search-details.png)

Desde aquí, puede filtrar aún más, modificar la consulta de registros, etc. Para más información sobre el uso de las consultas de registros en Azure Monitor, consulte [Análisis de datos de registro en Azure Monitor](../log-query/log-query-overview.md).

El campo **HelpLink** muestra la dirección URL de una página de TechNet con detalles adicionales acerca de ese error específico. Puede copiar y pegar este vínculo en la ventana del explorador para ver información sobre la solución de problemas y corregir el error.

También puede hacer clic **Exportar** para exportar los resultados a Excel. La exportación de datos le ayuda a visualizar los datos del error de replicación de la forma que prefiera.

![errores del Estado de replicación de AD exportados en Excel](./media/ad-replication-status/oms-ad-replication-export.png)

## <a name="ad-replication-status-faq"></a>P+F del Estado de replicación de AD
**P: ¿Con qué frecuencia se actualizan los datos del estado de replicación de AD?**
A. La información se actualiza cada cinco días.

**P: ¿Existe es una manera de configurar la frecuencia de actualización de estos datos?**
A. De momento, no.

**P: ¿Tengo que agregar todos mis controladores de dominio a mi área de trabajo de Log Analytics para ver el estado de replicación?**
A. No, basta con que se agregue un único controlador de dominio. Si tiene varios controladores de dominio en el área de trabajo de Log Analytics, los datos de todos ellos se envían a Azure Monitor.

**P: No quiero agregar controladores de dominio a mi área de trabajo de Log Analytics. ¿Puedo usar la solución de estado de replicación de AD?**

A. Sí. Puede configurar el valor de una clave del Registro para habilitar esta opción. Consulte [Habilitación de controladores que no son de dominio](#enable-non-domain-controller).

**P: ¿Cuál es el nombre del proceso que realiza la recopilación de datos?**
A. AdvisorAssessment.exe

**P: ¿Cuánto tiempo tarda la recopilación de datos?**
A. El tiempo de recopilación de datos depende del tamaño del entorno de Active Directory, pero normalmente tarda menos de 15 minutos.

**P: ¿Qué tipo de datos se recopilan?**
A. La información de replicación se recopila a través de LDAP.

**P: ¿Se puede configurar el momento en que se recopilan los datos?**
A. De momento, no.

**P: ¿Qué permisos necesito para recopilar datos?**
A. Los permisos de usuario normal para Active Directory son suficientes.

## <a name="troubleshoot-data-collection-problems"></a>Solución de problemas de recopilación de datos
A fin de recopilar datos, el paquete de solución Active Directory Replication Status requiere que haya al menos un controlador de dominio conectado a su área de trabajo de Log Analytics. Hasta que se conecte a un controlador de dominio, aparece un mensaje que indica que **todavía se están recopilando datos**.

Si necesita ayuda para conectarse a uno de los controladores de dominio, puede ver la documentación en [Conexión de equipos Windows a Azure Monitor](../platform/om-agents.md). Como alternativa, si el controlador de dominio ya está conectado a un entorno existente de System Center Operations Manager, puede ver documentación en [Conexión de Operations Manager con Azure Monitor](../platform/om-agents.md).

Si no desea conectar ninguno de los controladores de dominio directamente a Azure Monitor o a System Center Operations Manager, consulte [Habilitación de controladores que no son de dominio](#enable-non-domain-controller).

## <a name="next-steps"></a>Pasos siguientes
* Utilice [Consultas de registros en Azure Monitor](../log-query/log-query-overview.md) para ver datos detallados de Active Directory Replication Status.

