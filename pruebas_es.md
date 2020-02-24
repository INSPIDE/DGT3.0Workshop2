## **Autor**

| Empresa | Fecha | Versión |
|:-:|:-:|:-:|
| <img src="/images/inspide2.png" alt="INSPIDE" width="100"/>  | 12/03/2019 | 2.0 |

## **Ponente**


| Imagen  |  Nombre |  Empresa |Cargo|email|Linkedin|
|:-:|:-:|:-:|:-:|:-:|:-:|
|  <img src="/images/jgcasta_bn.png" alt="INSPIDE" width="50"/> | José Gómez Castaño  | <img src="/images/inspide2.png" alt="INSPIDE" width="100"/>   | **CTO** | jgcasta@inspide.com 	| [linkedin.com/in/josegomezcastano](https://linkedin.com/in/josegomezcastano) |

## Contacto de Soporte

Para solventar cualquier duda o incidencia se ha abierto un buzón de correo en el que se atenderán estas

soporte@cmobility30.es


# Información **práctica** <a name="id1"></a>

<img src="/images/question.png" alt="API" width="20"/> **Documentación del API Bandeja de Salida**


La información del API de la **Bandeja de Salida** se encuentra en las siguientes url:   [Swagger](https://app.swaggerhub.com/apis-docs/jgcasta/DGT3.0-v16/1.0#/) 

<img src="/images/question.png" alt="API" width="20"/> **Herramientas**


Para la ejecución de las pruebas es necesario el uso de un Cliente REST. Se propone la *descarga* de [Postman](https://www.getpostman.com/downloads/) como cliente para el seguimiento del workshop.

<img src="/images/question.png" alt="API" width="20"/> **Diagrama de secuencia**

<img src="/images/flujo_v16.png">

##  Generación automática de un cliente <a name="id1.1"></a>

Al estar basado en SWAGGER la publicación del API, es posible utilizar la utilidad [swagger-codegen-cli](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.swagger%22%20AND%20a%3A%22swagger-codegen-cli%22) para generar una aplicación completa, a partir del [fichero JSON obtenido de la descarga](https://github.com/INSPIDE/DGT3.0Workshop2/blob/master/aux/swagger.json)

Teniendo la utilidad en el mismo directorio que el fichero api-docs.json hay que ejecutar el comando.

```sh
java -jar swagger-codegen-cli-2.4.1.jar generate \
  -i bandejadesalida_1.0.json \
  --api-package es.xxxxxx.dgt30.v16.client.api \
  --model-package es.xxxxxx.dgt30.v16.client.model \
  --invoker-package es.xxxxxx.dgt30.v16.client.invoker \
  --group-id es.xxxxxx \
  --artifact-id es.xxxxxx.dgt30.v16 \
  --artifact-version 0.0.1-SNAPSHOT \
  -l java \
  --library resttemplate \
  -o DGT30BandejaSalidaClient
```

## Mensaje a enviar con la notificación de un evento

La conjunto de los atributos del mensaje se encuentra descrita en el [documento](https://github.com/INSPIDE/DGT3.0Workshop2/blob/master/aux/protocolo_v0_3.pdf)

```json
{
	"message_ver" : "1.0",
	"idcompany":"v16.example.com",
	"token": "asdfasdfasdfasdf",
	"actionid": "1",
	"event_detection_type":1,
	"detection_time": "2020-02-24 07:17:27",
	"lon": -3.5 ,
	"lat": 40.5,
	"device_event_type": 1,
	"device_event_type_value": 1,
	"information_quality": 1,
	"heading": 100,
	"station_type": 1,
	"speed": 0,
	"ambient_temperature": 25,
	"lane_position": 1,
	"use": 1
}
```

##  Tablas maestras <a name="id21"></a>

Para interpretar la codificación contenida en el mensaje POST a enviar al servicio, es necesario consultar ciertas Tablas Auxiliares.

Los atributos están codificados y pueden consultarse invocando al método GET de la tabla maestra correspondiente. Estas están constantemente actualizadas en https://app.swaggerhub.com/apis-docs/jgcasta/DGT3.0-v16/1.0#/

## Cambios en las tablas Maestras

Los cambios en alguna de las tablas es posible conocerlos consultando la operación /api/masterTables/1.0/getChangeTables que devuelve la última fecha de actualización de cada tabla

```json
{
    "info_code": 0,
    "info_desc": "OK",
    "data": [
        {
            "ts": "2020-02-24T07:53:24.090+0000",
            "name": "v16_device_events_types"
        },
        {
            "ts": "2020-02-24T07:54:21.671+0000",
            "name": "v16_event_detection_types"
        },
        {
            "ts": "2020-02-24T07:54:21.680+0000",
            "name": "v16_lane_positions"
        },
        {
            "ts": "2020-02-24T07:54:21.681+0000",
            "name": "v16_station_types"
        },
        {
            "ts": "2020-02-24T07:54:21.683+0000",
            "name": "v16_uses"
        }
    ]
}
```

# Ejercicios **prácticos** <a name="id2"></a>

##  Identificación en el servicio <a name="id21"></a>

Para llevar a cabo las operaciones del API es necesario obtener un token de sesión que caducará de forma aleatoria a lo largo de la misma. La operación que permite obtenerlo es **/api/1.0/getToken** que devuelve el siguiente resultado:

<img src="images/diagramasecuencia.png" alt="Ayuda" />

```json
{
  "info_code": 0,
  "info_desc": "OK",
  "data": [
    {
      "token": "28a9e96167a6ee0f84bb9c46e8a3b381032f7d9de59ce882539db044e4ee691f"
    }
  ]
}
```
El proceso de autenticación está basado en certificados digitales de cliente. Para la realización de pruebas y posterior pase a producción, el cliente deberá cumplimentar la documentación proporcionada en la que se le solicitan los datos necesarios de la empresa que vaya a consumir los servicios. El certificado será emitido por la PKI de la plataforma y entregado físicamente al responsable del mismo.

> <img src="/images/explain.png" alt="Ayuda" width="20"/>		**Explicación**
>
> Se trata de una identificación correcta gracias a la introducción de los valores `token`e`idcompany`. El token es el proporcionado temporalmente por el servicio, e idcompany corresponde al campo CN del certificado digital de cliente.
>

## Envío de evento

Desde un cliente, una vez obtenido el token de autorización, solo hay que invocar a la operación POST  **/api/v16/1.0/postincidence**

```json
{
	"message_ver" : "1.0",
	"idcompany":"v16.example.com",
	"token": "asdfasdfasdfasdf",
	"actionid": "1",
	"event_detection_type":1,
	"detection_time": "2020-02-24 07:17:27",
	"lon": -3.5 ,
	"lat": 40.5,
	"device_event_type": 1,
	"device_event_type_value": 1,
	"information_quality": 1,
	"heading": 100,
	"station_type": 1,
	"speed": 0,
	"ambient_temperature": 25,
	"lane_position": 1,
	"use": 1
}
```

El resultado de esta operación será

```json
{
    "info_code": 0,
    "info_desc": "OK",
    "data": []
}
```

© 2020 DGT. Todos los derechos reservados.
