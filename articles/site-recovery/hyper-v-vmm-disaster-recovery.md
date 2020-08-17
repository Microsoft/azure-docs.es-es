---
title: Configuración de la recuperación ante desastres de Hyper-V en un sitio secundario con Azure Site Recovery
description: Obtenga información sobre cómo configurar la recuperación ante desastres para máquinas virtuales de Hyper-V entre los sitios locales con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: how-to
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: b2164f8927e5c3224f8b07c30d057f48fb7bbc32
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495985"
---
# <a name="set-up-disaster-recovery-for-hyper-v-vms-to-a-secondary-on-premises-site"></a>Configuración de la recuperación ante desastres para máquinas virtuales de Hyper-V en el sitio local secundario

El servicio [Azure Site Recovery](site-recovery-overview.md) contribuye a su estrategia de recuperación ante desastres mediante la administración y la coordinación de la replicación, la conmutación por error y la conmutación por recuperación de máquinas locales y máquinas virtuales (VM) de Azure.

En este artículo se muestra cómo configurar la recuperación ante desastres en un sitio secundario, para máquinas virtuales de Hyper-V locales administradas en nubes de System Center Virtual Machine Manager (VMM). En este artículo aprenderá a:

> [!div class="checklist"]
> * Preparar los hosts de Hyper-V y los servidores VMM locales
> * Crear un almacén de Recovery Services para Site Recovery 
> * Configurar los entornos de replicación de origen y destino 
> * Configuración de la asignación de red 
> * Creación de una directiva de replicación
> * Habilitación de la replicación para una máquina virtual


## <a name="prerequisites"></a>Requisitos previos

Para realizar este escenario:

