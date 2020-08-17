---
author: tanmaygore
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: tagore
ms.openlocfilehash: 82c081ee4bbe78a534f0ad7bca949e9a50142e35
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088805"
---
## <a name="migrate-iaas-resources-from-the-classic-deployment-model-to-azure-resource-manager"></a>Migración de recursos IaaS del modelo de implementación clásica a Azure Resource Manager
En primer lugar, es importante conocer la diferencia entre operaciones del plano de datos y del plano de administración en los recursos de infraestructura como servicio (IaaS).

* El *plano de administración o control* describe las llamadas que entran en el plano de administración o control, o la API para modificar los recursos. Por ejemplo, operaciones como crear una máquina virtual, reiniciar una máquina virtual y actualizar una red virtual con una nueva subred administran los recursos en ejecución. No afectan directamente a la conexión con las máquinas virtuales.
* El *plano de datos* (aplicación) describe el runtime de la aplicación en sí e incluye la interacción con instancias que no pasan por la API de Azure. Por ejemplo, el acceso a su sitio web o la extracción de datos desde una instancia de SQL Server o un servidor MongoDB en ejecución se consideran interacciones de aplicación o del plano de datos. Otros ejemplos incluyen la copia de un blob de una cuenta de almacenamiento y el acceso a una dirección IP pública para usar el Protocolo de escritorio remoto (RDP) o Secure Shell (SSH) en la máquina virtual. Estas operaciones mantienen la aplicación en ejecución en procesos, redes y almacenamiento.

El plano de datos es el mismo entre el modelo de implementación clásica y las pilas de Resource Manager. La diferencia es que durante el proceso de migración, Microsoft traslada la representación de los recursos del modelo de implementación clásica al de la pila de Resource Manager. Como consecuencia, es preciso que use las nuevas herramientas, API y SDK para administrar los recursos de la pila de Resource Manager.

![Diagrama que muestra la diferencia entre el plano de administración o control, y el plano de datos](../articles/virtual-machines/media/virtual-machines-windows-migration-classic-resource-manager/data-control-plane.png)


> [!NOTE]
> En algunos escenarios de migración, la plataforma Azure se detiene, desasigna y reinicia las máquinas virtuales. Esto provoca que un breve período de inactividad en el plano de datos.
>

## <a name="the-migration-experience"></a>Experiencia de migración
Antes de empezar la migración:

* Asegúrese de que los recursos que desea migrar no usan las características o configuraciones no admitidas. Normalmente, la plataforma detecta estos problemas y genera un error.
* Si tiene máquinas virtuales que no están en una red virtual, estás se detienen y se desasignan, pero ello forma parte de la operación de preparación. Si no desea perder la dirección IP pública, considere la posibilidad de reservar la dirección IP antes de desencadenar la operación de preparación. Si las máquinas virtuales están en una red virtual, no se detienen ni se desasignan.
* Planee la migración fuera del horario de actividad, para dar cabida a errores inesperados que surjan durante ella.
* Descargue la configuración actual de las máquinas virtuales mediante PowerShell, los comandos de la interfaz de la línea de comandos (CLI) o las API de REST para facilitar la validación una vez que finalice el paso de preparación.
* Actualice los scripts de automatización y operacionalización para controlar el modelo de implementación con Resource Manager antes de iniciar la migración. También tiene la opción de realizar las operaciones GET cuando los recursos se encuentren en el estado preparado.
* Evalúe las directivas de control de acceso basado en rol (RBAC) configuradas en los recursos de IaaS en el modelo de implementación clásica y defina un plan para cuando se haya completado la migración.

El flujo de trabajo de la migración es el siguiente:

![Diagrama que muestra el flujo de trabajo de migración](../articles/virtual-machines/windows/media/migration-classic-resource-manager/migration-workflow.png)

> [!NOTE]
> Las operaciones que se describen en las siguientes secciones son todas idempotentes. Si tiene un problema que no sea que alguna función no se admite o un error de configuración, vuelva a intentar la operación de preparación, anulación o confirmación. Azure vuelve a intentar la acción.
>
>

