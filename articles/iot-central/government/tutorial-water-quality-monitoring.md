---
title: 'Tutorial: Creación de una aplicación de supervisión de la calidad del agua con Azure IoT Central'
description: 'Tutorial: Aprenda a crear una aplicación de supervisión de la calidad del agua mediante plantillas de aplicación de Azure IoT Central.'
author: miriambrus
ms.author: miriamb
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-central
services: iot-central
manager: abjork
ms.openlocfilehash: 77ec999d63175f63c1de6e31fdb3f72c963d228c
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "82024479"
---
# <a name="tutorial-create-a-water-quality-monitoring-application-in-azure-iot-central"></a>Tutorial: Creación de una aplicación de supervisión de la calidad del agua en Azure IoT Central



Este tutorial le guía por los pasos de creación de una aplicación de supervisión de la calidad del agua en Azure IoT Central. La aplicación se crea con la plantilla de aplicación **Water quality monitoring** de Azure IoT Central.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar la plantilla **Water quality monitoring** para crear una aplicación de supervisión de la calidad del agua
> * Explorar y personalizar el panel del operador
> * Explorar una plantilla de dispositivo de supervisión de la calidad del agua
> * Explorar los dispositivos simulados
> * Explorar y configurar reglas
> * Configurar trabajos
> * Realizar la personalización de marca de la aplicación mediante etiquetado.

## <a name="prerequisites"></a>Prerrequisitos

Se recomienda tener una suscripción a Azure para realizar este tutorial. Si no tiene una suscripción de Azure, puede crear una en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).

## <a name="create-a-water-quality-monitoring-application-in-azure-iot-central"></a>Creación de una aplicación de supervisión de la calidad del agua en Azure IoT Central

En esta sección, usará la plantilla **Water quality monitoring** de Azure IoT Central para crear una aplicación de supervisión de la calidad del agua.

