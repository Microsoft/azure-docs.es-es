---
title: Referencia de la actividad de auditoría de Azure Active Directory (Azure AD) | Microsoft Docs
description: Obtenga una visión general de las actividades de auditoría que se pueden registrar en los registros de auditoría de Azure Active Directory (Azure AD).
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: a1f93126-77d1-4345-ab7d-561066041161
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 01/24/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 56f9976faf2a803e747bce8b851a3555e6d746b0
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231153"
---
# <a name="azure-ad-audit-activity-reference"></a>Referencia sobre actividades de auditoría de Azure AD

Con los informes de Azure Active Directory (Azure AD), puede obtener toda la información que necesita para determinar cómo funciona el entorno.

La arquitectura de los informes de Azure AD consta de los siguientes componentes:

- **Informes de actividad** 
    - [Inicios de sesión](concept-sign-ins.md): proporciona información sobre el uso de aplicaciones administradas y actividades de inicio de sesión de usuario
    - Los [registros de auditoría](concept-audit-logs.md) proporcionan rastreabilidad mediante los registros de todos los cambios realizados por diversas características de Azure AD. 
    
- **Informes de seguridad** 
    - [Inicios de sesión peligrosos](../identity-protection/overview-identity-protection.md): un inicio de sesión peligroso es un indicador de un intento de inicio de sesión que puede haber realizado alguien que no es el propietario legítimo de una cuenta de usuario. 
    - [Usuarios marcados en riesgo](../identity-protection/overview-identity-protection.md): un usuario en peligro es un indicador de una cuenta de usuario que puede haber estado en peligro. 

En este artículo se enumeran las actividades de auditoría que se pueden registrar en los registros de auditoría.

## <a name="access-reviews"></a>Revisiones de acceso

|Categoría de auditoría|Actividad|
|---|---|
|Revisiones de acceso|Revisión de acceso finalizada|
|Revisiones de acceso|Incorporación de aprobador para solicitar la aprobación|
|Revisiones de acceso|Incorporación de revisor para acceder a la revisión|
|Revisiones de acceso|Aplicación de la revisión de acceso|
|Revisiones de acceso|Creación de revisión de acceso|
|Revisiones de acceso|Creación de programa|
|Revisiones de acceso|Creación de aprobación de solicitud|
|Revisiones de acceso|Eliminación de revisión de acceso|
|Revisiones de acceso|Eliminación de programa|
|Revisiones de acceso|Vinculación al control del programa|
|Revisiones de acceso|Incorporación a las revisiones de acceso de Azure AD|
|Revisiones de acceso|Eliminación del revisor en la revisión de acceso|
|Revisiones de acceso|Solicitud de detención de la revisión|
|Revisiones de acceso|Solicitud de la aplicación de los resultados de la revisión|
|Revisiones de acceso|Revisión de la pertenencia al rol Rbac|
|Revisiones de acceso|Revisión de la asignación de aplicaciones|
|Revisiones de acceso|Revisión de la pertenencia al grupo|
|Revisiones de acceso|Revisión de la solicitud de aprobación de la solicitud|
|Revisiones de acceso|Desvinculación al control del programa|
|Revisiones de acceso|Actualización de la revisión de acceso|
|Revisiones de acceso|Actualización del estado de incorporación de revisiones de acceso de Azure AD|
|Revisiones de acceso|Actualización de la configuración de las notificaciones por correo de revisión de acceso|
|Revisiones de acceso|Actualización de la configuración de recuento de repeticiones de la revisión de acceso|
|Revisiones de acceso|Actualización de la configuración de duración de repeticiones de la revisión de acceso en días|
|Revisiones de acceso|Actualización de la configuración de tipo de fin de repeticiones de la revisión de acceso|
|Revisiones de acceso|Actualización de la configuración de tipo de repeticiones de la revisión de acceso|
|Revisiones de acceso|Actualización de la configuración de los recordatorios de revisión de acceso|
|Revisiones de acceso|Actualización de programa|
|Revisiones de acceso|Actualización de la aprobación de solicitud|
|Revisiones de acceso|Usuario deshabilitado|

## <a name="account-provisioning"></a>Aprovisionamiento de cuentas

