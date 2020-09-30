---
title: Administración de inscripciones de dispositivos con los SDK de Azure DPS
description: Administración de las inscripciones de dispositivos en IoT Hub Device Provisioning Service (DPS) con los SDK de servicio
author: robinsh
ms.author: robinsh
ms.date: 04/04/2018
ms.topic: conceptual
ms.service: iot-dps
ms.custom: fasttrack-edit, iot
services: iot-dps
ms.openlocfilehash: 1dc97f92e6139475d0d5ac5ea1201d6ff6b8d470
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90532331"
---
# <a name="how-to-manage-device-enrollments-with-azure-device-provisioning-service-sdks"></a>Cómo administrar inscripciones de dispositivos con los SDK del servicio Azure Device Provisioning
La *inscripción de un dispositivo* crea un registro de un único dispositivo o de grupo de ellos que, en algún momento, pueden registrarse en el servicio Device Provisioning. El registro de inscripción contiene la configuración inicial preferida para los dispositivos como parte de la inscripción, incluida la instancia de IoT. En este artículo se muestra cómo administrar inscripciones de dispositivos para el servicio de aprovisionamiento mediante programación con los SDK del servicio de aprovisionamiento de Azure IoT.  Los SDK están disponibles en GitHub en el mismo repositorio que los SDK de Azure IoT.

