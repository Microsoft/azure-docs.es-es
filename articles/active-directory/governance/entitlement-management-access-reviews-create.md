---
title: Creación de una revisión de acceso de un paquete de acceso de administración de derechos de Azure AD
description: Aprenda a crear una directiva de revisión de acceso para los paquetes de acceso de administración de derechos en las revisiones de acceso de Azure Active Directory (versión preliminar).
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 012eefe9140703a62d7bb1074ab763191a0976cb
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798518"
---
# <a name="create-an-access-review-of-an-access-package-in-azure-ad-entitlement-management"></a>Creación de una revisión de acceso de un paquete de acceso de administración de derechos de Azure AD

Para reducir el riesgo de acceso obsoleto, debe habilitar revisiones periódicas de los usuarios que tienen asignaciones activas en un paquete de acceso de administración de derechos de Azure AD. Puede habilitar las revisiones cuando cree un nuevo paquete de acceso o edite un paquete de acceso existente. En este artículo se describe cómo habilitar revisiones de acceso de paquetes de acceso.

## <a name="prerequisites"></a>Prerrequisitos

Para habilitar las revisiones de los paquetes de acceso, debe cumplir los requisitos previos para crear un paquete de acceso:
- Azure AD Premium P2
- Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

Para obtener más información, consulte [Requisitos de licencia](entitlement-management-overview.md#license-requirements).


## <a name="create-an-access-review-of-an-access-package"></a>Creación de una revisión de acceso de un paquete de acceso

Puede habilitar revisiones de acceso cuando [cree un paquete de acceso](entitlement-management-access-package-create.md) o [edite una directiva de paquetes de acceso existente](entitlement-management-access-package-lifecycle-policy.md). Siga estos pasos para habilitar las revisiones de acceso de un paquete de acceso:

1. Abra la pestaña **Ciclo de vida** de un paquete de acceso y desplácese hacia abajo hasta **Revisiones de acceso**.

1. Mueva el botón de alternancia **Exigir revisiones de acceso** a la posición **Sí**.

    ![Adición de la revisión de acceso](./media/entitlement-management-access-reviews/access-reviews-pane.png)

1. Especifique la fecha en la que se iniciarán las revisiones junto a **Empezar el**.

1. A continuación, establezca el valor de **Frecuencia de revisión** en **Anualmente**, **Semestral**,  **Trimestral** o **Mensual**.
Esta configuración determina la frecuencia con que se producirán las revisiones de acceso.

1. Establezca la opción **Duración** para definir cuántos días estará abierta cada revisión de la serie periódica para recibir comentarios de los revisores. Por ejemplo, puede programar una revisión anual que comienza el 1 de enero y está abierta para su revisión durante 30 días, de modo que los revisores tengan hasta el final del mes para responder.

1. Junto a **Revisores**, seleccione **Autorrevisión** si quiere que los usuarios realicen su propia revisión de acceso o seleccione **Revisores específicos** si quiere designar un revisor.

    ![Seleccione de Agregar revisores](./media/entitlement-management-access-reviews/access-reviews-add-reviewer.png)

1. Si seleccionó **revisores específicos**, especifique qué usuarios realizarán la revisión de acceso:
    1. Seleccione **Agregar revisores**.
    1. En el panel **Seleccionar revisores**, busque y seleccione los usuarios que desea que sean revisores.
    1. Cuando haya seleccionado los revisores, haga clic en el botón **Seleccionar**.

    ![Especificación de los revisores](./media/entitlement-management-access-reviews/access-reviews-select-reviewer.png)

1. En la parte inferior de la página, haga clic en **Revisar y crear** si va a crear un paquete de acceso o en **Actualizar** si va a editar un paquete de acceso.

## <a name="view-the-status-of-the-access-review"></a>Visualización del estado de la revisión de acceso

Después de la fecha de inicio, se mostrará una revisión de acceso en la sección **Revisiones de acceso**. Siga estos pasos para ver el estado de una revisión de acceso:

1. En **Gobierno de identidades**, haga clic en **Paquetes de acceso** y seleccione el paquete de acceso con el estado de revisión de acceso que quiera comprobar.   

1. Cuando se encuentre en la página de información general del paquete de acceso, haga clic en **Revisiones de acceso** en el menú de la izquierda.
    
    ![Selección de revisiones de acceso](./media/entitlement-management-access-reviews/access-review-status-access-package-overview.png)

1. Aparecerá una lista que contiene todas las directivas que tienen asociadas las revisiones de acceso. Haga clic en la revisión para ver su informe.

    ![Lista de revisiones de acceso](./media/entitlement-management-access-reviews/access-review-status-select-access-reviews.png)
   
1. Al ver el informe, se muestra el número de usuarios revisados y las acciones que realiza el revisor en ellos.

    ![Visualización del estado de la revisión](./media/entitlement-management-access-reviews/access-review-status.png)
 

## <a name="access-reviews-email-notifications"></a>Notificaciones por correo electrónico de revisiones de acceso
Se pueden designar revisores o los usuarios pueden revisar su propio acceso. De forma predeterminada, Azure AD envía un correo electrónico a los autorrevisores poco después de iniciar la revisión.

El correo electrónico incluirá instrucciones sobre cómo revisar el acceso a los paquetes de acceso. Si la revisión es para que los usuarios revisen su acceso, muéstreles las instrucciones sobre cómo realizar una autorrevisión de sus paquetes de acceso.
  
Si ha asignado usuarios invitados como revisores, y no han aceptado su invitación a Azure AD, no recibirán correos electrónicos de las revisiones de acceso a Azure AD. Primero deben aceptar la invitación y crear una cuenta con Azure AD para poder recibir los mensajes de correo electrónico. 

## <a name="next-steps"></a>Pasos siguientes

- [Revisión del acceso a los paquetes de acceso](entitlement-management-access-reviews-review-access.md)
- [Autorrevisión de paquetes de acceso](entitlement-management-access-reviews-self-review.md)
