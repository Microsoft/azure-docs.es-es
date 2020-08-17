---
title: 'Protección de la comunicación de dispositivos OPC UA con OPC Vault: Azure | Microsoft Docs'
description: En este artículo se explica cómo registrar las aplicaciones OPC UA y cómo emitir certificados de aplicación firmados para los dispositivos OPC UA con OPC Vault.
author: mregen
ms.author: mregen
ms.date: 8/16/2018
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: caa3717fdf915e205c1a4f82f4e199346ca29fcb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082823"
---
# <a name="use-the-opc-vault-certificate-management-service"></a>Uso del servicio de administración de certificados de OPC Vault

En este artículo se explica cómo registrar las aplicaciones y cómo emitir certificados de aplicación firmados para los dispositivos OPC UA.

## <a name="prerequisites"></a>Prerrequisitos

### <a name="deploy-the-certificate-management-service"></a>Implementar el servicio de administración de certificados

En primer lugar, implemente el servicio en la nube de Azure. Para más información, consulte [Compilación e implementación del servicio de administración de certificados de OPC Vault](howto-opc-vault-deploy.md).

### <a name="create-the-issuer-ca-certificate"></a>Crear el certificado de entidad de certificación del emisor

Si todavía no lo ha hecho, cree el certificado de entidad de certificación del emisor. Para más información, consulte [Creación y administración del certificado del emisor para OPC Vault](howto-opc-vault-manage.md).

## <a name="secure-opc-ua-applications"></a>Protección de las aplicaciones OPC UA

### <a name="step-1-register-your-opc-ua-application"></a>Paso 1: registro de la aplicación OPC UA 

> [!IMPORTANT]
> El rol Escritor es necesario para registrar una aplicación.