|Categoría de auditoría|Actividad|
|---|---|
|Administración de aplicaciones|Recuperación de las concesiones de permisos de aplicaciones V2|
|Administración de aplicaciones|Recuperación de entidades de servicio de aplicaciones V2 en el inquilino actual|
|Administración de aplicaciones|Actualización de aplicación V1|
|Administración de aplicaciones|Actualización de aplicación V2|
|Administración de aplicaciones|Actualización de la concesión de permisos de aplicaciones V2|
|Administración de aplicaciones|Adición de OAuth2PermissionGrant|
|Administración de aplicaciones|Adición de la asignación de roles de aplicación a la entidad de servicio|

## <a name="application-proxy"></a>Proxy de aplicación

|Categoría de auditoría|Actividad|
|---|---|
|Administración de aplicaciones|Agregar aplicación|
|Administración de aplicaciones|Incorporación de un propietario a una aplicación|
|Administración de aplicaciones|Incorporación de un propietario a una entidad de servicio|
|Administración de aplicaciones|Incorporación de directiva a entidad de servicio|
|Administración de directorios|Agregar entidad de servicio|
|Administración de directorios|Agregar credenciales de la entidad de servicio|
|Administración de directorios|Consentimiento a la aplicación|
|Administración de directorios|Eliminar aplicación|
|Administración de directorios|Eliminación permanente de aplicación|
|Administración de directorios|Eliminación de Oauth2PermissionGrant|
|Administración de directorios|Eliminación de la asignación de roles de aplicación de la entidad de servicio|
|Administración de directorios|Eliminación de propietario de la aplicación|
|Resource|Eliminación de propietario de la entidad de servicio|
|Resource|Eliminación de directiva de entidad de servicio|
|Resource|Quitar entidad de servicio|


## <a name="automated-password-rollover"></a>Automated Password Rollover (Sustitución automática de contraseña)

|Categoría de auditoría|Actividad|
|---|---|
|Administración de aplicaciones|Quitar credenciales de la entidad de servicio|


## <a name="b2c"></a>B2C

