---
title: Procedimiento para corregir reglas predeterminadas modificadas - Azure AD Connect | Microsoft Docs
description: Obtenga información sobre cómo corregir las reglas predeterminadas modificadas que incluye Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
editor: curtand
ms.reviewer: darora10
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/21/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: fd16dd5a20a677568c928f805c1aaa5f2c222f24
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279948"
---
# <a name="fix-modified-default-rules-in-azure-ad-connect"></a>Corrección de las reglas predeterminadas modificadas en Azure AD Connect

Azure Active Directory (Azure AD) Connect utiliza reglas predeterminadas para la sincronización.  Desafortunadamente, estas reglas no se aplican de forma universal a todas las organizaciones. Según sus requisitos, es posible que deba modificarlas. En este artículo se describen dos ejemplos de las personalizaciones más comunes y se explica el método correcto para lograr estas personalizaciones.

>[!NOTE] 
> No se admite la modificación de las reglas predeterminadas existentes para lograr una personalización necesaria. Si lo hace, evitará que estas reglas se actualicen a la versión más reciente en futuras versiones. No obtendrá las correcciones de errores que necesita, ni nuevas características. En este documento se explica cómo lograr el mismo resultado sin modificar las reglas predeterminadas existentes. 

## <a name="how-to-identify-modified-default-rules"></a>Procedimiento para identificar las reglas predeterminadas modificadas
A partir de la versión 1.3.7.0 de Azure AD Connect, es fácil identificar la regla predeterminada que está modificada. Vaya a **Apps on Desktop** (Aplicaciones de escritorio) y seleccione el **editor de reglas de sincronización**.

![Azure AD Connect, con el Editor de reglas de sincronización resaltado](media/how-to-connect-fix-default-rules/default1.png)

En el editor se muestran las reglas predeterminadas modificadas con un icono de advertencia delante del nombre.

![Icono de advertencia](media/how-to-connect-fix-default-rules/default2.png)

 También se muestra una regla deshabilitada con el mismo nombre junto a estas (esta es la regla predeterminada estándar).

![Editor de reglas de sincronización, que muestra la regla predeterminada estándar y la regla predeterminada modificada](media/how-to-connect-fix-default-rules/default2a.png)

## <a name="common-customizations"></a>Personalizaciones comunes
Las siguientes son personalizaciones comunes a las reglas predeterminadas:

- Cambiar el flujo de atributos
- Cambiar el filtro de ámbito
- Cambiar la condición de combinación

Antes de cambiar las reglas:

- Deshabilite el programador de sincronización. De forma predeterminada, el programador se ejecuta cada 30 minutos. Asegúrese de que no se inicia mientras realiza cambios y soluciona problemas en las nuevas reglas. Para deshabilitar temporalmente el programador, inicie PowerShell y ejecute `Set-ADSyncScheduler -SyncCycleEnabled $false`.
 ![Comandos de PowerShell para deshabilitar al programador de sincronización](media/how-to-connect-fix-default-rules/default3.png)