1. Vaya a la [página principal de Azure IoT Central](https://aka.ms/iotcentral).

    Si tiene una suscripción a Azure, inicie sesión con las credenciales que usa para acceder a ella. Si no, inicie sesión con una cuenta Microsoft:

    ![Inicio de sesión en la cuenta de la organización](./media/tutorial-waterqualitymonitoring/sign-in.png)

1. Seleccione **Build** (Crear) en el panel izquierdo de Azure IoT Central y seleccione la pestaña **Government** (Administración Pública). En el panel Government (Administración Pública) se muestran varias plantillas de aplicación.

    ![Plantillas de aplicación de Administración Pública](./media/tutorial-waterqualitymonitoring/iotcentral-government-tab-overview1.png)

1. Seleccione la plantilla de aplicación **Water quality monitoring**. Esta plantilla incluye una plantilla de dispositivo de calidad del agua, dispositivos simulados, un panel del operador y reglas de supervisión preconfiguradas.

1. Seleccione **Crear aplicación**. Se abre el panel **New application** (Nueva aplicación) que muestra los elementos siguientes:

    * **Nombre de la aplicación**: de forma predeterminada, el nombre de la aplicación es **Water quality monitoring** seguido de una cadena de identificación única que genera Azure IoT Central. Si quiere, puede escribir un nombre para mostrar o cambiar el nombre de la aplicación más adelante.
    * **URL**: puede escribir cualquier dirección URL que desee o cambiar el valor de la dirección URL más adelante.
    * Si tiene una suscripción a Azure, escriba valores para **Directory** (Directorio), **Azure subscription** (Suscripción a Azure) y **Region** (Región). Si no tiene una suscripción, puede activar **7-day free trial** (Evaluación gratuita de 7 días) y completar la información de contacto necesaria.

    Para más información sobre los directorios y las suscripciones, consulte el inicio rápido [Creación de una aplicación](../core/quick-deploy-iot-central.md?toc=/azure/iot-central-pnp/toc.json&bc=/azure/iot-central-pnp/breadcrumb/toc.json).

1. Seleccione el botón **Create** (Crear) en la parte inferior izquierda de la página.

    ![Página de nueva aplicación de Azure IoT Central](./media/tutorial-waterqualitymonitoring/new-application-waterqualitymonitoring1.png)

    ![Información de facturación de nueva aplicación de Azure IoT Central](./media/tutorial-waterqualitymonitoring/new-application-waterqualitymonitoring1-billinginfo.png)

Ahora ha creado una aplicación de supervisión de la calidad del agua mediante la plantilla **Water quality monitoring** de Azure IoT Central.

La nueva aplicación incluye estos componentes preconfigurados:

* Paneles de operador
* Plantillas de dispositivos de supervisión de la calidad del agua
* Dispositivos de supervisión de la calidad del agua simulados
* Reglas y trabajos
* Personalización de marca con etiquetado

La aplicación se puede modificar en cualquier momento.

A continuación, explore la aplicación y realice algunas personalizaciones.

## <a name="explore-and-customize-the-operator-dashboard"></a>Exploración y personalización del panel del operador

Después de crear la aplicación, se abre el panel **Wide World water quality dashboard**.

   ![Panel de supervisión de la calidad del agua](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-dashboard1.png)

Como desarrollador, puede crear y personalizar las vistas del panel para los operadores. Pero antes de probar la personalización, explore primero el panel.

Todos los datos que se muestran en el panel se basan en los datos del dispositivo simulado, que se tratarán en la sección siguiente.

El panel incluye los siguientes tipos de iconos:

* **Icono de imagen del servicio de agua Wide World**: el primer icono de la esquina superior izquierda del panel es una imagen que muestra el servicio de agua ficticio llamado Wide World. Puede personalizar el icono para usar su propia imagen o puede quitar el icono.

* **Iconos de KPI de pH medio**: los iconos de KPI como **Average pH in the last 30 minutes** se encuentran en la parte superior del panel. Puede personalizar los iconos de KPI y establecerlos en un tipo y un intervalo de tiempo diferentes.

* **Water monitoring area map**: Azure IoT Central usa Azure Maps, que puede establecer directamente en la aplicación para mostrar la ubicación del dispositivo. También puede asignar información de ubicación de la aplicación al dispositivo y, luego, usar Azure Maps para mostrar la información en un mapa. Mantenga el mouse sobre el mapa y pruebe los controles.

* **Gráfico de mapa término de distribución de pH medio**: puede seleccionar diferentes gráficos de visualización para mostrar la telemetría del dispositivo de la manera más adecuada para su aplicación.

* **Gráfico de líneas de Critical quality indicators**: puede visualizar la telemetría del dispositivo representada en un gráfico de líneas durante un intervalo de tiempo.  

* **Gráfico de barras de Concentration of chemical agents**: puede visualizar la telemetría del dispositivo en un gráfico de barras.

* **Botón de acción**: el panel incluye un icono de acciones que puede iniciar un operador directamente desde el panel de supervisión. Un ejemplo de tales acciones podría ser el restablecimiento de las propiedades de un dispositivo.

* **Iconos de la lista de propiedades**: el panel incluye varios iconos de propiedades que representan la información de umbral, la información de estado del dispositivo y la información de mantenimiento.

### <a name="customize-the-dashboard"></a>Personalización del panel

Como desarrollador, puede personalizar las vistas del panel para los operadores.

1. Seleccione **Edit** (Editar) para personalizar el panel **Wide World water quality dashboard**. Para personalizar el panel, seleccione comandos en el menú **Edit** (Editar). Después de que el panel esté en modo de edición, puede agregar nuevos iconos o configurar los archivos existentes.

    ![Edición del panel](./media/tutorial-waterqualitymonitoring/edit-dashboard.png)

1. Seleccione **+ New** (+ Nuevo) para crear un panel que pueda configurar. Puede tener varios paneles y navegar entre ellos desde el menú del panel.

## <a name="explore-a-water-quality-monitoring-device-template"></a>Exploración de una plantilla de dispositivo de supervisión de la calidad del agua

Una plantilla de dispositivo de Azure IoT Central define las funcionalidades de un dispositivo. Las funcionalidades disponibles son la telemetría, las propiedades y los comandos. Como desarrollador, puede definir plantillas de dispositivo en Azure IoT Central que representen las funcionalidades de los dispositivos conectados. También puede crear dispositivos simulados para probar la aplicación y la plantilla de dispositivo.

La aplicación de supervisión de la calidad del agua que creó incluye una plantilla de dispositivo de supervisión de la calidad del agua.

Para ver la plantilla de dispositivo, siga estos pasos:

1. Seleccione **Device templates** (Plantillas de dispositivo) en el panel izquierdo de la aplicación en Azure IoT Central.
1. En la lista de plantillas de dispositivo, seleccione **Water Quality Monitor**. Se abre la plantilla de dispositivo.

    ![La plantilla de dispositivo](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-devicetemplate.png)

### <a name="customize-the-device-template"></a>Personalización de la plantilla de dispositivo

Practique la personalización de la siguiente configuración de la plantilla de dispositivo:

1. En el menú de la plantilla de dispositivo, seleccione **Customize** (Personalizar).
1. Vaya al tipo de telemetría **Temperature** (Temperatura).
1. Cambie el valor de **Display name** (Nombre para mostrar) a **Reported temperature** (Temperatura notificada).
1. Cambie la unidad de medida o establezca **Min value** (Valor mínimo) y **Max value** (Valor máximo).
1. Seleccione **Guardar**.

#### <a name="add-a-cloud-property"></a>Adición de una propiedad de la nube

1. En el menú de la plantilla de dispositivo, seleccione **Cloud property** (Propiedad de la nube).
1. Para agregar una nueva propiedad de la nube, seleccione **+ Add Cloud Property** (+ Agregar propiedad de la nube). En Azure IoT Central, puede agregar a un dispositivo una propiedad pertinente, aunque no se espera que la envíe el dispositivo. Un ejemplo de este tipo de propiedad sería un umbral de alerta específico del área de instalación, información de recursos o información de mantenimiento.
1. Seleccione **Guardar**.

### <a name="explore-views"></a>Exploración de las vistas

La plantilla de dispositivo de supervisión de la calidad del agua incluye vistas predefinidas. Las vistas definen cómo verán los operadores los datos del dispositivo y establecen las propiedades de la nube. Explore las vistas y practique la realización de cambios.

  ![Vistas de la plantilla de dispositivo](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-devicetemplate-views.png)

### <a name="publish-the-device-template"></a>Publicación de la plantilla de dispositivo

Si realiza algún cambio, asegúrese de seleccionar **Publish** (Publicar) para publicar la plantilla de dispositivo.

### <a name="create-a-new-device-template"></a>Creación de una nueva plantilla de dispositivo

1. Seleccione **+ New** (+ Nueva) para crear una plantilla de dispositivo y siga el proceso de creación.
1. Cree una plantilla de dispositivo personalizada o elija una del catálogo de dispositivos Azure IoT.

## <a name="explore-simulated-devices"></a>Explorar los dispositivos simulados

La aplicación de supervisión de la calidad del agua que creó con la plantilla de aplicación tiene dos dispositivos simulados. Estos dispositivos se asignan a la plantilla de dispositivo de supervisión de la calidad del agua.

### <a name="view-the-devices"></a>Visualización de los dispositivos

1. Seleccione **Devices** (Dispositivos) en el panel izquierdo de la aplicación.

   ![Dispositivos](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-devices.png)

1. Seleccione un dispositivo simulado.

    ![Selección del dispositivo 1](./media/tutorial-waterqualitymonitoring/waterqualitymonitor-device1.png)

1. En la pestaña **Cloud Properties** (Propiedades de la nube), cambie el valor de **Acidity (pH) threshold** de **8** a **9**.
1. Explore las pestañas **Device Properties** (Propiedades del dispositivo) **Device Dashboard** (Panel del dispositivo).

> [!NOTE]
> Todas las pestañas se han configurado desde las **vistas de plantilla de dispositivo**.

### <a name="add-new-devices"></a>Adición de nuevos dispositivos

En la pestaña **Devices** (Dispositivos), seleccione **+ New** )+ Nuevo) para agregar un nuevo dispositivo.

