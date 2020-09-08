---
title: Roles y permisos para Azure Data Factory
description: Describe los roles y permisos necesarios para crear factorías de datos y para trabajar con recursos secundarios.
ms.date: 11/5/2018
ms.topic: conceptual
ms.service: data-factory
services: data-factory
documentationcenter: ''
ms.workload: data-services
author: djpmsft
ms.author: daperlov
manager: anandsub
ms.openlocfilehash: 40fa6bce67aa6c5643e4a153da610dce65907b56
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88036358"
---
# <a name="roles-and-permissions-for-azure-data-factory"></a>Roles y permisos para Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]


Este artículo describe los roles necesarios para crear y administrar recursos de Azure Data Factory y los permisos concedidos por esos roles.

## <a name="roles-and-requirements"></a>Roles y requisitos

Para crear instancias de Data Factory, la cuenta de usuario que use para iniciar sesión en Azure debe ser miembro de los roles *colaborador* o *propietario*, o ser *administrador* de la suscripción de Azure. En Azure Portal, seleccione su nombre de usuario en la esquina superior derecha y, después, seleccione **Permisos** para ver los permisos que tiene en la suscripción. Si tiene acceso a varias suscripciones, elija la correspondiente. 

Para crear y administrar recursos secundarios para Data Factory incluidos los conjuntos de datos, servicios vinculados, canalizaciones, desencadenadores y entornos de ejecución de integración, se aplican los siguientes requisitos:
- Para crear y administrar recursos secundarios en Azure Portal, debe pertenecer al rol **Colaborador de Data Factory** en el nivel de **grupo de recursos** u otro nivel superior.
- Para crear y administrar recursos secundarios con Powershell o el SDK, el rol de **Colaborador** en el nivel de recurso u otro nivel superior es suficiente.

Para obtener instrucciones de ejemplo sobre cómo agregar un usuario a un rol, consulte el artículo sobre la [adición de roles](../cost-management-billing/manage/add-change-subscription-administrator.md).

## <a name="set-up-permissions"></a>Configuración de permisos

Después de crear una instancia de Data Factory, es posible que desee permitir que otros usuarios trabajen con la factoría de datos. Para conceder este acceso a otros usuarios, tendrá que agregarlos al rol integrado **Colaborador de Data Factory** en el **grupo de recursos** que contiene la instancia de Data Factory.

### <a name="scope-of-the-data-factory-contributor-role"></a>Ámbito del rol Colaborador de Data Factory

La pertenencia al rol **Colaborador de Data Factory** permite a los usuarios hacer lo siguiente:
- Crear, editar y eliminar las factorías de datos y recursos secundarios, incluidos los conjuntos de datos, servicios vinculados, canalizaciones, desencadenadores y entornos de ejecución de integración.
- Implementar plantillas de Resource Manager. El método de implementación que utiliza Data Factory en Azure Portal es la implementación de Resource Manager.
- Administrar las alertas de App Insights para una factoría de datos.
- Crear incidencias de soporte técnico.

Para más información acerca de este rol, consulte el [rol Colaborador de Data Factory](../role-based-access-control/built-in-roles.md#data-factory-contributor).

### <a name="resource-manager-template-deployment"></a>Implementación de plantillas del Administrador de recursos

El rol **Colaborador de Data Factory**, a nivel del grupo de recursos o por encima de este, permite a los usuarios implementar plantillas de Resource Manager. Como resultado, los miembros del rol pueden usar plantillas de Resource Manager para implementar tanto factorías de datos como sus recursos secundarios, incluidos los conjuntos de datos, servicios vinculados, canalizaciones, desencadenadores y los entornos de ejecución de integración. La pertenencia a este rol no permite al usuario crear otros recursos.

Los permisos para Azure Repos y GitHub son independientes de los permisos de Data Factory. Como resultado, un usuario con permisos de repositorio que es solo un miembro del rol de lector puede editar los recursos secundarios de Data Factory y confirmar los cambios en el repositorio, pero no puede publicar estos cambios.

> [!IMPORTANT]
> La implementación de plantillas de Resource Manager con el rol **Colaborador de Data Factory** no eleva sus permisos. Por ejemplo, si implementa una plantilla que crea una máquina virtual de Azure y no tiene permiso para crear máquinas virtuales, la implementación falla con un error de autorización.

### <a name="custom-scenarios-and-custom-roles"></a>Escenarios personalizados y roles personalizados

A veces, tendrá que conceder distintos niveles de acceso para los usuarios de diferentes factorías de datos. Por ejemplo:
- Es posible que necesite un grupo en el que los usuarios solo tengan permisos en una factoría de datos específica.
- O puede que necesite un grupo en el que los usuarios solo puedan supervisar una o varias factorías de datos pero no pueden modificarlas.

Estos escenarios personalizados se consiguen mediante la creación de roles personalizados y la asignación de usuarios a esos roles. Para más información sobre los roles personalizados, consulte [Roles personalizados en Azure](..//role-based-access-control/custom-roles.md).

Estos son algunos ejemplos que muestran lo que se puede lograr con los roles personalizados:

- Permitir que un usuario cree, edite o elimine cualquier factoría de datos en un grupo de recursos de Azure Portal.

  Asigne el rol integrado **Colaborador de Data Factory** a nivel del grupo de recursos para el usuario. Si quiere permitir el acceso a cualquier factoría de datos en una suscripción, asigne el rol en el nivel de suscripción.

- Permitir que un usuario visualice (lectura) y supervise una factoría de datos, pero que no pueda editarla ni cambiarla.

  Asigne el role integrado **lector** en el recurso de la factoría de datos para el usuario.

- Permitir que un usuario edite una única factoría de datos en Azure Portal.

  Este escenario requiere dos asignaciones de roles.

  1. Asigne el rol integrado **colaborador** a nivel de la factoría de datos.
  2. Cree un rol personalizado con el permiso **Microsoft.Resources/deployments/** . Asigne este rol personalizado al usuario a nivel del grupo de recursos.

- Permitir que un usuario pueda probar la conexión en un servicio vinculado u obtener una vista previa de los datos de un conjunto de datos

    Cree un rol personalizado con permisos para las siguientes acciones: **Microsoft.DataFactory/factories/getFeatureValue/read** y **Microsoft.DataFactory/factories/getDataPlaneAccess/action**. Asigne este rol personalizado en el recurso de la factoría de datos para el usuario.

- Permitir que un usuario actualice una factoría de datos desde PowerShell o el SDK, pero no en Azure Portal.

  Asigne el role integrado **colaborador** en el recurso de la factoría de datos para el usuario. Este rol permite al usuario ver los recursos en Azure Portal, pero no tiene acceso a los botones **Publicar** y **Publicar todo**.

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de los roles en Azure - [Descripción de definiciones de roles](../role-based-access-control/role-definitions.md)

- Más información acerca del rol **Colaborador de Data Factory** - [rol Colaborador de Data Factory](../role-based-access-control/built-in-roles.md#data-factory-contributor).
