---
title: Solución de problemas de dispositivos híbridos unidos a Azure Active Directory
description: Solución de problemas de dispositivos híbridos de Windows 10 y Windows Server 2016 unidos a Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jairoc
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: bc926c385aeee40601c00b3b4ab68065a4260f2f
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268781"
---
# <a name="troubleshooting-hybrid-azure-active-directory-joined-devices"></a>Solución de problemas de dispositivos híbridos unidos a Azure Active Directory

El contenido de este artículo se aplica a los dispositivos que ejecutan Windows 10 o Windows Server 2016.

Para otros clientes Windows, consulte el artículo [Solución de problemas de dispositivos híbridos de nivel inferior unidos a Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md).

En este artículo se da por supuesto que [configuró dispositivos híbridos unidos a Azure Active Directory](hybrid-azuread-join-plan.md) para que admitan los escenarios siguientes:

- Acceso condicional basado en dispositivos
- [Perfiles móviles de empresa](./enterprise-state-roaming-overview.md)
- [Windows Hello para empresas](/windows/security/identity-protection/hello-for-business/hello-identity-verification)

En este documento se proporcionan instrucciones sobre la solución de posibles problemas.

Para Windows 10 y Windows Server 2016, la unión híbrida a Azure Active Directory es compatible con la actualización de noviembre de 2015 de Windows 10 y versiones posteriores.

## <a name="troubleshoot-join-failures"></a>Solución de errores de unión

### <a name="step-1-retrieve-the-join-status"></a>Paso 1: Recuperación del estado de unión

**Paso 1: Recuperación del estado de unión:**

1. Abra un símbolo del sistema como administrador
2. Escriba `dsregcmd /status`

```
+----------------------------------------------------------------------+
| Device State                                                         |
+----------------------------------------------------------------------+

    AzureAdJoined: YES
 EnterpriseJoined: NO
         DeviceId: 5820fbe9-60c8-43b0-bb11-44aee233e4e7
       Thumbprint: B753A6679CE720451921302CA873794D94C6204A
   KeyContainerId: bae6a60b-1d2f-4d2a-a298-33385f6d05e9
      KeyProvider: Microsoft Platform Crypto Provider
     TpmProtected: YES
     KeySignTest: : MUST Run elevated to test.
              Idp: login.windows.net
         TenantId: 72b988bf-xxxx-xxxx-xxxx-2d7cd011xxxx
       TenantName: Contoso
      AuthCodeUrl: https://login.microsoftonline.com/msitsupp.microsoft.com/oauth2/authorize
   AccessTokenUrl: https://login.microsoftonline.com/msitsupp.microsoft.com/oauth2/token
           MdmUrl: https://enrollment.manage-beta.microsoft.com/EnrollmentServer/Discovery.svc
        MdmTouUrl: https://portal.manage-beta.microsoft.com/TermsOfUse.aspx
  dmComplianceUrl: https://portal.manage-beta.microsoft.com/?portalAction=Compliance
      SettingsUrl: eyJVcmlzIjpbImh0dHBzOi8va2FpbGFuaS5vbmUubWljcm9zb2Z0LmNvbS8iLCJodHRwczovL2thaWxhbmkxLm9uZS5taWNyb3NvZnQuY29tLyJdfQ==
   JoinSrvVersion: 1.0
       JoinSrvUrl: https://enterpriseregistration.windows.net/EnrollmentServer/device/
        JoinSrvId: urn:ms-drs:enterpriseregistration.windows.net
    KeySrvVersion: 1.0
        KeySrvUrl: https://enterpriseregistration.windows.net/EnrollmentServer/key/
         KeySrvId: urn:ms-drs:enterpriseregistration.windows.net
     DomainJoined: YES
       DomainName: CONTOSO

+----------------------------------------------------------------------+
| User State                                                           |
+----------------------------------------------------------------------+

             NgcSet: YES
           NgcKeyId: {C7A9AEDC-780E-4FDA-B200-1AE15561A46B}
    WorkplaceJoined: NO
      WamDefaultSet: YES
WamDefaultAuthority: organizations
       WamDefaultId: https://login.microsoft.com
     WamDefaultGUID: {B16898C6-A148-4967-9171-64D755DA8520} (AzureAd)
         AzureAdPrt: YES
```

