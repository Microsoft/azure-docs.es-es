---
title: 'Inicio rápido: Implementación de Azure API for FHIR mediante Azure Portal'
description: En este inicio rápido aprenderá a implementar Azure API for FHIR y a configurar las opciones mediante Azure Portal.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 03/15/2020
ms.author: matjazl
ms.openlocfilehash: 8c0448d31cd89e2ca969b81361b30bac3f9610e9
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851943"
---
# <a name="quickstart-deploy-azure-api-for-fhir-using-azure-portal"></a>Inicio rápido: Implementación de Azure API for FHIR mediante Azure Portal

En este inicio rápido va a aprender a implementar Azure API for FHIR mediante Azure Portal.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-new-resource"></a>Crear recurso

Abra [Azure Portal](https://portal.azure.com) y haga clic en **Crear un recurso**.

![Creación de un recurso](media/quickstart-paas-portal/portal-create-resource.png)

## <a name="search-for-azure-api-for-fhir"></a>Búsqueda de Azure API for FHIR

Puede encontrar Azure API for FHIR escribiendo "FHIR" en el cuadro de búsqueda:

:::image type="content" source="media/quickstart-paas-portal/portal-search-healthcare-apis.png" alt-text="Buscar API de atención sanitaria":::

## <a name="create-azure-api-for-fhir-account"></a>Creación de una cuenta de Azure API for FHIR

Seleccione **Crear** para crear una nueva cuenta de Azure API for FHIR:

:::image type="content" source="media/quickstart-paas-portal/portal-create-healthcare-apis.png" alt-text="Creación de una cuenta de Azure API for FHIR":::

## <a name="enter-account-details"></a>Especificación de los detalles de la cuenta

Seleccione un grupo de recursos existente o cree uno nuevo, elija un nombre para la cuenta y, por último, haga clic en  **Revisar y crear**:

:::image type="content" source="media/quickstart-paas-portal/portal-new-healthcareapi-details.png" alt-text="Detalles de la nueva API de atención sanitaria":::

Confirme la creación y espere la implementación de la API de FHIR.

## <a name="additional-settings-optional"></a>Opciones de configuración adicionales (opcional)

También puede hacer clic en **Siguiente: Opciones de configuración adicionales** para ver la configuración de la autenticación. La configuración predeterminada de Azure API for FHIR es [usar RBAC de Azure para asignar los roles de plano de datos](configure-azure-rbac.md). Cuando se configura en este modo, la "autoridad" del servicio FHIR se establecerá en el inquilino de Azure Active Directory de la suscripción:

:::image type="content" source="media/rbac/confirm-azure-rbac-mode-create.png" alt-text="Configuración de autenticación predeterminada":::

Observe que el cuadro para especificar los identificadores de objeto permitidos está atenuado, ya que se usa RBAC de Azure para configurar las asignaciones de roles en este caso.

Si desea configurar el servicio FHIR para que use un inquilino de Azure Active Directory externo o secundario, puede cambiar la autoridad y especificar los identificadores de objeto para los usuarios y grupos a los que se deba permitir el acceso al servidor. Para obtener más información, consulte la guía de [configuración local de RBAC](configure-local-rbac.md).

## <a name="fetch-fhir-api-capability-statement"></a>Captura de la instrucción de funcionalidad de la API de FHIR

Para validar que se ha aprovisionado la nueva cuenta de la API de FHIR, capture una instrucción de funcionalidad haciendo que el explorador apunte a `https://<ACCOUNT-NAME>.azurehealthcareapis.com/metadata`.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar el grupo de recursos, la instancia de Azure API for FHIR y todos los recursos relacionados. Para hacerlo, seleccione el grupo de recursos que contiene la cuenta de Azure API for FHIR, seleccione **Eliminar grupo de recursos** y confirme el nombre del grupo de recursos que se va a eliminar.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado Azure API for FHIR en su suscripción. Para establecer opciones de configuración adicionales en Azure API for FHIR, continúe con la guía paso a paso de configuración adicional.

>[!div class="nextstepaction"]
>[Configuración adicional en Azure API for FHIR](azure-api-for-fhir-additional-settings.md)
