---
title: Causas comunes del reciclado de roles del servicio en la nube | Microsoft Docs
description: Un rol del servicio en la nube que se recicla repentinamente puede causar un considerable tiempo de inactividad. A continuación encontrará algunos de los problemas comunes que provocan el reciclaje de los roles, lo que le ayudará a reducir el tiempo de inactividad.
services: cloud-services
documentationcenter: ''
author: simonxjx
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 533930d1-8035-4402-b16a-cf887b2c4f85
ms.service: cloud-services
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 06/15/2018
ms.author: v-six
ms.openlocfilehash: 4bf33db28d68b600d860a132027e5be440377b34
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89460293"
---
# <a name="common-issues-that-cause-roles-to-recycle"></a>Problemas comunes que causan el reciclaje de los roles
En este artículo se describen algunas de las causas comunes de problemas relacionados con la implementación y se proporcionan sugerencias para la resolución de dichos problemas. Aparece una indicación de que existe un problema con una aplicación cuando la instancia de rol no se inicia o cuando alterna entre los estados inicializando, ocupado y deteniendo.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="missing-runtime-dependencies"></a>Dependencias en tiempo de ejecución que faltan
Si un rol de la aplicación se basa en algún ensamblado que no forma parte de .NET Framework o de la biblioteca administrada de Azure, debe incluir explícitamente ese ensamblado en el paquete de aplicación. Tenga presente que otros marcos de trabajo de Microsoft no están disponibles en Azure de forma predeterminada. Si el rol se basa en uno de esos marcos de trabajo, debe agregar esos ensamblados al paquete de la aplicación.

Antes de compilar y empaquetar la aplicación, compruebe lo siguiente:

* Si usa Visual Studio, asegúrese de que la propiedad **Copy Local** se establece en **True** para cada ensamblado del proyecto que no forme parte de Azure SDK o de .NET Framework.
* Asegúrese de que el archivo web.config no hace referencia a ningún ensamblado sin usar en el elemento de compilación.
* El elemento **Build Action** de cada archivo .cshtml se establece en **Content**. Esto garantiza que los archivos aparecerán correctamente en el paquete y permite que otros archivos referenciados aparezcan en el paquete.

## <a name="assembly-targets-wrong-platform"></a>El ensamblado tiene como destino la plataforma equivocada
Azure es un entorno de 64 bits. Por lo tanto, los ensamblados de .NET compilados para un destino de 32 bits no funcionarán en Azure.

## <a name="role-throws-unhandled-exceptions-while-initializing-or-stopping"></a>El rol genera excepciones no controladas cuando se inicializa o se detiene
Todas las excepciones iniciadas por los métodos de la clase [RoleEntryPoint], que incluye los métodos [OnStart], [OnStop] y [Ejecutar], son excepciones no controladas. Si se produce una excepción no controlada en uno de estos métodos, el rol se reciclará. Si el rol se recicla de forma repetida, es posible que inicie una excepción no controlada cada vez que intente iniciarse.

## <a name="role-returns-from-run-method"></a>El rol se devuelve del método Run
El método [Ejecutar] está pensado para que se ejecute indefinidamente. Si el código invalida el método [Ejecutar] , podría quedar inactivo indefinidamente. Si se devuelve el método [Ejecutar] , el rol se recicla.

## <a name="incorrect-diagnosticsconnectionstring-setting"></a>Configuración incorrecta de DiagnosticsConnectionString
Si la aplicación usa Diagnósticos de Azure, el archivo de configuración de servicio debe especificar el valor de configuración de `DiagnosticsConnectionString` . Este valor debe especificar una conexión HTTPS a la cuenta de almacenamiento de Azure.

Para asegurarse de que el valor de `DiagnosticsConnectionString` sea correcto antes de implementar el paquete de aplicación en Azure, compruebe lo siguiente:  

* El valor de `DiagnosticsConnectionString` apunta a una cuenta de almacenamiento válida de Azure.  
  De forma predeterminada, este valor apunta a la cuenta de almacenamiento emulado, por lo que debe cambiar explícitamente esta configuración antes de implementar el paquete de aplicación. Si no cambia este valor, se produce una excepción cuando la instancia de rol intenta iniciar al monitor de diagnóstico. Esto puede hacer que la instancia de rol se recicle indefinidamente.
* La cadena de conexión se especifica en el siguiente [formato](../storage/common/storage-configure-connection-string.md). (El protocolo debe especificarse como HTTPS). Reemplace *MyAccountName* por el nombre de su cuenta de almacenamiento y *MyAccountKey* por la clave de acceso de su cuenta:    

```console
DefaultEndpointsProtocol=https;AccountName=MyAccountName;AccountKey=MyAccountKey
```

  Si está desarrollando una aplicación con Azure Tools para Microsoft Visual Studio, puede usar las páginas de propiedades para establecer este valor.

## <a name="exported-certificate-does-not-include-private-key"></a>El certificado exportado no incluye una clave privada
Para ejecutar un rol web con TLS, debe asegurarse de que el certificado de administración exportado incluya la clave privada. Si usa el *administrador de certificados de Windows* para exportar el certificado, asegúrese de seleccionar **Sí** en la opción **Exportar la clave privada**. El certificado se debe exportar al formato PFX, que es el único formato que se admite actualmente.

## <a name="next-steps"></a>Pasos siguientes
Vea más [artículos de solución de problemas](https://azure.microsoft.com/documentation/articles/?tag=top-support-issue&product=cloud-services) para servicios en la nube.

En la [series de blogs de Kevin Williamson](https://docs.microsoft.com/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data)puede ver más escenarios de reciclaje de roles.

[RoleEntryPoint]: https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx
[OnStart]: https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx
[OnStop]: https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[Ejecutar]: https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx
