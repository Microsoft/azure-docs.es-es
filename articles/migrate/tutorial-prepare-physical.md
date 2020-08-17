---
title: Preparación de servidores físicos para la evaluación y migración con Azure Migrate
description: Aprenda a prepararse para la evaluación y migración de servidores físicos con Azure Migrate.
ms.topic: tutorial
ms.date: 04/15/2020
ms.custom: mvc
ms.openlocfilehash: 55e6039e5844c575808210cde7ee348f658b40ec
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420794"
---
# <a name="prepare-for-assessment-and-migration-of-physical-servers-to-azure"></a>Preparación de la evaluación y migración de servidores físicos a Azure

En este artículo se indica cómo prepararse para la evaluación de servidores físicos en el entorno local con [Azure Migrate](migrate-services-overview.md).

[Azure Migrate](./migrate-services-overview.md) proporciona un centro de herramientas que le ayuda a detecta las aplicaciones, la infraestructura y las cargas de trabajo, a evaluarlas y a migrarlas a Microsoft Azure. Este centro incluye herramientas de Azure Migrate y ofertas de fabricantes de software independientes (ISV) de terceros. 

Este tutorial es el primero de una serie que muestra cómo evaluar servidores físicos con Azure Migrate. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Prepare Azure. Configure los permisos de la cuenta y los recursos de Azure para poder usar Azure Migrate.
> * Preparar los servidores físicos en el entorno local para la evaluación de servidores.


> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles. Para instrucciones detalladas, consulte los procedimientos para la evaluación de servidores físicos.


Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.


## <a name="prepare-azure-for-server-assessment"></a>Preparación de Azure para la evaluación del servidor

Configure Azure para que funcione con Azure Migrate. 

