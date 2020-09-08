---
title: 'Delegación de roles por tarea de administrador: Azure Active Directory | Microsoft Docs'
description: Roles a delegar para tareas de identidad en Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: reference
ms.date: 05/04/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 692b20bf301bbd7bf03ea1af5527e08d947b50da
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798370"
---
# <a name="administrator-roles-by-admin-task-in-azure-active-directory"></a>Roles de administrador por tarea de administrador en Azure Active Directory

En este artículo, puede encontrar la información necesaria para restringir los permisos de administrador de un usuario mediante la asignación de roles con privilegios mínimos en Azure Active Directory (Azure AD). Encontrará las tareas de administrador organizadas por área de características y el rol con privilegios mínimos necesario para realizar cada tarea, junto con roles de administrador no global que pueden realizar la tarea.

## <a name="application-proxy"></a>Proxy de aplicación

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Configurar aplicación de proxy de aplicación | Administrador de aplicaciones | 
Configurar propiedades del grupo de conectores | Administrador de aplicaciones | 
Crear registro de aplicación cuando se deshabilita la capacidad para todos los usuarios | Desarrollador de aplicaciones | Administrador de aplicaciones en la nube, Administrador de aplicaciones
Crear grupo de conectores | Administrador de aplicaciones | 
Eliminar grupo de conectores | Administrador de aplicaciones | 
Deshabilitar el proxy de aplicación | Administrador de aplicaciones | 
Descargar servicio de conector | Administrador de aplicaciones | 
Leer toda la configuración | Administrador de aplicaciones | 

## <a name="external-identitiesb2c"></a>Identidades externas o B2C

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Crear directorios de Azure AD B2C | Todos los usuarios que no sean invitados ([consulte la documentación](../fundamentals/users-default-permissions.md)) | 
Crear aplicaciones B2C | Administrador global | 
Crear aplicaciones empresariales | Administrador de aplicaciones en la nube | Administrador de aplicaciones
Crear, leer, actualizar y eliminar directivas de B2C | Administrador de directivas B2C con IEF | 
Crear, leer, actualizar y eliminar proveedores de identidades | Administrador de proveedor de identidades externo | 
Crear, leer, actualizar y eliminar flujos de usuario de restablecimiento de contraseña | Administrador de flujos de usuarios con id. externo | 
Crear, leer, actualizar y eliminar flujos de usuario de edición de perfiles | Administrador de flujos de usuarios con id. externo | 
Crear, leer, actualizar y eliminar flujos de usuario de inicio de sesión | Administrador de flujos de usuarios con id. externo | 
Crear, leer, actualizar y eliminar flujos de usuario de registro |Administrador de flujos de usuarios con id. externo | 
Crear, leer, actualizar y eliminar atributos de usuario | Administrador de atributos de flujos de usuarios con id. externo | 
Crear, leer, actualizar y eliminar usuarios | Administrador de usuarios
Leer toda la configuración | Lector global | 
Leer registros de auditoría de B2C | Lector global ([consulte la documentación](../../active-directory-b2c/faq.md)) | 

> [!NOTE]
> Los lectores globales de Azure AD B2C no tienen los mismos permisos que los administradores globales de Azure AD. Si tiene privilegios de administrador global de Azure AD B2C, asegúrese de que se encuentra en un directorio de Azure AD B2C, no de Azure AD.

## <a name="company-branding"></a>Personalización de marca de empresa

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Configuración de la personalización de marca de la compañía | Administrador global | 
Leer toda la configuración | Lectores de directorios | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md))

## <a name="company-properties"></a>Propiedades de la empresa

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Configurar propiedades de la empresa | Administrador global | 

## <a name="connect"></a>Conectar

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Autenticación de paso a través | Administrador global  | 
Leer toda la configuración | Lector global | Administrador global  |
Inicio de sesión único de conexión directa | Administrador global  | 

## <a name="cloud-provisioning"></a>Aprovisionamiento en la nube

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Autenticación de paso a través | Administrador de identidades híbridas  | 
Leer toda la configuración | Lector global | Administrador de identidades híbridas  |
Inicio de sesión único de conexión directa | Administrador de identidades híbridas  | 

## <a name="connect-health"></a>Connect Health

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Agregar o eliminar servicios | Propietario ([consulte la documentación](../hybrid/how-to-connect-health-operations.md)) | 
Aplicar correcciones de errores de sincronización | Colaborador ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Propietario
Configuración de notificaciones | Colaborador ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Propietario
Definición de configuración | Propietario ([consulte la documentación](../hybrid/how-to-connect-health-operations.md)) | 
Configurar notificaciones de sincronización | Colaborador ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Propietario
Leer informes de seguridad de ADFS | Lector de seguridad | Colaborador, Propietario
Leer toda la configuración | Lector ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Colaborador, Propietario
Leer errores de sincronización | Lector ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Colaborador, Propietario
Leer servicios de sincronización | Lector ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Colaborador, Propietario
Ver métricas y alertas | Lector ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Colaborador, Propietario
Ver métricas y alertas | Lector ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Colaborador, Propietario
Ver métricas y alertas del servicio de sincronización | Lector ([consulte la documentación](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)) | Colaborador, Propietario