### <a name="step-2-evaluate-the-join-status"></a>Paso 2: Evaluación del estado de unión

Revise los siguientes campos y asegúrese de que tengan los valores esperados:

#### <a name="domainjoined--yes"></a>DomainJoined: SÍ

Este campo indica si el dispositivo está unido o no a una implementación local de Active Directory. Si el valor es **NO**, el dispositivo no puede realizar una unión a Azure AD híbrido.

#### <a name="workplacejoined--no"></a>WorkplaceJoined: No

Este campo indica si el dispositivo está registrado con Azure AD, pero como un dispositivo personal (marcado como *Unido al área de trabajo*). Este valor debe ser **NO** para un equipo unido a un dominio que esté también unido a Azure AD híbrido. Si el valor es **SÍ**, se agrega una cuenta profesional o educativa antes de la finalización de la unión a Azure AD híbrido. En este caso, la cuenta se omite cuando se usa la versión de actualización de aniversario de Windows 10 (1607).

#### <a name="azureadjoined--yes"></a>AzureAdJoined : SÍ

Este campo indica si el dispositivo está unido. El valor será **SÍ** si el dispositivo es un dispositivo unido a un Azure AD o un dispositivo híbrido de Azure AD Unido.
Si el valor es **NO**, aún no ha finalizado la unión a Azure AD.

Continúe con los pasos siguientes para más información.

### <a name="step-3-find-the-phase-in-which-join-failed-and-the-errorcode"></a>Paso 3: Busque la fase en la que se produjo el error de unión y el código de error

#### <a name="windows-10-1803-and-above"></a>Windows 10 1803 y versiones posteriores

Busque el subapartado "Previous Registration" (Registro previo) en el apartado "Diagnostic Data" (Datos de diagnóstico) de la salida de estado de la unión. Esta sección solo se muestra si el dispositivo está unido a un dominio y no puede realizar una unión a Azure AD híbrido.
El campo "Error Phase" (Fase del error) denota la fase del error de unión y "Client ErrorCode" (Código de error del cliente), el código de error de la operación de unión.

```
+----------------------------------------------------------------------+
     Previous Registration : 2019-01-31 09:16:43.000 UTC
         Registration Type : sync
               Error Phase : join
          Client ErrorCode : 0x801c03f2
          Server ErrorCode : DirectoryError
            Server Message : The device object by the given id (e92325d0-xxxx-xxxx-xxxx-94ae875d5245) is not found.
              Https Status : 400
                Request Id : 6bff0bd9-820b-484b-ab20-2a4f7b76c58e
+----------------------------------------------------------------------+
```

#### <a name="older-windows-10-versions"></a>Versiones anteriores a Windows 10

Use registros de Visor de eventos para buscar la fase y el código de error para los errores de unión.

1. Abra los registros de eventos de **User Device Registration** (Registro de dispositivos del usuario) en el Visor de eventos; que se encuentran en **Applications and Services Log** > **Microsoft** > **Windows** > **User Device Registration** (Registros de aplicaciones y servicios > Microsoft > Windows > Registro de dispositivos del usuario)
2. Busque eventos con los siguientes identificadores de eventos: 304, 305, 307.

![Evento del registro de errores](./media/troubleshoot-hybrid-join-windows-current/1.png)

![Evento del registro de errores](./media/troubleshoot-hybrid-join-windows-current/2.png)

### <a name="step-4-check-for-possible-causes-and-resolutions-from-the-lists-below"></a>Paso 4: Busque posibles causas y soluciones en las siguientes listas

#### <a name="pre-check-phase"></a>Fase de comprobación previa

Posibles motivos del error:

- El dispositivo no tiene línea de visión al controlador de dominio.
   - El dispositivo debe encontrarse en la red interna de la organización o en una VPN con línea de visión de red a una implementación local del controlador de dominio de Active Directory (AD).

#### <a name="discover-phase"></a>Fase de detección

Posibles motivos del error:

- El objeto de punto de conexión de servicio (SCP) está mal configurado o no se puede leer el objeto SCP desde el controlador de dominio.
   - Se requiere un objeto SCP válido en el bosque de AD al que pertenece el dispositivo que apunte a un nombre de dominio comprobado en Azure AD.
   - Encontrará más información en la sección sobre la [Configuración de un punto de conexión de servicio](hybrid-azuread-join-federated-domains.md#configure-hybrid-azure-ad-join).
- Error al conectar y capturar los metadatos de detección del punto de conexión de detección.
   - El dispositivo debe poder acceder a `https://enterpriseregistration.windows.net` en el contexto del sistema para detectar los puntos de conexión de autorización y registro.
   - Si el entorno local requiere un proxy de salida, el administrador de TI debe asegurarse de que la cuenta del equipo del dispositivo detecte el proxy de salida y se autentique en él en modo silencioso.
- No se pudo conectar al punto de conexión del dominio Kerberos del usuario ni detectarlo (solo Windows 10 versión 1809 y versiones posteriores).
   - El dispositivo debe tener acceso a `https://login.microsoftonline.com` en el contexto del sistema para detectar dominio para el dominio Kerberos del dominio comprobado y determinar el tipo de dominio (administrado o federado).
   - Si el entorno local requiere un proxy de salida, el administrador de TI debe asegurarse de que el contexto del sistema del dispositivo detecte el proxy de salida y se autentique en él en modo silencioso.

**Códigos de error comunes**:

- **DSREG_AUTOJOIN_ADCONFIG_READ_FAILED** (0x801c001d/-2145648611)
   - Motivo: No se puede leer el objeto SCP y obtener la información del inquilino de Azure AD.
   - Resolución: Consulte la sección sobre la [Configuración de un punto de conexión de servicio](hybrid-azuread-join-federated-domains.md#configure-hybrid-azure-ad-join).
- **DSREG_AUTOJOIN_DISC_FAILED** (0x801c0021/-2145648607)
   - Motivo: Error de detección general. No se pudieron obtener los metadatos de detección de DRS.
   - Resolución: Busque el suberror a continuación para investigar más.
- **DSREG_AUTOJOIN_DISC_WAIT_TIMEOUT** (0x801c001f/-2145648609)
   - Motivo: Se agotó el tiempo de espera de la operación al realizar la detección.
   - Resolución: Asegúrese de que `https://enterpriseregistration.windows.net` sea accesible en el contexto del sistema. Para más información, consulte la sección sobre los [Requisitos de conectividad de red](hybrid-azuread-join-managed-domains.md#prerequisites).
- **DSREG_AUTOJOIN_USERREALM_DISCOVERY_FAILED** (0x801c0021/-2145648611)
   - Motivo: Error de detección de dominio Kerberos genérico. No se pudo determinar el tipo de dominio (administrado o federado) desde STS.
   - Resolución: Busque el suberror a continuación para investigar más.

**Códigos de suberror comunes**:

Para buscar el código de suberror del código de error de detección, use uno de los métodos siguientes.

##### <a name="windows-10-1803-and-above"></a>Windows 10 1803 y versiones posteriores

Busque el subapartado "DRS Discovery Test" (Prueba de detección de DRS) en el apartado "Diagnostic Data" (Datos de diagnóstico) de la salida de estado de la unión. Esta sección solo se muestra si el dispositivo está unido a un dominio y no puede realizar una unión a Azure AD híbrido.

```
+----------------------------------------------------------------------+
| Diagnostic Data                                                      |
+----------------------------------------------------------------------+

     Diagnostics Reference : www.microsoft.com/aadjerrors
              User Context : UN-ELEVATED User
               Client Time : 2019-06-05 08:25:29.000 UTC
      AD Connectivity Test : PASS
     AD Configuration Test : PASS
        DRS Discovery Test : FAIL [0x801c0021/0x80072ee2]
     DRS Connectivity Test : SKIPPED
    Token acquisition Test : SKIPPED
     Fallback to Sync-Join : ENABLED

+----------------------------------------------------------------------+
```

##### <a name="older-windows-10-versions"></a>Versiones anteriores a Windows 10

Use registros de Visor de eventos para buscar la fase y el código de error para los errores de unión.

1. Abra los registros de eventos de **User Device Registration** (Registro de dispositivos del usuario) en el Visor de eventos; que se encuentran en **Applications and Services Log** > **Microsoft** > **Windows** > **User Device Registration** (Registros de aplicaciones y servicios > Microsoft > Windows > Registro de dispositivos del usuario)
2. Busque eventos con los siguientes identificadores de eventos: 201.

![Evento del registro de errores](./media/troubleshoot-hybrid-join-windows-current/5.png)

###### <a name="network-errors"></a>Errores de red

- **WININET_E_CANNOT_CONNECT** (0x80072efd/-2147012867)
   - Motivo: No se pudo establecer una conexión con el servidor.
   - Resolución: Garantice la conectividad de red con los recursos de Microsoft necesarios. Para más información, consulte la sección sobre los [Requisitos de conectividad de red](hybrid-azuread-join-managed-domains.md#prerequisites).
- **WININET_E_TIMEOUT** (0x80072ee2/-2147012894)
   - Motivo: Tiempo de expiración de red general.
   - Resolución: Garantice la conectividad de red con los recursos de Microsoft necesarios. Para más información, consulte la sección sobre los [Requisitos de conectividad de red](hybrid-azuread-join-managed-domains.md#prerequisites).
- **WININET_E_DECODING_FAILED** (0x80072f8f/-2147012721)
   - Motivo: La pila de red no pudo descodificar la respuesta del servidor.
   - Resolución: Asegúrese de que el proxy de red no interfiera y modifique la respuesta del servidor.

###### <a name="http-errors"></a>Errores HTTP

- **DSREG_DISCOVERY_TENANT_NOT_FOUND** (0x801c003a/-2145648582)
   - Motivo: Objeto SCP configurado con un identificador de inquilino incorrecto. O no se encontraron suscripciones activas en el inquilino.
   - Resolución: Asegúrese de que el objeto SCP está configurado con el identificador de inquilino de Azure AD correcto y suscripciones activas en el inquilino.
- **DSREG_SERVER_BUSY** (0x801c0025/-2145648603)
   - Motivo: HTTP 503 desde el servidor DRS.
   - Resolución: El servidor no está disponible actualmente. Los intentos futuros de unión probablemente se realizarán correctamente una vez que el servidor vuelva a estar en línea.

###### <a name="other-errors"></a>Otros errores

- **E_INVALIDDATA** (0x8007000d/-2147024883)
   - Motivo: No se pudo analizar el JSON de respuesta del servidor. Probablemente se deba a que el proxy devuelve HTTP 200 con una página de autenticación HTML.
   - Resolución: Si el entorno local requiere un proxy de salida, el administrador de TI debe asegurarse de que el contexto del sistema del dispositivo detecte el proxy de salida y se autentique en él en modo silencioso.

#### <a name="authentication-phase"></a>Fase de autenticación

Solo se aplica a las cuentas de dominio federado.

Motivos del error:

- No se puede obtener un token de acceso en modo silencioso para el recurso de DRS.
   - Los dispositivos de Windows 10 adquieren el token de autenticación del servicio de federación mediante la autenticación integrada de Windows en un punto de conexión activo de WS-Trust. Detalles: [Configuración del servicio de federación](hybrid-azuread-join-manual.md#set-up-issuance-of-claims)

**Códigos de error comunes**:

Use los registros de Visor de eventos para buscar el código de error, el código de suberror, el código de error del servidor y el mensaje de error del servidor.

1. Abra los registros de eventos de **User Device Registration** (Registro de dispositivos del usuario) en el Visor de eventos; que se encuentran en **Applications and Services Log** > **Microsoft** > **Windows** > **User Device Registration** (Registros de aplicaciones y servicios > Microsoft > Windows > Registro de dispositivos del usuario)
2. Busque eventos con el siguiente identificador de eventos: 305.

![Evento del registro de errores](./media/troubleshoot-hybrid-join-windows-current/3.png)

##### <a name="configuration-errors"></a>Errores de configuración

- **ERROR_ADAL_PROTOCOL_NOT_SUPPORTED** (0xcaa90017/-894894057)
   - Motivo: El protocolo de autenticación no es WS-Trust.
   - Resolución: El proveedor de identidades local debe ser compatible con WS-Trust.
- **ERROR_ADAL_FAILED_TO_PARSE_XML** (0xcaa9002c/-894894036)
   - Motivo: El servicio de federación local no devolvió una respuesta XML.
   - Resolución: Asegúrese de que el punto de conexión MEX devuelve un XML válido. Asegúrese de que el proxy no interfiera y que devuelva respuestas que no sean XML.
- **ERROR_ADAL_COULDNOT_DISCOVER_USERNAME_PASSWORD_ENDPOINT** (0xcaa90023/-894894045)
   - Motivo: No se pudo detectar el punto de conexión para la autenticación de nombre de usuario/contraseña.
   - Resolución: Compruebe la configuración del proveedor de identidades local. Asegúrese de que los puntos de conexión de WS-Trust estén habilitados y de que la respuesta MEX contiene estos puntos de conexión correctos.

##### <a name="network-errors"></a>Errores de red

- **ERROR_ADAL_INTERNET_TIMEOUT** (0xcaa82ee2/-894947614)
   - Motivo: Tiempo de expiración de red general.
   - Resolución: Asegúrese de que `https://login.microsoftonline.com` sea accesible en el contexto del sistema. Asegúrese de que el proveedor de identidades local esté accesible en el contexto del sistema. Para más información, consulte la sección sobre los [Requisitos de conectividad de red](hybrid-azuread-join-managed-domains.md#prerequisites).
- **ERROR_ADAL_INTERNET_CONNECTION_ABORTED** (0xcaa82efe/-894947586)
   - Motivo: Se anuló la conexión con el punto de conexión de autenticación.
   - Resolución: Vuelva a intentarlo más tarde o intente conectarse desde una ubicación de red estable alternativa.
- **ERROR_ADAL_INTERNET_SECURE_FAILURE** (0xcaa82f8f/-894947441)
   - Motivo: No se pudo validar el certificado de Seguridad de la capa de transporte (TLS), antes Capa de sockets seguros (SSL), enviado por el servidor.
   - Resolución: Compruebe el sesgo de tiempo del cliente. Vuelva a intentarlo más tarde o intente conectarse desde una ubicación de red estable alternativa.
- **ERROR_ADAL_INTERNET_CANNOT_CONNECT** (0xcaa82efd/-894947587)
   - Motivo: Error al intentar conectarse a `https://login.microsoftonline.com`.
   - Resolución: Compruebe la conexión a `https://login.microsoftonline.com`.

##### <a name="other-errors"></a>Otros errores

- **ERROR_ADAL_SERVER_ERROR_INVALID_GRANT** (0xcaa20003/-895352829)
   - Motivo: Azure AD no aceptó el token SAML del proveedor de identidades local.
   - Resolución: Compruebe la configuración del servidor de federación. Busque el código de error del servidor en los registros de autenticación.
- **ERROR_ADAL_WSTRUST_REQUEST_SECURITYTOKEN_FAILED** (0xcaa90014/-894894060)
   - Motivo: La respuesta de WS-Trust del servidor ha comunicado una excepción de error y no se pudo obtener la aserción.
   - Resolución: Compruebe la configuración del servidor de federación. Busque el código de error del servidor en los registros de autenticación.
- **ERROR_ADAL_WSTRUST_TOKEN_REQUEST_FAIL** (0xcaa90006/-894894074)
   - Motivo: Se recibió un error al intentar obtener el token de acceso desde el punto de conexión del token.
   - Resolución: Busque el error subyacente en el registro de ADAL.
- **ERROR_ADAL_OPERATION_PENDING** (0xcaa1002d/-895418323)
   - Motivo: Error general de ADAL.
   - Resolución: Busque el código de suberror o de error del servidor en los registros de autenticación.

#### <a name="join-phase"></a>Fase de unión

Motivos del error:

Busque el tipo de registro y busque el código de error en la siguiente lista.

#### <a name="windows-10-1803-and-above"></a>Windows 10 1803 y versiones posteriores

Busque el subapartado "Previous Registration" (Registro previo) en el apartado "Diagnostic Data" (Datos de diagnóstico) de la salida de estado de la unión. Esta sección solo se muestra si el dispositivo está unido a un dominio y no puede realizar una unión a Azure AD híbrido.
El campo "Registration Type" (Tipo de registro) denota el tipo de unión realizada.

```
+----------------------------------------------------------------------+
     Previous Registration : 2019-01-31 09:16:43.000 UTC
         Registration Type : sync
               Error Phase : join
          Client ErrorCode : 0x801c03f2
          Server ErrorCode : DirectoryError
            Server Message : The device object by the given id (e92325d0-7ac4-4714-88a1-94ae875d5245) is not found.
              Https Status : 400
                Request Id : 6bff0bd9-820b-484b-ab20-2a4f7b76c58e
+----------------------------------------------------------------------+
```

#### <a name="older-windows-10-versions"></a>Versiones anteriores a Windows 10

Use registros de Visor de eventos para buscar la fase y el código de error para los errores de unión.

1. Abra los registros de eventos de **User Device Registration** (Registro de dispositivos del usuario) en el Visor de eventos; que se encuentran en **Applications and Services Log** > **Microsoft** > **Windows** > **User Device Registration** (Registros de aplicaciones y servicios > Microsoft > Windows > Registro de dispositivos del usuario)
2. Busque eventos con el siguiente identificador de eventos: 204.

![Evento del registro de errores](./media/troubleshoot-hybrid-join-windows-current/4.png)

##### <a name="http-errors-returned-from-drs-server"></a>Errores HTTP devueltos por el servidor DRS

- **DSREG_E_DIRECTORY_FAILURE** (0x801c03f2/-2145647630)
   - Motivo: Se recibió una respuesta de error de DRS con el código de error: "DirectoryError"
   - Resolución: Consulte el código de error del servidor para ver posibles causas y soluciones.
- **DSREG_E_DEVICE_AUTHENTICATION_ERROR** (0x801c0002/-2145648638)
   - Motivo: Se recibió una respuesta de error de DRS con el código de error: "AuthenticationError" y ErrorSubCode NO es "DeviceNotFound".
   - Resolución: Consulte el código de error del servidor para ver posibles causas y soluciones.
- **DSREG_E_DEVICE_INTERNALSERVICE_ERROR** (0x801c0006/-2145648634)
   - Motivo: Se recibió una respuesta de error de DRS con el código de error: "DirectoryError"
   - Resolución: Consulte el código de error del servidor para ver posibles causas y soluciones.

##### <a name="tpm-errors"></a>Errores de TPM

- **NTE_BAD_KEYSET** (0x80090016/-2146893802)
   - Motivo: Error en la operación de TPM o esta no es válida.
   - Resolución: Probablemente se debe a una imagen de Sysprep incorrecta. Asegúrese de que la máquina desde la que se creó la imagen de Sysprep no es Azure AD joined, Hybrid Azure AD joined ni Azure AD registered.
- **TPM_E_PCP_INTERNAL_ERROR** (0x80290407/-2144795641)
   - Motivo: Error de TPM genérico.
   - Resolución: Deshabilite TPM en los dispositivos con este error. La versión 1809 de Windows 10 y las posteriores detectan automáticamente los errores de TPM y completan Hybrid Azure AD joined sin usar TPM.
- **TPM_E_NOTFIPS** (0x80280036/-2144862154)
   - Motivo: TPM en modo FIPS no se admite actualmente.
   - Resolución: Deshabilite TPM en los dispositivos con este error. La versión 1809 de Windows detecta automáticamente los errores de TPM y completa Hybrid Azure AD joined sin usar TPM.
- **NTE_AUTHENTICATION_IGNORED** (0x80090031/-2146893775)
   - Motivo: TPM bloqueado.
   - Resolución: Se trata de un error transitorio. Espere el tiempo de enfriamiento. La unión debería poder realizarse correctamente tras un tiempo. Más información en la el artículo de [Conceptos básicos de TPM](/windows/security/information-protection/tpm/tpm-fundamentals#anti-hammering).

##### <a name="network-errors"></a>Errores de red

- **WININET_E_TIMEOUT** (0x80072ee2/-2147012894)
   - Motivo: El tiempo de espera de red general intenta registrar el dispositivo en DRS.
   - Resolución: Compruebe la conectividad de la red a `https://enterpriseregistration.windows.net`.
- **WININET_E_NAME_NOT_RESOLVED** (0x80072ee7/-2147012889)
   - Motivo: No se pudo resolver el nombre o la dirección del servidor.
   - Resolución: Compruebe la conectividad de la red a `https://enterpriseregistration.windows.net`. Asegúrese de que la resolución de DNS para el nombre de host sea precisa en el puerto N/W y en el dispositivo.
- **WININET_E_CONNECTION_ABORTED** (0x80072efe/-2147012866)
   - Motivo: La conexión con el servidor terminó de forma anómala.
   - Resolución: Vuelva a intentarlo más tarde o intente conectarse desde una ubicación de red estable alternativa.

##### <a name="federated-join-server-errors"></a>Errores del servidor de unión federado

| Código de error del servidor | Mensaje de error del servidor | Razones posibles | Solución |
| --- | --- | --- | --- |
| DirectoryError | La solicitud está limitada temporalmente. Vuelva a intentarlo después de 300 segundos. | Error previsto. Posiblemente debido a la realización de varias solicitudes de registro en una sucesión rápida. | Vuelva a intentar la unión tras el tiempo de enfriamiento. |

##### <a name="sync-join-server-errors"></a>Errores del servidor de unión sincronizado

| Código de error del servidor | Mensaje de error del servidor | Razones posibles | Solución |
| --- | --- | --- | --- |
| DirectoryError | AADSTS90002: no se ha encontrado del inquilino <UUID>. Este error puede producirse si no hay suscripciones activas para el inquilino. Compruébelo con el administrador de la suscripción. | El identificador de inquilino en el objeto SCP es incorrecto. | Asegúrese de que el objeto SCP está configurado con el identificador de inquilino de Azure AD correcto y suscripciones activas en el inquilino. |
| DirectoryError | No se encuentra el objeto de dispositivo por el identificador especificado. | Error previsto para la unión de sincronización. El objeto de dispositivo no se ha sincronizado de AD a Azure AD. | Espere a que se complete la sincronización de Azure AD Connect y el siguiente intento de unión después de la finalización de la sincronización resolverá el problema. |
| AuthenticationError | Comprobación del identificador de seguridad del equipo de destino | El certificado del dispositivo de Azure AD no coincide con el certificado usado para firmar el blob durante la unión de sincronización. Este error normalmente significa que la sincronización no ha terminado aún. |  Espere a que se complete la sincronización de Azure AD Connect y el siguiente intento de unión después de la finalización de la sincronización resolverá el problema. |

### <a name="step-5-collect-logs-and-contact-microsoft-support"></a>Paso 5: Recopilación de los registros y contacto con el soporte técnico de Microsoft

Descargue el archivo Auth.zip desde [https://github.com/CSS-Windows/WindowsDiag/tree/master/ADS/AUTH](https://github.com/CSS-Windows/WindowsDiag/tree/master/ADS/AUTH).

1. Descomprima los archivos y cambie el nombre de los archivos incluidos **start-auth.txt** y **stop-auth.txt** a **start-auth.cmd** y **stop-auth.cmd**.
1. En un símbolo del sistema con privilegios elevados, ejecute **start-auth.cmd**.
1. Use Cambiar de cuenta para ir a otra sesión con el usuario problemático.
1. Reproduzca el problema.
1. Use Cambiar de cuenta para volver a la sesión de administrador que ejecuta el seguimiento.
1. En un símbolo del sistema con privilegios elevados, ejecute **stop-auth.cmd**.
1. Comprima y envíe la carpeta **Authlogs** de la carpeta en la que se han ejecutado los scripts.

## <a name="troubleshoot-post-join-issues"></a>Solución de problemas tras la unión

### <a name="retrieve-the-join-status"></a>Recuperación del estado de unión

#### <a name="wamdefaultset-yes-and-azureadprt-yes"></a>WamDefaultSet: YES y AzureADPrt: SÍ

Estos campos indican que el usuario se ha autenticado correctamente en Azure AD al iniciar sesión en el dispositivo.
Si los valores son **NO**, podría ser debido:

- Clave de almacenamiento incorrecta en TPM asociada con el dispositivo tras el registro (compruebe KeySignTest mientras se ejecuta con privilegios elevados).
- Id. de inicio de sesión alternativo
- No se ha encontrado el proxy HTTP

## <a name="known-issues"></a>Problemas conocidos
- En Configuración-> Cuentas-> Access Work or School, Hybrid Azure AD joined devices (Acceder a dispositivos profesionales o educativos, híbridos unidos a Azure AD) se pueden mostrar dos cuentas diferentes, una para Azure AD y otra para AD local, cuando se conecta a zonas activas móviles o a redes Wi-Fi externas. Esto es solo un problema de la interfaz de usuario y no afecta a la funcionalidad.

## <a name="next-steps"></a>Pasos siguientes

Continúe la [solución de problemas de dispositivos con el comando dsregcmd](troubleshoot-device-dsregcmd.md)

Si tiene preguntas, consulte las [preguntas más frecuentes sobre la administración de dispositivos](faq.md).