---
title: Trabajar con la versión anterior de Azure Migrate
description: Se describe cómo trabajar con la versión anterior de Azure Migrate.
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: overview
ms.date: 11/19/2019
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: bb4cfcd48608f90898648450a20d246f9fde002b
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836010"
---
# <a name="work-with-the-previous-version-of-azure-migrate"></a>Trabajar con la versión anterior de Azure Migrate

En este artículo se proporciona información sobre cómo trabajar con la versión anterior de Azure Migrate. 


Existen dos versiones del servicio Azure Migrate:

- **Versión actual**: use esta versión para crear proyectos de Azure Migrate, detectar máquinas locales y organizar valoraciones y migraciones. [Obtenga más información](whats-new.md) sobre las novedades de esta versión.
- **Versión anterior**: Si usaba la versión anterior de Azure Migrate (solo se admitía la valoración de máquinas virtuales de VMware locales), ahora debería usar la versión actual. En este artículo, los proyectos de la versión anterior se denominan proyectos clásicos. Si todavía necesita usar proyectos de Azure Migrate creados en la versión anterior, esto es lo que puede y lo que no puede hacer:
    - Ya no puede crear proyectos de migración.
    - Recomendamos no realizar nuevas detecciones.
    - Todavía puede obtener acceso a los proyectos existentes.
    - Todavía puede ejecutar valoraciones.
    

## <a name="upgrade-between-versions"></a>Actualizar entre versiones

No se pueden actualizar proyectos ni componentes de la versión anterior a la nueva versión. Tiene que [crear un proyecto de Azure Migrate](create-manage-projects.md) y [agregarle herramientas de valoración y migración](how-to-add-tool-first-time.md). Use los tutoriales para aprender a usar las herramientas de valoración y migración disponibles. Si tiene un área de trabajo Log Analytics asociada a un proyecto clásico, puede adjuntarla a un proyecto de la versión actual después de eliminar el proyecto clásico.

## <a name="find-projects-from-previous-version"></a>Buscar proyectos desde una versión anterior

Busque proyectos de la versión anterior de la siguiente manera:

1. En Azure Portal > **Todos los servicios**, busque y seleccione **Azure Migrate**. 
2. En el panel de Azure Migrate, hay una notificación y un vínculo para acceder a proyectos antiguos de Azure Migrate.
3. Haga clic en el vínculo para abrir proyectos clásicos.

## <a name="delete-projects-from-previous-version"></a>Eliminación de proyectos de una versión anterior

Busque y elimine proyectos de la versión anterior de la siguiente manera:

1. En Azure Portal > **Todos los servicios**, busque y seleccione **Azure Migrate**. 
2. En el panel de Azure Migrate, hay una notificación y un vínculo para acceder a proyectos antiguos de Azure Migrate.
3. Haga clic en el vínculo para abrir proyectos clásicos.
4. Seleccione el proyecto que desea eliminar y elimínelo. 


## <a name="create-an-assessment"></a>Crear una evaluación

Después de detectar las máquinas virtuales en el portal, tiene que agruparlas y crear valoraciones.

- Podrá crear inmediatamente valoraciones como locales inmediatamente después de que las máquinas virtuales se detecten en el portal.
- Para valoraciones basadas en rendimiento, le recomendamos esperar al menos un día antes de crear una valoración basada en el rendimiento para que las recomendaciones de tamaño sean confiables.

Para crear una valoración, siga estos pasos:

1. En la página **Introducción** del proyecto, haga clic en **+Crear evaluación**.
2. Haga clic en **View all** (Ver todo) para revisar la configuración de la valoración.
3. Cree el grupo y especifique un nombre para él.
4. Seleccione las máquinas que desee agregar al grupo.
5. Haga clic en **Crear evaluación** para crear el grupo y la evaluación.
6. Una vez creada la evaluación, puede verla en **Introducción** > **Panel**.
7. Haga clic en **Exportar la evaluación** para descargarla como un archivo de Excel.

Si desea actualizar una evaluación existente con los datos de rendimiento más recientes, puede usar el comando **Recalcular** en la evaluación para actualizarla.

## <a name="review-an-assessment"></a>Revisión de una evaluación 

Una valoración tiene tres etapas:

- Una valoración comienza con un análisis de idoneidad para averiguar si los equipos son compatibles con Azure.
- Estimaciones de ajuste de tamaño.
- Estimación del costo mensual.

Una máquina solo se mueve a una fase posterior si aprueba la anterior. Por ejemplo, si no supera la comprobación de idoneidad, se marca como no adecuada para Azure, y el ajuste de tamaño y los costos no se calculan.


### <a name="review-azure-readiness"></a>Revisión de la preparación para Azure

La vista de preparación para Azure en la valoración muestra el estado de preparación de cada máquina virtual.

**Preparación** | **State** | **Detalles**
--- | --- | ---
Preparada para Azure | Sin problemas de compatibilidad. La máquina se puede migrar tal cual a Azure y arrancará en Azure con soporte técnico de Azure completo. | Para las VM que están preparadas, Azure Migrate recomienda un tamaño de VM en Azure.
Condicionalmente preparada para Azure | Es posible que el equipo arranque en Azure, pero puede que no tenga soporte técnico de Azure completo. Por ejemplo, una máquina con una versión anterior de Windows Server no se admite en Azure. | En Azure Migrate se explican los problemas de preparación y se proporcionan los pasos para corregirlos.
No está preparada para Azure |  La máquina virtual no arrancará en Azure. Por ejemplo, si una máquina virtual tiene un disco de más de 4 TB, no se puede hospedar en Azure. | En Azure Migrate se explican los problemas de preparación y se proporcionan los pasos para corregirlos.
Preparación desconocida | Azure Migrate no puede identificar la preparación para Azure, normalmente porque los datos no están disponibles.


#### <a name="azure-vm-properties"></a>Propiedades de máquinas virtuales de Azure
La preparación tiene en cuenta varias propiedades de máquina virtual para identificar si la máquina virtual puede ejecutarse en Azure.


**Propiedad** | **Detalles** | **Preparación**
--- | --- | ---
**Tipo de arranque** | BIOS compatible. UEFI no compatible. | Condicionalmente preparada si el tipo de arranque es UEFI.
**Núcleos** | Núcleos de las máquinas <= número máximo de núcleos (128) admitidos para una máquina virtual de Azure.<br/><br/> Si hay algún historial de rendimiento disponible, Azure Migrate tiene en cuenta los núcleos usados.<br/>Si se especifica un factor de confort en la configuración de evaluación, el número de núcleos utilizados se multiplica por dicho factor de confort.<br/><br/> Si no hay ningún historial de rendimiento, Azure Migrate usa los núcleos asignados sin aplicar el factor de confort. | Preparada si es menor o igual a los límites.
**Memoria** | Tamaño de memoria de la máquina <= memoria máxima (3892 GB en Azure M serie Standard_M128m&nbsp;<sup>2</sup>) para una máquina virtual de Azure. [Más información](../virtual-machines/sizes.md).<br/><br/> Si hay algún historial de rendimiento disponible, Azure Migrate tiene en cuenta la memoria usada.<br/><br/>Si se especifica un factor de confort, la memoria utilizada se multiplica por dicho factor de confort.<br/><br/> Si no hay ningún historial, se usa la memoria asignada sin aplicar el factor de confort.<br/><br/> | Preparada si está dentro de los límites.
**Disco de almacenamiento** | El tamaño asignado de un disco debe ser de 4 TB (4096 GB) o menos.<br/><br/> El número de discos conectados a la máquina debe ser 65 o menos, incluido el disco de sistema operativo. | Preparada si está dentro de los límites.
**Redes** | Una máquina debe tener 32 NIC o menos conectadas. | Preparada si está dentro de los límites.

#### <a name="guest-operating-system"></a>Sistema operativo invitado

Además de las propiedades de la máquina virtual, Azure Migrate también examina el sistema operativo invitado de la máquina virtual local para identificar si se puede ejecutar en Azure.

- Azure Migrate tiene en cuenta el sistema operativo especificado en vCenter Server.
- Puesto que la detección realizada por Azure Migrate está basada en el dispositivo, no hay manera de comprobar si el sistema operativo que se ejecuta dentro de la máquina virtual es el mismo que el especificado en vCenter Server.

