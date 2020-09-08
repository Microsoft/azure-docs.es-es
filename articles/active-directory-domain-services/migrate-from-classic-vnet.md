---
title: Migración de Azure AD Domain Services desde una red virtual clásica | Microsoft Docs
description: Aprenda a migrar un dominio administrado de Azure AD Domain Services existente desde el modelo de red virtual clásica a una red virtual basada en Resource Manager.
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 08/10/2020
ms.author: iainfou
ms.openlocfilehash: de27ee713caae0310f185cd717d5db2095feff32
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88054296"
---
# <a name="migrate-azure-active-directory-domain-services-from-the-classic-virtual-network-model-to-resource-manager"></a>Migración de Azure Active Directory Domain Services desde el modelo de red virtual clásica a Resource Manager

Azure Active Directory Domain Services (Azure AD DS) admite un único traslado para los clientes que actualmente usan el modelo de red virtual clásica al modelo de red virtual de Resource Manager. Los dominios administrados de Azure AD DS que usan el modelo de implementación de Resource Manager proporcionan características adicionales, como la directiva de contraseñas específica, los registros de auditoría y la protección de bloqueo de cuentas.

En este artículo se describen las consideraciones que hay que tener en cuenta para la migración, así como los pasos necesarios para migrar correctamente un dominio administrado existente. Para conocer algunas de las ventajas, consulte [Ventajas de la migración del modelo de implementación clásico a Resource Manager en Azure AD DS][migration-benefits].

> [!NOTE]
> En 2017, Azure AD Domain Services se comenzó a poder hospedar en una red de Azure Resource Manager. Desde entonces, hemos podido crear un servicio más seguro gracias a las modernas funcionalidades de Azure Resource Manager. Como las implementaciones de Azure Resource Manager están reemplazando totalmente a las implementaciones clásicas, las implementaciones de las redes virtuales clásicas de Azure AD DS se retirarán el 1 de marzo de 2023.
>
> Para obtener más información, vea el [aviso de desuso oficial](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/).

## <a name="overview-of-the-migration-process"></a>Introducción al proceso de migración

El proceso de migración toma un dominio administrado existente que se ejecuta en una red virtual clásica y lo mueve a una red virtual de Resource Manager existente. La migración se realiza mediante PowerShell y tiene dos fases principales de ejecución: *preparación* y *migración*.

![Introducción al proceso de migración de Azure AD DS](media/migrate-from-classic-vnet/migration-overview.png)

En la fase de *preparación*, Azure AD DS realiza una copia de seguridad del dominio para obtener la última instantánea de los usuarios, los grupos y las contraseñas sincronizados con el dominio administrado. A continuación, se deshabilita la sincronización y se elimina el servicio en la nube que hospeda el dominio administrado. Durante la fase de preparación, el dominio administrado no puede autenticar a los usuarios.

![Fase de preparación para migrar Azure AD DS](media/migrate-from-classic-vnet/migration-preparation.png)

En la fase de *migración* se copian los discos virtuales subyacentes de los controladores de dominio del dominio administrado clásico, para crear las máquinas virtuales con el modelo de implementación de Resource Manager. Después se vuelve a crear el dominio administrado, que incluye la configuración de LDAPS y DNS. Se reinicia la sincronización con Azure AD y se restauran los certificados LDAP. No es necesario volver a unir ninguna máquina a un dominio administrado: continúan unidas al dominio administrado y se ejecutan sin cambios.

![Migración de Azure AD DS](media/migrate-from-classic-vnet/migration-process.png)

## <a name="example-scenarios-for-migration"></a>Escenarios de ejemplo para la migración

En los siguientes ejemplos se incluyen algunos escenarios comunes para migrar un dominio administrado.

> [!NOTE]
> No convierta la red virtual clásica hasta que haya confirmado que la migración se ha realizado correctamente. Al convertir la red virtual, se pierde la opción de revertir o restaurar el dominio administrado si hay algún problema durante las fases de migración y comprobación.

### <a name="migrate-azure-ad-ds-to-an-existing-resource-manager-virtual-network-recommended"></a>Migración de Azure AD DS a una red virtual de Resource Manager existente (recomendado)

