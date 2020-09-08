---
title: Implementación de Resource Manager y clásica
description: Describe las diferencias entre el modelo de implementación del Administrador de recursos y el modelo de implementación clásica (o de administración del servicio).
ms.topic: conceptual
ms.date: 02/06/2020
ms.openlocfilehash: d9c1a27efc1f474e25d0ed164d106e8f64ba3e96
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88511412"
---
# <a name="azure-resource-manager-vs-classic-deployment-understand-deployment-models-and-the-state-of-your-resources"></a>Implementación mediante Azure Resource Manager frente a la implementación clásica: Conozca los modelos de implementación y el estado de los recursos

> [!NOTE]
> La información que se proporciona en este artículo se usa solo cuando se migra desde la implementación clásica a la implementación mediante Azure Resource Manager.

En este artículo aprenderá sobre Azure Resource Manager y los modelos de implementación clásicos. El modelo de implementación de Resource Manager y el modelo de implementación clásica representan dos formas diferentes de implementar y administrar las soluciones de Azure. Usted trabaja con ellos a través de dos conjuntos de API distintos y los recursos implementados pueden contener diferencias importantes. Los dos modelos no son compatibles entre sí. En este artículo se describen esas diferencias.

Para simplificar la implementación y administración de recursos, Microsoft recomienda que utilice Resource Manager para los nuevos recursos. Si es posible, Microsoft recomienda que vuelva a implementar los recursos existentes a través de Resource Manager.

Si no está familiarizado con Resource Manager, quizás quiera revisar primero la terminología definida en la [Introducción a Azure Resource Manager](overview.md).

## <a name="history-of-the-deployment-models"></a>Historia de los modelos de implementación

Originalmente, Azure solo proporcionaba el modelo de implementación clásica. En este modelo, cada recurso existía de forma independiente; no había manera de agrupar los recursos relacionados. En su lugar, debía realizarse un seguimiento manual de los recursos que componían la solución o aplicación, y acordarse de administrarlos de manera coordinada. Para implementar una solución, había que crear cada recurso individualmente mediante el portal o crear un script que implementara todos los recursos en el orden correcto. Para eliminar una solución, tenía que eliminar cada recurso individualmente. No se podían aplicar ni actualizar fácilmente las directivas de control de acceso para los recursos relacionados. Por último, no podía aplicar etiquetas a los recursos para etiquetarlos con términos que le ayudaran a supervisar los recursos y administrar la facturación.

En 2014, Azure presentó Resource Manager, que incorpora el concepto de un grupo de recursos. Un grupo de recursos es un contenedor para los recursos que comparten un ciclo de vida común. El modelo de implementación de Resource Manager ofrece varias ventajas:

* Puede implementar, administrar y supervisar todos los servicios para su solución como grupo, en lugar de controlar estos servicios individualmente.
* Puede implementar la solución repetidamente a lo largo del ciclo de vida y tener la seguridad de que los recursos se implementan de forma coherente.
* Puede aplicar control de acceso a todos los recursos del grupo de recursos y las directivas se aplican automáticamente cuando se agregan nuevos recursos al grupo de recursos.
* Puede aplicar etiquetas a los recursos para organizar de manera lógica todos los recursos de la suscripción.
* Puede usar la notación de objetos JavaScript (JSON) para definir la infraestructura de la solución. El archivo JSON se conoce como una plantilla de Resource Manager.
* Puede definir las dependencias entre recursos de modo que se implementen en el orden correcto.

Al agregarse el Administrador de recursos, todos los recursos se agregaron retroactivamente a los grupos de recursos predeterminados. Si ahora crea un recurso mediante la implementación clásica, el recurso se crea automáticamente dentro de un grupo de recursos predeterminado para el servicio, aunque no se especifique dicho grupo de recursos durante la implementación. Sin embargo, solo el hecho de existir dentro de un grupo de recursos no significa que el recurso se haya convertido al modelo de Resource Manager.

## <a name="understand-support-for-the-models"></a>Descripción de la compatibilidad para los modelos

Hay tres escenarios principales que se han de tener en cuenta:

