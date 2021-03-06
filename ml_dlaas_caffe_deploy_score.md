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

# Deploying and scoring a Caffe model

**This content has moved to a [new location](https://datascience.ibm.com/docs/content/analyze-data/ml_dlaas_model_caffe_deploy_score.html).  Check there for the most up-to-date information.**

Update any bookmarks you might have to the old location.


_____________


After a trained model has been identified for scoring, you can deploy it by using the deployment and scoring system of {{site.data.keyword.pm_full}}. The deployed model can then be used to perform online scoring.
{: shortdesc}

The [Command Line Interface (CLI)](ml_dlaas_environment.html) client or the [Python client library - `watson-machine-learning-client`](ml_dlaas_environment_pyclient.html) can be used for deploying and scoring the models.

## Prerequisites for deploying and scoring a Caffe model

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
* A file known as the weights file which is of the format: `<snapshot_prefix>_<n_iters>.caffemodel` is generated by the
training process. This file contains the weights for the neural network. The value for `snapshot_prefix` is picked up
from the solver prototxt file and `n_iters` refers to the `nth` iteration of training. The value of `n_iters` is equal
to the maximum number of iterations done for the training. This filename needs to be specified under the
`weights_file_name` key in deployment-meta.json

## Deploy a Caffe model

To deploy a Caffe model you must retrieve the model ID, which you will use to deploy the model. 

1. Identify the model ID of the model that has to be deployed. Choose one of the following techniques:
   
   **Using CLI client:**
   The `Model Id` value can be identified by using the `list models` option. The output displays the `Model Id` for the models that are stored in the {{site.data.keyword.pm_short}} repository.  
   
   From the command prompt, run the following command: 
   
   ```
   bx ml list models
   ```
   {: codeblock}
    
    Sample Output:
    
   ```
   Fetching the list of models ...
   SI No   Model Id                               Model Type       Model Name   
   1       2553cb96-0b55-4be9-adc2-43f0f810cdbf   caffe-1.0   training-83LXk0gmR model saved from CLI   
   
   1 records found.
   OK
   List models successful
   ```
    
   **Using the Python client - watson-machine-learning-client:**
   
   The `Model Id` value can be identified from the details returned by listing the models stored in the {{site.data.keyword.pm_short}} repository when you run the `client.repository.list()` API command. In the output, the `GUID` column represents the model ID of the corresponding model.
   
   From your Python client, run the following command:
   
   ```
   client.repository.list()
   ```
   {: codeblock}
   
   Sample Output:

   ```
   ------------------------------------  -------------------  --------------
   GUID                                  NAME                 FRAMEWORK
   2553cb96-0b55-4be9-adc2-43f0f810cdbf  caffe-model1-mnist   caffe-1.0
   ------------------------------------  -------------------  --------------
   ```
   
2. Deploy the model using the `Model Id` value that you identified in the preceding step.
   
   **Using CLI client:**
   From the command prompt, run the following command:
   
   ```
   bx ml deploy a8379aaa-ea31-4c22-824d-89a01315dd6d "my_deployment"
   ```
   {: codeblock}
   
   Sample Output:
   
   ```
    Deploying the model with MODEL-ID '2553cb96-0b55-4be9-adc2-43f0f810cdbf'...
    DeploymentId       a0990269-2e64-406e-b6ed-8a32fe3a9c74   
    Scoring endpoint   https://ibm-watson-ml-svt.stage1.mybluemix.net/v3/wml_instances/5f84ce0c-c13c-492a-9b8b-92d6c7077347/published_models/2553cb96-0b55-4be9-adc2-43f0f810cdbf/deployments/a0990269-2e64-406e-b6ed-8a32fe3a9c74/online   
    Name               test_caffe_mnist   
    Type               caffe-1.0
    Runtime            None Provided
    Status             DEPLOY_SUCCESS    
    Created at         2018-03-13T05:43:46.395Z   
    OK
    Deploy model successful
   ```
    
   **Using Python client - watson-machine-learning-client:** Run the following command:
   
   ```
   deployment_details = client.deployments.create(model_id, name="Mnist model deployment")
   ```
   {: codeblock}

## Online Scoring on a Caffe model

To perform online scoring on a Caffe model, you must create a scoring payload in a JSON document format. Then you must specify input data for the payload.

### Creating a scoring payload

The scoring payload must contain the following key-value entries.

| Key | Values |
| --- | --- |
| `modelId` | "Model Id" of the model that will be used for the scoring request |
| `deploymentId` | "DeploymentId" of the model that will be used for the scoring request |
| `payload` | Input data for scoring. Refer the following section on how to specify the input data |
{: caption="Table 1. Key and value entries required as scoring payload" caption-side="top"}

### Specifying the input data in the scoring payload
   
The input data for scoring is specified as a JSON object for the `payload` key in the scoring payload. The `payload` JSON object contains the key-value pairs in the following table.

| Key | Values |
| --- | --- |
|`values`| Input data for the model if it contains only one input layer|
|`keyed_values`| A list of JSON objects defined in "Table 3". This property must be used when the model contains more than 1 input layer|

{: caption="Table 2. Key and value entries required to specify input data for scoring" caption-side="top"}


| Key | Values |
| --- | --- |
|`key`| Name of the input layer used for scoring|
|`values`| Scoring input record corresponding to the input layer specified in `key` parameter|

{: caption="Table 3. Key and value entries to be specified for `keyed_values` in Table 2" caption-side="top"}
*Refer [Watson Machine Learning API documentation](http://watson-ml-v3-api.mybluemix.net/#!/Deployments/post_v3_wml_instances_instance_id_published_models_published_model_id_deployments_deployment_id_online)
for details about scoring input and output format specifications.*

</br>*NOTE*: "keyed_values" can be used even in cases where the model accepts only one input layer.

1. Creating the scoring payload. 

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
   </br>You can refer to the following two example on defining the `payload` property for a model with the input and output layers defined in the `.prototxt` file above:
   
   Method 1:
   ```
   {
       "values": [
           [0.0, 0.1, 0.2, 0.3],
           [0.4, 0.5, 0.6, 0.7]
       ] 
   }
   ```
       
   Method 2:
   ```
   {
       "keyed_values": [
           {
               "key": "data",
               "values": [
                   [0.0, 0.1, 0.2, 0.3],
                   [0.4, 0.5, 0.6, 0.7]
               ]
           }
       ]
   }
   ```
   The complete scoring payload looks as follow:
   
   Method 1:
   ```
   {   
       "modelId": "a8379aaa-ea31-4c22-824d-89a01315dd6d",
       "deploymentId": "9d6a656c-e9d4-4d89-b335-f9da40e52179"
       "payload": {
           "values": [
               [0.0, 0.1, 0.2, 0.3],
               [0.4, 0.5, 0.6, 0.7]
           ] 
       }
   }
   ```
   {:codeblock}
   
   Method 2:
   ```
      {   
          "modelId": "a8379aaa-ea31-4c22-824d-89a01315dd6d",
          "deploymentId": "9d6a656c-e9d4-4d89-b335-f9da40e52179"
          "payload": {
            "keyed_values": [
                {
                    "key": "data",
                    "values": [
                        [0.0, 0.1, 0.2, 0.3],
                        [0.4, 0.5, 0.6, 0.7]
                    ]
                }
            ]
        }
      }
      ```
   {: codeblock} 
      
   Where the values specified for the `values` key are the inputs for the single input layer of the Caffe model.
   
   *Refer [Watson Machine Learning API documentation](http://watson-ml-v3-api.mybluemix.net/#!/Deployments/post_v3_wml_instances_instance_id_published_models_published_model_id_deployments_deployment_id_online)
for complete details about scoring input and output format specifiaction.*
   
2. Extract URL for scoring. This step is required only if Python client is used for scoring. The URL for scoring should be extracted from the object returned from API used for deploying the model.
   
   **Using Python client - watson-machine-learning-client:**
   
   ```
   deployment_details = client.deployments.create(model_guid, name="Mnist model deployment")
   scoring_url = client.deployments.get_scoring_url(deployment_details))
   ```
   {: codeblock}
   
3. Perform scoring.
   
   **Using CLI client:** From the command prompt, run the following command:
   
   ```
   bx ml score scoring_payload.json
   ```
   {: codeblock}
   
   Sample Output:
   
   ```
   Fetching scoring results for the deployment 'a0990269-2e64-406e-b6ed-8a32fe3a9c74' ...
   {"values": [[0.09365840256214142, 0.16118229925632477, 0.07711118459701538, 0.08779221773147583, 0.1127830445766449, 0.11637580394744873, 0.09848705679178238, 0.1159897968173027, 0.05028100311756134, 0.08633922785520554]]}
   OK
   Score request successful
   ```
   
   **Using Python client - watson-machine-learning-client**:
   
   ```
   predictions = client.deployments.score(scoring_url, scoring_data)
   print(predictions)
   ```
   {: codeblock}
   
   Where `scoring_url` refers to the URL extracted in step and `scoring_data` refers to the scoring payload.
   
   Sample Output:
   
   ```
   {"values": [[0.09365840256214142, 0.16118229925632477, 0.07711118459701538, 0.08779221773147583, 0.1127830445766449, 0.11637580394744873, 0.09848705679178238, 0.1159897968173027, 0.05028100311756134, 0.08633922785520554]]}
   ```
   
### Interpreting the scoring output
   
The scoring output returns a JSON object where key-value pairs refers to the following.
   
| Key | Values |
| --- | --- |
|`values`| Output data for the model that contains only one output layer|
|`keyed_values`| A list of JSON objects defined in "Table 5". This property will be returned only when the model contains multiple output layers|

{: caption="Table 4. Key and value entries of the scoring output JSON" caption-side="top"}

| Key | Values |
| --- | --- |
|`key`| Name of the output layer used for scoring|
|`values`| Scoring output record corresponding to the output layer specified in `key` parameter|
{: caption="Table 5. Key and value entries to be specified for `keyed_values` in Table 4" caption-side="top"}

*Refer [Watson Machine Learning API documentation](http://watson-ml-v3-api.mybluemix.net/#!/Deployments/post_v3_wml_instances_instance_id_published_models_published_model_id_deployments_deployment_id_online)
for details about scoring input and output format specifications.*
   
Refer to the example of .prototxt provided in the preceding step. The scoring output for that `.prototxt` will be as follows:

```
{"values": ["neuron_output1", "neuron_output2", "neuron_output3", "neuron_output4", "neuron_output5", "neuron_output6", "neuron_output7", "neuron_output8", "neuron_output9", "neuron_output10"]}
```

Where the values specified for the `values` key are the outputs obtained from the single output layer of the Caffe model.
   
## Learn more

Get started using these [sample training runs](ml_dlaas_working_with_sample_models.html) or create your own [new training runs](ml_dlaas_working_with_new_models.html).

    
    

