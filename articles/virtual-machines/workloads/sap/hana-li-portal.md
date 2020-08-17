---
title: Control de HANA en Azure (instancias grandes) mediante Azure Portal | Microsoft Docs
description: Describe cómo puede identificar e interactuar con HANA en Azure (instancias grandes) mediante el portal.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/15/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d81a8b3a1596e8a447f7a2434e52df8c89b416b7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085272"
---
# <a name="azure-hana-large-instances-control-through-azure-portal"></a>Control de instancias grandes de HANA en Azure mediante Azure Portal
En este documento se describe la manera en que se presenta [HANA (instancias grandes)](./hana-overview-architecture.md) en [Azure Portal](https://portal.azure.com) y qué actividades se pueden realizar en este portal con las unidades de HANA (instancias grandes) que se implementan. La visibilidad de HANA (instancias grandes) en Azure Portal la proporciona un proveedor de recursos de Azure específico que actualmente está en versión preliminar pública.

## <a name="register-hana-large-instance-resource-provider"></a>Registro del proveedor de recursos de HANA (instancias grandes)
Normalmente, la suscripción de Azure que estaba utilizando para las implementaciones de HANA (instancias grandes) está registrada para el proveedor de recursos correspondiente. Sin embargo,si no ve las unidades de HANA (instancias grandes) implementadas, debe registrar el proveedor de recursos en la suscripción de Azure. Hay dos maneras de registrar el proveedor de recursos de HANA (instancias grandes).

### <a name="register-through-cli-interface"></a>Registro mediante la interfaz de la CLI
Debe haber iniciado sesión en la suscripción de Azure que se ha empleado para la implementación de HANA (instancias grandes) a través de la interfaz de la CLI de Azure. Puede volver a registrar el proveedor de recursos para HANA (instancias grandes) con este comando:
    
```azurecli
az provider register --namespace Microsoft.HanaOnAzure
```

Para más información, consulte el artículo [Tipos y proveedores de recursos de Azure](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-cli).


### <a name="register-through-azure-portal"></a>Registro mediante Azure Portal
Puede volver a registrar el proveedor de recursos para HANA (instancias grandes) mediante Azure Portal. Es necesario que aparezca en Azure Portal la suscripción que se usó para implementar las unidades de HANA (instancias grandes) y que haga doble clic en ella. Una vez que esté en la página de información general de la suscripción, seleccione "Proveedores de recursos" como se indica a continuación y escriba "HANA" en la ventana de búsqueda. 

![Registro del proveedor de recursos de HANA (instancias grandes) mediante Azure Portal](./media/hana-li-portal/portal-register-hli-rp.png)

En la captura de pantalla siguiente, el proveedor de recursos ya se ha registrado. En caso de que el proveedor de recursos aún no esté registrado, pulse "volver a registrar" o "registrar".

Para más información, consulte el artículo [Tipos y proveedores de recursos de Azure](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-powershell).


## <a name="display-of-hana-large-instance-units-in-the-azure-portal"></a>Visualización de unidades de HANA (instancias grandes) en Azure Portal
Al enviar una solicitud de implementación de HANA (instancias grandes), deberá especificar también la suscripción de Azure que va a conectar a HANA (instancias grandes). Se recomienda usar la misma suscripción que está utilizando para implementar el nivel de aplicación de SAP que funciona en las unidades de HANA (instancias grandes).
Cuando se implementa por primera vez HANA (instancias grandes), se crea un nuevo [grupo de recursos de Azure](../../../azure-resource-manager/management/manage-resources-portal.md) en la suscripción de Azure que envió en la solicitud de implementación.  El nuevo grupo de recursos mostrará todas las unidades de HANA (instancias grandes) que tenga implementadas en esa suscripción específica.

Para buscar el nuevo grupo de recursos de Azure, puede ver la lista de los grupos de recursos de la suscripción. Para ello, vaya al panel de navegación izquierdo de Azure Portal.

![Panel de navegación en Azure Portal](./media/hana-li-portal/portal-resource-group.png)

Una vez que vea la lista de recursos, puede que sea necesario filtrar por la suscripción que utilizó para implementar HANA (instancias grandes).

![Filtrado de grupos de recursos en Azure Portal](./media/hana-li-portal/portal-filtering-subscription.png)

Después de filtrar en la suscripción correcta, puede que todavía tenga una larga lista de grupos de recursos. Busque uno con un postfijo **-Txxx** donde "xxx" son tres dígitos, como **-T050**. 

Cuando encuentre el grupo de recursos, vea los detalles de este. El listado que recibe puede ser parecido a este:

![Lista de unidades de HANA (instancias grandes) en Azure Portal](./media/hana-li-portal/portal-hli-units-list.png)

Todas las unidades que aparecen representan una única unidad de HANA (instancias grandes) que se ha implementado en la suscripción. En este caso, puede ver ocho unidades de HANA (instancias grandes) que se implementaron en la suscripción.

Si ha implementado varios inquilinos de HANA (instancias grandes) en la misma suscripción de Azure, se encontrará con varios grupos de recursos de Azure. 


## <a name="look-at-attributes-of-single-hli-unit"></a>Búsqueda de atributos de una única unidad de HANA (instancias grandes)
En la lista de unidades de HANA (instancias grandes), puede hacer clic en una sola unidad para obtener los detalles de ella. 

En la pantalla de información general, después de hacer clic en "Mostrar más" aparecerá una presentación de la unidad con el siguiente aspecto:

![Información general de una unidad de HANA (instancias grandes)](./media/hana-li-portal/portal-show-overview.png)

Al examinar los diferentes atributos que aparecen podemos ver que apenas se diferencian de los atributos de las máquinas virtuales de Azure. En el encabezado de la izquierda aparece el grupo de recursos, la región de Azure, el nombre de la suscripción y el identificador, así como algunas etiquetas que ha agregado. De forma predeterminada, las unidades de HANA (instancias grandes) no tienen etiqueta asignada. En el lado derecho del encabezado aparece el nombre de la unidad como asignado si se ha realizado la implementación. Aparece también el sistema operativo, así como la dirección IP. Al igual que con las máquinas virtuales, también aparece el tipo de unidad de HANA (instancias grandes) junto con el número de subprocesos de CPU y la memoria. Aquí se muestra también más información sobre las diferentes unidades de HANA (instancias grandes):

- [SKU disponibles para HLI](./hana-available-skus.md)
- [Arquitectura de almacenamiento de SAP HANA (instancias grandes)](./hana-storage-architecture.md) 

Los datos adicionales del lado inferior derecho son la revisión del sello de HANA (instancias grandes). Los valores posibles son:

- Revisión 3
- Revisión 4

La revisión 4 es la arquitectura más reciente publicada de HANA (instancias grandes) con mejoras importantes en la latencia de red entre las máquinas virtuales de Azure y las unidades de HANA (instancias grandes) implementadas en los sellos o las filas de la revisión 4.
Otra información muy importante se encuentra en la esquina inferior derecha de la información general con el nombre del grupo de ubicación de proximidad de Azure que se crea automáticamente para cada unidad de HANA (instancias grandes) implementada. Se debe hacer referencia a este grupo de selección de ubicación de proximidad al implementar las máquinas virtuales de Azure que hospedan el nivel de aplicación de SAP. Mediante el uso del [grupo de ubicación de proximidad de Azure](../../linux/co-location.md) asociado a la unidad de HANA (instancias grandes), se asegura de que las máquinas virtuales de Azure se implementan cerca de la unidad de HANA (instancias grandes). La forma en que se pueden usar los grupos de selección de ubicación de proximidad para ubicar el nivel de aplicación de SAP en el mismo centro de datos de Azure que las unidades de HANA (instancias grandes) hospedadas de la revisión 4 se describe en [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md).

Un campo adicional en la columna derecha del encabezado informa sobre el estado de energía de la unidad de HANA (instancias grandes).

> [!NOTE]
> El estado de energía describe si la unidad de hardware está encendida o apagada. No informa sobre si el sistema operativo está en funcionamiento. Si reinicia una unidad de HANA (instancias grandes), verá que transcurre un pequeño intervalo en el que el estado de la unidad cambia de **Iniciándose** a **Iniciada**. El hecho de tener el estado **Iniciada** significa que el sistema operativo se está iniciando o que ya se ha iniciado por completo. Como resultado, después de un reinicio de la unidad, es posible que no pueda iniciar sesión inmediatamente en la unidad tan pronto como el estado cambie a **Iniciada**.
> 

Si presiona "Ver más", se muestra información adicional. Se muestra información adicional de la revisión del sello de HANA (instancias grandes), en la que se implementó la unidad. Consulte el artículo [¿Qué es SAP HANA en Azure (instancias grandes)?](./hana-overview-architecture.md) para las distintas revisiones de los sellos de HANA (instancias grandes).

## <a name="check-activities-of-a-single-hana-large-instance-unit"></a>Actividades de comprobación de una unidad de HANA (instancias grandes) 
Además de ofrecer información general de las unidades de HANA (instancias grandes), también puede comprobar las actividades de una unidad determinada. Un registro de actividades puede tener el siguiente aspecto:

![Panel de navegación en Azure Portal](./media/hana-li-portal/portal-activity-list.png)

Una de las actividades principales que se registran son los reinicios de una unidad. Entre los datos que aparecen, se incluye el estado de la actividad, la marca de hora a la que se desencadenó la actividad, el identificador de la suscripción en la que se desencadenó y el usuario de Azure que la desencadenó. 

Otra actividad que se registra son los cambios de la unidad en los metadatos de Azure. Después de que se inició el reinicio, puede ver la actividad **Write HANAInstances**. Este tipo de actividad no realiza cambios en la unidad de HANA (instancias grandes) en sí, pero documenta cambios en los metadatos de la unidad en Azure. En el caso que se muestra, se ha agregado y eliminado una etiqueta (consulte la sección siguiente).

## <a name="add-and-delete-an-azure-tag-to-a-hana-large-instance-unit"></a>Agregar y eliminar una etiqueta de Azure a una unidad de HANA (instancias grandes)
Otra posibilidad consiste en Agregar una [etiqueta](../../../azure-resource-manager/management/tag-resources.md) a una unidad de HANA (instancias grandes). La manera en que se asignan las etiquetas no difiere de la manera en que se asignan a las máquinas virtuales. Al igual que con las máquinas virtuales, las etiquetas existen en los metadatos de Azure y, en el caso de HANA (instancias grandes), las restricciones de las etiquetas también son las mismas.

La eliminación de las etiquetas funciona del mismo modo que con las máquinas virtuales. Ambas actividades, la aplicación y eliminación de una etiqueta, aparecerán en el registro de actividades de la unidad concreta de HANA (instancias grandes).

## <a name="check-properties-of-a-hana-large-instance-unit"></a>Comprobación de las propiedades de una unidad de HANA (instancias grandes)
La sección **Propiedades** incluye información importante que se obtiene cuando se le entregan las instancias. En esta sección puede obtener toda la información que necesitará en caso de que se produzca una incidencia de soporte técnico o al definir la configuración de las instantáneas de almacenamiento. Por lo tanto, esta sección es una recopilación de datos sobre la instancia, la conectividad de esta con Azure y el back-end de almacenamiento. La parte superior de la sección presenta el siguiente aspecto:


![Parte superior de las propiedades de HANA (instancias grandes) en Azure Portal](./media/hana-li-portal/portal-properties-top.png)

Los primeros elementos de datos ya aparecían en la pantalla de información general. Pero una parte importante de los datos es el identificador del circuito ExpressRoute, que obtuvo cuando se entregaron las primeras unidades implementadas. Puede que se le pidan esos datos en determinadas incidencias de soporte técnico. Una entrada de datos importante aparece en la parte inferior de la captura de pantalla. Este dato es la dirección IP del almacenamiento NFS principal que aísla el almacenamiento en el **inquilino** en la pila de HANA (instancias grandes). Esta dirección IP también es necesaria cuando se edita el [archivo de configuración de las copias de seguridad de las instantáneas de almacenamiento](./hana-backup-restore.md#set-up-storage-snapshots). 

A medida que se desplaza hacia abajo en el panel de propiedades, aparecen datos adicionales, como un identificador de recurso único para la unidad de HANA (instancias grandes) o el identificador de suscripción que se asignó a la implementación.

## <a name="restart-a-hana-large-instance-unit-through-azure-portal"></a>Reinicio de una unidad de HANA (instancias grandes) mediante Azure Portal
En cuanto al reinicio del sistema operativo Linux, hay varias situaciones en las que es posible que el sistema operativo no pueda finalizar un reinicio correctamente. Para forzar un reinicio, es necesario que abra una solicitud de servicio para que Microsoft Operations realice un reinicio de la energía de la unidad de HANA (instancias grandes). La funcionalidad de reinicio de energía de una unidad de HANA (instancias grandes) está integrada ya en Azure Portal. Cuando esté en la sección de información general de la unidad de HANA (instancias grandes), podrá ver el botón para reiniciar en la parte superior de la sección de datos.

![Paso 1 para reiniciar en Azure Portal](./media/hana-li-portal/portal-restart-first-step.png)

Cuando pulse el botón de reinicio, se le pedirá que confirme si realmente desea reiniciar la unidad. Cuando confirme con el botón "Sí", se reiniciará la unidad.

> [!NOTE]
> En el proceso de reinicio, verá que transcurre un pequeño intervalo en el que el estado de la unidad cambia de **Iniciándose** a **Iniciada**. El hecho de tener el estado **Iniciada** significa que el sistema operativo se está iniciando o que ya se ha iniciado por completo. Como resultado, después de un reinicio de la unidad, es posible que no pueda iniciar sesión inmediatamente en la unidad tan pronto como el estado cambie a **Iniciada**.

> [!IMPORTANT]
> Según la cantidad de memoria en la unidad de HANA (instancias grandes), un reinicio del hardware y del sistema operativo pueden tardar hasta una hora.


## <a name="open-a-support-request-for-hana-large-instances"></a>Abrir una solicitud de soporte técnico para HANA (instancias grandes)
Aparte de en la pantalla de todas las unidades de HANA (instancias grandes) en Azure Portal, también puede crear solicitudes de soporte técnico para una sola unidad específicamente. Para ello, siga el vínculo **Nueva solicitud de soporte técnico**. 

![Paso 1 para iniciar solicitud de servicio en Azure Portal](./media/hana-li-portal/portal-initiate-support-request.png)

Para que aparezcan todos los servicios de las unidades de SAP HANA (instancias grandes) en la pantalla siguiente, es posible que deba seleccionar "Todos los servicios" como se indica a continuación.

![Seleccionar Todos los servicios en Azure Portal](./media/hana-li-portal/portal-create-service-request.png)

En la lista de servicios, puede encontrar el servicio **SAP HANA (instancias grandes)** . Cuando selecciona un servicio, puede seleccionar también los tipos de problemas concretos como se indica a continuación:


![Seleccionar tipo de problema en Azure Portal](./media/hana-li-portal/portal-select-problem-class.png)

En cada uno de los diferentes tipos de problemas, se le ofrece una selección de subtipos de problemas que debe seleccionar para especificar aún más su problema. Después de seleccionar el subtipo, ahora puede asignar un nombre al asunto. Cuando haya terminado con el proceso de selección, puede continuar con el siguiente paso de la creación. En la sección **Soluciones**, se le dirige a documentación sobre HANA (instancias grandes) que podría facilitar información para encontrar una solución al problema. Si no puede encontrar una solución para su problema en la documentación sugerida, vaya al paso siguiente. En el siguiente paso, se le preguntará si el problema está relacionado con máquinas virtuales o con unidades de HANA (instancias grandes). Esta información le ayudará a dirigir la solicitud de soporte técnico a los especialistas adecuados. 

![Detalles de una incidencia de soporte técnico en Azure Portal](./media/hana-li-portal/portal-support-request-details.png)

A medida que responda a las preguntas y proporcione detalles adicionales, podrá pasar al paso siguiente para revisar la solicitud de soporte técnico y enviarla.

## <a name="next-steps"></a>Pasos siguientes

- [Procedimiento para supervisar SAP HANA en Azure (instancias grandes)](./troubleshooting-monitoring.md)
- [Supervisión y solución de problemas en el lado de HANA](./hana-monitor-troubleshoot.md)
