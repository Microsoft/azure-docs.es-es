---
title: 'Migración de directivas de acceso condicional con autenticación multifactor: Azure Active Directory'
description: En este artículo se muestra cómo migrar una directiva clásica que requiere autenticación multifactor en Azure Portal.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: nigu
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5cc99241f5fa7c09c69db13dcb3f6c3aaa75e376
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87027400"
---
# <a name="migrate-a-classic-policy-in-the-azure-portal"></a>Migración de una directiva clásica en Azure Portal

En este ejemplo se muestra cómo migrar una directiva clásica que requiere **autenticación multifactor** para una aplicación en la nube. Aunque no es un requisito previo, le recomendamos que lea [Migración de directivas clásicas en Azure Portal](policy-migration.md) antes de iniciar la migración de las directivas clásicas.

![Detalles de la directiva clásica que requieren MFA para la aplicación Salesforce](./media/policy-migration/33.png)

El proceso de migración consta de los pasos siguientes:

1. [Abra la directiva clásica](#open-a-classic-policy) para obtener las opciones de configuración.
1. Cree una nueva directiva de acceso condicional de Azure AD para reemplazar la directiva clásica. 
1. Deshabilite la directiva clásica.

## <a name="open-a-classic-policy"></a>Abrir una directiva clásica

1. En [Azure Portal](https://portal.azure.com), vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Directivas clásicas**.

   ![Vista de directivas clásicas](./media/policy-migration-mfa/12.png)

1. En la lista de directivas clásicas, seleccione la directiva que desea migrar. Documente los valores de configuración para poder recrearlos en una nueva directiva de acceso condicional.

## <a name="create-a-new-conditional-access-policy"></a>Creación de una directiva de acceso condicional

1. En [Azure Portal](https://portal.azure.com), vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Para crear una nueva directiva de acceso condicional, seleccione **Nueva directiva**.
1. En la página **Nuevo**, en el cuadro de texto **Nombre**, escriba un nombre para la directiva.
1. En la sección **Asignaciones**, haga clic en **Usuarios y grupos**.
   1. Si tiene todos los usuarios seleccionados en la directiva clásica, haga clic en **Todos los usuarios**. 
   1. Si tiene grupos seleccionados en la directiva clásica, haga clic en **Seleccionar usuarios y grupos** y, después, seleccione los grupos y usuarios necesarios.
   1. Si tiene los grupos excluidos, haga clic en la pestaña **Excluir** y, después, seleccione los grupos y usuarios necesarios. 
   1. Seleccione **Listo**
1. En la sección**Asignación**, haga clic en **Aplicaciones en la nube o acciones**.
1. En la página **Aplicaciones en la nube o acciones**, siga estos pasos:
   1. Haga clic en **Seleccionar aplicaciones**.
   1. Haga clic en **Seleccionar**.
   1. En la página **Seleccionar**, seleccione la aplicación en la nube y haga clic en **Seleccionar**.
   1. En la página **Aplicaciones en la nube**, haga clic en **Listo**.
1. Si seleccionó **Requerir autenticación multifactor**:
   1. En la sección **Controles de acceso**, haga clic en **Conceder**.
   1. En la página **Conceder**, haga clic en **Conceder acceso** y, después, haga clic en **Requerir autenticación multifactor**.
   1. Haga clic en **Seleccionar**.
1. Haga clic en **Activado** para habilitar la directiva y, después, seleccione **Guardar**.

   ![Creación de directivas de acceso condicional](./media/policy-migration-mfa/conditional-access-policy-migration.png)

## <a name="disable-the-classic-policy"></a>Deshabilitar la directiva clásica

Para deshabilitar la directiva clásica, haga clic en **Deshabilitar** en la vista **Detalles**.

![Deshabilitación de directivas clásicas](./media/policy-migration-mfa/14.png)

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre la migración de directivas clásicas, consulte [Migración de directivas clásicas en Azure Portal](policy-migration.md).
- [Uso del modo de solo informe de acceso condicional para determinar el impacto de las nuevas decisiones de directiva.](concept-conditional-access-report-only.md)