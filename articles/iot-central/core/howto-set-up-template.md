---
title: Definición de un nuevo tipo de dispositivo IoT en Azure IoT Central | Microsoft Docs
description: Este artículo le muestra, como desarrollador, cómo crear una plantilla de dispositivo de Azure IoT en la aplicación de Azure IoT Central. Se definen la telemetría, el estado, las propiedades y los comandos del tipo.
author: dominicbetts
ms.author: dobett
ms.date: 12/06/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: contperfq1
ms.openlocfilehash: a75fbeb9a12f61f827411e56c57ff6a4460ab083
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136276"
---
# <a name="define-a-new-iot-device-type-in-your-azure-iot-central-application"></a>Definición de un nuevo tipo de dispositivo IoT en la aplicación de Azure IoT Central

*Este artículo se aplica a generadores de soluciones y desarrolladores de dispositivos.*

Una plantilla de dispositivo es un plano técnico que define las características y los comportamientos de un tipo de dispositivo que se conecta a una [aplicación de Azure IoT Central](concepts-app-templates.md).

Por ejemplo, un generador puede crear una plantilla de dispositivo para un ventilador conectado que tenga las siguientes características:

- Envía datos de telemetría de temperatura.
- Envía la propiedad de ubicación.
- Envía eventos de error del motor del ventilador.
- Envía el estado de funcionamiento del ventilador.
- Proporciona una propiedad de velocidad del ventilador que se puede modificar
- Proporciona un comando para reiniciar el dispositivo
- Le ofrece una vista global del dispositivo mediante un panel