Un escenario común es aquel en el que ya ha transferido otros recursos clásicos existentes a un modelo de implementación y una red virtual de Resource Manager. Después se usa el emparejamiento de la red virtual de Resource Manager a la red virtual clásica que continúa ejecutando Azure AD DS. Este enfoque permite que las aplicaciones y servicios de Resource Manager usen la funcionalidad de autenticación y administración del dominio administrado en la red virtual clásica. Una vez efectuada la migración, todos los recursos se ejecutan con el modelo de implementación y la red virtual de Resource Manager.

![Migración de Azure AD DS a una red virtual de Resource Manager existente](media/migrate-from-classic-vnet/migrate-to-existing-vnet.png)

Los pasos generales implicados en este escenario de migración de ejemplo incluyen las siguientes partes:

1. Eliminación de las puertas de enlace de VPN existentes o del emparejamiento de redes virtuales configurado en la red virtual clásica
1. Migre el dominio administrado siguiendo los pasos descritos en este artículo.
1. Prueba y confirmación de que la migración se realizó correctamente y, a continuación, eliminación de la red virtual clásica

### <a name="migrate-multiple-resources-including-azure-ad-ds"></a>Migración de varios recursos, incluido Azure AD DS

En este escenario de ejemplo se migran Azure AD DS y otros recursos asociados desde el modelo de implementación clásica al modelo de implementación de Resource Manager. Si algunos recursos seguían ejecutándose en la red virtual clásica junto con el dominio administrado, pueden beneficiarse de la migración al modelo de implementación de Resource Manager.

![Migración de varios recursos al modelo de implementación de Resource Manager](media/migrate-from-classic-vnet/migrate-multiple-resources.png)

Los pasos generales implicados en este escenario de migración de ejemplo incluyen las siguientes partes:

1. Eliminación de las puertas de enlace de VPN existentes o del emparejamiento de redes virtuales configurado en la red virtual clásica
1. Migre el dominio administrado siguiendo los pasos descritos en este artículo.
1. Configuración del emparejamiento de redes virtuales entre la red virtual clásica y la red de Resource Manager
1. Prueba y confirmación de que la migración se ha realizado correctamente
1. [Movimiento de recursos clásicos adicionales, como máquinas virtuales][migrate-iaas]

### <a name="migrate-azure-ad-ds-but-keep-other-resources-on-the-classic-virtual-network"></a>Migración de Azure AD DS con mantenimiento de otros recursos en la red virtual clásica

Con este escenario de ejemplo se consigue el menor tiempo de inactividad en una sesión. Solo se migra Azure AD DS a una red virtual de Resource Manager y se mantienen los recursos existentes en el modelo de implementación y en la red virtual clásicas. En un período de mantenimiento siguiente, se pueden migrar los recursos adicionales del modelo de implementación y la red virtual clásicas según se desee.

![Migración solo de Azure AD DS al modelo de implementación de Resource Manager](media/migrate-from-classic-vnet/migrate-only-azure-ad-ds.png)

Los pasos generales implicados en este escenario de migración de ejemplo incluyen las siguientes partes:

1. Eliminación de las puertas de enlace de VPN existentes o del emparejamiento de redes virtuales configurado en la red virtual clásica
1. Migre el dominio administrado siguiendo los pasos descritos en este artículo.
1. Configuración del emparejamiento de redes virtuales entre la red virtual clásica y la nueva red virtual de Resource Manager
1. Más adelante, [migre los recursos adicionales][migrate-iaas] de la red virtual clásica según sea necesario.

## <a name="before-you-begin"></a>Antes de empezar

A medida que prepara y después migra un dominio administrado, existen algunas consideraciones que se deben tener en cuenta sobre la disponibilidad de los servicios de autenticación y administración. El dominio administrado no está disponible durante un período de tiempo de la migración. Asimismo, durante la migración se produce un tiempo de inactividad en las aplicaciones y los servicios que se basan en Azure AD DS.

