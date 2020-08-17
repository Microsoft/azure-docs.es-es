---
title: Diseño de la implementación de registros de Azure Monitor | Microsoft Docs
description: En este artículo se describen las consideraciones y las recomendaciones para los clientes que se preparan para implementar un área de trabajo en Azure Monitor.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/20/2019
ms.openlocfilehash: 3a6afd42c12a523523b45861b38b323fa680ecab
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317291"
---
# <a name="designing-your-azure-monitor-logs-deployment"></a>Diseño de la implementación de registros de Azure Monitor

Azure Monitor almacena los datos de [registro](data-platform-logs.md) en un área de trabajo de Log Analytics, que es un recurso de Azure y un contenedor en el que los datos se recopilan y se agregan, y que sirve como límite administrativo. Aunque puede implementar una o varias áreas de trabajo en su suscripción de Azure, hay varios aspectos que debe comprender para asegurarse de que la implementación inicial sigue nuestras instrucciones con el fin de que resulte una implementación rentable, fácil de administrar y escalable que satisfaga las necesidades de su organización.

En las áreas de trabajo, los datos se organizan en tablas, cada una de las cuales almacena distintos tipos de datos y tiene su conjunto exclusivo de propiedades según el recurso que genere los datos. En un área de trabajo Log Analytics, la mayoría de los orígenes de datos escribirán en sus propias tablas.

![Modelo de datos del área de trabajo de ejemplo](./media/design-logs-deployment/logs-data-model-01.png)

Un área de trabajo de Log Analytics proporciona lo siguiente:

