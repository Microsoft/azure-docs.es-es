---
title: Información general sobre la seguridad mediante la entidad de certificación X.509 de Azure IoT Hub | Microsoft Docs
description: 'Información general: cómo autenticar los dispositivos en IoT Hub con entidades de certificación X.509.'
author: eustacea
manager: arjmands
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 09/18/2017
ms.author: eustacea
ms.custom:
- 'Role: Cloud Development'
- 'Role: IoT Device'
- 'Role: System Architecture'
ms.openlocfilehash: 33c878b2af64a72212a8aac060e9ccac83b79a53
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326518"
---
# <a name="device-authentication-using-x509-ca-certificates"></a>Autenticación de dispositivos mediante certificados de entidades de certificación X.509

En este artículo se describe cómo usar los certificados de las entidades de certificación X.509 para autenticar los dispositivos conectados a IoT Hub.  En este artículo, encontrará la siguiente información:

* Cómo obtener un certificado de entidad de certificación X.509
* Cómo registrar el certificado de entidad de certificación X.509 en IoT Hub
* Cómo firmar dispositivos con certificado de entidad de certificación X.509
* Cómo autenticar los dispositivos firmados con entidades de certificación X.509

## <a name="overview"></a>Información general

La característica de la entidad de certificación X.509 habilita la autenticación de dispositivos en IoT Hub mediante una entidad de certificación. Simplifica significativamente el proceso de inscripción inicial del dispositivo y la logística de la cadena de suministro durante la fabricación del dispositivo. [Obtenga más información en este artículo de escenarios sobre el valor del uso de los certificados de entidades de certificación X.509](iot-hub-x509ca-concept.md) para la autenticación de dispositivos.  Recomendamos que lea este artículo de escenarios antes de continuar, ya que se explica por qué existen los pasos siguientes.

## <a name="prerequisite"></a>Requisito previo

Para utilizar la característica de entidades de certificación X.509, es necesario tener una cuenta de IoT Hub.  [Obtenga información sobre cómo crear una instancia de IoT Hub](quickstart-send-telemetry-dotnet.md) si aún no tiene ninguna.

## <a name="how-to-get-an-x509-ca-certificate"></a>Cómo obtener un certificado de entidad de certificación X.509

El certificado de entidad de certificación X.509 está en la parte superior de la cadena de certificados de cada dispositivo.  Puede comprar o crear uno en función de cómo pretenda usarlo.

Para un entorno de producción, se recomienda adquirir un certificado de entidades de certificación X.509 de una entidad de certificación raíz pública. Adquirir un certificado de entidad de certificación tiene la ventaja de que la entidad de certificación raíz actúa como un tercero de confianza para garantizar la legitimidad de los dispositivos. Tenga en cuenta esta opción si pretende que los dispositivos formen parte de una red de IoT abierta donde se espera que interactúen con productos o servicios de terceros.

También puede crear un certificado autofirmado de entidad de certificación X.509 para experimentación o para usarla en redes de IoT cerradas.

Independientemente de cómo obtenga el certificado de entidad de certificación X.509, asegúrese de que la clave privada correspondiente es secreta y está protegida en todo momento.  Esto es necesario para fomentar la confianza en la autenticación con la entidad de certificación X.509.

Obtenga información sobre cómo [crear un certificado autofirmado de entidad de certificación](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md), que puede usar para experimentación a través de la descripción de esta característica.

## <a name="sign-devices-into-the-certificate-chain-of-trust"></a>Registro de dispositivos en la cadena de certificados de confianza

El propietario de un certificado de entidad de certificación X.509 puede firmar criptográficamente una entidad de certificación intermedia que a su vez puede firmar otra entidad de certificación intermedia, y así sucesivamente, hasta que la última entidad de certificación intermedia termina este proceso mediante la firma de un dispositivo. El resultado es una cadena de certificados en cascada conocida como una cadena de certificados de confianza. En la vida real esto se desarrolla como la delegación de confianza en los dispositivos de firma. Esta delegación es importante porque establece una cadena de custodia criptográficamente variable y evita el uso compartido de las claves de firma.

![img-generic-cert-chain-of-trust](./media/generic-cert-chain-of-trust.png)

