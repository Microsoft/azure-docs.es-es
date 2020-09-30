---
title: 'Información acerca de Azure Orbital: versión preliminar'
description: Obtenga información sobre la nueva estación terrestre como solución de servicio de Microsoft - Azure Orbital.
services: vnf-manager
author: wamota
ms.service: vnf-manager
ms.topic: overview
ms.date: 09/11/2020
ms.author: wamota
ms.openlocfilehash: 5a858531f578612be8190625ae371992babca34e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945650"
---
# <a name="what-is-azure-orbital-preview"></a>¿Qué es Azure Orbital? (versión preliminar)

Azure Orbital es una estación terrestre basada en la nube totalmente administrada como servicio que le permite comunicarse con su nave espacial o constelaciones de satélites, descargar los datos y crear un vínculo superior, procesar los datos en la nube, encadenar servicios con servicios de Azure en escenarios únicos y generar productos para sus clientes. Azure Orbital le permite centrarse en los datos del producto y la misión sin tener que cargar la responsabilidad de la implementación y el mantenimiento de los recursos de la estación terrestre. Este sistema se basa en la infraestructura global de Azure y en la red de fibra global de latencia baja.

[:::image type="content" source="./media/azure-orbital-overview/orbital-all-ignite-link.png" alt-text="Vídeo de inicio de Ignite de Azure Orbital":::](https://aka.ms/orbitalatignite)
[Ver el anuncio de Azure Orbital en Ignite en el canal de YouTube de Azure](https://aka.ms/orbitalatignite)

Azure Orbital se centra en la compilación de un ecosistema de asociados para que los clientes puedan usar estaciones terrestres de asociados, además de estaciones terrestres de Orbital, así como usar módems en la nube de asociados e integrados.   Hemos asociado los enfoques de Azure Orbital sobre la asociación con líderes de la industria, como KSAT, además de otros proveedores de estaciones terrestres, como ViaSat - La Tierra en tiempo real (RTE) y Microdynamics Inc., para proporcionar una amplia cobertura que esté disponible por adelantado. Esta asociación también se extiende a los proveedores de telecomunicaciones vía satélite, como SES, y otros proveedores de estaciones terrestres, ViaSat - La Tierra en tiempo real (RTE) y US Electrodynamics Inc. para ofrecer una conectividad sin precedentes, como el acceso global a la flota de LEO/MEO o el acceso directo a Azure para las constelaciones de comunicación o el acceso global a su flota de LEO/MEO. Hemos llevado a cabo los pasos para virtualizar la señal RF y la asociación con líderes, como Kratos y Amergint, para traer sus módems a Marketplace. Nuestro objetivo es capacitar a nuestros clientes para conseguir más logros y compilar sistemas con nuestra plataforma de servicio de estación terrestre escalable y muy flexible.

Azure Orbital habilita varios casos de uso para nuestros clientes, como la observación terrestre y las comunicaciones globales. También proporciona una plataforma que permite la transformación digital de las estaciones terrestres existentes mediante la virtualización. Tiene acceso directo a todos los servicios de Azure, la infraestructura global de Azure y Marketplace, así como a nuestro ecosistema de asociados de primera categoría a través de nuestro servicio.

:::image type="content" source="./media/azure-orbital-overview/orbital-all-overview.png" alt-text="Información general sobre Azure Orbital":::

**Las propuestas de valor para los usuarios de Azure Orbital incluyen:**

* **Presencia global**: el servicio de estación terrestre de Azure Orbital también incluye las estaciones terrestres de nuestro asociado. La cobertura global está disponible sin retraso y los clientes pueden programar contactos mediante Orbital en las estaciones terrestres del asociado, además de las estaciones terrestres propiedad de Microsoft.

* **Conversión de los gastos de capital en gastos operativos**: a medida que se realiza la tarea de implementación y administración de las estaciones terrestres, se pueden usar los costos iniciales necesarios para las inversiones en la estación terrestre para centrarse en la misión y la implementación de los recursos. Nuestro modelo de consumo de pago por uso significa que solo se le cobrará por el tiempo que use.

* **Licencias**: nuestro equipo puede ayudarle a incorporar los satélites en nuestros sitios y cuerpos normativos.

* **Eficiencia operativa y escalabilidad**: ya no tiene que preocuparse por el mantenimiento, el leasing, la compilación o la ejecución de los costos operativos de las estaciones terrestres. Tendrá la opción de escalar rápidamente las comunicaciones por satélite a petición cuando la empresa las necesite.

* **Acceso directo a la red y a las regiones de Azure**: implementamos nuestras propias estaciones terrestres en las ubicaciones del centro de datos o cerca del perímetro de la red y también realizamos la interconexión con las redes y estaciones terrestres de los asociados para ofrecer proximidad a las regiones de Azure. Los datos se entregan a la nube de forma inmediata y en cualquier lugar en la ubicación deseada a través de la red de fibra global segura y de latencia baja de Azure.

* **RF digitalizado**: con la señal totalmente digitalizada disponible desde la antena, incluidos hasta 500 MHz de banda ancha, tiene un control total y una seguridad de los datos procedentes de su nave espacial. Los módems de software de nuestros asociados están integrados en la plataforma para un uso optimizado y también están disponibles en Marketplace para completar la cadena de procesamiento. También tenemos previsto que determinados clientes traigan sus propios módems (para las necesidades exclusivas de su misión), que es compatible con la entrega de RF digitalizado en el punto de conexión designado en la red virtual.

* **Azure Cloud y Marketplace**: aproveche todas las soluciones de Azure para procesar y almacenar los datos, incluidas, entre otras, IoT, IA y ML,Cognitive Services, análisis y almacenamiento y cadenas, junto con la carga de trabajo en un ambiente.

* **Flexibilidad**: la potencia de nuestro servicio de programación, las redes de asociados, el RF digitalizado y Marketplace significa que no está restringido a un determinado flujo de trabajo o conjunto de soluciones. Le recomendamos que piense de forma innovadora y se ponga en contacto con nosotros. Por ejemplo, la cadena de productos también se podría ofrecer en Marketplace para que otros usuarios la incorporen en sus productos. Las posibilidades son infinitas.

Para obtener más información sobre nuestra versión preliminar o para manifestar su interés en participar en ella, rellene el formulario de contacto [aquí](https://aka.ms/iaminterested) o envíenos un correo electrónico a [MSAzureOrbital@microsoft.com](mailto:MSAzureOrbital@microsoft.com).

## <a name="earth-observation"></a><a name="earth-observation"></a>Observación terrestre

:::image type="content" source="./media/azure-orbital-overview/orbital-eos-dataflow.png" alt-text="Flujo de datos de Azure Orbital para la observación terrestre" lightbox="./media/azure-orbital-overview/orbital-eos-dataflow-expanded.png":::

Puede usar Azure Orbital para programar contactos con satélites mediante un sistema de pago por uso para el mantenimiento y la carga de enlaces de bajada. Utilice los tiempos de acceso programados para ingerir datos del satélite, supervisar el estado de los satélites, o bien transmitir comandos al satélite. Los datos entrantes se entregan a la red virtual privada, lo que permite su procesamiento o almacenamiento en Azure.

Dado que el servicio está totalmente digitalizado, se puede usar un módem de software de Kratos y Amergint para realizar las funciones de modulación y desmodulación y codificación y descodificación para recuperar los datos. Tendrá la opción de comprar en Marketplace o podemos administrar esta parte por usted. Además, realice la integración con Kubos para aprovechar completamente una solución de un extremo a otro con el fin de administrar las operaciones de flota y las funciones de telemetría, seguimiento y control (TT&C). Implemente las cargas de trabajo en Azure mediante los recursos y los cuadros de herramientas de Azure para manipular los datos de carga en las ofertas finales.

:::image type="content" source="./media/azure-orbital-overview/orbital-eos-schedule.png" alt-text="Programación de Azure Orbital para la observación terrestre":::

### <a name="scheduling-contacts"></a><a name="scheduling-contacts"></a>Programación de contactos

La programación de contactos con Azure Orbital es un proceso sencillo de tres pasos: 

1. **Registro de una nave espacial**: introduzca el identificador de NORAD, TLE y la información de licencia de cada satélite.

2. **Creación de un perfil de contacto**: especifique los requisitos de frecuencia y ancho de banda del centro para cada vínculo, así como otros detalles, como los requisitos mínimos de elevación y seguimiento automático. No dude en crear tantos perfiles como sea necesario. Por ejemplo, uno solo para comandos, o uno para la carga de enlaces de bajada.

3. **Programación del contacto**: seleccione la nave espacial y un perfil de contacto junto con la ventana de fecha y hora para ver las fases disponibles en nuestro sitio y en los de la red de asociados que se van a reservar. En primer lugar, se producirá un primer algoritmo, pero la programación de prioridad o la programación garantizada se encuentran en el plan de desarrollo.

Para obtener más información sobre nuestra versión preliminar o para manifestar su interés en participar en ella, rellene el formulario de contacto [aquí](https://aka.ms/iaminterested) o envíenos un correo electrónico a [MSAzureOrbital@microsoft.com](mailto:MSAzureOrbital@microsoft.com).

## <a name="global-communication"></a><a name="global-communication"></a>Comunicación global

:::image type="content" source="./media/azure-orbital-overview/orbital-communications-use-flow.png" alt-text="Flujo de uso de Azure Orbital para comunicaciones globales":::

Los proveedores de satélites que proporcionan capacidades de comunicación global a sus clientes pueden usar Azure Orbital para ubicar nuevas estaciones terrestres en centros de datos de Azure o en el perímetro de la red de Azure, o bien para interconectar sus estaciones terrestres con la red troncal global de Azure. A continuación, pueden enrutar su tráfico en una red global de Microsoft y aprovechar la fuga de Internet desde el perímetro de la red de Azure para proporcionar servicios de Internet y otros servicios administrados a sus clientes.

El servicio de Azure Orbital entrega el tráfico desde la estación terrestre de Orbital a la red virtual del proveedor. Con estos servicios de Azure Orbital, un proveedor de satélites puede integrar o agrupar otros servicios de Azure (por ejemplo, servicios de seguridad, como firewall, servicios de conectividad, como SDWAN, etc.) junto con su conectividad por satélite para proporcionar servicios administrados a sus clientes, además de conectividad por satélite. 

Para obtener más información sobre nuestra versión preliminar o para manifestar su interés en participar en ella, rellene el formulario de contacto [aquí](https://aka.ms/iaminterested) o envíenos un correo electrónico a [MSAzureOrbital@microsoft.com](mailto:MSAzureOrbital@microsoft.com).

## <a name="partner-ground-stations"></a><a name="digital-transformation"></a>Estaciones terrestres de asociados

Además de crear nuestras propias estaciones terrestres, Azure Orbital permite a los clientes usar estaciones terrestres de los asociados para ingerir datos directamente en Azure.

Los proveedores de la estación terrestre pueden asociarse con Azure Orbital para transformar digitalmente sus estaciones terrestres. Al hacerlo, los clientes pueden usar estas estaciones terrestres para programar contactos para sus satélites y aprovechar todas las capacidades de procesamiento de datos y procesamiento de radio de software que ofrecen la plataforma y los asociados de Orbital a través de Marketplace. El servicio está estrechamente integrado con las cargas de trabajo en la nube y un ecosistema vivo de soluciones de terceros a través de Marketplace, como los módems, la administración de recursos y los servicios de control de misiones. Además, todos los datos pueden aprovechar la latencia baja y la red de fibra global de alta confiabilidad de Azure. En conjunto, creemos que ofrecerá la flexibilidad y cobertura más amplias posibles para que nuestros clientes se comuniquen con los satélites con la máxima agilidad y confiabilidad.

:::image type="content" source="./media/azure-orbital-overview/orbital-all-digital-transformations.png" alt-text="Azure Orbital para la transformación digital":::

Para obtener más información sobre nuestra versión preliminar o para manifestar su interés en participar en ella, rellene el formulario de contacto [aquí](https://aka.ms/iaminterested) o envíenos un correo electrónico a [MSAzureOrbital@microsoft.com](mailto:MSAzureOrbital@microsoft.com).

## <a name="partners"></a><a name="orbital-partners"></a>Asociados

A medida que avanzamos con nuestro viaje al espacio, agregaremos más asociados a nuestro ecosistema para ayudar a nuestros clientes a conseguir más logros con Azure Orbital. A medida que compilemos Azure Orbital, nos guiaremos por los asociados en nuestro enfoque. Nuestro objetivo también es compilar un ecosistema vivo de asociados para crear de forma conjunta más valor para nuestros asociados, así como para nuestros clientes. Imagínese que se trata de un arrecife de coral.

:::image type="content" source="./media/azure-orbital-overview/orbital-all-partners.png" alt-text="Asociados de Azure Orbital":::

En las secciones siguientes se muestra una lista de categorías de asociados y asociados de Azure Orbital que ya forman parte del ecosistema de Orbital:

### <a name="ground-station-infrastructure-partners"></a>Asociados de infraestructura de la estación terrestre

Nos hemos asociado con [KSAT](https://www.ksat.no), RTE (la Tierra en tiempo real) de [ViaSat](https://www.viasat.com/products/antenna-systems) y [US Electrodynamics](https://www.usei-teleport.com/) para permitir que nuestros clientes se comuniquen con sus satélites mediante el uso de estas estaciones terrestres del asociado y la ingesta de datos directamente en Azure.

### <a name="virtualized-modem-partners"></a>Asociados de módem virtualizado

Nos hemos asociado con [Kratos](https://www.kratosdefense.com/) y [Amergint](https://www.amergint.com/) para llevar sus capacidades de procesamiento de radio de software a la nube como parte de nuestra plataforma de Orbital. Estas capacidades se han actualizado para adoptar las aceleraciones de la plataforma de Azure (incluidas, entre otras, las redes aceleradas mediante DPDK y la aceleración basada en GPU con Azure Compute para usos especiales) para procesar la señal de radio en tiempo real a un alto rendimiento o ancho de banda. Además, nuestros clientes también podrán implementar estos módems de software de nuestros asociados de Azure Marketplace en sus propias redes virtuales para un control más pormenorizado y preciso sobre el procesamiento de señales.

### <a name="global-communication-partner"></a>Asociado de comunicación global

[SES](https://www.ses.com/) es uno de los proveedores de conectividad por satélite más grandes del sector espacial. Nos complace compartir que SES ha seleccionado Azure Orbital para aumentar sus necesidades de red terrestre para su [mPower de sistema de comunicación MEO de próxima generación](https://www.ses.com/networks/networks-and-platforms/o3b-mpower). Como parte de este lanzamiento, se colocarán nuevas estaciones terrestres dedicadas en nuestros centros de datos, además de la interconexión de las estaciones terrestres existentes con nuestra red troncal global. Así se permite que SES obtenga un tiempo de comercialización más rápido con una estación terrestre altamente escalable como servicio aprovechando los módems virtualizados basados en la nube que se proporcionan como parte de la plataforma de Orbital, además de la red troncal global de Azure.

SES aprovechará nuestra red troncal global para enrutar su tráfico globalmente y usar los servicios de Azure Orbital para proporcionar a sus clientes varios servicios administrados basados en la plataforma. Estos servicios van desde servicios de seguridad, SDWAN, procesos perimetrales y soluciones de movilidad 5G a varios otros servicios.

### <a name="ttc-solution-partner"></a>Asociado de la solución de TT&C

Nos hemos asociado con [Kubos](https://www.kubos.com/) para ofrecer Major Tom, su software de control de misiones basado en la nube, a los clientes de Azure Orbital mediante Azure Marketplace.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre nuestra versión preliminar o para manifestar su interés en participar en ella, rellene el formulario de contacto [aquí](https://aka.ms/iaminterested) o envíenos un correo electrónico a [MSAzureOrbital@microsoft.com](mailto:MSAzureOrbital@microsoft.com).