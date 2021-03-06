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

# Persistenza dei modelli

Utilizza il servizio {{site.data.keyword.pm_full}} per fornire le versioni dei modelli e mantenere il processo di machine learning ben organizzato. Puoi farlo utilizzando le funzionalità del token di accesso, dei metadati e delle versioni del servizio {{site.data.keyword.pm_short}}.
{: shortdesc}

## Persistenza del modello e controllo delle versioni

Visto che ti sforzi continuamente di migliorare i tuoi modelli
come parte del lavoro di sviluppo e ricerca, potresti aggiungere nuove funzioni
ai modelli esistenti o ottimizzare i parametri del modello.
Quando sviluppi i modelli in questo modo interattivo,
mantenere la traccia delle modifiche può trasformarsi rapidamente in un problema. Utilizzando le seguenti funzioni del controllo delle versioni del modello di dati di
{{site.data.keyword.pm_short}}, puoi mantenere l'intero processo ben organizzato. 

*  [Generazione del token di accesso](#generating-the-access-token)
*  [Creazione di metadati della pipeline](#creating-pipeline-metadata)
*  [Creazione della versione della pipeline](#creating-pipeline-version)
*  [Caricamento del contenuto della pipeline](#uploading-pipeline-content)
*  [Creazione di metadati del modello di pipeline](#creating-pipeline-model-metadata)
*  [Creazione della versione del modello di pipeline](#creating-pipeline-model-version)
*  [Caricamento del contenuto del modello di pipeline](#uploading-pipeline-model-content)

### Generazione del token di accesso

Genera un token di accesso utilizzando l'Utente e
la Password disponibili dalla scheda Credenziali del servizio
dell'istanza del servizio {{site.data.keyword.pm_full}}. 

Esempio di
richiesta:

```
curl --basic --user username:password https://ibm-watson-ml.mybluemix.net/v3/identity/token
```
{: codeblock}

Esempio di output:

```
{"token":"**********"}
```
{: codeblock}

Utilizza il seguente comando di terminale per assegnare il tuo valore del token alla
variabile di ambiente `access_token`:

```
access_token="<token_value>"
```
{: codeblock}

Salva la pipeline e il modello della pipeline. Crea la
e i metadati del modello di pipeline e quindi crea e
carica la versione della pipeline e del modello di
pipeline. 

### Creazione di metadati della pipeline

Per creare i metadati per la tua pipeline, descrivi le proprietà di base della pipeline in una richiesta
`curl` come mostrato nel seguente
esempio:

```
curl -i \
-X POST \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer $access_token" \
-d '{ 
  "name": "sample pipeline",
  "description": "sample description",
  "author": {
    "name": "John Smith",
    "email": "john.smith@example.com"
  },
  "type": "sparkml-pipeline-2.0",
  "runtimeEnvironment": "spark-2.0"
}' \
https://ibm-watson-ml.mybluemix.net/v2/artifacts/pipelines
```
{: codeblock}

Risposta di esempio:

```
 HTTP/1.1 201 Created
 X-Backside-Transport: OK OK
 Connection: Keep-Alive
 Transfer-Encoding: chunked
 Content-Type: application/json
 Date: Fri, 17 Feb 2017 10:46:16 GMT
 Location: https://ibm-watson-ml.stage1.mybluemix.net/v2/artifacts/pipelines/223b38a6-41c6-417c-91ad-51064261b6f7
 Server: nginx/1.11.5
 X-Global-Transaction-ID: 3172115887
* Connection #0 to host ibm-watson-ml.stage1.mybluemix.net left intact
{"ok":"true"}
```
{: codeblock}

### Creazione della versione della pipeline

Per creare una versione per la tua pipeline, specifica il valore
`parentVersionHref` della tua pipeline in una richiesta `curl` come mostrato nel seguente
esempio:

```
curl -i \
-X POST \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer $access_token" \
-d '{}' \
https://ibm-watson-ml.mybluemix.net/v2/artifacts/pipelines/1314f74d-a2a7-46d3-8f11-89baa1d7ffea/versions
```
{: codeblock}

Risposta di esempio:

```
 HTTP/1.1 201 Created
 X-Backside-Transport: OK OK
 Connection: Keep-Alive
 Transfer-Encoding: chunked
 Content-Type: application/json
 Date: Fri, 17 Feb 2017 10:47:56 GMT
 Location: https://ibm-watson-ml.stage1.mybluemix.net/v2/artifacts/pipelines/223b38a6-41c6-417c-91ad-51064261b6f7/versions/0bb830fc-bc1e-439a-b929-685d9fc7712d
 Server: nginx/1.11.5
 X-Global-Transaction-ID: 3510865585
* Connection #0 to host ibm-watson-ml.stage1.mybluemix.net left intact
{"ok":"true"}
```
{: codeblock}

### Caricamento del contenuto della pipeline

Per caricare il contenuto della pipeline, è necessario che la tua pipeline sia in formato
binario. A tal fine, richiama il
metodo `save` dalla pipeline SparkML. Puoi caricare il contenuto binario fornendo l'ID
della tua pipeline e la versione in un endpoint, come mostrato nel
seguente esempio:

```
curl -i \
-X PUT \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer $access_token" \
-d <path_to_your_binary_content> \
https://ibm-watson-ml.mybluemix.net/v2/artifacts/pipelines/1314f74d-a2a7-46d3-8f11-89baa1d7ffea/versions/a535417c-ba8f-43b5-aae5-7dd8af02f518/content
```
{: codeblock}

Risposta di esempio:

```
 HTTP/1.1 200 OK
 X-Backside-Transport: OK OK
 Connection: Keep-Alive
 Transfer-Encoding: chunked
 Content-Type: application/json
 Date: Fri, 17 Feb 2017 10:50:53 GMT
 Server: nginx/1.11.5
 X-Global-Transaction-ID: 980490895
* Connection #0 to host ibm-watson-ml.stage1.mybluemix.net left intact
{"ok":"true"}
```
{: codeblock}

Dopo aver salvato la pipeline, puoi procedere con il salvataggio del
modello di pipeline.

### Creazione di metadati del modello di pipeline

Devi descrivere lo schema dei dati del tuo modello di pipeline come mostrato
nel seguente esempio:

```
curl -i \
-X POST \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer $access_token" \
-d ' {
  "name": "sample-model",
  "description": "sample description",
  "author": {
    "name": "John Smith",
    "email": "john.smith@example.com"
  },
  "pipelineVersionHref": "string",
  "type": "sparkml-model-2.0",
  "runtimeEnvironment": "spark-2.0",
  "trainingDataSchema": {
    "type": "struct",
    "fields": [
      {
        "name": "id",
        "type": "double",
        "nullable": true,
        "metadata": {}
      },
      {
        "name": "review",
        "type": "string",
        "nullable": true,
        "metadata": {}
      },
      {
        "name": "label",
        "type": "int",
        "nullable": true,
        "metadata": {}
      }
    ]
  },
  "labelCol": "string",
  "inputDataSchema": {
    "type": "struct",
    "fields": [
      {
        "name": "id",
        "type": "double",
        "nullable": true,
        "metadata": {}
      },
      {
        "name": "review",
        "type": "string",
        "nullable": true,
        "metadata": {}
      }
    ]
  }
}' \
https://ibm-watson-ml.mybluemix.net/v2/artifacts/models
```
{: codeblock}

Risposta di esempio:

```
 HTTP/1.1 201 Created
 X-Backside-Transport: OK OK
 Connection: Keep-Alive
 Transfer-Encoding: chunked
 Content-Type: application/json
 Date: Fri, 17 Feb 2017 10:58:29 GMT
 Location: https://ibm-watson-ml.stage1.mybluemix.net/v2/artifacts/models/ce274d29-49d0-43d8-adf7-6d9afa73c9cb
 Server: nginx/1.11.5
 X-Global-Transaction-ID: 437931687
* Connection #0 to host ibm-watson-ml.stage1.mybluemix.net left intact
{"ok":"true"}
```
{: codeblock}

### Creazione della versione del modello di pipeline

Per creare una versione per il tuo modello di pipeline, utilizza una richiesta `curl` per specificare
dettagli come la posizione in cui memorizzare i dati di formazione e il metodo di valutazione del modello. Vedi il seguente esempio:

```
curl -i \
-X POST \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer $access_token" \
-d ' {
  "trainingDataRef": {
    "connection":{
      "auth_url": "https://identity.open.softlayer.com",
      "project": "object_storage_008c8b63_b3d6_47c4_9da2_6c3ea6cfa713",
      "projectId": "f771e5d082e24857adb7554d15fc357c",
      "region": "dallas",
      "userId": "9b2e44721b91471ab86ecc41aeb7d37f",
      "username": "admin_926e1098fc72ff3c59d9222a2ab31a8a22143497",
      "password": "xxxxxxxxxxxxxxxx",
      "domainId": "a7b64174a5d74134b73b1533f6b99ae6",
      "domainName": "1143537",
      "role": "admin"
    },
    "source": {
      "container": "notebooks",
      "filename": "WA_Fn-UseC_-Telco-Customer-Churn.csv",
      "fileformat": "csv",
      "inferschema": 1,
      "firstlineheader": true
    }
  },
  "evaluation": {
    "method": "Binary",
    "metrics": [
      {
        "name": "areaUnderROC",
        "threshold": 0.8
      }
    ]
  }
}' \
https://ibm-watson-ml.mybluemix.net/v2/artifacts/models/30ed894b-4018-4c88-9e53-86005548317a/versions
```
{: codeblock}

Risposta di esempio:

```
 HTTP/1.1 201 Created
 X-Backside-Transport: OK OK
 Connection: Keep-Alive
 Transfer-Encoding: chunked
 Content-Type: application/json
 Date: Fri, 17 Feb 2017 11:10:12 GMT
 Location: https://ibm-watson-ml.stage1.mybluemix.net/v2/artifacts/models/ce274d29-49d0-43d8-adf7-6d9afa73c9cb/versions/8be22c6e-e214-45a4-8cfb-be4cade109ef
 Server: nginx/1.11.5
 X-Global-Transaction-ID: 285978151
* Connection #0 to host ibm-watson-ml.stage1.mybluemix.net left intact
{"ok":"true"}
```
{: codeblock}

### Caricamento del contenuto del modello di pipeline

Per caricare il contenuto del modello di pipeline, è necessario che il tuo modello di pipeline sia in
formato binario. A tal fine,
richiama il metodo `save` dal modello di pipeline SparkML. Puoi caricare il contenuto binario fornendo l'ID
della tua pipeline e la versione in un endpoint, come mostrato nel
seguente esempio:

```
curl -i \
-X PUT \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer $access_token" \
-d <path_to_your_binary_content> \
https://ibm-watson-ml.mybluemix.net/ v2/artifacts/pipelines/1314f74d-a2a7-46d3-8f11-89baa1d7ffea/versions/bb328cc3-b78d-4527-9dcc-f2172f43ae9e/content
```
{: codeblock}

Risposta di esempio:

```
 HTTP/1.1 200 OK
 X-Backside-Transport: OK OK
 Connection: Keep-Alive
 Transfer-Encoding: chunked
 Content-Type: application/json
 Date: Fri, 17 Feb 2017 11:11:23 GMT
 Server: nginx/1.11.5
 X-Global-Transaction-ID: 605948113
* Connection #0 to host ibm-watson-ml.stage1.mybluemix.net left intact
{"ok":"true"}
```
{: codeblock}

## Ulteriori informazioni

Per ulteriori informazioni sullo sviluppo del modello, fai riferimento ai seguenti blocchi appunti: 

*  [Sviluppo di modelli SparkML con Python](https://apsportal.ibm.com/exchange/public/entry/view/d80de77f784fed7915c14353512ef14d)
*  [Sviluppo di modelli SparkML con Scala](https://apsportal.ibm.com/exchange/public/entry/view/d80de77f784fed7915c1435351309e93)
