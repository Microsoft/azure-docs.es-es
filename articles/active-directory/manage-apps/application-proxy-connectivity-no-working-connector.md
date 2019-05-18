---
title: No se encontró ningún grupo de conectores en funcionamiento para una aplicación de proxy de aplicación | Microsoft Docs
description: Solucione los problemas que puedan surgir cuando no haya ningún conector en funcionamiento en un grupo de conectores para su aplicación con el proxy de aplicación de Azure AD
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
ms.date: 05/21/2018
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4f829b8e8a4bc08b43d3c30a6333771ccd4e26e8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65783619"
---
# <a name="no-working-connector-group-found-for-an-application-proxy-application"></a>No se encontró ningún grupo de conectores en funcionamiento para una aplicación de proxy de aplicación

Este artículo lo ayuda a solucionar problemas habituales que surgen cuando no se detecta ningún conector para una aplicación de proxy de aplicación integrada con Azure Active Directory.

## <a name="overview-of-steps"></a>Introducción a los pasos
Si no hay ningún conector en funcionamiento en un grupo de conectores para la aplicación, hay varias maneras de resolver el problema:

-   Si no tiene ningún conector en el grupo, puede hacer lo siguiente:

    -   Descargar un conector nuevo a la derecha en el servidor local y lo asigna a este grupo

    -   Mueva un conector activo al grupo.

-   Si no tiene ningún conector activo en el grupo, puede hacer lo siguiente:

    -   Identifique el motivo por el que el conector está inactivo y resuélvalo.

    -   Mueva un conector activo al grupo.

Para averiguar el problema, abra el menú “Proxy de la aplicación” en la aplicación y vea el mensaje de advertencia del grupo de conectores. Si no hay ningún conector en el grupo, el mensaje de advertencia especifica que el grupo necesita al menos un conector. Si no tiene conectores activos, el mensaje de advertencia lo explica. Es habitual tener conectores inactivos. 

   ![Selección de grupos de conectores en Azure Portal](./media/application-proxy-connectivity-no-working-connector/no-active-connector.png)

Para ver información detallada sobre cada una de estas opciones, consulte la sección correspondiente a continuación. Las instrucciones dan por supuesto que empieza desde la página de administración del conector. Si ve el mensaje de error anterior, puede ir a esta página si hace clic en el mensaje de advertencia. También puede obtener la página yendo a **Azure Active Directory** y haciendo clic en **Aplicaciones empresariales** y en **Proxy de la aplicación**.

   ![Administración de grupos de conectores en Azure Portal](./media/application-proxy-connectivity-no-working-connector/app-proxy.png)

## <a name="download-a-new-connector"></a>Descarga de un nuevo conector

Para descargar un conector nuevo, use el botón "Descargar conector" en la parte superior de la página.

Instale el conector en una máquina con línea directa de visión a la aplicación de back-end. Normalmente, el conector se instala en el mismo servidor que la aplicación. Después de la descarga, el conector debería aparecer en este menú. Haga clic en el conector y compruebe en la lista desplegable "Grupo de conectores" que pertenece al grupo adecuado. Guarde el cambio.

   ![Descarga del conector desde Azure Portal](./media/application-proxy-connectivity-no-working-connector/download-connector.png)
   
## <a name="move-an-active-connector"></a>Movimiento de un conector activo

Si tiene un conector activo que debería pertenecer a este grupo y tiene línea de visión a la aplicación back-end de destino, puede moverlo al grupo asignado. Para ello, haga clic en el conector. En el campo "Grupo de conectores", seleccione el grupo correcto en la lista desplegable y haga clic en Guardar.

## <a name="resolve-an-inactive-connector"></a>Resolución de un conector inactivo

Si los únicos conectores del grupo están inactivos, es probable que estén en una máquina donde no están desbloqueados todos los puertos necesarios.

Consulte el documento de solución de problemas de puertos para más información sobre cómo investigar este problema.

## <a name="next-steps"></a>Pasos siguientes
[Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md)