1. Cloud Services no es compatible con el modelo de implementación de Resource Manager.
2. Las máquinas virtuales, las cuentas de almacenamiento y las redes virtuales son compatibles con Resource Manager y los modelos de implementación clásicos.
3. Todos los demás servicios de Azure admiten Resource Manager.

En el caso de las máquinas virtuales, las cuentas de almacenamiento y las redes virtuales, si el recurso se creó mediante la implementación clásica, debe continuar trabajando en él mediante las operaciones clásicas. Si la máquina virtual, la cuenta de almacenamiento o la red virtual se crearon con la implementación de Resource Manager, debe continuar usando operaciones de Resource Manager. Esta distinción puede resultar confusa cuando la suscripción contiene una combinación de los recursos creados mediante Resource Manager y la implementación clásica. Esta combinación de recursos puede crear resultados inesperados porque los recursos no admiten las mismas operaciones.

En algunos casos, un comando de Resource Manager puede recuperar información sobre un recurso creado mediante la implementación clásica, o puede realizar tareas administrativas tales como mover un recurso clásico a otro grupo de recursos. Sin embargo, estos casos no deben dar la impresión de que el tipo es compatible con las operaciones de Resource Manager. Por ejemplo, supongamos que tiene un grupo de recursos que contiene una máquina virtual creada con la implementación clásica. Si ejecuta el siguiente comando de PowerShell para Resource Manager:

```powershell
Get-AzResource -ResourceGroupName ExampleGroup -ResourceType Microsoft.ClassicCompute/virtualMachines
```

Devuelve la máquina virtual:

```powershell
Name              : ExampleClassicVM
ResourceId        : /subscriptions/{guid}/resourceGroups/ExampleGroup/providers/Microsoft.ClassicCompute/virtualMachines/ExampleClassicVM
ResourceName      : ExampleClassicVM
ResourceType      : Microsoft.ClassicCompute/virtualMachines
ResourceGroupName : ExampleGroup
Location          : westus
SubscriptionId    : {guid}
```

Sin embargo, el cmdlet **Get-AzVM** de Resource Manager solo devuelve las máquinas virtuales implementadas mediante Resource Manager. El siguiente comando no devuelve la máquina virtual creada mediante la implementación clásica.

```powershell
Get-AzVM -ResourceGroupName ExampleGroup
```

Solo los recursos creados a través del Administrador de recursos son compatibles con las etiquetas. No puede aplicar etiquetas a los recursos clásicos.

## <a name="changes-for-compute-network-and-storage"></a>Cambios de proceso, red y almacenamiento

En el siguiente diagrama se muestran los recursos de proceso, red y almacenamiento implementados a través de Resource Manager.

![Arquitectura de Resource Manager](./media/deployment-models/arm_arch3.png)

SRP: Proveedor de recursos de Storage, CRP: Proveedor de recursos de proceso, NRP: Proveedor de recursos de red

Tenga en cuenta las siguientes relaciones entre los recursos:

* Todos los recursos existen dentro de un grupo de recursos.
* La máquina virtual depende de una cuenta de almacenamiento específica definida en el proveedor de recursos de almacenamiento para almacenar sus discos en Blob Storage (obligatorio).
* La máquina virtual hace referencia a una tarjeta de interfaz de red específica definida en el proveedor de recursos de red (obligatorio) y un conjunto de disponibilidad definido en el proveedor de recursos de proceso (opcional).
* La tarjeta de interfaz de red hace referencia a la dirección IP asignada a la máquina virtual (obligatoria), la subred de la red virtual para la máquina virtual (obligatoria) y a un grupo de seguridad de red (opcional).
* La subred dentro de una red virtual hace referencia a un grupo de seguridad de red (opcional).
* La instancia de equilibrador de carga hace referencia al grupo de back-end de direcciones IP que incluye la tarjeta de interfaz de red de una máquina virtual (opcional) y hace referencia a una dirección IP pública o privada del equilibrador de carga (opcional).

Aquí se encuentran los componentes y sus relaciones para la implementación clásica:

![arquitectura clásica](./media/deployment-models/arm_arch1.png)

La solución clásica para hospedar una máquina virtual incluye:

