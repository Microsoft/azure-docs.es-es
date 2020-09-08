---
title: Problemas conocidos del aprovisionamiento de aplicaciones en Azure AD
description: Obtenga información sobre los problemas conocidos al trabajar con el aprovisionamiento automático de aplicaciones en Azure AD.
author: kenwith
ms.author: kenwith
manager: celestedg
services: active-directory
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 08/12/2020
ms.reviewer: arvinh
ms.openlocfilehash: 23c3dfc6670c96f44a10b2ad5d5bfeb3ff96382c
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271012"
---
# <a name="known-issues-application-provisioning"></a>Problemas conocidos: Aprovisionamiento de aplicaciones
Problemas conocidos que se deben tener en cuenta al trabajar con el aprovisionamiento de aplicaciones. Puede proporcionar comentarios sobre el servicio de aprovisionamiento de aplicaciones en UserVoice; vea [Aprovisionamiento de aplicaciones de Azure AD en UserVoice](https://aka.ms/appprovisioningfeaturerequest). Supervisamos UserVoice muy de cerca para poder mejorar el servicio. 

> [!NOTE]
> Esta no es una lista completa de problemas conocidos. Si sabe que hay un problema que no aparece en la lista, proporcione comentarios en la parte inferior de la página.

## <a name="authorization"></a>Authorization 

**No se puede guardar después de una prueba de conexión correcta**

Si puede probar correctamente una conexión, pero no puede guardar, ha superado el límite de almacenamiento permitido para las credenciales. Para más información, vea [Problema al guardar las credenciales del administrador](application-provisioning-config-problem-storage-limit.md).

**No se puede guardar**

La dirección URL del inquilino, el token secreto y el correo electrónico de notificación se deben rellenar para guardar. No se puede proporcionar solo uno de ellos. 

**No se puede volver a cambiar el modo de aprovisionamiento a manual**

Tras configurar el aprovisionamiento por primera vez, observará que el modo de aprovisionamiento cambia de manual a automático. No se puede volver a cambiar a manual. Sin embargo, puede desactivar el aprovisionamiento en la interfaz de usuario. Desactivar el aprovisionamiento en la interfaz de usuario tiene el mismo efecto que establecer la configuración en manual en el menú desplegable.  


## <a name="attribute-mappings"></a>Asignaciones de atributos 

**Atributo SamAccountName o userType no disponible como un atributo de origen**

Los atributos SamAccountName y userType no están disponibles como un atributo de origen de forma predeterminada. Extienda el esquema para agregar el atributo. Puede agregar los atributos a la lista de atributos de origen disponibles mediante la extensión del esquema. Para más información, vea [Falta el atributo de origen](user-provisioning-sync-attributes-for-mapping.md). 

**Faltan atributos de origen en la lista desplegable para la extensión del esquema**

A veces, pueden faltar extensiones en la lista desplegable de atributos de origen de la interfaz de usuario. Vaya a la configuración avanzada de las asignaciones de atributos y agregue manualmente los atributos. Para más información, vea [Personalización de asignaciones de atributos](customize-application-attributes.md).

**No se puede aprovisionar el atributo NULL**

Azure AD actualmente no puede aprovisionar atributos NULL. Si un atributo es NULL en el objeto de usuario, se omitirá. 

**Caracteres máximos para las expresiones de asignación de atributos**

Las expresiones de asignación de atributos pueden tener un máximo de 10 000 caracteres. 


## <a name="service-issues"></a>Problemas de servicio 

**Escenarios no admitidos**

- No se admite el aprovisionamiento de contraseñas. 
- No se admite el aprovisionamiento de grupos anidados. 
- No se admite el aprovisionamiento de inquilinos B2C debido al tamaño de los inquilinos. 

**El intervalo de aprovisionamiento es fijo**. El [tiempo](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user#how-long-will-it-take-to-provision-users) entre los ciclos de aprovisionamiento no se puede configurar actualmente. 

**Cambios que no se mueven de la aplicación de destino a Azure AD**

El servicio de aprovisionamiento de aplicaciones no reconoce los cambios realizados en las aplicaciones externas. Por tanto, no se realiza ninguna acción para la reversión. El servicio de aprovisionamiento de aplicaciones se basa en los cambios realizados en Azure AD. 

**El ciclo de aprovisionamiento continúa hasta la finalización**

Al establecer el aprovisionamiento `enabled = off`, o al seleccionar la opción de detención, el ciclo de aprovisionamiento actual continuará ejecutándose hasta que se complete. El servicio dejará de ejecutar todos los ciclos futuros hasta que vuelva a activar el aprovisionamiento.

**Miembro del grupo no aprovisionado**

Cuando un grupo está dentro del ámbito y un miembro está fuera, se aprovisionará el grupo. El usuario que está fuera del ámbito no se aprovisionará. Si el miembro vuelve al ámbito, el servicio no detectará el cambio inmediatamente. Al reiniciar el aprovisionamiento, se solucionará el problema. Se recomienda reiniciar periódicamente el servicio para asegurarse de que todos los usuarios se hayan aprovisionado correctamente.  


## <a name="next-steps"></a>Pasos siguientes
- [Funcionamiento del aprovisionamiento](how-provisioning-works.md)
