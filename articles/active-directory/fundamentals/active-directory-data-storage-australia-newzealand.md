---
title: 'Almacenamiento de datos de identidad para clientes australianos: Azure AD'
description: Obtenga información sobre dónde almacena Azure Active Directory los datos relacionados con la identidad de los clientes australianos.
services: active-directory
author: ajburnle
manager: daveba
ms.author: ajburnle
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 7/21/2020
ms.custom: it-pro, seodec18, references_regions
ms.collection: M365-identity-device-management
ms.openlocfilehash: edc53152fe532a071de744db77d5c08d799c445a
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89318529"
---
# <a name="customer-data-storage-for-australian-and-new-zealand-customers-in-azure-active-directory"></a>Almacenamiento de datos de clientes australianos y neozelandeses en Azure Active Directory

Azure Active Directory (Azure AD) almacena los datos de los clientes en una ubicación geográfica según el país que se especifique al suscribirse a un servicio en línea de Microsoft. Entre los servicios en línea de Microsoft se incluyen Office 365 y Azure. 

Para obtener información sobre la ubicación de los datos de Azure AD y otros servicios de Microsoft, consulte la sección [Dónde se encuentran sus datos](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) del Centro de confianza de Microsoft.

A partir del 26 de febrero de 2020, Microsoft comenzó a almacenar los datos de clientes de Azure AD para los nuevos inquilinos con una dirección de facturación de Australia o Nueva Zelanda dentro de los centros de datos de Australia. Entre el 1 de mayo de 2020 y el 31 de octubre de 2020, Microsoft migrará los inquilinos existentes que tengan una dirección de facturación de Australia o Nueva Zelanda a los centros de datos de Australia sin necesidad de que el cliente intervenga. El proceso de migración no implica ningún tiempo de inactividad para los clientes y no afectará a ninguna funcionalidad del inquilino durante la migración.

Además, algunas características de Azure AD todavía no admiten el almacenamiento de datos de clientes en Australia. Visite la [asignación de datos de Azure AD](https://msit.powerbi.com/view?r=eyJrIjoiYzEyZTc5OTgtNTdlZS00ZTVkLWExN2ItOTM0OWU4NjljOGVjIiwidCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsImMiOjV9) para obtener información específica sobre las características. Por ejemplo, Microsoft Azure Multi-Factor Authentication almacena los datos de los clientes en Estados Unidos y los procesa de forma global. Consulte [Residencia de datos y datos de cliente en Azure Multi-Factor Authentication](../authentication/concept-mfa-data-residency.md).

> [!NOTE]
> Los productos y servicios de Microsoft, así como las aplicaciones de terceros que se integran en Azure AD tienen acceso a datos del cliente. Evalúe cada producto, servicio y aplicación que use para determinar la manera en que cada producto, servicio y aplicación específico procesa los datos del cliente, y si cumple los requisitos de almacenamiento de datos de la empresa. Para obtener más información sobre la residencia de los datos de los servicios de Microsoft, consulte la sección [Where is your data located?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) (Dónde se encuentran sus datos) de Microsoft Trust Center.