---
title: Advanced Data Security de Azure Security Center para máquinas SQL (versión preliminar)
description: Más información sobre cómo habilitar Advanced Data Security para máquinas SQL en Azure Security Center
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: ba46c460-6ba7-48b2-a6a7-ec802dd4eec2
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/28/2020
ms.author: memildin
ms.openlocfilehash: e0085ef5213853a1577ec039d5e360114aa7c64e
ms.sourcegitcommit: 37afde27ac137ab2e675b2b0492559287822fded
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88566274"
---
# <a name="advanced-data-security-for-sql-machines-preview"></a>Advanced Data Security para máquinas SQL (versión preliminar)

Advanced Data Security de Azure Security Center para máquinas SQL protege los servidores SQL Server hospedados en Azure, en otros entornos en la nube e incluso en máquinas locales. Esto amplía las protecciones de los servidores SQL Server nativos de Azure, de modo que se admitan totalmente los entornos híbridos.

En esta característica en versión preliminar se incluye funcionalidad para identificar y mitigar posibles vulnerabilidades de la base de datos, además de detectar actividades anómalas que puedan suponer alguna amenaza para la base de datos: 

* **Evaluación de vulnerabilidades**: servicio de exploración que puede detectar, seguir y corregir posibles vulnerabilidades en las bases de datos. Los exámenes de evaluación proporcionan información general sobre el estado de seguridad de las máquinas SQL y detalles de las conclusiones de seguridad.

* [Advanced Threat Protection](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview): servicio de detección que supervisa continuamente los servidores SQL Server para detectar amenazas tales como inyección de código SQL, ataques por fuerza bruta y abuso de privilegios. Este servicio proporciona alertas de seguridad orientadas a acciones en Azure Security Center con detalles de la actividad sospechosa, instrucciones sobre cómo mitigar las amenazas y opciones para continuar las investigaciones con Azure Sentinel.

>[!TIP]
> Advanced Data Security para máquinas SQL es una extensión del [paquete Advanced Data Security](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security) de Azure Security Center, disponible para Azure SQL Database, Azure Synapse y Azure SQL Managed Instance.


## <a name="set-up-advanced-data-security-for-sql-machines"></a>Configuración de Advanced Data Security para máquinas SQL 

La configuración de Azure Security Center de Azure Security Center para máquinas SQL implica dos pasos:

* Aprovisione el agente de Log Analytics en el host de SQL Server. Así se proporciona la conexión a Azure.

* Habilite el conjunto de productos opcional en la página de precios y configuración de Security Center.

Ambos métodos se describen a continuación.

### <a name="step-1-provision-the-log-analytics-agent-on-your-sql-servers-host"></a>Paso 1. Aprovisione el agente de Log Analytics en el host de SQL Server:

- **SQL Server en Azure VM**: si la máquina SQL se hospeda en una máquina virtual de Azure, puede [aprovisionar automáticamente el agente de Log Analytics](security-center-enable-data-collection.md#workspace-configuration). Como alternativa, puede seguir el procedimiento manual para [agregar una máquina virtual de Azure](quick-onboard-azure-stack.md#add-the-virtual-machine-extension-to-your-existing-azure-stack-virtual-machines).

- **SQL Server en Azure Arc**: si el servidor SQL Server se hospeda en una máquina de [Azure Arc](https://docs.microsoft.com/azure/azure-arc/), puede implementar el agente de Log Analytics con la recomendación "El agente de Log Analytics debe estar instalado en las máquinas basadas en Windows de Azure Arc (versión preliminar)." de Security Center. Como alternativa, puede seguir el procedimiento manual en la [documentación de Azure Arc](https://docs.microsoft.com/azure/azure-arc/servers/manage-vm-extensions#enable-extensions-from-the-portal).

- **SQL Server local**: si el servidor SQL Server se hospeda en una máquina local de Windows sin Azure Arc, tiene dos opciones para conectarlo a Azure:
    
    - **Implementación de Azure Arc**: puede conectar cualquier máquina de Windows a Security Center. Sin embargo, Azure Arc proporciona una integración más profunda entre *todas* las de su entorno de Azure. Si configura Azure Arc, verá la página **SQL Server: Azure Arc** en el portal y las alertas de seguridad aparecerán en una pestaña **Seguridad** dedicada en esa página. Por lo tanto, la primera opción (y la recomendada) es [configurar Azure Arc en el host](https://docs.microsoft.com/azure/azure-arc/servers/onboard-portal#install-and-validate-the-agent-on-windows) y seguir las instrucciones anteriores para **SQL Server en Azure Arc**.
        
    - **Conexión de la máquina Windows sin Azure Arc**: si opta por conectar un servidor SQL Server que se ejecute en una máquina Windows sin usar Azure Arc, siga las instrucciones del artículo sobre cómo [conectar máquinas Windows a Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/agent-windows).


### <a name="step-2-enable-the-optional-bundle-in-security-centers-pricing-and-settings-page"></a>Paso 2. Habilite el conjunto de productos opcional en la página de precios y configuración de Security Center:

1. En la barra lateral de Security Center, abra la página **Precios y configuración**.

    - Si usa el **área de trabajo predeterminada de Azure Security Center** (denominada "defaultworkspace-[su identificador de suscripción]-[región]"), seleccione la **suscripción** correspondiente.

    - Si utiliza **un área de trabajo no predeterminada**, seleccione el **área de trabajo** correspondiente (escriba el nombre del área de trabajo en el filtro si es necesario):

        ![title](./media/security-center-advanced-iaas-data/pricing-and-settings-workspaces.png)


1. Active la opción de **SQL servers on VM (Preview)** (Servidores SQL Server en las máquinas virtuales [versión preliminar]) para habilitarla. 

    [![Página de precios de Security Center con paquetes opcionales](media/security-center-advanced-iaas-data/sql-servers-on-vms-in-pricing-small.png)](media/security-center-advanced-iaas-data/sql-servers-on-vms-in-pricing-large.png#lightbox)

    Advanced Data Security para servidores SQL Server en máquinas se habilitará en todos los servidores SQL Server conectados al área de trabajo seleccionada. La protección de datos estará totalmente activa después del primer reinicio de la instancia de SQL Server.

    >[!TIP] 
    > Para crear una nueva área de trabajo, siga las instrucciones de [Crear un área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace).


1. Opcionalmente, configure notificaciones de alertas de seguridad por correo electrónico. 
    Puede establecer una lista de destinatarios que reciban una notificación por correo electrónico cuando se generen alertas de Security Center. El correo electrónico contiene un vínculo directo a la alerta de Azure Security Center con todos los detalles pertinentes. Para más información, consulte el artículo sobre cómo [configurar las notificaciones por correo electrónico para recibir alertas](https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details).



## <a name="explore-vulnerability-assessment-reports"></a>Exploración de los informes de evaluación de vulnerabilidades

El servicio de evaluación de vulnerabilidades examina las bases de datos una vez a la semana. Los exámenes se ejecutan el mismo día de la semana en que se habilitó el servicio.

El panel de evaluación de vulnerabilidades proporciona una visión general de los resultados de la evaluación en todas las bases de datos, junto con un resumen de las bases de datos correctas y en mal estado, además de un resumen general de las comprobaciones con errores en función de la distribución del riesgo.

Puede ver los resultados de la evaluación de vulnerabilidades directamente desde Security Center.

1. En la barra lateral de Security Center, abra la página **Recomendaciones** y seleccione la recomendación **Es necesario corregir las vulnerabilidades de los servidores SQL Server en las máquinas (versión preliminar)** . Para más información, consulte [Recomendaciones de Security Center](security-center-recommendations.md). 


    [![Recomendación **Es necesario corregir las vulnerabilidades de los servidores SQL Server en las máquinas (versión preliminar)**](media/security-center-advanced-iaas-data/data-and-storage-sqldb-vulns-on-vm.png)](media/security-center-advanced-iaas-data/data-and-storage-sqldb-vulns-on-vm.png#lightbox)

    Aparece la vista detallada de esta recomendación.

    [![Vista detallada de la recomendación **Es necesario corregir las vulnerabilidades de los servidores SQL Server en las máquinas (versión preliminar)**](media/security-center-advanced-iaas-data/all-servers-view.png)](media/security-center-advanced-iaas-data/all-servers-view.png#lightbox)

1. Para más detalles, explore en profundidad lo siguiente:

    * Para obtener información general de los recursos examinados (bases de datos) y la lista de comprobaciones de seguridad que se probaron, seleccione el servidor que le interesa.

    * Para obtener información general sobre las vulnerabilidades agrupadas por una base de datos SQL específica, seleccione la base de datos que le interesa.

    En cada vista, las comprobaciones de seguridad se ordenan por **gravedad**. Haga clic en una comprobación de seguridad específica para ver un panel de detalles con una **descripción**, cómo **corregir** el problema y otra información relacionada como, por ejemplo, el **impacto** o el **banco de pruebas**.

## <a name="advanced-threat-protection-for-sql-servers-on-machines-alerts"></a>Alertas de Advanced Threat Protection para servidores SQL Server en las máquinas
Las alertas se generan a partir de intentos inusuales y potencialmente dañinos para acceder a las máquinas SQL o aprovechar sus vulnerabilidades. Estos eventos pueden desencadenar las alertas que se muestran en la sección de [alertas de SQL Database y SQL Data Warehouse de la página de referencia de alertas](alerts-reference.md#alerts-sql-db-and-warehouse).



## <a name="explore-and-investigate-security-alerts"></a>Exploración e investigación de las alertas de seguridad

Las alertas de seguridad están disponibles en la página de alertas de Security Center, en la pestaña seguridad del recurso o a través del vínculo directo de los correos electrónicos de alerta.

1. Para verlas, seleccione **Alertas de seguridad** en la barra lateral de Security Center y seleccione una alerta.

1. Las alertas están diseñadas para ser independientes, con pasos de corrección detallados e información de investigación en cada una. Puede investigar más con otras funcionalidades de Azure Security Center y Azure Sentinel para obtener una visión más general:

    * Habilite la característica de auditoría de SQL Server para realizar más investigaciones. Si es usuario de Azure Sentinel, puede cargar los registros de auditoría de SQL desde los eventos del registro de seguridad de Windows a Sentinel y disfrutar de una experiencia de investigación enriquecida. [Obtenga más información sobre cómo realizar auditorías de SQL Server](https://docs.microsoft.com/sql/relational-databases/security/auditing/create-a-server-audit-and-server-audit-specification?view=sql-server-ver15).
    * Para mejorar la posición de seguridad, siga las recomendaciones de Security Center para la máquina host indicadas en cada alerta. Esto reducirá los riesgos de futuros ataques. 

    [Obtenga más información sobre cómo administrar y responder a alertas](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts).


## <a name="next-steps"></a>Pasos siguientes

Para obtener material relacionado, vea el siguiente artículo:

- [Alertas de seguridad de SQL Database y SQL Data Warehouse](alerts-reference.md#alerts-sql-db-and-warehouse)
- [Configuración de notificaciones de alertas de seguridad por correo electrónico](security-center-provide-security-contact-details.md)
- [Más información sobre Azure Sentinel](https://docs.microsoft.com/azure/sentinel/)
- [Paquete Advanced Data Security de Azure Security Center](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)