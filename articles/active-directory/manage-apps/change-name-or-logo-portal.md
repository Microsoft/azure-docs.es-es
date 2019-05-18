---
title: Cambiar el nombre o el logotipo de una aplicación empresarial en Azure Active Directory | Microsoft Docs
description: Cómo cambiar el nombre o el logotipo de una aplicación empresarial personalizada en Azure Active Directory
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/05/2019
ms.author: mimart
ms.reviewer: asteen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 30da8d6843c27c42d4d99adef50b9ad98a131c95
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780920"
---
# <a name="change-the-name-or-logo-of-an-enterprise-application-in-azure-active-directory"></a>Cambiar el nombre o el logotipo de una aplicación empresarial en Azure Active Directory

Cambiar el nombre o el logotipo de una aplicación empresarial personalizada en Azure Active Directory (Azure AD) es fácil. Debe tener los permisos adecuados para realizar estos cambios, y debe ser el creador de la aplicación personalizada.

## <a name="how-do-i-change-an-enterprise-applications-name-or-logo"></a>¿Cómo se puede cambiar nombre o el logotipo de una aplicación empresarial?

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/) con una cuenta que sea un administrador global en el directorio. El **centro de administración de Azure Active Directory** aparece la página.
2. En el panel izquierdo, seleccione **Aplicaciones empresariales**. Aparece la lista de sus aplicaciones empresariales.
3. Seleccione una aplicación. Aparece la página de información general de la aplicación.
4. En el panel de información general de la aplicación, bajo el **administrar** encabezado, seleccione **propiedades**. El **propiedades** aparece la página.
5. Si desea cambiar el nombre, seleccione el **nombre** cuadro, escriba el nombre nuevo y presione **ENTRAR**.
6. Si desea cambiar el logotipo, busque el **logotipo** campo y seleccione el icono de carpeta junto a la **seleccionar un archivo** cuadro, que está por debajo de la imagen de logotipo de la aplicación actual.

   ![Selección del comando Propiedades](./media/change-name-or-logo-portal/change-logo.png)

   En caso contrario, si no se cambia el logotipo, vaya al paso 8.
7. En el selector de archivos, seleccione el archivo que desee como el nuevo logotipo. El nombre del archivo aparece en el cuadro debajo de la imagen de logotipo actual.

   > [!NOTE]
   > Azure requiere que la imagen de logotipo para ser un archivo PNG, y se aplica límites de ancho, alto y el tamaño de archivo.
8. Seleccione **Guardar**. Si elige un nuevo logotipo, el **logotipo** los cambios de imagen del campo para reflejar el nuevo archivo de logotipo.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Ver los grupos y miembros de su organización en Azure Active Directory](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Asignar un usuario o grupo a una aplicación empresarial](assign-user-or-group-access-portal.md)
* [Eliminación de asignaciones de usuario o grupo de una aplicación empresarial](remove-user-or-group-access-portal.md)
* [Deshabilitar los inicios de sesión de los usuarios de una aplicación empresarial](disable-user-sign-in-portal.md)
