---
title: 'ML Studio (clásico): administración de áreas de trabajo de Azure'
description: Administración del acceso a las áreas de trabajo de Azure Machine Learning Studio (clásico) e implementación y administración de servicios web de la API de Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 02/27/2017
ms.openlocfilehash: 6f3661956c61eb20f4c336f8644f85eb7e1b4100
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87433125"
---
# <a name="manage-an-azure-machine-learning-studio-classic-workspace"></a>Administración de un área de trabajo de Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


> [!NOTE]
> Para obtener información acerca de cómo administrar servicios web en el portal Servicios web Machine Learning, consulte [Administración de un servicio web mediante el portal Servicios web Azure Machine Learning](manage-new-webservice.md).
> 
> 

Puede administrar áreas de trabajo de Machine Learning Studio (clásico) en Azure Portal.



## <a name="use-the-azure-portal"></a>Uso de Azure Portal

Para administrar un área de trabajo de Studio en Azure Portal (clásico):

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) mediante una cuenta de administrador de la suscripción de Azure.
2. En el cuadro de búsqueda de la parte superior de la página, escriba "áreas de trabajo de Machine Learning Studio (clásico)" y, después, seleccione **Áreas de trabajo de Machine Learning Studio (clásico)**.
3. Haga clic en el área de trabajo que desea administrar.

Además de la información de administración de recursos estándar y de las opciones disponibles, puede:

- Ver **Propiedades**: esta página muestra la información del área de trabajo y de los recursos, y puede cambiar la suscripción y el grupo de recursos con el que esta área de trabajo está conectado.
- **Resincronizar las claves de almacenamiento**: el área de trabajo mantiene las claves de la cuenta de almacenamiento. Si la cuenta de almacenamiento cambia las claves, puede hacer clic en **Resincronizar claves** para sincronizar las claves con el área de trabajo.

Para administrar los servicios web asociados a esta área de trabajo de Studio (clásico), use el Portal de servicios web Machine Learning. Para obtener una información más completa, consulte [Administración de un servicio web mediante el portal Servicios web Azure Machine Learning](manage-new-webservice.md).

> [!NOTE]
> Para implementar o administrar nuevos servicios web, es preciso que se le haya asignado un rol de colaborador o administrador en la suscripción en la que se implementa el servicio web. Si invita a otro usuario a un área de trabajo de Machine Learning Studio (clásico), debe asignarle un rol de colaborador o administrador en la suscripción para que pueda implementar o administrar servicios web. 
> 
>Para más información acerca de cómo establecer los permisos de acceso, consulte [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Pasos siguientes
* Más información sobre cómo [implementar Machine Learning con plantillas de Azure Resource Manager](deploy-with-resource-manager-template.md). 
