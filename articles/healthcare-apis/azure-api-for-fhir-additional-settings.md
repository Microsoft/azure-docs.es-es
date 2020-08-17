---
title: Configuración adicional de Azure API for FHIR
description: Información general sobre la configuración adicional que se puede establecer para Azure API for FHIR
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.reviewer: matjazl
ms.author: cavoeg
author: CaitlinV39
ms.date: 11/22/2019
ms.openlocfilehash: 5de1d99442f307fc7850114915851f994258b537
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87845908"
---
# <a name="additional-settings-for-azure-api-for-fhir"></a>Configuración adicional de Azure API for FHIR

En esta guía paso a paso se revisa la configuración adicional que se recomienda establecer en Azure API for FHIR. Hay páginas adicionales que profundizan aún más en los detalles.

## <a name="configure-database-settings"></a>Configuración de las opciones de la base de datos

Azure API for FHIR usa una base de datos para almacenar sus datos. El rendimiento de la base de datos subyacente depende del número de unidades de solicitud (RU) seleccionado durante el aprovisionamiento del servicio o de la configuración de la base de datos una vez aprovisionado el servicio.

El rendimiento se debe aprovisionar de modo que se garantice que haya suficientes recursos del sistema disponibles para la base de datos en todo momento. El número de RU necesario para la aplicación depende de las operaciones que se realicen. Las operaciones pueden abarcar desde simples lecturas y escrituras hasta consultas más complejas.

Para obtener más información sobre cómo cambiar la configuración predeterminada, vea [Configuración de la base de datos](configure-database.md).

## <a name="access-control"></a>Control de acceso

Azure API for FHIR solo permite el acceso a FHIR API a los usuarios autorizados. Puede configurar usuarios autorizados mediante dos mecanismos diferentes. La forma principal y recomendada de configurar el control de acceso es usar e el [Control de acceso basado en rol (RBAC) de Azure](https://docs.microsoft.com/azure/role-based-access-control/), al cual puede acceder a través de la hoja **Control de acceso (IAM)** . RBAC de Azure solo funciona si se quiere proteger el acceso al plano de datos mediante el inquilino de Azure Active Directory asociado a la suscripción. Si se quiere usar otro inquilino, Azure API for FHIR ofrece un mecanismo de control de acceso al plano de datos de FHIR local. Las opciones de configuración no son tan completas cuando se usa el mecanismo RBAC local. Para obtener detalles, seleccione una de las siguientes opciones:

* [RBAC de Azure para el plano de datos de FHIR](configure-azure-rbac.md). Esta es la opción preferida cuando se usa el inquilino de Azure Active Directory asociado a la suscripción.
* [Control de acceso al plano de datos de FHIR local](configure-local-rbac.md). Use esta opción solo si necesita usar un inquilino de Azure Active Directory externo para el control de acceso al plano de datos. 

## <a name="enable-diagnostic-logging"></a>Activación del registro de diagnóstico
Se recomienda habilitar el registro de diagnóstico como parte de la configuración para poder supervisar el servicio y tener informes precisos para fines de cumplimiento. Para obtener detalles sobre cómo configurar el registro de diagnóstico, vea la [guía paso a paso](enable-diagnostic-logging.md) sobre cómo configurar el registro de diagnóstico, junto con algunas consultas de ejemplo. 

## <a name="use-custom-headers-to-add-data-to-audit-logs"></a>Uso de encabezados personalizados para agregar datos a los registros de auditoría
En Azure API for FHIR, se recomienda incluir información adicional procedente del sistema que llama en los registros. Para incluir esta información, puede usar encabezados personalizados.

Puede usar los encabezados personalizados para capturar varios tipos de información. Por ejemplo:

* Información de identidad o autorización
* Origen del autor de la llamada
* Organización de origen
* Detalles del sistema de cliente (registro médico electrónico, portal de pacientes)

Para agregar estos datos a los registros de auditoría, vea la guía paso a paso [Uso de encabezados HTTP personalizados para agregar datos a los registros de auditoría](use-custom-headers.md).

## <a name="next-steps"></a>Pasos siguientes

En esta guía paso a paso se configuran opciones adicionales para Azure API for FHIR.

Luego vea la serie de tutoriales para crear una aplicación web que lea datos de FHIR.

>[!div class="nextstepaction"]
>[Implementación de una aplicación de JavaScript](tutorial-web-app-fhir-server.md)