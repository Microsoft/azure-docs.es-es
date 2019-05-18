---
title: Versión de prueba de administrador de recursos de Azure | Azure Marketplace
description: Compilación de una versión de prueba de Marketplace con Azure Resource Manager
services: Azure, Marketplace, Cloud Partner Portal,
author: pbutlerm
manager: Patrick .Butler
ms.service: marketplace
ms.topic: conceptual
ms.date: 09/13/2018
ms.author: pabutler
ms.openlocfilehash: 92c55c7f15b3f350ad802157bf401f3e75983789
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606439"
---
# <a name="azure-resource-manager-test-drive"></a>Versión de prueba de Azure Resource Manager

Este artículo se dirige a los publicadores que tienen su oferta en Azure Marketplace, o que están en AppSource, pero solo quieren compilar su versión de prueba con recursos de Azure.

Una plantilla de Azure Resource Manager (Resource Manager) es un contenedor codificado de recursos de Azure que diseñe para representar mejor la solución. Si no está familiarizado con lo que es una plantilla de Resource Manager, lectura en [descripción de las plantillas de Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) y [creación de plantillas de Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates) para asegurarse de que sabe cómo compilar y probar sus propias plantillas.

Lo que la versión de prueba hace es utilizar la plantilla de Resource Manager suministrada y realizar la implementación de todos los recursos necesarios en esa plantilla en un grupo de recursos.

Si decide compilar una versión de prueba de Azure Resource Manager, los requisitos le permiten:

- Compilar, probar y luego cargar la plantilla de Resource Manager de la versión de prueba.
- Configurar todos los metadatos necesarios y la configuración para habilitar la versión de prueba.
- Volver a publicar la oferta con la versión de prueba habilitada.

## <a name="how-to-build-an-azure-resource-manager-test-drive"></a>Compilación de una versión de prueba de Azure Resource Manager

Este es el proceso para la creación de una unidad de prueba de Azure Resource Manager:

1. Diseño de qué desea hacer en un diagrama de flujo a los clientes.
1. Defina qué experiencias desea que sus clientes a compilar.
1. En función de las definiciones anteriores, decidir qué partes y los recursos necesarios para que los clientes a lograr esa experiencia: por ejemplo, D365 instancia o un sitio Web con una base de datos.
1. Compilar localmente el diseño y la experiencia de prueba.
1. Paquete de la experiencia en implementación de una plantilla ARM y, a partir de ahí:
    1. Defina qué partes de los recursos son parámetros de entrada;
    1. ¿Cuáles son las variables;
    1. ¿Qué salidas se les proporciona la experiencia del cliente.
1. Publicar, probar y lanzarla.

El aspecto más importante sobre la compilación de una versión de prueba de Azure Resource Manager consiste en definir los escenarios que quiere que experimenten sus clientes. ¿Se trata de un producto de firewall y quiere demostrar lo bien que controla los ataques por inyección de script? ¿Se trata de un producto de almacenamiento y quiere demostrar la rapidez y sencillez con que la solución comprime archivos?

Asegúrese de que dedicar una cantidad suficiente de tiempo a evaluar cuáles son las mejores formas de hacer alarde de su producto. Específicamente en torno a todos los recursos necesarios necesitaría, ya que facilita el empaquetado de la plantilla de Resource Manager lo suficientemente más fácil.

Para continuar con nuestro ejemplo de firewall, es posible que la arquitectura necesite una dirección URL de IP pública para el servicio y otra dirección URL de IP pública para el sitio web que el firewall protege. Cada dirección IP se implementa en una máquina virtual y se conecta con un grupo de seguridad de red y la interfaz de red.

Una vez haya diseñado el paquete de recursos deseado, ahora viene la escritura y la creación de la plantilla de Resource Manager de la unidad de prueba.

## <a name="writing-test-drive-resource-manager-templates"></a>Escritura de plantillas de Resource Manager de la versión de prueba

La versión de prueba ejecuta las implementaciones de modo completamente automatizado y, por ello, las plantillas de la versión de prueba tienen algunas restricciones que se describen a continuación.

### <a name="parameters"></a>Parámetros

La mayoría de las plantillas tienen un conjunto de parámetros. Los parámetros definen nombres de recursos, tamaños de recursos (por ejemplo, los tipos de cuentas de almacenamiento o los tamaños de máquinas virtuales), nombres de usuario y contraseñas, nombres DNS, etc. Al implementar soluciones con Azure Portal, puede rellenar manualmente todos estos parámetros, elegir nombres DNS o nombres de cuenta de almacenamiento disponibles y así sucesivamente.

