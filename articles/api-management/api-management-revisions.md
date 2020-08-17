---
title: Revisiones en Azure API Management | Microsoft Docs
description: Información acerca del concepto de revisiones en Azure API Management.
services: api-management
documentationcenter: ''
author: johndowns
ms.service: api-management
ms.topic: article
ms.date: 06/12/2020
ms.author: jodowns
ms.custom: fasttrack-new
ms.openlocfilehash: b099a6ea706482e25b2c37a87cf0a24f2fe475bb
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86528241"
---
# <a name="revisions-in-azure-api-management"></a>Revisiones de Azure API Management

Las revisiones le permiten realizar cambios en las API de una forma controlada y segura. Cuando desee realizar cambios, cree una revisión. Posteriormente, puede editar y probar la API sin molestar a los consumidores de la misma. Cuando esté listo, convierta la revisión en la versión actual. Al mismo tiempo, puede publicar una entrada en el registro de cambios con el fin de mantener al día a los consumidores de la API con los cambios que se han realizado. El registro de cambios se publica en el portal para desarrolladores.

> [!NOTE]
> El portal para desarrolladores no está disponible en el nivel Consumo.

Con las revisiones puede:

- Realizar cambios de forma segura en las definiciones y directivas de la API sin alterar la API de producción.
- Probar los cambios antes de publicarlos.
- Documentar los cambios que realice, con el fin de que los desarrolladores puedan conocer las novedades.
- Revertir cualquier operación si surgen problemas.

[Para empezar a usar las revisiones, siga nuestro tutorial.](./api-management-get-started-revise-api.md)

## <a name="accessing-specific-revisions"></a>Acceso a revisiones concretas

Para acceder a todas las revisiones de una API se puede usar una dirección URL con un formato especial. Para acceder a una revisión concreta de esa API, anexe `;rev={revisionNumber}` al final de la dirección URL de la API, pero antes de la cadena de consulta. Por ejemplo, esta dirección URL se puede usar para acceder a la revisión 3 de la API de `customers`:

`https://apis.contoso.com/customers;rev=3?customerId=123`

De forma predeterminada, todas las revisiones tienen la misma configuración de seguridad que la revisión actual. Puede cambiar deliberadamente las directivas de una revisión específica si desea que se aplique una seguridad diferente a cada revisión. Por ejemplo, puede agregar una [directiva de lista de direcciones IP permitidas](./api-management-access-restriction-policies.md#RestrictCallerIPs) para evitar que los autores de llamadas externos accedan a revisiones que aún estén en desarrollo.

Las revisiones se pueden desconectar, lo que hace que los autores de llamadas no puedan acceder a ellas, aunque intenten hacerlo a través de su dirección URL. Para marcar una revisión como sin conexión, use Azure Portal. Si usa PowerShell, puede utilizar el cmdlet `Set-AzApiManagementApiRevision` y establecer el argumento `Path` en `$null`.

> [!NOTE]
> Se recomienda que las revisiones estén sin conexión cuando no se usen para las pruebas.

## <a name="current-revision"></a>Revisión actual

Solo se puede establecer una única revisión como revisión *actual*. Esta revisión será la que se use para todas las solicitudes de API que no especifiquen un número de revisión explícito en la dirección URL. Para realizar la reversión a una revisión anterior, establezca esa revisión como la actual.

Se puede establecer una revisión como actual mediante Azure Portal. Si usa PowerShell, puede utilizar el cmdlet `New-AzApiManagementApiRelease`.

## <a name="revision-descriptions"></a>Descripciones de revisiones

Al crear una revisión, puede establecer una descripción para hacer un seguimiento de la misma. Las descripciones no aparecen para los usuarios de la API.

Si establece una revisión como actual, también puede especificar opcionalmente una nota pública en el registro de cambios. El registro de cambios se incluye en el portal para desarrolladores y todos los usuarios de la API pueden verlo. Para modificar la nota del registro de cambios, se usa el cmdlet de PowerShell `Update-AzApiManagementApiRelease`.

## <a name="versions-and-revisions"></a>Versiones y revisiones

Las versiones y las revisiones son características distintas. Cada versión puede tener varias revisiones, como si fuera una API sin versión. Se pueden usar revisiones sin utilizar versiones y viceversa. Normalmente, las versiones se usan para diferenciar aquellas versiones de API que tienen cambios importantes, mientras que las revisiones se pueden usar para cambios menores y no importantes en una API.

Si observa que su revisión tiene cambios importantes o desea convertir formalmente una revisión en una versión beta o de prueba, puede crear una versión a partir de una revisión. En Azure Portal, haga clic en "Crear versión a partir de esta revisión" en el menú contextual de la revisión en la pestaña Revisiones.