|Categoría de auditoría|Actividad|
|---|---|
|Administración de aplicaciones|Restauración de aplicación|
|Administración de aplicaciones|Revocación de consentimiento|
|Administración de aplicaciones|Actualización de una aplicación|
|Administración de aplicaciones|Actualizar secretos externos|
|Administración de aplicaciones|Actualización de entidad de servicio|
|Administración de aplicaciones|Emisión de un token de acceso a la aplicación|
|Administración de aplicaciones|Emisión de un código de autorización para la aplicación|
|Administración de aplicaciones|Emisión de un id_token para la aplicación|
|Administración de aplicaciones|Validación de las credenciales de la cuenta local|
|Administración de aplicaciones|Validación de la autenticación del usuario|
|Administración de aplicaciones|Incorporación de permisos de aplicaciones V2|
|Administración de aplicaciones|Incorporación de una clave basada en un secreto ASCII a un contenedor de claves CPIM|
|Administración de aplicaciones|Incorporación de una clave a un contenedor de claves CPIM|
|Administración de aplicaciones|AdminPolicyDatas-SetResources|
|Administración de aplicaciones|AdminUserJourneys-GetResources|
|Administración de aplicaciones|AdminUserJourneys-RemoveResources|
|Authentication|AdminUserJourneys-SetResources|
|Authentication|Creación de IdentityProvider|
|Authentication|Creación de aplicación V1|
|Authentication|Creación de aplicación V2|
|Authentication|Creación de un dominio personalizado en el inquilino|
|Authorization|Creación de un nuevo AdminUserJourney|
|Authorization|Creación de un json de recurso localizado|
|Authorization|Creación de un nuevo IDP personalizado|
|Authorization|Creación de nuevo IDP|
|Authorization|Creación o actualización de un recurso del directorio B2C|
|Authorization|Creación de directiva|
|Authorization|Creación de directiva de trustFramework|
|Authorization|Creación de directiva de trustFramework con prefijo configurable|
|Authorization|Creación de atributo de usuario|
|Authorization|CreateTrustFrameworkPolicy|
|Authorization|Crea o actualiza un nuevo AdminUserJourney|
|Authorization|Eliminación de IDP|
|Authorization|Eliminación de IdentityProvider|
|Authorization|Eliminación de aplicación V1|
|Authorization|Eliminación de aplicación V2|
|Authorization|Eliminación de la concesión de permisos de aplicaciones V2|
|Authorization|Eliminación de un recurso del directorio B2C|
|Authorization|Eliminación de un contenedor de claves CPIM|
|Authorization|Eliminación de directiva de trustFramework|
|Authorization|Eliminación de atributo de usuario|
|Authorization|Habilitar característica B2C|
|Authorization|Obtención de recursos de directorio B2C en una suscripción|
|Authorization|Obtención de IDP personalizado|
|Authorization|Obtención de IDP|
|Authorization|Obtención de aplicaciones V1 y V2|
|Authorization|Obtención de aplicación V1|
|Authorization|Obtención de aplicaciones V1|
|Authorization|Obtención de aplicación V2|
|Authorization|Obtención de aplicaciones V2|
|Authorization|Obtención del recurso del directorio B2C|
|Authorization|Obtención de una lista de dominios personalizados en el inquilino|
|Authorization|Obtención de un recorrido del usuario|
|Authorization|Obtención de notificaciones de aplicación permitidas para el recorrido del usuario|
|Authorization|Obtención de notificaciones autoafirmadas permitidas para el recorrido del usuario|
|Authorization|Obtención de notificaciones autoafirmadas permitidas de directiva|
|Authorization|Obtención de la lista de notificaciones de salida disponibles|
|Authorization|Obtención de definiciones de contenido para el recorrido del usuario|
|Authorization|Obtención de IDP para un flujo de administración específico|
|Authorization|Obtención de metadatos de clave activos del contenedor de claves en JWK|
|Authorization|Obtención de una lista de todos los flujos de administración|
|Authorization|Obtención de lista de etiquetas para todos los flujos de administración de todos los usuarios|
|Authorization|Obtención de la lista de inquilinos de un usuario|
|Authorization|Obtención de notificaciones autoafirmadas de las cuentas locales|
|Authorization|Obtención de un json de recurso localizado|
|Authorization|Obtención de operaciones del proveedor de recursos Microsoft.AzureActiveDirectory|
|Authorization|Obtención de directivas|
|Authorization|Obtención de directiva|
|Authorization|Obtención de propiedades de recurso de un inquilino|
|Authorization|Obtención de una lista de IDP compatible|
|Authorization|Obtención de lista de IDP compatible del recorrido del usuario|
|Authorization|Obtención de información del inquilino|
|Authorization|Obtención de características permitidas del inquilino|
|Authorization|Obtención de una lista de IDP personalizada definida por el inquilino|
|Authorization|Obtención de una lista de IDP definida por el inquilino|
|Authorization|Obtención de una lista de IDP local definida por el inquilino|
|Authorization|Obtención de los detalles de inquilino de un usuario para la creación de recursos|
|Authorization|Obtención de lista de inquilinos|
|Authorization|Obtención de tenantDomains|
|Authorization|Obtención de la referencia cultural predeterminada para CPIM|
|Authorization|Obtención de los detalles de un flujo de administración|
|Authorization|Obtención de la lista de UserJourneys para este inquilino|
|Authorization|Obtención del conjunto de referencias culturales admitidas disponibles para CPIM|
|Authorization|Obtención de directiva de trustFramework|
|Authorization|Obtención de directiva de trustFramework como xml|
|Authorization|Obtención de atributo de usuario|
|Authorization|Obtención de atributos de usuario|
|Authorization|Obtención de lista de recorridos del usuario|
|Authorization|GetIEFPolicies|
|Authorization|GetIdentityProviders|
|Authorization|GetTrustFrameworkPolicy|
|Authorization|Obtiene un contenedor de claves CPIM en formato jwk|
|Authorization|Obtiene una lista de contenedores de claves del inquilino|
|Authorization|Obtiene el tipo de inquilino|
|Authorization|MigrateTenantMetadata|
|Authorization|Revisión de IdentityProvider|
|Authorization|PutTrustFrameworkPolicy|
|Authorization|PutTrustFrameworkpolicy|
|Authorization|Eliminación de un recorrido del usuario|
|Authorization|Restauración de una copia de seguridad del contenedor de claves CPIM|
|Authorization|Recuperación de las concesiones de permisos de aplicaciones V2|
|Authorization|Recuperación de entidades de servicio de aplicaciones V2 en el inquilino actual|
|Authorization|Actualización de IDP personalizado|
|Authorization|Actualización de IDP|
|Authorization|Actualización de IDP local|
|Authorization|Actualización de aplicación V1|
|Authorization|Actualización de aplicación V2|
|Authorization|Actualización de la concesión de permisos de aplicaciones V2|
|Authorization|Actualización de directiva|
|Authorization|Actualización de atributo de usuario|
|Authorization|Carga de una clave cifrada CPIM|
|Authorization|Autorización de usuario: la API está deshabilitada para el conjunto de características del inquilino|
|Authorization|Autorización de usuario: se ha concedido acceso al usuario como "Administrador del inquilino"|
|Authorization|Autorización de usuario: se han concedido al usuario derechos de acceso de "Usuarios autenticados"|
|Authorization|Comprobación de si está habilitada la característica B2C|
|Authorization|Comprobación de si está habilitada la característica|
|Authorization|Creación de programa|
|Authorization|Eliminación de programa|
|Authorization|Vinculación al control del programa|
|Authorization|Incorporación a las revisiones de acceso de Azure AD|
|Authorization|Desvinculación al control del programa|
|Authorization|Actualización de programa|
|Authorization|Deshabilitar SSO de escritorio|
|Authorization|Deshabilitar SSO de escritorio para un dominio específico|
|Authorization|Deshabilitar el proxy de aplicación|
|Authorization|Deshabilitar la autenticación de paso a través|
|Authorization|Habilitar SSO de escritorio|
|Administración de directorios|Habilitar SSO de escritorio para un dominio específico|
|Administración de directorios|Habilitar proxy de aplicación|
|Administración de directorios|Habilitar la autenticación de paso a través|
|Administración de directorios|Creación de un dominio personalizado en el inquilino|
|Administración de directorios|Habilitar característica B2C|
|Administración de directorios|Obtención de una lista de dominios personalizados en el inquilino|
|Administración de directorios|Obtención de propiedades de recurso de un inquilino|
|Administración de directorios|Obtención de información del inquilino|
|Administración de directorios|Obtención de características permitidas del inquilino|
|Administración de directorios|Obtención de tenantDomains|
|Clave|Obtiene el tipo de inquilino|
|Clave|Comprobación de si está habilitada la característica B2C|
|Clave|Comprobación de si está habilitada la característica|
|Clave|Agregar asociado a la compañía|
|Clave|Incorporación de dominio sin comprobar|
|Clave|Incorporación de dominio comprobado|
|Clave|Creación de compañía|
|Clave|Crear configuración de compañía.|
|Clave|Eliminar configuración de compañía.|
|Clave|Disminución del nivel de asociado|
|Clave|Se ha eliminado el directorio|
|Otros|Se ha eliminado el directorio de forma permanente|
|Otros|Directorio programado para eliminación|
|Resource|Promoción de la compañía al asociado|
|Resource|Purgar propiedades de administración de derechos|
|Resource|Eliminación de asociado de la compañía|
|Resource|Eliminación de dominio sin comprobar|
|Resource|Eliminación de dominio comprobado|
|Resource|Establecer información sobre la compañía|
|Resource|Establecimiento de la característica DirSync|
|Resource|Establecimiento de la marca DirSyncEnabled|
|Resource|Establecer asociación|
|Resource|Establecimiento de umbral de eliminación accidental|
|Resource|Establecimiento de la ubicación de datos permitida de empresa|
|Resource|Establecimiento de característica de empresa multinacional habilitada|
|Resource|Establecimiento de característica de directorio en inquilino|
|Resource|Establecer la autenticación de dominio|
|Resource|Establecer la configuración de la federación en el dominio|
|Resource|Establecimiento de directiva de contraseñas|
|Resource|Establecimiento de propiedades de administración de derechos|
|Resource|Actualización de compañía|
|Resource|Actualización de configuración de compañía|
|Resource|Actualizar dominio|
|Resource|Comprobar dominio|
|Resource|Comprobar dominio verificado por correo electrónico|
|Resource|Incorporación|
|Resource|Actualización de la configuración de alertas|
|Resource|Actualización de la configuración de la ingesta semanal|
|Resource|Deshabilitar la reescritura de contraseñas para el directorio|
|Resource|Habilitar la reescritura de contraseñas para el directorio|
|Resource|Adición de la asignación de roles de aplicación al grupo|
|Resource|Agregar grupo|
|Resource|Agregar miembro a grupo|
|Resource|Incorporación de un propietario a un grupo|
|Resource|Creación de configuración de grupo|
|Resource|Eliminar grupo|
|Resource|Eliminación de la configuración de grupo|
|Resource|Finalización de la aplicación de licencias basadas en grupos a los usuarios|
|Resource|Eliminación permanente de grupo|
|Resource|Eliminación de la asignación de roles de aplicación de grupo|
|Resource|Quitar miembro de grupo|
|Resource|Eliminación de propietario del grupo|
|Resource|Restauración de grupo|
|Resource|Establecimiento de licencia de grupo|
|Resource|Establecer grupo que va a administrar el usuario.|
|Resource|Empezar a aplicar licencias basadas en grupo a los usuarios|
|Resource|Desencadenamiento del nuevo cálculo de licencias de grupo|
|Resource|Actualizar grupo|
|Resource|Actualización de la configuración de grupo|
|Resource|Agregar miembro|
|Resource|Crear grupo|
|Resource|Eliminar grupo|
|Resource|Quitar miembro|
|Resource|Actualizar grupo|
|Resource|Aprobación de una solicitud pendiente para unirse a un grupo|
|Resource|Cancelación de una solicitud pendiente para unirse a un grupo|
|Resource|Creación de directiva de administración del ciclo de vida|
|Resource|Eliminación de una solicitud pendiente para unirse a un grupo|
|Resource|Rechazo de una solicitud pendiente para unirse a un grupo|
|Resource|Renovación de grupo|
|Resource|Solicitud de unión a un grupo|
|Resource|Establecimiento de propiedades de grupo dinámico|
|Resource|Actualización de directiva de administración del ciclo de vida|
|Resource|Incorporación de una clave basada en un secreto ASCII a un contenedor de claves CPIM|
|Resource|Incorporación de una clave a un contenedor de claves CPIM|
|Resource|Eliminación de un contenedor de claves CPIM|
|Resource|Eliminación de un contenedor de claves|
|Resource|Obtención de metadatos de clave activos del contenedor de claves en JWK|
|Resource|Obtención de metadatos del contenedor de claves|
|Resource|Obtiene un contenedor de claves CPIM en formato jwk|
|Resource|Obtiene una lista de contenedores de claves del inquilino|
|Resource|Restauración de una copia de seguridad del contenedor de claves CPIM|
|Resource|Guardar un contenedor de claves|
|Resource|Carga de una clave cifrada CPIM|
|Resource|Emisión de un código de autorización para la aplicación|
|Resource|Emisión de un id_token para la aplicación|


