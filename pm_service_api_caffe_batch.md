---

copyright:
  years: 2016, 2018
lastupdated: "2018-02-21"

---
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:caption: .caption}
{:pre: .pre}
{:table: .aria-labeledby="caption"}

# Deploying and scoring a Caffe model in Batch

**This content has moved to a [new location](https://datascience.ibm.com/docs/content/analyze-data/pm_service_api_caffe_batch.html). Check there for the most up-to-date information.** 

Update any bookmarks you might have to the old location.


_____________


After a trained model has been identified for serving, you can deploy it by using the batch deployment. The batch deployment will read the input data file from input connection and save the prediction result to files in a output connection.
{: shortdesc}

## Prerequisites for deploying and serving a Caffe model in Batch

Before you deploy a Caffe model and score it, the following prerequisites must be implemented at the time you train the model:

* You need to train the model using the training service. Details on how to train a model can be found [here](https://datascience.ibm.com/docs/content/analyze-data/ml_dlaas_working_with_new_models.html)
* The following files are required as part of the training payload:
    * `<network_definition_1>.prototxt`: This file describes the layout of the neural network for training
    * `<network_definition_2>.prototxt`: This file describes the layout of the neural network for scoring
    * `<network_solver>.prototxt`: This file describes the training parameters like number of training iterations,
    learning rate, training checkpoints (called snapshots), prefix used to name the snapshots (`snapshot_prefix`), etc.
    </br>*Note*: `snapshot_prefix` key should contain the value in the following format:
      `./model/<model_prefix>`
    * `deployment-meta.json`: This file contains details such as: input layers, output layers, name of the network file
    and the name of the network weights file. This file will be used by the scoring service to detect the
    correct `.prototxt` and `.caffemodel` file for loading the model. The input and output layers specified in this file
    are used by the scoring service for making predictions.
    </br>A sample deployment-meta.json looks as follows:
    ```
    {
        "input_layers": ["data"],
        "output_layers": ["probability"],
        "network_definitions_file_name": "lenet.prototxt",
        "weights_file_name": "lenet_iter_10000.caffemodel"
    }
    ```
* The training command for caffe model: The `"command"` field used in the training-runs.yml/json file should be as follows:
    ```
    mkdir $RESULT_DIR/model; cp *.prototxt $RESULT_DIR/model; \
    cp deployment-meta.json $RESULT_DIR/model; ln -s $RESULT_DIR/model model; \
    caffe train -solver <solver>.prototxt
    ```
* You can use Experiments to train and save the model.
* A file known as the weights file which is of the format: `<snapshot_prefix>_<n_iters>.caffemodel` is generated by the
training process. This file contains the weights for the neural network. The value for `snapshot_prefix` is picked up
from the solver prototxt file and `n_iters` refers to the `nth` iteration of training. The value of `n_iters` is equal
to the maximum number of iterations done for the training. This filename needs to be specified under the
`weights_file_name` key in deployment-meta.json
* The model input layer's shape must be defined to accept batch of input records.
* [Cloud Object Storage](https://console.bluemix.net/catalog/services/cloud-object-storage) instance details has to be provided during the batch deployment. This is used for storing the input data files for the prediction and storing the prediction result.
* Model should be persisted in WML Repository.

## Generating the access token

Generate an access token by using the user and password available
from the Service Credentials tab of the {{site.data.keyword.pm_full}} service instance.

Request example:

```
curl --basic --user username:password https://ibm-watson-ml.mybluemix.net/v3/identity/token
```
{: codeblock}

Output example:

```
{"token":"**********"}
```
{: codeblock}

Use the following terminal command to assign your token value to
the environment variable token:

```
token="<token_value>"
```
{: codeblock}

## Working with published models

Use the following API call to get your instance details, which include the following items:

* published models `url` value
* deployments `url` value
* usage information

Request example:

```
curl -X GET --header "Content-Type: application/json" --header "Accept: application/json" --header "Authorization: Bearer $token" https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}
```
{: codeblock}

Output example:

```
{
	"metadata": {
		"guid": "5e89b7d3-962c-4ba4-a7d8-a5ce31fc4d76",
		"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}",
		"created_at": "2018-03-14T07:23:04.749Z",
		"modified_at": "2018-03-14T08:11:32.812Z"
	},
	"entity": {
		"source": "Bluemix",
		"published_models": {
			"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models"
		},
		"usage": {
			"expiration_date": "2018-04-01T00:00:00.000Z",
			"computation_time": {
				"limit": 180000,
				"current": 867
			},
			"model_count": {
				"limit": 200,
				"current": 2
			},
			"prediction_count": {
				"limit": 5000,
				"current": 0
			},
			"gpu_count": {
				"limit": 8,
				"current": 0
			},
			"capacity_units": {
				"limit": 180000000,
				"current": 867830
			},
			"deployment_count": {
				"limit": 5,
				"current": 0
			}
		},
		"plan_id": "3f6acf43-ede8-413a-ac69-f8af3bb0cbfe",
		"status": "Active",
		"organization_guid": "96370d2b-69f6-4919-8a4f-ed5e826f8d5f",
		"region": "us-south",
		"account": {
			"id": "b56398ea52f470c3173f4cf3bef5cc7e",
			"name": "IBM",
			"type": "TRIAL"
		},
		"owner": {
			"ibm_id": "*********",
			"email": "**********************",
			"user_id": "842c9fdb-189f-47e4-95e5-51ee9d1a14a1",
			"country_code": "POL",
			"beta_user": true
		},
		"deployments": {
			"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/deployments"
		},
		"space_guid": "aa3b3ebd-4b5d-4f52-b8a0-308d090b41d6",
		"plan": "lite"
	}
}
```
{: codeblock}

By supplying the **published_models** `url` value, you can use the following API call to get the model details:

Request example:

```
curl -X GET --header "Content-Type: application/json" --header "Accept: application/json" --header "Authorization: Bearer $token" https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/
```
{: codeblock}

Output example:

```
{
	"limit": 1000,
	"resources": [{
		"metadata": {
			"guid": "c7dd457e-5b93-48dc-892a-61c92e718708",
			"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708",
			"created_at": "2018-03-14T07:33:05.593Z",
			"modified_at": "2018-03-14T07:33:05.647Z"
		},
		"entity": {
			"runtime_environment": "python-2.7",
			"learning_configuration_url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708/learning_configuration",
			"name": "training-XPqQhBgmR model saved from CLI",
			"description": "saved from WML CLI",
			"label_col": "",
			"learning_iterations_url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708/learning_iterations",
			"feedback_url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708/feedback",
			"latest_version": {
				"url": "https://ibm-watson-ml.mybluemix.net/v3/ml_assets/models/c7dd457e-5b93-48dc-892a-61c92e718708/versions/dae7d519-1eab-4cbc-aa42-cd9725b24369",
				"guid": "dae7d519-1eab-4cbc-aa42-cd9725b24369",
				"created_at": "2018-03-14T07:33:05.647Z"
			},
			"model_type": "caffe-1.0",
			"deployments": {
				"count": 0,
				"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708/deployments"
			},
			"evaluation_metrics_url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708/evaluation_metrics"
		}
	}],
	"first": {
		"url": "https://deployment/v3/wml_instances/{instance_id}/published_models/?limit=1000"
	}
}
```
{: codeblock}

Note the **deployments** `url` value that you need to create the following batch deployment.

## Creating a batch deployment with Cloud Object Storage

To use a REST API call to create a batch deployment of your caffe model, provide the following details:

*  The access token created in the previous step
*  Cloud Object Storage details, which will be used for storing the input data files for the prediction and for storing the prediction result.
*  To create a deployment, use the **deployments** `url` value from previous section.
*  Before performing batch deployment and scoring on a caffe model, you must upload the input data files in Input Connection(Cloud Object Storage).
   
### Add the input data files in the Input Connection for Caffe model
   
The input records for batch scoring must be specified as a JSON object in separate files. The JSON object contains the key-value pairs in the following table.

| Key | Values |
| --- | --- |
|Input layer name specified in .prototxt | Scoring input record corresponding to the input layer specified by key of this JSON object|
{: caption="Table 2. Key and value entries required to specify input data for batch scoring" caption-side="top"}

* These input data files needs to be placed in the root of bucket in Input Connection(COS) and each input data file must have a single scoring record. 
* Multiple inputs records can provided by placing in separate input data files. 
* You must make sure only input data files are present in the input bucket of COS.
* Dimensions of input data specified in the file must match the dimension of the corresponding input layer.

1. Creating of input data file
   
   Let us say you have a model defined with the `.prototxt` file defined as below as below:
      
```
layer {
   name: "data"
   type: "Input"
   top: "data"
   input_param { shape: { dim: 64 dim: 1 dim: 28 dim: 28 } }
}
layer {
   name: "prob"
   type: "Softmax"
   bottom: "ip2"
   top: "prob'
}
```
{: codeblock}
   
   You can refer to the following example on defining the `input_data_file` for a model with the input and output layers defined in the `.prototxt` file above:
   
   Method 1:
   ```
   {
     "data": [[[[0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]]]]
   }
   ```
   
   *Refer [Watson Machine Learning API documentation](http://watson-ml-v3-api.mybluemix.net/#!/Deployments/post_v3_wml_instances_instance_id_published_models_published_model_id_deployments)
   for complete details about scoring input format specification.*

Request example:

```
curl -v -X POST \
    -H "Content-Type:application/json" \
    -H "Authorization:Bearer $token" \
    -H "X-Spark-Service-Instance: $spark_credentials" \
    -d '{
      "name":"MNIST Prediction",
      "type": "batch",
      "description": "Batch Deployment",
       "input":{
          "type": "cloudobjectstorage",  
          "source": {
          	 "bucket": "batchcaffe-in"
          },
          "connection": {
            "access_key": "*****",
            "secret_key": "***********",
            "url": "********************"
          }
       },
       "output":{
          "type": "cloudobjectstorage",  
          "target": {
          	 "bucket": "batchcaffe-out"
          },
       "connection": {
          "access_key": "*******",
          "secret_key": "**********",
          	 "url": "********************"
          }
       }
    }' \
    https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/{published_model_id}/deployments
```
{: codeblock}

Output example:

```
{
	"metadata": {
		"guid": "451ea861-6e90-47e8-b613-5d2483603c2e",
		"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708/deployments/451ea861-6e90-47e8-b613-5d2483603c2e",
		"created_at": "2018-03-14T09:32:19.843Z",
		"modified_at": "2018-03-14T09:32:20.350Z"
	},
	"entity": {
		"runtime_environment": "python-2.7",
		"name": "batchDeploymentTest",
		"description": "Testdescription",
		"published_model": {
			"name": "training-XPqQhBgmR model saved from CLI",
			"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708",
			"guid": "c7dd457e-5b93-48dc-892a-61c92e718708",
			"description": "saved from WML CLI",
			"created_at": "2018-03-14T08:52:01.814Z"
		},
		"model_type": "caffe-1.0",
		"status": "INITIALIZING",
		"output": {
			"type": "cloudobjectstorage",
			"target": {
				"bucket": "batchcaffe-out"
			},
			"connection": {
				"access_key": "**********",
				"secret_key": "*******************",
				"url": "******************************"
			}
		},
		"type": "batch",
		"deployed_version": {
			"url": "https://ibm-watson-ml.mybluemix.net/v3/ml_assets/models/c7dd457e-5b93-48dc-892a-61c92e718708/versions/dae7d519-1eab-4cbc-aa42-cd9725b24369",
			"guid": "dae7d519-1eab-4cbc-aa42-cd9725b24369"
		},
		"input": {
			"type": "cloudobjectstorage",
			"source": {
				"bucket": "batchcaffe-fvt-in"
			},
			"connection": {
				"access_key": "**********",
				"secret_key": "**********************",
				"url": "********************************"
			}
		}
	}
}
```
{: codeblock}

**Note**: You can also use the Dashboard to create a batch
deployment.

## Obtaining deployment details

You can check the status and parameters related to the deployment model by using the **metadata** `url` value. 
Request example:

```
curl -v -X GET -H "Content-Type:application/json" -H "Authorization: Bearer $token" https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/{published_model_id}/deployments/{deployment_id}
```
{: codeblock}

Output example:

```
{
	"metadata": {
		"guid": "451ea861-6e90-47e8-b613-5d2483603c2e",
		"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708/deployments/451ea861-6e90-47e8-b613-5d2483603c2e",
		"created_at": "2018-03-14T09:32:19.843Z",
		"modified_at": "2018-03-14T09:32:20.350Z"
	},
	"entity": {
		"runtime_environment": "python-2.7",
		"name": "batchDeploymentTest",
		"description": "Testdescription",
		"published_model": {
			"name": "training-XPqQhBgmR model saved from CLI",
			"url": "https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/c7dd457e-5b93-48dc-892a-61c92e718708",
			"guid": "c7dd457e-5b93-48dc-892a-61c92e718708",
			"description": "saved from WML CLI",
			"created_at": "2018-03-14T08:52:01.814Z"
		},
		"status_details": {
			"status": "SUCCESS"
		},
		"model_type": "caffe-1.0",
		"status": "SUCCESS",
		"output": {
			"type": "cloudobjectstorage",
			"target": {
				"bucket": "batchcaffe-out"
			},
			"connection": {
				"access_key": "*******",
				"secret_key": "**************",
				"url": "****************************"
			}
		},
		"type": "batch",
		"deployed_version": {
			"url": "https://ibm-watson-ml.mybluemix.net/v3/ml_assets/models/c7dd457e-5b93-48dc-892a-61c92e718708/versions/dae7d519-1eab-4cbc-aa42-cd9725b24369",
			"guid": "dae7d519-1eab-4cbc-aa42-cd9725b24369"
		},
		"input": {
			"type": "cloudobjectstorage",
			"source": {
				"bucket": "batchcaffe-fvt-in"
			},
			"connection": {
				"access_key": "**********",
				"secret_key": "********************",
				"url": "*******************************"
			}
		}
	}
}
```
{: codeblock}

The prediction result is saved to a json file in the IBM Cloud Object
Storage. Refer to the following sample row for an example of the output.

Input data file preview: (filename: `mnist_1.json`)
   
```
{
  "data": [[[[0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]]]]
}
```
{: codeblock}

Output file preview:

```
{
	"mnist_1.json": {
		"prob": [0.03850690275430679, 0.7881755232810974, 0.0047960542142391205, 0.004213341046124697, 0.03766868636012077, 0.03975054621696472, 0.024196786805987358, 0.04613213241100311, 0.0025692235212773085, 0.013990823179483414]
	}
}
```
{: codeblock}

## Deleting a batch deployment

To delete the deployment use the following query:

Request example:

```
curl -v -X DELETE -H "Content-Type:application/json" -H
"Authorization: Bearer $token" https://ibm-watson-ml.mybluemix.net/v3/wml_instances/{instance_id}/published_models/{published_model_id}/deployments/{deployment_id}
```
{: codeblock}

Output example:

```
HTTP/1.1 204 No Content
X-Backside-Transport: OK OK
Connection: Keep-Alive
Cache-Control: no-cache, no-store, must-revalidate
Date: Wed, 28 Jun 2017 11:54:19 GMT
Pragma: no-cache
Server: nginx/1.11.5
X-Content-Type-Options: nosniff
X-Xss-Protection: 1; mode=block
X-Global-Transaction-ID: 1600446575
```
{: codeblock}

### Interpreting the scoring output
   
The scoring output returns a JSON object where key-value pairs refers to the following.
   
| Key | Values |
| --- | --- |
|Name of the output layer used for scoring | Prediction result obtained from the corresponding output layer|
{: caption="Table 3. Key and value entries of the scoring output JSON" caption-side="top"}

**Note:** First key in object array of each prediction result is filename of input data file. This is required to map input data file with its corresponding prediction result. 
   
The scoring output will be formatted as in the following example:
   
```
{
	"mnist_1.json": {
		"prob": [0.03850690275430679, 0.7881755232810974, 0.0047960542142391205, 0.004213341046124697, 0.03766868636012077, 0.03975054621696472, 0.024196786805987358, 0.04613213241100311, 0.0025692235212773085, 0.013990823179483414]
	}
}
```

   
## Learn more

Get started using these [sample training runs](ml_dlaas_working_with_sample_models.html) or create your own [new training runs](ml_dlaas_working_with_new_models.html).

For more information about IBM Data Science Experience and the modeling
algorithms it provides, see [https://datascience.ibm.com](https://datascience.ibm.com).