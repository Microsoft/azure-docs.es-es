---
title: Tipos de versiones de prueba en el marketplace comercial de Microsoft
description: Tipos de versiones de prueba en el marketplace comercial
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 06/19/2020
ms.author: keferna
author: keferna
ms.openlocfilehash: 92fd4d629585ed465e2891be2dce1c1bdc8c88e6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287938"
---
# <a name="azure-resource-manager-test-drive"></a>Versión de prueba de Azure Resource Manager

Use este tipo si tiene una oferta en Azure Marketplace o AppSource, pero quiere crear una versión de prueba solo con recursos de Azure. Una plantilla de Azure Resource Manager (ARM) consiste en un contenedor codificado de recursos de Azure que el usuario diseña para que represente mejor su solución. La versión de prueba utiliza la plantilla de ARM proporcionada e implementa todos los recursos necesarios en un grupo de recursos. Es la única opción de versión de prueba para las ofertas de máquina virtual o de aplicación de Azure.

Si no está familiarizado con el concepto de plantilla de ARM, lea [¿Qué es Azure Resource Manager?](../azure-resource-manager/resource-group-overview.md) y [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/resource-group-authoring-templates.md) para comprender mejor cómo compilar y probar sus propias plantillas.

Para obtener información sobre una versión de prueba **hospedada** o de **aplicación lógica**, vea [¿Qué es una versión de prueba?](what-is-test-drive.md)

## <a name="technical-configuration"></a>Configuración técnica

Una plantilla de implementación contiene todos los recursos de Azure que componen la solución. Los productos que se ajustan a este escenario usan solo recursos de Azure. Establezca las siguientes propiedades en el Centro de partners:

- **Regiones** (obligatorias): Actualmente hay 26 regiones admitidas de Azure en las que se puede usar la versión de prueba. La habitual es que desee que su versión de prueba esté disponible en las regiones en que prevé un mayor número de clientes, con el fin de que puedan seleccionar la región más cercana para lograr el mejor rendimiento posible. Tendrá que asegurarse de que su suscripción puede implementar todos los recursos necesarios en cada una de las regiones que seleccione.

- **Instancias**: Seleccione el tipo (activo o inactivo) y el número de instancias disponibles, que se multiplicarán por el número de regiones en las que está disponible la oferta.

  - **Activa**: Este tipo de instancia se implementa y está en espera de acceso por cada región seleccionada. Los clientes pueden acceder al instante a las instancias *frecuentes* de una versión prueba, en lugar de tener que esperar a una implementación. La contrapartida es que estas instancias se ejecutan siempre en su suscripción de Azure, por lo que incurrirán en un mayor costo de tiempo de actividad. Se recomienda tener al menos una instancia *frecuente*, ya que la mayoría de los clientes no quieren esperar a que terminen las implementaciones completas, lo que genera un descenso en la utilización por parte de los clientes si no hay ninguna instancia *frecuente*.

  - **Inactiva**: Este tipo de instancia representa el número total de instancias que se pueden implementar por región. Las instancias inactivas requieren que se implemente toda la plantilla de la versión de prueba de Resource Manager en el momento en que el cliente solicita la versión de prueba, por lo que las instancias *inactivas* tardan mucho más tiempo en cargarse que las *activas*. A cambio solo tiene que pagar por la duración de la versión de prueba, *no* se ejecuta de forma ininterrumpida en su suscripción de Azure como las instancias *activas*.

- **Versión de prueba de la plantilla de Azure Resource Manager**: Cargue el archivo. zip que contiene la plantilla de Azure Resource Manager. Para más información acerca de cómo crear una plantilla de Azure Resource Manager, consulte el artículo del tutorial de inicio rápido [Creación e implementación de plantillas de Azure Resource Manager mediante Azure Portal](../azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal.md).

- **Duración de la versión de prueba** (obligatorio): especifique el número de horas que la versión de prueba permanecerá activa. La versión de prueba termina automáticamente al finalizar este período. Usar solo números enteros (por ejemplo, "2" horas es válido, mientras que "1,5" no lo es).

## <a name="write-the-test-drive-template"></a>Escribir la plantilla de la versión de prueba

Una vez diseñado el paquete de recursos deseado, escriba y compile la plantilla de ARM de la versión de prueba. Como la versión de prueba ejecuta las implementaciones de modo completamente automatizado, las plantillas de la versión de prueba tienen algunas restricciones:

### <a name="parameters"></a>Parámetros

La mayoría de las plantillas tienen una serie de parámetros que definen nombres de recursos, tamaños de recursos (por ejemplo, los tipos de cuentas de almacenamiento o los tamaños de máquinas virtuales), nombres de usuario y contraseñas, nombres DNS, etc. Al implementar soluciones con Azure Portal, puede rellenar manualmente todos estos parámetros, elegir nombres DNS o nombres de cuenta de almacenamiento disponibles y así sucesivamente.

