---
title: Configuración de una identidad de laboratorio en Azure DevTest Labs
description: Aprenda a configurar una identidad de laboratorio en Azure DevTest Labs.
ms.topic: article
ms.date: 08/20/2020
ms.openlocfilehash: a652eb5751f9b723911a1c1baaaaf9860febc5b6
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88719589"
---
# <a name="configure-a-lab-identity"></a>Configuración de la identidad del laboratorio

Un desafío común al compilar aplicaciones en la nube consiste en el modo de administrar las credenciales del código para autenticar los servicios en la nube. Proteger las credenciales es una tarea esencial. Lo ideal sería que nunca aparecieran en las estaciones de trabajo de los desarrolladores y que no se controlaran en el código fuente. Azure Key Vault proporciona una manera segura de almacenar credenciales, secretos y otras claves, pero el código debe autenticarse en Key Vault para recuperarlos. 

Las identidades administradas para recursos de Azure con las que cuenta Azure Active Directory (Azure AD) solucionan este problema. Esta característica proporciona a los servicios de Azure una identidad de sistema administrada automáticamente en Azure AD. Puede usar esta identidad para autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluido Key Vault, sin necesidad de credenciales en el código. Obtenga más información sobre [identidades administradas en Azure](../active-directory/managed-identities-azure-resources/overview.md). 

Hay dos tipos de identidades administradas: 

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema  

Una  **identidad administrada asignada por el sistema** se habilita directamente en una instancia de servicio de Azure. Cuando se habilita la identidad, Azure crea una identidad para la instancia del servicio en el inquilino de Azure AD de confianza de la suscripción de la instancia. Una vez creada la identidad, las credenciales se aprovisionan en la instancia. El ciclo de vida de una identidad asignada por el sistema está vinculado directamente a la instancia de servicio de Azure en que está habilitada. Si se elimina la instancia, Azure limpia automáticamente las credenciales y la identidad en Azure AD. 

### <a name="scenarios-for-using-labs-system-assigned-identity"></a>Escenarios para usar la identidad asignada por el sistema del laboratorio  

Cada instancia de DevTest Labs se crea con una identidad asignada por el sistema que sigue siendo válida durante la vigencia del laboratorio. La identidad asignada por el sistema se usa para los siguientes fines:  

- Todas las implementaciones basadas en [Azure Resource Manager](devtest-lab-create-environment-from-arm.md) que se usan para poner en marcha varias máquinas virtuales y/o plataformas como un entorno de servicio se ejecutarán con la identidad asignada por el sistema del laboratorio.  
- El cifrado de discos para los discos del laboratorio con una clave administrada por el cliente de admite a través de la identidad asignada por el sistema del laboratorio. Al proporcionar acceso explícito a la identidad del laboratorio para tener acceso al conjunto de cifrado de discos, el laboratorio puede cifrar todos los discos de las máquinas virtuales en su nombre. Obtenga más información sobre [cómo habilitar el cifrado de discos](encrypt-disks-customer-managed-keys.md) para los discos del laboratorio con una clave administrada por el cliente.  

### <a name="configure-identity"></a>Configurar la identidad

En esta sección se muestra cómo configurar la directiva de identidad del laboratorio.

> [!NOTE]
> En el caso de los laboratorios creados antes del 10 de agosto de 2020, la identidad asignada por el sistema se establecerá en Desactivada. Como propietario de un laboratorio, puede activarla en caso de que pretenda usar los laboratorios para los fines indicados en la sección anterior.  
>
> En el caso de los nuevos laboratorios creados después del 10 de agosto de 2020, la identidad asignada por el sistema del laboratorio está activada de forma predeterminada y el propietario del laboratorio no podrá desactivarla durante el ciclo de vida del laboratorio.  

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **DevTest Labs**.
1. En la lista de laboratorios, seleccione el laboratorio que quiera.
1. Seleccione **Configuración y directivas** -> **Identidad (versión preliminar)** . 

> [!div class="mx-imgBorder"]
> ![Configuración de la identidad](./media/configure-lab-identity/configure-identity.png)

## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario  

Se crea una identidad administrada asignada por el usuario como recurso de Azure independiente. Mediante un proceso de creación, Azure crea una identidad en el inquilino de Azure AD de confianza para la suscripción que se utiliza. Una vez creada la identidad, esta puede asignarse a una o varias instancias de servicio de Azure. El ciclo de vida de una identidad asignada por el usuario no se administra junto con el ciclo de vida de las instancias de servicio de Azure a las que se asigna. 

DevTest Labs admite identidades asignadas por el usuario tanto para máquinas virtuales como para entornos basados en Azure Resource Manager.  Para obtener más información, vea los temas siguientes:

- [Adición de una identidad asignada por el usuario para implementar entornos de Azure Resource Manager en un laboratorio](use-managed-identities-environments.md)
- [Adición de identidades asignadas por el usuario para implementar máquinas virtuales de laboratorio](enable-managed-identities-lab-vms.md)

## <a name="next-steps"></a>Pasos siguientes

Consultar la [Configuración de la administración de costos](devtest-lab-configure-cost-management.md)