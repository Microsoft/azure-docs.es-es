---
title: Establezca el ámbito para la detección de máquinas virtuales de VMware con Azure Migrate
description: Describe cómo establecer el ámbito de detección para la evaluación y la migración de máquinas virtuales de VMware con Azure Migrate.
ms.topic: how-to
ms.date: 06/09/2020
ms.openlocfilehash: dfc9c12edd93fc720ef716fd43b04e0c193d5803
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919733"
---
# <a name="set-discovery-scope-for-vmware-vms"></a>Establecimiento del ámbito de detección para las máquinas virtuales de VMware

En este artículo se describe cómo limitar el ámbito de las VM de VMware que detecta cuando realiza lo siguiente:

- La detección de VM con la [aplicación de Azure Migrate](migrate-appliance-architecture.md) cuando se usa la herramienta Azure Migrate:Server Assessment.
- La detección de VM con el dispositivo de [Azure Migrate](migrate-appliance-architecture.md) cuando se usa la herramienta Azure Migrate:Server Migration para la migración sin agente de VM de VMware a Azure.

Al configurar el dispositivo, se conecta a vCenter Server e inicia la detección. Antes de conectar el dispositivo a vCenter Server, puede limitar la detección a centros de datos, clústeres, una carpeta de clústeres, hosts, una carpeta de hosts o VM concretas de vCenter Server. Para establecer el ámbito, asigne permisos en la cuenta que usa el dispositivo para tener acceso a vCenter Server.

## <a name="before-you-start"></a>Antes de comenzar

Si no ha configurado una cuenta de usuario de vCenter que Azure Migrate usa para la detección, hágalo ahora para la [evaluación](tutorial-prepare-vmware.md#set-up-permissions-for-assessment) o la [migración sin agente](tutorial-prepare-vmware.md#assign-permissions-to-an-account).


## <a name="assign-permissions-and-roles"></a>Asignación de permisos y roles

Puede asignar permisos en objetos de inventario VMware mediante uno de estos dos métodos:

- En la cuenta usada por el dispositivo, asigne un rol con los permisos necesarios en los objetos que desee incluir en el ámbito.
- Como alternativa, puede asignar un rol a la cuenta en el nivel de centro de trabajo y propagarlo a los objetos secundarios. A continuación, conceda a la cuenta el rol **Sin acceso** para cada objeto que no quiera incluir en el ámbito. No se recomienda este enfoque porque resulta complicado y podría exponer los controles de acceso, ya que a cada objeto secundario nuevo se le concede automáticamente acceso heredado del objeto primario.

No se puede limitar el ámbito de la detección de inventario en el nivel de carpeta de VM de vCenter. Si necesita limitar el ámbito de la detección a las VM de una carpeta de VM, cree un usuario y conceda acceso de forma individual a cada VM requerida. Se admiten las carpetas de clúster y de host.


### <a name="assign-a-role-for-assessment"></a>Asignación de un rol para su evaluación

1. En la cuenta de la aplicación de vCenter que usa para la detección, aplique el rol de **Solo lectura** para todos los objetos primarios que hospedan VM que quiere detectar y evaluar (host, clúster, carpeta de hosts, carpeta de clústeres, hasta centro de trabajo).
2. Propague estos permisos a los objetos secundarios de la jerarquía.

    ![Asignación de permisos](./media/tutorial-assess-vmware/assign-perms.png)

### <a name="assign-a-role-for-agentless-migration"></a>Asignación de un rol para la migración sin agente

1. En la cuenta de la aplicación de vCenter que usa para la migración, aplique un rol definido por el usuario que tenga los [permisos necesarios](migrate-support-matrix-vmware-migration.md#vmware-requirements-agentless) para todos los objetos primarios que hospedan las VM que quiere detectar y migrar.
2. Puede asignar un nombre al rol con algo que sea más fácil de identificar. Por ejemplo, <em>Azure_Migrate</em>.

## <a name="work-around-vm-folder-restriction"></a>Solución alternativa de restricción de carpeta de VM

En la actualidad, la herramienta Server Assessment no puede detectar VM si se ha concedido acceso de nivel de carpeta de VM de vCenter. Si quiere limitar el ámbito de la detección y la evaluación por carpetas de VM, use esta solución alternativa.

1. Asigne permisos de solo lectura en todas las VM situadas en las carpetas de VM para las que quiere limitar el ámbito de la detección y evaluación.
2. Conceda acceso de solo lectura a todos los objetos primarios que hospedan el host de VM, el clúster, la carpeta de hosts, la carpeta de clústeres, hasta el centro de datos. No es necesario propagar los permisos a todos los objetos secundarios.
3. Para usar las credenciales para la detección, seleccione el centro de datos como **Ámbito del grupo**.


El control de acceso basado en rol configurado garantiza que la cuenta de usuario correspondiente de vCenter solo tenga acceso a las máquinas virtuales específicas del inquilino.


## <a name="next-steps"></a>Pasos siguientes

[Configuración del dispositivo](how-to-set-up-appliance-vmware.md)