## <a name="custom-domain-names"></a>Nombres de dominio personalizados

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Administrar dominios | Administrador global | 
Leer toda la configuración | Lectores de directorios | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md))

## <a name="domain-services"></a>Servicios de dominio

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Crear una instancia de Azure AD Domain Services | Administrador global | 
Realizar todas las tareas de Azure AD Domain Services | Grupo de administradores de DC de Azure AD ([consulte la documentación](../../active-directory-domain-services/tutorial-create-management-vm.md#administrative-tasks-you-can-perform-on-a-managed-domain)) | 
Leer toda la configuración | Lector en la suscripción de Azure que contiene el servicio AD DS | 

## <a name="devices"></a>Dispositivos

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Deshabilitar dispositivo | Administrador de dispositivos en la nube | 
Habilitar dispositivo | Administrador de dispositivos en la nube | 
Leer configuración básica | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md)) | 
Leer claves de BitLocker | Lector de seguridad | Administrador de contraseñas, Administrador de seguridad

## <a name="enterprise-applications"></a>Aplicaciones empresariales

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Dar consentimiento a los permisos delegados | Administrador de aplicaciones en la nube | Administrador de aplicaciones
Dar consentimiento a permisos de aplicación sin incluir a Microsoft Graph | Administrador de aplicaciones en la nube | Administrador de aplicaciones
Dar consentimiento a permisos de aplicación para Microsoft Graph | Administrador de roles con privilegios | 
Dar consentimiento a aplicaciones que acceden a datos propios | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md)) | 
Crear aplicación empresarial | Administrador de aplicaciones en la nube | Administrador de aplicaciones
Administrar proxy de aplicación | Administrador de aplicaciones | 
Administrar configuración de usuario | Administrador global | 
Leer revisión de acceso de un grupo o de una aplicación | Lector de seguridad | Administrador de seguridad, Administrador de usuarios
Leer toda la configuración | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md)) | 
Actualizar asignaciones de aplicaciones empresariales | Propietario de la aplicación empresarial ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de aplicaciones en la nube, Administrador de aplicaciones
Actualizar propietarios de aplicaciones empresariales | Propietario de la aplicación empresarial ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de aplicaciones en la nube, Administrador de aplicaciones
Actualizar propiedades de aplicaciones empresariales | Propietario de la aplicación empresarial ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de aplicaciones en la nube, Administrador de aplicaciones
Actualizar aprovisionamiento de aplicaciones empresariales | Propietario de la aplicación empresarial ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de aplicaciones en la nube, Administrador de aplicaciones
Actualizar autoservicio de aplicaciones empresariales | Propietario de la aplicación empresarial ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de aplicaciones en la nube, Administrador de aplicaciones
Actualizar propiedades del inicio de sesión único | Propietario de la aplicación empresarial ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de aplicaciones en la nube, Administrador de aplicaciones

## <a name="entitlement-management"></a>Administración de derechos
Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Adición de recursos a un catálogo | Administrador de usuarios | Con la administración de derechos, puede delegar esta tarea en el propietario del catálogo ([consulte la documentación](../governance/entitlement-management-catalog-create.md#add-additional-catalog-owners)).
Adición de sitios de SharePoint Online al catálogo | Administrador global


## <a name="groups"></a>Grupos

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Asignación de la licencia | Administrador de usuarios | 
Crear grupo | Administrador de grupos | Administrador de usuarios
Crear, actualizar o eliminar revisión de acceso de un grupo o de una aplicación | Administrador de usuarios | 
Administrar expiración de grupos | Administrador de usuarios | 
Administración de la configuración de grupo | Administrador de grupos | Administrador de usuarios | 
Leer toda la configuración (excepto pertenencia oculta) | Lectores de directorios | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md))
Leer pertenencias ocultas | Miembro del grupo | Propietario del grupo, Administrador de contraseñas, Administrador de Exchange, Administrador de SharePoint, Administrador de equipos, Administrador de usuarios
Leer pertenencia a grupos con pertenencia oculta | Administrador del departamento de soporte técnico | Administrador de usuarios, Administrador de equipos
Revocar licencia | Administrador de licencias | Administrador de usuarios
Actualizar pertenencia a grupo | Propietario del grupo ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de usuarios
Actualizar propietarios de grupo | Propietario del grupo ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de usuarios
Actualizar propiedades de grupo | Propietario del grupo ([consulte la documentación](../fundamentals/users-default-permissions.md)) | Administrador de usuarios
Eliminar grupo | Administrador de grupos | Administrador de usuarios