![Lista de parámetros de una instancia de Azure Resource Manager](media/test-drive/resource-manager-parameters.png)

No obstante, la versión de prueba funciona de forma automática, sin interacción humana, por lo que solo admite un conjunto limitado de las categorías de parámetro. Si un parámetro de la plantilla de ARM de la versión de prueba no se incluye en alguna de las categorías admitidas, habrá que reemplazar este parámetro por un valor constante o variable.

Puede utilizar cualquier nombre válido para los parámetros; la versión de prueba reconoce la categoría de parámetro mediante un valor de tipo metadatos. Especifique el tipo metadatos para todos los parámetros de la plantilla; de lo contrario, la plantilla no superará la validación:

```JSON
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

> [!NOTE]
> Todos los parámetros son opcionales, por lo que si no desea usar ninguno, no tiene que hacerlo.

### <a name="accepted-parameter-metadata-types"></a>Tipos aceptados de metadatos de parámetros

| Tipo de metadatos   | Tipo de parámetro  | Descripción     | Valor de ejemplo    |
|---|---|---|---|
| **baseuri**     | string          | Identificador URI base del paquete de implementación| `https:\//\<\..\>.blob.core.windows.net/\<\..\>` |
| **username**    | string          | Nuevo nombre de usuario aleatorio.| admin68876      |
| **password**    | cadena segura    | Nueva contraseña aleatoria. | Lp!ACS\^2kh     |
| **Id. de sesión**   | string          | Id. exclusivo (GUID) de sesión de la versión de prueba    | b8c8693e-5673-449c-badd-257a405a6dee |

#### <a name="baseuri"></a>baseuri

La versión de prueba inicializa este parámetro con un identificador **URI base** del paquete de implementación, por lo que puede usar este parámetro para construir un URI de los archivos incluidos en el paquete.

```JSON
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

Use este parámetro dentro de la plantilla para construir un URI de cualquier archivo desde el paquete de implementación de la versión de prueba. En el ejemplo siguiente se muestra cómo construir un URI de la plantilla vinculada:

```JSON
"templateLink": {
  "uri": "[concat(parameters('baseuri'),'templates/solution.json')]",
  "contentVersion": "1.0.0.0"
}
```

#### <a name="username"></a>username

La versión de prueba inicializa este parámetro con un nuevo nombre de usuario aleatorio:

```JSON
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

Valor de ejemplo: `admin68876`

Puede usar nombres de usuario aleatorios o constantes en la solución.

#### <a name="password"></a>password

La versión de prueba inicializa este parámetro con una nueva contraseña aleatoria:

```JSON
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

Valor de ejemplo: `Lp!ACS^2kh`

Puede usar contraseñas aleatorias o constantes en la solución.

#### <a name="session-id"></a>Identificador de sesión

La versión de prueba inicializa este parámetro con un GUID único que representa el identificador de sesión de la versión de prueba:

```JSON
"parameters": {
  ...
  "sessionid": {
    "type": "string",
    "metadata": {
      "type": "sessionid",
      "description": "Unique test drive session id."
    }
  },
  ...
}
```

Valor de ejemplo: `b8c8693e-5673-449c-badd-257a405a6dee`

Puede usar este parámetro para identificar de forma única la sesión de la versión de prueba, si es necesario.

### <a name="unique-names"></a>Nombres únicos

Algunos recursos de Azure, como las cuentas de almacenamiento o los nombres DNS, requieren nombres únicos globalmente. Esto significa que, cada vez que la versión de prueba implementa la plantilla de ARM, crea un grupo de recursos con un nombre único para todos sus recursos. Por lo tanto, debe utilizar la función concatenada [uniquestring](../azure-resource-manager/templates/template-functions.md) con los nombres de variables en los identificadores de grupo de recursos para generar valores únicos aleatorios:

```JSON
"variables": {
  ...
  "domainNameLabel": "[concat('contosovm',uniquestring(resourceGroup().id))]",
  "storageAccountName": "[concat('contosodisk',uniquestring(resourceGroup().id))]",
  ...
}
```

Asegúrese de concatenar las cadenas de parámetro/variable (`contosovm`) con una única cadena de salida (`resourceGroup().id`), porque así garantiza la exclusividad y confiabilidad de cada variable.

Por ejemplo, la mayoría de los nombres de recursos no pueden empezar con un dígito, pero la función de cadena única puede devolver una cadena que empiece con un dígito. Por lo tanto, si se utiliza el resultado de cadena única sin procesar, las implementaciones no se realizarán.

Puede encontrar información adicional sobre las reglas y restricciones de nomenclatura de recursos en [este artículo](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging).

### <a name="deployment-location"></a>Ubicación de implementación

Puede poner la versión de prueba a disposición de los usuarios en distintas regiones de Azure. La idea es permitir que un usuario elija la región más cercana, para ofrecer la mejor experiencia del usuario.

Cuando la versión de prueba crea una instancia del laboratorio, siempre crea un grupo de recursos en la región elegida por un usuario y luego ejecuta la plantilla de implementación en el contexto de este grupo. Por lo tanto, la plantilla debe elegir la ubicación de implementación del grupo de recursos:

```JSON
"variables": {
  ...
  "location": "[resourceGroup().location]",
  ...
}
```

Y luego usar esta ubicación para todos los recursos de una instancia concreta de laboratorio:

```JSON
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