## <a name="core-directory"></a>Core Directory (Directorio principal)

|Categoría de auditoría|Actividad|
|---|---|
|Administración de unidades administrativas|Descarga de un tipo de detección de riesgo único|
|Administración de unidades administrativas|Descarga de administradores y del estado de las notificaciones de ingesta semanales|
|Administración de unidades administrativas|Descarga de todos los tipos de detecciones de riesgo|
|Administración de unidades administrativas|Descarga de las detecciones de riesgo de usuarios gratuitos|
|Administración de unidades administrativas|Descarga de usuarios marcados en riesgo|
|Administración de aplicaciones|Invitaciones por lotes procesadas|
|Administración de aplicaciones|Invitaciones por lotes cargadas|
|Administración de aplicaciones|Incorporación de un propietario a una directiva|
|Administración de aplicaciones|Add policy|
|Administración de aplicaciones|Eliminación de directiva|
|Administración de aplicaciones|Eliminación de credenciales de directiva|
|Administración de aplicaciones|Actualización de directiva|
|Administración de aplicaciones|Establecimiento de la directiva de registro MFA|
|Administración de aplicaciones|Establecimiento de la directiva de riesgo de inicio de sesión|
|Administración de aplicaciones|Establecimiento de la directiva de riesgo de usuario|
|Administración de aplicaciones|Aceptación de términos de uso|
|Administración de aplicaciones|Creación de términos de uso|
|Administración de aplicaciones|Rechazo de términos de uso|
|Administración de aplicaciones|Eliminación de términos de uso|
|Administración de aplicaciones|Edición de términos de uso|
|Administración de aplicaciones|Publicación de los términos de uso|
|Administración de aplicaciones|Cancelación de la publicación de los términos de uso|
|Administración de aplicaciones|Adición de certificado TLS/SSL de la aplicación|
|Administración de aplicaciones|Eliminación del enlace TLS|
|Administración de aplicaciones|Registro del conector|
|Administración de aplicaciones|AdminPolicyDatas-RemoveResources|
|Administración de aplicaciones|AdminPolicyDatas-SetResources|
|Administración de aplicaciones|AdminUserJourneys-GetResources|
|Administración de directorios|AdminUserJourneys-RemoveResources|
|Administración de directorios|AdminUserJourneys-SetResources|
|Administración de directorios|Creación de IdentityProvider|
|Administración de directorios|Creación de un nuevo AdminUserJourney|
|Administración de directorios|Creación de un json de recurso localizado|
|Administración de directorios|Creación de un nuevo IDP personalizado|
|Administración de directorios|Creación de nuevo IDP|
|Administración de directorios|Creación o actualización de un recurso del directorio B2C|
|Administración de directorios|Creación de directiva|
|Administración de directorios|Creación de directiva de trustFramework|
|Administración de directorios|Creación de directiva de trustFramework con prefijo configurable|
|Administración de directorios|Creación de atributo de usuario|
|Administración de directorios|CreateTrustFrameworkPolicy|
|Administración de directorios|Eliminación de IDP|
|Administración de directorios|Eliminación de IdentityProvider|
|Administración de directorios|Eliminación de un recurso del directorio B2C|
|Administración de directorios|Eliminación de directiva de trustFramework|
|Administración de directorios|Eliminación de atributo de usuario|
|Administración de directorios|Obtención de recursos de directorio B2C en un grupo de recursos|
|Administración de directorios|Obtención de recursos de directorio B2C en una suscripción|
|Administración de directorios|Obtención de IDP personalizado|
|Administración de directorios|Obtención de IDP|
|Administración de directorios|Obtención del recurso del directorio B2C|
|Administración de directorios|Obtención de un recorrido del usuario|
|Administración de directorios|Obtención de notificaciones de aplicación permitidas para el recorrido del usuario|
|Administración de directorios|Obtención de notificaciones autoafirmadas permitidas para el recorrido del usuario|
|Administración de directorios|Obtención de notificaciones autoafirmadas permitidas de directiva|
|Administración de directorios|Obtención de la lista de notificaciones de salida disponibles|
|Administración de directorios|Obtención de definiciones de contenido para el recorrido del usuario|
|Administración de directorios|Obtención de IDP para un flujo de administración específico|
|Administración de directorios|Obtención de una lista de todos los flujos de administración|
|Administración de directorios|Obtención de lista de etiquetas para todos los flujos de administración de todos los usuarios|
|Administración de grupos|Descarga masiva de miembros del grupo - Iniciada|
|Administración de grupos|Descarga masiva de miembros del grupo - Terminada|
|Administración de grupos|Importación masiva de miembros del grupo - Iniciada|
|Administración de grupos|Importación masiva de miembros del grupo - Terminada|
|Administración de grupos|Quitar en masa a miembros del grupo - Iniciado|
|Administración de grupos|Quitar en masa a miembros del grupo - Terminado|
|Administración de grupos|Descarga masiva del grupo - Iniciada|
|Administración de grupos|Descarga masiva del grupo - Terminada|
|Administración de grupos|Obtención de la lista de inquilinos de un usuario|
|Administración de grupos|Obtención de notificaciones autoafirmadas de las cuentas locales|
|Administración de grupos|Obtención de un json de recurso localizado|
|Administración de grupos|Obtención de operaciones del proveedor de recursos Microsoft.AzureActiveDirectory|
|Administración de grupos|Obtención de directivas|
|Administración de grupos|Obtención de directiva|
|Administración de grupos|Obtención de una lista de IDP compatible|
|Administración de grupos|Obtención de lista de IDP compatible del recorrido del usuario|
|Administración de grupos|Obtención de una lista de IDP personalizada definida por el inquilino|
|Administración de grupos|Obtención de una lista de IDP definida por el inquilino|
|Administración de grupos|Obtención de una lista de IDP local definida por el inquilino|
|Administración de grupos|Obtención de los detalles de inquilino de un usuario para la creación de recursos|
|Administración de grupos|Obtención de la referencia cultural predeterminada para CPIM|
|Administración de grupos|Obtención de los detalles de un flujo de administración|
|Administración de grupos|Obtención de la lista de UserJourneys para este inquilino|
|Administración de grupos|Obtención del conjunto de referencias culturales admitidas disponibles para CPIM|
|Administración de grupos|Obtención de directiva de trustFramework|
|Administración de grupos|Obtención de directiva de trustFramework como xml|
|Administración de grupos|Obtención de atributo de usuario|
|Administración de directivas|Obtención de atributos de usuario|
|Administración de directivas|Obtención de lista de recorridos del usuario|
|Administración de directivas|GetIEFPolicies|
|Administración de directivas|GetIdentityProviders|
|Administración de directivas|GetTrustFrameworkPolicy|
|Resource|MigrateTenantMetadata|
|Resource|Traslado de recursos|
|Resource|Revisión de IdentityProvider|
|Resource|PutTrustFrameworkPolicy|
|Resource|PutTrustFrameworkpolicy|
|Resource|Eliminación de un recorrido del usuario|
|Resource|Actualización de IDP personalizado|
|Resource|Actualización de IDP|
|Resource|Actualización de IDP local|
|Resource|Actualización de un recurso del directorio B2C|
|Resource|Actualización de directiva|
|Resource|Actualización del estado de la suscripción|
|Administración de roles|Actualización de atributo de usuario|
|Administración de roles|Validación de los recursos en movimiento|
|Administración de roles|Incorporación de un dispositivo|
|Administración de roles|Adición de la configuración de dispositivo|
|Administración de roles|Incorporación de propietario registrado a dispositivo|
|Administración de roles|Incorporación de usuarios registrados a dispositivo|
|Administración de roles|Eliminar un dispositivo|
|Administración de roles|Eliminación de la configuración del dispositivo|
|Administración de roles|El dispositivo ya no es compatible|
|Administración de roles|El dispositivo ya no está administrado|
|User Management|AccessReview_Review|
|User Management|AccessReview_Update|
|User Management|ActivationAborted|
|User Management|ActivationApproved|
|User Management|ActivationCanceled|
|User Management|ActivationRequested|
|User Management|Incorporación de un miembro apto al rol|
|User Management|Incorporación de un miembro a un rol|
|User Management|Incorporación de asignación de roles a definición de roles|
|User Management|Incorporación de rol desde plantilla|
|User Management|Incorporación de miembro con ámbito agregado a rol|
|User Management|Se agregó|
|User Management|Assign|
|User Management|Creación masiva de usuarios - Iniciada|
|User Management|Creación masiva de usuarios - Terminada|
|User Management|Eliminación masiva de usuarios - Iniciada|
|User Management|Eliminación masiva de usuarios - Terminada|
|User Management|Descarga masiva de usuarios - Iniciada|
|User Management|Descarga masiva de usuarios - Terminada|
|User Management|Restauración masiva de usuarios eliminados - Iniciada|
|User Management|Restauración masiva de usuarios eliminados - Terminada|
|User Management|Invitación de usuario masiva - Iniciada|
|User Management|Invitación de usuario masiva - Terminada|
|User Management|Eliminación de propietario registrado del dispositivo|
|User Management|Eliminación de usuarios registrados del dispositivo|
|User Management|Eliminación de miembros aptos del rol|
|User Management|Eliminación de miembro del rol|
|User Management|Eliminación de la asignación de roles de la definición de roles|
|User Management|Eliminación de miembro con ámbito de rol|
|User Management|Actualización de dispositivo|
|User Management|Actualización de configuración de dispositivo|
|User Management|Actualización de rol|






