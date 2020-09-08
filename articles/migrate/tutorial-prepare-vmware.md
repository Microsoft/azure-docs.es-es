---
title: Preparación de máquinas virtuales de VMware para la evaluación y migración con Azure Migrate
description: Aprenda a prepararse para la evaluación y migración de máquinas virtuales de VMware con Azure Migrate.
ms.topic: tutorial
ms.date: 06/08/2020
ms.custom: mvc
ms.openlocfilehash: 8d4d6ac1149c397442a8ca7dd01f46f04ffc89b4
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88927313"
---
# <a name="prepare-vmware-vms-for-assessment-and-migration-to-azure"></a>Preparación de máquinas virtuales de VMware para la evaluación y migración a Azure

Este artículo ayuda a preparar la evaluación y migración de máquinas virtuales VMware locales a Azure mediante [Azure Migrate](migrate-services-overview.md).

Este tutorial es el primero de una serie que muestra cómo evaluar y migrar máquinas virtuales de VMware. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Preparar Azure para trabajar con Azure Migrate.
> * Preparar la evaluación de máquinas virtuales VMware con la herramienta Azure Migrate:Server Assessment.
> * Preparar la migración de máquinas virtuales VMware con la herramienta Azure Migrate:Server Migration. 

> [!NOTE]
> Los tutoriales muestran la ruta de implementación más sencilla para un escenario. Son de utilidad como una prueba de concepto rápida. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.


## <a name="prepare-azure"></a>Preparación de Azure

En la tabla se resumen las tareas que debe completar en Azure. Hay instrucciones para cada tarea a continuación de la tabla.

