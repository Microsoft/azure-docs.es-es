---
title: 'Autenticación: Herramienta de modelado de amenazas de Microsoft (Azure) | Microsoft Docs'
description: Más información sobre la mitigación de la autenticación en Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: has-adal-ref, devx-track-javascript
ms.openlocfilehash: 11158749f3cae222c0948286e7365b1629cebbf2
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543986"
---
# <a name="security-frame-authentication--mitigations"></a>Marco de seguridad: autenticación | Mitigaciones

| Producto o servicio | Artículo |
| --------------- | ------- |
| **Aplicación web**    | <ul><li>[Consideración sobre el uso de un mecanismo de autenticación estándar para autenticarse en la aplicación web](#standard-authn-web-app)</li><li>[Las aplicaciones deben administrar escenarios de errores de autenticación de forma segura](#handle-failed-authn)</li><li>[Habilitación de la autenticación adicional o adaptable](#step-up-adaptive-authn)</li><li>[Asegurarse de que las interfaces administrativas estén correctamente bloqueadas](#admin-interface-lockdown)</li><li>[Implementación de funcionalidades de contraseña olvidada de forma segura](#forgot-pword-fxn)</li><li>[Asegurarse de que se implementen directivas de cuenta y contraseña](#pword-account-policy)</li><li>[Implementación de controles para impedir la enumeración de nombres de usuario](#controls-username-enum)</li></ul> |
| **Base de datos** | <ul><li>[Cuando sea posible, usar la autenticación de Windows para conectarse a SQL Server](#win-authn-sql)</li><li>[Cuando sea posible, usar la autenticación de Azure Active Directory para conectarse a SQL Database](#aad-authn-sql)</li><li>[Cuando se use el modo de autenticación de SQL, asegurarse de que se apliquen directivas de cuenta y contraseña en SQL Server](#authn-account-pword)</li><li>[No usar la autenticación SQL en bases de datos independientes](#autn-contained-db)</li></ul> |
| **Centro de eventos de Azure** | <ul><li>[Uso de credenciales de autenticación por dispositivo mediante tokens SaS](#authn-sas-tokens)</li></ul> |
| **Límites de confianza de Azure** | <ul><li>[Habilitación de Azure Multi-Factor Authentication para administradores de Azure](#multi-factor-azure-admin)</li></ul> |
| **Límites de confianza de Service Fabric** | <ul><li>[Restricción del acceso anónimo al clúster de Service Fabric](#anon-access-cluster)</li><li>[Asegurarse de que el certificado de cliente a nodo de Service Fabric sea diferente del certificado de nodo a nodo](#fabric-cn-nn)</li><li>[Uso de AAD para autenticar a los clientes en los clústeres de Service Fabric](#aad-client-fabric)</li><li>[Asegurarse de que los certificados de Service Fabric se obtienen de una entidad de certificación (CA) aprobada](#fabric-cert-ca)</li></ul> |
| **Identity Server** | <ul><li>[Uso de escenarios de autenticación estándar admitidos por Identity Server](#standard-authn-id)</li><li>[Reemplazo de la caché de tokens predeterminada de Identity Server por una alternativa escalable](#override-token)</li></ul> |
| **Límite de confianza de la máquina** | <ul><li>[Asegurarse de que los archivos binarios de la aplicación implementada están firmados digitalmente](#binaries-signed)</li></ul> |
| **WCF** | <ul><li>[Habilitación de la autenticación al conectarse a las colas de MSMQ en WCF](#msmq-queues)</li><li>[No establecer clientCredentialType de WCF en Ninguno](#message-none)</li><li>[No establecer clientCredentialType de WCF en Ninguno](#transport-none)</li></ul> |
| **API web** | <ul><li>[Asegurarse de que se usan técnicas de autenticación estándar para proteger las API web](#authn-secure-api)</li></ul> |
| **Azure AD** | <ul><li>[Uso de escenarios de autenticación estándar compatibles con Azure Active Directory](#authn-aad)</li><li>[Reemplazo de la caché de tokens predeterminada de ADAL por una alternativa escalable](#adal-scalable)</li><li>[Asegurarse de que TokenReplayCache se usa para impedir la reproducción de tokens de autenticación de ADAL](#tokenreplaycache-adal)</li><li>[Uso de bibliotecas de ADAL para administrar las solicitudes de token de clientes de OAuth2 a AAD (o AD local)](#adal-oauth2)</li></ul> |
| **Puerta de enlace de campo de IoT** | <ul><li>[Autenticación de dispositivos que se conectan a la puerta de enlace de campo](#authn-devices-field)</li></ul> |
| **Puerta de enlace de nube de IoT** | <ul><li>[Asegurarse de que se autentican los dispositivos que se conectan a la puerta de enlace de nube](#authn-devices-cloud)</li><li>[Uso de credenciales de autenticación por dispositivo](#authn-cred)</li></ul> |
| **Almacenamiento de Azure** | <ul><li>[Asegurarse de que solo los contenedores y blobs requeridos reciben acceso anónimo de lectura](#req-containers-anon)</li><li>[Concesión de acceso limitado a objetos de Azure Storage mediante SAS o SAP](#limited-access-sas)</li></ul> |

## <a name="consider-using-a-standard-authentication-mechanism-to-authenticate-to-web-application"></a><a id="standard-authn-web-app"></a>Consideración sobre el uso de un mecanismo de autenticación estándar para autenticarse en la aplicación web

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| Detalles | <p>La autenticación es el proceso donde una entidad demuestra su identidad, normalmente por medio de credenciales, como un nombre de usuario y una contraseña. Hay varios protocolos de autenticación disponibles que se pueden considerar. A continuación se enumeran algunos de ellos:</p><ul><li>Certificados de cliente</li><li>Basado en Windows</li><li>Basado en formularios</li><li>Federación - ADFS</li><li>Federación - Azure AD</li><li>Federación - Identity Server</li></ul><p>Considere el uso de un mecanismo de autenticación estándar para identificar el proceso de origen.</p>|

## <a name="applications-must-handle-failed-authentication-scenarios-securely"></a><a id="handle-failed-authn"></a>Las aplicaciones deben administrar escenarios de errores de autenticación de forma segura

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| Detalles | <p>Las aplicaciones que explícitamente autentican a los usuarios deben administrar los escenarios de errores de autenticación de forma segura. En este sentido, el mecanismo de autenticación debe:</p><ul><li>Denegar el acceso a recursos con privilegios cuando se produzca un error de autenticación.</li><li>Mostrar un mensaje de error genérico después de que se produzca el error de autenticación y la denegación del acceso.</li></ul><p>Deberá comprobar:</p><ul><li>La protección de recursos con privilegios después de errores de inicios de sesión.</li><li>Se muestra un mensaje de error genérico en los eventos de error de autenticación y denegación del acceso.</li><li>Las cuentas se deshabilitan después de un número excesivo de intentos erróneos.</li><ul>|

## <a name="enable-step-up-or-adaptive-authentication"></a><a id="step-up-adaptive-authn"></a>Habilitación de la autenticación adicional o adaptable

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| Detalles | <p>Compruebe que la aplicación tenga autorización adicional (como, autenticación adicional o adaptable, mediante autenticación multifactor, como el envío de una contraseña de un solo uso (OTP) en mensajes SMS, correo electrónico etc., o la solicitud de una nueva autenticación), de forma que sea difícil para el usuario obtener acceso a información confidencial. Esta regla también se aplica a la hora de realizar cambios críticos en una cuenta o acción.</p><p>También significa que la adaptación de la autenticación se tiene que implementar de forma tal que la aplicación exija correctamente autorización contextual hasta el punto de no permitir la manipulación sin autorización de, por ejemplo, parámetros.</p>|

## <a name="ensure-that-administrative-interfaces-are-appropriately-locked-down"></a><a id="admin-interface-lockdown"></a>Asegurarse de que las interfaces administrativas estén correctamente bloqueadas

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| Detalles | La primera solución es conceder acceso únicamente desde un determinado intervalo de direcciones IP de origen a la interfaz administrativa. Si esta solución no es posible, siempre se recomienda exigir autenticación adicional o adaptable para iniciar sesión en la interfaz administrativa. |

## <a name="implement-forgot-password-functionalities-securely"></a><a id="forgot-pword-fxn"></a>Implementación de funcionalidades de contraseña olvidada de forma segura

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| Detalles | <p>Lo primero es comprobar que las rutas de recuperación de contraseñas olvidadas y otras rutas de recuperación, envían un vínculo que incluye un token de activación por tiempo limitado en lugar de la contraseña en sí. También puede exigirse autenticación adicional basada en tokens temporales (por ejemplo, token de SMS, aplicaciones móviles nativas, etc.) antes de que se envíe el vínculo. En segundo lugar, no debe bloquear la cuenta de los usuarios mientras el proceso de obtención de una nueva contraseña está en marcha.</p><p>Podría dar lugar a un ataque de denegación de servicio cada vez que un atacante decidiera bloquear intencionadamente a los usuarios con un ataque automatizado. En tercer lugar, siempre que la nueva solicitud de contraseña esté en marcha, el mensaje que se muestra debe generalizarse con el fin de evitar la enumeración de nombres de usuario. En cuarto lugar, nunca permita el uso de contraseñas anteriores e implemente una directiva de contraseñas seguras.</p> |

## <a name="ensure-that-password-and-account-policy-are-implemented"></a><a id="pword-account-policy"></a>Asegurarse de que se implementen directivas de cuenta y contraseña

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| Detalles | <p>Se deben implementar directivas de cuenta y contraseña en conformidad con la política de la organización y los procedimientos recomendados.</p><p>Para defenderse contra ataques por fuerza bruta y adivinación por diccionario, se debe implementar una directiva de contraseñas seguras que garantice que los usuarios crean contraseñas complejas (por ejemplo, con una longitud mínima de doce caracteres, que incluyan alfanuméricos y especiales).</p><p>Se pueden implementar directivas de bloqueo de la cuenta de la siguiente manera:</p><ul><li>**Bloqueo temporal:** puede ser una buena opción para proteger a los usuarios contra ataques por fuerza bruta. Por ejemplo, cada vez que el usuario escribe una contraseña errónea tres veces, la aplicación puede bloquear la cuenta durante un minuto con el fin de ralentizar el proceso de forzar su contraseña de forma que el atacante ceje en su empeño y abandone. Si fuera a implementar contramedidas de bloqueo permanente, en este ejemplo conseguiría un "DoS" al bloquear las cuentas de forma permanente. Como alternativa, puede generar una contraseña de un solo uso (OTP) y enviarla al usuario fuera de banda (por correo electrónico, SMS, etc.). Otro enfoque podría ser implementar CAPTCHA después de alcanzar un umbral de intentos erróneos.</li><li>**Bloqueo permanente:** este tipo de bloqueo se debe aplicar cada vez que detecte que un usuario ataca su aplicación. El contraataque consiste en el bloqueo permanente de su cuenta hasta que el equipo de respuesta tenga tiempo de realizar el análisis forense. Después de este proceso, puede decidir devolver la cuenta al usuario o tomar medidas legales contra él. Este tipo de enfoque impide que el atacante penetre aún más en la aplicación y la infraestructura.</li></ul><p>Para defenderse contra ataques en cuentas predeterminadas y predecibles, compruebe que todas las claves y contraseñas sean reemplazables y que se generen o reemplacen después del tiempo de instalación.</p><p>Si la aplicación tiene que generar automáticamente contraseñas, asegúrese de que las contraseñas generadas sean aleatorias y tengan una alta entropía.</p>|

## <a name="implement-controls-to-prevent-username-enumeration"></a><a id="controls-username-enum"></a>Implementación de controles para impedir la enumeración de nombres de usuario

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Todos los mensajes de error deben estar generalizados con el fin de evitar la enumeración de nombres de usuario. También, no podrá evitar a veces la pérdida de información en funcionalidades tales como una página de registro. Aquí debe usar métodos de limitación de velocidad, como CAPTCHA, para impedir ataques automatizados. |

## <a name="when-possible-use-windows-authentication-for-connecting-to-sql-server"></a><a id="win-authn-sql"></a>Cuando sea posible, usar la autenticación de Windows para conectarse a SQL Server

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | OnPrem |
| **Atributos**              | Versión de SQL: todas |
| **Referencias**              | [SQL Server: elegir un modo de autenticación](https://msdn.microsoft.com/library/ms144284.aspx) |
| **Pasos** | La autenticación de Windows usa el protocolo de seguridad de Kerberos, proporciona la aplicación de directivas de contraseñas en cuanto a la validación de la complejidad de las contraseñas seguras, ofrece compatibilidad para el bloqueo de cuentas y admite la expiración de las contraseñas.|

## <a name="when-possible-use-azure-active-directory-authentication-for-connecting-to-sql-database"></a><a id="aad-authn-sql"></a>Cuando sea posible, usar la autenticación de Azure Active Directory para conectarse a SQL Database

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | SQL Azure |
| **Atributos**              | Versión de SQL: V12 |
| **Referencias**              | [Conexión a SQL Database mediante autenticación de Azure Active Directory](https://azure.microsoft.com/documentation/articles/sql-database-aad-authentication/) |
| **Pasos** | **Versión mínima:** se requiere Azure SQL Database V12 para que Azure SQL Database pueda usar la autenticación de AAD en el directorio de Microsoft. |

## <a name="when-sql-authentication-mode-is-used-ensure-that-account-and-password-policy-are-enforced-on-sql-server"></a><a id="authn-account-pword"></a>Cuando se use el modo de autenticación de SQL, asegurarse de que se apliquen directivas de cuenta y contraseña en SQL Server

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Directiva de contraseñas de SQL Server](https://technet.microsoft.com/library/ms161959(v=sql.110).aspx) |
| **Pasos** | Cuando se usa la autenticación de SQL Server, se crean inicios de sesión en SQL Server que no se basan en cuentas de usuario de Windows. El nombre de usuario y la contraseña se crean mediante SQL Server y se almacenan en SQL Server. SQL Server puede emplear mecanismos de directiva de contraseñas de Windows. En este sentido, puede aplicar las mismas directivas de complejidad y expiración usadas en Windows a las contraseñas utilizadas dentro de SQL Server. |

## <a name="do-not-use-sql-authentication-in-contained-databases"></a><a id="autn-contained-db"></a>No usar la autenticación SQL en bases de datos independientes

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | OnPrem, SQL Azure |
| **Atributos**              | Versión de SQL: MSSQL2012, Versión de SQL: V12 |
| **Referencias**              | [Prácticas recomendadas de seguridad con bases de datos independientes](https://msdn.microsoft.com/library/ff929055.aspx) |
| **Pasos** | La ausencia de una directiva de contraseña exigida puede aumentar la probabilidad de que se establezca una credencial no segura en una base de datos independiente. Aproveche la autenticación de Windows. |

## <a name="use-per-device-authentication-credentials-using-sas-tokens"></a><a id="authn-sas-tokens"></a>Uso de las credenciales de autenticación por dispositivo mediante tokens SaS

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Centro de eventos de Azure |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Introducción al modelo de autenticación y seguridad de Event Hubs](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **Pasos** | <p>El modelo de seguridad de Event Hubs se basa en una combinación de tokens de firma de acceso compartido (SAS) y publicadores de eventos. El nombre del publicador representa el valor de DeviceID que recibe el token. Esto ayudaría a asociar los tokens generados con los dispositivos correspondientes.</p><p>Todos los mensajes se etiquetan con el originador en el lado del servicio, lo que permite la detección de los intentos de suplantación de origen en la carga. Al autenticar a los dispositivos, genere un token de SaS por dispositivo cuyo ámbito sea un único publicador.</p>|

## <a name="enable-azure-multi-factor-authentication-for-azure-administrators"></a><a id="multi-factor-azure-admin"></a>Habilitación de Azure Multi-Factor Authentication para administradores de Azure

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de Azure |
| **Fase de SDL**               | Implementación |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [¿Qué es Azure Multi-Factor Authentication?](https://azure.microsoft.com/documentation/articles/multi-factor-authentication/) |
| **Pasos** | <p>Multi-Factor Authentication (MFA) es un método de autenticación que requiere más de un método de comprobación y agrega un segundo nivel importante de seguridad crítico a las transacciones e inicios de sesión del usuario. Funciona mediante la solicitud de dos o más de los siguientes métodos de verificación:</p><ul><li>Un elemento que conoce (normalmente una contraseña).</li><li>Un elemento del que dispone (un dispositivo de confianza que no se puede duplicar con facilidad, como un teléfono).</li><li>Un elemento físico que le identifica (biométrica).</li><ul>|

## <a name="restrict-anonymous-access-to-service-fabric-cluster"></a><a id="anon-access-cluster"></a>Restricción del acceso anónimo al clúster de Service Fabric

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de Service Fabric |
| **Fase de SDL**               | Implementación |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Entorno: Azure  |
| **Referencias**              | [Escenarios de seguridad de los clústeres de Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security) |
| **Pasos** | <p>Los clústeres siempre deben estar protegidos para evitar que usuarios no autorizados se conecten a su clúster, especialmente cuando en él se están ejecutando cargas de trabajo de producción.</p><p>Al crear un clúster de Service Fabric, asegúrese de que el modo de seguridad esté establecido en "seguro" y configure el certificado de servidor X.509 necesario. La creación de un clúster "poco seguro" que expone puntos de conexión de administración en la red pública de Internet permitirá que cualquier usuario anónimo se conecte a él.</p>|

## <a name="ensure-that-service-fabric-client-to-node-certificate-is-different-from-node-to-node-certificate"></a><a id="fabric-cn-nn"></a>Asegurarse de que el certificado de cliente a nodo de Service Fabric sea diferente del certificado de nodo a nodo

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de Service Fabric |
| **Fase de SDL**               | Implementación |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Entorno: Azure, Entorno: independiente |
| **Referencias**              | [Seguridad de certificado de cliente a nodo de Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security/#_client-to-node-certificate-security), [Conexión a un clúster seguro mediante certificados de cliente](https://azure.microsoft.com/documentation/articles/service-fabric-connect-to-secure-cluster/) |
| **Pasos** | <p>La seguridad basada en certificados de cliente a nodo se configura al crear el clúster mediante Azure Portal, las plantillas de Resource Manager o una plantilla JSON independiente, especificando un certificado de cliente de administración y uno de cliente de usuario.</p><p>Los certificados de cliente de administración y de cliente de usuario que especifique deben ser diferentes de los certificados principales y secundarios que determine para la seguridad de nodo a nodo.</p>|

## <a name="use-aad-to-authenticate-clients-to-service-fabric-clusters"></a><a id="aad-client-fabric"></a>Uso de AAD para autenticar a los clientes en los clústeres de Service Fabric

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de Service Fabric |
| **Fase de SDL**               | Implementación |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Entorno: Azure |
| **Referencias**              | [Escenarios de seguridad de los clústeres de Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security/#security-recommendations) |
| **Pasos** | Los clústeres que se ejecutan en Azure también pueden proteger el acceso a los puntos de conexión de administración con Azure Active Directory (AAD), aparte de los certificados de cliente. Para los clústeres de Azure, se recomienda utilizar la seguridad de AAD para autenticar clientes y certificados para la seguridad de nodo a nodo.|

## <a name="ensure-that-service-fabric-certificates-are-obtained-from-an-approved-certificate-authority-ca"></a><a id="fabric-cert-ca"></a>Asegurarse de que los certificados de Service Fabric se obtienen de una entidad de certificación (CA) aprobada

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de Service Fabric |
| **Fase de SDL**               | Implementación |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Entorno: Azure |
| **Referencias**              | [Certificados X.509 y Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security/#x509-certificates-and-service-fabric) |
| **Pasos** | <p>Service Fabric usa certificados de servidor X.509 para autenticar nodos y clientes.</p><p>Algunos aspectos a tener en cuenta al usar certificados en Service Fabric:</p><ul><li>Los certificados usados en clústeres que ejecutan cargas de trabajo de producción deberán crearse mediante un servicio de certificados de Windows Server correctamente configurado u obtenerse de una entidad de certificación (CA) autorizada. La entidad de certificación puede ser una CA externa aprobada o una infraestructura de clave pública (PKI) interna administrada correctamente.</li><li>No use nunca certificados temporales o de pruebas en producción creados con herramientas como MakeCert.exe.</li><li>Puede usar un certificado autofirmado, pero solo para clústeres de prueba y no para los que se encuentran en fase de producción.</li></ul>|

## <a name="use-standard-authentication-scenarios-supported-by-identity-server"></a><a id="standard-authn-id"></a>Uso de escenarios de autenticación estándar admitidos por Identity Server

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Identity Server |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [IdentityServer3: The Big Picture](https://identityserver.github.io/Documentation/docsv2/overview/bigPicture.html) (IdentityServer3: el panorama global) |
| **Pasos** | <p>A continuación se muestran las interacciones habituales admitidas por Identity Server:</p><ul><li>Los exploradores se comunican con las aplicaciones web.</li><li>Las aplicaciones web se comunican con las API web (unas veces por sí mismas y otras en nombre de un usuario).</li><li>Las aplicaciones basadas en explorador se comunican con las API web.</li><li>Las aplicaciones nativas se comunican con las API web.</li><li>Las aplicaciones basadas en servidor se comunican con las API web.</li><li>Las API web se comunican con las API web (una veces por sí mismas y otras en nombre del usuario).</li></ul>|

## <a name="override-the-default-identity-server-token-cache-with-a-scalable-alternative"></a><a id="override-token"></a>Reemplazo de la caché de tokens predeterminada de Identity Server por una alternativa escalable

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Identity Server |
| **Fase de SDL**               | Implementación |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Identity Server Deployment - Caching](https://identityserver.github.io/Documentation/docsv2/advanced/deployment.html) (Identity Server: implementación y almacenamiento en caché) |
| **Pasos** | <p>IdentityServer incorpora una caché en memoria sencilla. Aunque esta caché es buena para aplicaciones nativas a pequeña escala, no se amplía para aplicaciones de back-end y nivel intermedio por los siguientes motivos:</p><ul><li>Muchos usuarios acceden a la vez a estas aplicaciones. Como todos los tokens de acceso se guardan en el mismo almacén, se crean problemas de aislamiento y surgen dificultades al funcionar a escala: un número elevado de usuarios, cada uno con tantos tokens como recursos a los que la aplicación accede en su nombre, puede traducirse en cifras enormes y operaciones de búsqueda costosas.</li><li>Estas aplicaciones se implementan normalmente en topologías distribuidas, donde varios nodos deben tener acceso a la misma caché</li><li>Los tokens almacenados en caché deben sobrevivir a los reciclajes y las desactivaciones de los procesos.</li><li>Por todas estas razones, al implementar aplicaciones web, se recomienda reemplazar la memoria caché de tokens predeterminada de Identity Server por una alternativa escalable, como Azure Cache for Redis.</li></ul>|

## <a name="ensure-that-deployed-applications-binaries-are-digitally-signed"></a><a id="binaries-signed"></a>Asegurarse de que los archivos binarios de la aplicación implementada están firmados digitalmente

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límite de confianza de la máquina |
| **Fase de SDL**               | Implementación |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que los archivos binarios de la aplicación implementada estén firmados digitalmente para que se pueda comprobar su integridad.|

## <a name="enable-authentication-when-connecting-to-msmq-queues-in-wcf"></a><a id="msmq-queues"></a>Habilitación de la autenticación al conectarse a las colas MSMQ en WCF

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico, .NET Framework 3 |
| **Atributos**              | N/D |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx) |
| **Pasos** | Si un programa no puede habilitar la autenticación al conectarse a colas MSMQ, un atacante podría enviar mensajes de forma anónima a la cola para procesarlos. Si no se usa autenticación para conectarse a una cola MSMQ que se utiliza para entregar un mensaje a otro programa, un atacante podría enviar un mensaje anónimo malintencionado.|

### <a name="example"></a>Ejemplo
El elemento `<netMsmqBinding/>` del archivo de configuración de WCF siguiente indica a WCF que deshabilite la autenticación al conectarse a una cola MSMQ para la entrega de mensajes.
```
<bindings>
    <netMsmqBinding>
        <binding>
            <security>
                <transport msmqAuthenticationMode=""None"" />
            </security>
        </binding>
    </netMsmqBinding>
</bindings>
```
Configure MSMQ para que exija la autenticación de dominios o certificados de Windows cada vez que entre o salga algún mensaje.

### <a name="example"></a>Ejemplo
El elemento `<netMsmqBinding/>` del archivo de configuración de WCF siguiente indica a WCF que habilite la autenticación mediante certificado al conectarse a una cola MSMQ. El cliente se autentica mediante certificados X.509. El certificado de cliente debe estar presente en el almacén de certificados del servidor.
```
<bindings>
    <netMsmqBinding>
        <binding>
            <security>
                <transport msmqAuthenticationMode=""Certificate"" />
            </security>
        </binding>
    </netMsmqBinding>
</bindings>
```

## <a name="wcf-do-not-set-message-clientcredentialtype-to-none"></a><a id="message-none"></a>No establecer clientCredentialType de WCF en Ninguno

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | .NET Framework 3 |
| **Atributos**              | ClientCredentialType: Ninguno |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify](https://community.microfocus.com/t5/UFT-Discussions/UFT-API-Test-with-WCF-wsHttpBinding/m-p/600927) |
| **Pasos** | La ausencia de autenticación significa que todo el mundo puede acceder a este servicio. Un servicio que no autentica a sus clientes permite el acceso a todos los usuarios. Configure la aplicación para autenticarse con credenciales de cliente. Para ello, establezca el mensaje clientCredentialType en Windows o Certificado. |

### <a name="example"></a>Ejemplo
```
<message clientCredentialType=""Certificate""/>
```

## <a name="wcf-do-not-set-transport-clientcredentialtype-to-none"></a><a id="transport-none"></a>No establecer Transport clientCredentialType de WCF en Ninguno

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico, .NET Framework 3 |
| **Atributos**              | ClientCredentialType: Ninguno |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify](https://community.microfocus.com/t5/UFT-Discussions/UFT-API-Test-with-WCF-wsHttpBinding/m-p/600927) |
| **Pasos** | La ausencia de autenticación significa que todo el mundo puede acceder a este servicio. Un servicio que no se autentica a sus clientes permite que todos los usuarios accedan a su funcionalidad. Configure la aplicación para autenticarse con credenciales de cliente. Para ello, establezca el valor de clientCredentialType de transporte en Windows o Certificado. |

### <a name="example"></a>Ejemplo
```
<transport clientCredentialType=""Certificate""/>
```

## <a name="ensure-that-standard-authentication-techniques-are-used-to-secure-web-apis"></a><a id="authn-secure-api"></a>Asegurarse de que se usan técnicas de autenticación estándar para proteger las API web

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Authentication and Authorization in ASP.NET Web API](https://www.asp.net/web-api/overview/security/authentication-and-authorization-in-aspnet-web-api), (Autenticación y autorización en la API web de ASP.NET) [External Authentication Services with ASP.NET Web API (C#)](https://www.asp.net/web-api/overview/security/external-authentication-services) (Servicios de autenticación externos con la API web de ASP.NET [C#]) |
| **Pasos** | <p>La autenticación es el proceso donde una entidad demuestra su identidad, normalmente por medio de credenciales, como un nombre de usuario y una contraseña. Hay varios protocolos de autenticación disponibles que se pueden considerar. A continuación se enumeran algunos de ellos:</p><ul><li>Certificados de cliente</li><li>Basado en Windows</li><li>Basado en formularios</li><li>Federación - ADFS</li><li>Federación - Azure AD</li><li>Federación - Identity Server</li></ul><p>Los vínculos de la sección de referencias proporcionan detalles de bajo nivel sobre cómo se puede implementar cada uno de los esquemas de autenticación para proteger una API web.</p>|

## <a name="use-standard-authentication-scenarios-supported-by-azure-active-directory"></a><a id="authn-aad"></a>Uso de escenarios de autenticación estándar compatibles con Azure Active Directory

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure AD |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Escenarios de autenticación para Azure AD](https://azure.microsoft.com/documentation/articles/active-directory-authentication-scenarios/), [Ejemplos de código de Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-code-samples/), [Guía del desarrollador de Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-developers-guide/) |
| **Pasos** | <p>Azure Active Directory (Azure AD) simplifica la autenticación para los desarrolladores al proporcionar identidad como servicio, con compatibilidad con protocolos estándar de la industria, como OAuth 2.0 y OpenID Connect. Estos son los cinco escenarios principales de aplicación que admite Azure AD:</p><ul><li>Explorador web para aplicación web: un usuario tiene que iniciar sesión en una aplicación web protegida por Azure AD</li><li>Aplicación de una sola página (SPA): un usuario tiene que iniciar sesión en una aplicación de una sola página protegida por Azure AD</li><li>Aplicación nativa para API web: una aplicación nativa que se ejecuta en teléfonos, tabletas o equipos tiene que autenticar a un usuario para obtener recursos de una API web protegida por Azure AD</li><li>Aplicación web para API web: una aplicación web tiene que obtener recursos de una API web protegida por Azure AD</li><li>Aplicación de servidor o de demonio para API web: una aplicación de demonio o de servidor sin interfaz de usuario web tiene que obtener recursos de una API web protegida por Azure AD</li></ul><p>Consulte los vínculos de la sección de referencias para obtener detalles de implementación de bajo nivel.</p>|

## <a name="override-the-default-adal-token-cache-with-a-scalable-alternative"></a><a id="adal-scalable"></a>Reemplazo de la caché de tokens predeterminada de ADAL por una alternativa escalable

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure AD |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Modern Authentication with Azure Active Directory for Web Applications](https://blogs.msdn.microsoft.com/microsoft_press/2016/01/04/new-book-modern-authentication-with-azure-active-directory-for-web-applications/) (Autenticación moderna con Azure Active Directory para aplicaciones web), [Using Redis as ADAL token cache](https://blogs.msdn.microsoft.com/mrochon/2016/09/19/using-redis-as-adal-token-cache/) (Uso de Redis como caché de tokens ADAL)  |
| **Pasos** | <p>La caché predeterminada que usa ADAL (Biblioteca de autenticación de Active Directory) es una caché en memoria que se basa en un almacén estático disponible en todo el proceso. Aunque esta caché funciona para aplicaciones nativas, no se amplía para aplicaciones de nivel intermedio y back-end por los siguientes motivos:</p><ul><li>Muchos usuarios acceden a la vez a estas aplicaciones. Como todos los tokens de acceso se guardan en el mismo almacén, se crean problemas de aislamiento y surgen dificultades al funcionar a escala: un número elevado de usuarios, cada uno con tantos tokens como recursos a los que la aplicación accede en su nombre, puede traducirse en cifras enormes y operaciones de búsqueda costosas.</li><li>Estas aplicaciones se implementan normalmente en topologías distribuidas, donde varios nodos deben tener acceso a la misma caché</li><li>Los tokens almacenados en caché deben sobrevivir a los reciclajes y las desactivaciones de los procesos.</li></ul><p>Por todas estas razones, al implementar aplicaciones web, se recomienda reemplazar la memoria caché de tokens predeterminada de ADAL por una alternativa escalable, como Azure Cache for Redis.</p>|

## <a name="ensure-that-tokenreplaycache-is-used-to-prevent-the-replay-of-adal-authentication-tokens"></a><a id="tokenreplaycache-adal"></a>Asegurarse de que TokenReplayCache se usa para impedir la reproducción de tokens de autenticación de ADAL

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure AD |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Modern Authentication with Azure Active Directory for Web Applications](https://blogs.msdn.microsoft.com/microsoft_press/2016/01/04/new-book-modern-authentication-with-azure-active-directory-for-web-applications/) (Autenticación moderna con Azure Active Directory para aplicaciones web) |
| **Pasos** | <p>La propiedad TokenReplayCache permite a los desarrolladores definir una caché de reproducción de tokens, un almacén que se puede usar para guardar tokens con el fin de comprobar que ningún token se pueda usar más de una vez.</p><p>Esta es una medida contra un ataque habitual, el llamado acertadamente ataque de reproducción de tokens: un atacante que intercepta el token enviado al inicio de sesión podría enviarlo de nuevo a la aplicación ("reproducirlo") para establecer una nueva sesión. Por ejemplo, en el flujo de concesión de código de OIDC, tras la autenticación correcta del usuario, se realiza una solicitud al punto de conexión "/signin-oidc" del usuario de confianza con los parámetros "id_token", "code" y "state".</p><p>El usuario de confianza valida esta solicitud y establece una nueva sesión. Si un adversario captura esta solicitud y la reproduce, podría establecer correctamente una sesión y suplantar al usuario. La presencia del valor de seguridad nonce de OpenID Connect puede limitar, pero no eliminar totalmente, las circunstancias en las que el ataque se puede aprobar correctamente. Para proteger sus aplicaciones, los desarrolladores pueden proporcionar una implementación de ITokenReplayCache y asignar una instancia a TokenReplayCache.</p>|

### <a name="example"></a>Ejemplo
```csharp
// ITokenReplayCache defined in ADAL
public interface ITokenReplayCache
{
bool TryAdd(string securityToken, DateTime expiresOn);
bool TryFind(string securityToken);
}
```

### <a name="example"></a>Ejemplo
Este es un ejemplo de implementación de la interfaz ITokenReplayCache. (Personalice e implemente el marco de almacenamiento en caché específico del proyecto).
```csharp
public class TokenReplayCache : ITokenReplayCache
{
    private readonly ICacheProvider cache; // Your project-specific cache provider
    public TokenReplayCache(ICacheProvider cache)
    {
        this.cache = cache;
    }
    public bool TryAdd(string securityToken, DateTime expiresOn)
    {
        if (this.cache.Get<string>(securityToken) == null)
        {
            this.cache.Set(securityToken, securityToken);
            return true;
        }
        return false;
    }
    public bool TryFind(string securityToken)
    {
        return this.cache.Get<string>(securityToken) != null;
    }
}
```
Se debe hacer referencia a la caché implementada en las opciones de OIDC mediante la propiedad "TokenValidationParameters", como se indica a continuación.
```csharp
OpenIdConnectOptions openIdConnectOptions = new OpenIdConnectOptions
{
    AutomaticAuthenticate = true,
    ... // other configuration properties follow..
    TokenValidationParameters = new TokenValidationParameters
    {
        TokenReplayCache = new TokenReplayCache(/*Inject your cache provider*/);
    }
}
```

Tenga en cuenta que para probar la eficacia de esta configuración, debe iniciar sesión en la aplicación local protegida por OIDC y capturar la solicitud al punto de conexión `"/signin-oidc"` en Fiddler. Cuando la protección no está implementada, la reproducción de esta solicitud en Fiddler establecerá una nueva cookie de sesión. Cuando la solicitud se reproduce después de agregar la protección de TokenReplayCache, la aplicación producirá una excepción de la manera siguiente:`SecurityTokenReplayDetectedException: IDX10228: The securityToken has previously been validated, securityToken: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSIsImtpZCI6Ik1uQ1......`.

## <a name="use-adal-libraries-to-manage-token-requests-from-oauth2-clients-to-aad-or-on-premises-ad"></a><a id="adal-oauth2"></a>Uso de bibliotecas de ADAL para administrar las solicitudes de token de clientes de OAuth2 a AAD (o AD local)

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure AD |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [ADAL](https://azure.microsoft.com/documentation/articles/active-directory-authentication-libraries/) |
| **Pasos** | <p>La biblioteca de autenticación de Azure AD (ADAL) permite a los desarrolladores de aplicaciones cliente autenticar fácilmente a los usuarios en Active Directory (AD) local o en la nube y luego obtener tokens de acceso para proteger las llamadas de API.</p><p>ADAL incluye muchas características que hacen que la autenticación resulte más fácil para los desarrolladores, como la compatibilidad asincrónica, una memoria caché de tokens configurable que almacena tokens de acceso y tokens de actualización, actualización automática de tokens cuando caduca un token de acceso y hay un token de actualización disponible, etc.</p><p>Al controlar la mayor parte de la complejidad, ADAL puede ayudar a los desarrolladores a centrarse en la lógica de negocios de su aplicación y proteger recursos fácilmente sin ser un experto en seguridad. Otras bibliotecas están disponibles para .NET, JavaScript (cliente y Node.js), Python, iOS, Android y Java.</p>|

## <a name="authenticate-devices-connecting-to-the-field-gateway"></a><a id="authn-devices-field"></a>Autenticación de los dispositivos que se conectan a la puerta de enlace de campo

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de campo de IoT |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que cada dispositivo se autentica en la puerta de enlace de campo antes de aceptar datos de ellos y antes de facilitar las comunicaciones ascendentes con la puerta de enlace de nube. Además, asegúrese de que los dispositivos se conectan con una credencial cada uno de forma que cada dispositivo se pueda identificar de manera unívoca.|

## <a name="ensure-that-devices-connecting-to-cloud-gateway-are-authenticated"></a><a id="authn-devices-cloud"></a>Asegurarse de que se autentican los dispositivos que se conectan a la puerta de enlace de nube

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de la nube de IoT |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico, C#, Node.JS,  |
| **Atributos**              | N/D, opción de puerta de enlace: Azure IoT Hub |
| **Referencias**              | N/D, [Introducción a Azure IoT Hub (.NET)](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/), [Introducción a Azure IoT Hub (Node)](https://azure.microsoft.com/documentation/articles/iot-hub-node-node-getstarted), [Protección de IoT con SAS y certificados](https://azure.microsoft.com/documentation/articles/iot-hub-sas-tokens/), [Repositorio de GIT](https://github.com/Azure/azure-iot-sdks/) |
| **Pasos** | <ul><li>**Genérico:** autenticación del dispositivo mediante Seguridad de la capa de transporte (TLS) o IPSec. La infraestructura debe admitir el uso de una clave precompartida (PSK) en los dispositivos que no pueden controlar la criptografía asimétrica completa. Aprovechamiento de Azure AD, OAuth</li><li>**C#:** al crear una instancia de DeviceClient, el método Create crea de forma predeterminada una instancia de DeviceClient que usa el protocolo AMQP para comunicarse con IoT Hub. Para usar el protocolo HTTPS, utilice la invalidación del método Create que permite especificar el protocolo. Si usa el protocolo HTTPS, también debe agregar el paquete NuGet `Microsoft.AspNet.WebApi.Client` al proyecto para incluir el espacio de nombres `System.Net.Http.Formatting`.</li></ul>|

### <a name="example"></a>Ejemplo
```csharp
static DeviceClient deviceClient;

static string deviceKey = "{device key}";
static string iotHubUri = "{iot hub hostname}";

var messageString = "{message in string format}";
var message = new Message(Encoding.ASCII.GetBytes(messageString));

deviceClient = DeviceClient.Create(iotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey("myFirstDevice", deviceKey));

await deviceClient.SendEventAsync(message);
```

### <a name="example"></a>Ejemplo
**Node.JS: autenticación**
#### <a name="symmetric-key"></a>Clave simétrica
* Cree un centro de IoT en Azure.
* Cree una entrada en el Registro de identidad de dispositivo:
    ```javascript
    var device = new iothub.Device(null);
    device.deviceId = <DeviceId >
    registry.create(device, function(err, deviceInfo, res) {})
    ```
* Cree un dispositivo simulado:
    ```javascript
    var clientFromConnectionString = require('azure-iot-device-amqp').clientFromConnectionString;
    var Message = require('azure-iot-device').Message;
    var connectionString = 'HostName=<HostName>DeviceId=<DeviceId>SharedAccessKey=<SharedAccessKey>';
    var client = clientFromConnectionString(connectionString);
    ```
  #### <a name="sas-token"></a>Token de SAS
* Se genera internamente cuando se usa la clave simétrica pero también se puede generar y usar explícitamente.
* Defina un protocolo:`var Http = require('azure-iot-device-http').Http;`
* Cree un token de SAS:
    ```javascript
    resourceUri = encodeURIComponent(resourceUri.toLowerCase()).toLowerCase();
    var deviceName = "<deviceName >";
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    var toSign = resourceUri + '\n' + expires;
    // using crypto
    var decodedPassword = new Buffer(signingKey, 'base64').toString('binary');
    const hmac = crypto.createHmac('sha256', decodedPassword);
    hmac.update(toSign);
    var base64signature = hmac.digest('base64');
    var base64UriEncoded = encodeURIComponent(base64signature);
    // construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "%2fdevices%2f"+deviceName+"&sig="
  + base64UriEncoded + "&se=" + expires;
    if (policyName) token += "&skn="+policyName;
    return token;
    ```
* Conéctese mediante el token de SAS:
    ```javascript
    Client.fromSharedAccessSignature(sas, Http);
    ```
  #### <a name="certificates"></a>Certificados
* Genere un certificado X509 autofirmado con cualquier herramienta como OpenSSL, para generar archivos .cert y .key a fin de almacenar el certificado y la clave respectivamente.
* Aprovisione un dispositivo que acepte conexiones seguras mediante certificados.
    ```javascript
    var connectionString = '<connectionString>';
    var registry = iothub.Registry.fromConnectionString(connectionString);
    var deviceJSON = {deviceId:"<deviceId>",
    authentication: {
        x509Thumbprint: {
        primaryThumbprint: "<PrimaryThumbprint>",
        secondaryThumbprint: "<SecondaryThumbprint>"
        }
    }}
    var device = deviceJSON;
    registry.create(device, function (err) {});
    ```
* Conecte un dispositivo mediante un certificado.
    ```javascript
    var Protocol = require('azure-iot-device-http').Http;
    var Client = require('azure-iot-device').Client;
    var connectionString = 'HostName=<HostName>DeviceId=<DeviceId>x509=true';
    var client = Client.fromConnectionString(connectionString, Protocol);
    var options = {
        key: fs.readFileSync('./key.pem', 'utf8'),
        cert: fs.readFileSync('./server.crt', 'utf8')
    };
    // Calling setOptions with the x509 certificate and key (and optionally, passphrase) will configure the client //transport to use x509 when connecting to IoT Hub
    client.setOptions(options);
    //call fn to execute after the connection is set up
    client.open(fn);
    ```

## <a name="use-per-device-authentication-credentials"></a><a id="authn-cred"></a>Uso de credenciales de autenticación por dispositivo

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de la nube de IoT  |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Elección de puerta de enlace: Azure IoT Hub |
| **Referencias**              | [Tokens de seguridad de Azure IoT Hub](https://azure.microsoft.com/documentation/articles/iot-hub-sas-tokens/) |
| **Pasos** | Use credenciales de autenticación por dispositivo mediante tokens de SaS basados en la clave de dispositivo y el certificado de cliente, en lugar de directivas de acceso compartido de nivel de IoT Hub. Así evitará la reutilización de tokens de autenticación entre dispositivos o puertas de enlace de campo. |

## <a name="ensure-that-only-the-required-containers-and-blobs-are-given-anonymous-read-access"></a><a id="req-containers-anon"></a>Asegurarse de que solo los contenedores y blobs requeridos reciben acceso anónimo de lectura

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | StorageType: blob |
| **Referencias**              | [Administración del acceso de lectura anónimo a contenedores y blobs](https://azure.microsoft.com/documentation/articles/storage-manage-access-to-resources/), [Firmas de acceso compartido, Parte 1: Descripción del modelo SAS](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/) |
| **Pasos** | <p>De forma predeterminada, solamente el dueño de la cuenta de almacenamiento puede obtener acceso a un contenedor y a todos los blobs en su interior. Para dar a los usuarios anónimos permisos de lectura para un contenedor y sus blobs, puede establecer los permisos del contenedor de forma que se permita el acceso público. Los usuarios anónimos pueden leer los blobs que estén en un contenedor con acceso público sin necesidad de tener que autenticar la solicitud.</p><p>Los contenedores ofrecen las siguientes opciones para administrar el acceso al contenedor:</p><ul><li>Acceso total de lectura público: los datos del contenedor y de los blobs se pueden leer mediante una solicitud anónima. Los clientes pueden enumerar los blobs del contenedor a través de una solicitud anónima, pero no pueden enumerar los contenedores que están en la cuenta de almacenamiento.</li><li>Acceso de lectura público solo para blobs: Los datos de blob dentro de este contenedor pueden leerse a través de una solicitud anónima, pero los datos del contenedor no están disponibles. Los clientes no pueden enumerar los blobs incluidos en el contenedor mediante una solicitud anónima.</li><li>Acceso de lectura no público: el propietario de la cuenta es el único que puede leer los datos del contenedor y de los blobs</li></ul><p>El acceso anónimo es mejor para escenarios donde ciertos blobs deben estar siempre disponibles para el acceso de lectura anónimo. Para un control más específico, puede crear una firma de acceso compartido, que permite delegar el acceso restringido mediante permisos diferentes y en un intervalo de tiempo especificado. Asegúrese de que los contenedores y blobs, que podrían contener datos confidenciales, no tengan acceso anónimo por accidente.</p>|

## <a name="grant-limited-access-to-objects-in-azure-storage-using-sas-or-sap"></a><a id="limited-access-sas"></a>Concesión de acceso limitado a objetos de Azure Storage mediante SAS o SAP

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage |
| **Fase de SDL**               | Build |
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D |
| **Referencias**              | [Firmas de acceso compartido, Parte 1: Comprender el modelo SAS](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/), [Firmas de acceso compartido, Parte 2: Creación y uso de una SAS con Blob Storage](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-2/), [Delegación del acceso a objetos en la cuenta mediante Firmas de acceso compartido y Directivas de acceso almacenadas](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_how-to-delegate-access-to-objects-in-your-account-using-shared-access-signatures-and-stored-access-policies) |
| **Pasos** | <p>El uso de una firma de acceso compartido (SAS) es una manera eficaz de conceder acceso limitado a los objetos de una cuenta de almacenamiento a otros clientes sin tener que exponer la clave de acceso de la cuenta. La SAS es un URI que incluye en sus parámetros de consulta toda la información necesaria para el acceso autenticado a un recurso de almacenamiento. Para obtener acceso a los recursos de almacenamiento con la SAS, el cliente solo tiene que pasar la SAS al método o constructor adecuados.</p><p>Puede usar una SAS cuando desee proporcionar acceso a los recursos en la cuenta de almacenamiento a un cliente al que no se puede confiar la clave de cuenta. Las claves de cuenta de almacenamiento incluyen una clave primaria y secundaria, que conceden acceso administrativo a la cuenta y a todos los recursos contenidos en ella. La exposición de alguna de las claves de cuenta hace que exista la posibilidad de que se haga un uso negligente o malintencionado de la cuenta. Las firmas de acceso compartido ofrecen una alternativa segura que permite a los demás clientes leer, escribir y eliminar datos en la cuenta de almacenamiento según los permisos que se le hayan concedido y sin que sea necesaria la clave de cuenta.</p><p>Si tiene un conjunto lógico de parámetros que son siempre parecidos, es mejor usar una directiva de acceso almacenado (SAP). Dado que el uso de una Firma de acceso compartido derivada de una Directiva de acceso almacenada ofrece la posibilidad de revocar esa firma de acceso compartido de inmediato, se recomienda usar siempre las Directivas de acceso almacenadas cuando sea posible.</p>|