## <a name="identity-protection"></a>Protección de identidad

|Categoría de auditoría|Actividad|
|---|---|
|Administración de directorios|Elevar|
|Administración de directorios|Quitado|
|Administración de directorios|Cambios en la configuración de roles|
|Otros|ScanAlertsNow|
|Otros|Signup|
|Otros|Reducir los privilegios|
|Otros|UpdateAlertSettings|
|Otros|UpdateCurrentState|
|Administración de directivas|Revisión de acceso finalizada|
|Administración de directivas|Incorporación de aprobador para solicitar la aprobación|
|Administración de directivas|Incorporación de revisor para acceder a la revisión|
|User Management|Aplicación de la revisión de acceso|
|User Management|Creación de revisión de acceso|


## <a name="invited-users"></a>Invited users (Usuarios invitados)

|Categoría de auditoría|Actividad|
|---|---|
|Otros|Creación de aprobación de solicitud|
|Otros|Eliminación de revisión de acceso|
|User Management|Eliminación del revisor en la revisión de acceso|
|User Management|Solicitud de la aplicación de los resultados de la revisión|
|User Management|Solicitud de detención de la revisión|
|User Management|Revisión de la asignación de aplicaciones|
|User Management|Revisión de la pertenencia al grupo|
|User Management|Revisión de la pertenencia al rol Rbac|


