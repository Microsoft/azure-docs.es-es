---
title: 'Sincronización de Azure AD Connect: descripción y personalización de la sincronización | Microsoft Docs'
description: Se explica cómo funciona la sincronización de Azure AD Connect y cómo personalizarla.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ee4bf802-045b-4da0-986e-90aba2de58d6
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/08/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: bf014feb9ef08b7a24c9640679c7c0baadf298c0
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89277160"
---
# <a name="azure-ad-connect-sync-understand-and-customize-synchronization"></a>Sincronización de Azure AD Connect: comprender y personalizar la sincronización
Los servicios de sincronización de Azure Active Directory Connect (sincronización de Azure AD Connect) es un componente principal de Azure AD Connect. Se encargan de todas las operaciones relacionadas con la sincronización de datos de identidad entre el entorno local y Azure AD. Sincronización de Azure AD Connect es el sucesor de DirSync, Sincronización de Azure AD y Forefront Identity Manager con Conector Azure Active Directory configurado.

Este tema es la principal referencia sobre la **sincronización de Azure AD Connect** (que también se conoce como **motor de sincronización**) y en el que se muestran vínculos a todos los otros demás temas relacionados con ella. Para obtener vínculos para Azure AD Connect, consulte [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).

El servicio de sincronización consta de dos componentes, el componente de **sincronización de Azure AD Connect** local y el lado del servicio en Azure AD llamado **servicio de sincronización de Azure AD Connect**.

## <a name="azure-ad-connect-sync-topics"></a>Temas de sincronización de Azure AD Connect
| Tema | ¿Qué aspectos cubre y cuándo debe leerlo? |
| --- | --- |
| **Fundamentos de la sincronización de Azure AD Connect** | |
| [Comprensión de la arquitectura](concept-azure-ad-connect-sync-architecture.md) |Para quienes no estén familiarizados con el motor de sincronización y quieran obtener información sobre la arquitectura y los términos empleados. |
| [Conceptos técnicos](how-to-connect-sync-technical-concepts.md) |Una versión abreviada del tema de la arquitectura que explica brevemente los términos empleados. |
| [Topologías de Azure AD Connect](plan-connect-topologies.md) |Describe las distintas topologías y escenarios que admite el motor de sincronización. |
| **Configuración personalizada** | |
| [Sincronización de Azure AD Connect: ejecución por segunda vez del Asistente para la instalación](how-to-connect-installation-wizard.md) |Explica qué opciones tiene disponibles si ejecuta de nuevo el Asistente para instalación de Azure AD Connect. |
| [Conocimiento del aprovisionamiento declarativo](concept-azure-ad-connect-sync-declarative-provisioning.md) |Describe el modelo de configuración denominado aprovisionamiento declarativo. |
| [Explicación de las expresiones declarativas de aprovisionamiento](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md) |Describe la sintaxis para el lenguaje de expresiones que se usa en el aprovisionamiento declarativo. |
| [Introducción a la configuración predeterminada](concept-azure-ad-connect-sync-default-configuration.md) |Describe las reglas no incluidas y la configuración predeterminada. También describe cómo colaboran las reglas para que los escenarios no incluidos funcionen. |
| [Descripción de usuarios y contactos](concept-azure-ad-connect-sync-user-and-contacts.md) |Continúa desde el tema anterior y describe cómo la configuración de los usuarios y contactos funciona conjuntamente, especialmente en un entorno de bosques múltiples. |
| [How to make a change to the default configuration (Realización de un cambio en la configuración predeterminada)](how-to-connect-sync-change-the-configuration.md) |En él se explica cómo cambiar una configuración común para los flujos de atributo. |
| [Prácticas recomendadas de cambio de la configuración predeterminada](how-to-connect-sync-best-practices-changing-default-configuration.md) |En él se describen las limitaciones de compatibilidad y se proporcionan directrices para realizar cambios en la configuración de fábrica. |
| [Configuración del filtrado](how-to-connect-sync-configure-filtering.md) |Describe las distintas opciones para limitar qué objetos se sincronizan con Azure AD y cómo configurar estas opciones paso a paso. |
| **Características y escenarios** | |
| [Evitar eliminaciones accidentales](how-to-connect-sync-feature-prevent-accidental-deletes.md) |En él se describe la característica para *evitar eliminaciones accidentales* y se explica cómo configurarla. |
| [Scheduler](how-to-connect-sync-feature-scheduler.md) |En él se describe el programador integrado que permite importar, sincronizar y exportar datos. |
| [Implementación de la sincronización de hash de contraseñas](how-to-connect-password-hash-synchronization.md) |Describe cómo funciona la sincronización de contraseñas, cómo implementarla y cómo utilizarla y solucionar problemas. |
| [Escritura diferida de dispositivos](how-to-connect-device-writeback.md) |Describe cómo funciona la reescritura de dispositivos en Azure AD Connect. |
| [Extensiones de directorio](how-to-connect-sync-feature-directory-extensions.md) |Describe cómo extender el esquema de Azure AD con sus propios atributos personalizados. |
| [Office 365 PreferredDataLocation](how-to-connect-sync-feature-preferreddatalocation.md) |Describe cómo incluir los recursos de Office 365 del usuario en la misma región en la que está el usuario. |
| **Servicio de sincronización** | |
| [Características del servicio de sincronización de Azure AD Connect](how-to-connect-syncservice-features.md) |Describe el lado del servicio de sincronización y cómo cambiar la configuración de sincronización de Azure AD. |
| [Resistencia de atributos duplicados](how-to-connect-syncservice-duplicate-attribute-resiliency.md) |En él se describe cómo habilitar y usar la resistencia de valores de atributos duplicados de **userPrincipalName** y **proxyAddresses**. |
| **Operaciones e interfaz de usuario** | |
| [Synchronization Service Manager](how-to-connect-sync-service-manager-ui.md) |En él se describe la interfaz de usuario de Synchronization Service Manager, incluidas las pestañas [Operations](how-to-connect-sync-service-manager-ui-operations.md) (Operaciones), [Connectors](how-to-connect-sync-service-manager-ui-connectors.md) (Conectores), [Metaverse Designer](how-to-connect-sync-service-manager-ui-mvdesigner.md) (Diseñador de metaverso) y [Metaverse Search](how-to-connect-sync-service-manager-ui-mvsearch.md) (Búsqueda de metaverso). |
| [Tareas y consideraciones operativas](./how-to-connect-sync-staging-server.md) |Describe las preocupaciones operativas, como la recuperación ante desastres. |
| **Procedimiento para:** | |
| [Restablecer la cuenta de Azure AD](how-to-connect-azureadaccount.md) |Procedimiento para restablecer las credenciales de la cuenta de servicio usada para conectarse desde el servicio de sincronización de Azure AD Connect a Azure AD. |
| **Más información y referencias** | |
| [Puertos](reference-connect-ports.md) |Enumera los puertos que necesita abrir entre el motor de sincronización y los directorios locales y Azure AD. |
| [Atributos sincronizados con Azure Active Directory](reference-connect-sync-attributes-synchronized.md) |Enumera todos los atributos que se sincronizan entre los AD locales y Azure AD. |
| [Referencia de funciones](reference-connect-sync-functions-reference.md) |Enumera todas las funciones disponibles en el aprovisionamiento declarativo. |

## <a name="additional-resources"></a>Recursos adicionales
* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)