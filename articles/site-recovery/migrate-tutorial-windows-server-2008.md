---
title: Migración de servidores de Windows Server 2008 a Azure con Azure Site Recovery
description: En este artículo se describe cómo migrar máquinas de Windows Server 2008 locales a Azure mediante Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/27/2020
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 8d672c1113f265f9fbbabc7caed8df071f548f2a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503827"
---
# <a name="migrate-servers-running-windows-server-2008-to-azure"></a>Migración de servidores que ejecutan Windows Server 2008 a Azure

En este tutorial se muestra cómo migrar los servidores locales que ejecutan Windows Server 2008 o 2008 R2 a Azure con Azure Site Recovery. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Preparar el entorno local para la migración.
> * Configure el entorno de destino.
> * Configurar una directiva de replicación.
> * Habilite la replicación.
> * Ejecute una migración de prueba para asegurarse de que todo funciona de la forma esperada.
> * Conmutar por error a Azure y completar la migración.

## <a name="migrate-with-azure-migrate"></a>Migración con Azure Migrate

Se recomienda migrar las máquinas a Azure mediante el servicio [Azure Migrate](../migrate/migrate-services-overview.md). Azure Migrate proporciona un centro de conectividad centralizado para la valoración y la migración de máquinas locales a Azure, mediante Azure Migrate, otros servicios de Azure y herramientas de terceros. Azure Site Recovery debe usarse solo para la recuperación ante desastres, no para la migración.

Azure Migrate admite la migración de servidores que ejecutan Windows Server 2008.


## <a name="migrate-with-site-recovery"></a>Migración con Site Recovery

### <a name="supported-operating-systems"></a>Sistemas operativos admitidos


|Sistema operativo  | Entorno  |
|---------|---------|
|Windows Server 2008 SP2: 32 y 64 bits (IA-32 y x86-64)</br>- Standard</br>- Enterprise</br>- Datacenter   |     Máquinas virtuales de VMware, máquinas virtuales de Hyper-V y servidores físicos    |
|Windows Server 2008 R2 SP1: 64 bits</br>- Standard</br>- Enterprise</br>- Datacenter     |     Máquinas virtuales de VMware, máquinas virtuales de Hyper-V y servidores físicos|

> [!WARNING]
> - No se admite la migración de servidores que ejecutan Server Core.
> - Asegúrese de que tiene el último Service Pack y las actualizaciones de Windows instalados antes de migrar.


### <a name="prerequisites"></a>Requisitos previos

Antes de empezar, es útil revisar la arquitectura de Azure Site Recovery para la [migración de servidores de VMware y físicos](vmware-azure-architecture.md) o la [migración de máquinas virtuales de Hyper-V](hyper-v-azure-architecture.md). 

Para migrar máquinas virtuales de Hyper-V que ejecutan Windows Server 2008 o Windows Server 2008 R2, siga los pasos descritos en el tutorial [migrar máquinas locales a Azure](migrate-tutorial-on-premises-azure.md).