> [!IMPORTANT]
> Lea por completo este artículo de migración y orientación antes de iniciar el proceso de migración. El proceso de migración afecta a la disponibilidad de los controladores de dominio de Azure AD DS durante períodos de tiempo. Los usuarios, los servicios y las aplicaciones no se pueden autenticar en el dominio administrado durante el proceso de migración.

### <a name="ip-addresses"></a>Direcciones IP

Las direcciones IP de controlador de dominio para un dominio administrado cambian después de la migración. Este cambio incluye la dirección IP pública para el punto de conexión de LDAP seguro. Las nuevas direcciones IP están dentro del intervalo de direcciones de la nueva subred de la red virtual de Resource Manager.

Si necesita realizar una reversión, es posible que las direcciones IP cambien después de esta.

Por lo general, Azure AD DS usa las dos primeras direcciones IP disponibles en el intervalo de direcciones, pero esto no se garantiza. Actualmente no se pueden especificar las direcciones IP que se van a usar después de la migración.

### <a name="downtime"></a>Tiempo de inactividad

El proceso de migración implica que los controladores de dominio están sin conexión durante un período de tiempo. No se puede acceder a los controladores de dominio mientras Azure AD DS se migra al modelo de implementación y la red virtual de Resource Manager.

El tiempo medio de inactividad es de 1 a 3 horas. Este período de tiempo comienza cuando los controladores de dominio se desconectan y termina en el momento en que el primer controlador de dominio vuelve a estar en línea. Esta media no incluye el tiempo que tarda el segundo controlador de dominio en replicarse ni el tiempo que pueden tardar los recursos adicionales en migrar al modelo de implementación de Resource Manager.

### <a name="account-lockout"></a>Bloqueo de cuenta

Los dominios administrados que se ejecutan en redes virtuales clásicas no tienen directivas de bloqueo de cuenta de AD. Si las máquinas virtuales están expuestas a Internet, los atacantes pueden usar métodos de difusión de contraseña para forzar el acceso a las cuentas. No hay ninguna directiva de bloqueo de cuenta para detener esos intentos. En el caso de los dominios administrados que usan el modelo de implementación y las redes virtuales de Resource Manager, las directivas de bloqueo de cuenta de AD protegen contra estos ataques de difusión de contraseña.

De forma predeterminada, cinco intentos de contraseña incorrecta en dos minutos bloquean una cuenta durante treinta minutos.

No se puede utilizar una cuenta bloqueada para iniciar sesión, lo que puede interferir con la capacidad de administrar el dominio administrado o las aplicaciones administradas por la cuenta. Después de migrar un dominio administrado, las cuentas pueden experimentar lo que parece un bloqueo permanente debido a repetidos intentos erróneos de inicio de sesión. Tras la migración, estos son dos escenarios comunes:

* Una cuenta de servicio que usa una contraseña expirada.
    * La cuenta de servicio intenta iniciar sesión repetidamente con una contraseña expirada, lo que bloquea la cuenta. Para corregirlo, busque la aplicación o la máquina virtual con credenciales expiradas y actualice la contraseña.
* Una entidad malintencionada usa intentos de fuerza bruta para iniciar sesión en las cuentas.
    * Cuando las máquinas virtuales se exponen a Internet, los atacantes suelen probar combinaciones de nombre de usuario y contraseña comunes mientras intentan iniciar sesión. Estos intentos repetidos de inicio de sesión erróneos pueden bloquear las cuentas. No se recomienda usar cuentas de administrador con nombres genéricos como *admin* o *administrador*, por ejemplo, para reducir la posibilidad de que se bloqueen las cuentas administrativas.
    * Reduzca el número de máquinas virtuales que se exponen a Internet. Puede usar [Azure Bastion][azure-bastion] para conectarse de forma segura a las máquinas virtuales mediante Azure Portal.

Si sospecha que algunas cuentas pueden estar bloqueadas después de la migración, en los pasos finales de la migración se describe cómo habilitar la auditoría o cómo cambiar la configuración de la directiva de contraseñas específica.

### <a name="roll-back-and-restore"></a>Reversión y restauración