### <a name="validate"></a>Validación
La operación de validación es el primer paso del proceso de migración. El objetivo de este paso es analizar el estado de los recursos que desea migrar en el modelo de implementación clásica. La operación evalúa si los recursos son capaces de realizar la migración (éxito o error).

Se seleccionará la red virtual o el servicio en la nube (si no está en una red virtual) que se quiere validar para la migración. Si el recurso no es capaz de realizar la migración, Azure enumera los motivos.

#### <a name="checks-not-done-in-the-validate-operation"></a>Comprobaciones que no se realizan en la operación de validación

La operación de validación solo analiza el estado de los recursos en el modelo de implementación clásica. En este modelo, es posible comprobar todos los errores y escenarios no admitidos debido a diversas configuraciones. Sin embargo, no es posible comprobar todos los problemas que podría imponer la pila de Azure Resource Manager sobre los recursos durante la migración. Estos problemas solo se comprueban cuando los recursos llevan a cabo la transformación en el siguiente paso de la migración (la operación de preparación). En la tabla siguiente se enumeran todos los problemas que no se comprueban en la operación de validación:


|Comprobaciones de red que no se realizan en la operación de validación|
|-|
|Que una red virtual tiene puertas de enlace de ER y de VPN.|
|Que una conexión de un puerta de enlace de red virtual se encuentra en estado de desconexión.|
|Todos los circuitos de ER se han migrado previamente a la pila de Azure Resource Manager.|
|Comprobaciones de la cuota Azure Resource Manager en los recursos de red. Por ejemplo: dirección IP pública estática, direcciones IP públicas dinámicas, equilibrador de carga, grupos de seguridad de red, tablas de rutas e interfaces de red. |
| Todas las reglas del equilibrador de carga son válidas en las implementaciones y la red virtual. |
| Conflictos de direcciones IP privadas entre máquinas virtuales con desasignación detenida de la misma red virtual. |

### <a name="prepare"></a>Preparación
La operación de preparación es el secundo paso del proceso de migración. El objetivo de este paso es simular la transformación de los recursos de IaaS desde el modelo de implementación clásica a los recursos de Resource Manager. Además, la operación de preparación presenta ambos en paralelo para facilitar su visualización.

> [!NOTE] 
> Los recursos del modelo de implementación clásica no se modifican en este paso. Es un paso seguro de ejecutar si se prueba la migración. 

Se seleccionará la red virtual o el servicio en la nube (si no es una red virtual) que se quiere preparar para la migración.

* Si el recurso no se puede migrar, Azure detiene el proceso de migración y muestra el motivo de error de la operación de preparación.
* Si el recurso se puede migrar, Azure bloquea las operaciones en el plano de administración de los recursos en proceso de migración. Por ejemplo, no puede agregar un disco de datos a una máquina virtual en proceso de migración.

Luego, Azure comienza la migración de los metadatos del modelo de implementación clásica al de Resource Manager para los recursos que se migran.

Una vez que se completa la operación de preparación, tiene la opción de visualizar los recursos tanto en el modelo de implementación clásica como en el de Resource Manager. Para cada servicio en la nube en el modelo de implementación clásica, la plataforma de Azure crea un nombre de grupo de recursos con el patrón `cloud-service-name>-Migrated`.

> [!NOTE]
> No es posible seleccionar el nombre de un grupo de recursos creado para los recursos migrados (es decir, "-Migrated"). Sin embargo, una vez que se haya completado la migración, puede usar la característica de movimiento de Azure Resource Manager para mover recursos al grupo de recursos que desee. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../articles/resource-group-move-resources.md).

Las dos capturas de pantalla siguientes muestran el resultado después de una operación de preparación correcta. La primera muestra un grupo de recursos que contiene el servicio en la nube original. La segunda muestra el nuevo grupo de recursos "-Migrated" que contiene los recursos equivalentes de Azure Resource Manager.

![Captura de pantalla que muestra el servicio en la nube original](../articles/virtual-machines/windows/media/migration-classic-resource-manager/portal-classic.png)

