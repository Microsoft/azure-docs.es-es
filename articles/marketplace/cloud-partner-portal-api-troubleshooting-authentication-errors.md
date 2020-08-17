---
title: Solución de errores de autenticación comunes | Azure Marketplace
description: Ofrece asistencia con errores de autenticación comunes al usar las API de Cloud Partner Portal.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: aa4269d68a176db97e36e6fbae4eba32041d7e05
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87271474"
---
# <a name="troubleshooting-common-authentication-errors"></a>Solución de errores de autenticación comunes

> [!NOTE]
> Las API de Cloud Partner Portal se integran en el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los cambios que se muestran en la [referencia de API de Cloud Partner Portal](./cloud-partner-portal-api-overview.md) para asegurarse de que el código siga funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Este artículo ofrece asistencia con errores de autenticación comunes al usar las API de Cloud Partner Portal.

## <a name="unauthorized-error"></a>Error no autorizado

Si recibe errores `401 unauthorized` sistemáticamente, compruebe que dispone de un token de acceso válido.  Si aún no lo ha hecho, cree una aplicación de Azure Active Directory (Azure AD) básica y una entidad de servicio, tal y como se describe en [Uso del portal para crear una aplicación de Azure Active Directory y una entidad de servicio con acceso a los recursos](../active-directory/develop/howto-create-service-principal-portal.md). A continuación, utilice la aplicación o una solicitud HTTP POST simple para comprobar el acceso.  Deberá incluir el identificador de inquilino, el identificador de aplicación, el identificador de objeto y la clave secreta para obtener el token de acceso, tal como se muestra en la siguiente imagen:

![Solución del error 401](./media/cloud-partner-portal-api-troubleshooting-authentication-errors/troubleshooting-401-error.jpg)

## <a name="forbidden-error"></a>Error prohibido

Si recibe un error `403 forbidden`, asegúrese de que se ha agregado la entidad de servicio correcta a su cuenta de publicador en Cloud Partner Portal.
Siga los pasos descritos en la página [Requisitos previos](./cloud-partner-portal-api-prerequisites.md) para agregar la entidad de servicio en el portal.

Si se ha agregado la entidad de servicio correcta, compruebe el resto de la información. Preste especial atención al identificador de objeto especificado en el portal. En la página de registro de la aplicación de Azure Active Directory hay dos identificadores de objeto; el que debe usar es el identificador de objeto local. Para encontrar el valor correcto, vaya a la página **Registros de aplicaciones** de su aplicación y haga clic en el nombre de la aplicación en **Aplicación administrada en directorio local**. Esto le llevará a las propiedades locales de la aplicación, donde puede encontrar el identificador de objeto correcto en la página **Propiedades**, tal como se muestra en la siguiente ilustración. Además, asegúrese de usar el identificador de publicador correcto al agregar la entidad de servicio y realizar la llamada de API.

![Solución del error 403](./media/cloud-partner-portal-api-troubleshooting-authentication-errors/troubleshooting-403-error.jpg)
