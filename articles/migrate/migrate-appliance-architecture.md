---
title: Arquitectura del dispositivo de Azure Migrate
description: Proporciona información general sobre el dispositivo de Azure Migrate usado en la evaluación y migración del servidor.
ms.topic: conceptual
ms.date: 06/09/2020
ms.openlocfilehash: a83e044acc329572a5f3bfd4856f90379319ba1d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919750"
---
# <a name="azure-migrate-appliance-architecture"></a>Arquitectura del dispositivo de Azure Migrate

En este artículo se describen los procesos y la arquitectura del dispositivo de Azure Migrate. El dispositivo de Azure Migrate es ligero y se implementa de forma local para detectar máquinas virtuales y servidores físicos para su migración a Azure. 

## <a name="deployment-scenarios"></a>Escenarios de implementación

El dispositivo Azure Migrate se usa en los escenarios siguientes.

**Escenario** | **Herramienta** | **Se usa para** 
--- | --- | ---
**Evaluación de máquinas virtuales de VMware** | Server Assessment de Azure Migrate | Detectar máquinas virtuales de VMware.<br/><br/> Detectar aplicaciones y dependencias de la máquina.<br/><br/> Recopilar metadatos de máquinas y de rendimiento para enviarlos a Azure.
**Migración de VM de VMware (sin agentes)** | Server Migration de Azure Migrate | Detectar máquinas virtuales de VMware<br/><br/>  Replicar máquinas virtuales de VMware con la [migración sin agentes](server-migrate-overview.md).
**Evaluación de máquinas virtuales de Hyper-V** | Server Assessment de Azure Migrate | Detectar máquinas virtuales de Hyper-V.<br/><br/> Recopilar metadatos de máquinas y de rendimiento para enviarlos a Azure.
**Máquina física** |  Server Assessment de Azure Migrate |  Detectar servidores físicos.<br/><br/> Recopilar metadatos de máquinas y de rendimiento para enviarlos a Azure.

## <a name="appliance-components"></a>Componentes del dispositivo

El dispositivo tiene una serie de componentes.

- **Aplicación de administración**: una aplicación web para la entrada del usuario durante la implementación del dispositivo. Se usa a la hora de evaluar las máquinas para su migración a Azure.
- **Agente de detección**: un agente que reúne los datos de configuración de la máquina. Se usa a la hora de evaluar las máquinas para su migración a Azure. 
- **Agente recopilador**: un agente que recopila datos de rendimiento. Se usa a la hora de evaluar las máquinas para su migración a Azure.
- **Agente de recuperación de datos (DRA)** : organiza la replicación de VM y coordina la comunicación entre las máquinas replicadas y Azure. Solo se usa al replicar máquinas virtuales de VMware en Azure mediante la migración sin agentes.
- **Puerta de enlace**: Envía los datos replicados a Azure. Solo se usa al replicar máquinas virtuales de VMware en Azure mediante la migración sin agentes.
- **Servicio de actualización automática**: actualiza los componentes del dispositivo (se ejecuta cada 24 horas).



## <a name="appliance-deployment"></a>Implementación del dispositivo

- El dispositivo Azure Migrate se puede configurar mediante una plantilla para [Hyper-V](how-to-set-up-appliance-hyper-v.md) o [VMware](how-to-set-up-appliance-vmware.md), o bien mediante un instalador de scripts de PowerShell para [VMware y Hyper-V](deploy-appliance-script.md) y para [servidores físicos](how-to-set-up-appliance-physical.md). 
- Los requisitos de compatibilidad del dispositivo y los requisitos previos de implementación se resumen en la [matriz de compatibilidad del dispositivo](migrate-appliance.md).


## <a name="appliance-registration"></a>Registro del dispositivo

Durante la configuración del dispositivo, este se registra con Azure Migrate y se producen las acciones resumidas en la siguiente tabla.