- Revise la [arquitectura del escenario y sus componentes](hyper-v-vmm-architecture.md).
- Asegúrese de que los servidores VMM y los hosts Hyper-V cumplan los [requisitos de compatibilidad](hyper-v-vmm-secondary-support-matrix.md).
- Compruebe que las máquinas que desea replicar cumplen con la [compatibilidad con máquinas replicadas](hyper-v-vmm-secondary-support-matrix.md#replicated-vm-support).
- Prepare los servidores VMM para la asignación de red.

### <a name="prepare-for-network-mapping"></a>Preparar la asignación de red

La [asignación de red](hyper-v-vmm-network-mapping.md) funciona entre redes de máquinas virtuales VMM locales en nubes de origen y nubes de destino. La asignación hace lo siguiente:

- Conecta las máquinas virtuales a las redes de máquinas virtuales de destino después de la conmutación por error. 
- Ubica de manera óptima las máquinas virtuales de réplica en servidores de host Hyper-V de destino. 
- Si no configura la asignación de red, las máquinas virtuales de réplica no se conectarán a ninguna red de máquina virtual después de la conmutación por error.

Prepare VMM de la siguiente manera:

1. Asegúrese de tener [redes lógicas VMM](/system-center/vmm/network-logical) en los servidores VMM de origen y destino.
    - La red lógica del servidor de origen se debe asociar con la nube de origen en la que se encuentran los hosts Hyper-V.
    - La red lógica del servidor de destino se debe asociar con la nube de destino.
1. Asegúrese de tener [redes de máquina virtual](/system-center/vmm/network-virtual) en los servidores VMM de origen y destino. Las redes de máquina virtual se deben vincular a la red lógica de cada ubicación.
2. Conecte máquinas virtuales en los host Hyper-V de origen a la red de máquina virtual de origen. 


## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

[!INCLUDE [site-recovery-create-vault](../../includes/site-recovery-create-vault.md)]


## <a name="choose-a-protection-goal"></a>Elección de un objetivo de protección

Seleccione aquello que desea replicar y la ubicación en donde se va a realizar la replicación.

1. Haga clic en **Site Recovery** > **Paso 1: Preparar la infraestructura** > **Objetivo de protección**.
2. Seleccione **Para sitio de recuperación** y seleccione **Sí, con Hyper-V**.
3. Seleccione **Sí** para indicar que usa VMM para administrar los hosts de Hyper-V.
4. Seleccione **Sí** si tiene un servidor VMM secundario. Si va a implementar la replicación entre nubes en un solo servidor VMM, haga clic en **No**. A continuación, haga clic en **Aceptar**.


## <a name="set-up-the-source-environment"></a>Configuración del entorno de origen

Instale el proveedor de Azure Site Recovery en servidores VMM, y detecte y registre servidores en el almacén.

1. Haga clic en **Preparar infraestructura** > **Origen**.
2. En **Preparar origen**, haga clic en **+ VMM** para agregar un servidor VMM.
3. En **Agregar servidor**, compruebe que aparezca **System Center VMM server** (Servidor VMM de System Center) en **Tipo de servidor**.
4. Descargue el archivo de instalación del proveedor de Azure Site Recovery.
5. Descargue la clave de registro. Necesitará esta información cuando instale el proveedor. La clave será válida durante cinco días a partir del momento en que se genera.

    ![Captura de pantalla de las opciones para descargar el proveedor y la clave de registro.](./media/hyper-v-vmm-disaster-recovery/source-settings.png)

6. Instale el proveedor en cada servidor VMM. No es necesario instalar explícitamente nada en los hosts de Hyper-V.

### <a name="install-the-azure-site-recovery-provider"></a>Instalación del proveedor de Azure Site Recovery

1. Ejecute el archivo de instalación del proveedor en cada servidor VMM. Si VMM está implementado en un clúster, instálelo por primera vez como sigue:
    -  Instale el proveedor en un nodo activo y finalice la instalación para registrar el servidor VMM en el almacén.
    - A continuación, instale el proveedor en los demás nodos. Los nodos del clúster deben ejecutar la misma versión del proveedor.
2. El programa de instalación ejecuta algunas comprobaciones de requisitos previos y solicita permiso para detener el servicio VMM. El servicio VMM se reiniciará automáticamente cuando finalice la instalación. Si lo instala en un clúster VMM, se le pide que detenga el rol de clúster.
3. En **Microsoft Update**, puede participar para especificar que las actualizaciones del proveedor se instalen según las directivas de Microsoft Update.
4. En **Instalación**, acepte o modifique la ubicación predeterminada de instalación y haga clic en **Instalar**.
5. Cuando finalice la instalación, haga clic en **Registrar** para registrar el servidor en el almacén.

    ![Captura de pantalla de Instalación del proveedor que incluye la ubicación de instalación.](./media/hyper-v-vmm-disaster-recovery/provider-register.png)
6. En **Nombre del almacén**, compruebe el nombre del almacén en el que se registrará el servidor. Haga clic en **Next**.
7. En **Conexión de proxy**, especifique cómo se conecta a Azure el proveedor que se ejecuta en el servidor VMM.
   - Puede especificar que el proveedor debe conectarse a Internet directamente o a través de un proxy. Especifique la configuración de proxy según sea necesario.
   - Si usa un proxy, se crea automáticamente una cuenta de ejecución de VMM (DRAProxyAccount) que usa las credenciales de proxy especificadas. Configure el servidor proxy para que esta cuenta pueda autenticarse correctamente. Se puede modificar la configuración de la cuenta de ejecución en la consola VMM > **Configuración** > **Seguridad** > **Cuentas de ejecución**.
   - Reinicie el servicio VMM para actualizar los cambios.
8. En **Clave de registro**, seleccione la clave que descargó del portal y copió en el servidor VMM.
9. La configuración de cifrado no es importante en este escenario. 
10. En **Nombre del servidor**, especifique un nombre descriptivo para identificar el servidor VMM en el almacén. En un clúster, especifique el nombre del rol de clúster VMM.
11. En **Sincronizar metadatos en la nube** seleccione si quiere sincronizar los metadatos de todas las nubes del servidor VMM. Esta acción solo se debe ejecutar una vez en cada servidor. Si no desea sincronizar todas las nubes, deje desactivada esta opción. Puede sincronizar individualmente cada nube en las propiedades de nube de la consola VMM.
12. Haga clic en **Next** para finalizar el proceso. Después del registro, Site Recovery recupera los metadatos del servidor VMM. El servidor se muestra en **Servidores** > **Servidores VMM** en el almacén.
13. Después de que el servidor aparece en el almacén, en **Origen** > **Preparar origen**, seleccione el servidor VMM y la nube en la que se encuentra el host Hyper-V. A continuación, haga clic en **Aceptar**.


## <a name="set-up-the-target-environment"></a>Configuración del entorno de destino

Seleccione el servidor VMM y la nube de destino:

1. Haga clic en **Preparar infraestructura** > **Destino** y seleccione el servidor VMM de destino.
2. Se muestran las nubes VMM que están sincronizadas con Site Recovery. Seleccione la nube de destino.

   ![Captura de pantalla de las selecciones de Servidor VMM y Nube de destino.](./media/hyper-v-vmm-disaster-recovery/target-vmm.png)


## <a name="set-up-a-replication-policy"></a>Configurar una directiva de replicación

Antes de comenzar, asegúrese de que todos los hosts que usan la directiva tienen el mismo sistema operativo. Si los hosts ejecutan versiones distintas de Windows Server, se necesitan varias directivas de replicación.

1. Para crear una nueva directiva de replicación, haga clic en **Preparar infraestructura** > **Configuración de la replicación** >  **+Crear y asociar**.
2. En **Crear y asociar directiva**, especifique un nombre de directiva. El tipo de origen y destino debe ser **Hyper-V**.
3. En **Versión del host de Hyper-V** , seleccione qué sistema operativo se ejecuta en el host.
4. En **Tipo de autenticación** y **Puerto de autenticación**, especifique cómo se autentica el tráfico entre los servidores host Hyper-V principal y secundario.
    - Seleccione **Certificado** , a menos que tenga un entorno Kerberos activo. Azure Site Recovery configurará automáticamente los certificados para la autenticación de HTTPS. No es necesario hacer nada manualmente.
    - De forma predeterminada, se abrirán los puertos 8083 y 8084 (para los certificados) en el Firewall de Windows en los servidores hosts de Hyper-V.
    - Si selecciona **Kerberos**, se usará un vale de Kerberos para la autenticación mutua de los servidores host. Kerberos solo es pertinente para servidores host de Hyper-V que se ejecutan en Windows Server 2012 R2 o versiones posteriores.
1. En **Frecuencia de copia**, especifique la frecuencia con la que desea replicar diferencias de datos después de la replicación inicial (cada 30 segundos, 5 o 15 minutos).
2. En **Retención de punto de recuperación**, especifique el tiempo (en horas) que estará disponible el período de retención para cada punto de recuperación. Las máquinas replicadas se pueden recuperar en cualquier punto dentro de un período.
3. En **Frecuencia de instantánea coherente con la aplicación**especifique la frecuencia (entre 1 y 12 horas) con la que se crearán los puntos de recuperación que contengan las instantáneas coherentes con la aplicación. Hyper-V usa dos tipos de instantáneas:
    - **Instantánea estándar**: proporciona una instantánea incremental de toda la máquina virtual.
    - **Instantánea coherente con la aplicación**: toma una instantánea en un momento dado de los datos de la aplicación dentro de la máquina virtual. El Servicio de instantáneas de volumen (VSS) garantiza que las aplicaciones se encuentren en un estado coherente cuando se toma la instantánea. La habilitación de las instantáneas coherentes con la aplicación afecta al rendimiento de la aplicación en las máquinas virtuales de origen. Establezca un valor que sea inferior al número de puntos de recuperación adicionales que configure.
4. En **Compresión de transferencia de datos**, especifique si se deben comprimir los datos replicados que se transfieren.
5. Seleccione **Eliminar máquina virtual de réplica** para especificar que la máquina virtual de réplica debe eliminarse si se deshabilita la protección para la máquina virtual de origen. Si habilita esta configuración, cuando deshabilita la protección para la máquina virtual de origen, dicha máquina virtual se quita de la consola de Site Recovery, la configuración de Site Recovery de VMM se quita de la consola VMM y la réplica se elimina.
6. En **Método de replicación inicial** , si va a replicar a través de la red, especifique si iniciará la replicación inicial o la programará. Para ahorrar ancho de banda de red, puede que quiera programarla fuera de las horas punta. A continuación, haga clic en **Aceptar**.

     ![Captura de pantalla de las opciones de la directiva de replicación.](./media/hyper-v-vmm-disaster-recovery/replication-policy.png)
     
7. La nueva directiva se asocia automáticamente a la nube VMM. En **Directiva de replicación**, haga clic en **Aceptar**. 


## <a name="enable-replication"></a>Habilitar replicación

1. Haga clic en **Replicar la aplicación** > **Origen**. 
2. En la hoja **Origen**, seleccione el servidor VMM y la nube donde se encuentran los hosts de Hyper-V que quiere replicar. A continuación, haga clic en **Aceptar**.
3. En **Destino**, compruebe el servidor VMM y la nube secundarios.
4. En **Máquinas virtuales**, seleccione las máquinas virtuales que quiere proteger de la lista.


Puede hacer un seguimiento del progreso de la acción **Habilitar protección** en **Trabajos** > **Trabajos de Site Recovery**. Una vez concluido el trabajo **Finalizar protección**, la replicación inicial finaliza y la máquina virtual está preparada para la conmutación por error.

## <a name="next-steps"></a>Pasos siguientes

[Exploración de la recuperación ante desastres](hyper-v-vmm-test-failover.md)
