---
title: Traslado de máquinas virtuales de Azure entre regiones mediante Azure Resource Mover
description: Aprenda a trasladar máquinas virtuales de Azure a otra región mediante Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: tutorial
ms.date: 09/09/2020
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 49b7a3700bf497ad868b7c4ab1f0802564b61bf3
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89652496"
---
# <a name="tutorial-move-azure-vms-across-regions"></a>Tutorial: Traslado de máquinas virtuales de Azure entre regiones

En este artículo, aprenderá a trasladar las máquinas virtuales de Azure, junto con los recursos de red y almacenamiento relacionados, a otra región de Azure mediante [Azure Resource Mover](overview.md).

> [!NOTE]
> Azure Resource Mover se encuentra actualmente en versión preliminar pública.


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprobar los requisitos previos y otras exigencias.
> * Seleccionar los recursos que quiere trasladar.
> * Resolver las dependencias de recursos.
> * Preparar y trasladar el grupo de recursos de origen. 
> * Preparar y trasladar los demás recursos.
> * Elegir si desea descartar o confirmar el traslado. 
> * Opcionalmente, quitar los recursos de la región después del traslado.

> [!NOTE]
> Los tutoriales muestran la manera más rápida de probar un escenario y utilizar las opciones predeterminadas. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar. Luego, inicie sesión en el [Portal de Azure](https://portal.azure.com).

## <a name="prerequisites"></a>Requisitos previos

-  Compruebe que tiene acceso como *propietario* a la suscripción que contiene los recursos que desea trasladar.
    - La primera vez que agregue un recurso de un par de origen y destino específicos a una suscripción de Azure, Azure Resource Mover creará una [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types), anteriormente llamada identidad de servicio administrado (MSI), en la que confiará la suscripción.
    - Para crear la identidad y asignarle el rol requerido (Colaborador o Administrador de acceso de usuario en la suscripción de origen), la cuenta que utilice para agregar recursos necesita permisos de *propietario* para la suscripción. [Obtenga más información](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) sobre los roles de Azure.
- La suscripción necesita tener cuota suficiente para crear los recursos que va a trasladar a la región de destino. Si no dispone de cuota suficiente, [solicite límites adicionales](/azure/azure-resource-manager/management/azure-subscription-service-limits).
- Compruebe los precios y los cargos asociados con la región de destino a la que va a trasladar las máquinas virtuales. El uso de la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) le resultará útil.
    

## <a name="check-vm-requirements"></a>Comprobación de los requisitos de máquina virtual

