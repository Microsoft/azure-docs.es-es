---
title: Alertas comunes y resolución en Azure AD Domain Services | Microsoft Docs
description: Aprenda a resolver alertas comunes generadas como parte del estado de mantenimiento de Azure Active Directory Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 54319292-6aa0-4a08-846b-e3c53ecca483
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: 29bd7490ea9a03118f134ba11923ede2075e6fd5
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88722592"
---
# <a name="known-issues-common-alerts-and-resolutions-in-azure-active-directory-domain-services"></a>Problemas conocidos: Alertas comunes y resolución en Azure Active Directory Domain Services

Como parte central de la identificación y la autenticación en las aplicaciones, Azure Active Directory Domain Services (Azure AD DS) a veces presenta problemas. En caso de que se produzcan errores, hay algunas alertas habituales y pasos de solución de problemas asociados que le ayudarán a poner todo de nuevo en funcionamiento. En cualquier momento también puede [abrir una solicitud de soporte técnico de Azure][azure-support] para obtener ayuda adicional de solución de problemas.

En este artículo se proporciona información para solución de problemas de alertas comunes en Azure AD DS.

## <a name="aadds100-missing-directory"></a>AADDS100: Falta un directorio

### <a name="alert-message"></a>Mensaje de alerta

*Es posible que se haya eliminado el directorio de Azure AD asociado con su dominio administrado. El dominio administrado ya no está en una configuración admitida. Microsoft no puede supervisar, administrar, revisar ni sincronizar el dominio administrado.*

### <a name="resolution"></a>Resolución

Este error se produce habitualmente cuando una suscripción de Azure se pasa a un nuevo directorio de Azure AD y se elimina el directorio de Azure AD anterior que está asociado con Azure AD DS.

Este error es irrecuperable. Para resolver la alerta, debe [eliminar el dominio administrado existente](delete-aadds.md) y volver a crearlo en el directorio nuevo. Si tiene problemas para eliminar el dominio administrado, [abra una solicitud de soporte técnico de Azure][azure-support] para obtener ayuda adicional para la solución de problemas.

## <a name="aadds101-azure-ad-b2c-is-running-in-this-directory"></a>AADDS101: Azure AD B2C se ejecuta en este directorio

### <a name="alert-message"></a>Mensaje de alerta

*Azure AD Domain Services no se puede habilitar en un directorio de Azure AD B2C.*

### <a name="resolution"></a>Resolución

Azure AD DS se sincroniza automáticamente con un directorio de Azure AD. Si el directorio de Azure AD está configurado para B2C, Azure AD DS no se puede implementar ni sincronizar.

Para usar Azure AD DS, debe volver a crear el dominio administrado en un directorio que no sea de Azure AD B2C mediante los pasos siguientes:

1. [Elimine el dominio administrado](delete-aadds.md) del directorio de Azure AD existente.
1. Cree un nuevo directorio de Azure AD que no sea un directorio de Azure AD B2C.
1. [Cree un dominio administrado de reemplazo](tutorial-create-instance.md).

El estado del dominio administrado se actualiza automáticamente en dos horas y elimina la alerta.

## <a name="aadds103-address-is-in-a-public-ip-range"></a>AADDS103: La dirección está en un intervalo IP público

### <a name="alert-message"></a>Mensaje de alerta

*El intervalo de direcciones IP de la red virtual en la que habilitó Azure AD Domain Services está en un intervalo IP público. Azure AD Domain Services debe estar habilitado en una red virtual con un intervalo de direcciones IP privado. Esta configuración afecta a la capacidad de Microsoft para supervisar, administrar, revisar y sincronizar el dominio administrado.*

### <a name="resolution"></a>Resolución

Antes de empezar, asegúrese de que comprende los [espacios de direcciones IP v4 privados](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces).

