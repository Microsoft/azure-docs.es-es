---
title: Implementación del servidor de configuración en Azure Site Recovery
description: En este artículo se describe cómo implementar un servidor de configuración para realizar la recuperación ante desastres de VMware con Azure Site Recovery
services: site-recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 10/15/2019
ms.author: ramamill
ms.openlocfilehash: f6c47d4cbfe6311333d95b07c0553afa2b3bb15c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287743"
---
# <a name="deploy-a-configuration-server"></a>Implementar un servidor de configuración

Cuando se usa [Azure Site Recovery](site-recovery-overview.md) para realizar la recuperación ante desastres de servidores físicos y máquinas virtuales de VMware en Azure, se implementa un servidor de configuración local. El servidor de configuración coordina la comunicación entre Azure y VMware local. También administra la replicación de datos. Este artículo lo guiará por los pasos necesarios para implementar el servidor de configuración a la hora de replicar máquinas virtuales VMware en Azure. Si necesita configurar un servidor de configuración para la replicación de servidores físicos, consulte [Configuración del servidor de configuración para la recuperación ante desastres de servidores físicos en Azure](physical-azure-set-up-source.md).

> [!TIP]
> Para obtener información sobre el rol de un servidor de configuración como parte de la arquitectura de Azure Site Recovery, consulte [Arquitectura de recuperación ante desastres de VMware a Azure](vmware-azure-architecture.md).

## <a name="deploy-a-configuration-server-through-an-ova-template"></a>Implementación de un servidor de configuración mediante una plantilla OVA

El servidor de configuración debe configurarse como una máquina virtual de VMware de alta disponibilidad con ciertos requisitos mínimos de hardware y tamaño. Para realizar una implementación cómoda y fácil, Site Recovery proporciona una plantilla de OVA (Open Virtualization Application) descargable para configurar el servidor de configuración que cumple con todos los requisitos obligatorios que se enumeran en este documento.

## <a name="prerequisites"></a>Prerrequisitos

En las secciones siguientes se resumen los requisitos mínimos de hardware necesarios para un servidor de configuración.

[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-configuration-and-scaleout-process-server-requirements.md)]

## <a name="azure-active-directory-permission-requirements"></a>Requisitos de permisos de Azure Active Directory

Necesita un usuario con uno de los siguientes permisos establecidos en Azure AD (Azure Active Directory) para registrar el servidor de configuración con los servicios de Azure Site Recovery.