1. Asegúrese de que las máquinas virtuales que desea trasladar están admitidas.

    - [Compruebe](support-matrix-move-region-azure-vm.md#windows-vm-support) las máquinas virtuales Windows admitidas.
    - [Compruebe](support-matrix-move-region-azure-vm.md#linux-vm-support) las máquinas virtuales Linux y las versiones de kernel admitidas.
    - Compruebe la configuración admitida de [proceso](support-matrix-move-region-azure-vm.md#supported-vm-compute-settings), [almacenamiento](support-matrix-move-region-azure-vm.md#supported-vm-storage-settings) y [redes](support-matrix-move-region-azure-vm.md#supported-vm-networking-settings).
2. Compruebe que las máquinas virtuales que desea trasladar están encendidas.
3. Asegúrese de que las máquinas virtuales dispongan de los certificados raíz de confianza más recientes, así como de una lista actualizada de revocación de certificados (CRL). Para ello, siga estos pasos:
    - En las máquinas virtuales Windows, instale las actualizaciones de Windows más recientes.
    - En las máquinas virtuales Linux, siga las indicaciones del distribuidor para que las máquinas dispongan de los certificados y la lista de revocación de certificados más recientes. 
4. Permita la conectividad saliente desde las máquinas virtuales:
    - Si utiliza un servidor proxy de firewall basado en direcciones URL para controlar la conectividad de salida, debe facilitar el acceso a estas [URL](support-matrix-move-region-azure-vm.md#url-access).
    - Si utiliza reglas de grupo de seguridad de red para determinar la conectividad saliente, cree estas [reglas de etiquetas de servicio](support-matrix-move-region-azure-vm.md#nsg-rules).

## <a name="select-resources"></a>Selección de recursos 

Seleccione los recursos que desea trasladar.

- Se mostrarán todos los tipos de recursos admitidos de los grupos de recursos de la región de origen seleccionada.
- Los recursos se trasladarán a una región de destino en la misma suscripción que la región de origen. Si desea cambiar la suscripción, puede hacerlo después de trasladar los recursos.

1. En Azure Portal, busque *resource mover*. A continuación, en **Servicios**, seleccione **Azure Resource Mover**.

    ![Resultados de la búsqueda de "resource mover" en Azure Portal](./media/tutorial-move-region-virtual-machines/search.png)

2. En **Información general**, haga clic en **Introducción**.

    ![Botón que permite agregar recursos para trasladarlos a otra región](./media/tutorial-move-region-virtual-machines/get-started.png)

3. En **Mover recursos** > **Origen + destino**, seleccione la suscripción y la región de origen.
4. En **Destino**, seleccione la región a la que desea trasladar las máquinas virtuales. A continuación, haga clic en **Siguiente**.
5. En **Metadata region** (Región de metadatos), seleccione la ubicación en la que desea almacenar los metadatos de los recursos que va a trasladar. Se creará un grupo de recursos específicamente con esta finalidad. A continuación, haga clic en **Siguiente**.

    ![Página para seleccionar las regiones de origen y de destino](./media/tutorial-move-region-virtual-machines/source-target.png)

6. En **Recursos que se van a mover** , haga clic en **Seleccionar recursos**.
7. En **Seleccionar recursos**, seleccione la máquina virtual. Solo es posible agregar [recursos que se puedan mover](#check-vm-requirements). A continuación, haga clic en **Hecho**.

    ![Página para seleccionar las máquinas virtuales que se van a trasladar](./media/tutorial-move-region-virtual-machines/select-vm.png)

8.  En **Recursos que se van a mover**, haga clic en **Siguiente**.
9. En **Revisar + agregar**, compruebe la configuración de origen y de destino. Es importante que comprenda que los metadatos de los recursos que se vayan a trasladar se almacenarán en un grupo de recursos creado con esta finalidad en la región de metadatos.

    ![Página para revisar la configuración y realizar el traslado de los recursos](./media/tutorial-move-region-virtual-machines/review.png)
10. Haga clic en **Continuar** para empezar a agregar recursos.
11. Una vez que haya terminado de agregarlos correctamente, haga clic en **Adding resources for move** (Agregar recursos para mover) en el icono de notificación.
12. Después de hacer clic en la notificación, revise los recursos de la página **Entre regiones**.

> [!NOTE]
> - Los recursos agregados tienen el estado *Prepare pending* (Preparación pendiente).
> - Si desea quitar un recurso de una colección de traslado, el método para hacerlo dependerá del punto en el que se encuentre en dicha operación. [Más información](remove-move-resources.md).

## <a name="resolve-dependencies"></a>Resolución de dependencias

1. Si los recursos muestran el mensaje *Validar dependencias* en la columna **Problemas**, haga clic en el botón **Validar dependencias**. Se iniciará el proceso de validación.
2. Si se encuentran dependencias, haga clic en **Agregar dependencias**. 
3. En **Agregar dependencias**, seleccione los recursos dependientes > **Agregar dependencias**. Compruebe el progreso en el área de notificaciones.

    ![Adición de dependencias](./media/tutorial-move-region-virtual-machines/add-dependencies.png)

4. Agregue más dependencias si es necesario y vuelva a validarlas. 
    ![Página para agregar otras dependencias](./media/tutorial-move-region-virtual-machines/add-additional-dependencies.png)

4. En la página **Entre regiones**, compruebe que los recursos tienen ahora el estado *Prepare pending* (Preparación pendiente) y que no presentan problemas.

    ![Página que muestra los recursos que tienen el estado de preparación pendiente](./media/tutorial-move-region-virtual-machines/prepare-pending.png)

> [!NOTE]
> Si desea editar la configuración de destino antes de trasladar los recursos, seleccione el vínculo en la columna **Destination configuration** (Configuración de destino) para el recurso y edite la configuración. Si edita la configuración de la máquina virtual de destino, el tamaño de esta máquina virtual no puede ser inferior al tamaño de la máquina virtual de origen.  

## <a name="move-the-source-resource-group"></a>Traslado del grupo de recursos de origen 

Antes de poder preparar y trasladar las máquinas virtuales, el grupo de recursos de la máquina virtual debe estar presente en la región de destino. 

### <a name="prepare-to-move-the-source-resource-group"></a>Preparación del grupo de recursos de origen para el traslado

Durante el proceso de preparación, Resource Mover genera plantillas de Azure Resource Manager mediante la configuración del grupo de recursos. Los recursos que pertenecen al grupo no se ven afectados.

Lleve a cabo los siguientes preparativos:

1. En **Entre regiones**, seleccione el grupo de recursos de origen > **Preparar**.
2. En **Preparar recursos**, haga clic en **Preparar**.

    ![Preparación del grupo de recursos](./media/tutorial-move-region-virtual-machines/prepare-resource-group.png)

> [!NOTE]
> Una vez preparado el grupo de recursos, tendrá el estado *Initiate move pending* (Iniciar movimiento pendiente). 

 
### <a name="move-the-source-resource-group"></a>Traslado del grupo de recursos de origen

Empiece a trasladar el grupo de recursos como se indica a continuación:

1. En **Entre regiones**, seleccione el grupo de recursos > **Iniciar movimiento**.
2. En **Mover recursos**, haga clic en **Iniciar movimiento**. El grupo de recursos pasará al estado *Initiate move in progress* (Iniciar movimiento en curso).
3. Una vez iniciado el traslado, se creará el grupo de recursos de destino, basado en la plantilla de Resource Manager generada. El grupo de recursos de origen pasa al estado *Commit move Pending* (Confirmar movimiento pendiente).

    ![Hacer clic en el botón Iniciar movimiento](./media/tutorial-move-region-virtual-machines/commit-move-pending.png)

Para confirmar y finalizar el proceso de traslado:

1. En **Entre regiones**, seleccione el grupo de recursos > **Confirmar movimiento**.
2. En **Mover recursos**, haga clic en **Confirmar**.

> [!NOTE]
> Una vez confirmado el traslado, el grupo de recursos de origen tendrá el estado *Delete source pending* (Eliminar origen pendiente).

## <a name="prepare-resources-to-move"></a>Preparación de los recursos que se van a trasladar

Ahora que el grupo de recursos de origen se ha trasladado, puede preparar el traslado de los demás recursos.

1. En **Entre regiones**, seleccione los recursos que desea preparar. 

    ![Página para seleccionar la preparación de otros recursos](./media/tutorial-move-region-virtual-machines/prepare-other.png)

2. Seleccione **Preparar**. 

> [!NOTE]
> - Durante el proceso de preparación, el agente de movilidad de Azure Site Recovery se instala en las máquinas virtuales para replicarlas.
> - Los datos de las máquinas virtuales se replican periódicamente en la región de destino. Esto no afecta a la máquina virtual de origen.
> - Resource Mover genera plantillas de Resource Manager para los demás recursos de origen.
> - Una vez preparados los recursos, pasarán al estado *Initiate move pending* (Iniciar movimiento pendiente).

![Página que muestra los recursos que tienen el estado de iniciar movimiento pendiente](./media/tutorial-move-region-virtual-machines/initiate-move-pending.png)


## <a name="initiate-the-move"></a>Inicio de la migración

Con los recursos ya preparados, puede iniciar el traslado. 

1. En **Entre regiones**, seleccione los recursos que tengan el estado *Initiate move pending* (Iniciar movimiento pendiente). A continuación, haga clic en **Iniciar movimiento**.
2. En **Mover recursos**, haga clic en **Iniciar movimiento**.

    ![Hacer clic en el botón Iniciar movimiento](./media/tutorial-move-region-virtual-machines/initiate-move.png)

3. Haga un seguimiento del progreso del traslado en la barra de notificaciones.

> [!NOTE]
> - En el caso de las máquinas virtuales, se crearán réplicas en la región de destino. La máquina virtual de origen se apagará y se producirá cierto tiempo de inactividad (normalmente es cuestión de minutos).
> - Resource Mover recreará otros recursos mediante las plantillas de Resource Manager preparadas. Normalmente, no se produce tiempo de inactividad.
> - Después de trasladar los recursos, tendrán el estado *Commit move pending* (Confirmar movimiento pendiente).

![Página que muestra los recursos que tienen el estado de "eliminar origen pendiente"](./media/tutorial-move-region-virtual-machines/delete-source-pending.png)


## <a name="discard-or-commit"></a>¿Descartar o confirmar?

Tras la operación inicial de traslado, puede decidir si desea confirmarla o descartarla. 

- **Descartar**: puede descartar un traslado cuando esté haciendo pruebas y no quiera trasladar verdaderamente el recurso de origen. Si el traslado se descarta, el recurso volverá a tener el estado *Initiate move pending* (Iniciar movimiento pendiente).
- **Confirmación**: la confirmación completa la operación de traslado a la región de destino. Tras la confirmación, un recurso de origen tendrá el estado *Delete source pending* (Eliminar origen pendiente) y el usuario puede decidir si desea eliminarlo.


## <a name="discard-the-move"></a>Descartar la operación de traslado 

Para descartar la operación de traslado, siga estos pasos:

1. En **Entre regiones**, seleccione los recursos que tengan el estado *Commit move pending* (Confirmar movimiento pendiente) y haga clic en **Descartar movimiento**.
2. En **Descartar movimiento**, haga clic en **Descartar**.
3. Haga un seguimiento del progreso del traslado en la barra de notificaciones.


> [!NOTE]
> Después de haber descartado recursos, las máquinas virtuales tendrán el estado *Initiate move pending* (Iniciar movimiento pendiente).

## <a name="commit-the-move"></a>Confirmación de la operación de traslado

Si desea completar la operación de traslado, confírmela. 

1. En **Entre regiones**, seleccione los recursos que tengan el estado *Commit move pending* (Confirmar movimiento pendiente) y haga clic en **Confirmar movimiento**.
2. En **Confirmar recursos**, haga clic en **Confirmar**.

    ![Página para confirmar los recursos y completar la operación de traslado](./media/tutorial-move-region-virtual-machines/commit-resources.png)

3. Haga un seguimiento del progreso de la operación de confirmación en la barra de notificaciones.

> [!NOTE]
> - Tras confirmar el traslado, las máquinas virtuales dejarán de replicarse. La confirmación no afecta a la máquina virtual de origen.
> - La confirmación tampoco afecta a los recursos de redes de origen.
> - Una vez confirmado el traslado, los recursos tendrán el estado *Delete source pending* (Eliminar origen pendiente).

![Página que muestra los recursos que tienen el estado de "eliminar origen pendiente"](./media/tutorial-move-region-virtual-machines/delete-source-pending.png)


## <a name="configure-settings-after-the-move"></a>Configuración de parámetros tras el traslado

El servicio Mobility no se desinstala automáticamente de las máquinas virtuales. Puede desinstalarlo manualmente, o bien mantenerlo si tiene previsto trasladar el servidor de nuevo.

## <a name="delete-source-resources-after-commit"></a>Eliminación de los recursos de origen después de la confirmación

Después del traslado, puede eliminar los recursos de la región de origen si lo desea. 

1. En **Entre regiones**, haga clic en el nombre de los recursos de origen que desee eliminar.
2. En la página Propiedades de cada recurso, seleccione **Eliminar**.

## <a name="delete-additional-resources-created-for-move"></a>Eliminación de recursos adicionales creados para el traslado

Después del traslado, puede eliminar manualmente la colección de traslado, junto con los recursos de Site Recovery que se crearon.

- La colección de traslado está oculta de forma predeterminada. Para verla, debe activar los recursos ocultos.
- El almacenamiento en caché tiene un bloqueo que debe eliminarse antes de que se pueda eliminar la colección.

Realice la eliminación de la siguiente manera: 
1. Identifique los recursos pertenecientes al grupo ```RegionMoveRG-<sourceregion>-<target-region>```.
2. Compruebe que todas las máquinas virtuales y los demás recursos de la región de origen se hayan trasladado o eliminado. De este modo, se garantiza que no haya recursos pendientes que los utilicen.
2. Eliminación de los recursos:

    - El nombre de la colección de traslado es ```movecollection-<sourceregion>-<target-region>```.
    - El nombre de la cuenta de almacenamiento en caché es ```resmovecache<guid>```.
    - El nombre del almacén es ```ResourceMove-<sourceregion>-<target-region>-GUID```.
## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Trasladar máquinas virtuales de Azure a otra región de Azure.
> * Trasladar los recursos asociados con máquinas virtuales a otra región.

Ahora veremos cómo trasladar grupos elásticos y bases de datos de Azure SQL a otra región.

> [!div class="nextstepaction"]
> [Traslado de recursos de Azure SQL](./tutorial-move-region-sql.md)