Dentro de una red virtual, las máquinas virtuales pueden realizar solicitudes a los recursos de Azure que se encuentran en el mismo intervalo de direcciones IP que los configurados para la subred. Si configura un intervalo de direcciones IP públicas para una subred, es posible que las solicitudes enrutadas dentro de una red virtual no lleguen a los recursos web deseados. Esta configuración puede conducir a errores impredecibles con Azure AD DS.

> [!NOTE]
> Si es propietario del intervalo de direcciones IP que está configurado en la red virtual, puede omitir esta alerta. Sin embargo, Azure AD Domain Services no puede garantizar el cumplimiento del [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/active-directory-ds/v1_0/) con esta configuración, ya que puede dar lugar a errores impredecibles.

Para resolver esta alerta, elimine el dominio administrado existente y vuelva a crearlo en una red virtual con un intervalo de direcciones IP privado. Este proceso es perjudicial porque el dominio administrado no está disponible y se pierden todos los recursos personalizados que haya creado como unidades organizativas o cuentas de servicio.

1. [Elimine el dominio administrado](delete-aadds.md) del directorio.
1. Para actualizar el intervalo de direcciones IP de la red virtual, busque *Red virtual* en Azure Portal y selecciónelo. Seleccione la red virtual para Azure AD DS que tiene un conjunto de intervalos de direcciones IP públicas establecido incorrectamente.
1. En *Configuración*, seleccione **Espacio de direcciones**.
1. Para actualizar el intervalo de direcciones, elija el intervalo existente y edítelo o agregue uno. Asegúrese de que el intervalo de direcciones IP nuevo esté en un intervalo IP privado. Cuando esté preparado, **guarde** los cambios.
1. En el menú de navegación de la izquierda, seleccione **Subredes**.
1. Elija la subred que quiera editar o cree una adicional.
1. Actualice o especifique un intervalo de direcciones IP privadas y, a continuación, **Guarde** los cambios.
1. [Cree un dominio administrado de reemplazo](tutorial-create-instance.md). Asegúrese de elegir una subred de red virtual actualizada con un intervalo de direcciones IP privadas.

El estado del dominio administrado se actualiza automáticamente en dos horas y elimina la alerta.

## <a name="aadds106-your-azure-subscription-is-not-found"></a>AADDS106: Su suscripción de Azure no se encuentra

### <a name="alert-message"></a>Mensaje de alerta

*Se ha eliminado la suscripción de Azure asociada a su dominio administrado.  Azure AD Domain Services requiere una suscripción activa para seguir funcionando correctamente.*

### <a name="resolution"></a>Resolución

Azure AD DS requiere una suscripción activa y no se puede pasar a otra suscripción. Si se elimina la suscripción de Azure a la que estaba asociado el dominio administrado, debe volver a crear una suscripción de Azure y un dominio administrado.

1. [Cree una suscripción a Azure](../cost-management-billing/manage/create-subscription.md).
1. [Elimine el dominio administrado](delete-aadds.md) del directorio de Azure AD existente.
1. [Cree un dominio administrado de reemplazo](tutorial-create-instance.md).

## <a name="aadds107-your-azure-subscription-is-disabled"></a>AADDS107: Su suscripción de Azure está deshabilitada

### <a name="alert-message"></a>Mensaje de alerta

*La suscripción de Azure asociada a su dominio administrado no está activa.  Azure AD Domain Services requiere una suscripción activa para seguir funcionando correctamente.*

### <a name="resolution"></a>Resolución

Azure AD DS requiere una suscripción activa. Si la suscripción de Azure a la que está asociado el dominio administrado no está activa, debe renovarla para reactivar la suscripción.

1. [Renueve su suscripción de Azure](../cost-management-billing/manage/subscription-disabled.md).
2. Una vez que se renueva la suscripción, una notificación de Azure AD DS le permite volver a habilitar el dominio administrado.

Cuando el dominio administrado se habilita de nuevo, el estado del dominio administrado se actualiza automáticamente en dos horas y elimina la alerta.

## <a name="aadds108-subscription-moved-directories"></a>AADDS108: La suscripción ha movido los directorios

### <a name="alert-message"></a>Mensaje de alerta