![Lista de parámetros de una instancia de Azure Resource Manager](./media/azure-resource-manager-test-drive/param1.png)

Pero la versión de prueba funciona en un modo totalmente automático, sin interacción humana, por lo que solo admite un conjunto limitado de las categorías de parámetro. Si un parámetro de la plantilla de Resource Manager de la versión de prueba no se incluye en alguna de las categorías admitidas, habrá que **reemplazar este parámetro por un valor constante o variable.**

Puede utilizar cualquier nombre válido para los parámetros, la versión de prueba reconoce la categoría de parámetro mediante un valor de tipo metadatos. Se **debe especificar el tipo metadatos para todos los parámetros de plantilla**, en caso contrario, la plantilla no superará la validación:

```json
"parameters": {
  ...
  "username": {
    "type": "string",
    "metadata": {
      "type": "username"
    }
  },
  ...
}
```

También es importante tener en cuenta que **todos los parámetros son opcionales**, así que si no quiere usar ninguno, no tiene que hacerlo.

### <a name="accepted-parameter-metadata-types"></a>Tipos aceptados de metadatos de parámetros

| Tipo de metadatos   | Tipo de parámetro  | DESCRIPCIÓN     | Valor de ejemplo    |
|---|---|---|---|
| **baseuri**     | string          | Identificador URI base del paquete de implementación| https:\//\<\..\>.blob.core.windows.net/\<\..\> |
| **username**    | string          | Nuevo nombre de usuario aleatorio.| admin68876      |
| **password**    | cadena segura    | Nueva contraseña aleatoria. | Lp!ACS\^2kh     |
| **Id. de sesión**   | string          | Identificador único (GUID) de sesión de la versión de prueba    | b8c8693e-5673-449c-badd-257a405a6dee |

#### <a name="username"></a>username

La versión de prueba inicializa este parámetro con un identificador **Uri base** del paquete de implementación, por lo que puede usar este parámetro para construir el Uri de los archivos incluidos en el paquete.

```json
"parameters": {
  ...
  "baseuri": {
    "type": "string",
    "metadata": {
      "type": "baseuri",
      "description": "Base Uri of the deployment package."
    }
  },
  ...
}
```

En la plantilla, puede usar este parámetro para construir un Uri de cualquier archivo desde el paquete de implementación de la versión de prueba. El ejemplo siguiente muestra cómo construir un Uri de la plantilla vinculada:

```json
"templateLink": {
  "uri": "[concat(parameters('baseuri'),'templates/solution.json')]",
  "contentVersion": "1.0.0.0"
}
```

#### <a name="username"></a>username

La versión de prueba inicializa este parámetro con un nuevo nombre de usuario aleatorio:

```json
"parameters": {
  ...
  "username": {
    "type": "string",
    "metadata": {
      "type": "username",
      "description": "Solution admin name."
    }
  },
  ...
}
```

Valor de ejemplo:

    admin68876

Puede usar nombres de usuario aleatorios o constantes en la solución.

#### <a name="password"></a>password

La versión de prueba inicializa este parámetro con una nueva contraseña aleatoria:

```json
"parameters": {
  ...
  "password": {
    "type": "securestring",
    "metadata": {
      "type": "password",
      "description": "Solution admin password."
    }
  },
  ...
}
```

Valor de ejemplo:

    Lp!ACS^2kh

Puede usar contraseñas aleatorias o constantes en la solución.

#### <a name="session-id"></a>Identificador de sesión

La versión de prueba inicializa este parámetro con un GUID único que representa el identificador de sesión de la versión de prueba:

```json
"parameters": {
  ...
  "sessionid": {
    "type": "string",
    "metadata": {
      "type": "sessionid",
      "description": "Unique Test Drive session id."
    }
  },
  ...
}
```

Valor de ejemplo:

    b8c8693e-5673-449c-badd-257a405a6dee

Puede usar este parámetro para identificar de forma única la sesión de la versión de prueba, si es necesario.

### <a name="unique-names"></a>Nombres únicos

Algunos recursos de Azure, como las cuentas de almacenamiento o los nombres DNS, requieren nombres únicos globalmente.

Esto significa que, cada vez que la versión de prueba implementa la plantilla de Resource Manager, crea un **grupo de recursos con un nombre único** para todos sus recursos \'. Por lo tanto, es preciso utilizar la función concatenada [uniquestring](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-functions#uniquestring) con los nombres de variables en los identificadores de grupo de recursos para generar valores únicos aleatorios:

```json
"variables": {
  ...
  "domainNameLabel": "[concat('contosovm',uniquestring(resourceGroup().id))]",
  "storageAccountName": "[concat('contosodisk',uniquestring(resourceGroup().id))]",
  ...
}
```

Asegúrese de concatenar las cadenas de parámetro/variable (\'contosovm\') con una única cadena de salida (\'resourceGroup().id\'), porque así garantiza la exclusividad y la confiabilidad de cada variable.

Por ejemplo, la mayoría de los nombres de recursos no pueden empezar con un dígito, pero la función de cadena única puede devolver una cadena que empiece con un dígito. Por lo tanto, si se utiliza el resultado de cadena única sin procesar, las implementaciones no se realizarán. 

Puede encontrar información adicional sobre las reglas y restricciones de nomenclatura de recursos en [este artículo](https://docs.microsoft.com/azure/guidance/guidance-naming-conventions).

### <a name="deployment-location"></a>Ubicación de la implementación

Puede poner la versión de prueba a disposición de los usuarios en distintas regiones de Azure. La idea es permitir que un usuario elija la región más cercana, para ofrecer la mejor experiencia del usuario.

Cuando la versión de prueba crea una instancia del laboratorio, siempre crea un grupo de recursos en la región elegida por un usuario y luego ejecuta la plantilla de implementación en el contexto de este grupo. Por lo tanto, la plantilla debe elegir la ubicación de implementación del grupo de recursos:

```json
"variables": {
  ...
  "location": "[resourceGroup().location]",
  ...
}
```

Y luego usar esta ubicación para todos los recursos de una instancia concreta de laboratorio:

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/publicIPAddresses",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/virtualNetworks",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/networkInterfaces",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Compute/virtualMachines",
    "location": "[variables('location')]",
    ...
  }
]
```

Debe asegurarse de que su suscripción puede implementar todos los recursos que quiere implementar en cada una de las regiones que seleccione. También debe asegurarse de que las imágenes de máquina virtual están disponibles en todas las regiones que va a habilitar, de lo contrario, la plantilla de implementación no funcionará en algunas regiones.

### <a name="outputs"></a>Salidas

Normalmente, con las plantillas de Resource Manager, puede implementar sin producir ninguna salida. Esto se debe a que conoce todos los valores que usa para rellenar los parámetros de plantilla y siempre puede inspeccionar manualmente las propiedades de cualquier recurso.

En cuanto a las plantillas de Resource Manager de la versión de prueba, es muy importante que se devuelva a la versión de prueba toda la información que sea necesaria para obtener acceso al laboratorio (identificadores URI del sitio web, nombres de host de máquina virtual, nombres de usuario y contraseñas). Asegúrese de que todos los nombres de salida sean legibles, porque estas variables se presentan al cliente.

No hay ninguna restricción relacionada con las salidas de plantilla. Tan solo recuerde que la versión de prueba convierte todos los valores de salida en **cadenas**, por lo que si envía un objeto a la salida, el usuario verá la cadena JSON.

Ejemplo:

```json
"outputs": {
  "Host Name": {
    "type": "string",
    "value": "[reference(variables('pubIpId')).dnsSettings.fqdn]"
  },
  "User Name": {
    "type": "string",
    "value": "[parameters('adminName')]"
  },
  "Password": {
    "type": "string",
    "value": "[parameters('adminPassword')]"
  }
}
```

### <a name="subscription-limits"></a>Límites de suscripción

Otra cosa que debe tener en cuenta son los límites del servicio y la suscripción. Por ejemplo, si desea implementar un máximo de diez máquinas virtuales de cuatro núcleos, tiene que asegurarse de que la suscripción que se utiliza para el laboratorio le permite usar 40 núcleos.

Puede encontrar más información sobre los límites del servicio y la suscripción de Azure en [este artículo](https://docs.microsoft.com/azure/azure-subscription-service-limits). Dado que se pueden realizar varias versiones de prueba al mismo tiempo, compruebe que la suscripción puede controlar el número de núcleos multiplicado por el número total de versiones de prueba simultáneas que se pueden realizar.

### <a name="what-to-upload"></a>Qué hay que cargar

La plantilla de Resource Manager de la versión de prueba se carga como archivo zip, que puede incluir varios artefactos de implementación, pero debe tener un archivo denominado **main-template.json**. Este archivo es la plantilla de implementación de Azure Resource Manager que la versión de prueba utiliza para crear una instancia de un laboratorio.

Si tiene otro recurso fuera de este archivo, puede hacer referencia a él como recurso externo dentro de la plantilla o puede incluir el recurso en el archivo zip.

Durante la certificación de publicación, la versión de prueba descomprime el paquete de implementación y coloca su contenido en un contenedor de blobs interno de la versión de prueba. La estructura de contenedor refleja la estructura del paquete de implementación:

| package.zip                       | Contenedor de blobs de la versión de prueba         |
|---|---|
| main-template.json                | https:\//\<\...\>.blob.core.windows.net/\<\...\>/main-template.json  |
| templates/solution.json           | https:\//\<\...\>.blob.core.windows.net/\<\...\>/templates/solution.json |
| scripts/warmup.ps1                | https:\//\<\...\>.blob.core.windows.net/\<\...\>/scripts/warmup.ps1  |


Llamamos Uri base a un Uri del contenedor de blobs. Cada revisión del laboratorio tiene su propio contenedor de blobs y, por lo tanto, cada revisión del laboratorio tiene su propio Uri base. La versión de prueba puede pasar un URI base del paquete de implementación descomprimido a la plantilla mediante los parámetros de plantilla.

## <a name="transforming-template-examples-for-test-drive"></a>Transformación de ejemplos de plantilla para la versión de prueba

El proceso de conversión de una arquitectura de recursos en una plantilla de Resource Manager de la versión de prueba puede ser abrumador. Con el fin de facilitar este proceso, hemos creado ejemplos sobre la mejor forma de [transformar las plantillas de implementación actuales](./transforming-examples-for-test-drive.md).

## <a name="how-to-publish-a-test-drive"></a>Publicación de una versión de prueba

Ahora que ha compilado la versión de prueba, en esta sección se explica cada uno de los campos necesarios para publicarla correctamente.

![Habilitación de la versión de prueba en la interfaz de usuario](./media/azure-resource-manager-test-drive/howtopub1.png)

El primer y más importante campo permite activar/desactivar si se habilita la versión de prueba en la oferta. Al seleccionar **Sí**, se presenta el resto del formulario con todos los campos obligatorios que tiene que rellenar. Si selecciona **No**, se deshabilita el formulario y, si vuelve a publicar con la versión de prueba deshabilitada, esta se quita de producción.

Nota: Si los usuarios utilizan activamente las versiones de prueba, esas versiones continuarán ejecutándose hasta que expiren sus sesiones.

### <a name="details"></a>Detalles

La siguiente sección que hay que rellenar corresponde a los detalles de la versión de prueba.

![Información más detallada de la versión de prueba](./media/azure-resource-manager-test-drive/howtopub2.png)

**Descripción:** *obligatorio* escriba aquí la descripción principal del contenido de la versión de prueba. El cliente leerá esta descripción para conocer qué escenarios abarca la versión de prueba del producto. 

**Manual del usuario:** *obligatorio* se trata del tutorial detallado de la experiencia con la versión de prueba. El cliente lo abrirá y podrá seguir exactamente lo que se quiere que haga durante la versión de prueba. Es importante que este contenido sea fácil de entender y de seguir. (Debe ser un archivo .pdf)

**Vídeo de demostración de la versión de prueba:** *Recomendable* De forma parecida al Manual del usuario, es recomendable incluir un tutorial en vídeo con la experiencia de la versión de prueba. El cliente lo verá antes o durante el uso de la versión de prueba y hará exactamente lo que desea que hagan con esa versión de prueba. Es importante que este contenido sea fácil de entender y de seguir.

- **Nombre**: título del vídeo.
- **Vínculo**: debe ser una dirección URL insertada desde la cámara o el vídeo. Abajo puede ver un ejemplo de cómo obtener la dirección URL insertada:
- **Miniatura**: Debe ser una imagen de alta calidad (533 × 324 píxeles). Aquí es recomendable realizar una captura de pantalla de alguna parte de su experiencia con la versión de prueba.

A continuación se muestra cómo aparecerán estos campos para el cliente durante su experiencia con la versión de prueba.

![Ubicación de los campos de la versión de prueba de la oferta de Marketplace](./media/azure-resource-manager-test-drive/howtopub4.png)

### <a name="technical-configuration"></a>Configuración técnica

En la siguiente sección que hay que rellenar se configura la plantilla de Resource Manager de la versión de prueba y se define cómo funciona concretamente esa instancia.

![](./media/azure-resource-manager-test-drive/howtopub5.png)

**Instancias:** *obligatorio* aquí se configura el número de instancias que se quiere, en qué regiones y la rapidez con la que sus clientes pueden obtener la versión de prueba.

- **Instancias**: en Seleccionar regiones se elige en donde se implementa la plantilla de Resource Manager de la versión de prueba. Se recomienda elegir solo una región en la que se espera que se encuentren la mayoría de los clientes.
- **Hot**: (Activas) número de instancias de la versión de prueba que ya están implementadas y en espera de acceso por región seleccionada. Los clientes tienen acceso inmediato a estas versiones de prueba en lugar de tener que esperar a una implementación. La contrapartida es que estas instancias se ejecutan siempre en su suscripción de Azure, por lo que incurrirán en un mayor costo de tiempo de actividad. Se recomienda tener **al menos una instancia activa**, ya que la mayoría de los clientes no quieren esperar a que terminen las implementaciones completas y, por lo tanto, hay un descenso en la utilización por parte de los clientes.
- **Warm**: (Semiactivas) número de instancias de la versión de prueba por región que se han implementado y después la máquina virtual se ha detenido y almacenado en Azure Storage. El tiempo de espera de las instancias semiactivas es más lento que el de las instancias activas, pero el costo de tiempo de actividad de almacenamiento es también más económico.
- **Cold**: (Inactivas) número de instancias de la versión de prueba por región que posiblemente puedan implementarse. Las instancias inactivas requieren que la plantilla de Resource Manager de la versión de prueba pase por una implementación en el momento en que el cliente solicita la versión de prueba, por lo que es más lenta que las instancias activas o semiactivas. Como contrapartida, solo tiene que pagar por la duración de la versión de prueba.

En este momento se calcula el número total de posibles versiones de prueba simultáneas que se van a poner a disposición de los usuarios y se comprueba que el límite de cuota de la suscripción puede controlar esa cantidad simultánea:

**(Número de regiones seleccionadas x instancias activas) + (número de regiones seleccionadas x instancias semiactivas) + (número de regiones seleccionadas x instancias inactivas)**

**Duración de la versión de prueba (horas):** *obligatorio* tiempo en número de horas durante el que la versión de prueba estará activa. La versión de prueba finaliza automáticamente cuando finaliza este período de tiempo.

**Test Drive Resource Manager template:** (Plantilla de Resource Manager de la versión de prueba) *obligatorio* cargue aquí la plantilla de Resource Manager. Se trata del archivo compilado en la sección anterior. Asigne al archivo de plantilla principal "main-template.json" y asegúrese de que la plantilla de Resource Manager contiene los parámetros de salida para las variables clave que se necesitan. (Debe ser un archivo .zip)

**Información de acceso:** *obligatorio* cuando un cliente obtiene su versión de prueba, se le presenta la información de acceso. Estas instrucciones tienen por objeto compartir los parámetros de salida útiles de la plantilla de Resource Manager de la versión de prueba. Para incluir parámetros de salida, use corchetes dobles (por ejemplo, **{{outputname}}**) y se insertarán correctamente en la ubicación. (En este caso se recomienda usar el formato de cadena HTML para la representación en el front-end).

### <a name="test-drive-deployment-subscription-details"></a>Detalles de suscripción de la implementación de la versión de prueba

La última sección que hay que rellenar permite implementar las versiones de prueba automáticamente mediante la conexión de la suscripción de Azure y Azure Active Directory (AD).

![Detalles de la suscripción de implementación de la versión de prueba](./media/azure-resource-manager-test-drive/subdetails1.png)

**Identificador de suscripción de Azure:** *obligatorio* concede acceso a servicios de Azure y a Azure Portal. En la suscripción es donde se notifica la utilización de recursos y se facturan los servicios. Si todavía no tiene una suscripción de Azure **independiente** solo para las versiones de prueba, consiga una. Para encontrar los identificadores de suscripción de Azure, inicie sesión en Azure Portal y navegue a Suscripciones en el menú de la izquierda. (Ejemplo: "a83645ac-1234-5ab6-6789-1h234g764ghty")

![Suscripciones de Azure](./media/azure-resource-manager-test-drive/subdetails2.png)

**Id. de inquilino de Azure AD** *obligatorio* si ya tiene un identificador de inquilino disponible, puede encontrarlo en Propiedades -\> Id. de directorio.

![Propiedades de Azure Active Directory](./media/azure-resource-manager-test-drive/subdetails3.png)

En caso contrario, cree un inquilino en Azure Active Directory.

![Lista de inquilinos de Azure Active Directory](./media/azure-resource-manager-test-drive/subdetails4.png)

![Definir la organización, el dominio y el país o región para el inquilino de Azure AD](./media/azure-resource-manager-test-drive/subdetails5.png)

![Confirmación de la selección](./media/azure-resource-manager-test-drive/subdetails6.png)

**Id. de aplicación de Azure AD:** *obligatorio* el siguiente paso consiste en crear una aplicación y registrarla. Esta aplicación se usará para realizar operaciones en la instancia de la versión de prueba.

1. Navegue al directorio recién creado o a uno que ya existe y seleccione Azure Active Directory en el panel de filtro.
2. Busque "Registros de aplicaciones" y haga clic en "Agregar".
3. Proporcione un nombre para la aplicación.
4. Como Tipo, seleccione "Aplicación web o API".
5. Puede proporcionar cualquier valor en URL de inicio de sesión ya que ese campo no se usará.
6. Haga clic en Crear.
7. Cuando haya creado la aplicación, vaya a Propiedades -\> Set the application as multi-tenant (Establecer la aplicación como multiinquilino) y presione Guardar.

Haga clic en Guardar. El último paso consiste en copiar el identificador de aplicación de esta aplicación registrada y pegarlo en el campo Versión de prueba.

![Detalle del identificador de aplicación de Azure AD](./media/azure-resource-manager-test-drive/subdetails7.png)

Dado que vamos a usar la aplicación para implementar en la suscripción, es necesario agregar la aplicación como colaborador en la suscripción. Las instrucciones son las siguientes:

1. Vaya a la hoja Suscripciones y seleccione la suscripción adecuada que va a usar solo para la versión de prueba.
1. Haga clic en **Control de acceso (IAM)**.
1. Haga clic en la pestaña **Asignaciones de roles**.  ![Adición de una nueva entidad de seguridad de Access Control](./media/azure-resource-manager-test-drive/SetupSub7_1.jpg)
1. Haga clic en **Agregar asignación de roles**.
1. Establezca el rol en **Colaborador**.
1. Escriba el nombre de la aplicación de Azure AD y selecciónela para asignar el rol.
    ![Adición de permisos](./media/azure-resource-manager-test-drive/SetupSub7_2.jpg)
1. Haga clic en **Save**(Guardar).

**Clave de aplicación de Azure AD:** *obligatorio* el último campo permite generar una clave de autenticación. En Claves, agregue una descripción de la clave, establezca la duración para que no expire nunca y luego seleccione Guardar. Esto es **importante** para evitar tener una clave expirada, lo que interrumpiría la versión de prueba en producción. Copie este valor y péguelo en el campo obligatorio Versión de prueba.

![Se muestran las claves de la aplicación de Azure AD](./media/azure-resource-manager-test-drive/subdetails8.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha rellenado todos los campos de la versión de prueba, revíselos y elija **Volver a publicar** la oferta. Una vez que la versión de prueba ha superado la certificación, debe comprobar ampliamente la experiencia del cliente en la **versión preliminar** de la oferta. Inicie una versión de prueba en la interfaz de usuario y luego abra su suscripción de Azure dentro de Azure Portal y compruebe que las versiones de prueba se han implementado por completo.

![Azure Portal](./media/azure-resource-manager-test-drive/subdetails9.png)

Es importante tener en cuenta que no ha eliminado ninguna instancia de la versión de prueba ya que están aprovisionadas para los clientes, por lo que el servicio Versión de prueba limpiará automáticamente estos grupos de recursos una vez que el cliente haya finalizado.

Cuando esté satisfecho con la oferta de versión preliminar, **publíquela**. Hay un proceso de revisión final que lleva a cabo Microsoft una vez publicada la oferta que comprueba de nuevo toda la experiencia. Si por algún motivo se rechaza la oferta, le enviaremos una notificación al contacto de ingeniería de la oferta explicando qué se necesita para arreglarla.

Si tiene más preguntas, busca consejos de solución de problemas o desea hacer la versión de prueba más satisfactoria, consulte la sección de [preguntas más frecuentes, solución de problemas y procedimientos recomendados](./marketing-and-best-practices.md).