## <a name="prerequisites"></a>Prerequisites
* Obtenga la cadena de conexión de la instancia del servicio Device Provisioning.
* Obtenga los artefactos de seguridad del dispositivo del [mecanismo de atestación](concepts-service.md#attestation-mechanism) correspondiente:
    * [**Módulo de plataforma segura (TPM)** ](/azure/iot-dps/concepts-security#trusted-platform-module):
        * Inscripción individual: identificador de registro y clave de aprobación de TPM desde el dispositivo físico o el simulador de TPM.
        * El grupo de inscripción no se aplica a la atestación de TPM.
    * [**X.509**](/azure/iot-dps/concepts-security):
        * Inscripción individual: el [certificado de hoja](/azure/iot-dps/concepts-security) del dispositivo físico o el emulador de [DICE](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/) del SDK.
        * Grupo de inscripción: el [certificado raíz/de entidad de certificación](/azure/iot-dps/concepts-security#root-certificate) o el [certificado intermedio](/azure/iot-dps/concepts-security#intermediate-certificate), que se usan para generar el certificado de dispositivo en un dispositivo físico.  También puede generarse desde el emulador de DICE del SDK.
* Las llamadas API exactas pueden ser diferentes debido a diferencias del lenguaje. Revise los ejemplos que se proporcionan en GitHub para más información:
   * [Ejemplos de cliente del servicio de aprovisionamiento de Java](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-samples)
   * [Ejemplos de cliente del servicio de aprovisionamiento de Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/service/samples)
   * [Ejemplos de cliente del servicio de aprovisionamiento de .NET](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/provisioning/service/samples)

## <a name="create-a-device-enrollment"></a>Creación de una inscripción de dispositivos
Hay dos maneras de inscribir los dispositivos con el servicio de aprovisionamiento:

* Un **grupo de inscripción** es una entrada para un grupo de dispositivos que comparten un mecanismo de atestación común de certificados X.509, firmados por el [certificado raíz](https://docs.microsoft.com/azure/iot-dps/concepts-security#root-certificate) o el [certificado intermedio](https://docs.microsoft.com/azure/iot-dps/concepts-security#intermediate-certificate). Se recomienda usar un grupo de inscripción para un gran número de dispositivos que compartan la configuración inicial deseada o para dispositivos que vayan todos al mismo inquilino. Tenga en cuenta que solo se pueden inscribir los dispositivos que usen el mecanismo de atestación de X.509 como *grupos de inscripción*. 

    Puede crear un grupo de inscripción con los SDK mediante este flujo de trabajo:

    1. En el grupo de inscripción, el mecanismo de atestación emplea el certificado raíz X.509.  Llame a la API ```X509Attestation.createFromRootCertificate``` del SDK de servicio con el certificado raíz para crear la atestación de la inscripción.  El certificado raíz X.509 se proporciona en un archivo PEM o como una cadena.
    1. Cree una nueva variable ```EnrollmentGroup``` mediante el elemento ```attestation``` creado y un valor único de ```enrollmentGroupId```.  También tiene la opción de definir parámetros, como ```IoTHubHostName```, ```ProvisioningStatus```.
    2. Llame a la API ```createOrUpdateEnrollmentGroup``` del SDK de servicio de la aplicación back-end con ```EnrollmentGroup``` para crear un grupo de inscripción.

* Una **inscripción individual** es una entrada para el registro de un único dispositivo. Las inscripciones individuales pueden usar certificados X.509 o tokens de SAS (de un módulo de plataforma segura físico o virtual) como mecanismos de atestación. Se recomiendan las inscripciones individuales para los dispositivos que requieran configuraciones iniciales únicas o para los dispositivos que solo se puedan utilizar tokens de SAS a través de módulos de plataforma segura físicos o virtuales como mecanismo de atestación. En las inscripciones individuales se puede especificar el identificador de dispositivo del centro de IoT deseado.

    Puede crear una inscripción individual con los SDK mediante este flujo de trabajo:
    
    1. Elija su mecanismo de ```attestation```, que puede ser TPM o X.509.
        1. **TPM**: puede llamar a la API ```TpmAttestation``` del SDK de servicio usando como entrada la clave de aprobación desde un dispositivo físico o el emulador de TPM, para crear la atestación para la inscripción. 
        2. **X.509**: puede llamar a la API del SDK de servicio ```X509Attestation.createFromClientCertificate``` con el certificado de cliente como entrada para crear la atestación de la inscripción.
    2. Cree una nueva variable ```IndividualEnrollment``` con el mecanismo de ```attestation``` creado y una entrada de ```registrationId```, que se encuentra en el dispositivo o que se ha generado con el simulador de TPM.  También tiene la opción de definir parámetros como ```Device ID```, ```IoTHubHostName``` o ```ProvisioningStatus```.
    3. Llame a la API ```createOrUpdateIndividualEnrollment``` del SDK de servicio de la aplicación back-end con ```IndividualEnrollment``` para crear una inscripción individual.

Después de haber creado correctamente una inscripción, el servicio Device Provisioning devolverá el resultado de la inscripción. Este flujo de trabajo se muestra en los ejemplo [mencionados anteriormente](#prerequisites).

## <a name="update-an-enrollment-entry"></a>Actualización de una entrada de inscripción

Después de haber creado una entrada de inscripción, puede que desee actualizar la inscripción.  Posibles escenarios incluyen actualizar la propiedad deseada, actualizar el método de atestación o revocar el acceso a los dispositivos.  Existen distintas API para inscripciones individuales y de grupo, pero no hay distinción para los mecanismos de atestación.

Puede actualizar una entrada de inscripción mediante este flujo de trabajo:
* **Inscripción individual**:
    1. Obtenga primero la inscripción más reciente del servicio de aprovisionamiento con la API ```getIndividualEnrollment``` del SDK de servicio.
    2. Modifique el parámetro de la inscripción más reciente, según sea necesario. 
    3. Mediante la inscripción más reciente, llame a la API ```createOrUpdateIndividualEnrollment``` del SDK de servicio para actualizar la entrada de inscripción.
* **Inscripción de grupo**:
    1. Obtenga primero la inscripción más reciente del servicio de aprovisionamiento con la API ```getEnrollmentGroup``` del SDK de servicio.
    2. Modifique el parámetro de la inscripción más reciente, según sea necesario.
    3. Mediante la inscripción más reciente, llame a la API ```createOrUpdateEnrollmentGroup``` del SDK de servicio para actualizar la entrada de inscripción.

Este flujo de trabajo se muestra en los ejemplo [mencionados anteriormente](#prerequisites).

## <a name="remove-an-enrollment-entry"></a>Eliminación de la entrada de una inscripción

* La **inscripción individual** se puede eliminar mediante la llamada a la API ```deleteIndividualEnrollment``` del SDK de servicio con ```registrationId```.
* La **inscripción de grupo** se puede eliminar mediante la llamada a la API ```deleteEnrollmentGroup``` del SDK de servicio con ```enrollmentGroupId```.

Este flujo de trabajo se muestra en los ejemplo [mencionados anteriormente](#prerequisites).

## <a name="bulk-operation-on-individual-enrollments"></a>Operación masiva sobre inscripciones individuales

Puede realizar operaciones masivas para crear, actualizar o eliminar varias inscripciones individuales mediante este flujo de trabajo:

1. Cree una variable que contenga varios elementos ```IndividualEnrollment```.  La implementación de esta variable es diferente en cada lenguaje.  Revise el ejemplo de operación masiva en GitHub para más información.
2. Llame a la API ```runBulkOperation``` del SDK de servicio con ```BulkOperationMode``` para realizar la operación deseada e implementar su variable para inscripciones individuales. Se admiten cuatro modos: crear, actualizar, updateIfMatchEtag y eliminar.

Después de haber realizado correctamente una operación, el servicio Device Provisioning devolvería el resultado de una operación masiva.

Este flujo de trabajo se muestra en los ejemplo [mencionados anteriormente](#prerequisites).