A partir de esta plantilla de dispositivo, un operador puede crear y conectar dispositivos de ventilador reales. Todos estos ventiladores tienen medidas, propiedades y comandos que los operadores utilizan para supervisar y administrar. Los operadores usan los [paneles de dispositivos](#add-dashboards) y los formularios para interactuar con los dispositivos de ventilación. Un desarrollador de dispositivos usa la plantilla para comprender cómo interactúa el dispositivo con la aplicación. Para más información, consulte [Cargas de telemetría, propiedades y comandos](concepts-telemetry-properties-commands.md).

> [!NOTE]
> Solo los generadores y administradores pueden crear, editar y eliminar plantillas de dispositivo. Cualquier usuario puede crear dispositivos en la página **Devices** (Dispositivos) a partir de las plantillas de dispositivo existentes.

En una aplicación de IoT Central, una plantilla de dispositivo usa un modelo de funcionalidad del dispositivo para describir las capacidades de un dispositivo. Como generador, tiene varias opciones para crear plantillas de dispositivo:

- Diseñe la plantilla de dispositivo en IoT Central y, después, [implemente el modelo de funcionalidad del dispositivo en el código del dispositivo](concepts-telemetry-properties-commands.md).
- Importe un modelo de funcionalidad del dispositivo desde el [catálogo de dispositivos Azure Certified for IoT](https://aka.ms/iotdevcat). Después, agregue las propiedades, las personalizaciones y los paneles en la nube que la aplicación de IoT Central necesita.
- Cree un modelo de funcionalidad del dispositivo mediante Visual Studio Code. Implemente el código del dispositivo a partir del modelo. Importe manualmente el modelo de funcionalidad del dispositivo en la aplicación de IoT Central y, a continuación, agregue las propiedades, las personalizaciones y los paneles en la nube que necesite la aplicación de IoT Central.
- Cree un modelo de funcionalidad del dispositivo mediante Visual Studio Code. Implemente el código del dispositivo desde el modelo y conecte el dispositivo real a la aplicación de IoT Central mediante una primera conexión del dispositivo. IoT Central busca e importa el modelo de funcionalidad del dispositivo desde el repositorio público. Después, puede agregar las propiedades, las personalizaciones y los paneles en la nube que la aplicación de IoT Central necesita a la plantilla del dispositivo.

También puede agregar plantillas de dispositivo a una aplicación de IoT Central mediante la [API de REST](https://docs.microsoft.com/learn/modules/manage-iot-central-apps-with-rest-api/) o la [CLI](howto-manage-iot-central-from-cli.md).

Algunas [plantillas de aplicación](concepts-app-templates.md) ya incluyen plantillas de dispositivo que son útiles en el escenario que admite la plantilla de aplicación. Por ejemplo, consulte [Arquitectura de análisis en tienda](../retail/store-analytics-architecture.md).

## <a name="create-a-device-template-from-the-device-catalog"></a>Creación de una plantilla de dispositivo desde el catálogo de dispositivos

Como desarrollador, puede empezar a compilar rápidamente la solución mediante un dispositivo certificado de IoT Plug and Play (versión preliminar). Consulte la lista en el [catálogo de dispositivos IoT de Azure](https://catalog.azureiotsolutions.com/alldevices). IoT Central se integra con el catálogo de dispositivos para que pueda importar un modelo de funcionalidad del dispositivo desde cualquiera de estos dispositivos certificados de IoT Plug and Play (versión preliminar). Para crear una plantilla de dispositivo desde uno de estos dispositivos en IoT Central:

1. Vaya a la página **Device Templates** (Plantillas de dispositivo) en la aplicación de IoT Central.
1. Seleccione **+ Nuevo** y, a continuación, seleccione cualquiera de los dispositivos certificados de IoT Plug and Play (versión preliminar) en el catálogo. IoT Central crea una plantilla de dispositivo basada en este modelo de funcionalidad del dispositivo.
1. Agregue las propiedades, las personalizaciones o las vistas en la nube a la plantilla de dispositivo.
1. Seleccione **Publicar** para que la plantilla esté disponible para que los operadores vean y conecten los dispositivos.

## <a name="create-a-device-template-from-scratch"></a>Creación de una plantilla de dispositivo desde cero

Una plantilla de dispositivo contiene:

- Un _modelo de funcionalidad del dispositivo_ que especifica la telemetría, las propiedades y los comandos que implementa el dispositivo. Estas funcionalidades están organizadas en una o varias interfaces.
- _Propiedades en la nube_ que definen la información que almacena la aplicación de IoT Central acerca de los dispositivos. Por ejemplo, una propiedad en la nube podría registrar la fecha en la que se realizó la última revisión de un dispositivo. Esta información nunca se comparte con el dispositivo.
- Las _personalizaciones_ permiten que el generador reemplace algunas de las definiciones en el modelo de funcionalidad del dispositivo. Por ejemplo, el desarrollador puede reemplazar el nombre de una propiedad del dispositivo. Los nombres de propiedad aparecen los paneles y formularios de IoT Central.
- Los _paneles y formularios_ permiten al generador crear una interfaz de usuario para que los operadores supervisen y administren los dispositivos conectados a la aplicación.

Para crear una plantilla de dispositivo en IoT Central:

1. Vaya a la página **Device Templates** (Plantillas de dispositivo) en la aplicación de IoT Central.
1. Seleccione **+ Nueva** > **Personalizada**.
1. Escriba un nombre para la plantilla, como **Sensor medioambiental**.
1. Presione **Entrar**. IoT Central crea una plantilla de dispositivo vacía.

## <a name="manage-a-device-template"></a>Administración de una plantilla de dispositivo

Puede cambiar el nombre o eliminar una plantilla desde la página principal de la plantilla.

Después de agregar un modelo de funcionalidad del dispositivo a la plantilla, puede publicarlo. No se puede conectar un dispositivo basado en esta plantilla para que los operadores puedan verlo en la página **Dispositivos** hasta que se haya publicado la plantilla.

## <a name="create-a-capability-model"></a>Creación de un modelo de funcionalidad

Para crear un modelo de funcionalidad del dispositivo, puede:

- Usar IoT Central para crear un modelo personalizado desde cero.
- Importar un modelo desde un archivo JSON. El desarrollador de dispositivos puede haber usado Visual Studio Code para crear un modelo de funcionalidad del dispositivo para la aplicación.
- Seleccione uno de los dispositivos en el catálogo de dispositivos. Esta opción importa el modelo de funcionalidad del dispositivo que el fabricante ha publicado para este dispositivo. Un modelo de funcionalidad del dispositivo importado como este se publica automáticamente.

## <a name="manage-a-capability-model"></a>Administración de un modelo de funcionalidad

Después de crear un modelo de funcionalidad del dispositivo, puede:

- Agregar interfaces al modelo. Un modelo debe tener al menos una interfaz.
- Editar los metadatos del modelo, como el identificador, el espacio de nombres y el nombre.
- Eliminar el modelo.

## <a name="create-an-interface"></a>Creación de una interfaz

Una funcionalidad del dispositivo debe tener al menos una interfaz. Una interfaz es una colección reutilizable de funcionalidades.

Para crear una interfaz:

1. Vaya al modelo de funcionalidad del dispositivo y elija **+ Agregar interfaz**.

1. En la página **Select an Interface** (Seleccione una interfaz), puede:

    - Crear una interfaz personalizada desde cero.
    - Importar una interfaz existente desde un archivo. El desarrollador de dispositivos puede haber usado Visual Studio Code para crear una interfaz para el dispositivo.
    - Elija una de las interfaces estándar, como la interfaz **Información del dispositivo**. Las interfaces estándar especifican las funcionalidades comunes a muchos dispositivos. Estas interfaces estándar las publica Azure IoT y no se pueden modificar ni editar.

1. Después de crear una interfaz, elija **Edit Identity** (Editar identidad) para cambiar el nombre para mostrar de la interfaz.

1. Si decide crear una interfaz personalizada desde cero, puede agregar las funcionalidades del dispositivo. Las funcionalidades del dispositivo son la telemetría, las propiedades y los comandos.

### <a name="telemetry"></a>Telemetría

La telemetría es un flujo de valores enviados desde el dispositivo, normalmente desde un sensor. Por ejemplo, un sensor podría informar de la temperatura ambiente.

En la siguiente tabla se muestran las opciones de configuración de una funcionalidad de telemetría:

| Campo | Descripción |
| ----- | ----------- |
| Display Name (Nombre para mostrar) | El nombre para mostrar del valor de telemetría que se usa en los paneles y formularios. |
| Nombre | El nombre del campo en el mensaje de telemetría. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. Este campo debe ser alfanumérico. |
| Capability Type (Tipo de funcionalidad) | Telemetría. |
| Semantic Type (Tipo semántico) | El tipo semántico de la telemetría, como la temperatura, el estado o el evento. La elección del tipo semántico determina cuál de los campos siguientes está disponible. |
| Schema | El tipo de datos de telemetría, como doble, cadena o vector. Las opciones disponibles vienen determinadas por el tipo semántico. El esquema no está disponible para los tipos semánticos de evento y estado. |
| severity | Solo está disponible para el tipo semántico de evento. Los niveles de gravedad son **Error**, **Información** o **Advertencia**. |
| State Values (Valores de estado) | Solo está disponible para el tipo semántico de estado. Defina los valores de estado posibles, cada uno de los cuales tiene el nombre para mostrar, el nombre, el tipo de enumeración y el valor. |
| Unidad | Una unidad para el valor de telemetría, como **mph**, **%** o **&deg;C**. |
| Display Unit (Unidad de visualización) | Una unidad de visualización para su uso en paneles y formularios. |
| Comentario | Cualquier comentario sobre la funcionalidad de telemetría. |
| Descripción | Una descripción de la funcionalidad de telemetría. |

### <a name="properties"></a>Propiedades

Las propiedades representan valores de un momento dado. Por ejemplo, un dispositivo puede usar una propiedad para notificar la temperatura objetivo que está intentando alcanzar. Puede establecer las propiedades que se pueden escribir desde IoT Central.

En la siguiente tabla se muestran las opciones de configuración de una funcionalidad de propiedad:

| Campo | Descripción |
| ----- | ----------- |
| Display Name (Nombre para mostrar) | El nombre para mostrar del valor de propiedad que se usa en los paneles y formularios. |
| Nombre | El nombre de la propiedad. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. Este campo debe ser alfanumérico. |
| Capability Type (Tipo de funcionalidad) | Propiedad. |
| Semantic Type (Tipo semántico) | El tipo semántico de la propiedad, como la temperatura, el estado o el evento. La elección del tipo semántico determina cuál de los campos siguientes está disponible. |
| Schema | El tipo de datos de la propiedad, como doble, cadena o vector. Las opciones disponibles vienen determinadas por el tipo semántico. El esquema no está disponible para los tipos semánticos de evento y estado. |
| Writeable (Grabable) | Si la propiedad no se puede escribir, el dispositivo puede notificar los valores de propiedad a IoT Central. Si la propiedad se puede escribir, el dispositivo puede notificar los valores de propiedad a IoT Central y este puede enviar actualizaciones de propiedades al dispositivo.
| severity | Solo está disponible para el tipo semántico de evento. Los niveles de gravedad son **Error**, **Información** o **Advertencia**. |
| State Values (Valores de estado) | Solo está disponible para el tipo semántico de estado. Defina los valores de estado posibles, cada uno de los cuales tiene el nombre para mostrar, el nombre, el tipo de enumeración y el valor. |
| Unidad | Una unidad para el valor de propiedad, como **mph**, **%** o **&deg;C**. |
| Display Unit (Unidad de visualización) | Una unidad de visualización para su uso en paneles y formularios. |
| Comentario | Cualquier comentario sobre la funcionalidad de propiedad. |
| Descripción | Una descripción de la funcionalidad de propiedad. |

### <a name="commands"></a>Comandos:

Puede llamar a los comandos de dispositivo desde IoT Central. Los comandos, opcionalmente, pasan parámetros al dispositivo y reciben una respuesta del dispositivo. Por ejemplo, puede llamar a un comando para que reinicie un dispositivo en 10 segundos.

En la tabla siguiente se muestran las opciones de configuración de una funcionalidad de comando:

| Campo | Descripción |
| ----- | ----------- |
| Display Name (Nombre para mostrar) | El nombre para mostrar del comando que se usa en los paneles y formularios. |
| Nombre | El nombre del comando. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. Este campo debe ser alfanumérico. |
| Capability Type (Tipo de funcionalidad) | Comando. |
| Get-Help | `SynchronousExecutionType`. |
| Comentario | Cualquier comentario sobre la funcionalidad del comando. |
| Descripción | Una descripción de la funcionalidad del comando: |
| Solicitud | Si está habilitada, una definición del parámetro de solicitud que incluye lo siguiente: nombre, nombre para mostrar, esquema, unidad y unidad de visualización. |
| Response | Si está habilitada, una definición de la respuesta del comando que incluye lo siguiente: nombre, nombre para mostrar, esquema, unidad y unidad de visualización. |

## <a name="manage-an-interface"></a>Administración de una interfaz

Si no ha publicado la interfaz, podrá editar las funcionalidades que define esta. Después de publicar la interfaz, si desea realizar cualquier cambio, deberá crear una nueva versión de la plantilla del dispositivo y controlar la versión de la interfaz. Los cambios que no requieren el control de versiones, como nombres para mostrar o unidades, se pueden realizar en la sección **Personalizar**.

También puede exportar la interfaz como un archivo JSON si desea reutilizarla en otro modelo de funcionalidad.

## <a name="add-cloud-properties"></a>Adición de propiedades de nube

Use las propiedades en la nube para almacenar información acerca de los dispositivos en IoT Central. Las propiedades en la nube no se envían nunca a un dispositivo. Por ejemplo, puede usar las propiedades en la nube para almacenar el nombre del cliente que ha instalado el dispositivo o la fecha del último servicio del dispositivo.

En la tabla siguiente se muestran las opciones de configuración de una propiedad en la nube:

| Campo | Descripción |
| ----- | ----------- |
| Display Name (Nombre para mostrar) | El nombre para mostrar del valor de propiedad en la nube que se usa en los paneles y formularios. |
| Nombre | El nombre de la propiedad en la nube. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. |
| Semantic Type (Tipo semántico) | El tipo semántico de la propiedad, como la temperatura, el estado o el evento. La elección del tipo semántico determina cuál de los campos siguientes está disponible. |
| Schema | El tipo de datos de la propiedad en la nube, como doble, cadena o vector. Las opciones disponibles vienen determinadas por el tipo semántico. |

## <a name="add-customizations"></a>Adición de personalizaciones

Use las personalizaciones cuando necesite modificar una interfaz importada o agregar características específicas de IoT Central a una funcionalidad. Y solo se pueden personalizar aquellos que no interrumpan la compatibilidad de la interfaz. Por ejemplo, puede:

- Personalizar el nombre para mostrar y las unidades de una funcionalidad.
- Agregar un color predeterminado para usarlo cuando el valor aparezca en un gráfico.
- Especificar los valores inicial, mínimo y máximo de una propiedad.

No se puede personalizar el tipo o el nombre de la funcionalidad. Si hay cambios que no se pueden realizar en la sección **Customize** (Personalizar), deberá crear una versión de la plantilla de dispositivo y la interfaz para modificar la funcionalidad.

### <a name="generate-default-views"></a>Generación de vistas predeterminadas

La generación de vistas predeterminadas es una forma rápida de visualizar la información importante del dispositivo. Puede generar hasta tres vistas predeterminadas para la plantilla de dispositivo:

- **Comandos** proporciona una vista con los comandos del dispositivo y permite que el operador los envíe al dispositivo.
- **Información general** proporciona una vista con los datos de telemetría del dispositivo, mostrando gráficos y métricas.
- **Acerca de** proporciona una vista con información del dispositivo, que muestra sus propiedades.

Una vez que haya seleccionado **Generar vistas predeterminadas**, verá que se han agregado automáticamente en la sección **Vistas** de la plantilla del dispositivo.

## <a name="add-dashboards"></a>Adición de paneles

Agregue paneles a una plantilla de un dispositivo para permitir que los operadores visualicen un dispositivo mediante gráficos y métricas. Puede tener varios paneles para una plantilla de dispositivo.

Para agregar un panel a una plantilla de dispositivo:

1. Vaya a la plantilla del dispositivo y seleccione **Vistas**.
1. Elija **Visualización del dispositivo**.
1. Escriba un nombre para el panel en **Dashboard Name** (Nombre del panel).
1. Agregue iconos al panel en la lista de iconos estáticos, de propiedades, de propiedades en la nube, de telemetría y de comandos. Arrastre y coloque los iconos que desee agregar al panel.
1. Para trazar varios valores de telemetría en un único icono de gráfico, seleccione los valores de telemetría y, a continuación, seleccione **Combinar**.
1. Configure cada icono que agregue para personalizar cómo muestra los datos. Puede hacerlo seleccionando el icono de engranaje o seleccionando **Cambiar configuración** en el icono del gráfico.
1. Organice y cambie el tamaño de los iconos en el panel.
1. Guarde los cambios.

### <a name="configure-preview-device-to-view-dashboard"></a>Configuración del dispositivo de versión preliminar para ver el panel

Para ver y probar el panel, seleccione **Configure preview device** (Configurar dispositivo de versión preliminar). Esto le permite ver el panel igual que lo ve el operador una vez publicado. Use esta opción para asegurarse de que las vistas muestran los datos correctos. Puede elegir entre las siguientes opciones:

- Ningún dispositivo de versión preliminar.
- El dispositivo de prueba real que ha configurado para la plantilla de dispositivo.
- Un dispositivo existente en la aplicación mediante el uso del identificador de dispositivo.

## <a name="add-forms"></a>Adición de formularios

Agregue formularios a una plantilla de dispositivo para permitir a los operadores administrar un dispositivo con la visualización y configuración de propiedades. Los operadores solo pueden editar las propiedades en la nube y las propiedades del dispositivo que se pueden escribir. Puede tener varios formularios para una plantilla de dispositivo.

Para agregar un formulario a una plantilla de dispositivo:

1. Vaya a la plantilla del dispositivo y seleccione **Vistas**.
1. Seleccione **Editar datos del dispositivo y la nube**.
1. Escriba un nombre para el formulario en **Form Name** (Nombre de formulario).
1. Seleccione el número de columnas que se usará para diseñar el formulario.
1. Agregue propiedades a una sección existente del formulario o seleccione las propiedades y elija **Add section** (Agregar sección). Use las secciones para agrupar las propiedades del formulario. Puede agregar un título a una sección.
1. Configure cada propiedad en el formulario para personalizar su comportamiento.
1. Organice las propiedades en el formulario.
1. Guarde los cambios.

## <a name="publish-a-device-template"></a>Publicación de una plantilla de dispositivo

Antes de poder conectar un dispositivo que implementa el modelo de funcionalidad del dispositivo, debe publicar la plantilla del dispositivo.

Después de publicar una plantilla de dispositivo, solo puede realizar cambios limitados en el modelo de funcionalidad del dispositivo. Para modificar una interfaz, debe [crear y publicar una nueva versión](./howto-version-device-template.md).

Para publicar una plantilla de dispositivo, vaya a su plantilla de dispositivo y seleccione **Publicar**.

Después de publicar una plantilla de dispositivo, el operador puede ir a la página **Dispositivos** y agregar dispositivos reales o simulados que usan la plantilla de dispositivo. Puede seguir modificando y guardando la plantilla de dispositivo mientras realiza los cambios. Si desea enviar estos cambios al operador para que los vea en la página **Dispositivos**, debe seleccionar **Publicar** cada vez.

## <a name="next-steps"></a>Pasos siguientes

Si es desarrollador de dispositivos, el siguiente paso sugerido es leer información sobre el [control de versiones de las plantillas de dispositivo](./howto-version-device-template.md).
