---
title: Baja de Azure IoT Hub Device Provisioning Service para un dispositivo
description: Baja del servicio Azure IoT Hub Device Provisioning Service (DPS) para evitar el aprovisionamiento de un dispositivo
author: wesmc7777
ms.author: wesmc
ms.date: 04/05/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: abc625c27617c27a79fe284393cdf7b281973bc4
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299270"
---
# <a name="how-to-disenroll-a-device-from-azure-iot-hub-device-provisioning-service"></a>Baja del servicio Azure IoT Hub Device Provisioning para un dispositivo

La administración adecuada de las credenciales del dispositivo es fundamental para los sistemas de alto perfil como las soluciones de IoT. Un procedimiento recomendado para estos sistemas es tener un plan claro de cómo revocar el acceso de los dispositivos en los casos en que sus credenciales, tanto un token de firmas de acceso compartido (SAS) como un certificado X.509, puedan suponer un riesgo. 

La inscripción en el servicio Device Provisioning habilita el [aprovisionamiento automático](concepts-auto-provisioning.md) en los dispositivos. Al aprovisionar un dispositivo, se registra en IoT Hub, lo cual le permite recibir el estado del [dispositivo gemelo](~/articles/iot-hub/iot-hub-devguide-device-twins.md) inicial y empezar a informar de los datos de telemetría. En este artículo se describe cómo dar de baja un dispositivo de la instancia del servicio de aprovisionamiento y evitar así que se vuelva a aprovisionar en el futuro.

> [!NOTE] 
> Tenga en cuenta la directiva de reintentos de los dispositivos para los que se revoca el acceso. Por ejemplo, un dispositivo que tiene una directiva de reintentos infinitos puede reintentar continuamente registrarse en el servicio de aprovisionamiento. Esta situación consume recursos del servicio y posiblemente afecta al rendimiento.

## <a name="disallow-devices-by-using-an-individual-enrollment-entry"></a>Denegar dispositivos con una entrada de inscripción individual

Las inscripciones individuales son aplicables a un solo dispositivo y pueden usar certificados X.509 o tokens de SAS (en un módulo de plataforma segura real o virtual) como mecanismos de atestación. (Los dispositivos que usan tokens de SAS como mecanismo de atestación solo se puede aprovisionar mediante una inscripción individual). Para denegar un dispositivo que tenga una inscripción individual, puede deshabilitar o eliminar su entrada de inscripción. 

Para denegar temporalmente el dispositivo, deshabilite su entrada de inscripción mediante estos pasos: 

1. Inicie sesión en Azure Portal y seleccione **Todos los recursos** en el menú a la izquierda.
2. En la lista de recursos, seleccione el servicio de aprovisionamiento en el que desea denegar su dispositivo.
3. En el servicio de aprovisionamiento, seleccione **Administrar inscripciones** y después, la pestaña **Inscripciones individuales**.
4. Seleccione la entrada de inscripción del dispositivo que desee denegar. 

    ![Seleccione su inscripción individual](./media/how-to-revoke-device-access-portal/select-individual-enrollment.png)

5. En la página de inscripción, desplácese hacia abajo y seleccione **Deshabilitar** para el conmutador **Habilitar entrada** y después **Guardar**.  

   ![Deshabilitación de una entrada de inscripción individual en el portal](./media/how-to-revoke-device-access-portal/disable-individual-enrollment.png)

Para denegar el dispositivo de forma permanente, elimine su entrada de inscripción mediante estos pasos:

1. Inicie sesión en Azure Portal y seleccione **Todos los recursos** en el menú a la izquierda.
2. En la lista de recursos, seleccione el servicio de aprovisionamiento en el que desea denegar su dispositivo.
3. En el servicio de aprovisionamiento, seleccione **Administrar inscripciones** y después, la pestaña **Inscripciones individuales**.
4. Seleccione la casilla que se encuentra junto a la entrada de inscripción del dispositivo que desee denegar. 
5. Seleccione **Eliminar** en la parte superior de la ventana y después seleccione **Sí** para confirmar que desea eliminar la inscripción. 

   ![Eliminación de una entrada de inscripción individual en el portal](./media/how-to-revoke-device-access-portal/delete-individual-enrollment.png)


Una vez completado el procedimiento, debe ver que la entrada se ha eliminado de la lista de inscripciones de individuales.  

## <a name="disallow-an-x509-intermediate-or-root-ca-certificate-by-using-an-enrollment-group"></a>Denegación de un certificado de entidad de certificación X.509 intermedia o raíz usando un grupo de inscripción