El certificado del dispositivo (también denominado certificado de hoja) tiene que tener *Nombre de sujeto* establecido en el **Id. de dispositivo** (`CN=deviceId`) que se usó al registrar el dispositivo IoT en Azure IoT Hub. Esta opción de configuración es necesaria para la autenticación.

Obtenga información aquí sobre cómo [crear una cadena de certificados](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) tal y como se hace cuando se firman los dispositivos.

## <a name="how-to-register-the-x509-ca-certificate-to-iot-hub"></a>Cómo registrar el certificado de entidad de certificación X.509 en IoT Hub

Registre el certificado de entidad de certificación X.509 en IoT Hub, donde se utilizará para autenticar los dispositivos durante el registro y la conexión.  El registro del certificado de entidad de certificación X.509 es un proceso de dos pasos que comprende la carga del archivo de certificado y la prueba de posesión.

El proceso de carga implica cargar un archivo que contiene el certificado.  Este archivo no debe contener nunca las claves privadas.

El paso de la prueba de posesión implica un desafío criptográfico y un proceso de respuesta entre el usuario y IoT Hub.  Dado que el contenido de los certificados digitales es público y, por tanto, susceptible a interceptaciones, es conveniente que IoT Hub confirme que realmente posee el certificado de entidad de certificación.  Esto se debe hacer mediante la generación de un desafío aleatorio que se debe firmar con la clave privada correspondiente del certificado de entidad de certificación.  Si mantiene la clave privada en secreto y protegida como se aconsejó anteriormente, lo único que necesitará son los conocimientos necesarios para completar este paso. La confidencialidad de las claves privadas es el origen de confianza en este método.  Después de firmar el desafío, complete este paso mediante la carga de un archivo que contiene los resultados.

Obtenga información aquí sobre cómo [registrar su certificado de entidad de certificación](iot-hub-security-x509-get-started.md#register-x509-ca-certificates-to-your-iot-hub)

## <a name="how-to-create-a-device-on-iot-hub"></a>Cómo crear un dispositivo en IoT Hub

Para impedir la suplantación del dispositivo, IoT Hub requiere que le permita saber qué dispositivos debe esperar.  Para ello, cree una entrada de dispositivo en el dispositivo de IoT Hub.  Este proceso es automático cuando se usa el IoT Hub [Device Provisioning Service](https://azure.microsoft.com/blog/azure-iot-hub-device-provisioning-service-preview-automates-device-connection-configuration/). 

Obtenga información aquí sobre cómo [crear manualmente un dispositivo en IoT Hub](iot-hub-security-x509-get-started.md#create-an-x509-device-for-your-iot-hub).

Creación de un dispositivo X.509 para una instancia de IoT Hub

## <a name="authenticating-devices-signed-with-x509-ca-certificates"></a>Autenticación de dispositivos firmados con certificados de entidad de certificación X.509

Con el certificado de entidad de certificación X.509 registrado y los dispositivos registrados en una cadena de certificados de confianza, lo que falta es la autenticación del dispositivo cuando este se conecta, incluso por primera vez.  Cuando un dispositivo firmado con la entidad de certificación X.509, carga su cadena de certificados para la validación. La cadena incluye los certificados del dispositivo y la entidad de certificación intermedia.  Con esta información, IoT Hub autentica el dispositivo en un proceso de dos pasos.  IoT Hub valida criptográficamente la cadena de certificados para comprobar la coherencia interna y, a continuación, emite un desafío de prueba de posesión en el dispositivo.  IoT Hub declara que el dispositivo es auténtico en una respuesta de prueba de posesión satisfactoria del dispositivo.  Esta declaración asume que la clave privada del dispositivo está protegida y que solo el dispositivo puede responder correctamente a este desafío.  Se recomienda utilizar chips seguros como módulos de seguridad de hardware (HSM) en los dispositivos para proteger las claves privadas.

Una conexión de dispositivo correcta a IoT Hub completa el proceso de autenticación y también es indicativa de una configuración apropiada.

Obtenga información aquí sobre cómo [completar este paso para la conexión del dispositivo](iot-hub-security-x509-get-started.md#authenticate-your-x509-device-with-the-x509-certificates).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre [el valor de la autenticación de la entidad de certificación X.509](iot-hub-x509ca-concept.md) en IoT.

Introducción al [Servicio IoT Hub Device Provisioning](https://docs.microsoft.com/azure/iot-dps/).
