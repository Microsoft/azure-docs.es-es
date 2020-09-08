---
title: Migración de máquinas virtuales de VMware sin agente mediante la migración de servidores de Azure Migrate
description: Aprenda a ejecutar una migración de máquinas virtuales de VMware sin agentes con Azure Migrate.
ms.topic: tutorial
ms.date: 06/09/2020
ms.custom: mvc
ms.openlocfilehash: c9a7e76496c2851b90c8fd1d164645c46d02609b
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89376047"
---
# <a name="migrate-vmware-vms-to-azure-agentless"></a>Migración de máquinas virtuales de VMware a Azure (sin agente)

En este artículo se muestra cómo migrar máquinas virtuales VMware locales a Azure mediante la migración sin agente con la herramienta [Azure Migrate:Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool). También puede migrar máquinas virtuales VMware mediante la migración basada en agente. [Compare](server-migrate-overview.md#compare-migration-methods) los métodos.

Este tutorial es el tercero de una serie que muestra cómo evaluar máquinas virtuales VMware y migrarlas a Azure. 

> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles. 


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar la herramienta Azure Migrate:Server Migration.
> * Detectar las máquinas virtuales que desea migrar.
> * Iniciar la replicación de las máquinas virtuales.
> * Ejecute una migración de prueba para asegurarse de que todo funciona de la forma esperada.
> * Ejecutar una migración completa de la máquina virtual.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, debe:

1. [Complete el primer tutorial](tutorial-prepare-vmware.md) para preparar Azure y VMware para la migración.
2. Se recomienda completar el segundo tutorial para [evaluar las máquinas virtuales VMware](tutorial-assess-vmware.md) antes de migrarlas a Azure, pero no es obligatorio. 


## <a name="add-the-azure-migrate-server-migration-tool"></a>Incorporación de la herramienta Azure Migrate Server Migration

Si aún no ha configurado un proyecto de Azure Migrate, [hágalo](how-to-add-tool-first-time.md) antes de agregar la herramienta. Si tiene un proyecto configurado, agregue la herramienta de la siguiente manera:

1. En el proyecto de Azure Migrate, haga clic en **Información general**. 
2. En **Detectar, evaluar y migrar servidores**, haga clic en **Evaluar y migrar servidores**.

     ![Evaluar y migrar servidores](./media/tutorial-migrate-vmware/assess-migrate.png)

3. En **Herramientas de migración** , seleccione **Click here to add a migration tool when you are ready to migrate** (Haga clic aquí para agregar una herramienta de migración cuando esté listo para migrar).

    ![Seleccionar una herramienta](./media/tutorial-migrate-vmware/select-migration-tool.png)

4. En la lista de herramientas, seleccione **Azure Migrate: Server Migration** > **Agregar herramienta**.

    ![Herramienta Server Migration](./media/tutorial-migrate-vmware/server-migration-tool.png)

## <a name="set-up-the-azure-migrate-appliance"></a>Configuración del dispositivo de Azure Migrate

Azure Migrate Server Migration ejecuta un dispositivo de máquina virtual VMware ligero que se usa para la detección, evaluación y migración sin agente de máquinas virtuales VMware. Si ha seguido el [tutorial de evaluación](tutorial-assess-vmware.md), ya ha configurado el dispositivo. Si no lo ha hecho, configúrelo ahora mediante uno de estos métodos:

- **Plantilla OVA**: [Configúrelo](how-to-set-up-appliance-vmware.md) en una máquina virtual de VMware mediante una plantilla de OVA descargada.
- **Script**: [configúrelo](deploy-appliance-script.md) en una máquina virtual VMware o en una máquina física con un script del instalador de PowerShell. Este método se debe usar si no se puede configurar una máquina virtual mediante una plantilla OVA o si trabaja con Azure Government.

Una vez creada la aplicación, compruebe que se puede conectar a Azure Migrate:Server Assessment, configúrela por primera vez y regístrela en el proyecto de Azure Migrate.

## <a name="replicate-vms"></a>Replicación de máquinas virtuales

Después de configurar el dispositivo y completar la detección, puede iniciar la replicación de las máquinas virtuales VMware en Azure. 

- Puede ejecutar hasta 300 replicaciones simultáneamente.
- En el portal puede seleccionar hasta 10 máquinas virtuales a la vez para la migración. Para migrar más máquinas, agrúpelas en lotes de 10.

Habilite la replicación como se indica a continuación:

1. En el proyecto de Azure Migrate > **Servidores**, **Azure Migrate: Migración del servidor**, haga clic en **Replicar**.

    ![Replicación de máquinas virtuales](./media/tutorial-migrate-vmware/select-replicate.png)

2. En **Replicar** > **Configuración de origen** >  **¿Las máquinas están virtualizadas?** , seleccione **Sí, con VMware vSphere Hypervisor**.
3. En **Dispositivo local**, seleccione el nombre del dispositivo de Azure Migrate que configuró > **OK**. 

    ![Configuración de origen](./media/tutorial-migrate-vmware/source-settings.png)

4. En **Máquinas virtuales**, seleccione las máquinas que desea replicar. Para aplicar el tamaño de la máquina virtual y el tipo de disco de una evaluación si ha ejecutado una, en **¿Importar la configuración de migración de una evaluación de Azure Migrate?** , seleccione **Sí** y seleccione el grupo de máquinas virtuales y el nombre de la evaluación. Si no usa la configuración de evaluación, seleccione **No**.
   
    ![Seleccionar evaluación](./media/tutorial-migrate-vmware/select-assessment.png)

5. En **Máquinas virtuales**, seleccione las máquinas virtuales que desea migrar. A continuación, haga clic en **Siguiente: Configuración de destino**.

    ![Selección de las máquinas virtuales](./media/tutorial-migrate-vmware/select-vms.png)

6. En **Configuración de destino**, seleccione la suscripción y la región de destino. Especifique el grupo de recursos en el que residen las máquinas virtuales de Azure después de la migración.
7. En **Red virtual**, seleccione la red virtual de Azure y la subred a la que se unirán las máquinas virtuales de Azure después de la migración.
8. En **Opciones de disponibilidad**, seleccione:
    -  La zona de disponibilidad para anclar la máquina migrada a una zona de disponibilidad específica de la región. Use esta opción para distribuir los servidores que forman una capa de aplicación de varios nodos en Availability Zones. Si selecciona esta opción, deberá especificar la zona de disponibilidad que se va a usar en cada una de las máquinas seleccionadas en la pestaña Proceso. Esta opción solo está disponible si la región de destino seleccionada para la migración admite Availability Zones.
    -  El conjunto de disponibilidad para colocar la máquina migrada en un conjunto de disponibilidad. Para usar esta opción, el grupo de recursos de destino seleccionado debe tener uno o varios conjuntos de disponibilidad.
    - No se requiere ninguna opción de redundancia de infraestructura si no necesita ninguna de estas configuraciones de disponibilidad para las máquinas migradas.

9. En **Ventaja híbrida de Azure**:

    - Seleccione **No** si no desea aplicar la Ventaja híbrida de Azure. A continuación, haga clic en **Siguiente**.
    - Seleccione **Sí** si tiene equipos con Windows Server que están incluidos en suscripciones activas de Software Assurance o Windows Server y desea aplicar el beneficio a las máquinas que va a migrar. A continuación, haga clic en **Siguiente**.

    ![Configuración de destino](./media/tutorial-migrate-vmware/target-settings.png)

10. En **Proceso**, revise el nombre, el tamaño, el tipo de disco del sistema operativo y la configuración de disponibilidad (si se ha seleccionado en el paso anterior) de la máquina virtual. Las máquinas virtuales deben cumplir los [requisitos de Azure](migrate-support-matrix-vmware-migration.md#azure-vm-requirements).

    - **Tamaño de VM**: si usa las recomendaciones de la evaluación, el menú desplegable de tamaño de máquina virtual muestra el tamaño recomendado. De lo contrario, Azure Migrate elige un tamaño en función de la coincidencia más cercana en la suscripción de Azure. También puede elegir un tamaño de manera manual en **Tamaño de la máquina virtual de Azure**. 
    - **Disco del sistema operativo**: especifique el disco del sistema operativo (arranque) de la máquina virtual. Este es el disco que tiene el cargador de arranque y el instalador del sistema operativo. 
    - **Zona de disponibilidad**: especifique la zona de disponibilidad que se va a usar.
    - **Conjunto de disponibilidad**: especifique el conjunto de disponibilidad que se va a usar.

> [!NOTE]
>Si quiere seleccionar otra opción de disponibilidad para un conjunto de máquinas virtuales, vaya al paso 1 y repita los pasos seleccionando diferentes opciones de disponibilidad tras el inicio de la replicación de uno de los conjuntos de máquinas virtuales.


 ![Configuración de los recursos de proceso de la máquina virtual](./media/tutorial-migrate-vmware/compute-settings.png)

11. En **Discos**, especifique si los discos de la máquina virtual se deben replicar en Azure y seleccione el tipo de disco (discos SSD o HDD estándar, o bien discos administrados prémium) en Azure. A continuación, haga clic en **Siguiente**.
   
    ![Discos](./media/tutorial-migrate-vmware/disks.png)

12. En **Revisar e iniciar la replicación**, revise la configuración y haga clic en **Replicar** para iniciar la replicación inicial de los servidores.

> [!NOTE]
> Puede actualizar la configuración de replicación en cualquier momento antes de que esta comience (**Administrar** > **Replicación de máquinas**). No se puede cambiar la configuración después de iniciar la replicación.

### <a name="provisioning-for-the-first-time"></a>Primer aprovisionamiento

Si se trata de la primera máquina virtual que va a replicar en el proyecto, Server Migration aprovisiona automáticamente estos recursos en el mismo grupo de recursos que el proyecto.

- **Service Bus**: Server Migration usa Service Bus para enviar mensajes de orquestación de replicación al dispositivo.
- **Cuenta de almacenamiento de puerta de enlace**: Server Migration usa la cuenta de almacenamiento de puerta de enlace para almacenar información del estado de las máquinas virtuales que se replican.
- **Cuenta de almacenamiento de registros**: el dispositivo con Azure Migrate carga los registros de replicación de las máquinas virtuales en una cuenta de almacenamiento de registros. Azure Migrate aplica la información de replicación a los discos administrados de réplica.
- **Almacén de claves**: el dispositivo con Azure Migrate usa el almacén de claves para administrar las cadenas de conexión de Service Bus y las claves de acceso de las cuentas de almacenamiento utilizadas en la replicación.

## <a name="track-and-monitor"></a>Seguimiento y supervisión

1. Realice un seguimiento del estado del trabajo en las notificaciones del portal.
2. Para supervisar el estado de la replicación, haga clic en **Replicando servidores** en **Azure Migrate: Server Migration**.

     ![Supervisión de la replicación](./media/tutorial-migrate-vmware/replicating-servers.png)

La replicación se produce de la manera siguiente:
- Cuando el trabajo de inicio de replicación finaliza correctamente, las máquinas comienzan su replicación inicial en Azure.
- Durante la replicación inicial, se crea una instantánea de la máquina virtual. Los datos de disco de la instantánea se replican en los discos administrados de réplica en Azure.
- Cuando finaliza esta replicación inicial, comienza la replicación diferencial. Los cambios incrementales de los discos locales se replican periódicamente en los discos de réplica de Azure.

## <a name="run-a-test-migration"></a>Ejecutar una migración de prueba


Cuando comienza la replicación diferencial, puede ejecutar una migración de prueba para las máquinas virtuales antes de ejecutar una migración completa a Azure. Le recomendamos encarecidamente que lo haga al menos una vez en cada máquina, antes de migrarla.

- La ejecución de una migración de prueba comprueba que la migración funcionará según lo previsto, sin afectar a las máquinas locales, que seguirán estando operativas y continuarán realizando la replicación. 
- Para simular la migración, la migración de prueba crea una máquina virtual de Azure usando datos replicados (normalmente, con una migración a una red virtual que no es de producción en la suscripción a Azure).
- Puede usar la máquina virtual de Azure de prueba replicada para validar la migración, realizar pruebas de aplicaciones y resolver los problemas antes de la migración completa.

Realice una migración de prueba como se indica a continuación:


1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Migration**, haga clic en **Probar servidores migrados**.

     ![Probar servidores migrados](./media/tutorial-migrate-vmware/test-migrated-servers.png)

2. Haga clic con el botón derecho en la máquina virtual que va a probar y haga clic en **Migración de prueba**.

    ![Migración de prueba](./media/tutorial-migrate-vmware/test-migrate.png)

3. En **Migración de prueba**, seleccione la red virtual de Azure en la que se ubicará la máquina virtual de Azure después de la migración. Se recomienda usar una red virtual que no sea de producción.
4. Comienza el trabajo de **Migración de prueba**. Supervise el trabajo en las notificaciones del portal.
5. Una vez finalizada la migración, la máquina virtual de Azure migrada se puede ver en **Máquinas virtuales** en Azure Portal. El nombre de la máquina tiene el sufijo **-Test**.
6. Una vez finalizada la prueba, haga clic con el botón derecho en la máquina virtual de Azure, en **Replicación de máquinas**, y haga clic en **Limpiar la migración de prueba**.

    ![Limpiar la migración](./media/tutorial-migrate-vmware/clean-up.png)


## <a name="migrate-vms"></a>Migración de máquinas virtuales

Después de comprobar que la migración de prueba funciona según lo previsto, puede migrar las máquinas locales.

1. En el proyecto de Azure Migrate > **Servidores** > **Azure Migrate: Server Migration**, haga clic en **Replicando servidores**.

    ![Replicando servidores](./media/tutorial-migrate-vmware/replicate-servers.png)

2. En **Replicación de máquinas**, haga clic con el botón derecho en la máquina virtual > **Migrar**.
3. En **Migrar** >  **¿Quiere apagar las máquinas virtuales y realizar una migración planificada sin perder datos?** , seleccione **Sí** > **Aceptar**.
    - De forma predeterminada, Azure Migrate apaga la máquina virtual local y ejecuta una replicación a petición para sincronizar los cambios que se han producido en la máquina virtual desde la última replicación. De esta forma se garantiza que no se pierden datos.
    - Si no desea apagar la máquina virtual, seleccione **No**
4. Se inicia un trabajo de migración de la máquina virtual. Realice un seguimiento del trabajo en las notificaciones de Azure.
5. Una vez finalizado el trabajo, la máquina virtual puede ver y administrar desde la página **Máquinas virtuales**.

## <a name="complete-the-migration"></a>Completar la migración

1. Una vez finalizada la migración, haga clic con el botón derecho en la máquina virtual > **Detener replicación**. Así se detiene la replicación en la máquina local y se limpia la información acerca del estado de replicación de la máquina virtual.
2. Instale el agente de [Windows](../virtual-machines/extensions/agent-windows.md) o [Linux](../virtual-machines/extensions/agent-linux.md) de la máquina virtual de Azure en las máquinas migradas.
3. Realice los ajustes de la aplicación posteriores a la migración, como actualizar las cadenas de conexión de la base de datos y las configuraciones del servidor web.
4. Realice las pruebas finales de la aplicación y la aceptación de la migración en la aplicación migrada que ahora se ejecuta en Azure.
5. Pase el tráfico a la instancia de máquina virtual de Azure migrada.
6. Quite las máquinas virtuales locales del inventario de máquinas virtuales local.
7. Quite las máquinas virtuales locales de las copias de seguridad locales.
8. Actualice la documentación interna para mostrar la nueva ubicación y la dirección IP las máquinas virtuales de Azure. 

## <a name="post-migration-best-practices"></a>Procedimientos recomendados después de la migración

- Para aumentar la resistencia:
    - Proteja los datos mediante la copia de seguridad de máquinas virtuales de Azure mediante el servicio Azure Backup. [Más información](../backup/quick-backup-vm-portal.md).
    - Mantenga las cargas de trabajo en ejecución y disponibles continuamente mediante la replicación de máquinas virtuales de Azure en una región secundaria con Site Recovery. [Más información](../site-recovery/azure-to-azure-tutorial-enable-replication.md).
- Para aumentar la seguridad:
    - Bloquee y limite el acceso de tráfico de entrada con la [administración Just-In-Time de Azure Security Center](../security-center/security-center-just-in-time.md).
    - Restrinja el tráfico de red a los puntos de conexión de administración con [grupos de seguridad de red](../virtual-network/security-overview.md).
    - Implemente [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) para ayudar a proteger discos y datos frente al robo y acceso no autorizado.
    - Obtenga más información sobre la [protección de recursos IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) y visite [Azure Security Center](https://azure.microsoft.com/services/security-center/).
- Para supervisión y administración:
-  Considere la posibilidad de implementar [Azure Cost Management](../cost-management-billing/cloudyn/overview.md) para supervisar el gasto y el uso de recursos.


## <a name="next-steps"></a>Pasos siguientes

Investigue el [proceso de la migración en la nube](/azure/architecture/cloud-adoption/getting-started/migrate) en el marco de Cloud Adoption Framework para Azure.