* Una ubicación geográfica para el almacenamiento de datos.
* Aislamiento de datos, ya que se conceden diferentes derechos de acceso a los usuarios en base a una de nuestras estrategias de diseño recomendadas.
* Ámbito para la configuración de opciones, como el [plan de tarifa](./manage-cost-storage.md#changing-pricing-tier), la [retención](./manage-cost-storage.md#change-the-data-retention-period) y el [límite de datos](./manage-cost-storage.md#manage-your-maximum-daily-data-volume).

En este artículo se proporciona información general detallada acerca de las consideraciones de diseño y migración, información general sobre el control de acceso y una descripción de las implementaciones de diseño que recomendamos para su organización de TI.



## <a name="important-considerations-for-an-access-control-strategy"></a>Consideraciones importantes para una estrategia de control de acceso

La identificación del número de áreas de trabajo que necesita se ve afectada por uno o varios de los siguientes requisitos:

* Una empresa internacional que necesita que los datos de registro estén almacenados en regiones concretas por motivos de soberanía de datos o cumplimiento normativo.
* Un usuario de Azure que desea evitar los gastos de transferencia de datos de salida manteniendo un área de trabajo en la misma región que los recursos de Azure que administra.
* Administra varios departamentos o grupos de negocio y desea que cada uno de ellos vea sus propios datos, pero no los de los demás. Además, no hay ningún requisito empresarial para las vistas consolidadas entre los distintos grupos de negocio o departamentos.

En la actualidad, las organizaciones de TI siguen modelos centralizados, descentralizados o usan una estructura híbrida. Como resultado, los siguientes modelos de implementación de áreas de trabajo se han usado normalmente para asignarse a una de estas estructuras organizativas:

* **Centralizado**: todos los registros se almacenan en un área de trabajo central y los administra un único equipo, y Azure Monitor proporcionar un acceso diferenciado por equipo. En este escenario, es fácil administrar, buscar en los recursos y correlacionar registros entre ellos. El área de trabajo puede crecer considerablemente en función de la cantidad de datos recopilados de varios recursos de la suscripción, lo que genera una sobrecarga administrativa adicional para mantener el control del acceso de los distintos usuarios. Este modelo se conoce como "concentrador y radio".
* **Descentralizado**: cada equipo tiene su propia área de trabajo creada en un grupo de recursos que poseen y administran, y los datos de registro se segregan por recurso. En este escenario, el área de trabajo se puede mantener seguro y el control de acceso es coherente con el acceso de los recursos, pero es difícil correlacionar los registros. Los usuarios que necesitan una visión amplia de muchos recursos no pueden analizar los datos de forma significativa.
* **Híbrido**: los requisitos de cumplimiento de auditorías de seguridad complican aún más este escenario, ya que muchas organizaciones implementan ambos modelos de implementación en paralelo. Habitualmente, esto genera una configuración compleja, cara y difícil de mantener con brechas en la cobertura de los registros.

Si se usan los agentes de Log Analytics para recopilar datos, es preciso conocer lo siguiente para planear la implementación de los agentes:

* Para recopilar datos de los agentes de Windows, puede [configurar cada agente para que informe a una o varias áreas de trabajo](./agent-windows.md), aun cuando informen a un grupo de administración de System Center Operations Manager. El agente de Windows puede notificar hasta cuatro áreas de trabajo.
* El agente de Linux no admite el hospedaje múltiple y solo puede informar a un área de trabajo.

Si usa System Center Operations Manager 2012 R2, o cualquier versión posterior:

* Cada grupo de administración de Operations Manager se puede [conectar a una sola área de trabajo](./om-agents.md). 
* Los equipos Linux que informan a un grupo de administración deben estar configurados para informar directamente a un área de trabajo Log Analytics. Si los equipos Linux ya están notificando directamente a un área de trabajo y desea supervisarlos con Operations Manager, siga estos para [notificar a un grupo de administración de Operations Manager](agent-manage.md#configure-agent-to-report-to-an-operations-manager-management-group). 
* El agente de Windows de Log Analytics se puede instalar en el equipo Windows y se le puede indicar que informe tanto a Operations Manager integrado con un área de trabajo como a otra área de trabajo.

## <a name="access-control-overview"></a>Introducción al control de acceso

Con el control de acceso basado en rol (RBAC) puede conceder a los usuarios y grupos solo la cantidad de acceso que necesitan para trabajar con los datos de supervisión en un área de trabajo. Esto le permite alinearse con el modelo de funcionamiento de la organización de TI mediante el uso de una sola área de trabajo para almacenar los datos recopilados habilitada en todos los recursos. Por ejemplo, puede conceder acceso al equipo responsable de los servicios de infraestructura hospedados en máquinas virtuales (VM) de Azure y, como consecuencia, este equipo solo tendrá acceso a los registros que hayan generados las máquinas virtuales. Esto sigue nuestro nuevo modelo de registro del contexto de recursos. La base de este modelo es que cada entrada de registro que emite un recurso de Azure se asocia automáticamente con este recurso. Los registros se reenvían a un área de trabajo central que respeta el ámbito y el RBAC en función de los recursos.

Los datos a los que un usuario tiene acceso vienen determinados por una combinación de factores que se enumeran en la tabla siguiente. Cada uno se describe en las secciones siguientes.

| Factor | Descripción |
|:---|:---|
| [Modo de acceso](#access-mode) | Método que utiliza el usuario para acceder al área de trabajo.  Define el ámbito de los datos disponibles y el modo de control de acceso que se aplica. |
| [Modo de control de acceso](#access-control-mode) | Configuración en el área de trabajo que define si los permisos se aplican en el nivel de área de trabajo o recurso. |
| [Permisos](manage-access.md) | Permisos aplicados a individuales o grupos de usuarios para el área de trabajo o el recurso. Define los datos a los que el usuario tendrá acceso. |
| [RBAC de nivel de tabla](manage-access.md#table-level-rbac) | Permisos granulares opcionales que se aplican a todos los usuarios independientemente de su modo de acceso o su modo de control de acceso. Define a qué tipos de datos puede tener acceso un usuario. |

## <a name="access-mode"></a>Modo de acceso

El *modo de acceso* hace referencia a cómo un usuario accede a un área de trabajo de Log Analytics y define el ámbito de datos al que puede tener acceso. 

Los usuarios tienen dos opciones para acceder a los datos:

* **Contexto del área de trabajo**: puede ver todos los registros del área de trabajo para la que tiene permiso. Las consultas en este modo se limitan a todos los datos de todas las tablas en el área de trabajo. Este es el modo de acceso utilizado cuando se accede a los registros con el área de trabajo como ámbito, como cuando se selecciona **Registros** desde el menú **Azure Monitor** en Azure Portal.

    ![Contexto de Log Analytics desde el área de trabajo](./media/design-logs-deployment/query-from-workspace.png)

* **Contexto del recurso**: al acceder al área de trabajo para un recurso, un grupo de recursos o una suscripción determinados (por ejemplo, cuando se selecciona **Registros** en un menú de recursos de Azure Portal), puede ver los registros solo de los recursos de todas las tablas a las que tiene acceso. Las consultas de este modo se limitan solo a los datos asociados a ese recurso. Este modo también permite el RBAC pormenorizado.

    ![Contexto de Log Analytics desde un recurso](./media/design-logs-deployment/query-from-resource.png)

    > [!NOTE]
    > Solo hay registros disponibles para las consultas de contexto del recurso si se asociaron correctamente al recurso pertinente. Actualmente, los siguientes recursos tienen limitaciones:
    > - Equipos fuera de Azure
    > - Service Fabric
    > - Application Insights
    >
    > Para probar si los registros están asociados correctamente con su recurso, puede ejecutar una consulta e inspeccionar los que le interesen. Si el identificador de recurso correcto se encuentra en la propiedad [_ResourceId](log-standard-properties.md#_resourceid), los datos están disponibles para las consultas basadas en el recurso.

Azure Monitor determina automáticamente el modo correcto en función del contexto desde el que se realiza la búsqueda de registros. El ámbito siempre se presenta en la sección superior izquierda de Log Analytics.

### <a name="comparing-access-modes"></a>Comparación de los modos de acceso

En la tabla siguiente se resumen los modos de acceso:

| Incidencia | Contexto del área de trabajo | Contexto del recurso |
|:---|:---|:---|
| ¿Para quién está pensado cada modelo? | Administración central. Los administradores que tienen que configurar colecciones de datos y los usuarios que necesitan acceder a una amplia variedad de recursos. También lo requieren actualmente los usuarios que necesitan acceder a registros de recursos fuera de Azure. | Equipos de la aplicación. Los administradores de los recursos de Azure que se están supervisando. |
| ¿Qué requiere un usuario para ver los registros? | Permisos para el área de trabajo. Vea los **permisos del área de trabajo** en [Administración del acceso mediante los permisos del área de trabajo](manage-access.md#manage-access-using-workspace-permissions). | Acceso de lectura al recurso. Vea los **permisos de los recursos** en [Administración del acceso mediante los permisos de Azure](manage-access.md#manage-access-using-azure-permissions). Los permisos pueden ser heredados (por ejemplo, del grupo de recursos que los contenga) o son asignados directamente al recurso. Se asignará automáticamente el permiso a los registros para el recurso. |
| ¿Qué es el ámbito de los permisos? | Área de trabajo. Los usuarios con acceso al área de trabajo pueden consultar todos los registros de esa área de trabajo desde las tablas para las que tengan permisos. Consulte [Control de acceso a la tabla](manage-access.md#table-level-rbac) | Recurso de Azure. Un usuario puede consultar los registros de recursos, grupos de recursos o suscripciones específicos a los que tenga acceso desde cualquier área de trabajo, pero no puede consultar los registros de otros recursos. |
| ¿Cómo puede el usuario acceder a los registros de acceso? | <ul><li>Inicie **Registros** en el menú de **Azure Monitor**.</li></ul> <ul><li>Inicie **Registros** desde **Áreas de trabajo de Log Analytics**.</li></ul> <ul><li>Desde [Libros](../visualizations.md#workbooks) de Azure Monitor.</li></ul> | <ul><li>Inicie **Registros** en el menú para el recurso de Azure</li></ul> <ul><li>Inicie **Registros** en el menú de **Azure Monitor**.</li></ul> <ul><li>Inicie **Registros** desde **Áreas de trabajo de Log Analytics**.</li></ul> <ul><li>Desde [Libros](../visualizations.md#workbooks) de Azure Monitor.</li></ul> |

## <a name="access-control-mode"></a>Modo de control de acceso

*Modo de control de acceso* es un valor que se encuentra en todas las áreas de trabajo y que define cómo se determinan los permisos para el área de trabajo.

* **Requerir permisos de área de trabajo**: este modo de control no admite RBAC granular. Para que un usuario pueda acceder al área de trabajo, debe tener permisos en el área de trabajo o en tablas específicas.

    Si un usuario accede al área de trabajo en el modo de contexto del área de trabajo, tendrá acceso a todos los datos de todas las tablas a las que se le haya concedido acceso. Si un usuario accede al área de trabajo en el modo de contexto del recurso, tendrá acceso solo a los datos de ese recurso en todas las tablas a las que se le haya concedido acceso.

    Se trata de la configuración predeterminada para todas las áreas de trabajo creadas antes de marzo de 2019.

* **Usar permisos de recurso o de área de trabajo**: este modo de control permite RBAC granular. A los usuarios se les puede conceder acceso solo a los datos asociados a los recursos que pueden ver mediante la asignación del permiso `read` de Azure. 

    Cuando un usuario accede al área de trabajo en modo contexto del área de trabajo, se aplican los permisos del área de trabajo. Cuando un usuario accede al área de trabajo en modo contexto del recurso, solo se comprueban los permisos de los recursos y se omiten los del área de trabajo. Para habilitar RBAC para un usuario, quítelos de los permisos del área de trabajo y permita que sus permisos de recursos sean reconocidos.

    Se trata de la configuración predeterminada para todas las áreas de trabajo creadas después de marzo de 2019.

    > [!NOTE]
    > Si un usuario solo tiene permisos de recurso en el área de trabajo, solo podrá acceder al área de trabajo mediante el modo contexto del recurso, siempre que el modo de acceso al área de trabajo esté establecido en **Usar permisos de recurso o de área de trabajo**.

Para obtener información sobre cómo cambiar el modo de control de acceso en el portal, con PowerShell o mediante una plantilla de Resource Manager, vea [Configuración del modo de control de acceso](manage-access.md#configure-access-control-mode).

## <a name="ingestion-volume-rate-limit"></a>Límite de velocidad por volumen de ingesta

Azure Monitor es un servicio de datos a gran escala que atiende a miles de clientes que envían terabytes de datos cada mes a un ritmo creciente. El umbral de velocidad de ingesta predeterminado se establece en **6 GB/min** por área de trabajo. Este es un valor aproximado, ya que el tamaño real puede variar entre los tipos de datos en función de la longitud del registro y su razón de compresión. Este límite no se aplica a los datos que se envían desde agentes o la [Data Collector API](data-collector-api.md).

Si envía datos a una velocidad superior a una sola área de trabajo, se quitan algunos datos y se envía un evento a la tabla *Operación* del área de trabajo cada 6 horas mientras se siga superando el umbral. Si el volumen de ingesta sigue superando el límite de velocidad o prevé que pronto lo alcanzará, puede solicitar un aumento en el área de trabajo mediante el envío de un correo electrónico a LAIngestionRate@microsoft.com o mediante la apertura de una solicitud de soporte técnico.
 
Para recibir notificaciones de este tipo de evento en el área de trabajo, cree una [regla de alerta de registro](alerts-log.md) mediante la siguiente consulta con la base de la lógica de alerta en el número de resultados mayor que cero.

``` Kusto
Operation
|where OperationCategory == "Ingestion"
|where Detail startswith "The rate of data crossed the threshold"
``` 


## <a name="recommendations"></a>Recomendaciones

![Ejemplo de diseño de contexto del recurso](./media/design-logs-deployment/workspace-design-resource-context-01.png)

En este escenario se trata un diseño de un área de trabajo único de la suscripción de una organización de TI que no está restringido por la soberanía de los datos ni por el cumplimiento normativo, o bien que debe asignarse a las regiones en las que se implementan los recursos. Brinda a los equipos de administración de seguridad y de TI de la organización la posibilidad de sacar provecho de la integración mejorada con la administración del acceso de Azure y un control del acceso más seguro.

Todos los recursos, las soluciones de supervisión y la información, como Application Insights y Azure Monitor para VM, que dan soporte tanto a la infraestructura como a las aplicaciones que mantienen los diferentes equipos están configurados para reenviar los datos de registro que recopilan al área de trabajo compartida centralizada de las organizaciones de TI. A los usuarios de cada equipo se les concede acceso a los registros de los recursos a los que se les ha dado acceso.

Una vez que haya implementado la arquitectura del área de trabajo, puede aplicarla a los recursos de Azure con [Azure Policy](../../governance/policy/overview.md). Proporciona una forma de definir directivas y garantizar el cumplimiento de los recursos de Azure para que envíen todos los registros de recursos a un área de trabajo específica. Por ejemplo, en el caso de las máquinas virtuales de Azure o los conjuntos de escalado de máquinas virtuales, puede usar las directivas existentes que evalúan el cumplimiento del área de trabajo y los resultados de los informes, o bien personalizarlas para tomas las medidas necesarias si no lo cumplen.  

## <a name="workspace-consolidation-migration-strategy"></a>Estrategia de migración de la consolidación de áreas de trabajo

En el caso de los clientes que ya han implementado varias áreas de trabajo y están interesados en consolidar el modelo de acceso de contexto del recurso, se recomienda adoptar un enfoque incremental para migrar al modelo de acceso recomendado y no intentar hacerlo de forma rápida o agresiva. Seguir un enfoque por fases para realizar el planeamiento, la migración, la validación y la retirada siguiendo una escala de tiempo razonable ayudará a evitar incidentes no planeado o un impacto inesperado en las operaciones en la nube. Si no tiene una directiva de retención de datos por motivos empresariales o de cumplimiento, debe evaluar el período adecuado durante el que se deben conservar los datos en el área de trabajo desde la que va a realizar la migración durante el proceso. Mientras reconfigura los recursos para informar al área de trabajo compartida, puede analizar los datos del área de trabajo original según sea necesario. Una vez completada la migración, si se deben conservar los datos en el área de trabajo original antes del final del período de retención, no los elimine.

Al planear la migración a este modelo, tenga en cuenta lo siguiente:

* Sepa qué normativas del sector y directivas internas relativas a la retención de datos debe cumplir.
* Asegúrese de que los equipos de aplicaciones pueden trabajar en la funcionalidad de contexto del recurso existente.
* Identifique el acceso concedido a los recursos de los equipos de aplicaciones y haga las pruebas necesarias en un entorno de desarrollo antes de realizar la implementación en producción.
* Configure el área de para habilitar **Usar permisos de recurso o de área de trabajo** .
* Retire a los equipos de aplicaciones el permiso para leer y consultar el área de trabajo.
* Habilite y configure las soluciones de supervisión, información como Azure Monitor para contenedores o Azure Monitor para VM, las cuentas de Automation y las soluciones de administración como Update Management, iniciar o detener máquinas virtuales, etc., que se implementaron en el área de trabajo original.

## <a name="next-steps"></a>Pasos siguientes

Para implementar los permisos y controles de seguridad que se recomiendan en esta guía, consulte el artículo relativo a la [administración del acceso a los registros](manage-access.md).