* Un servicio de nube requerido que actúa como contenedor para hospedar máquinas virtuales (cálculo). Las máquinas virtuales se proporcionan automáticamente con una tarjeta de interfaz de red y una dirección IP asignada por Azure. Además, el servicio de nube contiene una instancia de equilibrador de carga externa, una dirección IP pública y extremos predeterminados para permitir un escritorio remoto y tráfico de PowerShell remoto para máquinas virtuales basadas en Windows y tráfico de Secure Shell (SSH) para máquinas virtuales basadas en Linux.
* Una cuenta de almacenamiento necesaria que almacena los discos duros virtuales para una máquina virtual, incluido el sistema operativo y los discos de datos temporales y adicionales (almacenamiento).
* Una red virtual opcional que actúa como un contenedor adicional, en el que se puede crear una estructura de subredes y elegir la subred en la que se encuentra la máquina virtual (red).

En la siguiente tabla se describen los cambios en la interacción de los proveedores de recursos de Compute, Network y Storage:

| Elemento | Clásico | Resource Manager |
| --- | --- | --- |
| Servicio en la nube para máquinas virtuales |El servicio en la nube es un contenedor para las máquinas virtuales que exige la disponibilidad de la plataforma y el equilibrio de carga. |El servicio en la nube ya no es un objeto necesario para crear una máquina virtual usando el nuevo modelo. |
| Virtual Networks |Una red virtual es opcional para la máquina virtual. En caso de incluirse, la red virtual no se puede implementar con Resource Manager. |La máquina virtual requiere una red virtual que se ha implementado con Resource Manager. |
| Cuentas de almacenamiento |La máquina virtual requiere una cuenta de almacenamiento que almacena los discos duros virtuales para el sistema operativo, el almacenamiento temporal y los discos de datos adicionales. |La máquina virtual requiere una cuenta de almacenamiento para almacenar sus discos en Blob Storage. |
| Conjuntos de disponibilidad |La disponibilidad en la plataforma se ha indicado mediante la configuración del mismo "AvailabilitySetName" en las máquinas virtuales. El número máximo de dominios con error era de 2. |Un conjunto de disponibilidad es un recurso expuesto por el proveedor de Microsoft.Compute. Las máquinas virtuales que requieren alta disponibilidad deben incluirse en el conjunto de disponibilidad. Ahora, el recuento máximo de dominios con error es de 3. |
| Grupos de afinidad |Los grupos de afinidad eran necesarios para crear redes virtuales. Sin embargo, con la introducción de las redes virtuales regionales, ya no era necesario. |Para simplificar, no existe el concepto de grupos de afinidad en las API expuestas a través del Administrador de recursos de Azure. |
| Equilibrio de carga |La creación de un servicio en la nube proporciona un equilibrador de carga implícito para las máquinas virtuales implementadas. |El equilibrador de carga es un recurso expuesto por el proveedor de Microsoft.Network. La interfaz de red principal de las máquinas virtuales cuya carga se debe equilibrar debe hacer referencia al equilibrador de carga. Los equilibradores de carga pueden ser internos o externos. Una instancia de equilibrador de carga hace referencia al grupo de backend de direcciones IP que incluye la NIC de una máquina virtual (opcional) y hace referencia a una dirección IP pública o privada del equilibrador de carga (opcional). |
| Dirección IP virtual |Cloud Services obtiene una VIP (dirección IP virtual) predeterminada cuando se agrega una máquina virtual a un servicio en la nube. La dirección IP virtual es la dirección asociada al equilibrador de carga implícito. |La dirección IP pública es un recurso expuesto por el proveedor de Microsoft.Network. La dirección IP pública puede ser estática (reservada) o dinámica. Las direcciones IP públicas dinámicas pueden asignarse a un equilibrador de carga. Las direcciones IP públicas se pueden proteger mediante grupos de seguridad. |
| Dirección IP reservada |Puede reservar una dirección IP en Azure y asociarlo a un servicio en la nube para asegurarse de que la dirección IP es permanente. |La dirección IP pública puede crearse en modo "Estático" y ofrece la misma capacidad que una "dirección IP reservada". |
| Dirección IP pública (PIP) por máquina virtual |Las direcciones IP públicas también se pueden asociar directamente a una VM. |La dirección IP pública es un recurso expuesto por el proveedor de Microsoft.Network. La dirección IP pública puede ser estática (reservada) o dinámica. |
| Puntos de conexión |Es necesario configurar los extremos de entrada en una máquina virtual para abrir la conectividad para determinados puertos. Uno de los modos comunes de conectarse a las máquinas virtuales es mediante la configuración de extremos de entrada. |Las reglas de entrada NAT se puede configurar en equilibradores de carga para lograr la misma capacidad de habilitar los extremos en puertos específicos para conectarse a las máquinas virtuales. |
| Nombre DNS |Un servicio en la nube obtendría un nombre de DNS único global implícito. Por ejemplo: `mycoffeeshop.cloudapp.net`. |Los nombres DNS son parámetros opcionales que se pueden especificar en un recurso de dirección IP pública. El nombre de dominio completo tiene el siguiente formato: `<domainlabel>.<region>.cloudapp.azure.com`. |
| Interfaces de red |La interfaz de red principal y secundaria y sus propiedades se definieron como configuración de red de una máquina virtual. |La interfaz de red es un recurso expuesto por el proveedor de Microsoft.Network. El ciclo de vida de la interfaz de red no está asociado a una máquina virtual. Hace referencia a la dirección IP asignada a la máquina virtual (obligatoria), la subred de la red virtual para la máquina virtual (obligatoria) y a un grupo de seguridad de red (opcional). |