## <a name="microsoft-identity-manager-mim"></a>Microsoft Identity Manager (MIM)

|Categoría de auditoría|Actividad|
|---|---|
|Administración de grupos|Revisión de la solicitud de aprobación de la solicitud|
|Administración de grupos|Actualización de la revisión de acceso|
|Administración de grupos|Actualización de la configuración de las notificaciones por correo de revisión de acceso|
|Administración de grupos|Actualización de la configuración de recuento de repeticiones de la revisión de acceso|
|Administración de grupos|Actualización de la configuración de duración de repeticiones de la revisión de acceso en días|
|User Management|Actualización de la configuración de tipo de fin de repeticiones de la revisión de acceso|
|User Management|Actualización de la configuración de tipo de repeticiones de la revisión de acceso|



## <a name="privileged-identity-management"></a>Privileged Identity Management

|Categoría de auditoría|Actividad|
|---|---|
|PIM|ActivationAborted|
|PIM|ActivationApproved|
|PIM|ActivationCanceled|
|PIM|ActivationDenied|
|PIM|ActivationRequested|
|PIM|Se agregó|
|PIM|AddedOutsidePIM|
|PIM|Assign|
|PIM|DismissAlert|
|PIM|Elevar|
|PIM|ReactivateAlert|
|PIM|Quitado|
|PIM|RemovedOutsidePIM|
|PIM|Solicitud de detención de la revisión|
|PIM|Cambios en la configuración de roles|
|PIM|ScanAlertsNow|
|PIM|Signup|
|PIM|Unassign|
|PIM|Reducir los privilegios|
|PIM|UpdateAlertSettings|
|PIM|UpdateCurrentState|


