---
title: Habilitación automática de la copia de seguridad al crear máquinas virtuales mediante Azure Policy
description: Un artículo en el que se describe cómo usar Azure Policy para habilitar automáticamente la copia de seguridad de todas las máquinas virtuales creadas en un ámbito determinado
ms.topic: conceptual
ms.date: 11/08/2019
ms.openlocfilehash: de4923000bc842203535e03727fd532c67a8f517
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826079"
---
# <a name="auto-enable-backup-on-vm-creation-using-azure-policy"></a>Habilitación automática de la copia de seguridad al crear máquinas virtuales mediante Azure Policy

Una de las principales responsabilidades de un administrador de copia de seguridad o de cumplimiento en una organización es asegurarse de que se realiza una copia de seguridad de todas las máquinas críticas para la empresa con el período de retención adecuado.

Actualmente, Azure Backup proporciona una directiva integrada (mediante Azure Policy) que se puede asignar a **todas las máquinas virtuales de Azure de una ubicación especificada dentro de una suscripción o un grupo de recursos**. Cuando esta directiva se asigna a un ámbito determinado, todas las máquinas virtuales nuevas creadas en ese ámbito se configuran automáticamente para que se cree su copia de seguridad en un **almacén existente en la misma ubicación y suscripción**. El usuario puede especificar el almacén y la directiva de retención a los que se deben asociar las máquinas virtuales de las que se ha creado una copia de seguridad.

## <a name="supported-scenarios"></a>Escenarios admitidos

* La directiva integrada solo se admite actualmente para máquinas virtuales de Azure. Los usuarios deben tener cuidado para asegurarse de que la directiva de retención especificada durante la asignación sea una directiva de retención de máquinas virtuales. Consulte [este documento](./backup-azure-policy-supported-skus.md) para ver todas las SKU de máquina virtual que admite esta directiva.

* La directiva se puede asignar a una única ubicación y suscripción a la vez. Para habilitar la copia de seguridad de máquinas virtuales entre ubicaciones y suscripciones, es necesario crear varias instancias de la asignación de directiva, una para cada combinación de ubicación y suscripción.

* El almacén especificado y las máquinas virtuales configuradas para la copia de seguridad pueden pertenecer a distintos grupos de recursos.

* Actualmente no se admite el ámbito del grupo de administración.

* La directiva integrada no está disponible actualmente en nubes nacionales.

## <a name="using-the-built-in-policy"></a>Uso de la directiva integrada

Para asignar la directiva al ámbito necesario, siga los pasos a continuación:

1. Inicie sesión en Azure Portal y vaya al panel **Directiva**.
1. Seleccione **Definiciones** en el menú de la izquierda para obtener una lista de todas las directivas integradas en los recursos de Azure.
1. Filtre la lista por **Categoría = Copia de seguridad**. Verá la lista filtrada, donde se mostrará una única directiva denominada "Configurar la copia de seguridad para máquinas virtuales de una ubicación en un almacén central existente de la misma ubicación".
![Panel de directivas](./media/backup-azure-auto-enable-backup/policy-dashboard.png)
1. Seleccione el nombre de la directiva. Se le redirigirá a la definición detallada de dicha directiva.
![Panel de definición de directiva](./media/backup-azure-auto-enable-backup/policy-definition-blade.png)
1. Seleccione el botón **Asignar** en la parte superior del panel. Esta acción le redirigirá al panel **Asignar directiva**.
1. En **Conceptos básicos**, seleccione los tres puntos situados junto al campo **Ámbito**. Se abrirá el panel de contexto adecuado, donde podrá seleccionar la suscripción en la que se va a aplicar la directiva. También puede seleccionar un grupo de recursos, de modo que la directiva se aplique solo a las máquinas virtuales de un grupo de recursos determinado.
![Conceptos básicos de la asignación de directiva](./media/backup-azure-auto-enable-backup/policy-assignment-basics.png)
1. En la pestaña **Parámetros**, elija una ubicación de la lista desplegable y seleccione el almacén y la directiva de copia de seguridad a la que se deben asociar las máquinas virtuales del ámbito.
![Parámetros de la asignación de directiva](./media/backup-azure-auto-enable-backup/policy-assignment-parameters.png)
1. Asegúrese de que **Efecto** esté establecido en deployIfNotExists.
1. Vaya a **Revisar + crear** y seleccione **Crear**.

> [!NOTE]
>
> También se puede usar Azure Policy en máquinas virtuales existentes, mediante la [corrección](../governance/policy/how-to/remediate-resources.md).

> [!NOTE]
>
> Se recomienda que esta directiva no se asigne a más de 200 máquinas virtuales a la vez. Si la directiva está asignada a más de 200 máquinas virtuales, puede ocurrir que la copia de seguridad se desencadene unas horas después de lo especificado en la programación.

## <a name="next-steps"></a>Pasos siguientes

[Más información acerca de Azure Policy](../governance/policy/overview.md)