Asegúrese de que su suscripción puede implementar todos los recursos deseados en cada una de las regiones que seleccione. Asegúrese también de que las imágenes de máquina virtual están disponibles en todas las regiones que va a habilitar; de lo contrario, la plantilla de implementación no funcionará en algunas regiones.

### <a name="outputs"></a>Salidas

Normalmente, con las plantillas de Resource Manager, puede implementar sin producir ninguna salida. Esto se debe a que conoce todos los valores que usa para rellenar los parámetros de plantilla y siempre puede inspeccionar manualmente las propiedades de cualquier recurso.

En cuanto a las plantillas de Resource Manager de la versión de prueba, es importante que se devuelva a la versión de prueba toda la información que sea necesaria para acceder al laboratorio (identificadores URI del sitio web, nombres de host de máquina virtual, nombres de usuario y contraseñas). Asegúrese de que todos los nombres de salida sean legibles, porque estas variables se presentan al cliente.

No hay ninguna restricción relacionada con las salidas de plantilla. La versión de prueba convierte todos los valores de salida en cadenas, por lo que, si envía un objeto a la salida, un usuario verá la cadena JSON.

Ejemplo:

```JSON
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

No olvide los límites del servicio y la suscripción. Por ejemplo, si desea implementar un máximo de diez máquinas virtuales de cuatro núcleos, tiene que asegurarse de que la suscripción que se utiliza para el laboratorio le permite usar 40 núcleos. Para más información sobre los límites del servicio y la suscripción de Azure, vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md). Dado que se pueden realizar varias versiones de prueba al mismo tiempo, compruebe que la suscripción puede controlar el número de núcleos multiplicado por el número total de versiones de prueba simultáneas que se pueden realizar.

### <a name="what-to-upload"></a>Qué hay que cargar

La plantilla de ARM de la versión de prueba se carga como archivo ZIP, que puede incluir varios artefactos de implementación, pero debe tener un archivo denominado `main-template.json`. Esta es la plantilla de implementación de ARM que la versión de prueba usa para crear una instancia de un laboratorio. Si tiene otros recursos fuera de este archivo, puede hacer referencia a ellos como recurso externo dentro de la plantilla o incluirlos en el archivo ZIP.

Durante la certificación de publicación, la versión de prueba descomprime el paquete de implementación y coloca su contenido en un contenedor de blobs interno de la versión de prueba. La estructura de contenedor refleja la estructura del paquete de implementación:

| package.zip                       | Contenedor de blobs de la versión de prueba         |
|---|---|
| `main-template.json`                | `https:\//\<\...\>.blob.core.windows.net/\<\...\>/main-template.json`  |
| `templates/solution.json`           | `https:\//\<\...\>.blob.core.windows.net/\<\...\>/templates/solution.json` |
| `scripts/warmup.ps1`                | `https:\//\<\...\>.blob.core.windows.net/\<\...\>/scripts/warmup.ps1`  |
|||

Llamamos Uri base a un Uri del contenedor de blobs. Dado que cada revisión del laboratorio tiene su propio contenedor de blobs, cada revisión del laboratorio tiene su propio identificador URI base. La versión de prueba puede pasar un identificador URI base del paquete de implementación descomprimido a la plantilla mediante los parámetros de plantilla.

### <a name="transform-template-examples-for-test-drive"></a>Transformación de ejemplos de plantilla para la versión de prueba