## <a name="explore-and-configure-rules"></a>Exploración y configuración de reglas

En Azure IoT Central, puede crear reglas que supervisen automáticamente la telemetría del dispositivo. Estas reglas desencadenan una acción cuando se cumple cualquiera de sus condiciones. Una posible acción es enviar notificaciones por correo electrónico. Otras posibilidades incluyen una acción de Microsoft Flow o una acción de webhook para enviar datos a otros servicios.

La aplicación de supervisión de la calidad del agua que ha creado tiene dos reglas preconfiguradas.

### <a name="view-rules"></a>Visualización de reglas

1. Seleccione **Rules** (Reglas) en el panel izquierdo de la aplicación.

   ![Reglas](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-rules.png)

1. Seleccione **High pH alert**, que es una de las reglas preconfiguradas en la aplicación.

   ![La regla de alerta de pH alto](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-highphalert.png)

   La regla **High pH alert** está configurada para comprobar la condición de acidez (pH) que sea superior a 8.

A continuación, agregue una acción de correo electrónico a la regla:

1. Seleccione **+ Email** (+ Correo electrónico).
1. En el cuadro **Display name** (Nombre para mostrar), escriba **High pH alert**.
1. En el cuadro **To** (Para), escriba la dirección de correo electrónico asociada a la cuenta de Azure IoT Central.
1. Tiene la opción de escribir una nota e incluirla en el texto del correo electrónico.
1. Seleccione **Done** (Listo) para completar la acción.
1. Seleccione **Save** (Guardar) para guardar y activar la nueva regla.