El resto de este tutorial muestra cómo puede migrar máquinas virtuales de VMware locales y servidores físicos que ejecutan Windows Server 2008 o 2008 R2.
> [!TIP]
> ¿Busca una manera sin agente de migrar máquinas virtuales de VMware a Azure? [Haga clic aquí](https://aka.ms/migrateVMs-signup)


### <a name="limitations-and-known-issues"></a>Limitaciones y problemas conocidos

- El servidor de configuración, los servidores de procesos adicionales y el servicio de movilidad que se usan para migrar los servidores de Windows Server 2008 SP2 deben ejecutar la versión 9.19.0.0 o posterior del software de Azure Site Recovery.

- No se admiten puntos de recuperación coherentes con la aplicación y la característica de coherencia de varias máquinas virtuales para la replicación de servidores que ejecutan Windows Server 2008 SP2. Los servidores de Windows Server 2008 SP2 deben migrarse a un punto de recuperación coherente para el bloqueo. Los puntos de recuperación coherentes para el bloqueo se generan cada 5 minutos de forma predeterminada. Mediante una directiva de replicación con una frecuencia de instantánea coherente con la aplicación configurada hará que el estado de replicación se vuelva crítico debido a la falta de puntos de recuperación coherentes con la aplicación. Para evitar falsos positivos, establezca la frecuencia de instantáneas coherentes con la aplicación en la directiva de replicación en "Desactivado".

- Los servidores que se están migrando deben tener .NET Framework 3.5 Service Pack 1 para que el servicio de movilidad funcione.

- Si el servidor tiene discos dinámicos, verá que en determinadas configuraciones estos discos en el servidor con conmutación por error se marcan como desconectados o se muestran como discos externos. También es posible que vea que el estado del conjunto reflejado para los volúmenes reflejados en discos dinámicos se marca como "Error de redundancia". Puede solucionar este problema desde diskmgmt.msc importando manualmente estos discos y reactivándolos.

- Los servidores que se están migrando deben tener el controlador vmstorfl.sys. La conmutación por error puede producir un error si el controlador no está presente en el servidor que se está migrando. 
  > [!TIP]
  >Compruebe si el controlador se encuentra en "C:\Windows\system32\drivers\vmstorfl.sys". Si no se encuentra el controlador, puede solucionar el problema mediante la creación de un archivo ficticio en su lugar. 
  >
  > Abra un símbolo (Ejecutar > Cmd) y ejecute lo siguiente: "copy nul c:\Windows\system32\drivers\vmstorfl.sys"

- Es posible que no pueda usar el protocolo RDP a los servidores de Windows Server 2008 SP2 que ejecutan el sistema operativo de 32 bits inmediatamente después de que se conmuten por error o se conmuten por error de prueba a Azure. Reinicie la máquina virtual conmutada por error desde Azure Portal e intente conectarse de nuevo. Si todavía no puede conectarse, compruebe si el servidor está configurado para permitir conexiones a escritorio remotas y asegúrese de que no hay ninguna regla de firewall ni grupos de seguridad de red bloqueando la conexión. 
  > [!TIP]
  > Se recomienda una conmutación por error de prueba antes de migrar los servidores. Asegúrese de que ha realizado al menos una conmutación por error de prueba correcta en cada servidor que va a migrar. Como parte de la prueba de conmutación por error, conéctese a la máquina de la conmutación por error de prueba y asegúrese que las cosas funcionan según lo previsto.
  >
  >La operación de conmutación por error de prueba no es disruptiva y le ayuda a probar las migraciones mediante la creación de máquinas virtuales en una red aislada de su elección. A diferencia de la operación de conmutación por error, durante la operación de conmutación por error de prueba, la replicación de datos continúa realizándose. Puede realizar tantas conmutaciones por error de prueba como sea necesario antes de tener todo listo para migrar. 
  >
  


### <a name="get-started"></a>Primeros pasos

Realice las siguientes tareas para preparar el entorno de VMware y físico locales y la suscripción de Azure:

1. [Preparación de Azure](tutorial-prepare-azure.md)
2. Preparación de [VMware](vmware-azure-tutorial-prepare-on-premises.md) local


### <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

1. Inicie sesión en [Azure Portal](https://portal.azure.com) > **Recovery Services**.
2. Seleccione **Crear un recurso** > **Herramientas de administración** > **Backup and Site Recovery**.
3. En el apartado **Nombre**, especifique el nombre descriptivo **W2K8-migration**. Si tiene más de una suscripción, seleccione la apropiada.
4. Cree un grupo de recursos **w2k8migrate**.
5. Especifique una región de Azure. Para comprobar las regiones admitidas, consulte la disponibilidad geográfica en [Detalles de precios de Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
6. Para acceder rápidamente al almacén desde el panel, haga clic en **Anclar al panel** y, después, en **Crear**.

   ![Captura de pantalla que muestra las opciones de creación de almacenes.](media/migrate-tutorial-windows-server-2008/migrate-windows-server-2008-vault.png)

El nuevo almacén se agrega al **Panel**, en **Todos los recursos**, y en la página principal **Almacenes de Recovery Services**.


### <a name="prepare-your-on-premises-environment-for-migration"></a>Preparar los servidores locales para la migración

- Para migrar máquinas virtuales de Windows Server 2008 que se ejecutan en VMware, [configure el servidor de configuración local en VMware](vmware-azure-tutorial.md#set-up-the-source-environment).
- Si el servidor de configuración no puede configurarse como máquina virtual de VMware, [configure el servidor de configuración en una máquina virtual o servidor físico local](physical-azure-disaster-recovery.md#set-up-the-source-environment).

### <a name="set-up-the-target-environment"></a>Configuración del entorno de destino

Seleccione y compruebe los recursos de destino.

1. Haga clic en **Preparar infraestructura** > **Destino** y seleccione la suscripción de Azure que quiere usar.
2. Especifique el modelo de implementación de Resource Manager.
3. Site Recovery comprueba que tiene una o más redes y cuentas de Azure Storage compatibles.


### <a name="set-up-a-replication-policy"></a>Configurar una directiva de replicación

1. Para crear una directiva de replicación, haga clic en **Site Recovery Infrastructure (Infraestructura de Site Recovery)**  > **Directivas de replicación** >  **+Directiva de replicación**.
2. En **Crear directiva de replicación**, especifique un nombre de directiva.
3. En **Umbral de RPO**, especifique el límite del objetivo de punto de recuperación (RPO). Se genera una alerta si el RPO de replicación supera este límite.
4. En **Retención de punto de recuperación**, especifique la duración (en horas) del período de retención de cada punto de recuperación. Los servidores replicados se pueden recuperar a cualquier momento de este período. Se admite una retención de hasta 24 horas para máquinas replicadas en Premium Storage y 72 horas para almacenamiento estándar.
5. En **Frecuencia de las instantáneas coherentes con la aplicación**, especifique **Desactivado**. Haga clic en **Aceptar** para crear la directiva.

La directiva se asocia automáticamente al servidor de configuración.

> [!WARNING]
> Asegúrese de especificar **DESACTIVADO** en la configuración Frecuencia de las instantáneas coherentes con la aplicación de la directiva de replicación. Solo se admiten los puntos de recuperación coherentes con el bloqueo al replicar los servidores que ejecutan Windows Server 2008. Especificar cualquier otro valor para la Frecuencia de instantánea coherente con la aplicación generará alertas falsas volviendo el estado de replicación del servidor crítico debido a la falta de puntos de recuperación coherentes con la aplicación.

   ![Captura de pantalla que muestra las opciones de creación de directivas de replicación.](media/migrate-tutorial-windows-server-2008/create-policy.png)

### <a name="enable-replication"></a>Habilitar replicación

[Habilite la replicación](physical-azure-disaster-recovery.md#enable-replication) para el servidor de Windows Server 2008 SP2 o Windows Server 2008 R2 SP1 que se va a migrar.
   
   ![Captura de pantalla que muestra las opciones para agregar máquinas físicas.](media/migrate-tutorial-windows-server-2008/Add-physical-server.png)

   ![Captura de pantalla que muestra las opciones para habilitar la replicación.](media/migrate-tutorial-windows-server-2008/Enable-replication.png)

### <a name="run-a-test-migration"></a>Ejecutar una migración de prueba

Puede realizar una prueba de conmutación por error de replicación de servidores después de que se complete la replicación inicial y el estado del servidor pase a **Protegido**.

Ejecute una [conmutación por error de prueba](tutorial-dr-drill-azure.md) en Azure para asegurarse de que todo funciona de la forma esperada.

   ![Captura de pantalla que muestra el comando Probar conmutación por error.](media/migrate-tutorial-windows-server-2008/testfailover.png)


### <a name="migrate-to-azure"></a>Migración a Azure

Ejecute una conmutación por error para las máquinas que desea migrar.

1. En **Configuración** > **Elementos replicados**, haga clic en la máquina > **Conmutación por error**.
2. En **Conmutación por error**, seleccione un **punto de recuperación** en el que realizar la conmutación por error. Seleccione el punto de recuperación más reciente.
3. Seleccione **Apague la máquina antes de comenzar con la conmutación por error**. Después, Site Recovery intentará apagar el servidor antes de desencadenar la conmutación por error. La conmutación por error continúa aunque se produzca un error de cierre. Puede seguir el progreso de la conmutación por error en la página **Trabajos**.
4. Compruebe que la máquina virtual de Azure aparece en Azure según lo previsto.
5. En **Elementos replicados**, haga clic con el botón derecho en el servidor > **Completar migración**. Esto hace lo siguiente:

    - Finaliza el proceso de migración, detiene la replicación del servidor y detiene la facturación de Site Recovery para el servidor.
    - Este paso limpia los datos de replicación. No elimina las máquinas virtuales migradas.

   ![Captura de pantalla que muestra el comando Completar la migración.](media/migrate-tutorial-windows-server-2008/complete-migration.png)


> [!WARNING]
> **No cancele una conmutación por error en curso**: La replicación del servidor se detiene antes de que se inicie la conmutación por error. Si se cancela una conmutación por error en curso, esta se detiene, pero el servidor dejará de realizar la replicación.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Revise las preguntas habituales](../migrate/resources-faq.md) sobre Azure Migrate.