El proceso de conversión de una arquitectura de recursos en una plantilla de Resource Manager de la versión de prueba puede ser abrumador. Para obtener ayuda adicional, vea estos ejemplos que muestran cómo transformar mejor las plantillas de implementación actuales en [¿Qué es una versión de prueba?](what-is-test-drive.md#transforming-examples)

## <a name="test-drive-deployment-subscription-details"></a>Detalles de la suscripción de implementación de la versión de prueba

La última sección que hay que completar permite implementar las versiones de prueba automáticamente mediante la conexión de la suscripción de Azure y Azure Active Directory (AD).

![Detalles de la suscripción de implementación de la versión de prueba](media/test-drive/deployment-subscription-details.png)

1. Obtenga un **Identificador de suscripción de Azure**. Este identificador concede acceso a servicios de Azure y a Azure Portal. En la suscripción es donde se notifica la utilización de recursos y se facturan los servicios. Si todavía no tiene una suscripción de Azure independiente solo para las versiones de prueba, consiga una. Puede buscar los identificadores de suscripción de Azure (como `1a83645ac-1234-5ab6-6789-1h234g764ghty1`); para ello, inicie sesión en Azure Portal y seleccione **Suscripciones** en el menú de navegación izquierdo.

   ![Suscripciones de Azure](media/test-drive/azure-subscriptions.png)

2. Obtenga un **Identificador de inquilino de Azure AD**. Si ya tiene un identificador de inquilino disponible, puede encontrarlo en **Azure Active Directory** > **Propiedades** > **Id. de directorio**:

   ![Propiedades de Azure Active Directory](media/test-drive/azure-active-directory-properties.png)

   Si no tiene un identificador de inquilino, cree uno en Azure Active Directory. Para obtener ayuda con la configuración de un inquilino, vea [Inicio rápido: Configuración de un inquilino](../active-directory/develop/quickstart-create-new-tenant.md).

3. **Identificador de aplicación de Azure AD**: cree una aplicación y regístrela. Esta aplicación se usará para realizar operaciones en la instancia de la versión de prueba.

   1. Navegue al directorio recién creado o a uno que ya existe y seleccione Azure Active Directory en el panel de filtro.
   2. Busque **Registros de aplicaciones** y seleccione **Agregar**.
   3. Proporcione un nombre para la aplicación.
   4. Seleccione el **Tipo** de **Aplicación web o API**.
   5. Proporcione cualquier valor en la dirección URL de inicio de sesión; este campo no se usa.
   6. Seleccione **Crear**.
   7. Una vez creada la aplicación, seleccione **Propiedades** > **Set the application as multi-tenant** (Establecer la aplicación como multiinquilino) y, después, **Guardar**.

4. Seleccione **Guardar**.

5. Copie el identificador de aplicación de esta aplicación registrada y péguelo en el campo de la versión de prueba.

   ![Detalle del identificador de aplicación de Azure AD](media/test-drive/azure-ad-application-id-detail.png)

6. Dado que vamos a usar la aplicación para implementar en la suscripción, es necesario agregar la aplicación como colaborador en la suscripción:

   1. Seleccione el tipo de **Suscripción** que va a usar para la versión de prueba.
   1. Seleccione **Access Control (IAM)** .
   1. Seleccione la pestaña **Asignaciones de roles** y, después, **Agregar asignación de roles**.

      ![Adición de una nueva entidad de seguridad de Access Control](media/test-drive/access-control-principal.jpg)

   1. Establezca **Rol** y **Asignar acceso a** como se muestra. Escriba el nombre de la aplicación de Azure AD en el campo **Seleccionar**. Seleccione la aplicación a la que desea asignar el rol **Colaborador**.

      ![Adición de permisos](media/test-drive/access-control-permissions.jpg)

   1. Seleccione **Guardar**.

7. Genere una clave de autenticación para la **Aplicación de Azure AD**. En **Claves**, agregue una **Descripción de la clave**, establezca la duración en **No expira nunca** (una clave expirada interrumpirá la versión de prueba en producción) y, después, seleccione **Guardar**. Copie este valor y péguelo en el campo obligatorio de la versión de prueba.

![Se muestran las claves de la aplicación de Azure AD](media/test-drive/azure-ad-app-keys.png)

## <a name="republish"></a>Volver a publicar

Ahora que todos los campos de la versión de prueba están completos, **vuelva a publicar** la oferta. Una vez que la versión de prueba ha superado la certificación, compruebe la experiencia del cliente en la versión preliminar de la oferta:

1. Inicie una versión de prueba en la interfaz de usuario.
1. Abra la suscripción de Azure en Azure Portal.
1. Compruebe que la versión de prueba se está implementando correctamente.

   ![Azure portal](media/test-drive/azure-portal.png)

No elimine ninguna instancia de la versión de prueba aprovisionada para los clientes; el servicio de la versión de prueba limpiará automáticamente estos grupos de recursos una vez que el cliente haya finalizado.

Cuando esté satisfecho con la oferta de versión preliminar, es el momento de **publicarla**. Hay un proceso de revisión final para comprobar la experiencia completa de un extremo a otro. Si rechazamos la oferta, enviaremos por correo electrónico el contacto de ingeniería de su oferta con la explicación de lo que es necesario corregir.

## <a name="next-steps"></a>Pasos siguientes

- Si estaba siguiendo las instrucciones para crear la oferta en el Centro de partners, use la flecha hacia atrás para volver a ese tema.
- Obtenga más información sobre otros tipos de versiones de prueba en [¿Qué es una versión de prueba?](what-is-test-drive.md)
