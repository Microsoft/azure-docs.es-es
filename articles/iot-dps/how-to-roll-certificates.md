---
title: Implementación de certificados X.509 en Azure IoT Hub Device Provisioning Service
description: Cómo implementar certificados X.509 con la instancia de Device Provisioning Service (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 08/06/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: c2bbfcb4832adba767750256a25c378356cf4c23
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299275"
---
# <a name="how-to-roll-x509-device-certificates"></a>Implementación de certificados de dispositivo X.509

Durante el ciclo de vida de la solución de IoT, necesitará implementar certificados. Dos de las razones principales para implementar certificados sería una infracción de seguridad y la caducidad de los certificados. 

La implementación de certificados es un procedimiento recomendado de seguridad para ayudar a proteger el sistema en caso de una infracción de seguridad. Como parte de la [asunción de la metodología de infracción de seguridad](https://download.microsoft.com/download/C/1/9/C1990DBA-502F-4C2A-848D-392B93D9B9C3/Microsoft_Enterprise_Cloud_Red_Teaming.pdf), Microsoft recomienda la necesidad de contar con procesos de seguridad reactivos junto con medidas preventivas. La implementación de los certificados de dispositivo se debe incluir como parte de estos procesos de seguridad. La frecuencia con la que implementa los certificados dependerá de las necesidades de seguridad de la solución. Los clientes con soluciones que involucran datos altamente confidenciales pueden implementar el certificado diariamente, mientras que otros usuarios implementan sus certificados cada dos años.

La implementación de certificados de dispositivo implica actualizar el certificado almacenado en el dispositivo y el centro de IoT. Después, el dispositivo puede reaprovisionarse así mismo con el centro de IoT mediante [autoaprovisionamiento](concepts-auto-provisioning.md) normal con Device Provisioning Service.


## <a name="obtain-new-certificates"></a>Obtención de nuevos certificados

Hay muchas maneras de obtener nuevos certificados para los dispositivos de IoT. Entre ellas se incluyen la obtención de certificados de la fábrica del dispositivo, generar sus propios certificados y hacer que un tercero administre la creación de certificados para usted. 

Los certificados están firmados entre sí para formar una cadena de confianza a partir de un certificado de entidad de certificación raíz para un [certificado de hoja](concepts-security.md#end-entity-leaf-certificate). Un certificado de firma es el certificado que se usa para firmar el certificado de hoja al final de la cadena de confianza. Un certificado de firma puede ser un certificado de entidad de certificación raíz o un certificado intermedio en la cadena de confianza. Para más información, vea [Certificados X.509](concepts-security.md#x509-certificates).
 
Hay dos maneras diferentes de obtener un certificado de firma. La primera, que se recomienda para los sistemas de producción, consiste en adquirir un certificado de firma de una entidad de certificación raíz (CA). Esta manera encadena la seguridad a un origen de confianza. 

La segunda consiste en crear sus propios certificados X.509 mediante una herramienta como OpenSSL. Este enfoque es muy útil para probar los certificados X.509, pero proporciona pocas garantías para la seguridad. Le recomendamos que utilice este enfoque solo para las pruebas a menos que se prepare para actuar como su propio proveedor de entidad de certificación.
 

## <a name="roll-the-certificate-on-the-device"></a>Implementación del certificado en el dispositivo

Los certificados de un dispositivo deben almacenarse siempre en un lugar seguro, como un [módulo de seguridad de hardware (HSM)](concepts-device.md#hardware-security-module). La manera de implementar los certificados de dispositivo dependerá de cómo se crearon e instalaron en los dispositivos en primer lugar. 

Si obtuvo los certificados de terceros, debe observar cómo implementan los certificados. El proceso puede estar incluido en sus acuerdos con ellos, o puede ser un servicio independiente que ofrecen. 

Si está administrando sus propios certificados de dispositivo, tendrá que crear su propia canalización para actualizar los certificados. Asegúrese de que los certificados de hoja antiguos y nuevos tienen el mismo nombre común (CN). Al tener el mismo CN, el dispositivo puede reaprovisionarse así mismo sin necesidad de crear un registro duplicado. 


## <a name="roll-the-certificate-in-the-iot-hub"></a>Implementación del certificado en el centro de IoT

Puede agregar manualmente el certificado del dispositivo a centro de IoT. El certificado también se puede automatizar mediante una instancia de servicio Device Provisioning. En este artículo, supondremos que una instancia de servicio Device Provisioning se utiliza para admitir el aprovisionamiento automático.

Cuando un dispositivo se ha aprovisionado inicialmente con el aprovisionamiento automático, arranca y se pone en contacto con el servicio de aprovisionamiento. El servicio de aprovisionamiento responde realizando una comprobación de identidad antes de crear una identidad de dispositivo en un centro de IoT mediante el certificado de hoja del dispositivo como la credencial. Después, el servicio de aprovisionamiento indica el dispositivo al que está asignado el centro de IoT y, después, usa su certificado de hoja para autenticarse y conectarse a dicho centro. 

Una vez que se ha implementado un nuevo certificado de hoja en el dispositivo, ya no puede conectarse al centro de IoT porque está usando un nuevo certificado para conectarse. El centro de IoT solo reconoce el dispositivo con el certificado antiguo. El resultado del intento de conexión del dispositivo será un error de conexión "no autorizado". Para resolver este error, debe actualizar la entrada de inscripción del dispositivo para tener en cuenta el nuevo certificado de hoja del dispositivo. A continuación, el servicio de aprovisionamiento puede actualizar la información de registro del dispositivo de IoT Hub según sea necesario cuando se vuelve a aprovisionar el dispositivo. 

Una posible excepción a este error de conexión sería un escenario donde ha creado un [grupo de inscripción](concepts-service.md#enrollment-group) para el dispositivo en el servicio de aprovisionamiento. En este caso, si no implementa los certificados raíz o intermedios en la cadena de certificados del dispositivo de confianza, el dispositivo se reconocerá si el nuevo certificado forma parte de la cadena de confianza definida en el grupo de inscripción. Si este escenario se produce como respuesta a una infracción de seguridad, al menos debe denegar los certificados de dispositivo específicos en el grupo que se consideran que infringen la seguridad. Para más información, consulte [Denegación de dispositivos específicos en un grupo de inscripción](https://docs.microsoft.com/azure/iot-dps/how-to-revoke-device-access-portal#disallow-specific-devices-in-an-enrollment-group).

La actualización de las entradas de inscripción para certificados implementados se realiza en la página **Administrar inscripciones**. Para acceder a esa página, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la instancia IoT Hub Device Provisioning Service que tiene la entrada de inscripción para el dispositivo.

2. Haga clic en **Administrar inscripciones**.

    ![Administrar inscripciones](./media/how-to-roll-certificates/manage-enrollments-portal.png)


La forma de controlar la actualización de la entrada de inscripción dependerá de si está utilizando inscripciones individuales o inscripciones de grupo. También los procedimientos recomendados difieren dependiendo de si está implementando certificados debido a una infracción de seguridad o porque los certificados han expirado. En las secciones siguientes se describe cómo controlar estas actualizaciones.


## <a name="individual-enrollments-and-security-breaches"></a>Inscripciones individuales e infracciones de seguridad

Si está implementando certificados en respuesta a una infracción de seguridad, debe usar el siguiente enfoque que elimina inmediatamente el certificado actual:

1. Haga clic en **Inscripciones individuales** y haga clic en la entrada del identificador de registro en la lista. 

2. Haga clic en el botón **Delete current certificate** (Eliminar certificado actual) y luego haga clic en el icono de carpeta para seleccionar el nuevo certificado que se va a cargar para la entrada de inscripción. Haga clic en **Guardar** cuando termine.

    Estos pasos se deben completar para el certificado principal y secundario, si ambos están en peligro.

    ![Administración de inscripciones individuales](./media/how-to-roll-certificates/manage-individual-enrollments-portal.png)

3. Una vez que se ha quitado el certificado en peligro del servicio de aprovisionamiento, todavía puede utilizarse para realizar conexiones de dispositivo a IoT Hub, siempre y cuando allí exista un registro para ese dispositivo. Esto se soluciona de dos maneras: 

    La primera sería ir de forma manual a IoT Hub y eliminar inmediatamente el registro del dispositivo relativo al certificado en peligro. Después, al aprovisionar el dispositivo de nuevo con el certificado actualizado, se creará un nuevo registro para el dispositivo.     

    ![Eliminación del registro de dispositivos del centro de IoT](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    La segunda manera consiste en reaprovisionar el dispositivo en la misma instancia de IoT Hub con la ayuda del servicio técnico de reaprovisionamiento. Este enfoque puede utilizarse para reemplazar el certificado de registro del dispositivo en IoT Hub. Para más información, consulte [How to reprovision devices](how-to-reprovision.md) (Reaprovisionamiento de dispositivos).

## <a name="individual-enrollments-and-certificate-expiration"></a>Inscripciones individuales y expiración de certificados

Si está implementando certificados para controlar las expiraciones de los certificados, debe usar la configuración del certificado secundario como se indica a continuación para reducir el tiempo de inactividad de los dispositivos que intentan aprovisionar.

Posteriormente, cuando se aproxima la expiración del certificado secundario y necesita implementarse, puede pasar al uso de la configuración principal. La rotación entre los certificados principales y secundarios de este modo reduce el tiempo de inactividad de los dispositivos que intentan aprovisionar.


1. Haga clic en **Inscripciones individuales** y haga clic en la entrada del identificador de registro en la lista. 

2. Haga clic en **Certificado secundario** y luego haga clic en el icono de carpeta para seleccionar el nuevo certificado que se va a cargar para la entrada de inscripción. Haga clic en **Save**(Guardar).

    ![Administración de las inscripciones individuales mediante el certificado secundario](./media/how-to-roll-certificates/manage-individual-enrollments-secondary-portal.png)

3. Posteriormente, cuando el certificado principal haya expirado, vuelva y elimine ese certificado principal haciendo clic en el botón **Delete current certificate** (Eliminar certificado actual).

## <a name="enrollment-groups-and-security-breaches"></a>Grupos de inscripción e infracciones de seguridad

Para actualizar una inscripción de grupo en respuesta a una infracción de seguridad, debe usar uno de los enfoques que eliminará la entidad de certificación de raíz actual o el certificado intermedio inmediatamente.

#### <a name="update-compromised-root-ca-certificates"></a>Actualización de certificados de entidad de certificación raíz en peligro

1. Haga clic en la pestaña **Certificados** para la instancia de servicio Device Provisioning.

2. Haga clic en el certificado en peligro en la lista y luego en el botón **Eliminar**. Confirme la eliminación escribiendo el nombre del certificado y haga clic en **Aceptar**. Repita este proceso para todos los certificados en peligro.

    ![Eliminación de un certificado de entidad de certificación raíz](./media/how-to-roll-certificates/delete-root-cert.png)

3. Siga los pasos descritos en [Realización de una prueba de posesión de certificados de entidad de certificación X.509 con el servicio Device Provisioning](how-to-verify-certificates.md) para agregar y comprobar los nuevos certificados de entidad de certificación raíz.

4. Haga clic en la pestaña **Administrar inscripciones** para la instancia de servicio Device Provisioning y, luego, haga clic en la lista **Grupos de inscripción**. Haga clic en el nombre del grupo de inscripción en la lista.

5. Haga clic en **Certificado de entidad de certificación** y seleccione el nuevo certificado de entidad de certificación raíz. A continuación, haga clic en **Save**(Guardar). 

    ![Selección del nuevo certificado de entidad de certificación raíz](./media/how-to-roll-certificates/select-new-root-cert.png)

6. Una vez que se ha quitado el certificado en peligro del servicio de aprovisionamiento, todavía puede utilizarse para realizar conexiones de dispositivo a IoT Hub, siempre y cuando allí exista un registro para ese dispositivo. Esto se soluciona de dos maneras: 

    La primera sería ir de forma manual a IoT Hub y eliminar inmediatamente el registro del dispositivo relativo al certificado en peligro. Después, al reaprovisionar de nuevo los dispositivos con los certificados actualizados, se creará un nuevo registro de dispositivo para cada uno de ellos.     

    ![Eliminación del registro de dispositivos del centro de IoT](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    La segunda manera consiste en reaprovisionar los dispositivos en la misma instancia de IoT Hub con la ayuda del servicio técnico de reaprovisionamiento. Este enfoque puede utilizarse para reemplazar los certificados de registro del dispositivo en IoT Hub. Para más información, consulte [How to reprovision devices](how-to-reprovision.md) (Reaprovisionamiento de dispositivos).



#### <a name="update-compromised-intermediate-certificates"></a>Actualización de certificados intermedios en peligro

1. Haga clic en **Grupos de inscripción** y luego haga clic en el nombre de grupo en la lista. 

2. Haga clic en **Intermediate Certificate** (Certificado intermedio) y **Delete current certificate** (Eliminar certificado actual). Haga clic en el icono de carpeta para navegar al nuevo certificado intermedio que se va a cargar para el grupo de inscripción. Haga clic en **Guardar** cuando haya terminado. Estos pasos se deben completar para el certificado principal y secundario, si ambos están en peligro.

    Este nuevo certificado intermedio debe firmarse mediante un certificado de entidad de certificación raíz comprobado que ya se haya agregado al servicio de aprovisionamiento. Para más información, vea [Certificados X.509](concepts-security.md#x509-certificates).

    ![Administración de inscripciones individuales](./media/how-to-roll-certificates/enrollment-group-delete-intermediate-cert.png)


3. Una vez que se ha quitado el certificado en peligro del servicio de aprovisionamiento, todavía puede utilizarse para realizar conexiones de dispositivo a IoT Hub, siempre y cuando allí exista un registro para ese dispositivo. Esto se soluciona de dos maneras: 

    La primera sería ir de forma manual a IoT Hub y eliminar inmediatamente el registro del dispositivo relativo al certificado en peligro. Después, al reaprovisionar de nuevo los dispositivos con los certificados actualizados, se creará un nuevo registro de dispositivo para cada uno de ellos.     

    ![Eliminación del registro de dispositivos del centro de IoT](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    La segunda manera consiste en reaprovisionar los dispositivos en la misma instancia de IoT Hub con la ayuda del servicio técnico de reaprovisionamiento. Este enfoque puede utilizarse para reemplazar los certificados de registro del dispositivo en IoT Hub. Para más información, consulte [How to reprovision devices](how-to-reprovision.md) (Reaprovisionamiento de dispositivos).


## <a name="enrollment-groups-and-certificate-expiration"></a>Grupos de inscripciones y expiración de certificados

Si está implementando certificados para controlar las expiraciones de los certificados, debe usar la configuración del certificado secundario como se indica a continuación para asegurarse de que no hay tiempo de inactividad para los dispositivos que intentan aprovisionar.

Posteriormente, cuando se aproxima la expiración del certificado secundario y necesita implementarse, puede pasar al uso de la configuración principal. La rotación entre los certificados principales y secundarios de este modo asegura que no hay tiempo de inactividad para los dispositivos que intentan aprovisionar. 

#### <a name="update-expiring-root-ca-certificates"></a>Actualización de los certificados de entidad de certificación raíz que van a expirar

1. Siga los pasos descritos en [Realización de una prueba de posesión de certificados de entidad de certificación X.509 con el servicio Device Provisioning](how-to-verify-certificates.md) para agregar y comprobar los nuevos certificados de entidad de certificación raíz.

2. Haga clic en la pestaña **Administrar inscripciones** para la instancia de servicio Device Provisioning y, luego, haga clic en la lista **Grupos de inscripción**. Haga clic en el nombre del grupo de inscripción en la lista.

3. Haga clic en **Certificado de entidad de certificación** y seleccione el nuevo certificado de entidad de certificación raíz en la configuración **Certificado secundario**. A continuación, haga clic en **Save**(Guardar). 

    ![Selección del nuevo certificado de entidad de certificación raíz](./media/how-to-roll-certificates/select-new-root-secondary-cert.png)

4. Posteriormente, cuando el certificado principal haya expirado, haga clic en la pestaña **Certificados** para la instancia de servicio Device Provisioning. Haga clic en el certificado expirado en la lista y luego en el botón **Eliminar**. Confirme la eliminación escribiendo el nombre del certificado y haga clic en **Aceptar**.

    ![Eliminación de un certificado de entidad de certificación raíz](./media/how-to-roll-certificates/delete-root-cert.png)



#### <a name="update-expiring-intermediate-certificates"></a>Actualización de los certificados intermedios que van a expirar


1. Haga clic en **Grupos de inscripción** y luego haga clic en el nombre de grupo en la lista. 

2. Haga clic en **Certificado secundario** y luego haga clic en el icono de carpeta para seleccionar el nuevo certificado que se va a cargar para la entrada de inscripción. Haga clic en **Save**(Guardar).

    Este nuevo certificado intermedio debe firmarse mediante un certificado de entidad de certificación raíz comprobado que ya se haya agregado al servicio de aprovisionamiento. Para más información, vea [Certificados X.509](concepts-security.md#x509-certificates).

   ![Administración de las inscripciones individuales mediante el certificado secundario](./media/how-to-roll-certificates/manage-enrollment-group-secondary-portal.png)

3. Posteriormente, cuando el certificado principal haya expirado, vuelva y elimine ese certificado principal haciendo clic en el botón **Delete current certificate** (Eliminar certificado actual).


## <a name="reprovision-the-device"></a>Reaprovisionamiento del dispositivo

Una vez que el certificado se ha implementado tanto en el dispositivo como en Device Provisioning Service, el dispositivo se puede reaprovisionar a sí mismo poniéndose en contacto con el servicio Device Provisioning. 

Una manera sencilla de programar dispositivos para reaprovisionamiento es programar el dispositivo para que se comunique con el servicio de aprovisionamiento para recorrer el flujo de aprovisionamiento si el dispositivo recibe un error de “no autorizado” al intentar conectarse al centro de IoT.

Otra manera es que los certificados antiguo y nuevo sean válidos para una superposición breve y usar el centro de IoT para enviar un comando a los dispositivos que se vuelvan a registrar mediante el servicio de aprovisionamiento para actualizar su información de conexión de IoT Hub. Dado que cada dispositivo puede procesar los comandos de forma diferente, tendrá que programar el dispositivo para saber qué hacer cuando el comando se invoca. Hay varias formas de controlar el dispositivo mediante IoT Hub; nosotros recomendamos usar [métodos directos](../iot-hub/iot-hub-devguide-direct-methods.md) o [trabajos](../iot-hub/iot-hub-devguide-jobs.md) para iniciar el proceso.

Cuando el reaprovisionamiento se ha completado, los dispositivos podrán conectarse a IoT Hub usando sus nuevos certificados.


## <a name="disallow-certificates"></a>Denegación de certificados

En respuesta a una infracción de seguridad, es posible que necesite denegar un certificado de dispositivo. Para denegar un certificado, deshabilite la entrada de inscripción del certificado o dispositivo de destino. Para más información, consulte cómo denegar dispositivos en el artículo [Baja del servicio Azure IoT Hub Device Provisioning para un dispositivo](how-to-revoke-device-access-portal.md).

Una vez que un certificado se incluye como parte de una entrada de inscripción deshabilitada, todo intento de registro con un centro de IoT mediante esos certificados producirá un error incluso si se habilita como parte de otra entrada de inscripción.
 



## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los certificados X.509 en Device Provisioning Service, consulte [Seguridad](concepts-security.md) 
- Para más información sobre cómo realizar una prueba de posesión de certificados de entidad de certificación X.509 con Azure IoT Hub Device Provisioning Service, consulte [Realización de una prueba de posesión de certificados de entidad de certificación X.509 con el servicio Device Provisioning](how-to-verify-certificates.md)
- Para obtener información acerca de cómo usar el portal para crear un grupo de inscripción, consulte [Administración de inscripciones de dispositivos con Azure Portal](how-to-manage-enrollments.md).
