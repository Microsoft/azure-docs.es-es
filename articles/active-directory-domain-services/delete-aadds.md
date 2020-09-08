---
title: Eliminar Azure Active Directory Domain Services | Microsoft Docs
description: Obtenga información para deshabilitar o eliminar Azure Active Directory Domain Services mediante Azure Portal
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 89e407e1-e1e0-49d1-8b89-de11484eee46
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: 993a73a42887b3869c097d14b6945a1093ac6889
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723068"
---
# <a name="delete-an-azure-active-directory-domain-services-managed-domain-using-the-azure-portal"></a>Obtenga información para deshabilitar o eliminar Azure Active Directory Domain Services mediante Azure Portal

Si ya no necesita un dominio administrado, puede eliminar Azure Active Directory Domain Services (Azure AD DS). No hay ninguna opción para desactivar o deshabilitar temporalmente un dominio administrado de Azure AD DS. Al eliminar el dominio administrado no se elimina ni afecta negativamente al inquilino de Azure AD.

En este artículo se muestra cómo usar el Azure Portal para eliminar un dominio administrado.

> [!WARNING]
> **La eliminación es permanente y no se puede deshacer.**
> 
> Cuando se elimina un dominio administrado, se realizan los siguientes pasos:
>   * Los controladores del dominio administrado se desabastecerán y se quitarán de la red virtual.
>   * Los datos del dominio administrado se eliminarán permanentemente. Estos datos incluyen unidades organizativas personalizadas, GPO, registros DNS personalizados, entidades de servicio, GMSA y elementos similares que haya creado.
>   * Las máquinas unidas al dominio administrado perderán la relación de confianza con el dominio y deberán separarse de este.
>       * No podrá iniciar sesión en estas máquinas con credenciales corporativas de AD. En su lugar, use las credenciales de administrador local para la máquina.

## <a name="delete-the-managed-domain"></a>Eliminar el dominio administrado

Para eliminar un dominio administrado, realice los siguientes pasos:

1. En Azure Portal, busque y seleccione **Azure AD Domain Services**.
1. Seleccione el nombre del dominio administrado, como *aaddscontoso.com*.
1. En la página **Información general**, seleccione **Eliminar**. Para confirmar la eliminación, escriba de nuevo el nombre de dominio del dominio administrado y seleccione **Eliminar**.

La eliminación del dominio administrado puede tardar 15-20 minutos o más.

## <a name="next-steps"></a>Pasos siguientes

Puede de [compartir comentarios][feedback] de las características que le gustaría ver en Azure AD DS.

Si desea empezar a usar Azure AD DS de nuevo, consulte [Crear y configurar un dominio administrado de Azure Active Directory Domain Services][create-instance].

<!-- INTERNAL LINKS -->
[feedback]: https://feedback.azure.com/forums/169401-azure-active-directory?category_id=160593%3fcategory_id%3d160593
[create-instance]: tutorial-create-instance.md