**Acción** | **Detalles** | **Permisos**
--- | --- | ---
**Registro de proveedores de recursos** | Estos proveedores de recursos se registran en la suscripción que elija durante la configuración del dispositivo: Microsoft.OffAzure, Microsoft.Migrate y Microsoft.KeyVault.<br/><br/> Al registrar un proveedor de recursos se configura la suscripción para que funcione con este. | Para registrar los proveedores de recursos debe tener el rol colaborador o propietario de la suscripción.
**Creación de una aplicación de Azure AD: comunicación** | Azure Migrate crea una aplicación de Azure Active Directory (Azure AD) que se usa para la comunicación (autenticación y autorización) entre los agentes que se ejecutan en la aplicación y sus respectivos servicios que se ejecutan en Azure.<br/><br/> Esta aplicación no tiene privilegios para realizar llamadas a Azure Resource Manager ni acceso RBAC en ningún recurso. | Necesita [estos permisos](tutorial-prepare-vmware.md#assign-permissions-to-create-azure-ad-apps) para que Azure Migrate cree la aplicación.
**Creación de una aplicación de Azure AD: Key Vault** | Esta aplicación solo se crea para la migración sin agentes de máquinas virtuales de VMware a Azure.<br/><br/> Se usa exclusivamente para acceder a la instancia del almacén de claves creada en la suscripción del usuario para la migración sin agentes.<br/><br/> Tiene acceso RBAC a Azure Key Vault (la instancia creada en el inquilino del cliente) cuando se inicia la detección desde el dispositivo. | Necesita [estos permisos](tutorial-prepare-vmware.md#assign-permissions-to-create-a-key-vault) para que Azure Migrate cree la aplicación.



## <a name="collected-data"></a>Datos recopilados

Los datos recopilados por el cliente para todos los escenarios de implementación se resumen en la [matriz de compatibilidad del dispositivo](migrate-appliance.md).

## <a name="discovery-and-collection-process"></a>Proceso de detección y recopilación

![Architecture](./media/migrate-appliance-architecture/architecture.png)

El dispositivo se comunica con instancias de vCenter Server y hosts o clústeres de Hyper-V mediante el siguiente proceso.

1. **Inicio de la detección**:
    - Cuando inicia la detección en el dispositivo de Hyper-V, se comunica con los hosts de Hyper-V en los puertos 5985 (HTTP) y 5986 (HTTPS) de WinRM.
    - Al iniciar la detección en el dispositivo de VMware, se comunica con vCenter Server en el puerto TCP 443 de forma predeterminada. Si vCenter Server escucha en otro puerto, puede configurarlo en la aplicación web del dispositivo.
2. **Recopilación de metadatos y datos de rendimiento**:
    - El dispositivo usa una sesión del Modelo de información común (CIM) para recopilar datos de VM de Hyper-V del host de Hyper-V en los puertos 5985 y 5986.
    - De forma predeterminada, el dispositivo se comunica con el puerto 443 para recopilar datos de VM de VMware de vCenter Server.
3. **Envío de datos**: El dispositivo envía los datos recopilados a Azure Migrate Server Assessment y Azure Migrate Server Migration a través del puerto SSL 443. El dispositivo puede conectarse a Azure a través de Internet, o puede usar ExpressRoute con emparejamiento público o de Microsoft.
    - Para los datos de rendimiento, el dispositivo recopila los datos de uso en tiempo real.
        - Los datos de rendimiento se recopilan cada 20 segundos para VMware, y cada 30 segundos para Hyper-V, para cada métrica de rendimiento.
        - Los datos recopilados se acumulan para crear un único punto de datos durante 10 minutos.
        - El valor de uso máximo se selecciona desde todos los puntos de datos a 20/30 segundos, y se envía a Azure para el cálculo de evaluación.
        - Según el valor de percentil especificado en las propiedades de evaluación (50/90/95/99), los puntos a diez minutos se clasifican en orden ascendente, y el valor de percentil adecuado se usa para calcular la evaluación.
    - Para Server Migration, el dispositivo empieza a recopilar datos de VM y los replica en Azure.
4. **Evaluación y migración**: Ahora puede crear evaluaciones a partir de los metadatos recopilados por el dispositivo mediante Azure Migrate Server Assessment. Además, también puede empezar a migrar VM de VMware mediante Azure Migrate Server Migration para orquestar la replicación de VM sin agente.





## <a name="appliance-upgrades"></a>Actualizaciones del dispositivo

El dispositivo se actualiza a medida que se actualizan los agentes de Azure Migrate que se ejecutan en el dispositivo. Esto sucede automáticamente porque la actualización automática está habilitada en el dispositivo de forma predeterminada. Puede cambiar esta configuración predeterminada para actualizar los agentes manualmente.

Para desactivar la actualización automática en el registro, establezca la clave "AutoUpdate" que se encuentra en HKEY_LOCAL_MACHINE \SOFTWARE\Microsoft\AzureAppliance en 0 (DWORD).

 

## <a name="next-steps"></a>Pasos siguientes

[Revise](migrate-appliance.md) la matriz de compatibilidad del dispositivo.

