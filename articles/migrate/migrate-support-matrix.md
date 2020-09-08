---
title: Matriz de compatibilidad para Azure Migrate
description: Proporciona un resumen de limitaciones y configuraciones de compatibilidad para el servicio Azure Migrate.
ms.topic: conceptual
ms.date: 07/23/2020
ms.author: raynew
ms.openlocfilehash: aac7b7d39102a1fe115ddea483aee36af79e7fc4
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88612119"
---
# <a name="azure-migrate-support-matrix"></a>Matriz de compatibilidad para Azure Migrate

Puede usar el [servicio Azure Migrate](./migrate-services-overview.md) para evaluar y migrar máquinas a la nube de Microsoft Azure. En este artículo se resumen las configuraciones y limitaciones de compatibilidad generales para los escenarios e implementaciones de Azure Migrate.

## <a name="supported-assessmentmigration-scenarios"></a>Escenarios de migración o evaluación admitidos

En la tabla se resumen los escenarios de detección, evaluación y migración admitidos.

**Implementación** | **Detalles** 
--- | --- 
**Detección** | Puede detectar metadatos de máquinas y datos de rendimiento dinámicos.
**Detección de aplicaciones** | Puede detectar aplicaciones, roles y características que se ejecutan en máquinas virtuales de VMware. Actualmente, esta característica solo se limita a la detección. La evaluación está actualmente en el nivel de máquina. Todavía no se ofrecen valoraciones basadas en aplicaciones, roles ni características. 
**Valoración** | Evalúe las cargas de trabajo y los datos locales que se ejecutan en máquinas virtuales de VMware, de Hyper-V y en servidores físicos. Evalúe el uso de Azure Migrate Server Assessment y Microsoft Data Migration Assistant (DMA), así como otras herramientas y ofertas de ISV.
**Migración** | Migre las cargas de trabajo y los datos que se ejecutan en servidores físicos, máquinas virtuales de VMware, máquinas virtuales de Hyper-V, servidores físicos y máquinas virtuales basadas en la nube a Azure. Utilice Azure Migrate Server Assessment y Microsoft Database Migration Service (DMS) para realizar la migración, así como otras herramientas y ofertas de ISV.

> [!NOTE]
> Actualmente, las herramientas de ISV no pueden enviar datos a Azure Migrate en Azure Government. Puede usar las herramientas de Microsoft integradas, o bien usar herramientas de asociados de forma independiente.

## <a name="supported-tools"></a>Herramientas admitidas


La compatibilidad con herramientas concretas se resume en la siguiente tabla.

