---
title: 'Preguntas frecuentes y problemas conocidos con identidades administradas: Azure AD'
description: Problemas conocidos con identidades administradas para recursos de Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.assetid: 2097381a-a7ec-4e3b-b4ff-5d2fb17403b6
ms.service: active-directory
ms.subservice: msi
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 08/06/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 4bcd36a1ce38d4d9eb6a0faec470f7427852894b
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89260227"
---
# <a name="faqs-and-known-issues-with-managed-identities-for-azure-resources"></a>Preguntas frecuentes y problemas conocidos con identidades administradas para recursos de Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

## <a name="frequently-asked-questions-faqs"></a>Preguntas más frecuentes (P+F)

> [!NOTE]
> Identidades administradas para recursos de Azure es el nombre con el que ahora se conoce al servicio Managed Service Identity (MSI).


### <a name="how-can-you-find-resources-that-have-a-managed-identity"></a>¿Cómo se pueden encontrar los recursos que tienen una identidad administrada?

Puede encontrar la lista de recursos con una identidad administrada asignada por el sistema mediante el siguiente comando de la CLI de Azure: 

`az resource list --query "[?identity.type=='SystemAssigned'].{Name:name,  principalId:identity.principalId}" --output table`




### <a name="do-managed-identities-have-a-backing-app-object"></a>¿Las identidades administradas tienen un objeto de aplicación de respaldo?

No. Las identidades administradas y los registros de aplicaciones de Azure AD no son lo mismo en el directorio. 

Los registros de aplicaciones tienen dos componentes: Un objeto de aplicación y un objeto de entidad de servicio. Las identidades administradas para los recursos de Azure solo tienen uno de estos componentes: un objeto de entidad de servicio. 

Las identidades administradas no tienen un objeto de aplicación en el directorio, que es lo que se suele usar para conceder permisos de aplicación para MS Graph. En su lugar, los permisos de MS Graph para identidades administradas deben concederse directamente a la entidad de servicio.  

### <a name="does-managed-identities-for-azure-resources-work-with-azure-cloud-services"></a>¿Funcionan las identidades administradas para recursos de Azure con Azure Cloud Services?

No, no hay planes que admitan las identidades administradas para recursos de Azure en Azure Cloud Services.

### <a name="what-is-the-credential-associated-with-a-managed-identity-how-long-is-it-valid-and-how-often-is-it-rotated"></a>¿Cuál es la credencial asociada a una identidad administrada? ¿Qué validez tiene y con qué frecuencia se rota?

> [!NOTE]
> La forma de autenticarse las identidades administradas es un dato de implementación interno que está sujeto a cambios sin previo aviso.

Las identidades administradas usan la autenticación basada en certificados. Cada credencial de identidad administrada expira al cabo de 90 días y se revierte después de 45 días.

### <a name="what-is-the-security-boundary-of-managed-identities-for-azure-resources"></a>¿Cuál es el límite de seguridad de las identidades administradas para recursos de Azure?

El límite de seguridad de la identidad es el recurso al que está asociada. Por ejemplo, el límite de seguridad para una máquina virtual que tenga las identidades administradas para recursos de Azure habilitadas, es la Máquina virtual. Cualquier código que se ejecute en esa máquina virtual puede llamar a las identidades administradas para los tokens de punto de conexión y solicitud de recursos de Azure. Esta experiencia es similar con otros recursos que admiten las identidades administradas para recursos de Azure.

### <a name="what-identity-will-imds-default-to-if-dont-specify-the-identity-in-the-request"></a>¿A qué identidad se asignará el IMDS de manera predeterminada si no se especifica la identidad en la solicitud?

- Si se habilita la identidad administrada asignada por el sistema y no se especifica ninguna identidad en la solicitud, el IMDS utilizará de manera predeterminada la esta identidad administrada.
- Si la identidad administrada asignada por el sistema no está habilitada, y solo existe una identidad administrada asignada por el usuario, IMDS será la identidad administrada asignada de manera predeterminada a ese único usuario. 
- Si la identidad administrada asignada por el sistema no está habilitada y existen varias identidades administradas asignadas por el usuario, es necesario especificar una identidad administrada en la solicitud.



### <a name="will-managed-identities-be-recreated-automatically-if-i-move-a-subscription-to-another-directory"></a>¿Se van a volver a crear automáticamente las identidades administradas si muevo una suscripción a otro directorio?

No. Si mueve una suscripción a otro directorio, tendrá que volver a crearlas manualmente y volver a asignar los roles de Azure.
- Para las identidades administradas asignadas por el sistema, tiene que desactivarlas y volver a activarlas. 
- Para las identidades administradas asignadas por el usuario, tiene que eliminarlas, volver a crearlas y adjuntarlas de nuevo a los recursos necesarios (por ejemplo, máquinas virtuales).

### <a name="can-i-use-a-managed-identity-to-access-a-resource-in-a-different-directorytenant"></a>¿Puedo usar una identidad administrada para acceder a un recurso en un directorio o inquilino diferente?

No. Las identidades administradas no admiten actualmente escenarios entre directorios. 

