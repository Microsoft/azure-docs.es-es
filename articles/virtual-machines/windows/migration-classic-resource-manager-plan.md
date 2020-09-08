---
title: Planificación de la migración del modelo clásico a Azure Resource Manager
description: Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager
author: tanmaygore
manager: vashan
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: dc8279f8227c437a09a61fad2b84c5b061257eb5
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510222"
---
# <a name="planning-for-migration-of-iaas-resources-from-classic-to-azure-resource-manager-in-windows"></a>Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager en Windows

> [!IMPORTANT]
> En la actualidad, aproximadamente el 90 % de las VM de IaaS usan [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). A partir del 28 de febrero de 2020, las máquinas virtuales clásicas han quedado en desuso y se van a retirar por completo el 1 de marzo de 2023. [Obtenga más información]( https://aka.ms/classicvmretirement) sobre esta caída en desuso y [cómo se va a ver afectado](../classic-vm-deprecation.md#how-does-this-affect-me).

Aunque Azure Resource Manager ofrece muchas características increíbles, es fundamental planear la trayectoria de migración para asegurarse de que el proceso se desarrolla con facilidad. Dedicar tiempo a la planificación garantizará que no se planteen problemas al ejecutar las actividades de migración.

La trayectoria de migración consta de cuatro fases generales:<br>

![Fases de migración](../media/virtual-machines-windows-migration-classic-resource-manager/plan-labtest-migrate-beyond.png)

## <a name="plan"></a>Plan

### <a name="technical-considerations-and-tradeoffs"></a>Consideraciones técnicas y compromisos

En función de la envergadura de los requisitos técnicos, de las zonas geográficas y de las prácticas operativas, debería tener en cuenta lo siguiente:

1. ¿Por qué le interesa contar con Azure Resource Manager en su organización?  ¿Por qué motivos empresariales desea realizar una migración?
2. ¿Cuáles son las razones técnicas para disponer de Azure Resource Manager?  ¿De qué servicios adicionales de Azure desearía beneficiarse, si procede?
3. ¿Qué aplicación o conjuntos de máquinas virtuales se incluyen en la migración?
4. ¿Qué escenarios son compatibles con la API de migración?  Revise las [configuraciones y características no admitidas](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#unsupported-features-and-configurations).
5. ¿Los equipos operativos ahora admiten aplicaciones y máquinas virtuales tanto en el modelo clásico como en Azure Resource Manager?
6. ¿Cómo Azure Resource Manager cambia los procesos de creación de informes, la supervisión, la administración y la implementación de máquinas virtuales, en caso de que sea posible?  ¿Es necesario actualizar los scripts de implementación?
7. ¿Cuál es el plan de comunicaciones para avisar a las partes interesadas (usuarios finales, propietarios de aplicaciones y propietarios de infraestructuras)?
8. Según la complejidad del entorno, ¿debe haber un período de mantenimiento durante el cual la aplicación no está disponible para los usuarios finales y los propietarios de aplicaciones?  En su caso, ¿durante cuánto tiempo?
9. ¿Cuál es el plan de aprendizaje para asegurarse de que las partes interesadas conocen y dominan Azure Resource Manager?
10. ¿Cuál es el plan de administración de programas o de proyectos para la migración?
11. ¿Cuáles son las escalas de tiempo para la migración de Azure Resource Manager y para otros mapas de rutas tecnológicos relacionados?  ¿Están perfectamente alineados?

### <a name="patterns-of-success"></a>Patrones de éxito

Los clientes de éxito disponen de planes detallados en los que se abordan, documentan y administran las preguntas anteriores.  Asegúrese de que los planes de migración se comunican de forma amplia a los patrocinadores y a las partes interesadas.  Adquiera conocimientos sobre las opciones de migración; se recomienda leer este documento de migración a continuación.

* [Información general sobre la migración compatible con la plataforma de recursos de IaaS desde el modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](../migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager)
* [Migración de recursos de IaaS de la implementación clásica a Resource Manager con Azure PowerShell](migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure](../linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Herramientas de la comunidad para ayudar con la migración de recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de los errores más comunes en la migración](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de las preguntas más frecuentes acerca de cómo migrar recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](../migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

### <a name="pitfalls-to-avoid"></a>Errores que hay que evitar

- Errores de planificación.  Los pasos tecnológicos de esta migración se han comprobado y el resultado es predecible.
- En todos los escenarios se tendrá en cuenta la suposición de que la plataforma admite la API de migración. Leer las [configuraciones y características no admitidas](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#unsupported-features-and-configurations) para saber qué escenarios se admiten.
- Ausencia de planificación de interrupciones potenciales de la aplicación para los usuarios finales.  Planee suficiente búfer para advertir a los usuarios finales del tiempo durante el cual la aplicación posiblemente no estará disponible.


## <a name="lab-test"></a>Análisis de laboratorio

**Replicación de un entorno y realización de una migración de prueba**
  > [!NOTE]
  > La replicación exacta de un entorno existente se ejecuta mediante una herramienta en la que ha contribuido la comunidad que no es compatible oficialmente con el Soporte técnico de Microsoft. Por lo tanto, se trata de un paso **opcional**, pero es la mejor manera de encontrar los problemas sin tocar los entornos de producción. Si no tiene la opción de usar una herramienta en la que ha contribuido la comunidad, lea la recomendación Simulacro de validación, preparación y anulación a continuación.
  >

  Realizar un análisis de laboratorio de un escenario exacto (proceso, red y almacenamiento) es la mejor forma de asegurar una migración sin problemas. Esto le ayudará a garantizar:

- Que dispone para el análisis de un laboratorio totalmente independiente o de un entorno existente que no sea de producción. Se recomienda un laboratorio completamente independiente que se pueda migrar varias veces y que se pueda modificar de forma destructiva.  A continuación se indican los scripts para recopilar e hidratar metadatos de suscripciones reales.
- Es una buena idea crear el laboratorio en una suscripción independiente. El motivo es que el laboratorio se desactivará varias veces, por lo que disponer de una suscripción aislada e independiente reducirá la posibilidad de eliminar por error cualquier elemento real.

  Esto puede realizarse con la herramienta AsmMetadataParser. [Aquí puede encontrar más información sobre esta herramienta](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/AsmToArmMigrationApiToolset).

### <a name="patterns-of-success"></a>Patrones de éxito

A continuación se indican problemas detectados en muchas de las migraciones más grandes. No se trata de una lista exhaustiva, por lo que debe remitirse a [Configuraciones y características no admitidas](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#unsupported-features-and-configurations) para más detalles.  Puede encontrarse o no con estos problemas técnicos, pero, de plantearse, podrá tener una experiencia más fluida si los soluciona antes de intentar realizar la migración.

- **Realice un simulacro de validación, preparación y anulación**: se trata quizás del paso más importante para asegurarse de que la migración del modelo clásico a Azure Resource Manager se realiza correctamente. La API de migración se compone de tres pasos principales: validación, preparación y confirmación. Con la validación se leerá el estado del entorno clásico y se devolverá un resultado de todos los problemas. Sin embargo, como algunos problemas pueden existir en la pila de Azure Resource Manager, la validación puede no detectarlo todo. En el paso siguiente del proceso de migración, la preparación ayudará a exponer los problemas detectados. Con la preparación, todos los metadatos se moverán del modelo clásico a Azure Resource Manager, pero no se confirmará la migración ni se eliminará ni modificará nada del modelo clásico. El simulacro conlleva preparar la migración y después anular (**no confirmar**) la preparación de la migración. El objetivo del simulacro de validación, preparación y anulación es que todos los metadatos se encuentren en la pila de Azure Resource Manager, para después examinarlos (*mediante programación o en el portal*) y verificar que todo se ha migrado correctamente, pero también con el propósito de abordar los problemas técnicos.  También le permitirá hacerse una idea de la duración de la migración, a fin de que pueda planear el tiempo de inactividad.  Un simulacro de validación, preparación y anulación no causa ningún tiempo de inactividad al usuario, por lo que no produce ninguna interrupción del uso de la aplicación.
  - Será necesario solucionar los puntos siguientes antes del simulacro, aunque, en caso de omitirlos, la prueba de simulacro eliminará con seguridad estos pasos de preparación. Durante la migración empresarial, se ha observado que el simulacro es una forma segura e inestimable de garantizar la preparación de la migración.
  - Cuando la preparación se encuentra en ejecución, el plano de control (operaciones de administración de Azure) se bloquearán en toda la red virtual, por lo que no se puede realizar ninguna modificación en los metadatos de máquinas virtuales durante el proceso de validación, preparación y anulación.  No obstante, en caso contrario, ninguna función de la aplicación (Escritorio remoto, uso de máquina virtual, etc.) se verá afectada.  Los usuarios de las máquinas virtuales no sabrán que el simulacro está en ejecución.

- **VPN y circuitos ExpressRoute**. Actualmente no se pueden migrar las puertas de enlace de ExpressRoute con vínculos de autorización sin tiempos de inactividad. Para consultar una solución alternativa, vea [Migración de circuitos ExpressRoute y las redes virtuales asociadas del modelo de implementación clásica a Resource Manager](../../expressroute/expressroute-migration-classic-resource-manager.md).

- **Extensiones de máquina virtual**: las extensiones de máquina virtual posiblemente son uno de los obstáculos más importantes para migrar las máquinas virtuales en ejecución. La corrección de extensiones de máquina virtual puede tardar de 1 a 2 días, por lo que es necesario realizar la planificación teniendo esto en cuenta.  Se precisa de un agente de Azure operativo para informar del estado de las extensiones de las máquinas virtuales en ejecución. Si se notifica un estado incorrecto de una máquina virtual en ejecución, la migración se detendrá. El agente no necesita estar operativo para habilitar la migración, pero, si existen extensiones en la máquina virtual, se necesitará un agente operativo Y una conectividad saliente a Internet con DNS para que la migración progrese.
  - Si se pierde la conectividad con un servidor DNS durante la migración, hay que eliminar todas las extensiones de máquina virtual , excepto BGInfo versión 1. \*, de todas las máquinas virtuales antes de preparar la migración y, después, volver a agregarlas a la máquina virtual después de la migración de Azure Resource Manager.  **Esto solo se aplica a las máquinas virtuales en ejecución.**  Si las máquinas virtuales se detienen cuando están desasignadas, no es necesario eliminar las extensiones de máquina virtual.

  > [!NOTE]
  > Muchas extensiones, como la supervisión de Azure Diagnósticos y Security Center, se reinstalarán automáticamente después de la migración, por lo que su eliminación no plantea ningún problema.

  - Además, asegúrese de que los Grupos de seguridad de red no restringen el acceso saliente a Internet. Esto puede suceder con algunas configuraciones de Grupos de seguridad de red. El acceso saliente a Internet, y DNS, resulta necesario para migrar las extensiones de máquina virtual a Azure Resource Manager.
  - Existen dos versiones de la extensión BGInfo, que se denominan versiones 1 y 2.  

      - Si la máquina virtual usa la extensión de la versión 1 de BGInfo, dicha extensión se puede dejar tal cual. La API de migración omite esta extensión. La extensión BGInfo puede agregarse después de la migración.
      - Si la máquina virtual usa la extensión de la versión 2 de BGInfo basada en JSON, significa que la máquina virtual se creó mediante Azure Portal. La API de migración incluye esta extensión en la migración a Azure Resource Manager, siempre que el agente funcione y tenga acceso saliente a Internet (y DNS).

  - **Opción de corrección 1**. Si sabe que las máquinas virtuales no tendrán acceso saliente a Internet, un servicio DSN activo y agentes de Azure operativos en las máquinas virtuales, desinstale las extensiones de máquina virtual como parte de la migración antes de la fase de preparación y luego reinstálelas después de la migración.
  - **Opción de corrección 2**. Si las extensiones de máquina virtual plantean un gran obstáculo, otra opción consiste en apagar o desasignar todas las máquinas virtuales antes de la migración. Migre las máquinas virtuales desasignadas y luego reinícielas en Azure Resource Manager. Aquí la ventaja es que las extensiones de máquina virtual se migrarán. El inconveniente es que todas las IP virtuales públicas se perderán, algo que no tiene sentido, ya que las máquinas virtuales se apagarán con un impacto mucho mayor en las aplicaciones operativas.

    > [!NOTE]
    > Si se configura una directiva de Azure Security Center con respecto a las máquinas virtuales en ejecución que se van a migrar, es necesario detener dicha directiva de seguridad antes de quitar las extensiones, ya que, de lo contrario, la extensión de supervisión de seguridad se reinstalará automáticamente en la máquina virtual después de quitarla.

- **Conjuntos de disponibilidad**: para migrar una red virtual (vNet) a Azure Resource Manager, todas las máquinas virtuales contenidas en la implementación clásica, es decir, el servicio en la nube, deben encontrarse en un conjunto de disponibilidad, o bien ninguna máquina virtual debe pertenecer a ningún conjunto de disponibilidad. Azure Resource Manager no admite que haya más de un conjunto de disponibilidad en el servicio en la nube y, de ser así, la migración se detendrá.  Además, no puede haber algunas máquinas virtuales en un conjunto de disponibilidad, y algunas máquinas virtuales no están en un conjunto de disponibilidad. Para solucionar esta cuestión, será necesario corregir o reorganizar el servicio en la nube.  Tenga esto en cuenta para realizar la planificación, ya que esta operación puede llevar bastante tiempo.

- **Implementaciones de roles web y roles de trabajo**: Cloud Services con roles web y roles de trabajo no se puede migrar a Azure Resource Manager. Para migrar el contenido de los roles web y de trabajo, deberá migrar el propio código al App Services PaaS más reciente (este tema queda fuera del ámbito de este documento). Si quiere dejar los roles web o de trabajo tal cual, pero migrar las máquinas virtuales clásicas al modelo de implementación de Resource Manager, primero se deben quitar los roles web o de trabajo de la red virtual antes de que se pueda iniciar la migración.  Una solución típica consiste en trasladar las instancias de rol web o de trabajo a una red virtual clásica independiente que también esté vinculada a un circuito ExpressRoute. En el caso de reimplementación anterior, cree una red virtual clásica nueva, mueva los roles web y los roles de trabajo a esa nueva red virtual o reimpleméntelos ahí y luego elimine las implementaciones de la red virtual que se va a migrar. No se requiere ningún cambio de código. La nueva funcionalidad [Emparejamiento de red virtual](../../virtual-network/virtual-network-peering-overview.md) se puede utilizar para emparejar la red virtual clásica que contiene los roles web y los roles de trabajo con otras redes virtuales de la misma región de Azure, como la red virtual que se va a migrar (**después de que termine la migración de la red virtual, ya que las máquinas virtuales emparejadas no se pueden migrar**), a fin de ofrecer las mismas funcionalidades sin perder rendimiento y sin penalizaciones de latencia o ancho de banda. Gracias a la adición del [Emparejamiento de red virtual](../../virtual-network/virtual-network-peering-overview.md), las implementaciones de roles web y roles de trabajo ahora pueden mitigarse con facilidad y no bloquean la migración a Azure Resource Manager.

- **Cuotas de Azure Resource Manager**: las regiones de Azure tienen cuota y límites independientes para el modelo clásico y Azure Resource Manager. Aunque en un escenario de migración no se utiliza ningún hardware nuevo *(se están cambiando las máquinas virtuales existentes del modelo clásico a Azure Resource Manager)* , todavía es necesario que haya cuotas de Azure Resource Manager con capacidad suficiente para poder iniciar la migración. A continuación, se especifican los límites principales con los que se han detectado problemas.  Abra un vale de soporte sobre cuotas para aumentar los límites.

    > [!NOTE]
    > Estos límites hay que aumentarlos en la misma región a la que se va a migrar el entorno actual.
    >

  - Interfaces de red
  - Equilibradores de carga
  - Direcciones IP públicas
  - Direcciones IP públicas estáticas
  - Núcleos
  - Grupos de seguridad de red
  - Tablas de ruta

    Puede comprobar las cuotas actuales de Azure Resource Manager mediante los comandos siguientes con la última versión de Azure PowerShell.



    **Proceso** *(núcleos, conjuntos de disponibilidad)*

    ```powershell
    Get-AzVMUsage -Location <azure-region>
    ```

    **Red** *(redes virtuales, direcciones IP públicas estáticas, direcciones IP públicas, grupos de seguridad de red, interfaces de red, equilibradores de carga y tablas de rutas)*

    ```powershell
    Get-AzUsage /subscriptions/<subscription-id>/providers/Microsoft.Network/locations/<azure-region> -ApiVersion 2016-03-30 | Format-Table
    ```

    **Almacenamiento** *(cuenta de almacenamiento)*

    ```powershell
    Get-AzStorageUsage
    ```

- **Límites de la API de Azure Resource Manager**: si tiene un entorno lo suficientemente grande (por ejemplo, más de 400 máquinas virtuales en una red virtual), podría alcanzar los límites predeterminados de escritura de la API (actualmente `1200 writes/hour`) en Azure Resource Manager. Antes de iniciar la migración, debe presentar un vale de soporte para aumentar este límite en la suscripción.


- **Estado de máquina virtual de tiempo de espera de aprovisionamiento agotado**: si alguna máquina virtual presenta el estado `provisioning timed out`, es necesario resolver este problema antes de la migración. La única manera de hacerlo es que el tiempo de inactividad, mediante el desaprovisionamiento y el reaprovisionamiento de la máquina virtual, es decir, eliminarla, conservar el disco y volver a crearla.

- **Estado de máquina virtual RoleStateUnknown**: si la migración se detiene debido a un mensaje de error `role state unknown`, inspeccione la máquina virtual en el portal y asegúrese de que está en ejecución. Este error suele desaparecer por sí solo (no se precisa de ninguna corrección) transcurridos unos minutos, suele ser de carácter transitorio y suele detectarse durante las operaciones `start`, `stop` y `restart` de una máquina virtual. **Práctica recomendada:** vuelva a intentar la migración después de unos minutos.

- **El clúster de Fabric no existe** : en algunos casos, determinadas máquinas virtuales no se pueden migrar por distintas razones poco comunes. Uno de estos casos conocidos se da si la máquina virtual se ha creado recientemente (durante la última semana más o menos) y ha iniciado un clúster de Azure que aún no está equipado para cargas de trabajo de Azure Resource Manager.  Obtendrá un error que indica `fabric cluster does not exist` y, en este caso, la máquina virtual no se puede migrar. Normalmente, basta con esperar un par de día para que este problema concreto se resuelva, ya que el clúster no tardará en estar habilitado para Azure Resource Manager. Sin embargo, una solución inmediata consiste en `stop-deallocate` la máquina virtual, continuar después con la migración e iniciar la copia de seguridad de la máquina virtual en Azure Resource Manager después de la migración.

### <a name="pitfalls-to-avoid"></a>Errores que hay que evitar

- No use métodos abreviados y omita las migraciones del simulacro de validación, preparación y anulación.
- La mayoría de los problemas potenciales, si no todos, se detectarán durante los pasos del simulacro de validación, preparación y anulación.

## <a name="migration"></a>Migración

### <a name="technical-considerations-and-tradeoffs"></a>Consideraciones técnicas y compromisos

Ahora está listo porque ha trabajado con los problemas conocidos en su entorno.

Para las migraciones reales, conviene tener en cuenta lo siguiente:

1. Planifique y programe la red virtual (unidad más pequeña de migración) con mayor prioridad.  Céntrese primero en las redes virtuales sencillas y continúe después con las más complicadas.
2. La mayoría de los clientes tendrán entornos de producción y de no producción.  Programe la producción en última instancia.
3. **(OPCIONAL)**  Programe un tiempo de inactividad por mantenimiento con una gran cantidad de búfer en caso de que surjan problemas inesperados.
4. Comuníquese con los equipos de soporte y póngase de acuerdo con ellos en caso de que surjan problemas.

### <a name="patterns-of-success"></a>Patrones de éxito

La orientación técnica de la sección _Análisis de laboratorio_ se debe tener en cuenta y abordar antes de realizar la migración real.  Con las pruebas oportunas, la migración realmente no es ningún evento.  Para entornos de producción, puede resultar útil contar con soporte técnico adicional, como un asociado de Microsoft de confianza o Microsoft Premier Services.

### <a name="pitfalls-to-avoid"></a>Errores que hay que evitar

El hecho de no realizar las pruebas completas puede generar problemas y retrasos en la migración.  

## <a name="beyond-migration"></a>Después de la migración

### <a name="technical-considerations-and-tradeoffs"></a>Consideraciones técnicas y compromisos

Ahora que ya está en Azure Resource Manager, maximice la plataforma.  Lea la [información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md) para obtener información sobre otras ventajas.

Puntos que se deben tener en cuenta:

- Agrupe la migración con otras actividades.  La mayoría de los clientes optan por una ventana de mantenimiento de la aplicación.  En su caso, podría usar este tiempo de inactividad para habilitar otras funcionalidades de Azure Resource Manager, como el cifrado y la migración a Managed Disks.
- Revise los motivos técnicos y empresariales para migrar a Azure Resource Manager; habilite los servicios adicionales disponibles solo en Azure Resource Manager que sean aplicables a su entorno.
- Modernice el entorno con servicios PaaS.

### <a name="patterns-of-success"></a>Patrones de éxito

Determine qué servicios desea habilitar en Azure Resource Manager.  Muchos clientes encuentran atractivos los siguientes para sus entornos de Azure:

- [Control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/overview.md).
- [Plantillas de Azure Resource Manager para una implementación más sencilla y controlada](../../azure-resource-manager/templates/overview.md).
- [Etiquetas](../../azure-resource-manager/management/tag-resources.md).
- [Control de actividad](../../azure-resource-manager/management/view-activity-logs.md)
- [Directivas de Azure](../../governance/policy/overview.md)

### <a name="pitfalls-to-avoid"></a>Errores que hay que evitar

Recuerde por qué ha iniciado esta trayectoria de migración del modelo clásico a Azure Resource Manager.  ¿Cuáles eran los objetivos empresariales originales? ¿Ha conseguido el objetivo empresarial?


## <a name="next-steps"></a>Pasos siguientes

* [Información general sobre la migración compatible con la plataforma de recursos de IaaS desde el modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](../migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Migración de recursos de IaaS de la implementación clásica a Resource Manager con Azure PowerShell](migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure](../linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Migración de VPN Gateway (clásico) a Resource Manager](../../vpn-gateway/vpn-gateway-classic-resource-manager-migration.md)
* [Migración de circuitos ExpressRoute y las redes virtuales asociadas del modelo de implementación clásica a Resource Manager](../../expressroute/expressroute-migration-classic-resource-manager.md)
* [Herramientas de la comunidad para ayudar con la migración de recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de los errores más comunes en la migración](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de las preguntas más frecuentes acerca de cómo migrar recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](../migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