**Herramienta** | **Evaluar** | **Migrar** 
--- | --- | ---
Evaluación de servidores de Azure Migrate | Evaluar [máquinas virtuales de VMware](tutorial-prepare-vmware.md), [máquinas virtuales de Hyper-V](tutorial-prepare-hyper-v.md) y [servidores físicos](tutorial-prepare-physical.md). |  No disponible (N/D)
Migración de servidores de Azure Migrate | N/D | Migrar [máquinas virtuales de VMware](tutorial-migrate-vmware.md), [máquinas virtuales de Hyper-V](tutorial-migrate-hyper-v.md) y [servidores físicos](tutorial-migrate-physical-virtual-machines.md).
[Carbonite](https://www.carbonite.com/data-protection-resources/resource/Datasheet/carbonite-migrate-for-microsoft-azure) | N/D | Migrar máquinas virtuales de VMware, máquinas virtuales de Hyper-V, servidores físicos y cargas de trabajo en la nube pública. 
[Cloudamize](https://www.cloudamize.com/platform#tab-0)| Evaluar máquinas virtuales de VMware, máquinas virtuales de Hyper-V, servidores físicos y cargas de trabajo en la nube pública. | N/D
[Corent Technology](https://go.microsoft.com/fwlink/?linkid=2084928) | Evaluar y migrar máquinas virtuales de VMware, máquinas virtuales de Hyper-V, servidores físicos y cargas de trabajo en la nube pública. |  Migrar máquinas virtuales de VMware, máquinas virtuales de Hyper-V, servidores físicos y cargas de trabajo en la nube pública.
[Device 42](https://go.microsoft.com/fwlink/?linkid=2097158) | Evaluar máquinas virtuales de VMware, máquinas virtuales de Hyper-V, servidores físicos y cargas de trabajo en la nube pública.| N/D
[DMA](/sql/dma/dma-overview?view=sql-server-2017) | Evaluar bases de datos de SQL Server. | N/D
[DMS](../dms/dms-overview.md) | N/D | Migrar SQL Server, Oracle, MySQL, PostgreSQL y MongoDB. 
[Lakeside](https://go.microsoft.com/fwlink/?linkid=2104908) | Evaluar infraestructura de escritorio virtual (VDI) | N/D
[Movere](https://www.movere.io/) | Evaluar máquinas virtuales de VMware, máquinas virtuales de Hyper-V, máquinas virtuales de Xen, máquinas físicas, estaciones de trabajo (incluida VDI) y cargas de trabajo en la nube pública | N/D
[RackWare](https://go.microsoft.com/fwlink/?linkid=2102735) | N/D | Migrar máquinas virtuales de VMWare, máquinas virtuales de Hyper-V, máquinas virtuales de Xen, máquinas virtuales de KVM, máquinas físicas y cargas de trabajo en la nube pública 
[Turbonomic](https://go.microsoft.com/fwlink/?linkid=2094295)  | Evaluar máquinas virtuales de VMware, máquinas virtuales de Hyper-V, servidores físicos y cargas de trabajo en la nube pública. | N/D
[UnifyCloud](https://go.microsoft.com/fwlink/?linkid=2097195) | Evaluar máquinas virtuales de VMware, máquinas virtuales de Hyper-V, servidores físicos, cargas de trabajo en la nube pública y bases de datos de SQL Server. | N/D
[Migration Assistant para aplicaciones web](https://appmigration.microsoft.com/) | Evaluar aplicaciones web | Migrar aplicaciones web.


## <a name="azure-migrate-projects"></a>Proyectos de Azure Migrate

**Soporte técnico** | **Detalles**
--- | ---
Subscription | Puede tener varios proyectos de Azure Migrate en una suscripción.
Permisos de Azure | Necesita permisos de colaborador o propietario en la suscripción para crear un proyecto de Azure Migrate.
Máquinas virtuales de VMware  | Evalúe hasta 35 000 máquinas virtuales de VMware en un único proyecto.
Máquinas virtuales de Hyper-V    | Evalúe hasta 35 000 máquinas virtuales de Hyper-V en un único proyecto.

Un proyecto puede incluir máquinas virtuales de VMware y máquinas virtuales de Hyper-V, hasta los límites de evaluación.

## <a name="azure-permissions"></a>Permisos de Azure

Para que Azure Migrate funcione con Azure, necesita estos permisos antes de empezar a evaluar y migrar máquinas.

**Task** | **Permisos** | **Detalles**
--- | --- | ---
Crear un proyecto de Azure Migrate | La cuenta de Azure necesita permisos para crear un proyecto. | Realice la configuración para [VMware](tutorial-prepare-vmware.md#assign-permissions-to-create-project), [Hyper-V](tutorial-prepare-hyper-v.md#assign-permissions-to-create-project) o [servidores físicos](tutorial-prepare-physical.md#assign-permissions-to-create-project).
Registrar el dispositivo de Azure Migrate| Azure Migrate usa un [dispositivo de Azure Migrate](migrate-appliance.md) ligero para evaluar las máquinas con Azure Migrate Server Assessment y para ejecutar la [migración sin agentes](server-migrate-overview.md) de máquinas virtuales de VMware con Azure Migrate Server Migration. Este dispositivo detecta las máquinas y envía sus metadatos y datos de rendimiento a Azure Migrate.<br/><br/> Durante el registro, los proveedores de registro (Microsoft.OffAzure, Microsoft.Migrate y Microsoft.KeyVault) se registran con la suscripción elegida en el dispositivo, de forma que la suscripción funciona con el proveedor de recursos. Para registrarse, necesita acceso de colaborador o propietario en la suscripción.<br/><br/> **VMware**: durante la incorporación, Azure Migrate crea dos aplicaciones de Azure Active Directory (Azure AD). La primera aplicación comunica los agentes del dispositivo con el servicio Azure Migrate. La aplicación no tiene permisos para realizar llamadas de administración de recursos de Azure u obtener acceso RBAC a los recursos. La segunda aplicación accede a una instancia de Azure Key Vault creada en la suscripción de usuario solo para la migración de VMware sin agente. En la migración sin agente, Azure Migrate crea un almacén de claves para administrar las claves de acceso a la cuenta de almacenamiento de replicación de la suscripción. Tiene acceso RBAC en la instancia de Azure Key Vault (en el inquilino del cliente) cuando se inicia la detección desde el dispositivo.<br/><br/> **Hyper-V**: durante la incorporación. Azure Migrate crea una aplicación de Azure AD. La aplicación comunica los agentes del dispositivo con el servicio Azure Migrate. La aplicación no tiene permisos para realizar llamadas de administración de recursos de Azure u obtener acceso RBAC a los recursos. | Realice la configuración para [VMware](tutorial-prepare-vmware.md#assign-permissions-to-create-azure-ad-apps), [Hyper-V](tutorial-prepare-hyper-v.md#assign-permissions-to-create-azure-ad-apps) o [servidores físicos](tutorial-prepare-physical.md#assign-permissions-to-register-the-appliance).
Crear un almacén de claves para la migración sin agentes de VMware | Cuando se migran VM de VMware con Azure Migrate Server Migration sin agentes, Azure Migrate crea una instancia de Key Vault para administrar las claves de acceso a la cuenta de almacenamiento de replicación de la suscripción. Para crear el almacén, debe establecer permisos (propietario, colaborador y administrador de acceso de usuario) en el grupo de recursos en el que reside el proyecto de Azure Migrate. | [Configure](tutorial-prepare-vmware.md#assign-permissions-to-create-a-key-vault) los permisos.

## <a name="supported-geographies-public-cloud"></a>Ubicaciones geográficas admitidas (nube pública)

Puede crear un proyecto de Azure Migrate en varias zonas geográficas de la nube pública.

- Aunque solo puede crear proyectos en estas zonas geográficas, puede evaluar o migrar las máquinas para otras ubicaciones de destino.
- La geografía del proyecto solo se usa para almacenar los metadatos detectados.
- Cuando cree un proyecto, seleccione una geografía. El proyecto y los recursos relacionados se crean en una de las regiones de la geografía. El servicio Azure Migrate asigna la región.

**Geografía** | **Ubicación de almacenamiento de metadatos**
--- | ---
Asia Pacífico | Este de Asia o Sudeste de Asia
Australia | Este de Australia o Sudeste de Australia
Brasil | Sur de Brasil
Canadá | Centro de Canadá o Este de Canadá
Europa | Norte de Europa y Oeste de Europa
Francia | Centro de Francia
India | Centro de la India o Sur de la India
Japón |  Japón Oriental u Japón Occidental
Corea | Centro de Corea del Sur o Sur de Corea del Sur
Reino Unido | Sur de Reino Unido u Oeste de Reino Unido
Estados Unidos | Centro de EE. UU. u Oeste de EE. UU. 2


## <a name="supported-geographies-azure-government"></a>Ubicaciones geográficas admitidas (Azure Government)

**Task** | **Geografía** | **Detalles**
--- | --- | ---
Crear proyecto | Estados Unidos | Los metadatos se almacenan en las ubicaciones de US Gov Arizona, US Gov Virginia
Evaluación del destino | Estados Unidos | Regiones de destino: US Gov Arizona, US Gov Virginia, US Gov Texas
Replicación de destino | Estados Unidos | Regiones de destino: US DoD (centro), US DoD (este), US Gov Arizona, US Gov Iowa, US Gov Texas, US Gov Virginia


## <a name="vmware-assessment-and-migration"></a>Evaluación y migración de VMware

[Revise](migrate-support-matrix-vmware.md) la matriz de compatibilidad de la evaluación de servidores y migración de servidores de Azure Migrate para máquinas virtuales de VMware.

## <a name="hyper-v-assessment-and-migration"></a>Evaluación y migración de Hyper-V

[Revise](migrate-support-matrix-hyper-v.md) la matriz de compatibilidad de la evaluación de servidores y migración de servidores de Azure Migrate para máquinas virtuales de Hyper-V.



## <a name="azure-migrate-versions"></a>Versiones de Azure Migrate

Existen dos versiones del servicio Azure Migrate:

- **Versión actual**: use esta versión para crear nuevos proyectos de Azure Migrate, detectar evaluaciones locales y organizar valoraciones y migraciones. [Más información](whats-new.md).
- **Versión anterior**: para clientes que usaban la versión anterior de Azure Migrate (solo se admitía la evaluación de VM de VMware locales), ahora debería usar la versión actual. En la versión anterior, no puede crear nuevos proyectos de Azure Migrate ni realizar nuevas detecciones.

## <a name="next-steps"></a>Pasos siguientes

- [Evaluación de las máquinas virtuales de VMware](tutorial-assess-vmware.md) para la migración.
- [Evaluación de las máquinas virtuales de Hyper-V](tutorial-assess-hyper-v.md) para la migración.