Se usa la siguiente lógica.

**Sistema operativo** | **Detalles** | **Preparación**
--- | --- | ---
Windows Server 2016 y todos los Service Pack | Azure proporciona compatibilidad completa. | Preparada para Azure
Windows Server 2012 R2 y todos los Service Pack | Azure proporciona compatibilidad completa. | Preparada para Azure
Windows Server 2012 y todos los Service Pack | Azure proporciona compatibilidad completa. | Preparada para Azure
Windows Server 2008 R2 y todos los Service Pack | Azure proporciona compatibilidad completa.| Preparada para Azure
Windows Server 2008 (32 bits y 64 bits) | Azure proporciona compatibilidad completa. | Preparada para Azure
Windows Server 2003 y 2003 R2 | Fuera de soporte técnico y necesita un [contrato de soporte técnico personalizado (CSA)](https://aka.ms/WSosstatement) para la compatibilidad con Azure. | Condicionalmente preparada para Azure, considere la posibilidad de actualizar el sistema operativo antes de migrar a Azure.
Windows 2000, 98, 95, NT, 3.1, MS-DOS | Fuera de soporte técnico. La máquina puede arrancar en Azure, pero Azure no proporciona ningún sistema operativo. | Condicionalmente preparada para Azure, se recomienda actualizar el sistema operativo antes de migrar a Azure.
Cliente de Windows 7, 8 y 10 | Azure solo proporciona compatibilidad con la [suscripción de Visual Studio](../virtual-machines/windows/client-images.md). | Condicionalmente preparada para Azure
Windows 10 Pro Desktop | Azure proporciona compatibilidad con [derechos de hospedaje multiinquilino.](../virtual-machines/windows/windows-desktop-multitenant-hosting-deployment.md) | Condicionalmente preparada para Azure
Windows Vista, XP Professional | Fuera de soporte técnico. La máquina puede arrancar en Azure, pero Azure no proporciona ningún sistema operativo. | Condicionalmente preparada para Azure, se recomienda actualizar el sistema operativo antes de migrar a Azure.
Linux | Azure aprueba estos [sistemas operativos Linux](../virtual-machines/linux/endorsed-distros.md). Otros sistemas operativos Linux pueden arrancar en Azure, pero recomendamos actualizar el sistema operativo a una versión aprobada antes de migrar a Azure. | Preparada para Azure si la versión está aprobada.<br/><br/>Condicionalmente preparada si la versión no se ha aprobado.
Otros sistemas operativos<br/><br/> Por ejemplo, Oracle Solaris, Apple macOS, FreeBSD, etc. | Azure no aprueba estos sistemas operativos. La máquina puede arrancar en Azure, pero Azure no proporciona ningún sistema operativo. | Condicionalmente preparada para Azure, se recomienda instalar un sistema operativo compatible antes de migrar a Azure.  
Sistema operativo especificado como **otro** en vCenter Server. | Azure Migrate no puede identificar el sistema operativo en este caso. | Preparación desconocida. Asegúrese de que el sistema operativo que se ejecuta dentro de la máquina virtual se admite en Azure.
Sistemas operativos de 32 bits | La máquina puede arrancar en Azure, pero Azure no puede proporcionar soporte técnico completo. | Condicionalmente preparada para Azure, considere la posibilidad de actualizar el sistema operativo de la máquina de un sistema operativo de 32 bits a uno de 64 bits antes de migrar a Azure.


### <a name="review-sizing"></a>Revisar el tamaño

 La recomendación de tamaño de Azure Migrate depende del criterio de tamaño que se especifica en las propiedades de la valoración.

- Si el ajuste de tamaño está basado en el rendimiento, se realiza la recomendación de tamaño teniendo en cuenta el historial de rendimiento de las máquinas virtuales (CPU y memoria) y discos (IOPS y rendimiento).
- Si el criterio de ajuste de tamaño es "como local", se realiza la recomendación de tamaño en Azure en función del tamaño de la máquina virtual local. El ajuste de tamaño del disco se basa en el tipo de almacenamiento especificado en las propiedades de valoración (el valor predeterminado es discos Premium). Azure Migrate no tiene en cuenta los datos de rendimiento de la máquina virtual y los discos.

### <a name="review-cost-estimates"></a>Revisar estimaciones de costos

Las estimaciones de costos muestran el cálculo total y el costo de almacenamiento por ejecutar las máquinas virtuales en Azure, además de los detalles de cada máquina.

- Las estimaciones de costos se calculan con las recomendaciones de tamaño para una máquina virtual, sus discos y las propiedades de valoración.
- Los costos mensuales estimados para el proceso y almacenamiento se agregan para todas las VM del grupo.
- La estimación de costos es por ejecutar la máquina virtual local como máquinas virtuales de infraestructura como servicio (IaaS) de Azure. Azure Migrate no tiene en cuenta los costos de cualquier plataforma como servicio (PaaS) o software como servicio (SaaS).

### <a name="review-confidence-rating-performance-based-assessment"></a>Revisar la clasificación de confianza (valoración basada en el rendimiento)

Cada valoración basada en el rendimiento está asociada a una clasificación de confianza.

- La clasificación de confianza oscila entre una estrella y cinco estrellas (una estrella es la más baja y cinco la más alta).
- La clasificación de confianza se asigna a una valoración, que se basa en la disponibilidad de puntos de datos necesarios para calcular tal valoración.
- La clasificación de confianza de una valoración le ayuda a calcular la confiabilidad de las recomendaciones de tamaño que proporciona Azure Migrate.
- La clasificación de confianza no está disponible para aplicarse "tal cual" a valoraciones locales.

Para ajustes de tamaño basados en el rendimiento, Azure Migrate necesita lo siguiente:
- Datos de uso de la CPU.
- Datos de memoria de la máquina virtual.
- Para cada disco asociado a la máquina virtual, necesita el valor de IOPS de disco y los datos de rendimiento.
- Para cada adaptador de red asociado a una máquina virtual, Azure Migrate necesita la entrada o la salida de red.
- Si alguno de los elementos anteriores no está disponible, es posible que las recomendaciones de tamaño (y, por tanto, las clasificaciones de confianza) no sean confiables.


Según el porcentaje de puntos de datos disponibles, se resumen las clasificaciones de confianza posibles en la tabla.

**Disponibilidad de puntos de datos** | **Clasificación de confianza**
--- | ---
0 % - 20 % | 1 estrella
21 % - 40 % | 2 estrellas
41 % - 60 % | 3 estrellas
61 % - 80 % | 4 estrellas
81 % - 100 % | 5 estrellas


#### <a name="assessment-issues-affecting-confidence-ratings"></a>Problemas de valoración que afectan a las clasificaciones de confianza

Puede que las valoraciones no tengan todos los puntos de datos disponibles por varios motivos:

- No perfiló su entorno para la duración de la valoración. Por ejemplo, si crea la valoración con la duración de rendimiento establecida en un día, debe esperar al menos un día después de empezar la detección para que se recopilen todos los puntos de datos.
- Se apagaron algunas máquinas virtuales en el período durante el que se calculó la valoración. Si alguna máquina virtual se apagó por algún tiempo, Azure Migrate no puede recopilar los datos de rendimiento durante ese período.
- Algunas máquinas virtuales se crearon durante el período de cálculo de la valoración. Por ejemplo, si crea una valoración con el historial de rendimiento del último mes, pero creó un número de máquinas virtuales en el entorno hace una semana, el historial de rendimiento de las nuevas máquinas virtuales no será para toda la duración.

> [!NOTE]
> Si la clasificación de confianza de una valoración se sitúa por debajo de las cinco estrellas, espere al menos un día para que el dispositivo genere el perfil del entorno y luego recalcule la valoración. Si no se realiza el ajuste de tamaño basado en el rendimiento, es posible que no sea confiable. Si no quiere recalcular, recomendamos cambiar a ajuste de tamaño local. Para ello, cambie las propiedades de valoración.



## <a name="create-groups-using-dependency-visualization"></a>Crear grupos con la visualización de dependencias

Además de crear grupos manualmente, puede crear grupos con la visualización de dependencias.
- Este método se utiliza normalmente cuando se quiere valorar grupos con mayores niveles de confianza mediante la comprobación cruzada de dependencias de máquina antes de ejecutar una valoración.
- La visualización de dependencia puede ayudarle a planear la migración a Azure de forma eficaz. Le ayuda a asegurarse de que no se deja nada atrás y de que no se produzcan interrupciones inesperadas al realizar una migración a Azure.
- Puede detectar todos los sistemas interdependientes que tienen que migrarse juntos e identificar si un sistema en ejecución sigue ofreciendo servicio a los usuarios o si es un candidato para la retirada en lugar de la migración.
- Azure Migrate usa la solución Service Map en Azure Monitor para habilitar la visualización de dependencias.

> [!NOTE]
> La visualización de dependencias no está disponible en Azure Government.

Para configurar la visualización de dependencias, debe asociar un área de trabajo de Log Analytics a un proyecto de Azure Migrate, instalar agentes en las máquinas para los que quiere visualizar las dependencias y, a continuación, crear grupos con la información de dependencias. 



### <a name="associate-a-log-analytics-workspace"></a>Asociar un área de trabajo de Log Analytics

Para usar la visualización de dependencias, debe asociar un área de trabajo de Log Analytics a un proyecto de migración. Solo se puede crear o vincular un área de trabajo en la misma suscripción donde se crea el proyecto de migración.

1. Para asociar un área de trabajo de Log Analytics a un proyecto, en **Introducción**, vaya a la sección **Essentials** del proyecto y haga clic en **Requiere configuración**.
2. Puede crear una nueva área de trabajo o adjuntar una existente:
  - Para crear una nueva área de trabajo, especifique un nombre. El área de trabajo se crea en una región en la misma [ubicación geográfica de Azure](https://azure.microsoft.com/global-infrastructure/geographies/) que el proyecto de migración.
  - Al asociar un área de trabajo existente, puede elegir entre las disponibles en la misma suscripción del proyecto de migración. Solo se enumeran las áreas de trabajo que se crearon en una [región donde se admite Service Map](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions). Para asociar un área de trabajo, asegúrese de que tiene acceso de lectura a ella.

> [!NOTE]
> No se puede cambiar el área de trabajo asociada a un proyecto de migración.

### <a name="download-and-install-vm-agents"></a>Descargar e instalar agentes de máquinas virtuales

Después de configurar un área de trabajo, descargue e instale agentes en cada máquina local que vaya a valorar. Además, si tiene máquinas sin conectividad a Internet, debe descargar e instalar en ellas la [puerta de enlace de Log Analytics](../azure-monitor/platform/gateway.md).

1. En **Introducción**, haga clic en **Administrar** > **Máquinas**y seleccione la máquina requerida.
2. En la columna **Dependencias**, haga clic en **Instalar agentes**.
3. En la página **Dependencias**, descargue e instale el agente de Microsoft Monitoring Agent (MMA), así como el agente de dependencia en cada máquina virtual que vaya a valorar.
4. Copie la clave y el identificador de área de trabajo. Las necesitará cuando se instala MMA en la máquina local.

> [!NOTE]
> Para automatizar la instalación de agentes, puede usar cualquier herramienta de implementación, como Configuration Manager, o bien la herramienta de alguno de nuestros asociados, como [Intigua](https://www.intigua.com/intigua-for-azure-migration), que proporciona una solución de implementación de agentes para Azure Migrate.


#### <a name="install-the-mma-agent-on-a-windows-machine"></a>Instalar el agente MMA en una máquina Windows

Para instalar al agente en una máquina Windows, siga estos pasos:

1. Haga doble clic en el agente descargado.
2. En la página **principal**, haga clic en **Siguiente**. En la página **Términos de licencia**, haga clic en **Acepto** para aceptar la licencia.
3. En **Carpeta de destino**, mantenga o modifique la carpeta de instalación predeterminada y después haga clic en **Siguiente**.
4. En **Opciones de instalación del agente**, seleccione **Azure Log Analytics** > **Siguiente**.
5. Haga clic en **Agregar** para agregar un área de trabajo de Log Analytics nueva. Pegue la clave y el identificador de área de trabajo que ha copiado desde el portal. Haga clic en **Next**.

Puede instalar al agente desde la línea de comandos o mediante un método automatizado, como Configuration Manager. [Obtenga más información](../azure-monitor/platform/log-analytics-agent.md#installation-and-configuration) sobre el uso de estos métodos para instalar el agente MMA.

#### <a name="install-the-mma-agent-on-a-linux-machine"></a>Instalar el agente MMA en una máquina Linux

Para instalar al agente en una máquina Linux, siga estos pasos:

1. Transfiera el paquete adecuado (x86 o x x64) al equipo Linux mediante scp o sftp.
2. Instale el paquete mediante el argumento --install.

    ```sudo sh ./omsagent-<version>.universal.x64.sh --install -w <workspace id> -s <workspace key>```

[Más información](../azure-monitor/platform/log-analytics-agent.md#supported-linux-operating-systems) sobre la lista de compatibilidad de MMA con sistemas operativos Linux.

### <a name="install-the-mma-agent-on-a-machine-monitored-by-operations-manager"></a>Instalar el agente MMA en una máquina supervisada por Operations Manager

Para las máquinas supervisadas por System Center Operations Manager 2012 R2 o versiones posteriores, no hay necesidad de instalar el agente MMA. Service Map se integra con el MMA de Operations Manager para recopilar los datos de dependencia necesarios. [Más información](../azure-monitor/insights/service-map-scom.md#prerequisites). No es necesario instalar el agente de dependencia.

### <a name="install-the-dependency-agent"></a>Instalación del agente de dependencia

1. Para instalar al agente de dependencia en una máquina Windows, haga doble clic en el archivo de instalación y siga los pasos del asistente.
2. Para instalar el agente de dependencia en una máquina Linux, instale como raíz mediante el siguiente comando:

    ```sh InstallDependencyAgent-Linux64.bin```

- Obtenga más información sobre la [compatibilidad de Dependency Agent](../azure-monitor/insights/vminsights-enable-overview.md#supported-operating-systems) para los sistemas operativos Windows y Linux.
- [Más información](../azure-monitor/insights/vminsights-enable-hybrid.md#dependency-agent) acerca de cómo puede utilizar scripts para instalar el agente de dependencia.

>[!NOTE]
> El artículo Azure Monitor para VM al que se hace referencia para proporcionar información general sobre los requisitos previos del sistema y los métodos para implementar el agente de dependencia también son aplicables a la solución Service Map.

### <a name="create-a-group-with-dependency-mapping"></a>Crear un grupo con asignación de dependencias

1. Después de instalar los agentes, vaya al portal y haga clic en **Administrar** > **Máquinas**.
2. Busque la máquina en que instaló los agentes.
3. La columna **Dependencias** de la máquina debería aparecer ahora como **Ver dependencias**. Haga clic en la columna para ver las dependencias de la máquina.
4. En el mapa de dependencias de la máquina aparecen los siguientes detalles:
    - Conexiones TCP de entrada (clientes) y de salida (servidores) hacia y desde la máquina
        - Las máquinas dependientes que no tienen instalado el agente MMA y de dependencias se agrupan en función de los números de puerto.
        - Las máquinas dependientes que tienen instalado el agente MMA y de dependencias se muestran como cuadros diferentes.
    - Procesos que se ejecutan dentro de la máquina; aquí puede expandir cada casilla de la máquina para ver los procesos
    - Se muestran las propiedades de la máquina, incluido el FQDN, el sistema operativo y la dirección MAC. Puede hacer clic en el cuadro de cada máquina para ver los detalles.

4. Puede ver las dependencias de distintas duraciones; para ello, haga clic en la duración correspondiente en la etiqueta de intervalo de tiempo. De forma predeterminada, el intervalo es una hora. Puede modificar el intervalo de tiempo o especificar las fechas de inicio y finalización, y la duración.

   > [!NOTE]
   >    Se admite un intervalo de tiempo de hasta una hora. Use los registros de Azure Monitor para [consultar datos de dependencia](./how-to-create-group-machine-dependencies.md) durante un período más largo.

5. Después de identificar máquinas dependientes que desea agrupar, use Ctrl+clic para seleccionar varias máquinas en el mapa y haga clic en **Agrupar máquinas**.
6. Especifique un nombre para el grupo. Verifique que Azure Migrate puede detectar las máquinas dependientes.

    > [!NOTE]
    > Si Azure Migrate no detecta una máquina dependiente, no puede agregarla al grupo. Para agregar estas máquinas al grupo, debe volver a ejecutar el proceso de detección con el ámbito correcto en la instancia de vCenter Server y asegurarse de que Azure Migrate detecta la máquina.  

7. Si desea crear una evaluación para este grupo, seleccione la casilla para crear una evaluación para el grupo.
8. Haga clic en **Aceptar** para guardar el grupo.

Una vez creado el grupo, se recomienda instalar los agentes en todas las máquinas del grupo y restringir el grupo mediante la visualización de la dependencia de todo el grupo.

## <a name="query-dependency-data-from-azure-monitor-logs"></a>Consulta de datos de dependencia de los registros de Azure Monitor

Los datos de dependencia capturados por Service Map están disponibles para su consulta en el área de trabajo de Log Analytics asociado con el proyecto de Azure Migrate. [Obtenga más información](../azure-monitor/insights/service-map.md#log-analytics-records) sobre las tablas de datos de Service Map que puede consultar en los registros de Azure Monitor. 

Para ejecutar consultas de Kusto:

1. Después de instalar los agentes, vaya al portal y haga clic en **Introducción**.
2. En **Introducción**, vaya a la sección **Essentials** del proyecto y haga clic en el nombre del área de trabajo que se proporciona junto al **Área de trabajo de OMS**.
3. En la página del área de trabajo de Log Analytics, haga clic en **General** > **Registros**.
4. Escriba la consulta para recopilar los datos de dependencia mediante los registros de Azure Monitor. Busque consultas de ejemplo en la sección siguiente.
5. Ejecute la consulta haciendo clic en Ejecutar. 

[Más información](../azure-monitor/log-query/get-started-portal.md) sobre la escritura de consultas de Kusto. 

### <a name="sample-azure-monitor-logs-queries"></a>Consultas de registros de Azure Monitor de ejemplo

A continuación se muestran consultas de ejemplo que puede utilizar para extraer datos de dependencia. Puede modificar las consultas para extraer los puntos de datos preferidos. [Aquí](../azure-monitor/insights/service-map.md#log-analytics-records) hay disponible una lista exhaustiva de los campos de registros de datos de dependencia. Busque más consultas de ejemplo [aquí](../azure-monitor/insights/service-map.md#sample-log-searches).

#### <a name="summarize-inbound-connections-on-a-set-of-machines"></a>Resumen de las conexiones entrantes en un conjunto de máquinas

Los registros de la tabla de métricas de conexión, VMConnection, no representan conexiones de red físicas individuales. Varias conexiones de red físicas se agrupan en una conexión lógica. [Obtenga más información](../azure-monitor/insights/service-map.md#connections) sobre cómo se agregan los datos de conexión de red física en un solo registro lógico en VMConnection. 

```
// the machines of interest
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
let StartDateTime = datetime(2019-03-25T00:00:00Z);
let EndDateTime = datetime(2019-03-30T01:00:00Z); 
VMConnection
| where Direction == 'inbound' 
| where TimeGenerated > StartDateTime and TimeGenerated  < EndDateTime
| join kind=inner (ips) on $left.DestinationIp == $right.ips
| summarize sum(LinksEstablished) by Computer, Direction, SourceIp, DestinationIp, DestinationPort
```

#### <a name="summarize-volume-of-data-sent-and-received-on-inbound-connections-between-a-set-of-machines"></a>Resumen del volumen de datos enviados y recibidos en conexiones entrantes entre un conjunto de máquinas

```
// the machines of interest
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
let StartDateTime = datetime(2019-03-25T00:00:00Z);
let EndDateTime = datetime(2019-03-30T01:00:00Z); 
VMConnection
| where Direction == 'inbound' 
| where TimeGenerated > StartDateTime and TimeGenerated  < EndDateTime
| join kind=inner (ips) on $left.DestinationIp == $right.ips
| summarize sum(BytesSent), sum(BytesReceived) by Computer, Direction, SourceIp, DestinationIp, DestinationPort
```


## <a name="next-steps"></a>Pasos siguientes
[Obtenga más información](migrate-services-overview.md) sobre cómo instalar la versión más reciente de Azure Migrate.