*La suscripción que utiliza Azure AD Domain Services se ha movido a otro directorio. Azure AD Domain Services necesita tener una suscripción activa en el mismo directorio para que funcione correctamente.*

### <a name="resolution"></a>Resolución

Azure AD DS requiere una suscripción activa y no se puede pasar a otra suscripción. Si se ha movido la suscripción de Azure a la que está asociado el dominio administrado, vuelva a colocarla en el directorio anterior o bien [elimine el dominio administrado](delete-aadds.md) del directorio existente y [cree un dominio administrado de reemplazo en la suscripción elegida](tutorial-create-instance.md).

## <a name="aadds109-resources-for-your-managed-domain-cannot-be-found"></a>AADDS109: No se encuentran recursos del dominio administrado

### <a name="alert-message"></a>Mensaje de alerta

*Se ha eliminado un recurso que se usa con el dominio administrado. Este recurso es necesario para que Azure AD Domain Services funcione correctamente.*

### <a name="resolution"></a>Resolución

Azure AD DS crea recursos adicionales para que funcionen correctamente, como las direcciones IP públicas, las interfaces de red virtual y un equilibrador de carga. Si se elimina alguno de estos recursos designados, el dominio administrado está en un estado incompatible e impide que se administre el dominio. Para más información sobre estos recursos, consulte [Recursos de red usados por Azure AD DS](network-considerations.md#network-resources-used-by-azure-ad-ds).

Esta alerta se genera cuando se elimina uno de estos recursos necesarios. Si el recurso se eliminó hace menos de 4 horas, existe la posibilidad de que la plataforma de Azure pueda volver a crear automáticamente el recurso eliminado. En los pasos siguientes se describe cómo comprobar el estado de mantenimiento y la marca de tiempo para la eliminación de recursos:

1. En Azure Portal, busque y seleccione **Domain Services**. Elija el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. En el panel de navegación izquierdo, seleccione **Estado**.
1. En la página de estado, seleccione la alerta con el identificador *AADDS109*.
1. La alerta tendrá una marca de tiempo del momento en que se encontró por primera vez. Si esa marca de tiempo es de hace menos de 4 horas, es posible que la plataforma Azure pueda volver a crear automáticamente el recurso y resolver la alerta por sí sola.

    Si tiene más, el dominio administrado se encuentra en un estado irrecuperable. [Elimine el dominio administrado](delete-aadds.md) y, a continuación, [cree un dominio administrado de reemplazo](tutorial-create-instance.md).

## <a name="aadds110-the-subnet-associated-with-your-managed-domain-is-full"></a>AADDS110: La subred asociada al dominio administrado está completa

### <a name="alert-message"></a>Mensaje de alerta

*La subred seleccionada para la implementación de Azure AD Domain Services está llena y carece de espacio para el controlador de dominio adicional que hay que crear.*

### <a name="resolution"></a>Resolución

La subred de la red virtual para Azure AD DS necesita direcciones IP suficientes para los recursos creados automáticamente. Este espacio de direcciones IP incluye la necesidad de crear recursos de reemplazo si hay un evento de mantenimiento. Para minimizar el riesgo de quedarse sin direcciones IP disponibles, no implemente recursos adicionales, como sus propias máquinas virtuales, en la misma subred de red virtual que el dominio administrado.

Este error es irrecuperable. Para resolver la alerta, [elimine el dominio administrado existente](delete-aadds.md) y vuelva a crearlo. Si tiene problemas para eliminar el dominio administrado, [abra una solicitud de soporte técnico de Azure][azure-support] para obtener ayuda adicional para la solución de problemas.

## <a name="aadds111-service-principal-unauthorized"></a>AADDS111: Entidad de servicio no autorizada

### <a name="alert-message"></a>Mensaje de alerta

*Una entidad de servicio que Azure AD Domain Services utiliza para atender a su dominio no está autorizado para administrar recursos en la suscripción de Azure. La entidad de servicio debe tener permisos para atender al dominio administrado.*

### <a name="resolution"></a>Resolución

Algunas entidades de servicio generadas automáticamente se utilizan para administrar y crear recursos para un dominio administrado. Si se modifican los permisos de acceso para una de estas entidades de servicio, el dominio no podrá administrar correctamente los recursos. En los pasos siguientes se muestra cómo comprender los permisos de acceso a una entidad de servicio para poder concederlos:

1. Lea sobre el [control de acceso basado en rol y cómo conceder acceso a las aplicaciones en Azure Portal](../role-based-access-control/role-assignments-portal.md).
2. Revise el acceso que la entidad de servicio con identificador *abba844e-bc0e-44b0-947a-dc74e5d09022* tiene y conceda el acceso que se ha denegado en una fecha anterior.

## <a name="aadds112-not-enough-ip-address-in-the-managed-domain"></a>AADDS112: No hay suficientes direcciones IP en el dominio administrado

### <a name="alert-message"></a>Mensaje de alerta

*Hemos identificado que la subred de la red virtual de este dominio no tiene suficientes direcciones IP. Azure AD Domain Services necesita como mínimo dos direcciones IP disponibles en la subred en que se habilita. Se recomienda tener al menos de tres a cinco direcciones IP de reserva en la subred. Esto puede deberse a que se hayan implementado otras máquinas virtuales dentro de la subred, agotando así el número de direcciones IP disponibles, o a que haya una restricción en el número de direcciones IP disponibles en la subred.*

### <a name="resolution"></a>Resolución

La subred de la red virtual para Azure AD DS necesita direcciones IP suficientes para los recursos creados automáticamente. Este espacio de direcciones IP incluye la necesidad de crear recursos de reemplazo si hay un evento de mantenimiento. Para minimizar el riesgo de quedarse sin direcciones IP disponibles, no implemente recursos adicionales, como sus propias máquinas virtuales, en la misma subred de red virtual que el dominio administrado.

Para resolver esta alerta, elimine el dominio administrado existente y vuelva a crearlo en una red virtual con un intervalo de direcciones IP suficientemente grande. Este proceso es perjudicial porque el dominio administrado no está disponible y se pierden todos los recursos personalizados que haya creado como unidades organizativas o cuentas de servicio.

1. [Elimine el dominio administrado](delete-aadds.md) del directorio.
1. Para actualizar el intervalo de direcciones IP de la red virtual, busque *Red virtual* en Azure Portal y selecciónelo. Seleccione la red virtual para el dominio administrado que tenga el intervalo de direcciones IP pequeño.
1. En *Configuración*, seleccione **Espacio de direcciones**.
1. Para actualizar el intervalo de direcciones, elija el intervalo existente y edítelo o agregue uno. Asegúrese de que el nuevo intervalo de direcciones IP sea lo suficientemente grande para el intervalo de subred del dominio administrado. Cuando esté preparado, **guarde** los cambios.
1. En el menú de navegación de la izquierda, seleccione **Subredes**.
1. Elija la subred que quiera editar o cree una adicional.
1. Actualice o especifique un intervalo de direcciones IP suficientemente grande y, a continuación, **guarde** los cambios.
1. [Cree un dominio administrado de reemplazo](tutorial-create-instance.md). Asegúrese de elegir una subred de red virtual actualizada con un intervalo de direcciones IP suficientemente grande.

El estado del dominio administrado se actualiza automáticamente en dos horas y elimina la alerta.

## <a name="aadds113-resources-are-unrecoverable"></a>AADDS113: Los recursos son irrecuperables

### <a name="alert-message"></a>Mensaje de alerta

*Se ha detectado que los recursos que utiliza Azure AD Domain Services tienen un estado inesperado y no se pueden recuperar.*

### <a name="resolution"></a>Resolución

Este error es irrecuperable. Para resolver la alerta, [elimine el dominio administrado existente](delete-aadds.md) y vuelva a crearlo. Si tiene problemas para eliminar el dominio administrado, [abra una solicitud de soporte técnico de Azure][azure-support] para obtener ayuda adicional para la solución de problemas.

## <a name="aadds114-subnet-invalid"></a>AADDS114: Subred no válida

### <a name="alert-message"></a>Mensaje de alerta

*La subred seleccionada para la implementación de Azure AD Domain Services no es válida y no se puede utilizar.*

### <a name="resolution"></a>Resolución

Este error es irrecuperable. Para resolver la alerta, [elimine el dominio administrado existente](delete-aadds.md) y vuelva a crearlo. Si tiene problemas para eliminar el dominio administrado, [abra una solicitud de soporte técnico de Azure][azure-support] para obtener ayuda adicional para la solución de problemas.

## <a name="aadds115-resources-are-locked"></a>AADDS115: Los recursos están bloqueados

### <a name="alert-message"></a>Mensaje de alerta

*Uno o varios de los recursos de red que se utilizan en el dominio administrado no pueden ejecutarse porque el ámbito de destino se ha bloqueado.*

### <a name="resolution"></a>Resolución

Los bloqueos de recursos se pueden aplicar a recursos de Azure para impedir el cambio o la eliminación. Dado que Azure AD DS es un servicio administrado, la plataforma Azure necesita la capacidad de realizar cambios en la configuración. Si se aplica un bloqueo de recursos en algunos de los componentes de Azure AD DS, la plataforma Azure no puede realizar sus tareas de administración.

Para comprobar los bloqueos de recursos en los componentes de Azure AD DS y quitarlos, siga estos pasos:

1. Para cada uno de los componentes de red del dominio administrado en el grupo de recursos, como la red virtual, la interfaz de red o la dirección IP pública, compruebe los registros de operaciones en Azure Portal. Estos registros de operaciones deben indicar por qué se produce un error en una operación y dónde se aplica un bloqueo de recursos.
1. Seleccione el recurso en el que se aplica un bloqueo y, en **Bloqueos**, seleccione los bloqueos y quítelos.

## <a name="aadds116-resources-are-unusable"></a>AADDS116: Los recursos no son utilizables

### <a name="alert-message"></a>Mensaje de alerta

*Uno o varios de los recursos de red que se utilizan en el dominio administrado no pueden ejecutarse debido a restricciones de directiva.*

### <a name="resolution"></a>Resolución

Las directivas se aplican a los recursos y los grupos de recursos de Azure que controlan las acciones de configuración que se permiten. Dado que Azure AD DS es un servicio administrado, la plataforma Azure necesita la capacidad de realizar cambios en la configuración. Si se aplica una directiva en algunos de los componentes de Azure AD DS, la plataforma Azure no puede realizar sus tareas de administración.

Para comprobar las directivas aplicadas en los componentes de Azure AD DS y quitarlas, siga estos pasos:

1. Para cada uno de los componentes de red del dominio administrado en el grupo de recursos, como la red virtual, el adaptador de red o la dirección IP pública, compruebe los registros de operaciones en Azure Portal. Estos registros de operaciones deben indicar por qué se produce un error en una operación y dónde se aplica una directiva restrictiva.
1. Seleccione el recurso al que se aplica una directiva y, en **Directivas**, seleccione y edite la directiva para que sea menos restrictiva.

## <a name="aadds500-synchronization-has-not-completed-in-a-while"></a>AADDS500: Hace un tiempo que no se realiza una sincronización

### <a name="alert-message"></a>Mensaje de alerta

*El dominio administrado se sincronizó por última vez con Azure AD el [fecha]. Los usuarios no pueden iniciar sesión en el dominio o puede que las pertenencias a grupos no estén sincronizadas con Azure AD.*

### <a name="resolution"></a>Resolución

[Compruebe el estado de Azure AD DS](check-health.md) para ver las alertas que podrían indicar problemas en la configuración del dominio administrado. Los problemas con la configuración de red pueden bloquear la sincronización desde Azure AD. Si puede resolver las alertas que indican un problema de configuración, espere dos horas y compruebe de nuevo para ver si se ha completado la sincronización correctamente.

Los siguientes motivos comunes provocan que la sincronización se detenga en un dominio administrado:

* La conectividad de red necesaria se bloquea. Para obtener más información sobre cómo comprobar la red virtual de Azure para detectar si hay problemas y qué se requiere, consulte [cómo solucionar problemas de grupos de seguridad de red](alert-nsg.md) y los [requisitos de red para Azure AD DS](network-considerations.md).
*  La sincronización de contraseñas no se configuró o se completó incorrectamente cuando se implementó el dominio administrado. Puede configurar la sincronización de contraseñas para los [usuarios solo de nube](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) o para los [usuarios de entornos híbridos desde el entorno local](tutorial-configure-password-hash-sync.md).

## <a name="aadds501-a-backup-has-not-been-taken-in-a-while"></a>AADDS501: Hace un tiempo que no se realiza una copia de seguridad

### <a name="alert-message"></a>Mensaje de alerta

*Se hizo copia de seguridad del dominio administrado por última vez el [fecha].*

### <a name="resolution"></a>Resolución

[Compruebe el mantenimiento de Azure AD DS](check-health.md) para ver las alertas que podrían indicar problemas en la configuración del dominio administrado. Los problemas con la configuración de red pueden impedir que la plataforma Azure realice las copias de seguridad correctamente. Si puede resolver las alertas que indican un problema de configuración, espere dos horas y compruebe de nuevo para ver si se ha completado la sincronización correctamente.

## <a name="aadds503-suspension-due-to-disabled-subscription"></a>AADDS503: Suspensión debida a una suscripción deshabilitada

### <a name="alert-message"></a>Mensaje de alerta

*El dominio administrado se suspende debido a que la suscripción de Azure asociada al dominio no está activa.*

### <a name="resolution"></a>Resolución

> [!WARNING]
> Si un dominio administrado se suspende durante un largo período, se expone a ser eliminado. Resuelva el motivo de la suspensión lo más rápido posible. Para más información, consulte [Descripción de los estados suspendidos para Azure AD DS](suspension.md).

Azure AD DS requiere una suscripción activa. Si la suscripción de Azure a la que está asociado el dominio administrado no está activa, debe renovarla para reactivar la suscripción.

1. [Renueve su suscripción de Azure](../cost-management-billing/manage/subscription-disabled.md).
2. Una vez que se renueva la suscripción, una notificación de Azure AD DS le permite volver a habilitar el dominio administrado.

Cuando el dominio administrado se habilita de nuevo, el estado del dominio administrado se actualiza automáticamente en dos horas y elimina la alerta.

## <a name="aadds504-suspension-due-to-an-invalid-configuration"></a>AADDS504: Suspensión debida a una configuración no válida

### <a name="alert-message"></a>Mensaje de alerta

*El dominio administrado se suspende debido a una configuración no válida. El servicio lleva mucho tiempo sin poder administrar, actualizar o aplicar revisiones a los controladores de dominio en el dominio administrado.*

### <a name="resolution"></a>Resolución

> [!WARNING]
> Si un dominio administrado se suspende durante un largo período, se expone a ser eliminado. Resuelva el motivo de la suspensión lo más rápido posible. Para más información, consulte [Descripción de los estados suspendidos para Azure AD DS](suspension.md).

[Compruebe el mantenimiento de Azure AD DS](check-health.md) para ver las alertas que podrían indicar problemas en la configuración del dominio administrado. Si puede resolver las alertas que indican un problema de configuración, espere dos horas y compruebe de nuevo para ver si se ha completado la sincronización. Cuando esté listo, [abra una solicitud de soporte técnico de Azure][azure-support] para volver a habilitar el dominio administrado.

## <a name="next-steps"></a>Pasos siguientes

Si los problemas persisten, [abra una solicitud de soporte técnico de Azure][azure-support] para obtener ayuda adicional de solución de problemas.

<!-- INTERNAL LINKS -->
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md