1. Abra el servicio de certificados en `https://myResourceGroup-app.azurewebsites.net` e inicie sesión.
2. Vaya a **Register New** (Registrar nuevo). Para un registro de aplicación, un usuario debe tener asignado al menos el rol Escritor.
2. El formulario de entrada sigue las convenciones de nomenclatura de OPC UA. Como ejemplo, en la siguiente captura de pantalla, se muestra la configuración del ejemplo de [servidor de referencia de OPC UA](https://github.com/OPCFoundation/UA-.NETStandard/tree/master/Applications/ReferenceServer) en OPC UA en la pila .NET Standard:

   ![Captura de pantalla del Registro del servidor de referencia de UA](media/howto-opc-vault-secure/reference-server-registration.png "Registro del servidor de referencia de UA")

5. Seleccione **Register** (Registrar) para registrar la aplicación en la base de datos de aplicación del servicio de certificados. El flujo de trabajo guía directamente al usuario al paso siguiente para solicitar un certificado firmado para la aplicación.

### <a name="step-2-secure-your-application-with-a-ca-signed-application-certificate"></a>Paso 2: protección de la aplicación con un certificado de aplicación firmado por una entidad de certificación

Proteja la aplicación OPC UA mediante la emisión de un certificado firmado basado en una solicitud de firma de certificado (CSR). Como alternativa, puede solicitar un nuevo par de claves, que incluye una nueva clave privada en formato PFX o PEM. Para obtener información sobre qué método es compatible con su aplicación, consulte la documentación del dispositivo OPC UA. En general, se recomienda el método CSR, ya que no requiere la transferencia de una clave privada a través de una conexión.

#### <a name="request-a-new-certificate-with-a-new-keypair"></a>solicitud de un nuevo certificado con un par de claves nuevo

1. Vaya a **Applications** (Aplicaciones).
3. Seleccione **New Request** (Nueva solicitud) para una aplicación de la lista.

   ![Captura de pantalla de solicitud de nuevo certificado](media/howto-opc-vault-secure/request-new-certificate.png "Solicitud de nuevo certificado")

3. Seleccione **Request new KeyPair and Certificate** (Solicitar nuevo par de claves y certificado) para solicitar una clave privada y un nuevo certificado firmado con la clave pública para la aplicación.

   ![Captura de pantalla de generación de nuevo par de claves y certificado](media/howto-opc-vault-secure/generate-new-key-pair.png "Generación de nuevo par de claves")

4. Rellene el formulario con un asunto y los nombres de dominio. Para la clave privada, elija PEM o PFX con contraseña. Seleccione **Generate New KeyPair** (Generar nuevo par de claves) para crear la solicitud de certificado.

   ![Captura de pantalla de visualización detalles de solicitud de certificado](media/howto-opc-vault-secure/approve-reject.png "Aprobación de certificado")

5. La aprobación requiere un usuario con el rol Aprobador y con permisos de firma en Azure Key Vault. En el flujo de trabajo típico, los roles Aprobador y Solicitante deben estar asignados a distintos usuarios. Seleccione **Approve** (Aprobar) o **Reject** (Rechazar) para iniciar o cancelar la creación real del par de claves y la operación de firma. El nuevo par de claves se crea y se almacena de manera segura en Azure Key Vault hasta que el solicitante del certificado lo descarga. La entidad de certificación firma el certificado resultante con la clave pública. Estas operaciones pueden tardar unos segundos en finalizar.

   ![Captura de pantalla de visualización de detalles de solicitud de certificado, con mensaje de aprobación en la parte inferior](media/howto-opc-vault-secure/view-key-pair.png "Visualización de par de claves")

7. La clave privada (PFX o PEM) y el certificado (DER) resultantes se pueden descargar desde aquí en el formato seleccionado como descarga de archivos binarios. También está disponible una versión codificada en Base64, por ejemplo, para copiar y pegar el certificado en una línea de comandos o en una entrada de texto. 
8. Una vez que la clave privada se haya descargado y almacenado de forma segura, puede seleccionar **Delete Private Key** (Eliminar clave privada). El certificado con la clave pública sigue disponible para su uso futuro.
9. Debido al uso de un certificado firmado por una entidad de certificación, el certificado de entidad de certificación y la lista de revocación de certificados (CLR) también se deben descargar aquí.

Ahora la aplicación del nuevo par de claves depende del dispositivo OPC UA. Por lo general, el certificado de entidad de certificación y la CRL se copian en una carpeta `trusted`, mientras que la clave pública y la clave privada del certificado de aplicación se aplican a una carpeta `own` en el almacén de certificados. Es posible que algunos dispositivos ya admitan la inserción de servidor para las actualizaciones de certificados. Consulte la documentación del dispositivo OPC UA.

#### <a name="request-a-new-certificate-with-a-csr"></a>solicitud de un certificado nuevo con un CSR 

1. Vaya a **Applications** (Aplicaciones).
3. Seleccione **New Request** (Nueva solicitud) para una aplicación de la lista.

   ![Captura de pantalla de solicitud de nuevo certificado](media/howto-opc-vault-secure/request-new-certificate.png "Solicitud de nuevo certificado")

3. Seleccione **Request new Certificate with signing request** (Solicitar nuevo certificado con solicitud de firma) para solicitar un certificado firmado nuevo para la aplicación.

   ![Captura de pantalla de generación de nuevo certificado](media/howto-opc-vault-secure/generate-new-certificate.png "Generación de nuevo certificado")

4. Para cargar CSR, seleccione un archivo local o pegue un CSR con codificación Base64 en el formulario. Seleccione **Generate New Certificate** (Generar nuevo certificado).

   ![Captura de pantalla de visualización detalles de solicitud de certificado](media/howto-opc-vault-secure/approve-reject-csr.png "Aprobación de CSR")

5. La aprobación requiere un usuario con el rol Aprobador y con permisos de firma en Azure Key Vault. Seleccione **Approve** (Aprobar) o **Reject** (Rechazar) para iniciar o cancelar la operación de firma real. La entidad de certificación firma el certificado resultante con la clave pública. Esta operación puede tardar unos segundos en finalizar.

   ![Captura de pantalla de visualización de detalles de solicitud de certificado, con mensaje de aprobación en la parte inferior](media/howto-opc-vault-secure/view-cert-csr.png "Visualización de certificado")

6. El certificado resultante (DER) se puede descargar desde aquí como archivo binario. También está disponible una versión codificada en Base64, por ejemplo, para copiar y pegar el certificado en una línea de comandos o en una entrada de texto. 
10. Una vez que el certificado se haya descargado y almacenado de forma segura, puede seleccionar **Delete Certificate** (Eliminar certificado).
11. Debido al uso de un certificado firmado por una entidad de certificación, el certificado de entidad de certificación y la CLR también se deben descargar aquí.

Ahora la aplicación del nuevo certificado depende del dispositivo OPC UA. Por lo general, el certificado de entidad de certificación y la CRL se copian en una carpeta `trusted`, mientras que el certificado de aplicación se aplica a una carpeta `own` en el almacén de certificados. Es posible que algunos dispositivos ya admitan la inserción de servidor para las actualizaciones de certificados. Consulte la documentación del dispositivo OPC UA.

### <a name="step-3-device-secured"></a>Paso 3: dispositivo protegido

El dispositivo OPC UA ya está listo para comunicarse con otros dispositivos de OPC UA protegidos por certificados firmados por una entidad de certificación sin más configuraciones.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a proteger dispositivos OPC Vault, puede hacer lo siguiente:

> [!div class="nextstepaction"]
> [Ejecución de un servicio de administración de certificados seguro](howto-opc-vault-secure-ca.md)