![Captura de pantalla que muestra los recursos de Azure Resource Manager en la operación de preparación](../articles/virtual-machines/windows/media/migration-classic-resource-manager/portal-arm.png)

Esta es una vista en segundo plano de los recursos tras finalizar la fase de preparación. Tenga en cuenta que el recurso del plano de datos es el mismo. Se representa en el plano de administración (modelo de implementación clásica) y el plano de control (Resource Manager).

![Diagrama de la fase de preparación](../articles/virtual-machines/windows/media/migration-classic-resource-manager/behind-the-scenes-prepare.png)

> [!NOTE]
> Las máquinas virtuales que no se encuentran en una red virtual en el modelo de implementación clásica se detienen y se desasignan en esta fase de la migración.
>

### <a name="check-manual-or-scripted"></a>Comprobación (manual o mediante scripts)
En este paso de la comprobación, tiene la opción de usar la configuración que descargó antes para validar si la migración parece correcta. Como alternativa, puede iniciar sesión en el portal y revisar las propiedades y los recursos para validar que la migración de los metadatos es correcta.

Si están migrando una red virtual, no se reinicia la mayoría de la configuración de las máquinas virtuales. En el caso de las aplicaciones de dichas máquinas virtuales, puede validar si la aplicación sigue en ejecución.

Puede probar los scripts operacionales y de supervisión y automatización para ver si las máquinas virtuales funcionan de forma esperada y si los scripts actualizados funcionan correctamente. Tenga en cuenta que solo se admiten operaciones GET cuando los recursos se encuentran en el estado preparado.

No hay ninguna ventana de tiempo establecida antes del cual deba confirmar la migración. Puede mantenerse el tiempo que desee en este estado. No obstante, el plano de administración queda bloqueado para estos recursos hasta que lleve a cabo la operación de anulación o confirmación.

Si ve algún problema, siempre puede anular la migración y volver al modelo de implementación clásica. Si lo hace, Azure abrirá las operaciones del plano de administración en los recursos para que pueda reanudar las operaciones normales en esas máquinas virtuales en el modelo de implementación clásica.

### <a name="abort"></a>Anulación
Este paso es opcional si desea revertir los cambios realizados en el modelo de implementación clásica y detener la migración. Esta operación elimina los metadatos de Resource Manager (creados en el paso de preparación) en los recursos. 

![Diagrama de paso de anulación](../articles/virtual-machines/windows/media/migration-classic-resource-manager/behind-the-scenes-abort.png)


> [!NOTE]
> Esta operación no se puede realizar una vez que se ha desencadenado la operación de confirmación.     
>

### <a name="commit"></a>Commit
Después de finalizar la validación, puede confirmar la migración. Los recursos no aparecen en el modelo de implementación clásica y solo están disponibles en el modelo de implementación con Resource Manager. Los recursos migrados solo se pueden administrar en el nuevo portal.

