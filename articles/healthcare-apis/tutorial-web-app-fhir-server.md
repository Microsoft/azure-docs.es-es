---
title: 'Tutorial de aplicación web: configuración de Azure API for FHIR'
description: En este tutorial se explica un ejemplo para implementar una aplicación web sencilla. En este primer tutorial se describen los requisitos previos y la implementación de Azure API for FHIR.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 4b39c32da04efa3782cb2166c8e1137029b21258
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852963"
---
# <a name="deploy-javascript-app-to-read-data-from-fhir-service"></a>Implementación de una aplicación de JavaScript para leer datos del servicio de FHIR
En este tutorial va a implementar una pequeña aplicación de JavaScript, que lee los datos de un servicio de FHIR. A continuación se muestran los pasos de este tutorial:
1. Implementación de un servidor de FHIR
1. Registro de una aplicación cliente pública
1. Prueba de acceso a la aplicación
1. Creación de una aplicación web que lea los datos de FHIR

## <a name="prerequisites"></a>Requisitos previos
Antes de iniciar este conjunto de tutoriales, necesitará los siguientes elementos:
1. Una suscripción de Azure
1. Un inquilino de Azure Active Directory
1. [Postman](https://www.getpostman.com/) instalado

> [!NOTE]
> En este tutorial, el servicio de FHIR, la aplicación Azure AD y los usuarios de Azure AD se encuentran en el mismo inquilino de Azure AD. Si no es así, puede seguir este tutorial, pero es posible que necesite profundizar en algunos de los documentos a los que se hace referencia para realizar pasos adicionales.

## <a name="deploy-azure-api-for-fhir"></a>Implementación de Azure API for FHIR
El primer paso del tutorial es obtener correctamente la configuración de Azure API for FHIR.

1. Implementación de [Azure API for FHIR](fhir-paas-portal-quickstart.md)
1. Una vez que tenga implementado Azure API for FHIR, configure las opciones de [CORS](configure-cross-origin-resource-sharing.md) en Azure API for FHIR y seleccione CORS. 
    1. Establezca **Orígenes** en *.
    1. Establezca **Encabezados** en *.
    1. En **Métodos**, elija **Seleccionar todo**.
    1. Establezca **Antigüedad máxima** en **600**.

## <a name="next-steps"></a>Pasos siguientes
Ahora que la configuración de Azure API for FHIR ya está implementada, está preparado para registrar una aplicación cliente pública.

>[!div class="nextstepaction"]
>[Registrar una aplicación cliente pública](tutorial-web-app-public-app-reg.md)
