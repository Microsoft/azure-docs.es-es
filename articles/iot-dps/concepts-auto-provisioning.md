---
title: 'Servicio IoT Hub Device Provisioning: Conceptos de aprovisionamiento automático'
description: Este artículo proporciona información conceptual general sobre las fases de aprovisionamiento automático de un dispositivo, mediante el servicio IoT Hub Device Provisioning Service (DPS), IoT Hub y los SDK de cliente.
author: wesmc7777
ms.author: wesmc
ms.date: 04/04/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: 4755d3ac30a4f6fdc0568dd88fa0e362d7d140a9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87066561"
---
# <a name="auto-provisioning-concepts"></a>Conceptos de aprovisionamiento automático

Como se describe en la [Información general](about-iot-dps.md), Device Provisioning es un servicio auxiliar que permite el aprovisionamiento just-in-time de dispositivos a un centro de IoT, sin necesidad de intervención humana. Después de que se realice correctamente el aprovisionamiento, los dispositivos se conectan directamente con su instancia designada de IoT Hub. Este proceso se conoce como aprovisionamiento automático y proporciona una experiencia de registro y configuración inicial de uso inmediato para los dispositivos.

## <a name="overview"></a>Información general

El aprovisionamiento automático de Azure IoT se puede dividir en tres fases:

1. **Configuración del servicio**: una única configuración de las instancias de Azure IoT Hub y del servicio IoT Hub Device Provisioning, estableciendo ambos servicios y vinculándolos entre sí.

   > [!NOTE]
   > Independientemente del tamaño de la solución de IoT, incluso si piensa dar cabida a millones de dispositivos, se trata de una **configuración única**.

