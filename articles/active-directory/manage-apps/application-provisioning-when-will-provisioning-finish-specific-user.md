---
title: Averiguar cuándo un usuario específico podrá acceder a una aplicación | Microsoft Docs
description: Cómo averiguar cuándo un usuario sumamente importante puede acceder a una aplicación que se ha configurado para el aprovisionamiento de usuarios con Azure AD
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 08a78a311771395f7c2113f03d5137d59999d7a7
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783865"
---
# <a name="find-out-when-a-specific-user-will-be-able-to-access-an-application"></a>Averiguar cuándo un usuario específico podrá acceder a una aplicación
Cuando se utiliza el aprovisionamiento automático de usuarios con una aplicación, Azure AD aprovisiona y actualiza automáticamente las cuentas de usuario en una aplicación en función de aspectos tales como la [asignación de usuario y grupo](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal) con un intervalo programado periódicamente, normalmente cada 10 minutos.

## <a name="how-long-does-it-take"></a>¿Cuánto tiempo tarda?

El tiempo necesario para aprovisionar un usuario determinado depende principalmente de si ya se ha producido una sincronización inicial "completa" o no.

La sincronización inicial entre Azure AD y una aplicación puede tardar desde 20 minutos hasta varias horas, según el tamaño del directorio de Azure AD y el número de usuarios en el ámbito de aprovisionamiento. 

Las sincronizaciones posteriores a la inicial serán más rápidas (por ejemplo, unos 10 minutos), ya que el servicio de aprovisionamiento almacena marcas de agua que representan el estado de ambos sistemas tras la sincronización inicial, lo cual mejora el rendimiento de las posteriores.

## <a name="how-to-check-the-status-of-a-user"></a>Comprobación del estado de un usuario

Para ver el estado de aprovisionamiento de un usuario seleccionado, consulte los registros de auditoría de Azure AD.

Puede acceder a los registros de auditoría de aprovisionamiento en Azure Portal, en la pestaña **Azure Active Directory &gt; Aplicaciones empresariales &gt; \[Nombre de la aplicación\] &gt; Registros de auditoría**. Filtre los registros en la categoría **Aprovisionamiento de cuentas** para ver solo los eventos de aprovisionamiento para esa aplicación. Puede buscar usuarios por el "identificador de coincidencia" que se configuró para ellos en las asignaciones de atributos. 

Por ejemplo, si configuró el "nombre principal de usuario" o la "dirección de correo electrónico" como atributo de coincidencia en el lado de Azure AD y el usuario que no se aprovisiona tiene el valor "audrey@contoso.com", busque "audrey@contoso.com" en los registros de auditoría y revise las entradas devueltas.

Los registros de auditoría de aprovisionamiento registran todas las operaciones realizadas por el servicio de aprovisionamiento, lo que incluye:

* Consultar en Azure AD los usuarios asignados que están en el ámbito de aprovisionamiento
* Consultar en la aplicación de destino la existencia de esos usuarios
* Comparar los objetos de usuario entre el sistema
* Agregar, actualizar o deshabilitar la cuenta de usuario en el sistema de destino en función de la comparación

## <a name="next-steps"></a>Pasos siguientes
[Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-app-provisioning)