## <a name="identity-protection"></a>Protección de identidad

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Configurar notificaciones de alerta| Administrador de seguridad | 
Configurar y habilitar o deshabilitar la directiva de MFA| Administrador de seguridad | 
Configurar y habilitar o deshabilitar la directiva de riesgo de inicio de sesión| Administrador de seguridad | 
Configurar y habilitar o deshabilitar la directiva de riesgo de usuario | Administrador de seguridad | 
Configurar resúmenes semanales | Administrador de seguridad| 
Descartar todas las detecciones de riesgo | Administrador de seguridad | 
Corregir o descartar vulnerabilidad | Administrador de seguridad | 
Leer toda la configuración | Lector de seguridad | 
Leer todas las detecciones de riesgo | Lector de seguridad | 
Leer vulnerabilidades | Lector de seguridad | 

## <a name="licenses"></a>Licencias

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Asignación de la licencia | Administrador de licencias | Administrador de usuarios
Leer toda la configuración | Lectores de directorios | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md))
Revocar licencia | Administrador de licencias | Administrador de usuarios
Probar o comprar suscripción | Administrador de facturación | 


## <a name="monitoring---audit-logs"></a>Supervisión: registros de auditoría

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Leer registros de auditoría | Lector de informes | Lector de seguridad, Administrador de seguridad

## <a name="monitoring---sign-ins"></a>Supervisión: inicios de sesión

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Leer registros de inicio de sesión | Lector de informes | Lector de seguridad, Administrador de seguridad

## <a name="multi-factor-authentication"></a>Multi-Factor Authentication

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Eliminar todas las contraseñas de aplicación existentes generadas por los usuarios seleccionados | Administrador global | 
Deshabilitar MFA | Administrador global | 
Habilitar MFA | Administrador global | 
Administrar la configuración del servicio MFA | Administrador global | 
Requerir a los usuarios seleccionados que vuelvan a proporcionar métodos de contacto | Administrador de autenticación | 
Restaurar autenticación multifactor en todos los dispositivos recordados  | Administrador de autenticación | 

## <a name="mfa-server"></a>Servidor MFA

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Bloqueo y desbloqueo de usuarios | Administrador global | 
Configurar bloqueo de cuentas | Administrador global | 
Configurar reglas de caché | Administrador global | 
Configurar alerta de fraude | Administrador global
Configuración de notificaciones | Administrador global | 
Configurar la omisión por única vez | Administrador global | 
Configurar la configuración de la llamada de teléfono | Administrador global | 
Configurar proveedores | Administrador global | 
Configuración del servidor | Administrador global | 
Leer informe de actividades | Lector global | 
Leer toda la configuración | Lector global | 
Leer estado del servidor | Lector global |  

## <a name="organizational-relationships"></a>Relaciones organizativas

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Administrar proveedores de identidad | Administrador de proveedor de identidades externo | 
Administración de la configuración | Administrador global | 
Administrar términos de uso | Administrador global | 
Leer toda la configuración | Lector global | 

## <a name="password-reset"></a>Restablecimiento de contraseña

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Configurar métodos de autenticación | Administrador global |
Configurar personalización | Administrador global |
Configurar notificación | Administrador global |
Configurar integración local | Administrador global |
Configurar las propiedades de restablecimiento de contraseña | Administrador de usuarios | Administrador global
Configurar registro | Administrador global |
Leer toda la configuración | Administrador de seguridad | Administrador de usuarios |

## <a name="privileged-identity-management"></a>Privileged Identity Management

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Asignación de usuarios a roles | Administrador de roles con privilegios | 
Configuración de los roles | Administrador de roles con privilegios | 
Ver actividad de auditoría | Lector de seguridad | 
Ver pertenencias a roles | Lector de seguridad | 

## <a name="roles-and-administrators"></a>Roles y administradores

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Administración de asignaciones de roles | Administrador de roles con privilegios | 
Leer revisión de acceso de un rol de Azure AD  | Lector de seguridad | Administrador de seguridad, Administrador de roles con privilegios
Leer toda la configuración | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md)) | 

## <a name="security---authentication-methods"></a>Seguridad: métodos de autenticación

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Configurar métodos de autenticación | Administrador global | 
Configuración de la protección con contraseña | Administrador de seguridad
Configuración del bloqueo inteligente | Administrador de seguridad
Leer toda la configuración | Lector global | 