## <a name="self-service-group-management"></a>Self-service group management (Administración de grupos de autoservicio)

|Categoría de auditoría|Actividad|
|---|---|
|Administración de grupos|Restablecer contraseña de usuario|
|Administración de grupos|Restauración de usuario|
|Administración de grupos|Establecer el cambio forzado de la contraseña de usuario|
|Administración de grupos|Establecimiento del administrador de usuarios|
|Administración de grupos|Establecimiento de metadatos de token Oath de usuarios habilitado|
|Administración de grupos|Actualización de la marca de tiempo StsRefreshTokenValidFrom|
|Administración de grupos|Actualizar secretos externos|
|Administración de grupos|Actualizar usuario|
|Administración de grupos|El administrador genera una contraseña temporal|


## <a name="self-service-password-management"></a>Self-service Password Management (Administración de contraseñas de autorservicio)

|Categoría de auditoría|Actividad|
|---|---|
|Administración de directorios|El administrador solicita al usuario que restablezca su contraseña|
|Administración de directorios|Asignación de usuarios externos a la aplicación|
|User Management|El correo electrónico no se ha enviado, el usuario canceló la suscripción|
|User Management|Invitación a usuario externo|
|User Management|Canje de invitación a usuario externo|
|User Management|Creación de inquilino viral|
|User Management|Creación de usuario viral|
|User Management|Registro de contraseña de usuario|
|User Management|Restablecimiento de contraseña de usuario|
|User Management|Bloqueado para el restablecimiento de contraseña de autoservicio|


## <a name="terms-of-use"></a>Términos de uso

|Categoría de auditoría|Actividad|
|---|---|
|Términos de uso|Aceptación de términos de uso|
|Términos de uso|Creación de términos de uso|
|Términos de uso|Rechazo de términos de uso|
|Términos de uso|Eliminación de consentimiento|
|Términos de uso|Eliminación de términos de uso|
|Términos de uso|Edición de términos de uso|
|Términos de uso|Expiración de los términos de uso|
|Términos de uso|Eliminación permanente de los términos de uso|
|Términos de uso|Publicación de los términos de uso|
|Términos de uso|Cancelación de la publicación de los términos de uso|


## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre informes de Azure AD](overview-reports.md).
- [Informe de registros de auditoría](concept-audit-logs.md) 
- [Acceso mediante programación a los informes de Azure AD](concept-reporting-api.md)