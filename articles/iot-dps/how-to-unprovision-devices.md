---
title: Desaprovisionamiento de dispositivos aprovisionados mediante el servicio Azure IoT Hub Device Provisioning
description: Desaprovisionamiento de dispositivos aprovisionados mediante el servicio Azure IoT Hub Device Provisioning Service (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 05/11/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: efd19d9dc1fdc857a51d7af34c54bbe2d6077767
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294397"
---
# <a name="how-to-deprovision-devices-that-were-previously-auto-provisioned"></a>Desaprovisionamiento de dispositivos aprovisionados automáticamente 

Puede que sea necesario desaprovisionar los dispositivos que se hayan aprovisionado automáticamente mediante el servicio Device Provisioning. Por ejemplo, un dispositivo puede venderse, trasladarse a otra instancia de IoT Hub, perderse, robarse o verse en peligro de otro modo. 

En general, el desaprovisionamiento de un dispositivo implica dos pasos:

1. Dé de baja el dispositivo del servicio de aprovisionamiento para evitar el aprovisionamiento automático en el futuro. Puede deshabilitar o eliminar la entrada de inscripción, dependiendo de si desea denegar el acceso de manera temporal o permanente. Para los dispositivos que usan la atestación de X.509, es aconsejable deshabilitar/eliminar una entrada en la jerarquía de los grupos de inscripción existentes.  
 
   - Para aprender a dar de baja un dispositivo, consulte [Baja del servicio Azure IoT Hub Device Provisioning para un dispositivo](how-to-revoke-device-access-portal.md).
   - Para información sobre cómo dar de baja un dispositivo mediante programación con uno de los SDK del servicio de aprovisionamiento, consulte el artículo sobre la [administración de las inscripciones de dispositivos con los SDK de servicio](how-to-manage-enrollments-sdks.md).

2. Anule el registro del dispositivo de la instancia de IoT Hub para evitar la transferencia de datos y la comunicación en el futuro. De nuevo, deshabilite temporalmente o elimine la entrada del dispositivo en el registro de identidades de manera definitiva para la instancia de IoT Hub en la que se aprovisionó. Consulte [Deshabilitación de dispositivos](/azure/iot-hub/iot-hub-devguide-identity-registry#disable-devices) para más información sobre la deshabilitación. En [Azure Portal](https://portal.azure.com), consulte "Device Management / IoT Devices" (Administración de dispositivos /Dispositivos IoT) para el recurso de IoT Hub.

Los pasos exactos que se realizan para el desaprovisionamiento de un dispositivo dependen del mecanismo de atestación y de la entrada de inscripción aplicable en el servicio de aprovisionamiento. En las secciones siguientes se ofrece una introducción al proceso basada en el tipo de inscripción y la atestación.

## <a name="individual-enrollments"></a>Inscripciones individuales
Los dispositivos que usan la atestación de TPM o la atestación de X.509 con un certificado de hoja se aprovisionan mediante una entrada de inscripción individual. 

Para desaprovisionar un dispositivo con inscripción individual, siga estos pasos: 

1. Dé de baja el dispositivo del servicio de aprovisionamiento:

   - En el caso de dispositivos que usan la atestación de TPM, elimine la entrada de inscripción individual para revocar el acceso del dispositivo al servicio de aprovisionamiento de manera definitiva o deshabilite la entrada para revocarlo temporalmente. 
   - En dispositivos que usan la atestación de X.509, puede eliminar o deshabilitar la entrada. Sin embargo, tenga en cuenta que si elimina una inscripción individual de un dispositivo que usa X.509 y existe un grupo de inscripción habilitado para un certificado de firma en la cadena de certificados de ese dispositivo, el dispositivo se puede volver a inscribir. En estos dispositivos, puede ser más seguro deshabilitar la entrada de inscripción. De esta forma, evita que el dispositivo se vuelva a inscribir, con independencia de si existe un grupo de inscripción habilitado para uno de sus certificados de firma.

2. Deshabilite o elimine el dispositivo del registro de identidades de la instancia de IoT Hub en que se aprovisionó. 


## <a name="enrollment-groups"></a>Grupos de inscripción
Con la atestación X.509, también se pueden aprovisionar los dispositivos a través de un grupo de inscripción. Los grupos de inscripción se configuran con un certificado de firma, ya sea un certificado de entidad de certificación raíz o intermedia, y controlan el acceso al servicio de aprovisionamiento para los dispositivos con ese certificado en su cadena de certificados. Para más información sobre los grupos de inscripción y los certificados X.509 en el servicio de aprovisionamiento, consulte [Certificados X.509](concepts-security.md#x509-certificates). 

Para ver una lista de dispositivos que se han aprovisionado a través de un grupo de inscripción, puede visualizar los detalles del grupo de inscripción. Se trata de una manera fácil de comprender a qué instancia de IoT Hub se ha aprovisionado cada dispositivo. Para ver la lista de dispositivos: 

1. Inicie sesión en Azure Portal y haga clic en **Todos los recursos** en el menú de la izquierda.
2. Haga clic en el servicio de aprovisionamiento en la lista de recursos.
3. En el servicio de aprovisionamiento, haga clic en **Manage enrollments** (Administrar inscripciones) y seleccione la pestaña **Enrollment Groups** (Grupos de inscripción).
4. Haga clic en el grupo de inscripción para abrirlo.

   ![Ver la entrada del grupo de inscripción en el portal](./media/how-to-unprovision-devices/view-enrollment-group.png)

Hay dos escenarios que se deben tener en cuenta en relación con los grupos de inscripción:

- Para desaprovisionar todos los dispositivos que se han aprovisionado mediante un grupo de inscripción, siga estos pasos:
  1. Deshabilite el grupo de inscripción para no permitir su certificado de firma. 
  2. Use la lista de dispositivos aprovisionados de ese grupo de inscripción para deshabilitar o eliminar todos los dispositivos del registro de identidades de su respectiva instancia de IoT Hub. 
  3. Después de deshabilitar o eliminar todos los dispositivos de sus respectivas instancias de IoT Hub, puede eliminar opcionalmente el grupo de inscripción. Sin embargo, tenga en cuenta que, si elimina el grupo de inscripción y hay un grupo de inscripción habilitado para un certificado de firma más arriba en la cadena de certificados de uno o varios de los dispositivos, estos dispositivos se podrán volver a inscribir. 

- Para desaprovisionar un único dispositivo de un grupo de inscripción, siga estos pasos:
  1. Cree una inscripción individual deshabilitada para su certificado de hoja (dispositivo). Esto revoca el acceso al servicio de aprovisionamiento para dicho dispositivo mientras se sigue permitiendo el acceso para otros dispositivos que tienen el certificado de firma del grupo de inscripción en su cadena. No elimine la inscripción individual deshabilitada para el dispositivo. Si lo hace, permitirá al dispositivo volver a inscribirse a través del grupo de inscripción. 
  2. Use la lista de dispositivos aprovisionados para ese grupo de inscripción para buscar la instancia de IoT Hub en que se ha aprovisionado el dispositivo y deshabilitarlo o eliminarlo del registro de identidades de dicha instancia. 