**Task** | **Detalles** | **Permisos**
--- | --- | ---
**Crear un proyecto de Azure Migrate** | Un proyecto de Azure Migrate proporciona una ubicación central para orquestar y administrar las evaluaciones y migraciones con herramientas de Azure Migrate, herramientas de Microsoft y otras ofertas de terceros. | La cuenta de Azure necesita permisos de colaborador o propietario en el grupo de recursos en el que reside el proyecto.
**Registrar dispositivo** | Azure Migrate usa un dispositivo Azure Migrate ligero para detectar máquinas virtuales, para evaluarlas con la herramienta Server Assessment y para migrarlas mediante la migración sin agente con la herramienta Server Migration. [Más información](migrate-appliance-architecture.md#appliance-registration) sobre el registro. | Para registrar el dispositivo, la cuenta de Azure necesita permisos de colaborador o propietario en la suscripción de Azure.
**Crear aplicaciones de Azure AD** | Al registrar un dispositivo, Azure Migrate crea dos aplicaciones de Azure Active Directory (Azure AD). <br/><br/> - La primera aplicación se usa para la comunicación entre los agentes que se ejecutan en el dispositivo y Azure Migrate. <br/><br/> - La segunda aplicación se usa exclusivamente para acceder a la instancia de KeyVault creada en la suscripción del usuario para la migración de la máquina virtual de VMware sin agente.   | La cuenta de Azure necesita estos [permisos](https://docs.microsoft.com/azure/migrate/tutorial-prepare-vmware#assign-permissions-to-create-azure-ad-apps) para crear aplicaciones de Azure AD.
**Crear un almacén de claves** | - La primera instancia de Key Vault se crea como parte del registro del dispositivo y se usa para la administración del certificado que se descargó en el dispositivo durante su configuración. <br/><br/> \- Para migrar máquinas virtuales VMware mediante la migración sin agente, Azure Migrate crea otra instancia de Key Vault para administrar las claves de acceso a la cuenta de replicación de la suscripción.| Para permitir que Azure Migrate cree el almacén de claves, debe establecer permisos (propietario, colaborador y administrador de acceso de usuario) en el grupo de recursos en el que reside el proyecto de Azure Migrate.


### <a name="assign-permissions-to-create-project"></a>Asignación de permisos para crear un proyecto

1. En Azure Portal, abra la suscripción y seleccione **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y haga clic en ella para ver los permisos.
3. Debe tener permisos de **Colaborador** o **Propietario**.
    - Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción.
    - Si no es el propietario, trabaje con él para asignar el rol.

### <a name="assign-permissions-to-create-azure-ad-apps"></a>Asignación de permisos para crear aplicaciones de Azure AD

La cuenta de Azure necesita permisos para crear aplicaciones de Azure AD para registrar el dispositivo. Asigne los permisos mediante uno de estos métodos:

- **Método 1: conceder permisos a la cuenta**: un administrador de inquilinos o un administrador global pueden conceder permisos a las cuentas de usuario del inquilino para crear y registrar aplicaciones de Azure AD.
- **Método 2: asignar un rol con los permisos a una cuenta de usuario**: un administrador de inquilinos o un administrador global pueden asignar el rol de desarrollador de aplicaciones (que tiene los permisos necesarios) a la cuenta de usuario.

> [!NOTE]
> Solo necesita estos permisos al registrar un nuevo dispositivo. Puede quitar los permisos una vez configurado el dispositivo.


#### <a name="method-1-grant-permissions-to-the-account"></a>Método 1: conceder permisos a la cuenta

Conceda permisos a la cuenta de la siguiente manera:

1. Asegúrese de que es un administrador del inquilino o un administrador global. A continuación, en Azure AD, vaya a **Azure Active Directory** > **Usuarios** > **Configuración de usuario**.
2. Establezca **Registros de aplicaciones** en **Sí**. Se trata de una configuración predeterminada que no es confidencial. [Más información](../active-directory/develop/active-directory-how-applications-are-added.md#who-has-permission-to-add-applications-to-my-azure-ad-instance).

    ![Permisos de Azure AD](./media/tutorial-prepare-vmware/aad.png)



#### <a name="method-2-assign-application-developer-role"></a>Método 2: Asignación del rol de desarrollador de aplicaciones

Como alternativa, el administrador de inquilinos o administrador global puede asignar el rol de desarrollador de aplicaciones a una cuenta. [Más información](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md) sobre la asignación de roles.

### <a name="assign-permissions-to-create-a-key-vault"></a>Asignación de permisos para crear un almacén de claves

Para habilitar Azure Migrate para crear un almacén de claves, asigne los permisos de la siguiente manera:

1. En el grupo de recursos de Azure Portal, seleccione **Control de acceso (IAM**).
2. En **Comprobar acceso**, busque la cuenta correspondiente y haga clic en ella para ver los permisos.

    - Para ejecutar Server Assessment, son suficientes los permisos de **colaborador**.
    - Para ejecutar la migración de un servidor sin agente, debe disponer de permisos de **propietario** (o **colaborador** y **administrador de acceso de usuario**).

3. Si no tiene los permisos necesarios, pídaselos al propietario del grupo de recursos.

## <a name="prepare-for-assessment"></a>Preparación para la evaluación

Para preparar la valoración de máquinas virtuales de VMware, tiene que:

1. **Comprobar la configuración de VMware**. Asegúrese de que las máquinas virtuales y vCenter Server que desea migrar cumplan los requisitos.
2. **Configurar los permisos para la evaluación**. Azure Migrate usa una cuenta de vCenter para acceder a vCenter Server, con el fin de detectar y evaluar máquinas virtuales.
3. **Comprobar los requisitos de los dispositivos**. Compruebe los requisitos de implementación del dispositivo Azure Migrate antes de implementarlo en el siguiente tutorial.

### <a name="verify-vmware-settings"></a>Comprobación de la configuración de VMware

1. [Compruebe los requisitos de VMware](migrate-support-matrix-vmware.md#vmware-requirements) para la evaluación.
2. [Asegúrese](migrate-support-matrix-vmware.md#port-access-requirements) de que los puertos necesarios están abiertos en vCenter Server.
3. En vCenter Server, compruebe que la cuenta tiene permisos para crear una máquina virtual mediante un archivo OVA. Esto es necesario al implementar el dispositivo Azure Migrate como una máquina virtual VMware mediante un archivo OVA.
4. Hay algunos cambios necesarios en las máquinas virtuales antes de migrarlas a Azure.

    - En algunos sistemas operativos, Azure Migrate realiza estos cambios automáticamente. 
    - Es importante realizar estos cambios antes de comenzar la migración. Si migra la máquina virtual antes de realizar el cambio, es posible que la máquina virtual no arranque en Azure.
    - Revise los cambios en [Windows](prepare-for-migration.md#windows-machines) y [Linux](prepare-for-migration.md#linux-machines) que debe realizar.


### <a name="set-up-permissions-for-assessment"></a>Configuración de los permisos para la evaluación

Azure Migrate necesita acceder a vCenter Server para que el dispositivo Azure Migrate pueda detectar máquinas virtuales para su evaluación y migración sin agente. Configure una cuenta de la siguiente manera:

1. En el cliente web de vSphere, abra **Administration** > **Access** > **SSO Users and Groups** (Administración>Acceso>Usuarios y grupos de inicio de sesión único).
2. En **Users** (Usuarios), haga clic en el icono **New User** (Nuevo usuario).
3. Escriba los detalles del nuevo usuario.
4. Seleccione los permisos de acuerdo con los valores de la tabla.

    **Característica** | **Permisos de cuenta**
    --- | ---
    [Evaluar máquinas virtuales](tutorial-assess-vmware.md) | Para la evaluación, la cuenta necesita acceso de solo lectura.
    [Detectar aplicaciones, roles y características de máquinas virtuales](how-to-discover-applications.md) | Si desea usar la detección de aplicaciones, la cuenta de solo lectura que se usa para la evaluación necesita privilegios habilitados para **Máquinas virtuales** > **Operaciones de invitado**.
    [Analizar las dependencias en las máquinas virtuales (sin agente)](how-to-create-group-machine-dependencies-agentless.md) | Si desea analizar las dependencias, la cuenta de solo lectura que se usa para la evaluación necesita privilegios habilitados para **Máquinas virtuales** > **Operaciones de invitado**.
    
> [!NOTE]
> Si quiere limitar la detección de máquinas virtuales para su evaluación a un ámbito específico, revise [este artículo](set-discovery-scope.md#assign-a-role-for-assessment).

### <a name="verify-appliance-settings-for-assessment"></a>Comprobación de la configuración del dispositivo para su evaluación

En el [siguiente tutorial](tutorial-assess-vmware.md), va a configurar el dispositivo Azure Migrate y comenzar la evaluación. Antes de hacerlo, revise los requisitos del dispositivo como se indica a continuación: 

1. [Revise los requisitos](migrate-appliance.md#appliance---vmware) para implementar el dispositivo Azure Migrate.
2. Revise las direcciones URL de Azure a las que el dispositivo necesita acceder en nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).
3. [Anote](migrate-support-matrix-vmware.md#port-access-requirements) los puertos que usa el dispositivo.
4. [Revise los datos](migrate-appliance.md#collected-data---vmware) que recopila el dispositivo durante la detección y evaluación.

## <a name="prepare-for-agentless-vmware-migration"></a>Preparación para la migración de VMware sin agente

Puede migrar máquinas virtuales VMware mediante la [migración basada en agente o sin agente](server-migrate-overview.md). En esta sección se resumen los requisitos de la migración sin agente.

1. [Decida](server-migrate-overview.md#compare-migration-methods) si desea utilizar la migración sin agente.
2. [Revise](migrate-support-matrix-vmware-migration.md#vmware-requirements-agentless) los requisitos del hipervisor de las máquinas que desea migrar.
3. [Revise](migrate-support-matrix-vmware-migration.md#vm-requirements-agentless) los requisitos de las máquinas virtuales VMware que desea mediante la migración sin agente.
4. [Revise](migrate-support-matrix-vmware-migration.md#appliance-requirements-agentless) los requisitos del dispositivo Azure Migrate para la migración sin agente.
5. Tenga en cuenta que es necesario acceder a la dirección URL para las nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).
6. Revise los requisitos de [acceso a los puertos](migrate-support-matrix-vmware-migration.md#port-requirements-agentless).
7. Configure los permisos para la migración sin agente, como se describe en el procedimiento siguiente.


### <a name="assign-permissions-to-an-account"></a>Asignación de permisos a una cuenta

El dispositivo Azure Migrate se conecta a vCenter Server para detectar y migrar máquinas virtuales mediante la migración sin agente. Puede asignar los permisos que necesita el dispositivo a una cuenta de usuario o crear un rol con los permisos y asignar ese rol a una cuenta de usuario.

1. En el cliente web de vSphere, abra **Administration** > **Access** > **SSO Users and Groups** (Administración>Acceso>Usuarios y grupos de inicio de sesión único).
2. En **Users** (Usuarios), haga clic en el icono **New User** (Nuevo usuario).
3. Escriba los detalles del nuevo usuario.
4. Asigne [estos permisos](migrate-support-matrix-vmware-migration.md#vmware-requirements-agentless).

#### <a name="create-a-role-and-assign-to-an-account"></a>Creación de un rol y asignación a una cuenta

Como alternativa, puede crear una cuenta. A continuación, cree un rol y asígnelo a la cuenta, como se indica a continuación:

1. Inicie sesión en el cliente web de Sphere como administrador de vCenter Server.
2. Seleccione la instancia de vCenter Server > **Crear rol**.
3. Especifique un nombre de rol (por ejemplo, <em>Azure_Migrate</em>) y asigne los [permisos necesarios](migrate-support-matrix-vmware-migration.md#vmware-requirements-agentless) al rol.

    ![Privilegios de cuenta de vCenter Server](./media/tutorial-prepare-vmware/vcenter-server-permissions.png)

4. A continuación, cree una cuenta y asigne el rol a la cuenta.

> [!NOTE]
> Si quiere limitar la detección de máquinas virtuales para la migración sin agente a un ámbito específico, revise [este artículo](set-discovery-scope.md#assign-a-role-for-agentless-migration).



## <a name="prepare-for-agent-based-vmware-migration"></a>Preparación para la migración de VMware basada en agente

Puede migrar máquinas virtuales VMware mediante la [migración basada en agente o sin agente](server-migrate-overview.md). En esta sección se resumen los requisitos de la migración basada en agente.

1. [Decida](server-migrate-overview.md#compare-migration-methods) si desea utilizar la migración basada en agente.
1. [Revise](migrate-support-matrix-vmware-migration.md#vmware-requirements-agent-based) los requisitos del hipervisor de las máquinas que desea migrar.
2. [Revise](migrate-support-matrix-vmware-migration.md#vm-requirements-agent-based) los requisitos de las máquinas virtuales de VMware, entre los que se incluyen la instalación de Mobility Service en todas las máquinas virtuales que desee migrar.
3. La migración basada en agente usa una aplicación de replicación:
    - [Revise](migrate-replication-appliance.md#appliance-requirements) los requisitos de implementación de la aplicación de replicación.
    - [Revise las opciones](migrate-replication-appliance.md#mysql-installation) para instalar MySQL en la aplicación.
    - Tenga en cuenta que es necesario acceder a la dirección URL para las nubes [públicas](migrate-replication-appliance.md#url-access) y [gubernamentales](migrate-replication-appliance.md#azure-government-url-access).
    - Revise los requisitos de [acceso a los puertos](migrate-replication-appliance.md#port-access) del dispositivo de replicación.
4. Hay algunos cambios necesarios en las máquinas virtuales antes de poder migrarlas a Azure. Revise los cambios en [Windows](prepare-for-migration.md#windows-machines) y [Linux](prepare-for-migration.md#linux-machines) que debe realizar.



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Configuró permisos de Azure.
> * Preparó VMware para la migración y evaluación.


Continúe con el segundo tutorial para configurar un proyecto de Azure Migrate y evaluar las máquinas virtuales de VMware para la migración a Azure.

> [!div class="nextstepaction"]
> [Evaluación de máquinas virtuales de VMware](./tutorial-assess-vmware.md)
