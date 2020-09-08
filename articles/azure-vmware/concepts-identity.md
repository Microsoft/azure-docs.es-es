---
title: Conceptos sobre identidad y acceso
description: Obtenga información sobre los conceptos de identidad y acceso de Azure VMware Solution
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: 9d5b7b4c7c2e0d55cffc99a3f371494f40320a16
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750590"
---
# <a name="azure-vmware-solution-identity-concepts"></a>Conceptos de identidad en Azure VMware Solution

Un servidor vCenter y NSX-T Manager se aprovisionan cuando se implementa una nube privada. Puede usar vCenter para administrar cargas de trabajo de máquinas virtuales, y NSX-T Manager para ampliar la red definida por software de la nube privada.

Administración de acceso e identidad usa los privilegios del grupo CloudAdmin para vCenter y derechos de administrador restringidos para NSX-T Manager. Esta directiva garantiza que la plataforma de nube privada se pueda actualizar automáticamente. Esto ofrece las características y revisiones más recientes de forma periódica. Consulte el artículo de [conceptos de actualización de nubes privadas][concepts-upgrades] para obtener más detalles sobre las actualizaciones de las nubes privadas.

## <a name="vcenter-access-and-identity"></a>Acceso e identidad de vCenter

Los privilegios en vCenter se proporcionan a través del grupo CloudAdmin. Ese grupo se puede administrar localmente en vCenter, o a través de la integración del inicio de sesión único de vCenter LDAP con Azure Active Directory. Se le proporciona la capacidad de habilitar esa integración después de implementar una nube privada.

Los privilegios de CloudAdmin y CloudGlobalAdmin se muestran en la tabla siguiente.

|  Conjunto de privilegios           | CloudAdmin | CloudGlobalAdmin | Comentario |
| :---                     |    :---:   |       :---:      |   :--:  |
|  Alarmas                  | Un usuario de CloudAdmin tiene todos los privilegios de Alarmas para las alarmas en las VM y Compute-ResourcePool.     |          --        |  -- |
|  Implementación automática             |  --  |        --        |  Microsoft se encarga de la administración de hosts.  |
|  Certificados            |  --  |        --       |  Microsoft se encarga de la administración de certificados.  |
|  Biblioteca de contenido         | Un usuario de CloudAdmin tiene privilegios para crear y usar archivos en una biblioteca de contenido.    |         Habilitado con SSO.         |  Microsoft distribuirá los archivos de la biblioteca de contenido a los hosts ESXi.  |
|  Centro de datos              |  --  |        --          |  Microsoft realiza todas las operaciones de centros de datos.  |
|  Almacén de datos               | Datastore.AllocateSpace, Datastore.Browse, Datastore.Config, Datastore.DeleteFile, Datastore.FileManagement, Datastore.UpdateVirtualMachineMetadata     |    --    |   -- |
|  ESX Agent Manager       |  --  |         --       |  Microsoft realiza todas las operaciones.  |
|  Carpeta                  |  Un usuario de CloudAdmin tiene todos los privilegios de Carpeta.     |  --  |  --  |
|  Global                  |  Global.CancelTask, Global.GlobalTag, Global.Health, Global.LogEvent, Global.ManageCustomFields, Global.ServiceManagers, Global.SetCustomField, Global.SystemTag         |                  |    |
|  Host                    |  Host.Hbr.HbrManagement      |        --          |  Microsoft realiza todas las demás operaciones de Host.  |
|  InventoryService        |  InventoryService.Tagging      |        --          |  --  |
|  Red                 |  Network.Assign    |                  |  Microsoft realiza todas las demás operaciones de Red.  |
|  Permisos             |  --  |        --       |  Microsoft realiza todas las operaciones de Permisos.  |
|  Almacenamiento controlado por perfiles  |  --  |        --       |  Microsoft realiza todas las operaciones de Perfil.  |
|  Resource                |  Un usuario de CloudAdmin tiene todos los privilegios de Recurso.        |      --       | --   |
|  Tarea programada          |  Un usuario de CloudAdmin tiene todos los privilegios de Tarea programada.   |   --   | -- |
|  Sesiones                |  Sessions.GlobalMessage, Sessions.ValidateSession      |   --   |  Microsoft realiza todas las demás operaciones de Sesiones.  |
|  Vistas de almacenamiento           |  StorageViews.View   |        --          |  Microsoft realiza todas las demás operaciones de Vista de almacenamiento (configurar servicio).  |
|  Tareas                   |  --  |  --   |  Microsoft administra extensiones que administran Tareas.  |
|  vApp                    |  Un usuario de CloudAdmin tiene todos los privilegios de vApp.  |  --  |  --  |
|  Máquina virtual         |  Un usuario de CloudAdmin tiene todos los privilegios de Máquina virtual.  |  --  |  --  |
|  vService                |  Un usuario de CloudAdmin tiene todos los privilegios de vService.  |  --  |  --  |

## <a name="nsx-t-manager-access-and-identity"></a>Acceso e identidad de NSX-T Manager

Puede acceder a NSX-T Manager con la cuenta "administrador". Esa cuenta tiene privilegios completos y le permite crear y administrar enrutadores T1, conmutadores lógicos y todos los servicios. Los privilegios completos en NSX-T también le brindan acceso al enrutador de NSX-T T0. Un cambio en el enrutador T0 podría provocar una degradación en el rendimiento de la red o una pérdida de acceso a una nube privada. Para cumplir los requisitos de compatibilidad, es necesario que abra una solicitud de soporte técnico en Azure Portal para solicitar cambios en el enrutador NSX-T T0.
  
## <a name="next-steps"></a>Pasos siguientes

El siguiente paso es obtener información sobre los [conceptos de actualización de nubes privadas][concepts-upgrades].

<!-- LINKS - external -->

<!-- LINKS - internal -->
[concepts-upgrades]: ./concepts-upgrades.md