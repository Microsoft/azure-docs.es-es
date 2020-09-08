---
title: Integración de los registros con ArcSight mediante Azure Monitor | Microsoft Docs
description: Aprenda a integrar los registros de Azure Active Directory con ArcSight mediante Azure Monitor.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: b37bef0d-982e-4e28-86b2-6c61ca524ae1
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/19/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: fc7bcab04da005fd0d46d18e7b708dcb1c9d58e5
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230524"
---
# <a name="integrate-azure-active-directory-logs-with-arcsight-using-azure-monitor"></a>Integración de los registros de Azure Active Directory con ArcSight mediante Azure Monitor

[Micro Focus ArcSight](https://software.microfocus.com/products/siem-security-information-event-management/overview) es una solución de administración de eventos e información de seguridad (SIEM) que ayuda a detectar y responder a amenazas de seguridad en la plataforma. Ahora puede enrutar los registros de Azure Active Directory (Azure AD) a ArcSight con Azure Monitor mediante el conector de ArcSight para Azure AD. Esta característica permite supervisar el inquilino para comprobar si se compromete la seguridad mediante ArcSight.  

En este artículo, aprenderá a enrutar los registros de Azure AD a ArcSight mediante Azure Monitor. 

## <a name="prerequisites"></a>Prerrequisitos

Para usar esta característica, necesita:
* Un centro de eventos de Azure que contenga registros de actividad de Azure AD. Aprenda cómo [transmitir los registros de actividad a un centro de eventos](./tutorial-azure-monitor-stream-logs-to-event-hub.md). 
* Una instancia configurada de ArcSight Syslog NG Daemon SmartConnector (SmartConnector) o del equilibrador de carga de ArcSight. Si los eventos se envían al equilibrador de carga de ArcSight, el equilibrador de carga los envía posteriormente a SmartConnector.

Descarga y abra la [guía de configuración de ArcSight SmartConnector para el centro de eventos de Azure Monitor](https://community.microfocus.com/t5/ArcSight-Connectors/SmartConnector-for-Microsoft-Azure-Monitor-Event-Hub/ta-p/1671292). Esta guía contiene los pasos necesarios para instalar y configurar ArcSight SmartConnector para Azure Monitor. 

## <a name="integrate-azure-ad-logs-with-arcsight"></a>Integración de registros de Azure AD con ArcSight

1. En primer lugar, complete los pasos de la sección **Requisitos previos** de la guía de configuración. Esta sección incluye los siguientes pasos:
    * Establecer permisos de usuario en Azure, a fin de garantizar que haya un usuario con el rol **propietario** para implementar y configurar el conector.
    * Abra los puertos del servidor con Syslog NG Daemon SmartConnector, para que se pueda acceder desde Azure. 
    * La implementación ejecuta un script de Windows PowerShell, por lo que debe habilitar PowerShell para que ejecute scripts en el equipo en el que desea implementar el conector.

2. Siga los pasos descritos en la sección de **implementación del conector** de la guía de configuración para implementar el conector. En esta sección se explica cómo descargar y extraer el conector, configurar las propiedades de la aplicación y ejecutar el script de implementación desde la carpeta extraída. 

3. Siga los pasos descritos en la sección de **verificación de la implementación en Azure** para asegurarse de que el conector está configurado y funciona correctamente. Verifique lo siguiente:
    * Que las funciones de Azure necesarias se han creado en la suscripción de Azure.
    * Que los registros de Azure AD se transmiten al destino correcto. 
    * Que la configuración de la aplicación de la implementación se conserva en la configuración de la aplicación en Azure Function App. 
    * Que se ha creado un grupo de recursos para ArcSight en Azure, con una aplicación de Azure AD para el conector de ArcSight y las cuentas de almacenamiento que contienen los archivos asignados con el formato CEF.

4. Por último, complete los pasos posteriores a la implementación descritos en la sección de **configuraciones posteriores a la implementación** de la guía de configuración. En esta sección se explica cómo realizar una configuración adicional si tiene un plan de App Service para prevenir que las aplicaciones de función queden inactivas tras un período de tiempo de expiración, cómo configurar el streaming de los registros de recursos desde el centro de eventos y cómo actualizar el certificado del almacén de claves de SysLog NG Daemon SmartConnector para asociarlo con la cuenta de almacenamiento recién creada.

5. En la guía de configuración también se explica cómo personalizar las propiedades del conector en Azure y cómo actualizar y desinstalar el conector. También hay una sección dedicada a las mejoras de rendimiento, incluida la actualización a un [plan de consumo de Azure](https://azure.microsoft.com/pricing/details/functions) y la configuración de un equilibrador de carga de ArcSight si la carga de eventos es mayor de lo que puede manipular un único conector Syslog NG Daemon SmartConnector.

## <a name="next-steps"></a>Pasos siguientes

[Guía de configuración de ArcSight SmartConnector para el centro de eventos de Azure Monitor](https://community.microfocus.com/t5/ArcSight-Connectors/SmartConnector-for-Microsoft-Azure-Monitor-Event-Hub/ta-p/1671292)