2. **Inscripción de dispositivos**: el proceso de hacer que la instancia del servicio Device Provisioning sea consciente de los dispositivos que intentarán registrarse en el futuro. La [inscripción](concepts-service.md#enrollment) se realiza mediante la configuración de información de identidad de los dispositivos en el servicio de aprovisionamiento, bien como "inscripción individual" para un único dispositivo, o como "inscripción de grupo" para varios dispositivos. La identidad se basa en el [mecanismo de atestación](concepts-security.md#attestation-mechanism) que el dispositivo está diseñado para usar, y que permite al servicio de aprovisionamiento dar fe de la autenticidad del dispositivo durante el registro:

   - **TPM**: configurado como "inscripción individual", la identidad del dispositivo se basa en el identificador de registro TPM y la clave de aprobación pública. Dado que el TPM es una [especificación](https://trustedcomputinggroup.org/work-groups/trusted-platform-module/), el servicio únicamente espera dar fe según la especificación, independientemente de la implementación del TPM (hardware o software). Consulte [Device provisioning: Identity attestation with TPM](https://azure.microsoft.com/blog/device-provisioning-identity-attestation-with-tpm/) (Aprovisionamiento de dispositivos: atestación de identidad con TPM) para obtener detalles sobre la atestación basada en TPM. 

   - **X509**: configurado como parte de una "inscripción individual" o una "inscripción de grupo", la identidad del dispositivo se basa en un certificado digital X.509, que se carga en la inscripción como un archivo .cer o .pem.

   > [!IMPORTANT]  
   > Aunque no se trata de un requisito previo para el uso del servicio Device Provisioning, es muy recomendable que el dispositivo use un módulo de seguridad de hardware (HSM) para almacenar información de identidad de dispositivo confidencial, como las claves y los certificados X.509.

3. **Registro y configuración del dispositivo**: se inician tras el arranque mediante el software de registro, que se compila usando un SDK de cliente del servicio Device Provisioning que sea adecuado para el dispositivo y el mecanismo de atestación. El software establece una conexión con el servicio de aprovisionamiento para la autenticación del dispositivo y el subsiguiente registro en la instancia de IoT Hub. Al registrarse correctamente, el dispositivo se proporciona con su identificador de dispositivo exclusivo e información de conexión de IoT Hub, lo que le permite extraer su configuración inicial y comenzar el proceso de telemetría. En entornos de producción, esta fase puede producirse semanas o meses después de las dos fases anteriores.

## <a name="roles-and-operations"></a>Roles y operaciones

Las fases que se describen en la sección anterior pueden continuar durante semanas o meses, debido a las realidades de la producción como los tiempos de fabricación, los envíos, los procesos de aduanas, etc. Además, pueden abarcar las actividades de varios roles teniendo en cuenta las distintas entidades implicadas. Esta sección profundiza en los distintos roles y operaciones relacionados con cada fase, y luego muestra el flujo en un diagrama de secuencia. 

El aprovisionamiento automático también establece requisitos para el fabricante del dispositivo, que son específicos para la habilitación del mecanismo de atestación. Las operaciones de fabricación también pueden producirse independientemente de la sincronización de fases de aprovisionamiento automático, especialmente en casos donde se adquieren nuevos dispositivos después de que se haya establecido ya el aprovisionamiento automático.

La tabla de contenido a la izquierda proporciona una serie de guías de inicio rápido para ayudar a entender el aprovisionamiento automático mediante la experiencia práctica. Para facilitar y simplificar el proceso de aprendizaje, el software se usa para simular un dispositivo físico para su inscripción y registro. Algunas de estas guías de inicio rápido, debido a su naturaleza de simulacros, le solicitan que realice operaciones que corresponden a varios roles, incluidas operaciones para roles inexistentes.

| Role | Operación | Descripción |
|------| --------- | ------------|
| Fabricante | Codificar la identidad y la URL de registro | Según el mecanismo de atestación utilizado, el fabricante es responsable de codificar la información de identidad del dispositivo y la dirección URL de registro del servicio Device Provisioning.<br><br>**Guías de inicio rápido**: puesto que el dispositivo está simulado, no hay ningún rol de fabricante. Consulte el rol de desarrollador para ver más detalles de cómo se obtiene esta información, que se utiliza en la codificación de una aplicación de registro de ejemplo. |
| | Proporcionar la identidad del dispositivo | Como autor de la información de identidad del dispositivo, el fabricante es responsable de comunicársela al operador (o a un agente designado) o de inscribirla directamente en el servicio Device Provisioning mediante las API.<br><br>**Guías de inicio rápido**: puesto que el dispositivo está simulado, no hay ningún rol de fabricante. Consulte el rol de operador para más información acerca de cómo obtener la identidad del dispositivo que se usa para inscribir un dispositivo simulado en la instancia del servicio Device Provisioning. |
| Operator | Configurar el aprovisionamiento automático | Esta operación se corresponde con la primera fase del aprovisionamiento automático.<br><br>**Inicios rápidos:** en ellas realiza el rol de operador, de modo que configura las instancias del servicio Device Provisioning y de IoT Hub en su suscripción de Azure. |
|  | Inscribir la identidad del dispositivo | Esta operación se corresponde con la segunda fase del aprovisionamiento automático.<br><br>**Inicios rápidos:** en ellas realiza el rol de operador, de modo que inscribe el dispositivo simulado en la instancia del servicio Device Provisioning. La identidad del dispositivo está determinada por el método de atestación simulado en la guía de inicio rápido (TPM o X.509). Consulte el rol de desarrollador para ver los detalles de atestación. |
| Servicio Device Provisioning,<br>IoT Hub | \<all operations\> | Para obtener una implementación de producción con dispositivos físicos y guías de inicio rápido con dispositivos simulados, estos roles se realizan mediante los servicios IoT que configure en la suscripción de Azure. Las operaciones o roles funcionan exactamente igual, ya que los servicios IoT no diferencian entre el aprovisionamiento de dispositivos físicos y el de dispositivos simulados. |
| Desarrollador | Compilar/implementar software de registro | Esta operación se corresponde con la tercera fase del aprovisionamiento automático. El desarrollador es responsable de compilar e implementar el software de registro en el dispositivo mediante el SDK adecuado.<br><br>**Inicios rápidos:** la aplicación de registro de ejemplo que compila simula un dispositivo real para su plataforma/lenguaje preferido, que se ejecuta en la estación de trabajo (en lugar de implementarla en un dispositivo físico). La aplicación de registro realiza las mismas operaciones que una implementada en un dispositivo físico. Especifique el método de atestación (TPM o certificado X.509), además de la dirección URL de registro y el "Ámbito de identificador" de la instancia del servicio Device Provisioning. La identidad del dispositivo viene determinada por la lógica de atestación del SDK en tiempo de ejecución, en función del método que especifique: <ul><li>**Atestación del TPM**: su estación de trabajo de desarrollo ejecuta una [aplicación de simulador del TPM](how-to-use-sdk-tools.md#trusted-platform-module-tpm-simulator). Una vez que se ejecuta, se utiliza una aplicación independiente para extraer la "Clave de aprobación" del TPM y el "Identificador del registro" para su uso en la inscripción de la identidad del dispositivo. La lógica de atestación del SDK también utiliza el simulador durante el registro, para presentar un token SAS firmado para la autenticación y comprobación de inscripción.</li><li>**Atestación X509**: se utiliza una herramienta para [generar un certificado](how-to-use-sdk-tools.md#x509-certificate-generator). Una vez generado, se crea el archivo de certificado necesario para su uso en la inscripción. La lógica de atestación del SDK también utiliza el certificado durante el registro, para presentarlo para la autenticación y comprobación de inscripción.</li></ul> |
| Dispositivo | Arranque y registro | Esta operación se corresponde con la tercera fase de aprovisionamiento automático, que cumple el software de registro del dispositivo compilado por el desarrollador. Consulte el rol de desarrollador para ver los detalles. Tras el primer arranque: <ol><li>La aplicación se conecta con la instancia del servicio Device Provisioning, por la dirección URL global y el "Ámbito de identificador" especificado durante el desarrollo.</li><li>Una vez conectado, el dispositivo se autentica en el método de atestación y la identidad especificada durante la inscripción.</li><li>Una vez autenticado, el dispositivo está registrado con la instancia de IoT Hub especificada por la instancia del servicio de aprovisionamiento.</li><li>Después de registrarse correctamente, se devuelve un identificador único de dispositivo y un punto de conexión de IoT Hub a la aplicación de registro para la comunicación con IoT Hub.</li><li> A partir de ahí, el dispositivo puede extraer el estado de [dispositivo gemelo](~/articles/iot-hub/iot-hub-devguide-device-twins.md) inicial para la configuración, y comenzar el proceso de enviar informes de datos de telemetría.</li></ol>**Guías de inicio rápido**: puesto que el dispositivo está simulado, el software de registro se ejecuta en la estación de trabajo de desarrollo.|

En el diagrama siguiente se resume los roles y la secuencia de operaciones durante el aprovisionamiento automático de dispositivos:
<br><br>
[![Secuencia de aprovisionamiento automático para un dispositivo](./media/concepts-auto-provisioning/sequence-auto-provision-device-vs.png)](./media/concepts-auto-provisioning/sequence-auto-provision-device-vs.png#lightbox) 

> [!NOTE]
> Si lo desea, el fabricante también puede realizar la operación "Inscribir la identidad del dispositivo" utilizando las API del servicio Device Provisioning (en lugar de mediante el operador). Para obtener una explicación detallada de esta secuencia y mucho más, consulte el vídeo [Zero touch device registration with Azure IoT](https://youtu.be/cSbDRNg72cU?t=2460) (Registro de dispositivos sin intervención del usuario con Azure IoT), empezando en el marcador 41:00

## <a name="roles-and-azure-accounts"></a>Roles y cuentas de Azure

La forma en que se asigna cada rol a una cuenta de Azure depende del escenario, y hay bastantes escenarios que pueden estar implicados. Los patrones comunes que se presentan a continuación deberían ayudar a proporcionar una comprensión general sobre cómo se asignan normalmente los roles a una cuenta de Azure.

#### <a name="chip-manufacturer-provides-security-services"></a>El fabricante de chips proporciona servicios de seguridad

En este escenario, el fabricante administra la seguridad para los clientes de nivel uno. Este escenario puede ser el preferido por estos clientes de nivel uno, ya que no tienen que administrar una seguridad detallada. 

El fabricante introduce la seguridad en los módulos de seguridad de hardware (HSM). Esta seguridad puede incluir que el fabricante obtenga claves, certificados, etc., de clientes potenciales que ya tienen instancias de DPS y grupos de inscripción configurados. El fabricante también podría generar esta información de seguridad para sus clientes.

En este escenario, puede haber dos cuentas de Azure implicadas:

- **Cuenta n.º 1**: probablemente compartida a través de los roles de operador y desarrollador hasta cierto punto. Esta entidad puede comprar los chips HSM al fabricante. Estos chips apuntan a instancias de DPS asociadas con la cuenta n.º 1. Con las inscripciones de DPS, esta entidad puede conceder dispositivos a varios clientes de nivel dos mediante la reconfiguración de la configuración de inscripción de dispositivos en DPS. Esta entidad también puede tener centros de IoT Hub asignados para sistemas backend de usuario final con los que interactuar para acceder a la telemetría de dispositivos, etc. En este último caso, puede que no sea necesaria una segunda cuenta.

- **Cuenta n.º 2**: los usuarios finales, los clientes de nivel 2, pueden tener sus propios centros de IoT Hub. La entidad asociada con la cuenta n.º 1 solo señala los dispositivos concedidos al centro correcto en esta cuenta. Esta configuración requiere vincular los centros de DPS y IoT Hub en las cuentas de Azure, que puede hacerse con las plantillas de Azure Resource Manager.

#### <a name="all-in-one-oem"></a>OEM todo en uno

El fabricante podría ser un "OEM todo en uno" en el que solo se necesitaría una única cuenta de fabricante. El fabricante se encarga de la seguridad y del aprovisionamiento de principio a fin.

El fabricante puede proporcionar una aplicación basada en la nube a los clientes que compren dispositivos. Esta aplicación interactuaría con el centro de IoT Hub asignado por el fabricante.

Las máquinas expendedoras o las máquinas de café automatizadas representan ejemplos de este escenario.




## <a name="next-steps"></a>Pasos siguientes

Le puede resultar útil marcar este artículo como punto de referencia, mientras va realizando las Guías de inicio rápido del aprovisionamiento automático correspondientes. 

Empiece por completar la guía de inicio rápido de "Configuración del aprovisionamiento automático" que mejor se adapte a sus preferencias de herramienta de administración, que le guiará por la fase de "Configuración del servicio":

- [Configuración del aprovisionamiento automático con la CLI de Azure](quick-setup-auto-provision-cli.md)
- [Configuración del aprovisionamiento automático con Azure Portal](quick-setup-auto-provision.md)
- [Configuración del aprovisionamiento automático con una plantilla de Resource Manager](quick-setup-auto-provision-rm.md)

A continuación, continúe con una guía de inicio rápido de "Aprovisionamiento automático de un dispositivo simulado" que se adapte a su mecanismo de atestación de dispositivo y las preferencias de lenguaje/SDK del servicio Device Provisioning. Esta guía de inicio rápido le guiará por las fases de "Inscripción de dispositivos" y "Registro y configuración de dispositivos": 

| Mecanismo de atestación de dispositivo simulado | Guía de inicio rápido SDK/lenguaje |
| -------------------------------------- | ----------------------- |
| Módulo de plataforma segura (TPM) | [C](quick-create-simulated-device.md)<br>[Java](quick-create-simulated-device-tpm-java.md)<br>[C#](quick-create-simulated-device-tpm-csharp.md)<br>[Python](quick-create-simulated-device-tpm-python.md) |
| Certificado X.509 | [C](quick-create-simulated-device-x509.md)<br>[Java](quick-create-simulated-device-x509-java.md)<br>[C#](quick-create-simulated-device-x509-csharp.md)<br>[Node.js](quick-create-simulated-device-x509-node.md)<br>[Python](quick-create-simulated-device-x509-python.md) |




