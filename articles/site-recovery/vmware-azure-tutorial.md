---
title: Configuración de la recuperación ante desastres de máquinas virtuales de VMware en Azure con Azure Site Recovery
description: Aprenda a configurar la recuperación ante desastres para máquinas virtuales de VMware locales en Azure con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 11/12/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 9831305f3889f977a270630b40fa0d78ec1085bd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501208"
---
# <a name="set-up-disaster-recovery-to-azure-for-on-premises-vmware-vms"></a>Configurar la recuperación ante desastres para máquinas virtuales de VMware locales en Azure

En este artículo, se describe cómo habilitar la replicación de máquinas virtuales de VMware locales para la recuperación ante desastres en Azure mediante el servicio [Azure Site Recovery](site-recovery-overview.md).

Este es el tercer tutorial de una serie que muestra cómo configurar la recuperación ante desastres en Azure para máquinas virtuales locales de VMware. En el tutorial anterior, hemos [preparado el entorno de VMware local](vmware-azure-tutorial-prepare-on-premises.md) para la recuperación ante desastres en Azure.


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar los valores de replicación del origen y un servidor de configuración de Site Recovery en el entorno local.
> * Configurar el destino de la replicación.
> * Crear una directiva de replicación.
> * Habilitar la replicación para una máquina virtual de VMware.

> [!NOTE]
> Los tutoriales muestran la ruta de implementación más sencilla para un escenario. Usan opciones predeterminadas siempre que es posible y no muestran todos los valores y las rutas de acceso posibles. Para obtener instrucciones detalladas, consulte el artículo de la sección de procedimientos de la tabla de contenido de Site Recovery.

## <a name="before-you-start"></a>Antes de comenzar

