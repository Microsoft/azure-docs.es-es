---
title: 'Seguridad de la capa de transporte (TLS): Hiperescala (Citus) en Azure Database for PostgreSQL'
description: 'Instrucciones e información para configurar Azure Database for PostgreSQL: Hiperescala (Citus) y las aplicaciones asociadas, a fin de usar correctamente las conexiones TLS.'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 07/16/2020
ms.openlocfilehash: 659f86a107e4b08db4ec5195635ea32d2260d677
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071468"
---
# <a name="configure-tls-in-azure-database-for-postgresql---hyperscale-citus"></a>Configuración de TLS in Azure Database for PostgreSQL: Hiperescala (Citus)
El nodo de coordinación de Hiperescala (Citus) requiere que las aplicaciones cliente se conecten a Seguridad de la capa de transporte (TLS). La exigencia de TLS entre el servidor de base de datos y las aplicaciones cliente ayuda a mantener la confidencialidad de los datos en tránsito. La configuración de comprobación adicional que se describe a continuación también protege frente a ataques de tipo "man in the middle".

## <a name="enforcing-tls-connections"></a>Aplicación de conexiones TLS
Las aplicaciones usan una "cadena de conexión" para identificar la base de datos y la configuración de destino de una conexión. Los distintos clientes requieren una configuración diferente. Para ver una lista de las cadenas de conexión que usan los clientes comunes, consulte la sección **Cadenas de conexión** para el grupo de servidores en Azure Portal.

Los parámetros de TLS `ssl` y `sslmode` varían en función de las capacidades del conector; por ejemplo: `ssl=true` o `sslmode=require` o `sslmode=required`.

## <a name="ensure-your-application-or-framework-supports-tls-connections"></a>Comprobación de que la aplicación o el marco de trabajo admiten conexiones TLS
Algunos marcos de trabajo de la aplicación no habilitan TLS de forma predeterminada para las conexiones de PostgreSQL. Sin embargo, sin una conexión segura, una aplicación no se puede conectar a un nodo de coordinador de Hiperescala (Citus). Consulte la documentación de la aplicación para aprender a habilitar las conexiones TLS.

## <a name="applications-that-require-certificate-verification-for-tls-connectivity"></a>Aplicaciones que requieren la verificación del certificado para la conectividad TLS
En algunos casos, las aplicaciones requieren un archivo de certificado local generado a partir de un archivo de certificado (.cer) de una entidad de certificación (CA) de confianza para conectarse de forma segura. El certificado para conectarse a un servidor Azure Database for PostgreSQL: Hiperescala (Citus) se encuentra en https://cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem. Descargue el archivo de certificado y guárdelo en la ubicación que prefiera.

> [!NOTE]
>
> Para comprobar la autenticidad del certificado, puede verificar su huella digital SHA-256 mediante la herramienta de línea de comandos OpenSSL:
>
> ```sh
> openssl x509 -in DigiCertGlobalRootCA.crt.pem -noout -sha256 -fingerprint
>
> # should output:
> # 43:48:A0:E9:44:4C:78:CB:26:5E:05:8D:5E:89:44:B4:D8:4F:96:62:BD:26:DB:25:7F:89:34:A4:43:C7:01:61
> ```

### <a name="connect-using-psql"></a>Conexión mediante psql
En el ejemplo siguiente se muestra cómo conectarse a su nodo de coordinación de Hiperescala (Citus) mediante la utilidad de línea de comandos psql. Use la configuración de la cadena de conexión `sslmode=verify-full` para aplicar la comprobación del certificado TLS. Pase la ruta de acceso al archivo del certificado local al parámetro `sslrootcert`.

Luego aparece un ejemplo de la cadena de conexión psql:
```
psql "sslmode=verify-full sslrootcert=DigiCertGlobalRootCA.crt.pem host=mydemoserver.postgres.database.azure.com dbname=citus user=citus password=your_pass"
```
> [!TIP]
> Confirme que el valor pasado a `sslrootcert` coincide con la ruta de acceso al archivo del certificado que guardó.

## <a name="next-steps"></a>Pasos siguientes
Aumente aún más la seguridad con las [reglas de firewall de Azure Database for PostgreSQL: Hiperescala (Citus)](concepts-hyperscale-firewall-rules.md).
