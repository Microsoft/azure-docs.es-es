---
title: Inscripción de certificados X.509 en Azure IoT Central
description: Inscripción de certificados X.509 en la aplicación de IoT Central
author: v-krghan
ms.author: v-krghan
ms.date: 07/31/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 76e2b9542d20b5788a2875dec89d447ce38276a0
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121967"
---
# <a name="how-to-roll-x509-device-certificates-in-iot-central-application"></a>Inscripción de certificados de dispositivo X.509 en la aplicación de IoT Central

Durante el ciclo de vida de la solución de IoT, necesitará implementar certificados. Dos de las razones principales para implementar certificados sería una infracción de seguridad y la caducidad de los certificados. 

La implementación de certificados es un procedimiento recomendado de seguridad para ayudar a proteger el sistema en caso de una infracción de seguridad. Como parte de la [asunción de la metodología de infracción de seguridad](https://download.microsoft.com/download/C/1/9/C1990DBA-502F-4C2A-848D-392B93D9B9C3/Microsoft_Enterprise_Cloud_Red_Teaming.pdf), Microsoft recomienda la necesidad de contar con procesos de seguridad reactivos junto con medidas preventivas. La implementación de los certificados de dispositivo se debe incluir como parte de estos procesos de seguridad. La frecuencia con la que implementa los certificados dependerá de las necesidades de seguridad de la solución. Los clientes con soluciones que involucran datos altamente confidenciales pueden implementar el certificado diariamente, mientras que otros usuarios implementan sus certificados cada dos años.


## <a name="obtain-new-x509-certificates"></a>Obtención de nuevos certificados X.509

Use una herramienta como OpenSSL para crear sus propios certificados X.509. Este enfoque es muy útil para probar los certificados X.509, pero proporciona pocas garantías para la seguridad. Use este enfoque solo para las pruebas, a menos que se prepare para actuar como proveedor de entidad de certificación propio.


## <a name="enrollment-groups-and-security-breaches"></a>Grupos de inscripción e infracciones de seguridad

Para actualizar un grupo de inscripción en respuesta a una infracción de seguridad, debe usar el siguiente enfoque, que actualiza inmediatamente el certificado actual:

1. Vaya a **Administration** (Administración) en el panel izquierdo y haga clic en **Device connection** (Conexión del dispositivo).

2. Haga clic en **Grupos de inscripción** y luego haga clic en el nombre de grupo en la lista.

    ![Conexión del dispositivo](./media/how-to-roll-x509-certificates/device-connection.png)


3. Para la actualización de certificados, haga clic en **Administrar principal** o en **Administrar secundario**.

    ![Administración de certificados](./media/how-to-roll-x509-certificates/certificates.png)


4. Agregue y verifique un certificado X.509 raíz en el grupo de inscripción.

   Complete estos pasos para el certificado principal y el secundario, si ambos están en riesgo.



## <a name="enrollment-groups-and-certificate-expiration"></a>Grupos de inscripciones y expiración de certificados

Si va a inscribir certificados para controlar sus fechas de expiración, actualice el certificado actual inmediatamente mediante el siguiente enfoque:

1. Vaya a **Administration** (Administración) en el panel izquierdo y haga clic en **Device connection** (Conexión del dispositivo). 

2. Haga clic en **Grupos de inscripción** y luego haga clic en el nombre de grupo en la lista.

    ![Conexión del dispositivo](./media/how-to-roll-x509-certificates/device-connection.png)


3. Para la actualización del certificado, haga clic en **Administrar principal**.

    ![Conexión del dispositivo](./media/how-to-roll-x509-certificates/manage-certs.png)

4. Agregue y verifique un certificado X.509 raíz en el grupo de inscripción.

5. Más adelante, cuando el certificado secundario haya expirado, vuelva y actualícelo.



## <a name="individual-enrollments-and-security-breaches"></a>Inscripciones individuales e infracciones de seguridad

Si está inscribiendo certificados en respuesta a una infracción de seguridad, use el siguiente enfoque para actualizar inmediatamente el certificado actual:


1. Haga clic en **Dispositivos** y seleccione el dispositivo. 

2. Haga clic en **Conectar** y seleccione el método de conexión **Individual Enrollment** (Inscripción individual).

3. Seleccione el mecanismo **Certificados (X.509)** .

    ![Administración de inscripciones individuales](./media/how-to-roll-x509-certificates/certificate-update.png)

4. Para actualizar un certificado, haga clic en el icono de carpeta para seleccionar el nuevo certificado que se va a cargar para la entrada de inscripción. Haga clic en **Save**(Guardar).

    Complete estos pasos para el certificado principal y el secundario, si ambos están en riesgo.



## <a name="individual-enrollments-and-certificate-expiration"></a>Inscripciones individuales y expiración de certificados

Si está implementando certificados para controlar las expiraciones de los certificados, debe usar la configuración del certificado secundario como se indica a continuación para reducir el tiempo de inactividad de los dispositivos que intentan aprovisionar.

Posteriormente, cuando se aproxima la expiración del certificado secundario y necesita implementarse, puede pasar al uso de la configuración principal. La rotación entre los certificados principales y secundarios de este modo reduce el tiempo de inactividad de los dispositivos que intentan aprovisionar.

1. Haga clic en **Dispositivos** y seleccione el dispositivo.

2. Haga clic en **Conectar** y seleccione el método de conexión **Individual Enrollment** (Inscripción individual).

3. Seleccione el mecanismo **Certificados (X.509)** .

    ![Administración de inscripciones individuales](./media/how-to-roll-x509-certificates/certificate-update.png)

4. Para actualizar un certificado secundario, haga clic en el icono de carpeta para seleccionar el nuevo certificado que se va a cargar para la entrada de inscripción. Haga clic en **Save**(Guardar).


5. Más adelante, cuando el certificado principal haya expirado, vuelva y actualícelo.


## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido inscribir certificados X.509 en la aplicación de Azure IoT Central, puede realizar la [Conexión a Azure IoT Central](concepts-get-connected.md).