## <a name="security---conditional-access"></a>Seguridad: acceso condicional

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Configurar direcciones IP de confianza de MFA | Administrador de acceso condicional | 
Crear controles personalizados | Administrador de acceso condicional | Administrador de seguridad
Crear ubicaciones con nombre | Administrador de acceso condicional | Administrador de seguridad
Creación de directivas | Administrador de acceso condicional | Administrador de seguridad
Crear términos de uso | Administrador de acceso condicional | Administrador de seguridad
Crear certificado de conectividad VPN | Administrador de acceso condicional | Administrador de seguridad
Eliminar directiva clásica | Administrador de acceso condicional | Administrador de seguridad
Eliminar términos de uso | Administrador de acceso condicional | Administrador de seguridad
Eliminar certificado de conectividad VPN | Administrador de acceso condicional | Administrador de seguridad
Deshabilitar directivas clásicas | Administrador de acceso condicional | Administrador de seguridad
Administrar controles personalizados | Administrador de acceso condicional | Administrador de seguridad
Administrar ubicaciones con nombre | Administrador de acceso condicional | Administrador de seguridad
Administrar términos de uso | Administrador de acceso condicional | Administrador de seguridad
Leer toda la configuración | Lector de seguridad | Administrador de seguridad
Leer ubicaciones con nombre | Lector de seguridad | Administrador de acceso condicional, Administrador de seguridad

## <a name="security---identity-security-score"></a>Seguridad: puntuación de seguridad de identidad

Tarea | Rol con privilegios mínimos | Roles adicionales | 
---- | --------------------- | ----------------
Leer toda la configuración | Lector de seguridad | Administrador de seguridad
Leer puntuación de seguridad | Lector de seguridad | Administrador de seguridad
Actualizar estado del evento | Administrador de seguridad | 

## <a name="security---risky-sign-ins"></a>Seguridad: inicios de sesión de riesgo

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Leer toda la configuración | Lector de seguridad | 
Leer inicios de sesión de riesgo | Lector de seguridad | 

## <a name="security---users-flagged-for-risk"></a>Seguridad: usuarios marcados en riesgo

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Descartar todos los eventos | Administrador de seguridad | 
Leer toda la configuración | Lector de seguridad | 
Leer usuarios marcados en riesgo | Lector de seguridad | 

## <a name="users"></a>Usuarios

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Agregar usuario a rol de directorio | Administrador de roles con privilegios | 
Agregar usuario a un grupo | Administrador de usuarios | 
Asignación de la licencia | Administrador de licencias | Administrador de usuarios
Crear usuario invitado | Invitador de usuarios | Administrador de usuarios
Crear usuario | Administrador de usuarios | 
Eliminación de usuarios | Administrador de usuarios | 
Invalidar tokens de actualización de los administradores limitados (consulte la documentación) | Administrador de usuarios | 
Invalidar tokens de actualización de usuarios no administradores (consulte la documentación) | Administrador de contraseñas | Administrador de usuarios
Invalidar tokens de actualización de administradores con privilegios (consulte la documentación) | Administrador de autenticación con privilegios | 
Leer configuración básica | Rol de usuario predeterminado ([consulte la documentación](../fundamentals/users-default-permissions.md)) | 
Restablecer contraseñas para administradores limitados (consulte la documentación) | Administrador de usuarios | 
Restablecer contraseñas de usuarios no administradores (consulte la documentación) | Administrador de contraseñas | Administrador de usuarios
Restablecer contraseñas de administradores con privilegios | Administrador de autenticación con privilegios | 
Revocar licencia | Administrador de licencias | Administrador de usuarios
Actualizar todas las propiedades excepto el nombre principal de usuario | Administrador de usuarios | 
Actualizar el nombre principal de usuario para administradores limitados (consulte la documentación) | Administrador de usuarios | 
Actualizar la propiedad nombre principal de usuario en administradores con privilegios (consulte la documentación) | Administrador global | 
Actualizar configuración de usuario | Administrador global | 


## <a name="support"></a>Soporte técnico

Tarea | Rol con privilegios mínimos | Roles adicionales
---- | --------------------- | ----------------
Enviar incidencia de soporte técnico | Administrador de servicios | Administrador de aplicaciones, Administrador de Azure Information Protection, Administrador de facturación, Administrador de aplicaciones en la nube, Administrador de cumplimiento, Administrador de Dynamics 365, Administrador de análisis de escritorio, Administrador de Exchange, Administrador de contraseñas, Administrador de Intune, Administrador de Skype Empresarial, Administrador de Power BI, Administrador de autenticación con privilegios, Administrador de SharePoint, Administrador de comunicaciones de equipos, Administrador de equipos, Administrador de usuarios, Administrador de Workplace Analytics

## <a name="next-steps"></a>Pasos siguientes

* [Asignación o eliminación de roles de administrador de Azure AD](directory-manage-roles-portal.md)
* [Referencia de roles de administrador de Azure AD](directory-assign-admin-roles.md)