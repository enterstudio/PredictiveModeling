---

copyright:
  years: 2016, 2017
lastupdated: "2017-11-16"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Gestión de los modelos SPSS desplegados

Puede utilizar la API de servicio de {{site.data.keyword.pm_full}} para cargar un archivo que contenga la rama de puntuación de IBM® SPSS® Modeler para desplegar. Cuando se haya cargado, estará disponible para puntuar datos de las aplicaciones.
{: shortdesc}

Específicamente, puede realizar las tareas siguientes:

*  [Despliegue o renovación de un modelo de predicción](#deploying-or-refreshing-a-predictive-model)
*  [Recuperación de una lista de todos los modelos desplegados actualmente](#retrieving-a-list-of-all-currently-deployed-models)
*  [Descarga una copia de un determinado archivo del modelo desplegado](#downloading-a-copy-of-a-specific-deployed-model-file)
*  [Supresión de un modelo de predicción desplegado](#deleting-a-deployed-predictive-model)

## Despliegue o renovación de un modelo de predicción

A cada archivo del modelo se le asigna un ID de contexto, que sirve como alias para hacer referencia al modelo desplegado en las siguientes llamadas al servicio. Si existe un
modelo para un ID de contexto, se sustituye por la siguiente llamada `PUT` como método
de renovar el análisis predictivo que utilizan las
aplicaciones.

Ejemplo de solicitud:

```
PUT http://{PA Bluemix load balancer
URL}/pm/v1/model/{contextId}?accesskey={access_key for this bound
application}
```
{: codeblock}

Parámetros de solicitud:

```
    Content-Type: multipart/form-data
    Parámetros:
        Parámetros de formulario:
            model_file: archivo de modelo a cargar
        Parámetros de vía de acceso:
            contextId: identificador único que se asigna a su modelo o una referencia al modelo desplegado para renovar
        Parámetros de consulta:
            accesskey: access_key de env.VCAP_SERVICES
```
{: codeblock}

Respuesta cuando el despliegue se ejecuta correctamente:

```
    Content-Type: application/json
    Status code: 200
    body:
        {
           "flag":true, 
           "message":"detailed information"  
         }
```
{: codeblock}

Respuesta cuando el despliegue falla:

```
    Content-Type: application/json
    Status code: 200
    body:
        {
           "flag":false,
           "message":"reason"
        }
```
{: codeblock}

## Recuperación de una lista de todos los modelos desplegados actualmente

Utilice la llamada de API siguiente para recuperar un resumen de todos los modelos actualmente desplegados en
esta instancia de servicio.

Ejemplo de solicitud:

```
GET http://{PA Bluemix load balancer
URL}/pm/v1/model?accesskey={access_key for this bound
application}
```
{: codeblock}

Parámetros de solicitud:

```
    Content-Type: */*
    Parámetros:
        Parámetros de consulta:
            accesskey: access_key de env.VCAP_SERVICES
```
{: codeblock}

Respuesta cuando la solicitud de resumen de un modelo desplegado se ejecuta correctamente:

```
    Content-Type: application/json
    Status code: 200
    body: una matriz vacía si no se ha desplegado ningún modelo o un resumen de los modelos desplegados...
        [
            {
                "stream":"neural_net1.str",
                "id":"1"
            },
            {
                "stream":"neural_net2.str",
                "id":"2"
            },
            ...
        ]
```
{: codeblock}

Respuesta cuando la solicitud de resumen de un modelo desplegado falla:

```
    Content-Type: application/json
    Status code: 200
    body:
        {
           "flag":false, 
           "message":"reason"  
         }
```
{: codeblock}

## Descarga una copia de un determinado archivo del modelo desplegado

Utilice la siguiente llamada de API para descargar una copia de un archivo del modelo desplegado específico.

Ejemplo de solicitud:

```
GET http://{PA Bluemix load balancer
URL}/pm/v1/model/{contextId}?accesskey={access_key for this bound
application}
```
{: codeblock}

Parámetros de solicitud:

```
    Content-Type: */*
    Parámetros:
        Parámetros de vía de acceso:
            contextId: el identificador del modelo de predicción desplegado que desea descargar
        Parámetros de consulta:
            accesskey: access_key de env.VCAP_SERVICES
```
{: codeblock}

Respuesta cuando la solicitud de descarga se ejecuta correctamente:

```
    Content-Type: application/octet-stream
    Status code: 200
    Header:
        "Content-Disposition":"attachment; filename=model_file_name.str"
```
{: codeblock}

Respuesta cuando la solicitud de descarga falla:

```
    Content-Type: application/json
    Status code: 200
    body:
        {
           "flag":false, 
           "message":String // failure reason 
        }
```
{: codeblock}

## Supresión de un modelo de predicción desplegado

Utilice la siguiente llamada de API para suprimir el modelo predictivo de la instancia de servicio de Machine
Learning. Después de ejecutar esta llamada, el modelo predictivo
dejará de estar disponible para su descarga o para puntuar datos en las
aplicaciones.

Ejemplo de solicitud:

```
DELETE http://{service
instance}/pm/v1/model/{contextId}?accesskey={access_key for this
bound application}
```
{: codeblock}

Parámetros de solicitud:

```
    Content-Type: */*
    Parámetros:
        Parámetros de vía de acceso:
            contextId: el identificador del modelo desplegado cuyo despliegue desea anular y que desea suprimir
        Parámetros de consulta:
            accesskey: access_key de env.VCAP_SERVICES
```
{: codeblock}

Respuesta cuando la eliminación del despliegue se ejecuta correctamente:

```
    Content-Type: application/json
    Status code: 200
    body:
        {
           "flag":true, 
           "message":"detailed information"  
         }
```
{: codeblock}

Respuesta cuando la eliminación del despliegue falla:

```
    Content-Type: application/json
    Status code: 200
    body:
        {
           "flag":false,
           "message":"reason"
        }
```
{: codeblock}

## Información adicional

¿Preparado para ponerse en marcha? Para crear una instancia de servicio o enlazar
una aplicación, consulte [Utilización del servicio con modelos Spark y Python](using_pm_service_dsx.html) o
[Utilización del servicio con modelos IBM® SPSS®](using_pm_service.html).

Para obtener más información sobre la API, consulte [API del servicio para modelos Spark y Python](pm_service_api_spark.html) o [API del servicio para modelos IBM® SPSS®](pm_service_api_spss.html).

Para obtener más información sobre IBM® SPSS® Modeler y los algoritmos de modelado que proporciona,
consulte [IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/SS3RA7).

Para obtener más información sobre IBM Data Science Experience y los algoritmos de modelado que proporciona, consulte [https://datascience.ibm.com](https://datascience.ibm.com).
