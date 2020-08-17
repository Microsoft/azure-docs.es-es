---
title: Agregar etiquetas a un laboratorio de Azure DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo agregar una etiqueta a un laboratorio de Azure DevTest Labs
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 30f0f2ed4b8cc98985326db652371b8ceb16bfb4
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287588"
---
# <a name="add-tags-to-a-lab-in-azure-devtest-labs"></a>Agregar etiquetas a un laboratorio de Azure DevTest Labs

Puede crear etiquetas personalizadas y aplicarlas a los recursos de DevTest Labs para clasificar los recursos de manera lógica. Así, más adelante podrá ver todos los recursos de la suscripción que tengan esa etiqueta de forma totalmente fácil y rápida. Las etiquetas son útiles si necesita organizar recursos para facturación o administración.

Estos son los recursos en los que se pueden usar etiquetas:

* Máquinas virtuales de Compute
* NIC
* Direcciones IP
* Equilibradores de carga
* Cuentas de almacenamiento
* Discos administrados

Puede aplicar etiquetas al [crear un laboratorio](devtest-lab-create-lab.md) y, posteriormente, administrarlas a través de la hoja Etiquetas en Configuración y directivas.

Cada etiqueta se compone de un par **nombre**/**valor**. Por ejemplo, se podría crear una etiqueta con el nombre *centroDeCoste* que tenga el valor *34543*. Una etiqueta como esta nos serviría más adelante para identificar los recursos de laboratorio que son facturables en este área concreta de la organización. Puede elegir los nombres y valores que sean más convenientes en cuanto a cómo quiere organizar la suscripción.

## <a name="steps-to-manage-tags-in-an-existing-lab"></a>Pasos para administrar etiquetas en un laboratorio existente

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Si es necesario, seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista. Puede que su laboratorio ya aparezca en el panel, en **Todos los recursos**.
1. En la lista de laboratorios, seleccione aquel en el que quiera agregar o administrar etiquetas.
1. En el área **Introducción** del laboratorio, seleccione **Configuración y directivas**.

    ![Botón Configuración y directivas](./media/devtest-lab-add-tag/devtestlab-config-and-policies.png)

1. A la izquierda, en **ADMINISTRAR**, seleccione **Etiquetas**.
1. Para crear una etiqueta para este laboratorio, escriba un par **nombre**/**valor** y haga clic en **Guardar**. También puede seleccionar una etiqueta existente de la lista para ver o administrar los recursos asociados a dicha etiqueta.

    ![Administrar etiquetas](./media/devtest-lab-add-tag/devtestlab-manage-tags.png)

> [!NOTE]
> Etiquetas creadas en el flujo de nivel de laboratorio a través de todos los recursos facturables que el laboratorio pone en marcha en su suscripción. Por ejemplo, las etiquetas de nivel de laboratorio fluyen a las máquinas virtuales de proceso subyacentes de las máquinas virtuales de laboratorio. Puede utilizar etiquetas en el contexto de administración de costos. Las etiquetas de nivel de laboratorio aparecen en el filtro de etiquetas de Cost Management.

## <a name="understanding-limitations-to-tags"></a>Descripción de las limitaciones de las etiquetas

Se aplican las siguientes limitaciones a las etiquetas:

* Cada recurso o grupo de recursos puede tener un máximo de 15 pares de nombre/valor de etiqueta. Esta limitación solo se aplica a las etiquetas que se aplican directamente al recurso o grupo de recursos. Un grupo de recursos puede contener muchos recursos con 15 pares de clave/valor de etiqueta cada uno.
* El nombre de etiqueta está limitado a 512 caracteres y el valor de la etiqueta, a 256. En las cuentas de almacenamiento, el nombre de etiqueta se limita a 128 caracteres y el valor de la etiqueta, a 256.
* Los recursos de un grupo de recursos no heredan las etiquetas aplicadas a este.

En [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md) encontrará más detalles sobre el uso de etiquetas en Azure, incluido cómo administrarlas con PowerShell o la CLI de Azure.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Pasos siguientes
* Puede aplicar restricciones y convenciones a través de su suscripción con directivas personalizadas. La directiva que define podría requerir que todos los recursos tengan un valor para una etiqueta determinada. Para más información, vea [Configuración de directivas y programaciones](devtest-lab-set-lab-policy.md).
* Explore la [galería de plantillas de inicio rápido de Azure Resource Manager de DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
