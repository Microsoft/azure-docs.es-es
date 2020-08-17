---
title: 'Criptografía: Microsoft Threat Modeling Tool (Azure) | Microsoft Docs'
description: Obtenga información sobre la mitigación de la criptografía para las amenazas expuestas en Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
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
ms.openlocfilehash: 7de0cad91e01187a1ed84257c9e3a7cd8106951a
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87539958"
---
# <a name="security-frame-cryptography--mitigations"></a>Marco de seguridad: Criptografía | Mitigaciones 

| Producto o servicio | Artículo |
| --------------- | ------- |
| **Aplicación web** | <ul><li>[Use solamente los cifrados de bloques simétricos y longitudes de clave aprobados](#cipher-length)</li><li>[Use modos aprobados de cifrado de bloques y vectores de inicialización apropiados para los cifrados simétricos](#vector-ciphers)</li><li>[Use algoritmos asimétricos, longitudes de clave y rellenos aprobados](#padding)</li><li>[Use generadores de números aleatorios aprobados](#numgen)</li><li>[No use cifrados de flujo simétricos](#stream-ciphers)</li><li>[Use los algoritmos hash con clave MAC o HMAC aprobados](#mac-hash)</li><li>[Use solamente las funciones hash criptográficas aprobadas](#hash-functions)</li></ul> |
| **Base de datos** | <ul><li>[Use algoritmos de cifrado de alta seguridad para cifrar los datos de la base de datos](#strong-db)</li><li>[Los paquetes SSIS deben cifrarse y firmarse digitalmente](#ssis-signed)</li><li>[Agregue una firma digital a los elementos protegibles críticos de base de datos ](#securables-db)</li><li>[Use la administración extensible de claves (EKM) de SQL Server para proteger las claves de cifrado](#ekm-keys)</li><li>[Use la característica AlwaysEncrypted si las claves de cifrado no deben mostrarse al motor de base de datos](#keys-engine)</li></ul> |
| **Dispositivo IoT** | <ul><li>[Almacene las claves criptográficas de forma segura en dispositivos IoT](#keys-iot)</li></ul> | 
| **Puerta de enlace de nube de IoT** | <ul><li>[Genere una clave simétrica aleatoria de longitud suficiente para la autenticación en IoT Hub](#random-hub)</li></ul> | 
| **Cliente para dispositivos móviles de Dynamics CRM** | <ul><li>[Asegúrese de que hay una directiva de administración de dispositivos que requiere el uso de un PIN y permite el borrado remoto](#pin-remote)</li></ul> | 
| **Cliente de Outlook de Dynamics CRM** | <ul><li>[Asegúrese de que hay una directiva de administración de dispositivos que requiere un bloqueo automático con PIN o contraseña, y cifra todos los datos (por ejemplo, BitLocker)](#bitlocker)</li></ul> | 
| **Identity Server** | <ul><li>[Asegúrese de que las claves de firma se sustituyen cuando se usa Identity Server](#rolled-server)</li><li>[Asegúrese de que se usan un identificador de cliente y un secreto de cliente de alta seguridad criptográfica en Identity Server](#client-server)</li></ul> | 

## <a name="use-only-approved-symmetric-block-ciphers-and-key-lengths"></a><a id="cipher-length"></a>Use solamente los cifrados de bloques simétricos y longitudes de clave aprobados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Los productos tienen que usar solo los cifrados de bloques simétricos y las longitudes de clave asociadas que haya aprobado de forma explícita el asesor de criptografía de su organización. Los algoritmos simétricos aprobados de Microsoft incluyen los cifrados de bloques siguientes:</p><ul><li>Para el nuevo código son aceptables AES-128, AES-192 y AES-256</li><li>Para mantener la compatibilidad con el código existente, el algoritmo 3DES de tres claves es aceptable</li><li>Para los productos con cifrados de bloques simétricos:<ul><li>El estándar de cifrado avanzado (AES) es necesario para el nuevo código</li><li>El estándar de cifrado de datos triple (3DES) con tres claves está permitido en el código existente para proporcionar la compatibilidad con versiones anteriores</li><li>Todos los demás cifrados de bloques, como RC2, DES, 3DES de dos claves, DESX y Skipjack, solo pueden usarse para descifrar datos antiguos y tienen que reemplazarse si se usan para el cifrado</li></ul></li><li>Para los algoritmos de cifrado de bloques simétrico, se requiere una longitud de clave mínima de 128 bits. El único algoritmo de cifrado de bloques recomendado para el nuevo código es AES (AES-128, AES-192 y AES-256 son aceptables)</li><li>El algoritmo 3DES de tres claves se acepta actualmente si ya está en uso en el código existente; se recomienda la transición al algoritmo AES. DES, DESX, RC2 y Skipjack ya no se consideran seguros. Estos algoritmos solo pueden utilizarse para descifrar datos existentes por motivos de compatibilidad con versiones anteriores, y los datos se deben volver a cifrar mediante uno de los cifrados de bloques recomendados</li></ul><p>Tenga en cuenta que todos los cifrados de bloques simétricos tienen que usarse con un modo de cifrado aprobado, lo que requiere el uso de un vector de inicialización (IV) adecuado. Un vector de inicialización adecuado, suele ser un número aleatorio y nunca un valor constante</p><p>El uso de algoritmos criptográficos heredados o no aprobados, y de longitudes de clave más pequeñas con el fin de leer los datos existentes (no de escribir nuevos datos), se puede permitir una vez que la junta de criptografía de su organización lo revise. Sin embargo, debe solicitar una excepción en contra de este requisito. Además, en las implementaciones empresariales, los productos deben considerar advertir a los administradores cuando se use una criptografía no segura para leer los datos. Dichas advertencias deben ser explicativas y prácticas. En algunos casos, puede ser adecuado, establecer una directiva de grupo para controlar el uso de criptografía no segura</p><p>Algoritmos .NET permitidos para criptografía personal administrada (en orden de preferencia)</p><ul><li>AesCng (compatible con FIPS)</li><li>AuthenticatedAesCng (compatible con FIPS)</li><li>AESCryptoServiceProvider (compatible con FIPS)</li><li>AESManaged (no compatible con FIPS)</li></ul><p>Tenga en cuenta que ninguno de estos algoritmos se puede especificar a través de los métodos `SymmetricAlgorithm.Create` o `CryptoConfig.CreateFromName` sin realizar cambios en el archivo machine.config. Además, tenga también en cuenta que AES, en las versiones de .NET anteriores a .NET 3.5, se denomina `RijndaelManaged`, y que `AesCng` y `AuthenticatedAesCng` están > disponibles a través de CodePlex y requieren CNG en el sistema operativo subyacente</p>

## <a name="use-approved-block-cipher-modes-and-initialization-vectors-for-symmetric-ciphers"></a><a id="vector-ciphers"></a>Use modos aprobados de cifrado de bloques y vectores de inicialización apropiados para los cifrados simétricos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Todos los cifrados de bloques simétricos tienen que utilizarse con un modo de cifrado simétrico aprobado. Los únicos modos aprobados son CBC y CTS. En concreto, debe evitarse el modo de libro de códigos electrónico (ECB); el uso de ECB requiere revisión por parte de la junta de criptografía de su organización. Todo posible uso de OFB, CFB, CTR, CCM y GCM o cualquier otro modo de cifrado tiene que revisarlo la junta de criptografía de su organización. Reutilizar el mismo vector de inicialización (IV) con el cifrado de bloques en "modos de cifrados de flujo" como CTR, puede provocar que se muestren los datos cifrados. Todos los cifrados de bloques simétricos tienen que utilizarse también con un vector de inicialización (IV) adecuado. Un vector de inicialización adecuado es un número aleatorio de alta seguridad criptográfica y nunca un valor constante. |

## <a name="use-approved-asymmetric-algorithms-key-lengths-and-padding"></a><a id="padding"></a>Use algoritmos asimétricos, longitudes de clave y rellenos aprobados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>El uso de algoritmos criptográficos prohibidos supone un riesgo significativo para la seguridad de los productos y debe evitarse. Los productos tienen que usar solo los algoritmos criptográficos, las longitudes de clave y los rellenos que la junta de criptografía de su organización haya aprobado explícitamente.</p><ul><li>**RSA:** puede utilizarse para el cifrado, el intercambio de claves y la firma. El cifrado RSA tiene que usar solo los modos de relleno OAEP o RSA-KEM. El código existente puede utilizar el modo de relleno PKCS #1 v1.5 solo si es necesario para garantizar la compatibilidad. El uso de relleno nulo está explícitamente prohibido. El nuevo código requiere claves de 2048 bits o más. El código existente puede admitir claves de menos de 2048 bits solamente para garantizar la compatibilidad con versiones anteriores, y después de una revisión por parte de la junta de criptografía de su organización. La claves de menos de 1024 bits se pueden usar únicamente para descifrar o verificar datos antiguos, y tienen que remplazarse si se utilizan para operaciones de cifrado o firma</li><li>**ECDSA:** puede utilizarse solo para la firma. El nuevo código requiere ECDSA con claves de 256 bits o más. Las firmas basadas en ECDSA tienen que utilizar una de las tres curvas NIST aprobadas (p-256, p-384 o P521). Las curvas que se han analizado exhaustivamente pueden utilizarse solo después de una revisión por parte de la junta de criptografía de su organización.</li><li>**ECDH:** puede utilizarse solamente para el intercambio de claves. El nuevo código requiere ECDH con claves de 256 bits o más. El intercambio de claves de ECDH tiene que utilizar una de las tres curvas NIST aprobadas (p-256, p-384 o P521). Las curvas que se han analizado exhaustivamente pueden utilizarse solo después de una revisión por parte de la junta de criptografía de su organización.</li><li>**DSA:** puede ser aceptable después de la revisión y aprobación por parte de la junta de criptografía de su organización. Póngase en contacto con su asesor de seguridad para programar la revisión de la junta de criptografía de su organización. Si se aprueba el uso de DSA, tenga en cuenta que tiene que prohibir el uso de claves de menos de 2048 bits de longitud. CNG admite longitudes de clave de 2048 bits y superiores a partir de Windows 8.</li><li>**Diffie-Hellman:** se puede usar para la administración de claves de sesión únicamente. El nuevo código requiere claves con una longitud de 2048 bits o más. El código existente puede admitir claves con una longitud inferior a 2048 bits solo para garantizar la compatibilidad con versiones anteriores y después de una revisión por parte de la junta de criptografía de la organización. La claves de menos de 1024 no se pueden usar.</li><ul>

## <a name="use-approved-random-number-generators"></a><a id="numgen"></a>Use generadores de números aleatorios aprobados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Los productos tienen que usar generadores de números aleatorios aprobados. Las funciones pseudoaleatorias, como la función en tiempo de ejecución de C, la clase de .NET Framework System.Random, o funciones del sistema como GetTickCount no deben por lo tanto usarse nunca en este código. Se prohíbe el uso del algoritmo (DUAL_EC_DRBG) del generador de números aleatorios de curva elíptica dual</p><ul><li>**CNG:** BCryptGenRandom (uso de la marca BCRYPT_USE_SYSTEM_PREFERRED_RNG recomendado a menos que el llamador pueda ejecutarse en cualquier IRQL mayor que 0 [es decir, PASSIVE_LEVEL])</li><li>**CAPI:** cryptGenRandom</li><li>**Win32/64:** RtlGenRandom (las implementaciones nuevas deben utilizar BCryptGenRandom o CryptGenRandom) * rand_s * SystemPrng (para el modo de kernel)</li><li>**.NET:** RNGCryptoServiceProvider o RNGCng</li><li>**Aplicaciones de la Tienda Windows:** Windows.Security.Cryptography.CryptographicBuffer.GenerateRandom o .GenerateRandomNumber</li><li>**Apple OS X (10.7+)/iOS(2.0+)-** int SecRandomCopyBytes (SecRandomRef random, size_t count, uint8_t \*bytes)</li><li>**Apple OS X (< 10.7)-** Use /dev/random para recuperar números aleatorios</li><li>**Java (incluido código Google Android Java) -** java.security.SecureRandom clase. Tenga en cuenta que para Android 4.3 (Jelly Bean), los desarrolladores tienen que seguir la solución alternativa recomendada por Android y actualizar sus aplicaciones para inicializar explícitamente PRNG con la entropía de /dev/urandom o /dev/Random</li></ul>|

## <a name="do-not-use-symmetric-stream-ciphers"></a><a id="stream-ciphers"></a>No use cifrados de flujo simétricos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | No pueden usarse cifrados de flujos simétricos, como RC4. En lugar de cifrados de flujos simétricos, los productos deberían utilizar un cifrado por bloques, específicamente AES con una longitud de clave de 128 bits como mínimo. |

## <a name="use-approved-machmackeyed-hash-algorithms"></a><a id="mac-hash"></a>Use los algoritmos hash con clave MAC o HMAC aprobados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Los productos deben usar solo los algoritmos aprobados de código de autenticación de mensajes (MAC) o código de autenticación de mensajes basado en hash (HMAC).</p><p>Un código de autenticación de mensajes (MAC) es un fragmento de información que se adjunta a un mensaje, y que le permite a su destinatario comprobar la autenticidad del remitente y la integridad del mensaje mediante una clave secreta. El uso o bien de un MAC basado en hash ([HMAC](https://csrc.nist.gov/publications/nistpubs/800-107-rev1/sp800-107-rev1.pdf)) o [MAC basado en el cifrado de bloques](https://csrc.nist.gov/publications/nistpubs/800-38B/SP_800-38B.pdf) está permitido siempre todos los hash subyacentes o algoritmos de cifrado simétrico estén también aprobados para su uso; actualmente esto incluye las funciones de HMAC-SHA2 (HMAC-SHA256, HMAC-SHA384 y HMAC-SHA512) y los MAC basados en cifrado por bloques CMAC/OMAC1 y OMAC2 (estos se basan en AES).</p><p>El uso de HMAC-SHA1 puede estar autorizado para la compatibilidad con plataformas, pero tendrá que solicitar una excepción para este procedimiento y someterse a una revisión de cifrado de su organización. No se permite el truncamiento de los HMAC por debajo de los 128 bits. El uso de métodos de cliente para producir valores hash de una clave y datos no está aprobado, y tiene que someterse a la revisión de la junta de criptografía de su organización antes de su uso.</p>|

## <a name="use-only-approved-cryptographic-hash-functions"></a><a id="hash-functions"></a>Use solamente las funciones hash criptográficas aprobadas

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Los productos deben utilizar la familia SHA-2 de algoritmos hash (SHA256, SHA384 y SHA512). Si se necesita un hash más corto, por ejemplo, una longitud de la salida de 128 bits para ajustar una estructura de datos diseñada pensando en el hash MD5 más corto, los equipos del producto pueden truncar uno de los valores de hash de SHA2 (normalmente SHA256). Tenga en cuenta que SHA384 es una versión truncada de SHA512. No se permite el truncamiento de los algoritmos hash criptográficos por debajo de los 128 bits por motivos de seguridad. El nuevo código no debe usar los algoritmos hash MD2, MD4, MD5, SHA-0, SHA-1 o RIPEMD. Las colisiones de hash son factibles desde el punto de vista computacional para estos algoritmos, lo que de forma efectiva supone su ruptura.</p><p>Algoritmos hash de .NET permitidos para criptografía personal administrada (en orden de preferencia):</p><ul><li>SHA512Cng (compatible con FIPS)</li><li>SHA384Cng (compatible con FIPS)</li><li>SHA256Cng (compatible con FIPS)</li><li>SHA512Managed (no compatible con FIPS) (use SHA512 como nombre de algoritmo en las llamadas a HashAlgorithm.Create o CryptoConfig.CreateFromName)</li><li>SHA384Managed (no compatible con FIPS) (use SHA384 como nombre de algoritmo en las llamadas a HashAlgorithm.Create o CryptoConfig.CreateFromName)</li><li>SHA256Managed (no compatible con FIPS) (use SHA256 como nombre de algoritmo en las llamadas a HashAlgorithm.Create o CryptoConfig.CreateFromName)</li><li>SHA512CryptoServiceProvider (compatible con FIPS)</li><li>SHA256CryptoServiceProvider (compatible con FIPS)</li><li>SHA384CryptoServiceProvider (compatible con FIPS)</li></ul>| 

## <a name="use-strong-encryption-algorithms-to-encrypt-data-in-the-database"></a><a id="strong-db"></a>Use algoritmos de cifrado de alta seguridad para cifrar los datos de la base de datos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Elegir un algoritmo de cifrado](https://technet.microsoft.com/library/ms345262(v=sql.130).aspx) |
| **Pasos** | Los algoritmos de cifrado definen las transformaciones de datos que los usuarios no autorizados no pueden revertir fácilmente. SQL Server permite a los administradores y programadores elegir entre varios algoritmos, incluidos DES, Triple DES, TRIPLE_DES_3KEY, RC2, RC4, RC4 de 128 bits, DESX, AES de 128 bits, AES de 192 bits y AES de 256 bits |

## <a name="ssis-packages-should-be-encrypted-and-digitally-signed"></a><a id="ssis-signed"></a>Los paquetes SSIS deben cifrarse y firmarse digitalmente

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Identificar el origen de paquetes con firmas digitales](https://msdn.microsoft.com/library/ms141174.aspx), [Amenaza y mitigación de vulnerabilidades (Integration Services)](https://msdn.microsoft.com/library/bb522559.aspx) |
| **Pasos** | El origen de un paquete es el individuo o la organización que creó el paquete. La ejecución de un paquete desde un origen desconocido o que no sea de confianza puede ser arriesgado. Para evitar la modificación no autorizada de paquetes SSIS, deben usarse firmas digitales. Además, para garantizar la confidencialidad de los paquetes durante el tránsito o almacenamiento, los paquetes SSIS tienen que cifrarse |

## <a name="add-digital-signature-to-critical-database-securables"></a><a id="securables-db"></a>Agregue una firma digital a los elementos protegibles críticos de base de datos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [ADD SIGNATURE (Transact-SQL)](https://msdn.microsoft.com/library/ms181700)[Agregar firma (Transact-SQL)] |
| **Pasos** | En casos donde la integridad de una base de datos crítica protegible tiene que comprobarse, deben usarse firmas digitales. Los elementos protegibles de una base de datos, como un procedimiento almacenado, una función, un ensamblado o un desencadenador pueden firmarse digitalmente. A continuación se muestra un ejemplo de cuándo una firma digital puede ser útil: supongamos que un ISV (fabricante de software independiente) ha proporcionado soporte técnico para un software entregado a uno de sus clientes. Antes de proporcionar soporte técnico, el ISV debería asegurarse de que un elemento protegible en la base de datos del software no se alteró por error o a través de un intento malicioso. Si el elemento protegible está firmado digitalmente, el ISV puede comprobar la firma digital y validar su integridad.| 

## <a name="use-sql-server-ekm-to-protect-encryption-keys"></a><a id="ekm-keys"></a>Use la administración extensible de claves (EKM) de SQL Server para proteger las claves de cifrado

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Administración extensible de claves (EKM) de SQL Server](https://msdn.microsoft.com/library/bb895340), [Administración extensible de claves con Azure Key Vault (SQL Server)](https://msdn.microsoft.com/library/dn198405) |
| **Pasos** | La Administración extensible de claves de SQL Server permite que las claves de cifrado que protegen los archivos de base de datos se almacenen en un dispositivo externo, como una tarjeta inteligente, un dispositivo USB o un módulo EKM/HSM. Esto también habilita la protección de datos para los administradores de bases de datos (exceptuando a los miembros del grupo de sysadmin). Los datos se pueden cifrar utilizando claves de cifrado a las que solo tiene acceso el usuario de la base de datos en el módulo EKM/HSM externo. |

## <a name="use-alwaysencrypted-feature-if-encryption-keys-should-not-be-revealed-to-database-engine"></a><a id="keys-engine"></a>Use la característica AlwaysEncrypted si las claves de cifrado no deben mostrarse al motor de base de datos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | SQL Azure, OnPrem |
| **Atributos**              | Versión de SQL: V12, MsSQL2016 |
| **Referencias**              | [Always Encrypted (motor de base de datos)](https://msdn.microsoft.com/library/mt163865) |
| **Pasos** | Always Encrypted es una característica creada para proteger la información confidencial, como números de tarjetas de crédito o números de identificación nacionales (por ejemplo, números de la seguridad social de EE. UU.), almacenados en bases de datos de Azure SQL Database o SQL Server. Always Encrypted permite a los clientes cifrar información confidencial dentro de aplicaciones cliente y no revelar las claves de cifrado al motor de base de datos (SQL Database o SQL Server). En consecuencia, Always Encrypted proporciona una separación entre quienes poseen los datos (y pueden verlos) y quienes administran los datos (pero no deben tener acceso a los mismos) |

## <a name="store-cryptographic-keys-securely-on-iot-device"></a><a id="keys-iot"></a>Almacene las claves criptográficas de forma segura en dispositivos IoT

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dispositivo IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Sistema operativo del dispositivo: Windows IoT Core, Conectividad de dispositivos: SDK de dispositivo IoT de Azure |
| **Referencias**              | [TPM on Windows IoT Core](https://developer.microsoft.com/windows/iot/docs/tpm) (TPM en Windows IoT Core), [Configuración de TPM en Windows IoT Core](https://docs.microsoft.com/windows/iot-core/secure-your-device/setuptpm), [TPM con SDK de dispositivo IoT de Azure](https://github.com/Azure/azure-iot-hub-vs-cs/wiki/Device-Provisioning-with-TPM) |
| **Pasos** | Claves privadas de certificado o simétricas en un almacenamiento protegido de hardware como chips TPM o de tarjeta inteligente. Windows 10 IoT Core es compatible con el usuario de un TPM y se pueden usar varios TPM compatibles: https://docs.microsoft.com/windows/iot-core/secure-your-device/tpm#discrete-tpm-dtpm. Se recomienda usar un TPM discreto o de firmware. Un TPM de software solo se debe usar con fines de desarrollo y prueba. Una vez que un TPM está disponible y se aprovisiona con las claves, se debe escribir el código que genera el token sin codificar de forma rígida ninguna información confidencial en él. | 

### <a name="example"></a>Ejemplo
```
TpmDevice myDevice = new TpmDevice(0);
// Use logical device 0 on the TPM 
string hubUri = myDevice.GetHostName(); 
string deviceId = myDevice.GetDeviceId(); 
string sasToken = myDevice.GetSASToken(); 

var deviceClient = DeviceClient.Create( hubUri, AuthenticationMethodFactory. CreateAuthenticationWithToken(deviceId, sasToken), TransportType.Amqp); 
```
Como se puede ver, la clave principal de dispositivo no está presente en el código. En su lugar, se almacena en el TPM en la ranura 0. El dispositivo TPM genera un token SAS de corta duración que se utiliza para conectar con IoT Hub. 

## <a name="generate-a-random-symmetric-key-of-sufficient-length-for-authentication-to-iot-hub"></a><a id="random-hub"></a>Genere una clave simétrica aleatoria de longitud suficiente para la autenticación en IoT Hub

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de la nube de IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Elección de puerta de enlace: Azure IoT Hub |
| **Referencias**              | N/D  |
| **Pasos** | IoT Hub contiene un registro de identidad del dispositivo y durante el aprovisionamiento de un dispositivo genera automáticamente una clave simétrica aleatoria. Se recomienda utilizar esta característica de registro de identidad de Azure IoT Hub para generar la clave usada para la autenticación. IoT Hub también permite que se especifique una clave al crear el dispositivo. Si se genera una clave fuera de IoT Hub durante el aprovisionamiento de dispositivos, se recomienda crear una clave simétrica aleatoria de al menos de 256 bits. |

## <a name="ensure-a-device-management-policy-is-in-place-that-requires-a-use-pin-and-allows-remote-wiping"></a><a id="pin-remote"></a>Asegúrese de que hay una directiva de administración de dispositivos que requiere el uso de un PIN y permite el borrado remoto

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Cliente móvil de Dynamics CRM | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que hay una directiva de administración de dispositivos que requiere el uso de un PIN y permite el borrado remoto |

## <a name="ensure-a-device-management-policy-is-in-place-that-requires-a-pinpasswordauto-lock-and-encrypts-all-data-eg-bitlocker"></a><a id="bitlocker"></a>Asegúrese de que hay una directiva de administración de dispositivos que requiere un bloqueo automático con PIN o contraseña, y cifra todos los datos (por ejemplo, BitLocker)

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Cliente de Outlook de Dynamics CRM | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que hay una directiva de administración de dispositivos que requiere un bloqueo automático con PIN o contraseña, y cifra todos los datos (por ejemplo, BitLocker) |

## <a name="ensure-that-signing-keys-are-rolled-over-when-using-identity-server"></a><a id="rolled-server"></a>Asegúrese de que las claves de firma se sustituyen cuando se usa Identity Server

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Identity Server | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Identity Server: claves, firmas y criptografía](https://identityserver.github.io/Documentation/docsv2/configuration/crypto.html) |
| **Pasos** | Asegúrese de las claves de firma se sustituyen cuando se usa Identity Server. El vínculo en la sección de referencias explica cómo debe planearse esta acción sin causar interrupciones en las aplicaciones basadas en Identity Server. |

## <a name="ensure-that-cryptographically-strong-client-id-client-secret-are-used-in-identity-server"></a><a id="client-server"></a>Asegúrese de que se usan un identificador de cliente y un secreto de cliente de alta seguridad criptográfica en Identity Server

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Identity Server | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Asegúrese de que se usan un identificador de cliente y un secreto de cliente de alta seguridad criptográfica en Identity Server. Las siguientes directrices se deben usar al generar un identificador y un secreto de cliente:</p><ul><li>Generar un GUID aleatorio como el identificador de cliente</li><li>Generar una clave de 256 bits criptográficamente aleatoria como secreto</li></ul>|