- El cambio en el filtro de ámbito puede provocar la eliminación de objetos en el directorio de destino. Tenga cuidado antes de realizar cambios en el ámbito de los objetos. Se recomienda que realice los cambios en un servidor provisional antes de realizarlos en el servidor activo.
- Ejecute una vista previa en un único objeto después de agregar cualquier nueva regla, tal como se mencionó en la sección [Validación de la regla de sincronización](#validate-sync-rule).
- Ejecute una sincronización completa después de agregar nuevas reglas o modificar cualquier regla de sincronización personalizada. Esta sincronización aplica las reglas nuevas a todos los objetos.

## <a name="change-attribute-flow"></a>Cambiar el flujo de atributos
Hay tres escenarios diferentes para cambiar el flujo de atributos:
- Agregar un nuevo atributo.
- Invalidar el valor de un atributo existente.
- Optar por no sincronizar un atributo existente.

Puede realizar cualquiera de estas acciones sin modificar las reglas predeterminadas estándar.

### <a name="add-a-new-attribute"></a>Agregar un nuevo atributo
Si descubre que un atributo no fluye del directorio de origen al directorio de destino, utilice [Sincronización de Azure AD Connect: Extensiones de directorio](how-to-connect-sync-feature-directory-extensions.md) para corregirlo.

Si las extensiones no funcionan, pruebe a agregar dos nuevas reglas de sincronización como se describe en las secciones siguientes.


#### <a name="add-an-inbound-sync-rule"></a>Agregar una regla de sincronización de entrada
Una regla de sincronización de entrada significa que el origen del atributo es un espacio conector y el destino es el metaverso. Por ejemplo, para que exista un nuevo flujo de atributo de Active Directory local a Azure Active Directory, cree una nueva regla de sincronización de entrada. Inicie el **Editor de reglas de sincronización**, seleccione la dirección **Entrante** y seleccione **Agregar nueva regla**. 

 ![Editor de reglas de sincronización](media/how-to-connect-fix-default-rules/default3a.png)

Siga su propia convención de nomenclatura para asignar un nombre a la regla. En este caso, utilizamos **Custom In from AD - User**. Esto significa que la regla es personalizada y de entrada del espacio del conector de Active Directory al metaverso.   

 ![Crear regla de sincronización de entrada](media/how-to-connect-fix-default-rules/default3b.png)

Proporcione su propia descripción de la regla, para que el mantenimiento futuro de la regla sea fácil. Por ejemplo, la descripción puede basarse en cuál es el objetivo de la regla y por qué es necesaria.

Seleccione los valores necesarios para los campos **Connected System** (sistema conectado), **Connected System Object Type** (tipo de objeto del sistema conectado) y **Metaverse Object Type** (tipo de objeto del metaverso).

Especifique el valor de prioridad entre 0 y 99 (a menor número, mayor será la prioridad). Para los campos **Etiqueta**, **Habilitar sincronización de contraseña** y **Deshabilitada**, utilice las selecciones predeterminadas.

Deje **Scoping filter** (Filtro de ámbito) vacío. Esto significa que la regla se aplica a todos los objetos combinados entre el sistema conectado de Active Directory y el metaverso.

Deje **Join rules** (Reglas de unión) vacío. Esto significa que esta regla utiliza la condición de combinación definida en la regla predeterminada estándar. Este es otro motivo para no deshabilitar o eliminar la regla predeterminada estándar. Si no hay ninguna condición de combinación, el atributo no fluirá. 

Agregue las transformaciones adecuadas para el atributo. Puede asignar una constante, para que el valor de esa constante fluya hacia el atributo de destino. Puede utilizar la asignación directa entre el atributo de origen o destino. O bien puede utilizar una expresión para el atributo. Estas son varias [funciones de expresión](./reference-connect-sync-functions-reference.md) que puede utilizar.

#### <a name="add-an-outbound-sync-rule"></a>Agregar una regla de sincronización de salida
Para vincular el atributo con el directorio de destino, deberá crear una regla de salida. Esto significa que el origen es el metaverso y el destino es el sistema conectado. Para crear una regla de salida, inicie el **Editor de reglas de sincronización**, cambie la **dirección** a **saliente** y seleccione **Agregar nueva regla**. 

![Editor de reglas de sincronización](media/how-to-connect-fix-default-rules/default3c.png)

Al igual que con la regla de entrada, puede utilizar su propia convención de nomenclatura para nombrar la regla. Seleccione el **sistema conectado** como inquilino de Azure AD y seleccione el objeto de sistema conectado al que quiere asignarle el valor del atributo. Establezca el valor de precedencia entre 0 y 99. 

![Crear regla de sincronización de salida](media/how-to-connect-fix-default-rules/default3d.png)

Mantenga los campos **Scoping filter** (filtro de ámbito) y **Join rules** (reglas de unión) vacíos. Rellene la transformación como constante, directa o expresión. 

Ahora sabe cómo crear un nuevo atributo para un flujo de objeto de usuario de Active Directory a Azure Active Directory. Puede utilizar estos pasos para asignar cualquier atributo de cualquier objeto a un origen y un destino. Para obtener más información, consulte [creación de reglas de sincronización personalizadas](how-to-connect-create-custom-sync-rule.md) y [preparación para aprovisionar usuarios](/office365/enterprise/prepare-for-directory-synchronization).

### <a name="override-the-value-of-an-existing-attribute"></a>Invalidar el valor de un atributo existente
Es posible que quiera invalidar el valor de un atributo que ya está asignado. Por ejemplo, si quiere establecer siempre un valor null a un atributo en Azure AD, basta con crear una regla de solo entrada. Haga que el constante (`AuthoritativeNull`) fluya al atributo de destino. 

>[!NOTE] 
> Utilice `AuthoritativeNull` en lugar de `Null` en este caso. Debe hacerlo así porque el valor que no es null reemplaza al valor null, incluso si tiene una prioridad más baja (un valor numérico mayor en la regla). Por otro lado, las demás reglas no reemplazan a `AuthoritativeNull`por un valor distinto de null. 

### <a name="dont-sync-existing-attribute"></a>No sincronizar un atributo existente
Si quiere excluir a un atributo de la sincronización, utilice la característica de filtrado de atributos proporcionada en Azure AD Connect. Inicie **Azure AD Connect** desde el icono de escritorio y, a continuación, seleccione **Personalizar las opciones de sincronización**.

![Opciones de tarea adicionales en Azure AD Connect](media/how-to-connect-fix-default-rules/default4.png)

 Asegúrese de que la opción **Filtrado de aplicaciones y atributos de Azure AD** está marcada y, a continuación, seleccione **Siguiente**.

![Características opcionales de Azure AD Connect](media/how-to-connect-fix-default-rules/default5.png)

Desmarque los atributos que quiera excluir de la sincronización.

![Atributos de Azure AD Connect](media/how-to-connect-fix-default-rules/default6a.png)

## <a name="change-scoping-filter"></a>Cambiar el filtro de ámbito
Sincronización de Azure AD se encarga de la mayoría de los objetos. Puede reducir el ámbito de los objetos y el número de objetos que se exportarán sin cambiar las reglas de sincronización predeterminadas estándar. 

Utilice uno de los métodos siguientes para reducir el ámbito de los objetos que va a sincronizar:

- Atributo cloudFiltered
- Filtrado de la unidad de organización

Si reduce el ámbito de los usuarios que se van a sincronizar, la sincronización de hash de contraseña también se detiene para los usuarios no incluidos en el filtro. Si los objetos ya se están sincronizando, después de reducir el ámbito, los objetos no incluidos en el filtro se eliminan del directorio de destino. Por este motivo, asegúrese de seleccionar el ámbito con mucho cuidado.

>[!IMPORTANT] 
> No se recomienda ampliar el ámbito de los objetos que ya configuró Azure AD Connect. Si lo hace, dificulta que el equipo de soporte técnico de Microsoft entienda sus personalizaciones. Si debe aumentar el ámbito de los objetos, edite la regla existente, clónela y deshabilite la regla original. 

### <a name="cloudfiltered-attribute"></a>Atributo cloudFiltered
No se puede establecer este atributo en Active Directory. Establezca el valor de este atributo mediante la adición de una nueva regla de entrada. A continuación, puede utilizar **transformación** y **expresión** para establecer este atributo en el metaverso. En el ejemplo siguiente, no se quiere sincronizar a todos los usuarios cuyo nombre de departamento empieza con **HRD** (distingue mayúsculas de minúsculas):

`cloudFiltered <= IIF(Left(LCase([department]), 3) = "hrd", True, NULL)`

En primer lugar convertimos el departamento de origen (Active Directory) a minúsculas. A continuación, mediante la función `Left`, tomamos solo los tres primeros caracteres y los comparamos con `hrd`. Si coincide, el valor se establece en `True`. En caso contrario, se establece en `NULL`. Ya que el valor se estableció como null, alguna otra regla con prioridad más baja (un valor numérico más alto) puede sobrescribirlo con una condición diferente. Ejecute la vista previa en un objeto para validar la regla de sincronización como se menciona en la sección [Validación de la regla de sincronización](#validate-sync-rule).

![Crear las opciones de la regla de sincronización de entrada](media/how-to-connect-fix-default-rules/default7a.png)

### <a name="organizational-unit-filtering"></a>Filtrado de la unidad de organización
Puede crear una o varias unidades organizativas (OU) y mover los objetos que no quiera sincronizar a dichas unidades organizativas. A continuación, configure el filtrado de unidades organizativas en Azure AD Connect. Inicie **Azure AD Connect** desde el icono del escritorio y seleccione las siguientes opciones. También puede configurar el filtrado de unidades organizativas mientras instala Azure AD Connect. 

![Tareas adicionales en Azure AD Connect](media/how-to-connect-fix-default-rules/default8.png)

Siga los pasos del asistente y desmarque las unidades organizativas que no quiera sincronizar.

![Opciones de dominio y filtrado de unidades organizativas de Azure AD Connect](media/how-to-connect-fix-default-rules/default9.png)

## <a name="change-join-condition"></a>Cambiar la condición de combinación
Utilice las condiciones de combinación predeterminadas que Azure AD Connect ya configuró. Si cambia las condiciones de combinación predeterminadas, dificulta al soporte técnico de Microsoft entender las personalizaciones y ofrecer soporte técnico al producto.

## <a name="validate-sync-rule"></a>Validación de la regla de sincronización
Puede validar la regla de sincronización recién agregada mediante la característica de vista previa sin ejecutar el ciclo completo de sincronización. En Azure AD Connect, seleccione **Synchronization Service**.

![Azure AD Connect, con Synchronization Service resaltado](media/how-to-connect-fix-default-rules/default10.png)

Seleccione **Metaverse Search** (Búsqueda de metaverso). Seleccione el objeto de ámbito como **persona**, seleccione **Agregar cláusula** y mencione los criterios de búsqueda. A continuación, seleccione **Buscar** y haga doble clic en el objeto de los resultados de búsqueda. Asegúrese de que los datos en Azure AD Connect están actualizados para ese objeto mediante la ejecución de importación y sincronización en el bosque antes de ejecutar este paso.

![Synchronization Service Manager](media/how-to-connect-fix-default-rules/default11.png)

En **Metaverse Object Properties** (propiedades del objeto de metaverso), seleccione **Conectores**, luego el objeto en el conector correspondiente (bosque) y finalmente **Propiedades...** .

![Metaverse Object Properties](media/how-to-connect-fix-default-rules/default12.png)

Seleccione **Vista previa...**

![Propiedades de objeto del espacio del conector](media/how-to-connect-fix-default-rules/default13a.png)

En la ventana de vista previa, seleccione **Generate Preview** (Generar vista previa) e **Import Attribute Flow** (importar flujo de atributo) en el panel izquierdo.

![Vista previa](media/how-to-connect-fix-default-rules/default14.png)
 
En este caso, tenga en cuenta que la regla recién agregada se ejecuta en el objeto y la propiedad `cloudFiltered` está establecida en true.

![Vista previa](media/how-to-connect-fix-default-rules/default15a.png)
 
Para comparar la regla modificada con la regla predeterminada, exporte ambas reglas por separado como archivos de texto. Estas reglas se exportan como un archivo de script de PowerShell. Puede compararlas mediante cualquier herramienta de comparación de archivos (por ejemplo, WinDiff) para ver los cambios. 
 
Tenga en cuenta que en la regla modificada, el atributo `msExchMailboxGuid` se cambia al tipo **Expression** en lugar de **Direct**. Además, se cambia el valor a **NULL** y la opción a **ExecuteOnce**. Puede omitir las diferencias de identificación y prioridad. 

![resultados de la herramienta WinDiff](media/how-to-connect-fix-default-rules/default17.png)
 
Para corregir las reglas y cambiarlas a su configuración predeterminada, elimine la regla modificada y habilite la regla predeterminada. Asegúrese de que no se pierda la personalización que está intentando lograr. Cuando esté listo, ejecute **Sincronización completa**.

## <a name="next-steps"></a>Pasos siguientes
- [Hardware y requisitos previos](how-to-connect-install-prerequisites.md) 
- [Configuración rápida](how-to-connect-install-express.md)
- [Configuración personalizada](how-to-connect-install-custom.md)