Para obtener información sobre cómo conectar redes virtuales de diferentes modelos de implementación, consulte [Conexión a redes virtuales a partir de diferentes modelos de implementación del portal](../../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md).

## <a name="migrate-from-classic-to-resource-manager"></a>Migración de implementaciones clásicas a Resource Manager

Si está listo para migrar los recursos de la implementación clásica a la implementación de Resource Manager, consulte:

1. [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-deep-dive.md)
2. [Migración compatible con la plataforma de recursos de IaaS del modelo clásico a Azure Resource Manager](../../virtual-machines/windows/migration-classic-resource-manager-overview.md)
3. [Migración de recursos de IaaS de la implementación clásica a la de Resource Manager con Azure PowerShell](../../virtual-machines/windows/migration-classic-resource-manager-ps.md)
4. [Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure](../../virtual-machines/linux/migration-classic-resource-manager-cli.md)

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**¿Puedo crear una máquina virtual mediante Resource Manager para implementar en una red virtual creada mediante la implementación clásica?**

No se admite esta configuración. No se puede usar Resource Manager para implementar una máquina virtual en una red virtual que se creó con la implementación clásica.

**¿Puedo crear una máquina virtual mediante Resource Manager desde una imagen de usuario que se creó utilizando el modelo de implementación clásico?**

No se admite esta configuración. Sin embargo, puede copiar los archivos de los discos duros virtuales de una cuenta de almacenamiento que se creó mediante el modelo de implementación clásico y agregarlos a una cuenta nueva creada mediante Resource Manager.

**¿Cuál es el impacto en la cuota de mi suscripción?**

Las cuotas de las máquinas virtuales, redes virtuales y cuentas de almacenamiento creadas mediante Azure Resource Manager son independientes de otras cuotas. Cada suscripción obtiene cuotas para crear los recursos mediante las nuevas API. Puede leer más acerca de las cuotas adicionales [aquí](../../azure-resource-manager/management/azure-subscription-service-limits.md).

**¿Puedo continuar y usar mis scripts automatizados para aprovisionar máquinas virtuales, redes virtuales y cuentas de almacenamiento a través de las API de Resource Manager?**

Toda la automatización y los scripts que ha creado continúan funcionando para las máquinas virtuales existentes, las redes virtuales se crean en el modo de administración de servicios de Azure. Sin embargo, los scripts deben actualizarse para utilizar el nuevo esquema para crear los mismos recursos a través del modo de Resource Manager.

**¿Dónde puedo encontrar ejemplos de plantillas del Administrador de recursos de Azure?**

Puede encontrar un conjunto completo de plantillas de inicio en [Plantillas de inicio rápido de Azure Resource Manager](https://azure.microsoft.com/documentation/templates/).

## <a name="next-steps"></a>Pasos siguientes

* Para ver los comandos para implementar una plantilla, consulte [Implementación de una aplicación con la plantilla del Administrador de recursos de Azure](../templates/deploy-powershell.md).
