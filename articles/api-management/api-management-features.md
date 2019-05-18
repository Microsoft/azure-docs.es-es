---
title: Comparación de características de los planes de tarifa de Azure API Management | Microsoft Docs
description: En este artículo se comparan los planes de tarifa de API Management según las características que ofrecen.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/26/2018
ms.author: apimpm
ms.openlocfilehash: d881c8de7ecc32be0ca0cc2c5a82e0d2d51a7054
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780316"
---
# <a name="feature-based-comparison-of-the-azure-api-management-tiers"></a>Comparación de características de los planes de tarifa de Azure API Management

Cada [plan de tarifa](https://aka.ms/apimpricing) de API Management ofrece un conjunto diferente de características y [capacidad](api-management-capacity.md) por unidad. En la tabla siguiente se resumen las características clave disponibles en cada uno de los planes. Algunas características podrían funcionar de manera diferente o tener funcionalidades diferentes según el plan. En tales casos, se indican las diferencias en los artículos de documentación que describen estas características individuales.

| Característica                                                                                      | Consumo <sup>VERSIÓN PRELIMINAR</sup> | Desarrollador      | Básica          | Estándar       | Premium        |
| -------------------------------------------------------------------------------------------- | ----------------------------- | -------------- | -------------- | -------------- | -------------- |
| Integración de Azure AD<sup>1</sup>                                                             | No                            | Sí            | No             | Sí            | Sí            |
| Compatibilidad con redes virtuales (VNet)                                                               | No                            | Sí            | No             | No             | Sí            |
| Implementación en varias regiones                                                                      | No                            | No             | No             | No             | Sí            |
| Múltiples nombres de dominio personalizados                                                                 | No                            | No             | No             | No             | Sí            |
| Portal para desarrolladores<sup>2</sup>                                                                 | No                            | Sí            | Sí            | Sí            | Sí            |
| Memoria caché integrada                                                                               | No                            | Sí            | Sí            | Sí            | Sí            |
| Análisis integrado                                                                           | No                            | Sí            | Sí            | Sí            | Sí            |
| [Configuración de SSL](api-management-howto-manage-protocols-ciphers.md)                             | No                            | Sí            | Sí            | Sí            | Sí            |
| [Caché externa](https://aka.ms/apimbyoc)                                                    | Sí                           | Sí            | Sí            | Sí            | Sí            |
| [Autenticación de certificados de clientes](api-management-howto-mutual-certificates-for-clients.md) | No<sup>3</sup>                | Sí            | Sí            | Sí            | Sí            |
| [Copia de seguridad y restauración](api-management-howto-disaster-recovery-backup-restore.md)               | No                            | Sí            | Sí            | Sí            | Sí            |
| [Administración a través de Git](api-management-configuration-repository-git.md)                        | No                            | Sí            | Sí            | Sí            | Sí            |
| API de administración directa                                                                        | No                            | Sí            | Sí            | Sí            | Sí            |
| Métricas y registros de Azure Monitor                                                               | No<sup>4</sup>                | Sí            | Sí            | Sí            | Sí            |

<sup>1</sup> habilita el uso de Azure AD (y Azure AD B2C) como una identidad del proveedor para el usuario inicie sesión en el portal para desarrolladores.<br/>
<sup>2</sup> Incluye funciones relacionadas, por ejemplo, plantillas y notificaciones de usuarios, grupos, problemas, aplicaciones y correo electrónico.<br/>
<sup>3</sup> autenticación de certificados de cliente se agregarán a la capa de consumo antes de su disponibilidad General.<br/>
<sup>4</sup> se agregará compatibilidad con la supervisión de Azure completa a la capa de consumo.