1. El usuario debe tener un rol de desarrollador de aplicaciones para crear una aplicación.
    - Para comprobarlo, inicie sesión en Azure Portal.</br>
    - Vaya a **Azure Active Directory** > **Roles y administradores**.</br>
    - Compruebe si el rol de desarrollador de aplicaciones está asignado al usuario. Si no lo está, utilice un usuario con este permiso o póngase en contacto con un [administrador para que habilite el permiso](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md#assign-roles).
    
2. Si no se puede asignar el rol de desarrollador de aplicaciones, asegúrese de que la marca **Los usuarios pueden registrar aplicaciones** está establecida en **true** para que el usuario pueda crear la identidad. Para habilitar estos permisos, siga estos pasos:
    - Inicie sesión en Azure Portal.
    - Vaya a **Azure Active Directory** > **Configuración de usuario**.
    - En **Registros de aplicaciones**, **Los usuarios pueden registrar aplicaciones**, seleccione **Sí**.

      ![AD_application_permission de Azure](media/vmware-azure-deploy-configuration-server/AAD_application_permission.png)

> [!NOTE]
> Los Servicios de federación de Active Directory *no se admiten*. Use una cuenta administrada mediante [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md).

## <a name="download-the-template"></a>Descarga de la plantilla

1. En el almacén, vaya a **Preparar infraestructura** > **Origen**.
2. En **Preparar origen**, seleccione **+Servidor de configuración**.
3. En **Agregar servidor**, compruebe que aparezca **Servidor de configuración para VMware** en **Tipo de servidor**.
4. Descargue la plantilla OVA para el servidor de configuración.

   > [!TIP]
   >También puede descargar la versión más reciente de la plantilla del servidor de configuración directamente desde el [Centro de descarga de Microsoft](https://aka.ms/asrconfigurationserver).

> [!NOTE]
> La licencia que se proporciona con la plantilla OVA es una licencia de evaluación válida durante 180 días. Después de este período, debe adquirir una licencia.

## <a name="import-the-template-in-vmware"></a>Importación de la plantilla en VMware

1. Inicie sesión en el servidor de VMware vCenter o en el host de vSphere ESXi con el cliente de VMware vSphere.
2. En el menú **File** (Archivo), seleccione **Deploy OVF Template** (Implementar plantilla OVF) para iniciar el **Asistente para implementar plantillas OVF**.

     ![Implementación de la plantilla OVF](./media/vmware-azure-deploy-configuration-server/vcenter-wizard.png)

3. En **Select source** (Seleccionar origen), especifique la ubicación de la plantilla OVF descargada.
4. En **Review details** (Revisar detalles), seleccione **Next** (Siguiente).
5. En **Select name and folder** (Seleccionar el nombre y la carpeta) y **Select configuration** (Seleccionar la configuración), acepte la configuración predeterminada.
6. En **Select storage** (Seleccionar almacenamiento), seleccione **Thick Provision Eager Zeroed** (Aprovisionamiento grueso diligente con ceros) en **Select virtual disk format** (Seleccionar formato de disco virtual) para obtener un rendimiento mejor. El uso de la opción de aprovisionamiento fino puede afectar al rendimiento del servidor de configuración.
7. En el resto de las páginas del asistente, acepte la configuración predeterminada.
8. En **Ready to complete** (Listo para completar):

    * Para configurar la máquina virtual con la configuración predeterminada, seleccione **Power on after deployment** > **Finish** (Encender después de la implementación > Finalizar).
    * Para agregar una interfaz de red adicional, desactive **Power on after deployment** (Encender después de la implementación) y seleccione **Finish** (Finalizar). De forma predeterminada, la plantilla del servidor de configuración se implementa con una sola NIC. Puede agregar NIC adicionales después de la implementación.

> [!IMPORTANT]
> No cambie las configuraciones de los recursos (como la restricción de memoria, núcleos o CPU) ni modifique o elimine los servicios o archivos instalados en el servidor de configuración después de la implementación. Estos tipos de cambios afectan al registro del servidor de configuración con los servicios de Azure y al rendimiento del servidor de configuración.

## <a name="add-an-additional-adapter"></a>Incorporación de un adaptador adicional

> [!NOTE]
> Si planea conservar las direcciones IP de las máquinas de origen en conmutación por error y desea realizar la conmutación por recuperación en el entorno local, se necesitan dos NIC. Una NIC se conectará a las máquinas de origen y la otra se usará para la conectividad de Azure.

Si desea agregar una NIC adicional al servidor de configuración, hágalo antes de registrar el servidor en el almacén. No se admiten adaptadores adicionales después del registro.

1. En el inventario de cliente de vSphere, haga clic con el botón derecho en la máquina virtual y seleccione **Editar configuración**.
2. En **Hardware**, seleccione **Agregar** > **Adaptador Ethernet**. Luego, seleccione **Siguiente**.
3. Seleccione el tipo de adaptador y la red.
4. Para conectar la NIC virtual al encender la máquina virtual, seleccione **Connect at power on** (Conectar al encender). Después, seleccione **Siguiente** > **Finalizar** > **Aceptar**.

## <a name="register-the-configuration-server-with-azure-site-recovery-services"></a>Registrar el servidor de configuración con los servicios de Azure Site Recovery

1. Desde la consola de cliente de VMware vSphere, active la máquina virtual.
2. La máquina virtual se inicia en una experiencia de instalación de Windows Server 2016. Acepte el contrato de licencia y especifique una contraseña de administrador.
3. Una vez finalizada la instalación, inicie sesión en la máquina virtual como administrador.
4. La primera vez que inicie sesión, se inicia la herramienta de configuración de Azure Site Recovery en unos segundos.
5. Especifique el nombre que se usará para registrar el servidor de configuración en Site Recovery. Luego, seleccione **Siguiente**.
6. La herramienta comprueba que la máquina virtual pueda conectarse a Azure. Una vez establecida la conexión, haga clic en **Iniciar sesión** para iniciar sesión en la suscripción de Azure.</br>
    a. Las credenciales deben tener acceso al almacén donde desea registrar el servidor de configuración.</br>
    b. Asegúrese de que la cuenta de usuario elegida tiene permisos para crear una aplicación en Azure. Para habilitar los permisos necesarios, siga las instrucciones de la sección [Requisitos de permisos de Azure Active Directory](#azure-active-directory-permission-requirements).
7. La herramienta realiza algunas tareas de configuración y, a continuación, se reinicia.
8. Inicie sesión de nuevo en la máquina. En pocos segundos, el Asistente para administración del servidor de configuración se inicia automáticamente.

### <a name="configure-settings"></a>Definición de configuración

1. En el Asistente para administración del servidor de configuración, seleccione **Setup connectivity** (Conectividad de configuración). En los cuadros desplegables, seleccione primero la NIC que el servidor de procesos integrado utiliza para la detección y la instalación de extracción de Mobility Service en las máquinas de origen. A continuación, seleccione la NIC que el servidor de configuración usa para la conectividad con Azure. Seleccione **Guardar**. Una vez configurada, esta opción no se puede cambiar. No cambie la dirección IP del servidor de configuración. Asegúrese de que la dirección IP asignada al servidor de configuración sea una dirección IP estática y no una dirección IP de DHCP.
2. En **Selección de almacén de Recovery Services**, inicie sesión en Microsoft Azure con las credenciales usadas en el paso 6 de [Registrar el servidor de configuración con los servicios de Azure Site Recovery](#register-the-configuration-server-with-azure-site-recovery-services).
3. Después de iniciar sesión, seleccione la suscripción de Azure y el grupo de recursos y el almacén correspondientes.

    > [!NOTE]
    > Una vez registrado, no hay ninguna flexibilidad para cambiar el almacén de Recovery Services.
    > El cambio del almacén de Recovery Services requiere desasociar el servidor de configuración del almacén actual y provoca la detención de la replicación de todas las máquinas virtuales protegidas en el servidor de configuración. Para obtener más información, consulte [Administración del servidor de configuración para la recuperación ante desastres de la VM de VMware](vmware-azure-manage-configuration-server.md#register-a-configuration-server-with-a-different-vault).

4. En **Instalar software de terceros**, siga estos pasos:

    |Escenario   |Pasos a seguir  |
    |---------|---------|
    |¿Puedo descargar e instalar MySQL manualmente?     |  Sí. Descargue la aplicación MySQL y colóquela en la carpeta **C:\Temp\ASRSetup**; luego, instálela manualmente. Cuando acepte los términos y seleccione **Descargar e instalar**, el portal indicará que *ya está instalado*. Siga con el paso siguiente.       |
    |¿Se puede evitar la descarga de MySQL en línea?     |   Sí. Coloque la aplicación del instalador de MySQL en la carpeta **C:\Temp\ASRSetup**. Acepte los términos, seleccione **Descargar e instalar** y el portal usará el instalador que ha agregado para instalar la aplicación. Una vez finalizada la instalación, continúe con el paso siguiente.    |
    |Me gustaría descargar e instalar MySQL a través de Azure Site Recovery.    |  Acepte el contrato de licencia y haga clic en **Descargar e instalar**. Una vez finalizada la instalación, continúe con el paso siguiente.       |

5. En **Validar configuración de dispositivo**, se comprueban los requisitos previos antes de continuar.
6. En **Configurar vCenter Server/vSphere ESXi server** (Configurar vCenter Server/vSphere ESXi Server), especifique la dirección IP o FQDN del servidor vCenter o host de vSphere en los que se encuentran las máquinas virtuales donde desea realizar la replicación. Especifique el puerto en el que escucha el servidor. Escriba un nombre descriptivo que se usará para el servidor de VMware en el almacén.
7. Especifique las credenciales que usará el servidor de configuración para conectarse al servidor de VMware. Site Recovery usa estas credenciales para detectar automáticamente las máquinas virtuales de VMware disponibles para la replicación. Seleccione **Add** > **Continue** (Agregar > Continuar). Las credenciales especificadas se guardan localmente.
8. En **Configure virtual machine credentials** (Configuración de credenciales de máquina virtual), especifique el nombre de usuario y la contraseña de las máquinas virtuales para instalar automáticamente Mobility Service durante la replicación. Para las máquinas **Windows**, la cuenta necesita privilegios de administrador local en las máquinas que se vayan a replicar. Para **Linux**, proporcione los detalles de la cuenta raíz.
9. Seleccione **Finalize configuration** (Terminar configuración) para completar el registro.
10. Una vez concluido el registro, abra Azure Portal, compruebe que el servidor de configuración y el servidor de VMware se muestran en **Almacén de Recovery Services** > **Administrar** > **Infraestructura de Site Recovery** > **Servidores de configuración**.

## <a name="upgrade-the-configuration-server"></a>Actualización del servidor de configuración

Para actualizar el servidor de configuración a la versión más reciente, consulte [Administración del servidor de configuración para la recuperación ante desastres de la VM de VMware](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server). Para obtener instrucciones sobre cómo actualizar todos los componentes de Site Recovery, consulte [Actualizaciones de servicio en Azure Site Recovery](service-updates-how-to.md).

## <a name="manage-the-configuration-server"></a>Administración del servidor de configuración

Para evitar interrupciones en la replicación en curso, asegúrese de que la dirección IP del servidor de configuración no cambie cuando el servidor de configuración esté registrado en un almacén. Para obtener más información sobre las tareas comunes de administración del servidor de configuración, consulte [Administración del servidor de configuración para la recuperación ante desastres de la VM de VMware](vmware-azure-manage-configuration-server.md).

## <a name="troubleshoot-deployment-issues"></a>Solución de problemas de implementación

Consulte el [artículo Solucionar problemas](vmware-azure-troubleshoot-configuration-server.md) para resolver problemas de conectividad e implementación.

## <a name="faqs"></a>Preguntas más frecuentes

* ¿Cuánto dura la licencia proporcionada en el servidor de configuración implementado a través de OVF? ¿Qué ocurre si no reactivo la licencia?

    La licencia que se proporciona con la plantilla OVA es una licencia de evaluación válida durante 180 días. Antes de que expire, debe activar la licencia. En caso contrario, puede provocar que la configuración del servidor se detenga y, por lo tanto, se entorpezcan las actividades de replicación. Para obtener más información, consulte [Administración del servidor de configuración para la recuperación ante desastres de la VM de VMware](vmware-azure-manage-configuration-server.md#update-windows-license).

* ¿Puedo usar la máquina virtual donde está instalado el servidor de configuración para otros fines?

    No. Use la máquina virtual solo para el servidor de configuración. Asegúrese de seguir todas las especificaciones mencionadas en [Requisitos previos](#prerequisites) para administrar eficazmente la recuperación ante desastres.
* ¿Puedo cambiar el almacén ya registrado en el servidor de configuración por un almacén recién creado?

    No. Una vez que un almacén está registrado con el servidor de configuración, no se puede cambiar.
* ¿Puedo usar el mismo servidor de configuración para proteger las máquinas físicas y virtuales?

    Sí. Puede usarse el mismo servidor de configuración para la replicación de máquinas físicas y virtuales. Pero solo se puede realizar la conmutación por recuperación de una máquina física a una máquina virtual de VMware.
* ¿Cuál es el propósito de un servidor de configuración y dónde se usa?

    Consulte la [arquitectura de replicación de VMware a Azure](vmware-azure-architecture.md) para obtener más información sobre el servidor de configuración y sus funciones.
* ¿Dónde puedo encontrar la versión más reciente del servidor de configuración?

    Para obtener los pasos de actualización del servidor de configuración a través del portal, vea [Actualización del servidor de configuración](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server). Para obtener instrucciones sobre cómo actualizar todos los componentes de Site Recovery, consulte [Actualizaciones de servicio en Azure Site Recovery](https://aka.ms/asr_how_to_upgrade).
* ¿Dónde puedo descargar la frase de contraseña para el servidor de configuración?

    Para descargar la frase de contraseña, consulte [Administración del servidor de configuración para la recuperación ante desastres de la VM de VMware](vmware-azure-manage-configuration-server.md#generate-configuration-server-passphrase).
* ¿Se puede cambiar la frase de contraseña?

    No. No cambie la frase de contraseña del servidor de configuración. El cambio de la frase de contraseña interrumpe la replicación de las máquinas protegidas y lleva a un estado de mantenimiento crítico.
* ¿Dónde puedo descargar las claves de registro del almacén?

    En **Almacén de Recovery Services**, seleccione **Administrar** > **Infraestructura de Site Recovery** > **Servidores de configuración**. En **Servers** (Servidores), seleccione **Download registration key** (Descargar clave de registro) para descargar el archivo de credenciales de almacén.
* ¿Puedo clonar un servidor de configuración y usarlo para la orquestación de replicación?

    No. No se admite el uso de componentes del servidor de configuración clonados. La clonación del servidor de procesos de escalabilidad horizontal también es un escenario no admitido. La clonación de componentes de Site Recovery afecta a las replicaciones en curso.

* ¿Puedo cambiar la dirección IP del servidor de configuración?

    No. No cambie la dirección IP del servidor de configuración. Asegúrese de que todas las direcciones IP asignadas al servidor de configuración sean direcciones IP estáticas y no direcciones IP de DHCP.
* ¿Puedo configurar el servidor de configuración en Azure?

    Configure el servidor de configuración en el entorno local con una línea de vista directa con vCenter para reducir las latencias de transferencia de datos. Puede realizar copias de seguridad programadas del servidor de configuración con [fines de conmutación por recuperación](vmware-azure-manage-configuration-server.md#failback-requirements).

* ¿Puedo cambiar el controlador de caché en un servidor de configuración o en un servidor de procesos de escalabilidad horizontal?

    No, el controlador de caché no se puede cambiar una vez que se haya configurado.

Para ver más preguntas más frecuentes sobre los servidores de configuración, consulte [Preguntas comunes sobre el servidor de configuración](vmware-azure-common-questions.md#configuration-server).

## <a name="next-steps"></a>Pasos siguientes

Configure la recuperación ante desastres en Azure para [máquinas virtuales VMware](vmware-azure-tutorial.md).
