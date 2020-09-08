---
title: Migración de máquinas virtuales de Hyper-V a Azure con la migración de servidores de Azure Migrate
description: Aprenda a migrar máquinas virtuales de Hyper-V locales a Azure con la migración de servidores de Azure Migrate Server
ms.topic: tutorial
ms.date: 06/08/2020
ms.custom:
- MVC
- fasttrack-edit
ms.openlocfilehash: 370712ea976db027abee943d786f6ce81ac99c4a
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89376016"
---
# <a name="migrate-hyper-v-vms-to-azure"></a>Migración de máquinas virtuales de Hyper-V a Azure 

En este artículo se muestra cómo migrar máquinas virtuales Hyper-V locales a Azure mediante la herramienta [Azure Migrate:Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool).

Este tutorial es el tercero de una serie que muestra cómo evaluar máquinas y migrarlas a Azure. 

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

1. [Revisar](hyper-v-migration-architecture.md)la arquitectura de migración de Hyper-V.
2. [Revise](migrate-support-matrix-hyper-v-migration.md#hyper-v-host-requirements) los requisitos de los hosts de Hyper-V para la migración y las direcciones URL de Azure a las que los hosts y clústeres de Hyper-V necesitan acceder para la migración de las máquinas virtuales.
3. [Consulte](migrate-support-matrix-hyper-v-migration.md#hyper-v-vms) los requisitos de las máquinas virtuales de Hyper-V que desea migrar a Azure.
4. Se recomienda [evaluar las máquinas virtuales Hyper-V](tutorial-assess-hyper-v.md) antes de migrarlas a Azure, pero no es obligatorio.

   
## <a name="add-the-azure-migrateserver-migration-tool"></a>Incorporación de la herramienta Azure Migrate:Server Migration

Agregue la herramienta Azure Migrate:Server Migration. Si aún no tiene un proyecto de Azure Migrate, [créelo en primer lugar](how-to-add-tool-first-time.md) para configurar un proyecto de Azure Migrate. Al crear el proyecto, se agrega la herramienta Azure Migrate:Server Migration.

Si tiene un proyecto configurado, agregue la herramienta de la siguiente manera:

1. En el proyecto de Azure Migrate, haga clic en **Información general**. 
2. En **Detectar, evaluar y migrar servidores**, haga clic en **Evaluar y migrar servidores**.
3. En **Herramientas de migración** , seleccione **Click here to add a migration tool when you are ready to migrate** (Haga clic aquí para agregar una herramienta de migración cuando esté listo para migrar).

    ![Seleccionar una herramienta](./media/tutorial-migrate-hyper-v/select-migration-tool.png)

4. En la lista de herramientas, seleccione **Azure Migrate: Server Migration** > **Agregar herramienta**.

    ![Herramienta Server Migration](./media/tutorial-migrate-hyper-v/server-migration-tool.png)

## <a name="download-and-install-the-provider"></a>Descarga e instalación del proveedor

Para migrar máquinas virtuales Hyper-V, Azure Migrate:Server Migration instala proveedores de software (proveedor de Microsoft Azure Site Recovery y agente de Microsoft Azure Recovery Service) en los hosts de Hyper-V o en los nodos del clúster. Tenga en cuenta que no se utiliza el [dispositivo Azure Migrate](migrate-appliance.md) para la migración de Hyper-V.

1. En el proyecto de Azure Migrate > **Servidores**, en **Azure Migrate: Migración del servidor**, haga clic en **Detectar**.
2. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione **Sí, con Hyper-V**.
3. En **Región de destino**, seleccione la región de Azure a la que desea migrar las máquinas.
6. Seleccione **Confirme que la región de destino de la migración es nombreDeRegión**.
7. Haga clic en **Crear recursos**. Esto crea un almacén de Azure Site Recovery en segundo plano.
    - Si ya ha configurado la migración con Azure Migrate Server Migration, esta opción no aparecerá, ya que los recursos se configuraron anteriormente.
    - Después de hacer clic en este botón ya no se puede cambiar la región de destino de este proyecto.
    - Todas las migraciones posteriores se realizan a esta región.
    
8. En **Preparar los servidores host de Hyper-V**, descargue el proveedor de replicación de Hyper-V y el archivo de la clave de registro.
    - La clave de registro es necesaria para registrar el host de Hyper-V en Azure Migrate Server Migration.
    - La clave será válida durante cinco días a partir del momento en que se genera.

    ![Descargar el proveedor y la clave](./media/tutorial-migrate-hyper-v/download-provider-hyper-v.png)

4. Copie el archivo de instalación del proveedor y el archivo de la clave de registro en todos los hosts de Hyper-V (o nodos de clúster) que ejecuten las máquinas virtuales que desea replicar.
5. Ejecute el archivo de instalación del proveedor en cada host, como se describe en el procedimiento siguiente.
6. Después de instalar el proveedor en los hosts, en **Detectar máquinas**, haga clic en **Finalizar registro**.

    ![Finalizar el registro](./media/tutorial-migrate-hyper-v/finalize-registration.png)

Tras la finalización del registro, pueden pasar 15 minutos hasta que las máquinas virtuales detectadas aparecen en Azure Migrate Server Migration. A medida que se detectan las máquinas virtuales, aumenta el número de **Servidores detectados**.

![Servidores detectados](./media/tutorial-migrate-hyper-v/discovered-servers.png)


## <a name="replicate-hyper-v-vms"></a>Replicación de máquinas virtuales de Hyper-V

Una vez finalizada la detección, puede comenzar la replicación de máquinas virtuales de Hyper-V en Azure.

> [!NOTE]
> Puede replicar hasta 10 máquinas juntas. Si necesita replicar más, replíquelas simultáneamente en lotes de 10.

1. En el proyecto de Azure Migrate > **Servidores**, **Azure Migrate: Migración del servidor**, haga clic en **Replicar**.
2. En **Replicar** > **Configuración de origen** >  **¿Las máquinas están virtualizadas?** , seleccione **Sí, con Hyper-V**. A continuación, haga clic en **Siguiente: Máquinas virtuales**.
3. En **Máquinas virtuales**, seleccione las máquinas que desea replicar.
    - Si ha ejecutado una evaluación para las máquinas virtuales, puede aplicar las recomendaciones de tamaño y tipo de disco (Premium/estándar) de máquina virtual que sugieren los resultados de dicha evaluación. Para ello, en **¿Quiere importar la configuración de migración de evaluación de Azure Migrate?** , seleccione la opción **Sí**.
    - Si no ha ejecutado una evaluación o no desea usar la configuración de evaluación, seleccione la opción **No**.
    - Si ha decidido usar la evaluación, seleccione el grupo de máquinas virtuales y el nombre de la evaluación.

        ![Seleccionar evaluación](./media/tutorial-migrate-hyper-v/select-assessment.png)

4. En **Máquinas virtuales**, busque las máquinas virtuales que necesite y compruebe todas las que desee migrar. Después, haga clic en **Siguiente: Configuración de destino**.

    ![Selección de las máquinas virtuales](./media/tutorial-migrate-hyper-v/select-vms.png)

5. En **Configuración de destino**, seleccione la región de destino a la que va a realizar la migración, la suscripción y el grupo de recursos en el que residirán las máquinas virtuales de Azure después de la migración.
7. En **Cuenta de almacenamiento de replicación**, seleccione la cuenta de Azure Storage en la que se almacenarán los datos replicados en Azure.
8. **Red virtual**, seleccione la red virtual o la subred de Azure a la que se unirán las máquinas virtuales de Azure después de la migración.
9. En **Opciones de disponibilidad**, seleccione:
    -  La zona de disponibilidad para anclar la máquina migrada a una zona de disponibilidad específica de la región. Use esta opción para distribuir los servidores que forman una capa de aplicación de varios nodos en Availability Zones. Si selecciona esta opción, deberá especificar la zona de disponibilidad que se va a usar en cada una de las máquinas seleccionadas en la pestaña Proceso. Esta opción solo está disponible si la región de destino seleccionada para la migración admite Availability Zones.
    -  El conjunto de disponibilidad para colocar la máquina migrada en un conjunto de disponibilidad. Para usar esta opción, el grupo de recursos de destino seleccionado debe tener uno o varios conjuntos de disponibilidad.
    - No se requiere ninguna opción de redundancia de infraestructura si no necesita ninguna de estas configuraciones de disponibilidad para las máquinas migradas.
10. En **Ventaja híbrida de Azure**:

    - Seleccione **No** si no desea aplicar la Ventaja híbrida de Azure. A continuación, haga clic en **Siguiente**.
    - Seleccione **Sí** si tiene equipos con Windows Server que están incluidos en suscripciones activas de Software Assurance o Windows Server y desea aplicar el beneficio a las máquinas que va a migrar. A continuación, haga clic en **Siguiente**.

    ![Configuración de destino](./media/tutorial-migrate-hyper-v/target-settings.png)

11. En **Proceso**, revise el nombre, el tamaño, el tipo de disco del sistema operativo y la configuración de disponibilidad (si se ha seleccionado en el paso anterior) de la máquina virtual. Las máquinas virtuales deben cumplir los [requisitos de Azure](migrate-support-matrix-hyper-v-migration.md#azure-vm-requirements).

    - **Tamaño de VM**: si usa las recomendaciones de la evaluación, el menú desplegable de tamaño de VM contendrá el tamaño recomendado. De lo contrario, Azure Migrate elige un tamaño en función de la coincidencia más cercana en la suscripción de Azure. También puede elegir un tamaño de manera manual en **Tamaño de la máquina virtual de Azure**. 
    - **Disco del sistema operativo**: especifique el disco del sistema operativo (arranque) de la máquina virtual. Este es el disco que tiene el cargador de arranque y el instalador del sistema operativo. 
    - **Conjunto de disponibilidad**: si la máquina virtual debe estar incluida en un conjunto de disponibilidad de Azure después de la migración, especifique el conjunto. El conjunto debe estar en el grupo de recursos de destino que especifique para la migración.

    ![Configuración de los recursos de proceso de la máquina virtual](./media/tutorial-migrate-hyper-v/compute-settings.png)

12. En **Discos**, especifique si los discos de la máquina virtual se deben replicar en Azure y seleccione el tipo de disco (discos SSD o HDD estándar, o bien discos administrados prémium) en Azure. A continuación, haga clic en **Siguiente**.
    - Puede excluir discos de la replicación.
    - Si excluye discos, no estarán presentes en la máquina virtual de Azure después de la migración. 

    ![Discos](./media/tutorial-migrate-hyper-v/disks.png)

13. En **Revisar e iniciar la replicación**, revise la configuración y haga clic en **Replicar** para iniciar la replicación inicial de los servidores.

> [!NOTE]
> Puede actualizar la configuración de replicación en cualquier momento antes de que esta comience; para ello, vaya a **Administrar** > **Replicación de máquinas**. Una vez iniciada la replicación, su configuración no se puede cambiar.

## <a name="provision-for-the-first-time"></a>Primer aprovisionamiento

Si se trata de la primera máquina virtual que va a replicar en el proyecto de Azure Migrate, Azure Migrate: Server Migration aprovisiona automáticamente estos recursos en el mismo grupo de recursos que el proyecto.

- **Service Bus**: Azure Migrate: Server Migration usa Service Bus para enviar mensajes de orquestación de replicación al dispositivo.
- **Cuenta de almacenamiento de puerta de enlace**: Azure Migrate: Server Migration usa la cuenta de almacenamiento de puerta de enlace para almacenar información del estado de las máquinas virtuales que se replican.
- **Cuenta de almacenamiento de registros**: el dispositivo con Azure Migrate carga los registros de replicación de las máquinas virtuales en una cuenta de almacenamiento de registros. Azure Migrate aplica la información de replicación a los discos administrados de réplica.
- **Almacén de claves**: el dispositivo con Azure Migrate usa el almacén de claves para administrar las cadenas de conexión de Service Bus y las claves de acceso de las cuentas de almacenamiento utilizadas en la replicación. Debe haber configurado los permisos que necesita el almacén de claves para acceder a la cuenta de almacenamiento al [preparar Azure](tutorial-prepare-hyper-v.md#prepare-azure) para la evaluación y migración de la máquina virtual de Hyper-V. 


## <a name="track-and-monitor"></a>Seguimiento y supervisión


- Al hacer clic en **Replicar**, comienza el trabajo de inicio de replicación. 
- Cuando el trabajo de inicio de replicación finaliza correctamente, las máquinas comienzan su replicación inicial en Azure.
- Cuando finaliza esta replicación inicial, comienza la replicación diferencial. Los cambios incrementales de los discos locales se replican periódicamente en Azure.

Puede realizar un seguimiento del estado del trabajo en las notificaciones del portal.

Para supervisar el estado de la replicación, haga clic en **Replicando servidores** en **Azure Migrate: Server Migration**.
![Supervisión de la replicación](./media/tutorial-migrate-hyper-v/replicating-servers.png)



## <a name="run-a-test-migration"></a>Ejecutar una migración de prueba


Cuando comienza la replicación diferencial, puede ejecutar una migración de prueba para las máquinas virtuales antes de ejecutar una migración completa a Azure. Le recomendamos encarecidamente que lo haga al menos una vez en cada máquina, antes de migrarla.

- La ejecución de una migración de prueba comprueba que la migración funcionará según lo previsto, sin afectar a las máquinas locales, que seguirán estando operativas y continuarán realizando la replicación. 
- Para simular la migración, la migración de prueba crea una máquina virtual de Azure usando datos replicados (normalmente, con una migración a una red virtual de Azure que no sea de producción en la suscripción a Azure).
- Puede usar la máquina virtual de Azure de prueba replicada para validar la migración, realizar pruebas de aplicaciones y resolver los problemas antes de la migración completa.

Realice una migración de prueba como se indica a continuación:


1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Migration**, haga clic en **Probar servidores migrados**.

     ![Probar servidores migrados](./media/tutorial-migrate-hyper-v/test-migrated-servers.png)

2. Haga clic con el botón derecho en la máquina virtual que va a probar y haga clic en **Migración de prueba**.

    ![Migración de prueba](./media/tutorial-migrate-hyper-v/test-migrate.png)

3. En **Migración de prueba**, seleccione la red virtual de Azure en la que se ubicará la máquina virtual de Azure después de la migración. Se recomienda usar una red virtual que no sea de producción.
4. Comienza el trabajo de **Migración de prueba**. Supervise el trabajo en las notificaciones del portal.
5. Una vez finalizada la migración, la máquina virtual de Azure migrada se puede ver en **Máquinas virtuales** en Azure Portal. El nombre de la máquina tiene el sufijo **-Test**.
6. Una vez finalizada la prueba, haga clic con el botón derecho en la máquina virtual de Azure, en **Replicación de máquinas**, y haga clic en **Limpiar la migración de prueba**.

    ![Limpiar la migración](./media/tutorial-migrate-hyper-v/clean-up.png)


## <a name="migrate-vms"></a>Migración de máquinas virtuales

Después de comprobar que la migración de prueba funciona según lo previsto, puede migrar las máquinas locales.

1. En el proyecto de Azure Migrate > **Servidores** > **Azure Migrate: Server Migration**, haga clic en **Replicando servidores**.

    ![Replicando servidores](./media/tutorial-migrate-hyper-v/replicate-servers.png)

2. En **Replicación de máquinas**, haga clic con el botón derecho en la máquina virtual > **Migrar**.
3. En **Migrar** >  **¿Quiere apagar las máquinas virtuales y realizar una migración planificada sin perder datos?** , seleccione **Sí** > **Aceptar**.
    - De forma predeterminada, Azure Migrate apaga la máquina virtual local y ejecuta una replicación a petición para sincronizar los cambios que se han producido en la máquina virtual desde la última replicación. De esta forma se garantiza que no se pierden datos.
    - Si no desea apagar la máquina virtual, seleccione **No**
4. Se inicia un trabajo de migración de la máquina virtual. Realice un seguimiento del trabajo en las notificaciones de Azure.
5. Una vez finalizado el trabajo, la máquina virtual puede ver y administrar desde la página **Máquinas virtuales**.

## <a name="complete-the-migration"></a>Completar la migración

1. Una vez finalizada la migración, haga clic con el botón derecho en la máquina virtual > **Detener migración**. Esto hace lo siguiente:
    - Detiene la replicación en la máquina local.
    - Quita la máquina del recuento de **Servidores en replicación** en Azure Migrate: Server Migration.
    - Limpia la información de estado de replicación de la máquina virtual.
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
