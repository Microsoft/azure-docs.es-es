---
title: 'Sincronización de Azure AD Connect: Extensiones de directorio | Microsoft Docs'
description: En este tema se describe la característica de extensiones de directorio en Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 995ee876-4415-4bb0-a258-cca3cbb02193
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/12/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1b013c4edcaceeea585494bd8924ccea08aa1d37
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89277058"
---
# <a name="azure-ad-connect-sync-directory-extensions"></a>Sincronización de Azure AD Connect: Sincronización de Azure AD Connect: Extensiones de directorio
Puede usar extensiones de directorio para ampliar el esquema de Azure Active Directory (Azure AD) con sus propios atributos desde Active Directory local. Esta característica le permite compilar aplicaciones de LOB mediante el consumo de atributos que sigue administrando de forma local. Estos atributos se pueden consumir mediante [extensiones](/graph/extensibility-overview
). Puede ver los atributos disponibles mediante el [Explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer). También puede usar esta característica para crear grupos dinámicos en Azure AD.

En la actualidad, ninguna carga de trabajo de Office 365 consume estos atributos.

## <a name="customize-which-attributes-to-synchronize-with-azure-ad"></a>Personalización de los atributos que se van a sincronizar con Azure AD

Configure qué atributos adicionales desea sincronizar en la ruta de acceso de configuración personalizada en el Asistente para instalación.

> [!NOTE]
> En versiones de Azure AD Connect anteriores a 1.2.65.0, el cuadro de búsqueda **Atributos disponibles** distingue mayúsculas de minúsculas.

![Asistente para la extensión de esquema](./media/how-to-connect-sync-feature-directory-extensions/extension2.png)  

La instalación muestra los atributos siguientes, que son candidatos válidos:

* Tipos de objetos de usuario y de grupo
* Atributos de valor único: cadena, booleano, entero, binario
* Atributos con varios valores: cadena, binario


>[!NOTE]
> Aunque Azure AD Connect admite la sincronización con varios valores de atributos de Active Directory con Azure AD como extensiones de directorio con varios valores, actualmente no hay forma de recuperar o consumir los datos cargados en los atributos de extensiones de directorio con varios valores.

La lista de atributos se lee de la memoria caché de esquemas creada durante la instalación de Azure AD Connect. Si ha ampliado el esquema de Active Directory con atributos adicionales, se debe [actualizar el esquema](how-to-connect-installation-wizard.md#refresh-directory-schema) para que los nuevos atributos estén visibles.

Un objeto de Azure AD puede tener hasta 100 atributos para las extensiones de directorio. La longitud máxima es de 250 caracteres. Si un valor de atributo es mayor, el motor de sincronización lo trunca.

## <a name="configuration-changes-in-azure-ad-made-by-the-wizard"></a>Cambios de configuración en Azure AD realizados por el asistente

Durante la instalación de Azure AD Connect, se registra una aplicación donde estos atributos estén disponibles. Puede ver esta aplicación en el Portal de Azure. Su nombre es siempre **Tenant Schema Extension App**.

![Aplicación de extensión de esquema](./media/how-to-connect-sync-feature-directory-extensions/extension3new.png)

Asegúrese de seleccionar **Todas las aplicaciones** para ver esta aplicación.

Los atributos tienen el prefijo **extensión \_{IdDeAplicación}\_** . IdDeAplicación tiene el mismo valor en todos los atributos del inquilino de Azure AD. Necesitará este valor para los demás escenarios de este tema.

## <a name="viewing-attributes-using-the-microsoft-graph-api"></a>Visualización de atributos mediante Microsoft Graph API

Estos atributos ya están disponibles a través de Microsoft Graph API, mediante el [Explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer#).

>[!NOTE]
> En Microsoft Graph API, debe pedir que se devuelvan los atributos. Seleccione los atributos de forma explícita así: `https://graph.microsoft.com/beta/users/abbie.spencer@fabrikamonline.com?$select=extension_9d98ed114c4840d298fad781915f27e4_employeeID,extension_9d98ed114c4840d298fad781915f27e4_division`.
>
> Para más información, consulte [Microsoft Graph: Usar parámetros de consulta](/graph/query-parameters#select-parameter).

## <a name="use-the-attributes-in-dynamic-groups"></a>Uso de los atributos en grupos dinámicos

Uno de los escenarios más útiles consiste en usar estos atributos en grupos dinámicos de seguridad o de Office 365.

1. Cree un grupo en Azure AD. Asígnele un nombre adecuado y asegúrese de que la opción **Tipo de pertenencia** está establecida en **Usuario dinámico**.

   ![Captura de pantalla con un nuevo grupo](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup1.png)

2. Seleccione **Agregar una consulta dinámica**. Si examina las propiedades, no verá estos atributos extendidos. Antes debe agregarlos manualmente. Haga clic en **Obtener propiedades de extensión personalizadas**, escriba el identificador de la aplicación y haga clic en **Actualizar propiedades**.

   ![Captura de pantalla en la que se han agregado las extensiones de directorio](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup2.png) 

3. Abra la lista desplegable de propiedades y observe que los atributos que ha agregado ahora están visibles.

   ![Captura de pantalla con los nuevos atributos en la interfaz de usuario](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup3.png)

   Complete la expresión para satisfacer sus necesidades. En nuestro ejemplo, la regla se establece en **(user.extension_9d98ed114c4840d298fad781915f27e4_division -eq "Sales and marketing")** .

4. Una vez creado el grupo, dé a Azure AD algún tiempo para rellenar los miembros y, a continuación, revíselos.

   ![Captura de pantalla con miembros en el grupo dinámico](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup4.png)  

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la configuración de la [Sincronización de Azure AD Connect](how-to-connect-sync-whatis.md) .

Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).