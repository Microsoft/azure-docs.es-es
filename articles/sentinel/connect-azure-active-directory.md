---
title: Conexión de datos de Azure Active Directory a Azure Sentinel | Microsoft Docs
description: Aprenda a recopilar datos de Azure Active Directory y a transmitir registros de inicio de sesión y registros de auditoría de Azure AD a Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 0a8f4a58-e96a-4883-adf3-6b8b49208e6a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/11/2020
ms.author: yelevin
ms.openlocfilehash: 279f54c3de964580cc37d1288a6e1b7726348e10
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88208615"
---
# <a name="connect-data-from-azure-active-directory-azure-ad"></a>Conexión de datos desde Azure Active Directory (Azure AD)



Puede usar el conector integrado de Azure Sentinel para recopilar datos de [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) y transmitirlos a Azure Sentinel. El conector permite transmitir [registros de inicio de sesión](../active-directory/reports-monitoring/concept-sign-ins.md) y [registros de auditoría](../active-directory/reports-monitoring/concept-audit-logs.md).

## <a name="prerequisites"></a>Requisitos previos


- Cualquier licencia de Azure AD (Gratis, O365 P1 o P2) basta para ingerir registros de inicio de sesión en Azure Sentinel. Se pueden aplicar cargos adicionales por gigabyte para Azure Monitor (Log Analytics) y Azure Sentinel.

- El usuario debe tener asignado el rol Colaborador de Azure Sentinel en el área de trabajo.

- El usuario debe tener asignados los roles Administrador global o Administrador de seguridad en el inquilino desde el que quiere transmitir los registros.

- El usuario debe tener permisos de lectura y escritura en la configuración de diagnóstico de Azure AD para poder ver el estado de la conexión. 


## <a name="connect-to-azure-active-directory"></a>Conectarse a Azure Active Directory

1. En el menú de navegación de Azure Sentinel, seleccione **Conectores de datos**.

1. En la galería de conectores de datos, seleccione **Azure Active Directory** y luego **Open connector page** (Abrir página del conector).

1. Active las casillas que se encuentran junto a los registros que quiere transmitir a Azure Sentinel y haga clic en **Conectar**.

1. Puede seleccionar si desea que las alertas de Azure AD generen incidentes automáticamente en Azure Sentinel. En **Create incidents** (Crear incidentes) seleccione **Habilitar** para habilitar la regla de análisis predeterminada que crea incidentes automáticamente a partir de alertas generadas en el servicio de seguridad conectado. Luego, puede editar esta regla en **Analytics** (Análisis) y **Active rules** (Reglas activas).

1. Si quiere usar el esquema correspondiente en Log Analytics para consultar las alertas de Azure AD, escriba `SigninLogs` o `AuditLogs` en la ventana de consulta.

## <a name="next-steps"></a>Pasos siguientes
En este documento, aprendió a conectar Azure Active Directory a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
