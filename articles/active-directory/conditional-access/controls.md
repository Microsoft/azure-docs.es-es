---
title: Controles personalizados con acceso condicional de Azure AD
description: Aprenda cómo funcionan los controles personalizados en el acceso condicional de Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 08/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: inbarc
ms.collection: M365-identity-device-management
ms.openlocfilehash: 59f83d232874a7702598b6ea5fdd458c101e7e79
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266585"
---
# <a name="custom-controls-preview"></a>Controles personalizados (versión preliminar)

Los controles personalizados son una funcionalidad en versión preliminar de Azure Active Directory. Al utilizar controles personalizados, los usuarios se redirigen a un servicio compatible para satisfacer requisitos de autenticación fuera de Azure Active Directory. Para satisfacer este control, el explorador de un usuario se redirige al servicio externo, lleva a cabo todas las actividades de autenticación necesarias y luego se vuelve a redirigir a Azure Active Directory. Azure Active Directory comprueba la respuesta y, si el usuario se autenticó o validó correctamente, seguirá en el flujo de acceso condicional.

> [!NOTE]
> Para obtener más información sobre los cambios que estamos planeando para la funcionalidad de control personalizado, consulte [Archivo de novedades](../fundamentals/whats-new-archive.md#upcoming-changes-to-custom-controls) de febrero de 2020.

## <a name="creating-custom-controls"></a>Creación de controles personalizados

Los controles personalizados funcionan con un conjunto limitado de proveedores de autenticación aprobados. Para crear un control personalizado, primero debe ponerse en contacto con el proveedor que desea utilizar. Cada proveedor que no es Microsoft tiene su propio proceso y requisitos para registrarse, suscribirse o pasar a formar parte del servicio e indicar que usted desea integrarse con el acceso condicional. En ese momento, el proveedor le proporcionará un bloque de datos en formato JSON. Estos datos permiten al proveedor y al acceso condicional trabajar conjuntamente para su inquilino, crean el nuevo control y definen cómo el acceso condicional puede indicar si los usuarios han realizado correctamente la comprobación con el proveedor.

Copie los datos JSON y, a continuación, péguelos en el cuadro de texto relacionado. No realice ningún cambio en JSON a menos que entienda explícitamente dicho cambio. Realizar cualquier cambio podría interrumpir la conexión entre el proveedor y Microsoft y dejarle tanto a usted como a sus usuarios fuera de sus cuentas.

La opción para crear un control personalizado está en la sección **Administrar** de la página **Acceso condicional**.

![Interfaz de controles personalizados en el acceso condicional](./media/controls/custom-controls-conditional-access.png)

Al hacer clic en **Nuevo control personalizado** se abre una hoja con un cuadro de texto para los datos JSON del control.  

![Nuevo control personalizado](./media/controls/new-custom-controls-conditional-access.png)

## <a name="deleting-custom-controls"></a>Eliminación de controles personalizados

Para eliminar un control personalizado, primero debe asegurarse de que no se usa en ninguna directiva de acceso condicional. Cuando haya terminado:

1. Vaya a la lista Controles personalizados.
1. Haga clic en ...  
1. Seleccione **Eliminar**.

## <a name="editing-custom-controls"></a>Edición de controles personalizados

Para editar un control personalizado, debe eliminar el control actual y crear un nuevo control con la información actualizada.

## <a name="known-limitations"></a>Restricciones conocidas

Los controles personalizados no se pueden usar con la automatización de Identity Protection, que requiere Azure Multi-Factor Authentication, el autoservicio de restablecimiento de contraseña (SSPR) de Azure AD, para satisfacer los requisitos de notificaciones multifactor, para elevar roles en Privileged Identity Manager (PIM), como parte de la inscripción de dispositivos de Intune o al unir dispositivos a Azure AD.

## <a name="next-steps"></a>Pasos siguientes

- [Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

- [Modo de solo informe](concept-conditional-access-report-only.md)

- [Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)