A los pocos minutos de que se cumpla la condición configurada, recibirá un correo electrónico.

> [!NOTE]
> La aplicación envía un correo electrónico cada vez que se cumple una condición. Seleccione **Disable** (Deshabilitar) en una regla para dejar de recibir correo electrónico automatizado de esa regla.
  
Para crear una regla, seleccione **Rules** (Reglas) en el panel izquierdo de la aplicación y, a continuación, seleccione **+ New** (+ Nueva).

## <a name="configure-jobs"></a>Trabajos de configuración

Con los trabajos de Azure IoT Central, puede desencadenar actualizaciones de propiedades de dispositivo o de la nube en varios dispositivos. También puede usar trabajos para desencadenar comandos de dispositivo en varios dispositivos. Azure IoT Central automatiza el flujo de trabajo.

1. Seleccione **Jobs** (Trabajos) en el panel izquierdo de la aplicación.
1. Seleccione **+New** (+ Nuevo) y configure uno o varios trabajos.

## <a name="customize-your-application"></a>Personalización de la aplicación

Como creador, puede cambiar varias opciones de configuración para personalizar la experiencia del usuario en la aplicación.

1. Seleccione **Administration** > **Customize your application** (Administración > Personalizar la aplicación).
1. En **Application logo** (Logotipo de la aplicación), seleccione **Change** (Cambiar) para elegir la imagen que quiere cargar como logotipo.
1. En **Browser icon** (Icono del explorador), seleccione **Change** (Cambiar) para elegir la imagen que aparece en las pestañas del explorador.
1. En **Browser colors** (Colores del explorador), puede reemplazar los valores predeterminados por códigos de color HTML hexadecimales.
1. Seleccione **Settings** (Configuración) para cambiar el valor de **Theme** (Tema).

   ![Personalización de la aplicación](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-customize-your-application1.png)

### <a name="update-the-application-image"></a>Actualización de la imagen de la aplicación

1. Seleccione **Administration** > **Application settings** (Administración > Configuración de la aplicación).

1. Use el botón **Select image** (Seleccionar imagen) para elegir una imagen para cargar como la imagen de la aplicación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando la aplicación, elimínela mediante los siguientes pasos:

1. Abra la pestaña **Administration** (Administración) del panel izquierdo de la aplicación.
1. Seleccione **Application settings** (Configuración de la aplicación) y seleccione el botón **Delete** (Eliminar).

    ![Eliminación de la aplicación](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-application-settings-delete-app1.png)

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre los [conceptos de la supervisión de la calidad del agua](./concepts-waterqualitymonitoring-architecture.md).
