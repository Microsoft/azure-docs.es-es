---
title: Completar una recuperación ante desastres de VM
description: Este artículo muestra cómo realizar una recuperación ante desastres de VM mediante Azure VMware Solution
ms.topic: how-to
ms.date: 05/04/2020
ms.openlocfilehash: 76a417b9ba00c4c0e6e958e5a04d19aecfe24563
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752266"
---
# <a name="complete-a-disaster-recovery-of-virtual-machines-using-azure-vmware-solution"></a>Realización de una recuperación ante desastres de máquinas virtuales usando la solución de Azure VMware

Este artículo contiene el proceso para realizar una recuperación ante desastres de máquinas virtuales con la solución Hybrid Cloud Extension (HCX) de VMware y cómo usar una nube privada de la solución de Azure VMware como sitio de recuperación o de destino.

VMware HCX permite realizar varias operaciones que proporcionan el control y la granularidad precisos en las directivas de replicación. Las operaciones disponibles son las siguientes:

- Invertir: Después de que se haya producido un desastre. Invertir ayuda a convertir el sitio B en el sitio de origen y el sitio A en el lugar donde reside la VM protegida.

- Pausar: Pausa la directiva de replicación actual asociada a la VM seleccionada.

- Reanudar: Reanuda la directiva de replicación actual asociada a la VM seleccionada.

- Quitar: Quita la directiva de replicación actual asociada a la VM seleccionada.

- Sincronizar ahora: Sincronización fuera de los límites de la VM de origen con la VMl protegida.

En esta guía se cubren los siguientes escenarios de replicación:

- Proteger una VM o un grupo de VM.

- Realizar una recuperación de prueba de una VM o un grupo de VM.

- Proteger una VM o un grupo de VM.

- Protección inversa de una VM o un grupo de VM.

## <a name="protect-virtual-machines"></a>Proteger VM

Iniciar sesión en el **cliente de vSphere** en el sitio de origen y acceda al **complemento de HCX**.

:::image type="content" source="./media/disaster-recovery/hcx-vsphere.png" alt-text="Opción HCX en vSphere" border="true":::

Entre en el área **Recuperación ante desastres** y haga clic en **PROTEGER LAS VM**.

:::image type="content" source="./media/disaster-recovery/protect-virtual-machine.png" alt-text="seleccionar proteger VM" border="true":::

En la ventana que se abre, seleccione el origen y los sitios remotos. En este caso, el sitio remoto debería ser la nube privada de Azure VMware Solution.

:::image type="content" source="./media/disaster-recovery/protect-virtual-machines.png" alt-text="Ventana proteger VM" border="true":::

Si fuese necesario, seleccione las opciones de replicación predeterminadas:

- **Habilitar compresión:** Recomendado para escenarios con un bajo rendimiento.

- **Habilitar inactividad:** Pausa la VM para asegurarse de que se sincroniza una copia coherente con el sitio remoto.

- **Almacenamiento de destino:** Seleccione el almacén de datos remoto para las VM protegidas. En una nube privada de Azure VMware Solution, esta selección debe ser el almacén de información de vSAN.

- **Contenedor del proceso:** El clúster de vSphere remoto o el grupo de recursos.

- **Carpeta de destino:** La carpeta de destino remoto; este valor es opcional y si no se selecciona ninguna carpeta, las VMse colocarán directamente en el clúster seleccionado.

- **RPO:** Este valor es el intervalo de sincronización entre la VM de origen y la VM protegida, y puede oscilar entre 5 minutos y 24 horas.

- **Intervalo de instantánea:** Intervalo entre instantáneas.

- **Número de instantáneas:** Número total de instantáneas en el intervalo de instantánea configurado.

:::image type="content" source="./media/disaster-recovery/protect-virtual-machine-options.png" alt-text="Opciones de protección de VM" border="true":::

Seleccione una o varias VM de la lista y configure las opciones de replicación según sea necesario.

De forma predeterminada, las VM heredarán la directiva de configuración global establecida en las opciones de replicación predeterminadas. Para cada interfaz de red de la VM seleccionada, configure el**Grupo de puertos de red** remoto y seleccione **Finalizar** para iniciar el proceso de protección.

:::image type="content" source="./media/disaster-recovery/network-interface-options.png" alt-text="Opciones de la interfaz de red" border="true":::

Como se puede observar en la siguiente imagen, puede supervisar el proceso de cada una de las VM seleccionadas en el mismo área de recuperación ante desastres.

:::image type="content" source="./media/disaster-recovery/protect-monitor-progress.png" alt-text="Supervisión del progreso de protección" border="true":::

Una vez que la VM se ha protegido, las distintas instantáneas se pueden visualizar en la pestaña **Instantáneas**.

:::image type="content" source="./media/disaster-recovery/list-of-snapshots.png" alt-text="Lista de instantáneas" border="true":::

El triángulo amarillo significa que las instantáneas y la VM no se han probado en una operación de recuperación de prueba.

Hay diferencias clave entre una VM que está apagada y otra que está encendida.
La captura de pantalla anterior muestra el proceso de sincronización para una VM encendida. Se inicia el proceso de sincronización hasta que finaliza la primera instantánea, que es una copia completa de la VM y, a continuación, se completan las siguientes en el intervalo configurado.