Complete los tutoriales anteriores:
1. Asegúrese de que ha [configurado Azure](tutorial-prepare-azure.md) parea la recuperación ante desastres de VMware en un entorno local en Azure.
2. Siga [estos pasos](vmware-azure-tutorial-prepare-on-premises.md) para preparar la implementación de VMware en un entorno local para la recuperación ante desastres en Azure.
3. En este tutorial se muestra cómo replicar una sola máquina virtual. Si va a implementar varias máquinas virtuales de VMware, debe utilizar la [herramienta Deployment Planner](https://aka.ms/asr-deployment-planner). [Más información](site-recovery-deployment-planner.md) sobre esta herramienta.
4. Este tutorial usa varias opciones que se pueden usar de forma diferente:
    - En este tutorial se usa una plantilla de OVA para crear la máquina virtual de VMware del servidor de configuración. Si por cualquier motivo no puede hacerlo, siga [estas instrucciones](physical-manage-configuration-server.md) para establecer los valores del servidor de configuración manualmente.
    - En este tutorial, Site Recovery descarga e instala automáticamente MySQL en el servidor de configuración. Si lo prefiere, puede configurarlo manualmente. [Más información](vmware-azure-deploy-configuration-server.md#configure-settings).




## <a name="select-a-protection-goal"></a>Selección de un objetivo de protección

1. En **Almacenes de Recovery Services**, seleccione el nombre del almacén. En este escenario se usa **ContosoVMVault**.
2. En **Introducción**, seleccione Site Recovery. Luego, seleccione **Preparar infraestructura**.
3. En **Objetivo de protección** >  **¿Dónde están ubicadas las máquinas?** , seleccione **Local**.
4. En **¿A dónde quiere replicar las máquinas?** , seleccione **En Azure**.
5. En **¿Las máquinas están virtualizadas?** , seleccione **Yes, with VMware vSphere Hypervisor** (Sí, con VMware vSphere Hypervisor). Después, seleccione **Aceptar**.



## <a name="set-up-the-source-environment"></a>Configuración del entorno de origen

En el entorno de origen, es necesaria una máquina local individual de alta disponibilidad que hospede los componentes locales de Site Recovery:

- **Servidor de configuración**: El servidor de configuración coordina la comunicación entre el entorno local y Azure, además de administrar la replicación de datos.
- **Servidor de proceso**: El servidor de procesos actúa como puerta de enlace de replicación. Recibe los datos de la replicación; los optimiza mediante el almacenamiento en la caché, la compresión y el cifrado, y los envía a una cuenta de almacenamiento en Azure. El servidor de procesos también instala el agente de Mobility Service en las máquinas virtuales que se desean replicar, además de realizar la detección automática de las máquinas virtuales de VMware locales.
- **Servidor de destino principal**: El servidor de destino maestro controla los datos de replicación durante la conmutación por recuperación desde Azure.


Todos estos componentes se instalan conjuntamente en la máquina local individual que se conoce como *servidor de configuración*. De manera predeterminada, para la recuperación ante desastres de VMware, el servidor de configuración se configura como una máquina virtual de VMware de alta disponibilidad. Para ello, es preciso descargar una plantilla de Open Virtualization Application (OVA) preparada e importarla en VMware para crear la máquina virtual.

- La versión más reciente del servidor de configuración está disponible en el portal. También se puede descargar directamente del [Centro de descarga de Microsoft](https://aka.ms/asrconfigurationserver).
- Si por cualquier motivo no puede se puede usar una plantilla de OVA para configurar una máquina virtual, siga [estas instrucciones](physical-manage-configuration-server.md) para establecer los valores del servidor de configuración manualmente.
- La licencia que se proporciona con la plantilla de OVF es una licencia de evaluación válida durante 180 días. Windows debe ejecutarse en la máquina virtual de forma activa con la licencia requerida.


### <a name="download-the-vm-template"></a>Descarga de la plantilla de máquina virtual

1. En el almacén, vaya a **Preparar infraestructura** > **Origen**.
2. En **Preparar origen**, seleccione **+Servidor de configuración**.
3. En **Agregar servidor**, compruebe que aparezca **Servidor de configuración para VMware** en **Tipo de servidor**.
4. Descargue la plantilla OVA para el servidor de configuración.



## <a name="import-the-template-in-vmware"></a>Importación de la plantilla en VMware


1. Inicie sesión en el servidor VMware vCenter o el host vSphere ESXi con el cliente de VMware vSphere.
2. En el menú **File** (Archivo), seleccione **Deploy OVF Template** (Implementar plantilla OVF) para iniciar el **Asistente para implementar plantillas OVF**.

     ![Captura de pantalla del comando Deploy OVF template (Implementar plantilla de OVF) en el cliente de VMWare vSphere.](./media/vmware-azure-tutorial/vcenter-wizard.png)

3. En **Select source** (Seleccionar origen), especifique la ubicación de la plantilla OVF descargada.
4. En **Review details** (Revisar detalles), seleccione **Next** (Siguiente).
5. En **Select name and folder** (Seleccionar el nombre y la carpeta) y **Select configuration** (Seleccionar la configuración), acepte la configuración predeterminada.
6. En **Select storage** (Seleccionar almacenamiento), seleccione **Thick Provision Eager Zeroed** (Aprovisionamiento grueso diligente con ceros) en **Select virtual disk format** (Seleccionar formato de disco virtual) para obtener un rendimiento mejor.
7. En el resto de las páginas del asistente, acepte la configuración predeterminada.
8. En **Ready to Complete** (Listo para completarse), para configurar la máquina virtual con la configuración predeterminada, seleccione **Power on after deployment** > **Finish** (Encender después de la implementación > Finalizar).

   > [!TIP]
   > Si desea agregar una NIC adicional, desactive **Power on after deployment** > **Finish** (Encender después de la implementación > Finalizar). De forma predeterminada, la plantilla contiene una sola NIC. Puede agregar NIC adicionales después de la implementación.

## <a name="add-an-additional-adapter"></a>Incorporación de un adaptador adicional

Si desea agregar una NIC adicional al servidor de configuración, hágalo antes de registrar el servidor en el almacén. No se admiten adaptadores adicionales después del registro.

1. En el inventario de cliente de vSphere, haga clic con el botón derecho en la máquina virtual y seleccione **Editar configuración**.
2. En **Hardware**, seleccione **Agregar** > **Adaptador Ethernet**. Luego, seleccione **Siguiente**.
3. Seleccione el tipo de adaptador y la red.
4. Para conectar la NIC virtual al encender la máquina virtual, seleccione **Connect at power on** (Conectar al encender). Seleccione **Siguiente** > **Finalizar**. Después, seleccione **Aceptar**.


## <a name="register-the-configuration-server"></a>Registro del servidor de configuración

Una vez configurado dicho servidor, se registra en el almacén.

1. Desde la consola de cliente de VMware vSphere, encienda la VM.
2. La máquina virtual se inicia en una experiencia de instalación de Windows Server 2016. Acepte el contrato de licencia y especifique una contraseña de administrador.
3. Una vez finalizada la instalación, inicie sesión en la máquina virtual como administrador.
4. La primera vez que inicie sesión, se inicia la herramienta de configuración de Azure Site Recovery en unos segundos.
5. Especifique el nombre que se usará para registrar el servidor de configuración en Site Recovery. Luego, seleccione **Siguiente**.
6. La herramienta comprueba que la máquina virtual pueda conectarse a Azure. Una vez establecida la conexión, haga clic en **Iniciar sesión** para iniciar sesión en la suscripción de Azure. Las credenciales deben tener acceso al almacén donde desea registrar el servidor de configuración. Asegúrese de que los usuarios tengan asignados los roles [necesarios](vmware-azure-deploy-configuration-server.md#azure-active-directory-permission-requirements).
7. La herramienta realiza algunas tareas de configuración y, a continuación, se reinicia.
8. Inicie sesión de nuevo en la máquina. En pocos segundos, el Asistente para administración del servidor de configuración se inicia automáticamente.


### <a name="configure-settings-and-add-the-vmware-server"></a>Configuración de los valores y adición del servidor de VMware

Finalice la configuración y el registro del servidor de configuración. Antes de continuar, asegúrese de que se cumplen todos los [requisitos previos](vmware-azure-deploy-configuration-server.md#prerequisites) para configurar correctamente el servidor de configuración.


1. En el Asistente para administración del servidor de configuración, seleccione **Setup connectivity** (Conectividad de configuración). En las listas desplegables, seleccione primero la NIC que utiliza el servidor de procesos integrados para la detección y la instalación de inserción del servicio de movilidad en los equipos de origen y, a continuación, seleccione la NIC que usa el servidor de configuración para la conectividad con Azure. Después, seleccione **Guardar**. Una vez configurada, esta opción no se puede cambiar.
2. En **Seleccionar almacén de Recovery Services**, seleccione la suscripción de Azure y el grupo de recursos y almacén correspondientes.
3. En **Instalar software de terceros**, acepte el contrato de licencia. Seleccione **Descargar e instalar** para instalar MySQL Server. Si ha colocado MySQL en la ruta de acceso, este paso se puede omitir. [Más información](vmware-azure-deploy-configuration-server.md#configure-settings)
4. En **Validate appliance configuration** (Comprobar configuración del dispositivo), se comprueban los requisitos previos antes de continuar.
5. En **Configure vCenter Server/vSphere ESXi server** (Configurar vCenter Server/vSphere ESXi Server), especifique la dirección IP o FQDN del servidor vCenter o host de vSphere en los que se encuentran las máquinas virtuales donde desea realizar la replicación. Especifique el puerto en el que escucha el servidor. Escriba un nombre descriptivo que se usará para el servidor de VMware en el almacén.
6. Especifique las credenciales de usuario que utilizará el servidor de configuración para conectarse al servidor de VMware. Asegúrese de que el nombre de usuario y la contraseña son correctos y forman parte del grupo de administradores de la máquina virtual que se puede proteger. Site Recovery usa estas credenciales para detectar automáticamente las máquinas virtuales de VMware disponibles para la replicación. Seleccione **Agregar** y, luego, **Continuar**.
7. En **Configure virtual machine credentials** (Configurar las credenciales de la máquina virtual), especifique el nombre de usuario y la contraseña que se usarán para instalar automáticamente Mobility Service en las máquinas, cuando la replicación esté habilitada.
    - Para las máquinas Windows, la cuenta necesita privilegios de administrador local en las máquinas que se vayan a replicar.
    - Para Linux, proporcione los detalles de la cuenta raíz.
8. Seleccione **Finalize configuration** (Terminar configuración) para completar el registro.
9. Una vez concluido el registro, abra Azure Portal, compruebe que el servidor de configuración y el servidor de VMware se muestran en **Almacén de Recovery Services** > **Administrar** > **Infraestructura de Site Recovery** > **Servidores de configuración**.


Tras registrar el servidor de configuración, Site Recovery usa la configuración especificada para conectarse a los servidores de VMware y detecta las máquinas virtuales.

> [!NOTE]
> El nombre de la cuenta podría tardar 15 minutos o más en aparecer en el portal. Para que se actualice inmediatamente, seleccione **Servidores de configuración** > ***nombre de servidor*** > **Actualizar servidor**.

## <a name="set-up-the-target-environment"></a>Configuración del entorno de destino

Seleccione y compruebe los recursos de destino.

1. Seleccione **Preparar infraestructura** > **Destino**. Seleccione la suscripción de Azure que quiere usar. Estamos usando un modelo de Resource Manager.
2. Site Recovery comprueba que tiene una o varias redes virtuales. Debería tenerlos cuando configure los componentes de Azure en el [primer tutorial](tutorial-prepare-azure.md) de esta serie de tutoriales.

   ![Captura de pantalla de las opciones de Preparar infraestructura > Destino.](./media/vmware-azure-tutorial/storage-network.png)

## <a name="create-a-replication-policy"></a>Creación de una directiva de replicación

1. Abra [Azure Portal](https://portal.azure.com). Busque **Almacenes de Recovery Services** y selecciónelo.
2. Seleccione el almacén de Recovery Services (**ContosoVMVault** en este tutorial).
3. Para crear una directiva de replicación, seleccione **Site Recovery Infrastructure (Infraestructura de Site Recovery)**  > **Directivas de replicación** >  **+Directiva de replicación**.
4. En **Crear la directiva de replicación**, especifique el nombre de directiva. Usamos **VMwareRepPolicy**.
5. En **Umbral de RPO**, use el valor predeterminado de 60 minutos. Este valor define la frecuencia de creación de puntos de recuperación. Se genera una alerta cuando la replicación continua supera este límite.
6. En **Retención de punto de recuperación**, especifique cuánto tiempo se conserva cada punto de recuperación. En este tutorial se selecciona 72 horas. Las máquinas virtuales replicadas se pueden recuperar a cualquier momento de una ventana de retención.
7. En **Frecuencia de instantánea coherente con la aplicación**, especifique la frecuencia (en minutos) con la que se crearán puntos de recuperación que contengan las instantáneas coherentes con la aplicación. Usamos el valor predeterminado de 60 minutos. Seleccione **Aceptar** para crear la directiva.

   ![Captura de pantalla de las opciones de Crear directiva de replicación.](./media/vmware-azure-tutorial/replication-policy.png)

- La directiva se asocia automáticamente al servidor de configuración.
- De forma predeterminada, se crea automáticamente una directiva correspondiente para la conmutación por recuperación. Por ejemplo, si la directiva de replicación es **rep-policy**, la directiva de conmutación por recuperación será **rep-policy-failback**. Esta directiva no se usa hasta que se inicie una conmutación por recuperación desde Azure.

Nota: En el escenario de VMware a Azure, la instantánea coherente con el bloqueo se toma a un intervalo de 5 minutos.

## <a name="enable-replication"></a>Habilitar replicación

Habilite la replicación para máquinas virtuales como sigue:

1. Seleccione **Replicar la aplicación** > **Origen**.
2. En **Origen**, seleccione **Local** y seleccione el servidor de configuración en **Ubicación de origen**.
3. En **Tipo de máquina**, seleccione **Máquinas virtuales**.
4. En **vCenter/vSphere Hypervisor**, seleccione el host de vSphere o servidor vCenter que administra el host.
5. Seleccione el servidor de proceso (que se instala de forma predeterminada en la máquina virtual del servidor de configuración). Después, seleccione **Aceptar**. El estado de mantenimiento de cada servidor de procesos se indica según los límites recomendados y otros parámetros. Elija un servidor de procesos correcto. No se puede elegir un servidor de procesos [crítico](vmware-physical-azure-monitor-process-server.md#process-server-alerts). Puede [localizar y solucionar](vmware-physical-azure-troubleshoot-process-server.md) los errores **o** configurar un [servidor de procesos de escalabilidad horizontal](vmware-azure-set-up-process-server-scale.md).
6. En **Destino**, seleccione la suscripción y el grupo de recursos donde desee crear las máquinas virtuales conmutadas por error. Usamos el modelo de implementación de Resource Manager.
7. Seleccione la red y la subred de Azure a la que se conectarán las máquinas virtuales de Azure cuando se creen después de la conmutación por error.
8. Seleccione **Configurar ahora para las máquinas seleccionadas** para aplicar la configuración de red a todas las máquinas habilitadas para replicación. Seleccione **Configurar más tarde** para seleccionar la red de Azure por máquina.
9. En **Máquinas virtuales** > **Seleccionar máquinas virtuales**, seleccione cada máquina que desea replicar. Solo puede seleccionar aquellas máquinas en las que se pueda habilitar la replicación. Después, seleccione **Aceptar**. Si no puede ver o seleccionar alguna máquina virtual en concreto, [aquí encontrará más información](https://aka.ms/doc-plugin-VM-not-showing) para resolver el problema.
10. En **Propiedades** > **Configurar propiedades**, seleccione la cuenta que usará el servidor de procesos para instalar automáticamente Mobility Service en la máquina.
11. En **Configuración de la replicación** > **Establecer configuración de replicación**, compruebe que se ha seleccionado la directiva de replicación correcta.
12. Seleccione **Habilitar replicación**. Site Recovery instala Mobility Service cuando se habilita la replicación para una máquina virtual.
13. Puede hacer un seguimiento del progreso del trabajo **Habilitar protección** en **Configuración** > **Trabajos** > **Trabajos de Site Recovery**. Una vez que se ejecuta el trabajo **Finalizar protección** y se ha completado la generación de un punto de recuperación, la máquina está lista para la conmutación por error.
14. Los cambios pueden tardar 15 minutos o más en aplicarse y aparecer en el portal.
15. Para supervisar las máquinas virtuales que agregue, compruebe la última hora de detección de máquinas virtuales en **Servidores de configuración** > **Último contacto a las**. Para agregar máquinas virtuales sin esperar a la detección programada, resalte el servidor de configuración (no haga clic en él) y haga clic en **Actualizar**.

## <a name="next-steps"></a>Pasos siguientes
Después de habilitar la replicación, realice una exploración en profundidad para comprobar que todo funciona según lo previsto.
> [!div class="nextstepaction"]
> [Exploración de la recuperación ante desastres](site-recovery-test-failover-to-azure.md)