**Task** | **Detalles** 
--- | --- 
**Crear un proyecto de Azure Migrate** | La cuenta de Azure necesita permisos de colaborador o propietario para crear un proyecto. 
**Registro de proveedores de recursos (solo evaluación)** | Azure Migrate usa un dispositivo ligero propio para detectar y evaluar máquinas con Azure Migrate Server Assessment.<br/><br/> Durante el registro de la aplicación, los proveedores de recursos se registran con la suscripción elegida en la aplicación. [Más información](migrate-appliance-architecture.md#appliance-registration).<br/><br/> Para registrar los proveedores de recursos debe tener el rol colaborador o propietario de la suscripción.
**Creación de aplicaciones de Azure AD (solo evaluación)** | Al registrar el dispositivo, Azure Migrate crea una aplicación de Azure Active Directory (Azure AD) que se usa para la comunicación entre los agentes que se ejecutan en el dispositivo con sus respectivos servicios que se ejecutan en Azure. [Más información](migrate-appliance-architecture.md#appliance-registration).<br/><br/> Para crear aplicaciones de Azure AD, se necesitan permisos (disponibles en el Desarrollador de aplicaciones).


### <a name="assign-permissions-to-create-project"></a>Asignación de permisos para crear un proyecto 

Compruebe que tenga permiso para crear un proyecto de Azure Migrate.

1. En Azure Portal, abra la suscripción y seleccione **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y haga clic en ella para ver los permisos.
3. Debe tener permisos de **Colaborador** o **Propietario**.
    - Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción.
    - Si no es el propietario, trabaje con él para asignar el rol.


### <a name="assign-permissions-to-register-the-appliance"></a>Asignación de permisos para registrar el dispositivo 

Puede asignar permisos para Azure Migrate con el fin de crear las aplicaciones de Azure AD durante el registro del dispositivo mediante uno de los métodos siguientes:

- Un administrador de inquilinos o administrador global puede conceder permisos a los usuarios del inquilino para crear y registrar aplicaciones de Azure AD.
- Un administrador de inquilinos o administrador global puede asignar el rol de desarrollador de aplicaciones (que tiene los permisos) a la cuenta.

> [!NOTE]
> - Las aplicaciones no tienen otros permisos de acceso en la suscripción distintos de los descritos anteriormente.
> - Solo necesita estos permisos al registrar un nuevo dispositivo. Puede quitar los permisos una vez configurado el dispositivo.


#### <a name="grant-account-permissions"></a>Concesión de permisos de cuenta

El administrador de inquilinos o global puede conceder permisos como se indica:

1. En Azure AD, el administrador de inquilinos o global debe ir a **Azure Active Directory** > **Usuarios** > **Configuración de usuario**.
2. El administrador debe establecer **Registros de aplicaciones** en **Sí**.

    ![Permisos de Azure AD](./media/tutorial-prepare-hyper-v/aad.png)

> [!NOTE]
> Se trata de una configuración predeterminada que no es confidencial. [Más información](../active-directory/develop/active-directory-how-applications-are-added.md#who-has-permission-to-add-applications-to-my-azure-ad-instance).

#### <a name="assign-application-developer-role"></a>Asignación del rol de desarrollador de aplicaciones

El administrador de inquilinos o administrador global puede asignar el rol de desarrollador de aplicaciones a una cuenta. [Más información](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md).


## <a name="prepare-azure-for-physical-server-migration"></a>Preparación de Azure para la migración del servidor físico

Prepare Azure para migrar los servidores físicos mediante Server Migration.

**Task** | **Detalles**
--- | ---
**Crear un proyecto de Azure Migrate** | La cuenta de Azure necesita permisos de colaborador o propietario para crear un proyecto.
**Comprobación de los permisos de la cuenta de Azure** | Su cuenta de Azure necesita permisos para crear una máquina virtual y escribir en un disco administrado de Azure.
**Creación de una red de Azure** | Configure una red en Azure.


### <a name="assign-permissions-to-create-project"></a>Asignación de permisos para crear un proyecto

1. En Azure Portal, abra la suscripción y seleccione **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y haga clic en ella para ver los permisos.
3. Debe tener permisos de **Colaborador** o **Propietario**.
    - Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción.
    - Si no es el propietario, trabaje con él para asignar el rol.


### <a name="assign-azure-account-permissions"></a>Asignación de los permisos de la cuenta de Azure

Asigne el rol de colaborador de la máquina virtual a la cuenta de Azure. Este rol proporciona permisos para:
  - Crear una máquina virtual en el grupo de recursos seleccionado.
  - Crear una máquina virtual en la red virtual seleccionada.
  - Escribir en un disco administrado de Azure. 

### <a name="create-an-azure-network"></a>Creación de una red de Azure

[Configure](../virtual-network/manage-virtual-network.md#create-a-virtual-network) una red virtual de Azure. Al realizar la replicación en Azure, se crean máquinas virtuales de Azure y se unen a la red virtual de Azure que se especifica al configurar la migración.


## <a name="prepare-for-physical-server-assessment"></a>Preparación de la valoración de servidores físicos

Para preparar la valoración de los servidores físicos, tiene que comprobar su configuración así como la de la implementación de los dispositivos.

### <a name="verify-physical-server-settings"></a>Comprobación de la configuración de los servidores físicos

1. Compruebe los [requisitos de los servidores físicos](migrate-support-matrix-physical.md#physical-server-requirements) para la valoración de los servidores.
2. Asegúrese de que los [puertos necesarios](migrate-support-matrix-physical.md#port-access) están abiertos en los servidores físicos.


### <a name="verify-appliance-settings"></a>Comprobación de la configuración del dispositivo

Antes de configurar el dispositivo de Azure Migrate y comenzar la evaluación en el siguiente tutorial, prepare la implementación del dispositivo.

1. [Compruebe](migrate-appliance.md#appliance---physical) los requisitos del dispositivo para los servidores físicos.
2. Revise las direcciones URL de Azure a las que el dispositivo necesita acceder en nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).
3. [Revise](migrate-appliance.md#collected-data---vmware) lo que el dispositivo va a recopilar durante la detección y la evaluación.
4. [Consulte](migrate-support-matrix-physical.md#port-access) los requisitos de acceso a los puertos para la evaluación del servidor físico.


### <a name="set-up-an-account-for-physical-server-discovery"></a>Configuración de una cuenta para la detección de servidores físicos

Azure Migrate necesita permisos para detectar los servidores en el entorno local.

- **Windows:** Necesita una cuenta de dominio para detectar los servidores unidos a un dominio y una cuenta local para detectar las máquinas que no están unidas a ningún dominio. Debe agregar la cuenta de usuario a estos grupos: Usuarios de administración remota, Usuarios de Monitor de rendimiento y Usuarios del registro de rendimiento.
- **Linux:** Necesita una cuenta raíz en los servidores Linux que desee detectar.

## <a name="prepare-for-physical-server-migration"></a>Preparación para la migración de servidores físicos

Revise los requisitos para la migración de servidores físicos.

> [!NOTE]
> Al migrar máquinas físicas, Azure Migrate:Server Migration emplea la misma arquitectura de replicación que la recuperación ante desastres basada en agente del servicio Azure Site Recovery, y algunos componentes comparten el mismo código base. Puede que algún contenido se vincule a la documentación de Site Recovery.

1. [Revise](migrate-support-matrix-physical-migration.md#physical-server-requirements) requisitos del servidor físico para la migración.
2. Azure Migrate:Server Migration emplea un servidor de replicación para la migración del servidor físico:
    - [Revise](migrate-replication-appliance.md#appliance-requirements) los requisitos de implementación para el dispositivo de replicación y las [opciones](migrate-replication-appliance.md#mysql-installation) para instalar MySQL en el dispositivo.
    - Revise las [direcciones URL de Azure](migrate-appliance.md#url-access) necesarias para que el dispositivo de replicación acceda a las nubes públicas y gubernamentales.
    - Revise los requisitos de acceso [port] (migrate-replication-appliance.md#port-access) del dispositivo de replicación.
3. Hay algunos cambios necesarios en las máquinas virtuales antes de migrarlas a Azure.
    - Es importante realizar estos cambios antes de comenzar la migración. Si migra la máquina virtual antes de realizar el cambio, es posible que la máquina virtual no arranque en Azure.
    - Revise los cambios en [Windows](prepare-for-migration.md#windows-machines) y [Linux](prepare-for-migration.md#linux-machines) que debe realizar.



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Configurar los permisos de la cuenta de Azure.
> * Preparar los servidores físicos para la evaluación.

Continúe con el siguiente tutorial para crear un proyecto de Azure Migrate y evaluar los servidores físicos para la migración a Azure

> [!div class="nextstepaction"]
> [Evaluación de servidores físicos](./tutorial-assess-physical.md)