Los certificados X.509 normalmente están organizados en una cadena de certificados de confianza. Si un certificado en cualquier fase de una cadena está en peligro, se rompe la confianza. Se debe denegar el certificado para evitar que el servicio Device Provisioning aprovisione dispositivos en un nivel inferior de cualquier cadena que contiene ese certificado. Para más información sobre los certificados X.509 y cómo se utilizan con el servicio de aprovisionamiento, consulte [certificados X.509](./concepts-security.md#x509-certificates). 

Un grupo de inscripción es una entrada para dispositivos que comparten un mecanismo de atestación común de certificados X.509, firmados por la misma entidad de certificación intermedia o raíz. La entrada del grupo de inscripción se configura con el certificado X.509 asociado con la entidad de certificación intermedia o raíz. La entrada también se configura con cualquiera de los valores de configuración, como estado gemelo y conexión de IoT Hub compartidos por los dispositivos con ese certificado en su cadena de certificados. Para denegar el certificado, puede deshabilitar o eliminar su grupo de inscripción.

Para denegar temporalmente el certificado, deshabilite su grupo de inscripción: 

1. Inicie sesión en Azure Portal y seleccione **Todos los recursos** en el menú a la izquierda.
2. En la lista de recursos, seleccione el servicio de aprovisionamiento para el que desee denegar el certificado de firma.
3. En el servicio de aprovisionamiento, seleccione **Administrar inscripciones** y después, la pestaña **Grupos de inscripción**.
4. Seleccione el grupo de inscripción con el certificado que desee denegar.
5. Seleccione **Deshabilitar** en el conmutador **Habilitar entrada** y después seleccione **Guardar**.  

   ![Deshabilitación de entrada de grupo de inscripción en el portal](./media/how-to-revoke-device-access-portal/disable-enrollment-group.png)

    
Para denegar el certificado de forma permanente, elimine su grupo de inscripción:

1. Inicie sesión en Azure Portal y seleccione **Todos los recursos** en el menú a la izquierda.
2. En la lista de recursos, seleccione el servicio de aprovisionamiento en el que desea denegar su dispositivo.
3. En el servicio de aprovisionamiento, seleccione **Administrar inscripciones** y después, la pestaña **Grupos de inscripción**.
4. Seleccione la casilla que se encuentra junto al grupo de inscripción del certificado que desee denegar. 
5. Seleccione **Eliminar** en la parte superior de la ventana y después seleccione **Sí** para confirmar que desea eliminar el grupo de inscripción. 

   ![Eliminación de la entrada de grupo de inscripción en el portal](./media/how-to-revoke-device-access-portal/delete-enrollment-group.png)

Una vez completado el procedimiento, debe ver que la entrada se ha eliminado de la lista de grupos de inscripción.  

> [!NOTE]
> Si elimina un grupo de inscripción de un certificado, los dispositivos que tienen el certificado en su cadena de certificados pueden seguir realizando inscripciones si existe un grupo de inscripción habilitado para el certificado raíz u otro certificado intermedio más arriba en la cadena de certificados.

## <a name="disallow-specific-devices-in-an-enrollment-group"></a>Denegar dispositivos específicos en un grupo de inscripción

Los dispositivos que implementen el mecanismo de atestación de X.509 usan la cadena de certificados del dispositivo y la clave privada para autenticar. Cuando un dispositivo se conecta y se autentica con el servicio Device Provisioning, el servicio busca primero una inscripción individual que coincida con las credenciales del dispositivo. El servicio busca después los grupos de inscripción para determinar si se puede aprovisionar el dispositivo. Si el servicio detecta una inscripción individual deshabilitada para el dispositivo, impide que el dispositivo se conecte. El servicio impide la conexión incluso si existe un grupo de inscripción habilitado para una entidad de certificación intermedia o raíz en la cadena de certificados del dispositivo. 

Para denegar un dispositivo individual en un grupo de inscripción, siga estos pasos:

1. Inicie sesión en Azure Portal y seleccione **Todos los recursos** en el menú a la izquierda.
2. En la lista de recursos, seleccione el servicio de aprovisionamiento que contiene el grupo de inscripción para el dispositivo que desee denegar.
3. En el servicio de aprovisionamiento, seleccione **Administrar inscripciones** y después, la pestaña **Inscripciones individuales**.
4. Seleccione el botón **Agregar inscripción individual** situado en la parte superior. 
5. En la página **Agregar inscripción**, seleccione **X.509** como el **Mecanismo** de atestación para el dispositivo.

    Cargue el certificado del dispositivo y escriba el identificador del dispositivo que va a denegar. Para el certificado, utilice el certificado de entidad final firmado instalado en el dispositivo. El dispositivo usa el certificado de entidad final firmado para la autenticación.

    ![Establecimiento de las propiedades de dispositivo para el dispositivo denegado](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group-1.png)

6. Desplácese hasta la parte inferior de la página **Agregar inscripción** y seleccione **Deshabilitar** en el conmutador **Habilitar entrada** y después **Guardar**. 

    [![Uso de la deshabilitación de una entrada de inscripción individual para deshabilitar en el portal la inscripción en grupo del dispositivo](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group.png)](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group.png#lightbox)

Al crear la inscripción correctamente, debería ver que la inscripción del dispositivo deshabilitado aparece en la pestaña **Inscripciones individuales**. 

## <a name="next-steps"></a>Pasos siguientes

La baja también forma parte del proceso global de desaprovisionamiento. El desaprovisionamiento de un dispositivo incluye tanto la baja del servicio de aprovisionamiento como la cancelación del registro en IoT Hub. Para información sobre el proceso completo, consulte [Desaprovisionamiento de dispositivos aprovisionados automáticamente](how-to-unprovision-devices.md) 