> [!NOTE]
> Se trata de una operación idempotente. Si se produce un error, vuelva a intentar la operación. Si sigue teniendo errores, cree una incidencia de soporte técnico o cree un foro en [Microsoft Q&A](https://docs.microsoft.com/answers/index.html).
>
>

![Diagrama de paso de confirmación](../articles/virtual-machines/windows/media/migration-classic-resource-manager/behind-the-scenes-commit.png)

## <a name="migration-flowchart"></a>Diagrama de flujo de migración

Este es un diagrama de flujo que muestra cómo realizar la migración:

![Screenshot that shows the migration steps](../articles/virtual-machines/windows/media/migration-classic-resource-manager/migration-flow.png)

## <a name="translation-of-the-classic-deployment-model-to-resource-manager-resources"></a>Traslación del modelo de implementación clásica a los recursos de Resource Manager
Puede encontrar las representaciones del modelo de implementación clásica y de Resource Manager de los recursos en la tabla siguiente. Actualmente no se admiten otras funciones y recursos.

| Representación clásica | Representación de Resource Manager | Notas |
| --- | --- | --- |
| Nombre del servicio en la nube |Nombre DNS |Durante la migración, se crea un nuevo grupo de recursos para cada servicio en la nube con el patrón de nomenclatura `<cloudservicename>-migrated`. Este grupo de recursos contiene todos los recursos. El nombre del servicio en la nube se convierte en un nombre DNS que está asociado a la dirección IP pública. |
| Máquina virtual |Máquina virtual |Las propiedades específicas de la máquina virtual se migran sin cambios. Determinada información de osProfile, como el nombre de equipo, no se almacena en el modelo de implementación clásica y permanece vacía después de la migración. |
| Recursos de disco conectados a la máquina virtual |Implícitos discos conectados a la máquina virtual |Los discos no se modelan como recursos de nivel superior en el modelo de implementación de Resource Manager. Se migran como discos implícitos en la máquina virtual. Actualmente, se admiten solo los discos que están conectados a una VM. Las máquinas virtuales de Resource Manager ya pueden usar ahora cuentas de almacenamiento en el modelo de implementación clásica, los que permite que los discos se migren fácilmente sin actualizaciones. |
| Extensiones de máquina virtual |Extensiones de máquina virtual |Todas las extensiones de recursos, excepto las extensiones XML, se migran desde el modelo de implementación clásica. |
| Certificados de la máquina virtual |Certificados en Azure Key Vault |Si un servicio en la nube contiene certificados de servicio, la migración crea un nuevo almacén de claves de Azure por cada servicio en la nube y mueve los certificados a dicho almacén. Las máquinas virtuales se actualizan para hacer referencia a los certificados del almacén de claves. <br><br> No elimine el almacén de claves, ya que la máquina virtual puede ponerse en estado de error. |
| Configuración de WinRM |Configuración de WinRM en osProfile |La configuración de Administración remota de Windows se traslada tal cual como parte de la migración. |
| Propiedad de conjunto de disponibilidad |Recurso de conjunto de disponibilidad | La especificación del conjunto de disponibilidad es una propiedad de la máquina virtual en el modelo de implementación clásica. Los conjuntos de disponibilidad se convierten en un recurso de nivel superior como parte de la migración. Las siguientes configuraciones no son compatibles: varios conjuntos de disponibilidad por servicio en la nube o uno o varios conjuntos de disponibilidad junto con máquinas virtuales que no están en ningún conjunto de disponibilidad de un servicio en la nube. |
| Configuración de red en una máquina virtual |Interfaz de red principal |La configuración de red en una máquina virtual se representa como el recurso de la interfaz de red principal después de la migración. En el caso de las máquinas virtuales que no están en una red virtual, la dirección IP interna cambia durante la migración. |
| Varias interfaces de red en una máquina virtual |Interfaces de red |Si una máquina virtual tiene varias interfaces de red asociadas, cada una de ellas se convierte en un recurso de nivel superior como parte de la migración, junto con todas las propiedades. |
| Conjunto de punto de conexión de carga equilibrada |Equilibrador de carga |En el modelo de implementación clásica, la plataforma asignaba un equilibrador de carga implícito para cada servicio en la nube. Durante la migración, se crea un nuevo recurso de equilibrador de carga, y el conjunto de puntos de conexión con equilibrio de carga se convierte en reglas de equilibrador de carga. |
| Reglas NAT de entrada |Reglas NAT de entrada |Los puntos de conexión de entrada definidos en la VM se convierten en reglas de traducción de direcciones de red de entrada en el equilibrador de carga durante la migración. |
| Dirección VIP |Dirección IP pública con nombre DNS |La dirección IP virtual se convierte en una dirección IP pública y se asocia con el equilibrador de carga. Solo se puede migrar una dirección IP virtual si hay un punto de conexión de entrada asignado a ella. |
| Virtual network |Virtual network |La red virtual se migra con todas sus propiedades al modelo de implementación de Resource Manager. Se crea un nuevo grupo de recursos con el nombre `-migrated`. |
| Direcciones IP reservadas |Dirección IP pública con método de asignación estático |Las direcciones IP reservadas asociadas con el equilibrador de carga se migran junto con la migración del servicio en la nube o de la máquina virtual. Las direcciones IP reservadas no asociadas se pueden migrar mediante [Move-AzureReservedIP](/powershell/module/servicemanagement/azure.service/move-azurereservedip?view=azuresmps-4.0.0).  |
| Dirección IP pública por máquina virtual |Dirección IP pública con método de asignación dinámico |La dirección IP pública asociada a la máquina virtual se convierte como un recurso de dirección IP público con el método de asignación establecido en estático. |
| Grupos de seguridad de red |Grupos de seguridad de red |Los grupos de seguridad de red asociados a una subred se clonan como parte de la migración al modelo de implementación de Resource Manager. El grupo de seguridad de red no se quita en el modelo de implementación clásica durante la migración. Sin embargo, las operaciones de plano de administración para el grupo de seguridad de red se bloquean cuando la migración está en curso. Los NSG no asociados se pueden migrar mediante [Move-AzureNetworkSecurityGroup](/powershell/module/servicemanagement/azure.service/move-azurenetworksecuritygroup?view=azuresmps-4.0.0).|
| Servidores DNS |Servidores DNS |Los servidores DNS asociados con la máquina virtual o una red virtual se migran como parte de la migración de recursos correspondiente junto con todas las propiedades. |
| Rutas definidas por el usuario |Rutas definidas por el usuario |Las rutas definidas por el usuario asociadas a una subred se clonan como parte de la migración al modelo de implementación de Resource Manager. Las rutas definidas por el usuario no se quitan en el modelo de implementación clásica durante la migración. Las operaciones de plano de administración para las rutas definidas por el usuario se bloquean cuando la migración está en curso. Las UDR no asociadas se pueden migrar mediante [Move-AzureRouteTable](/powershell/module/servicemanagement/azure.service/Move-AzureRouteTable?view=azuresmps-4.0.0). |
| Propiedad de reenvío de IP en una configuración de red de máquina virtual |Propiedad de reenvío de IP en la NIC |La propiedad de reenvío de IP en una máquina virtual se convierte en una propiedad en la interfaz de red durante la migración. |
| Equilibrador de carga con varias IP |Equilibrador de carga con varios recursos de dirección IP pública |Todas las direcciones IP públicas asociadas con el equilibrador de carga se convierten en un recurso de dirección IP pública y se asocian con el equilibrador de carga después de la migración. |
| Nombres DNS internos en la máquina virtual |Nombres DNS internos en la NIC |Durante la migración, se migran los sufijos DNS internos para la máquina virtual a una propiedad de solo lectura denominada "InternalDomainNameSuffix" en la NIC. El sufijo no cambia después de la migración y la resolución de la máquina virtual debería seguir funcionando como antes. |
| Puerta de enlace de red virtual |Puerta de enlace de red virtual |Las propiedades de la puerta de enlace de red virtual se migran sin cambios. La VIP asociada a la puerta de enlace no cambia. |
| Sitio de red local |Puerta de enlace de red local |Las propiedades del sitio de red local se migran sin cambios a un nuevo recurso denominado puerta de enlace de red local. Esto representa prefijos de direcciones locales y la IP de la puerta de enlace remota. |
| Referencias de conexiones |Conexión |Las referencias de conectividad entre la puerta de enlace y el sitio de red local en la configuración de red se representan mediante un recurso nuevo denominado Connection. Todas las propiedades de referencia de conectividad de los archivos de configuración de red se copian sin cambios al recurso Connection. La conectividad entre redes virtuales en el modelo de implementación clásica se logra mediante la creación de dos túneles IPsec a sitios de red local que representan las redes virtuales. Esto se transforma en un tipo de conexión de red virtual a red virtual en el modelo de Resource Manager sin necesidad de puertas de enlace de red local. |

## <a name="changes-to-your-automation-and-tooling-after-migration"></a>Cambios en la automatización y las herramientas después de la migración
Como parte de la migración de los recursos desde el modelo de implementación clásica al de Resource Manager, es preciso actualizar la automatización o las herramientas existentes para asegurarse de que siguen funcionando después de la migración.