En el caso de una VM apagada, se sincronizará una copia y, después, la VM aparecerá como inactiva y la operación de protección se mostrará como completada.

Cuando la VM está encendida, se inicia el proceso de sincronización con el sitio remoto.

## <a name="complete-a-test-recover-of-virtual-machines"></a>Realizar una recuperación de prueba de VM

Inicie sesión en el **cliente de vSphere** en el sitio remoto, que es la nube privada de Azure VMware Solution. En el **complemento de HCX**, en el área recuperación ante desastres, seleccione los puntos suspensivos verticales en cualquier VM para mostrar el menú operaciones. Seleccione **Probar recuperar VM**.

:::image type="content" source="./media/disaster-recovery/test-recover-virtual-machine.png" alt-text="Selección de Probar recuperar VM" border="true":::

En la ventana nueva, seleccione las opciones para realizar la prueba. Seleccione la instantánea que desea usar para probar los distintos estados de la VM.

:::image type="content" source="./media/disaster-recovery/choose-snapshot.png" alt-text="Seleccionar una instantánea y hacer clic en Probar" border="true":::

Después de hacer clic en **Probar**, se iniciará la operación de recuperación.

Una vez finalizada la operación de recuperación de prueba, se puede registrar la nueva VM en la instancia de vCenter de la nube privada de Azure VMware Solution.

:::image type="content" source="./media/disaster-recovery/verify-test-recovery.png" alt-text="Comprobación de la operación de recuperación" border="true":::

Por último, después de realizar las pruebas en la VM o en cualquier aplicación que se ejecute en ella, se hace una limpieza para eliminar la instancia de prueba.

:::image type="content" source="./media/disaster-recovery/cleanup-test-instance.png" alt-text="Limpieza de la instancia de prueba" border="true":::

## <a name="recover-virtual-machines"></a>Recuperación de máquinas virtuales

Inicie sesión en el **cliente de vSphere** en el sitio remoto, que es la nube privada de Azure VMware Solution, y acceda al **complemento HCX**.

En el escenario de recuperación, se usa un grupo de VM para este ejemplo.

Seleccione la VM que se va a recuperar de la lista, abra el menú **ACCIONES** y seleccione **Recuperar VM**.

:::image type="content" source="./media/disaster-recovery/recover-virtual-machines.png" alt-text="Recuperación de VM" border="true":::

Configure las opciones de recuperación de cada instancia y haga clic en **Recuperar** para iniciar la operación de recuperación.

:::image type="content" source="./media/disaster-recovery/recover-virtual-machines-confirm.png" alt-text="Confirmación de recuperación de VM" border="true":::

Una vez completada la operación de recuperación, las nuevas VM aparecerán en el inventario de vCenter Server remoto.

## <a name="complete-a-reverse-replication-on-virtual-machines"></a>Completar una replicación inversa en VM

Inicie sesión en el **cliente de vSphere** de la nube privada de Azure VMware Solution y acceda al **complemento HCX**.
Es necesario apagar las VM originales en el sitio de origen antes de iniciar la replicación inversa. Se produce un error en la operación si las VM no están apagadas.

Seleccione las VM que se van a volver a replicar en el sitio de origen de la lista, abra el menú **ACCIONES** y seleccione **Invertir**. En la ventana emergente, haga clic en **Invertir** para iniciar la replicación.

:::image type="content" source="./media/disaster-recovery/reverse-operation-virtual-machines.png" alt-text="Seleccione acción inversa en proteger operaciones" border="true":::

La replicación se puede supervisar en la sección de detalles de cada VM.

:::image type="content" source="./media/disaster-recovery/review-reverse-operation.png" alt-text="revisar los resultados de la acción inversa" border="true":::

## <a name="disaster-recovery-plan-automation"></a>Automatización de un plan de recuperación ante desastres

Actualmente, VMware HCX no tiene un mecanismo integrado para crear y automatizar un plan de recuperación ante desastres. Esta funcionalidad no existe en HCX. Sin embargo, proporciona un conjunto de API de REST, incluidas las API para la operación de recuperación ante desastres.

Se puede acceder a la especificación de API dentro de HCX Manager en la URL.

Estas API incluyen las siguientes operaciones en la recuperación ante desastres.

- Protección

- Recover

- Recuperación de prueba

- Recuperación planificada

- Reverse

- Consultar

- Limpieza de prueba

- Pausar

- Reanudación

- Eliminación de la protección

- Volver a configurar

A continuación se muestra un ejemplo de una carga de operación de recuperación en JSON.

```json
[

    {

        "replicationId": "string",

        "needPowerOn": true,

        "instanceId": "string",

        "source": {

            "endpointType": "string",

            "endpointId": "string",

            "endpointName": "string",

            "resourceType": "string",

            "resourceId": "string",

            "resourceName": "string"

        },

        "destination": {

            "endpointType": "string",

            "endpointId": "string",

            "endpointName": "string",

            "resourceType": "string",

            "resourceId": "string",

            "resourceName": "string"

        },

        "placement": [

            {

                "containerType": "string",

                "containerId": "string"

            }

        ],

        "resourceId": "string",

        "forcePowerOff": true,

        "isTest": true,

        "forcePowerOffAfterTimeout": true,

        "isPlanned": true

    }

]
```

Con estas API, un cliente puede crear un mecanismo personalizado para automatizar la creación y ejecución de un plan de recuperación ante desastres.