Si la migración no se realiza correctamente, hay un proceso para revertir o restaurar un dominio administrado. La reversión es una opción de autoservicio para devolver inmediatamente el estado del dominio administrado a la situación anterior al intento de migración. Como último recurso, los ingenieros de soporte técnico de Azure también pueden restaurar un dominio administrado a partir de una copia de seguridad. Para más información, consulte [Reversión o restauración de una migración errónea](#roll-back-and-restore-from-migration).

### <a name="restrictions-on-available-virtual-networks"></a>Restricciones en las redes virtuales disponibles

Existen algunas restricciones en las redes virtuales a las que se puede migrar un dominio administrado. La red virtual de Resource Manager de destino debe cumplir los siguientes requisitos:

* La red virtual de Resource Manager debe estar en la misma suscripción de Azure que la red virtual clásica en la que Azure AD DS está actualmente implementado.
* La red virtual de Resource Manager debe estar en la misma región que la red virtual clásica en la que Azure AD DS está actualmente implementado.
* La subred de la red virtual de Resource Manager debe tener al menos de tres a cinco direcciones IP disponibles.
* La subred de la red virtual de Resource Manager debe ser una subred dedicada para Azure AD DS y no debe hospedar otras cargas de trabajo.

Para más información sobre los requisitos de red virtual, consulte [Consideraciones de diseño y opciones de configuración de redes virtuales][network-considerations].

## <a name="migration-steps"></a>Pasos de migración

La migración al modelo de implementación y la red virtual de Resource Manager se divide en cinco pasos principales:

| Paso    | Se realiza en  | Tiempo estimado  | Tiempo de inactividad  | ¿Reversión o restauración? |
|---------|--------------------|-----------------|-----------|-------------------|
| [Paso 1: Actualización y ubicación de la nueva red virtual](#update-and-verify-virtual-network-settings) | Azure portal | 15 minutos | No se requiere tiempo de inactividad. | N/D |
| [Paso 2: Preparación del dominio administrado para la migración](#prepare-the-managed-domain-for-migration) | PowerShell | De 15 a 30 minutos como media | El tiempo de inactividad de Azure AD DS comienza una vez finalizado este comando. | Reversión y restauración disponibles. |
| [Paso 3: Traslado del dominio administrado a una red virtual existente](#migrate-the-managed-domain) | PowerShell | De 1 a 3 horas como media | Una vez finalizado este comando está disponible un controlador de dominio; finaliza el tiempo de inactividad. | En caso de error, están disponibles la reversión (autoservicio) y la restauración. |
| [Paso 4: Prueba y espera del controlador de dominio de réplica](#test-and-verify-connectivity-after-the-migration)| PowerShell y Azure Portal | 1 hora o más, en función del número de pruebas | Ambos controladores de dominio están disponibles y deben funcionar con normalidad. | N/D Una vez que la primera máquina virtual se ha migrado correctamente, no hay ninguna opción de reversión o restauración. |
| [Paso 5: Pasos de configuración opcionales](#optional-post-migration-configuration-steps) | Azure Portal y máquinas virtuales | N/D | No se requiere tiempo de inactividad. | N/D |

> [!IMPORTANT]
> Para evitar tiempo de inactividad adicional, lea por completo este artículo de migración y orientación antes de iniciar el proceso de migración. El proceso de migración afecta a la disponibilidad de los controladores de dominio de Azure AD DS durante un período de tiempo. Los usuarios, los servicios y las aplicaciones no se pueden autenticar en el dominio administrado durante el proceso de migración.

## <a name="update-and-verify-virtual-network-settings"></a>Actualización y comprobación de la configuración de red virtual

Antes de comenzar el proceso de migración, lleve a cabo las siguientes comprobaciones y actualizaciones iniciales. Estos pasos pueden producirse en cualquier momento antes de la migración y no afectan al funcionamiento del dominio administrado.

1. Actualice el entorno de Azure PowerShell local a la versión más reciente. Para completar los pasos de migración, necesita al menos la versión *2.3.2*.

    Para más información sobre cómo realizar la comprobación y la actualización de la versión de PowerShell, consulte [Introducción a Azure PowerShell][azure-powershell].

1. Cree una red virtual de Resource Manager o seleccione una existente.

    Asegúrese de que la configuración de red no bloquea los puertos necesarios para Azure AD DS. Los puertos deben estar abiertos tanto en la red virtual clásica como en la red virtual de Resource Manager. Esta configuración incluye las tablas de rutas (aunque no se recomienda usarlas) y los grupos de seguridad de red.

    Para ver los puertos necesarios, consulte [Grupos de seguridad de red y puertos necesarios][network-ports]. Para reducir a un mínimo los problemas de comunicación de red, se recomienda esperar y aplicar un grupo de seguridad de red o una tabla de rutas a la red virtual de Resource Manager después de que la migración se haya completado correctamente.

    Tome nota de este grupo de recursos de destino, de la red virtual de destino y de la subred de la red virtual de destino. Estos nombres de recursos se usan durante el proceso de migración.

1. Compruebe el estado del dominio administrado en Azure Portal. Si hay alguna alerta para el dominio administrado, resuélvala antes de iniciar el proceso de migración.
1. Opcionalmente, si tiene previsto trasladar otros recursos al modelo de implementación y la red virtual de Resource Manager, confirme que esos recursos se pueden migrar. Para más información, consulte [Migración compatible con la plataforma de recursos de IaaS del modelo clásico al de Resource Manager][migrate-iaas].

    > [!NOTE]
    > No convierta la red virtual clásica a una red virtual de Resource Manager. Si lo hace, no dispondrá de ninguna opción para revertir o restaurar el dominio administrado.

## <a name="prepare-the-managed-domain-for-migration"></a>Preparación del dominio administrado para la migración

Para preparar el dominio administrado para la migración se utiliza Azure PowerShell. Estos pasos incluyen realizar una copia de seguridad, pausar la sincronización y eliminar el servicio en la nube que hospeda a Azure AD DS. Cuando se completa este paso, Azure AD DS se desconecta durante un período de tiempo. Si se produce un error en el paso de preparación, se puede [revertir al estado anterior](#roll-back).

Para preparar el dominio administrado para la migración, realice los siguientes pasos:

1. Instale el script `Migrate-Aaads` de la [Galería de PowerShell][powershell-script]. Este script de migración de PowerShell está firmado digitalmente por el equipo de ingeniería de Azure AD.

    ```powershell
    Install-Script -Name Migrate-Aadds
    ```

1. Utilice el cmdlet [Get-Credential][get-credential] para crear una variable que contenga las credenciales que utilizará el script de migración.

    La cuenta de usuario que especifique necesitará privilegios de *administrador global* en el inquilino de Azure AD, para habilitar Azure AD DS, y privilegios de *colaborador* en la suscripción de Azure, para crear los recursos de Azure AD DS necesarios.

    Cuando se le solicite, escriba una cuenta de usuario y una contraseña apropiadas:

    ```powershell
    $creds = Get-Credential
    ```
    
1. Defina una variable para el identificador de suscripción de Azure. Si es necesario, puede usar el cmdlet [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription) para enumerar y ver los identificadores de suscripción. Proporcione su propio identificador de suscripción en el comando siguiente:

   ```powershell
   $subscriptionId = 'yourSubscriptionId'
   ```

1. Ahora ejecute el cmdlet `Migrate-Aadds` con el parámetro *-Prepare*. Proporcione el valor *-ManagedDomainFqdn* de su propio dominio administrado, como *aaddscontoso.com*:

    ```powershell
    Migrate-Aadds `
        -Prepare `
        -ManagedDomainFqdn aaddscontoso.com `
        -Credentials $creds `
        -SubscriptionId $subscriptionId
    ```

## <a name="migrate-the-managed-domain"></a>Migración del dominio administrado

Ahora que el dominio administrado está preparado y se ha efectuado la copia de seguridad, se puede migrar el dominio. En este paso se vuelven a crear las máquinas virtuales del controlador de dominio de Azure AD DS con el modelo de implementación de Resource Manager. Este paso puede tardar de 1 a 3 horas en completarse.

Ejecute el cmdlet `Migrate-Aadds` con el parámetro *-Commit*. Proporcione el valor *-ManagedDomainFqdn* de su propio dominio administrado preparado en la sección anterior, como *aaddscontoso.com*:

Especifique el grupo de recursos de destino que contiene la red virtual a la que desea migrar Azure AD DS como, por ejemplo, *myResourceGroup*. Indique la red virtual de destino, por ejemplo *myVnet*, y la subred, por ejemplo *DomainServices*.

Después de ejecutar este comando, ya no se puede revertir:

```powershell
Migrate-Aadds `
    -Commit `
    -ManagedDomainFqdn aaddscontoso.com `
    -VirtualNetworkResourceGroupName myResourceGroup `
    -VirtualNetworkName myVnet `
    -VirtualSubnetName DomainServices `
    -Credentials $creds `
    -SubscriptionId $subscriptionId
```

Después de que el script valide que el dominio administrado está preparado para la migración, escriba *Y* para iniciar el proceso de migración.

> [!IMPORTANT]
> No convierta la red virtual clásica a una red virtual de Resource Manager durante el proceso de migración. Si efectúa esta conversión, ya no podrá revertir ni restaurar el dominio administrado, puesto que la red virtual original habrá dejado de existir.

Durante el proceso de migración, un indicador de progreso informa cada dos minutos del estado actual, tal como se muestra en la siguiente salida de ejemplo:

![Indicador de progreso de la migración de Azure AD DS](media/migrate-from-classic-vnet/powershell-migration-status.png)

El proceso de migración continúa ejecutándose aunque se cierre el script de PowerShell. En Azure Portal, el estado del dominio administrado aparece como *Migrando*.

Cuando la migración se completa correctamente, puede ver la dirección IP del primer controlador de dominio en Azure Portal o mediante Azure PowerShell. También se muestra una estimación del tiempo que tardará el segundo controlador de dominio en estar disponible.

En esta fase, puede trasladar otros recursos existentes desde el modelo de implementación y la red virtual clásicas. O bien, puede mantener los recursos en el modelo de implementación clásica y emparejar las redes virtuales entre sí una vez finalizada la migración de Azure AD DS.

## <a name="test-and-verify-connectivity-after-the-migration"></a>Prueba y comprobación de la conectividad después de la migración

El segundo controlador de dominio puede tardar un tiempo en implementarse correctamente y estar disponible para su uso en el dominio administrado.

Con el modelo de implementación de Resource Manager, los recursos de red para el dominio administrado se muestran en Azure Portal o en Azure PowerShell. Para más información sobre qué son y qué hacen estos recursos de red, consulte [Recursos de red usados por Azure AD DS][network-resources].

Cuando haya al menos un controlador de dominio disponible, complete los siguientes pasos de configuración para la conectividad de red con las máquinas virtuales:

* **Actualice la configuración del servidor DNS**. Para permitir que otros recursos de la red virtual de Resource Manager resuelvan y usen el dominio administrado, actualice la configuración de DNS con las direcciones IP de los nuevos controladores de dominio. Azure Portal puede configurar automáticamente estos valores.

    Para más información sobre cómo configurar la red virtual de Resource Manager, consulte [Actualización de la configuración DNS para la red virtual de Azure][update-dns].
* **Reinicie las máquinas virtuales unidas a los dominios**. Las direcciones IP del servidor DNS para los controladores de dominio de Azure AD DS cambian, por lo que debe reiniciar las máquinas virtuales unidas a los dominios; de este modo usarán la nueva configuración de servidor DNS. Si las aplicaciones o las máquinas virtuales tienen configuraciones de DNS manuales, actualícelas también manualmente con las nuevas direcciones IP del servidor DNS para los controladores de dominio que se muestran en Azure Portal.

Ahora pruebe la conexión y la resolución de nombres de la red virtual. En una máquina virtual conectada a la red virtual de Resource Manager, o emparejada con ella, realice las siguientes pruebas de comunicación de red:

1. Compruebe si puede hacer ping a la dirección IP de uno de los controladores de dominio; por ejemplo, `ping 10.1.0.4`.
    * Las direcciones IP de los controladores de dominio se muestran en la página **Propiedades** del dominio administrado en Azure Portal.
1. Compruebe la resolución de nombres del dominio administrado; por ejemplo, `nslookup aaddscontoso.com`.
    * Especifique el nombre DNS de su propio dominio administrado para comprobar que la configuración de DNS es correcta y se resuelve.

El segundo controlador de dominio debe estar disponible de una a dos horas después de que finalice el cmdlet de migración. Para comprobar si el segundo controlador de dominio está disponible, consulte la página **Propiedades** del dominio administrado en Azure Portal. Si se muestran dos direcciones IP, el segundo controlador de dominio está listo.

## <a name="optional-post-migration-configuration-steps"></a>Pasos de configuración opcionales posteriores a la migración

Cuando el proceso de migración se completa correctamente, algunos pasos de configuración opcionales incluyen la habilitación de registros de auditoría o de notificaciones por correo electrónico, o la actualización de la directiva de contraseñas específica.

### <a name="subscribe-to-audit-logs-using-azure-monitor"></a>Suscripción a los registros de auditoría mediante Azure Monitor

Azure AD DS expone registros de auditoría para ayudar a solucionar problemas y ver eventos en los controladores de dominio. Para más información, consulte [Habilitación y uso de los registros de auditoría][security-audits].

Puede usar plantillas para supervisar información importante expuesta en los registros. Por ejemplo, la plantilla de libro del registro de auditoría puede supervisar posibles bloqueos de cuenta en el dominio administrado.

### <a name="configure-email-notifications"></a>Configuración de notificaciones de correo electrónico

Para recibir una notificación cuando se detecte un problema en el dominio administrado, actualice la configuración de notificaciones por correo electrónico en Azure Portal. Para más información, consulte [Configuración de las opciones de notificación][notifications].

### <a name="update-fine-grained-password-policy"></a>Actualización de la directiva de contraseñas específica

Si es necesario, puede actualizar la directiva de contraseñas específica para que sea menos restrictiva que la configuración predeterminada. Puede usar los registros de auditoría para determinar si tiene sentido una configuración menos restrictiva y, después, puede configurar la directiva según sea necesario. Use los siguientes pasos generales para revisar y actualizar la configuración de directiva para las cuentas que se bloquean repetidamente después de la migración:

1. [Configure la directiva de contraseñas][password-policy] de modo que haya menos restricciones en el dominio administrado y observe los eventos de los registros de auditoría.
1. Si los registros de auditoría identifican cuentas de servicio que usan contraseñas expiradas, actualice esas cuentas con la contraseña correcta.
1. Si la máquina virtual está expuesta a Internet, revise los nombres de cuenta genéricos, como *administrador*, *usuario* o *invitado*, con un gran número de intentos de inicio de sesión. Siempre que sea posible, actualice estas máquinas virtuales para que usen cuentas con nombre menos genéricos.
1. Use un seguimiento de red en la máquina virtual para localizar el origen de los ataques y bloquee esas direcciones IP para que no puedan realizar intentos de inicio de sesión.
1. Cuando haya problemas mínimos de bloqueo, actualice la directiva de contraseñas específica para que sea tan restrictiva como sea necesario.

### <a name="creating-a-network-security-group"></a>Creación de un grupo de seguridad de red

Azure AD DS necesita un grupo de seguridad de red para proteger los puertos necesarios para el dominio administrado y bloquear el resto del tráfico entrante. Este grupo de seguridad de red actúa como un nivel de protección adicional para bloquear el acceso al dominio administrado y no se crea automáticamente. Para crear el grupo de seguridad de red y abrir los puertos necesarios, revise los pasos siguientes:

1. En Azure Portal, seleccione el recurso Azure AD DS. En la página Información general se muestra un botón para crear un grupo de seguridad de red si no hay ninguno asociado a Azure AD Domain Services.
1. Si usa LDAP seguro, agregue una regla al grupo de seguridad de red para permitir el tráfico entrante en el puerto *TCP* *636*. Para más información, consulte [Configuración de LDAP seguro][secure-ldap].

## <a name="roll-back-and-restore-from-migration"></a>Reversión y restauración de la migración

Hasta cierto punto en el proceso de migración, puede optar por revertir o restaurar el dominio administrado.

### <a name="roll-back"></a>Reversión

Si se produce un error al ejecutar el cmdlet de PowerShell para preparar la migración en el paso 2 o para la propia migración en el paso 3, el dominio administrado puede revertirse a la configuración original. Esta reversión requiere la red virtual clásica original. Las direcciones IP todavía pueden cambiar después de la reversión.

Ejecute el cmdlet `Migrate-Aadds` con el parámetro *-Abort*. Proporcione el valor *-ManagedDomainFqdn* de su propio dominio administrado preparado en una sección anterior, como *aaddscontoso.com*, y el nombre de la red virtual clásica, como *myClassicVnet*:

```powershell
Migrate-Aadds `
    -Abort `
    -ManagedDomainFqdn aaddscontoso.com `
    -ClassicVirtualNetworkName myClassicVnet `
    -Credentials $creds `
    -SubscriptionId $subscriptionId
```

### <a name="restore"></a>Restauración

Como último recurso, Azure AD Domain Services se puede restaurar a partir de la última copia de seguridad disponible. En el paso 1 de la migración se realiza una copia de seguridad, para asegurarse de que se dispone de la copia de seguridad más reciente. Esta copia se almacena durante treinta días.

Para restaurar el dominio administrado a partir de una copia de seguridad, [abra una incidencia de soporte técnico mediante Azure Portal][azure-support]. Proporcione el identificador de directorio, el nombre de dominio y el motivo para efectuar la restauración. El proceso de soporte y restauración puede tardar varios días en completarse.

## <a name="troubleshooting"></a>Solución de problemas

Si tiene problemas después de la migración al modelo de implementación de Resource Manager, revise algunas de las siguientes áreas de solución de problemas comunes:

* [Solución de problemas de unión a un dominio][troubleshoot-domain-join]
* [Solución de problemas de bloqueo de cuentas][troubleshoot-account-lockout]
* [Solución de problemas de inicio de sesión de cuentas][troubleshoot-sign-in]
* [Solución de problemas de conectividad con LDAP seguro][tshoot-ldaps]

## <a name="next-steps"></a>Pasos siguientes

Con el dominio administrado migrado al modelo de implementación de Resource Manager, [cree una máquina virtual de Windows y únase a un dominio][join-windows] y, después, [instale las herramientas de administración][tutorial-create-management-vm].

<!-- INTERNAL LINKS -->
[azure-bastion]: ../bastion/bastion-overview.md
[network-considerations]: network-considerations.md
[azure-powershell]: /powershell/azure/
[network-ports]: network-considerations.md#network-security-groups-and-required-ports
[Connect-AzAccount]: /powershell/module/az.accounts/connect-azaccount
[Set-AzContext]: /powershell/module/az.accounts/set-azcontext
[Get-AzResource]: /powershell/module/az.resources/get-azresource
[Set-AzResource]: /powershell/module/az.resources/set-azresource
[network-resources]: network-considerations.md#network-resources-used-by-azure-ad-ds
[update-dns]: tutorial-create-instance.md#update-dns-settings-for-the-azure-virtual-network
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
[security-audits]: security-audit-events.md
[notifications]: notifications.md
[password-policy]: password-policy.md
[secure-ldap]: tutorial-configure-ldaps.md
[migrate-iaas]: ../virtual-machines/windows/migration-classic-resource-manager-overview.md
[join-windows]: join-windows-vm.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md
[troubleshoot-domain-join]: troubleshoot-domain-join.md
[troubleshoot-account-lockout]: troubleshoot-account-lockout.md
[troubleshoot-sign-in]: troubleshoot-sign-in.md
[tshoot-ldaps]: tshoot-ldaps.md
[get-credential]: /powershell/module/microsoft.powershell.security/get-credential
[migration-benefits]: concepts-migration-benefits.md

<!-- EXTERNAL LINKS -->
[powershell-script]: https://www.powershellgallery.com/packages/Migrate-Aadds/
