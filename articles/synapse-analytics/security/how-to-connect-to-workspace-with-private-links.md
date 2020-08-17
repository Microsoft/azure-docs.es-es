---
title: Conexión a un área de trabajo de Synapse mediante vínculos privados
description: En este artículo se explica cómo conectarse a su área de trabajo de Azure Synapse mediante vínculos privados.
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 04/15/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: 17636bf993df5105093ca690e36db22493a2472e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005980"
---
# <a name="connect-to-your-azure-synapse-workspace-using-private-links-preview"></a>Conexión a su área de trabajo de Azure Synapse mediante vínculos privados (versión preliminar)

En este artículo se explica cómo crear un punto de conexión privado a su área de trabajo de Azure Synapse. Para obtener más información, consulte [vínculos privados y puntos de conexión privados](https://docs.microsoft.com/azure/private-link/).

## <a name="step-1-register-network-resource-provider"></a>Paso 1: Registro del proveedor de recursos de red

Si todavía no lo ha hecho, registre el proveedor de recursos de red. Al registrar un proveedor de recursos se configura la suscripción para que funcione con este. Elija *Microsoft.Network* en la lista de proveedores de recursos al [efectuar el registro](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types). Si el proveedor de recursos de red ya está registrado, continúe con el paso 2.


## <a name="step-2-open-your-azure-synapse-workspace-in-azure-portal"></a>Paso 2: Apertura del área de trabajo de Azure Synapse en Azure Portal

Seleccione **Conexiones de punto de conexión privado** en **Seguridad** y, a continuación, seleccione **+ Punto de conexión privado**.
![Apertura del área de trabajo de Azure Synapse en Azure Portal](./media/how-to-connect-to-workspace-with-private-links/private-endpoint-1.png)

## <a name="step-3-select-your-subscription-and-region-details"></a>Paso 3: Selección de los detalles de su suscripción y área de trabajo.

En la pestaña **Básico** de la ventana **Crear un punto de conexión privado**, elija su **Suscripción** y **Grupo de recursos**. Asigne un **Nombre** al punto de conexión privado que desea crear. Seleccione la **Región** en la que desea crear el punto de conexión privado.

Los puntos de conexión privados se crean en una subred. La suscripción, el grupo de recursos y la región seleccionados filtran las subredes de puntos de conexión privados. Seleccione **Siguiente: Recurso >** cuando haya terminado.
![Selección de los detalles de suscripción y región](./media/how-to-connect-to-workspace-with-private-links/private-endpoint-2.png)

## <a name="step-4-select-your-azure-synapse-workspace-details"></a>Paso 4: Selección de los detalles del área de trabajo de Azure Synapse

Seleccione **Conectarse a un recurso de Azure en mi directorio** en la pestaña **Recurso**. Seleccione la **Suscripción** que contiene el área de trabajo de Azure Synapse. El **Tipo de recurso** para crear puntos de conexión privados en un área de trabajo de Azure Synapse es *Microsoft.Synapse/workspaces*.

Seleccione el área de trabajo de Azure Synapse como **Recurso**. Cada área de trabajo de Azure Synapse tiene tres opciones de **Subrecurso de destino** donde puede crear un punto de conexión privado: Sql, SqlOnDemand y Dev.

Seleccione **Siguiente: Configuración >** para avanzar a la siguiente parte de la configuración.
![Selección de los detalles de suscripción y región](./media/how-to-connect-to-workspace-with-private-links/private-endpoint-3.png)

En la pestaña **Configuración**, seleccione la **Red virtual** y la **Subred** en la que se debe crear el punto de conexión privado. También debe crear un registro DNS que se asigne al punto de conexión privado.

Seleccione **Sí** para **Integrar con la zona DNS privada** para integrar el punto de conexión privado en una zona DNS privada. Si no tiene una zona DNS privada asociada a la instancia de Microsoft Azure Virtual Network, se crea una nueva zona DNS privada. Seleccione **Revisar y crear** cuando haya terminado.

![Selección de los detalles de suscripción y región](./media/how-to-connect-to-workspace-with-private-links/private-endpoint-4.png)

Una vez que se complete la implementación, abra el área de trabajo de Azure Synapse en Azure Portal y seleccione **Conexiones de punto de conexión privado**. Se muestran el nuevo punto de conexión privado y el nombre de la conexión del punto de conexión privado asociada al punto de conexión privado.

![Selección de los detalles de suscripción y región](./media/how-to-connect-to-workspace-with-private-links/private-endpoint-5.png)

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [Red virtual del área de trabajo administrada](./synapse-workspace-managed-vnet.md)

Más información sobre los [puntos de conexión privados administrados](./synapse-workspace-managed-private-endpoints.md)

[Creación de puntos de conexión privados administrados a los orígenes de datos](./how-to-create-managed-private-endpoints.md)