### <a name="what-azure-rbac-permissions-are-required-to-managed-identity-on-a-resource"></a>¿Qué permisos RBAC de Azure son necesarios para una identidad administrada en un recurso? 

- Identidad administrada asignada por el sistema: Se necesitan permisos de escritura sobre el recurso. Por ejemplo, para las máquinas virtuales es necesario Microsoft.Compute/virtualMachines/write. Esta acción se incluye en los roles integrados específicos del recurso como, por ejemplo, [Colaborador de máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).
- Identidad administrada asignada por el usuario: Se necesitan permisos de escritura sobre el recurso. Por ejemplo, para las máquinas virtuales es necesario Microsoft.Compute/virtualMachines/write. Además de la asignación de roles [Operador de identidad administrada](../../role-based-access-control/built-in-roles.md#managed-identity-operator) sobre la identidad administrada.



## <a name="known-issues"></a>Problemas conocidos

### <a name="automation-script-fails-when-attempting-schema-export-for-managed-identities-for-azure-resources-extension"></a>Se produce un error en el "script de automatización" al intentar exportar los esquemas de la extensión de identidades administradas para recursos de Azure.

Cuando se habilitan las identidades administradas para recursos de Azure en una máquina virtual, se muestra el siguiente error al intentar usar la característica de "Script de automatización" para la máquina virtual o su grupo de recursos:

![Error de exportación del script de automatización de identidades administradas para recursos de Azure](./media/msi-known-issues/automation-script-export-error.png)

La extensión de máquina virtual de identidades administradas para recursos de Azure (que dejará de utilizarse en enero de 2019) no admite actualmente la posibilidad de exportar su esquema a una plantilla de grupo de recursos. Como resultado, en la plantilla generada no se muestran los parámetros de configuración para habilitar las identidades administradas para recursos de Azure en el recurso. Estas secciones se pueden agregar de forma manual siguiendo los ejemplos de [Configuración de identidades administradas para recursos de Azure en una máquina virtual de Azure mediante una plantilla](qs-configure-template-windows-vm.md).

Cuando la funcionalidad de exportación de esquema está disponible para la extensión de máquina virtual de identidades administradas para recursos de Azure (que dejará de utilizarse en enero de 2019), se enumerará en [Exportación de grupos de recursos que contienen extensiones de máquina virtual](../../virtual-machines/extensions/export-templates.md#supported-virtual-machine-extensions).

### <a name="vm-fails-to-start-after-being-moved-from-resource-group-or-subscription"></a>La máquina virtual no puede iniciarse después de moverse del grupo de recursos o de la suscripción

Si mueve una máquina virtual en estado de ejecución, continúa ejecutándose durante el desplazamiento. Sin embargo, después, si la máquina virtual se detiene y se reinicia, no se podrá iniciar. Este problema se produce porque la máquina virtual no está actualizando la referencia a la identidad referente a las identidades administradas para recursos de Azure, y continúa apuntando a ella en el grupo de recursos anterior.

**Solución alternativa** 
 
Desencadene una actualización en la máquina virtual para que pueda obtener los valores correctos para las identidades administradas para recursos de Azure. Puede hacer un cambio de propiedad de máquina virtual para actualizar la referencia a la identidad referente a las identidades administradas para recursos de Azure. Por ejemplo, puede establecer un nuevo valor de etiqueta en la máquina virtual con el siguiente comando:

```azurecli-interactive
 az  vm update -n <VM Name> -g <Resource Group> --set tags.fixVM=1
```
 
Este comando establece una nueva etiqueta "fixVM" con un valor de 1 en la máquina virtual. 
 
Al establecer esta propiedad, la máquina virtual se actualiza con el URI del recurso correcto de las identidades administradas para recursos de Azure y, después, debería poder iniciar la máquina virtual. 
 
Una vez que se inicia la máquina virtual, la etiqueta puede quitarse con el comando siguiente:

```azurecli-interactive
az vm update -n <VM Name> -g <Resource Group> --remove tags.fixVM
```



### <a name="transferring-a-subscription-between-azure-ad-directories"></a>Transferencia de una suscripción entre directorios de Azure AD

Las identidades administradas no se actualizan cuando una suscripción se mueve o transfiere a otro directorio. Como resultado, se interrumpe cualquier identidad administrada asignada por el sistema o por el usuario. 

Solución alternativa para identidades administradas en una suscripción que se ha movido a otro directorio:

 - Para las identidades administradas asignadas por el sistema, tiene que desactivarlas y volver a activarlas. 
 - Para las identidades administradas asignadas por el usuario, tiene que eliminarlas, volver a crearlas y adjuntarlas de nuevo a los recursos necesarios (por ejemplo, máquinas virtuales).

Para más información, consulte [Transferencia de una suscripción de Azure a otro directorio de Azure AD (versión preliminar)](../../role-based-access-control/transfer-subscription.md).

### <a name="moving-a-user-assigned-managed-identity-to-a-different-resource-groupsubscription"></a>Movimiento de una identidad administrada asignada por el usuario a otro grupo de recursos o suscripción

No se admite el movimiento de una identidad administrada asignada por el usuario a otro grupo de recursos.