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

# オンライン・モデルのデプロイ

{{site.data.keyword.pm_full}} サービスを使用して、モデルをデプロイし、デプロイされたモデルに対してスコアリング要求を行うことによって予測分析を生成することができます。
{: shortdesc}

**シナリオ名**: 顧客満足度予測。

**シナリオの説明**: ある通信会社が、解約する恐れがあるのはどの顧客であるかを知りたいと考えています。その会社は、その質問に回答するソリューションを提供するように求めています。データ・サイエンティストが、予測モデルを準備し、それを開発者と共有します。開発者のタスクは、モデルをデプロイし、デプロイ済みモデルに対してスコアリング要求を行うことにより、予測分析を生成することです。

## サンプル・モデルの使用

1. [ここ](https://github.com/pmservice/wml-sample-models/blob/master/spss/customer-satisfaction-prediction/model/customer-satisfaction-prediction.str)で、Git リポジトリーからサンプル・モデルをダウンロードします。

2. 新規モデルをアップロードするには、以下の要求を使用します。

   ```
   curl -X PUT -H "Content-Type:multipart/form-data;charset=utf-8" -F "model_file=@customer-satisfaction-prediction.str" https://ibm-watson-ml.mybluemix.net/pm/v1/model/{context_id}?accesskey={accesskey_value}
   ```
   {: codeblock}

   応答:

   ```
   {"flag":true,"message":"success to upload stream with given context id context_csp2","url":"http://pmdevlb.pmservice.ibmcloud.com:80/pm/v1/model/context_csp2?accesskey=pcB4lKG1brpgSCEonNoBdzew9kiOOzN8awh1cJ2sHAxf1yyjh50nnrQngWw4DD/tM13eGXGHaJ0voQU+cAi1t/nmJBaSgi+xeMY8Wia68PB227SsqjgA5nvrX+eU9Sbr"}
```
   {: codeblock}

3. 既存のモデルを最新表示するには、以下の要求を使用します。

   ```
   curl -X PUT -H "Content-Type:multipart/form-data;Charset=UTF-8" -F "model_file=@customer-satisfaction-prediction.str" https://ibm-watson-ml.mybluemix.net/pm/v1/model/{context_id}?accesskey={accesskey_value}
   ```
   {: codeblock}

   応答:

   ```
   {"flag":true,"message":"success to update stream with given context id context_csp2","url":"http://pmdevlb.pmservice.ibmcloud.com:80/pm/v1/model/context_csp2?accesskey=pcB4lKG1brpgSCEonNoBdzew9kiOOzN8awh1cJ2sHAxf1yyjh50nnrQngWw4DD/tM13eGXGHaJ0voQU+cAi1t/nmJBaSgi+xeMY8Wia68PB227SsqjgA5nvrX+eU9Sbr"}
```
   {: codeblock}

4. すべてのデプロイ済みモデルのリストを取得するには、以下の要求を使用します。

   ```
   curl -X GET -i -H "Content-Type:*/*" https://ibm-watson-ml.mybluemix.net/pm/v1/model?accesskey={accesskey_value}
   ```
   {: codeblock}

   応答:

   ```
   [{"stream":"customer-satisfaction-prediction.str","id":"context_csp2"}]
```
   {: codeblock}

5. 特定のデプロイ済みモデルのコピーをダウンロードするには、以下の要求を使用します。

   ```
   curl -X GET -v -H "Content-Type:*/*" https://ibm-watson-ml.mybluemix.net/pm/v1/model/{context_id}?accesskey={accesskey_value} >> output.str
   ```
   {: codeblock}

   応答 (これにより、モデルのコンテンツが output.str ファイルにダウンロードされます)。

   ```
   > GET /pm/v1/model/context_csp2?accesskey=pcB4lKG1brpgSCEonNoBdzew9kiOOzN8awh1cJ2sHAxf1yyjh50nnrQngWw4DD/tM13eGXGHaJ0voQU+cAi1t/nmJBaSgi+xeMY8Wia68PB227SsqjgA5nvrX+eU9Sbr HTTP/1.1

   > Host: ibm-watson-ml-dev.stage1.mybluemix.net

   > User-Agent: curl/7.50.3

   > Accept: */*

   > Content-Type:*/*

   >

   * STATE: DO => DO_DONE handle 0x600057960; line 1664 (connection #0)

   * STATE: DO_DONE => WAITPERFORM handle 0x600057960; line 1791 (connection #0)

   * STATE: WAITPERFORM => PERFORM handle 0x600057960; line 1801 (connection #0)

   { [5 bytes data]

   * HTTP 1.1 or later with persistent connection, pipelining supported

   < HTTP/1.1 200 OK

   < X-Backside-Transport: OK OK

   < Connection: Keep-Alive

   < Transfer-Encoding: chunked

   < Content-Disposition: attachment; filename=customer-satisfaction-prediction.str

   < Content-Type: application/octet-stream

   < Date: Fri, 17 Feb 2017 15:55:20 GMT

   * Server nginx/1.11.5 is not blacklisted

   < Server: nginx/1.11.5

   < Set-Cookie: NSC_qnefw-bggjojuz=ffffffff0987c39d45525d5f4f58455e445a4a421548;expires=Fri, 17-Feb-2017 15:57:20 GMT;path=/;httponly

   < X-Powered-By: Servlet/3.0

   < X-Global-Transaction-ID: 3450392319

   <
   
   0 0 0 0 0 0 0 0 --:--:-- 0:00:01 --:--:-- 0{ [5 bytes data]

   100 74410 0 74410 0 0 15966 0 --:--:-- 0:00:04 --:--:-- 15978* STATE: PERFORM => DONE handle 0x600057960; line 1965 (connection #0)

   * multi_done

   * Curl_http_done: called premature == 0

   100 99k 0 99k 0 0 21114 0 --:--:-- 0:00:04 --:--:-- 24508

   * Connection #0 to host ibm-watson-ml-dev.stage1.mybluemix.net left intact
```
   {: codeblock}

6. デプロイ済みモデルを削除します。

   ```
   curl -X DELETE -v -H "Content-Type:*/*" https://ibm-watson-ml.mybluemix.net/pm/v1/model/{context_id}?accesskey={accesskey_value}
   ```
   {: codeblock}

   応答:

   ```
   > DELETE /pm/v1/model/context_csp2?accesskey=pcB4lKG1brpgSCEonNoBdzew9kiOOzN8awh1cJ2sHAxf1yyjh50nnrQngWw4DD/tM13eGXGHaJ0voQU+cAi1t/nmJBaSgi+xeMY8Wia68PB227SsqjgA5nvrX+eU9Sbr HTTP/1.1

   > Host: ibm-watson-ml-dev.stage1.mybluemix.net

   > User-Agent: curl/7.50.3

   > Accept: */*

   > Content-Type:*/*

   >

   < HTTP/1.1 200 OK

   < X-Backside-Transport: OK OK

   < Connection: Keep-Alive

   < Transfer-Encoding: chunked

   < Content-Type: application/json

   < Date: Fri, 17 Feb 2017 16:21:20 GMT

   * Server nginx/1.11.5 is not blacklisted

   < Server: nginx/1.11.5

   < Set-Cookie: NSC_qnefw-bggjojuz=ffffffff0987c39d45525d5f4f58455e445a4a421548;expires=Fri, 17-Feb-2017 16:23:20 GMT;path=/;httponly

   < X-Powered-By: Servlet/3.0

   < X-Global-Transaction-ID: 2495867153

   <
   
   * STATE: PERFORM => DONE handle 0x600057960; line 1965 (connection #0)

   * multi_done

   * Curl_http_done: called premature == 0

   * Connection #0 to host ibm-watson-ml-dev.stage1.mybluemix.net left intact

   {"flag":true,"message":"success to delete stream with specified id context_csp2"}

   Response when trying to delete a model that doesn't exist (or
   was already deleted):

   > DELETE /pm/v1/model/context_csp2?accesskey=pcB4lKG1brpgSCEonNoBdzew9kiOOzN8awh1cJ2sHAxf1yyjh50nnrQngWw4DD/tM13eGXGHaJ0voQU+cAi1t/nmJBaSgi+xeMY8Wia68PB227SsqjgA5nvrX+eU9Sbr HTTP/1.1

   > Host: ibm-watson-ml-dev.stage1.mybluemix.net

   > User-Agent: curl/7.50.3

   > Accept: */*

   > Content-Type:*/*

   >

   * STATE: DO => DO_DONE handle 0x600057960; line 1664 (connection #0)

   * STATE: DO_DONE => WAITPERFORM handle 0x600057960; line 1791 (connection #0)

   * STATE: WAITPERFORM => PERFORM handle 0x600057960; line 1801 (connection #0)

   * HTTP 1.1 or later with persistent connection, pipelining supported

   < HTTP/1.1 404 Not Found

   < X-Backside-Transport: FAIL FAIL

   < Connection: Keep-Alive

   < Transfer-Encoding: chunked

   < Content-Type: application/json

   < Date: Fri, 17 Feb 2017 16:23:15 GMT

   * Server nginx/1.11.5 is not blacklisted

   < Server: nginx/1.11.5

   < Set-Cookie: NSC_qnefw-bggjojuz=ffffffff0987c39d45525d5f4f58455e445a4a421548;expires=Fri, 17-Feb-2017 16:25:15 GMT;path=/;httponly

   < X-Powered-By: Servlet/3.0

   < X-Global-Transaction-ID: 94445277

   <
   
   * STATE: PERFORM => DONE handle 0x600057960; line 1965 (connection #0)

   * multi_done

   * Curl_http_done: called premature == 0

   * Connection #0 to host ibm-watson-ml-dev.stage1.mybluemix.net left intact

   Not Found Model:context_csp2
```
   {: codeblock}

7. デプロイ済みモデルをスコアリングします。

   ```
   curl -X POST -v -H "Content-Type:application/json;Charset=UTF-8" -d '{"tablename":"Input data","header":["customerID","gender","SeniorCitizen","Partner","Dependents","tenure","PhoneService","MultipleLines","InternetService","OnlineSecurity","OnlineBackup","DeviceProtection","TechSupport","StreamingTV","StreamingMovies","Contract","PaperlessBilling","PaymentMethod","MonthlyCharges","TotalCharges","Churn","SampleWeight"],"data":[["9237-HQITU","Female",0,"No","No",2,"Yes","No","Fiber optic","No","No","No","No","No","No","Month-to-month","Yes","Electronic check",70.700,151.650,"Yes",1.000]]}' https://ibm-watson-ml.mybluemix.net/pm/v1/score/{context_id}?accesskey={accesskey_value}
   ```
   {: codeblock}

   応答:

   ```
   > POST /pm/v1/score/context_csp2?accesskey=pcB4lKG1brpgSCEonNoBdzew9kiOOzN8awh1cJ2sHAxf1yyjh50nnrQngWw4DD/tM13eGXGHaJ0voQU+cAi1t/nmJBaSgi+xeMY8Wia68PB227SsqjgA5nvrX+eU9Sbr HTTP/1.1

   > Host: ibm-watson-ml-dev.stage1.mybluemix.net

   > User-Agent: curl/7.50.3

   > Accept: */*

   > Content-Type:application/json;charset=UTF-8

   > Content-Length: 525

   >

   < HTTP/1.1 200 OK

   < X-Backside-Transport: OK OK

   < Connection: Keep-Alive

   < Transfer-Encoding: chunked

   < Content-Type: application/json;charset=UTF-8

   < Date: Fri, 17 Feb 2017 16:40:05 GMT

   * Server nginx/1.11.5 is not blacklisted

   < Server: nginx/1.11.5

   < Set-Cookie: NSC_qnefw-bggjojuz=ffffffff0987c39d45525d5f4f58455e445a4a421548;expires=Fri, 17-Feb-2017 16:42:05 GMT;path=/;httponly

   < X-Powered-By: Servlet/3.0

   < X-Global-Transaction-ID: 3291552207



   [{"header":["customerID","Churn","Predicted Churn","Probability of Churn"],"data":[["9237-HQITU","Yes","Yes",0.8829830706957551]]}] ```
   {: codeblock}

## 詳細はこちら
 
さあ始めましょう。サービス・インスタンスの作成またはアプリケーションのバインドについては、『[Spark モデルおよび Python モデルを用いたサービスの使用](using_pm_service_dsx.html)』または『[IBM® SPSS® モデルを用いたサービスの使用](using_pm_service.html)』を参照してください。

API について詳しくは、[Spark モデルおよび Python モデル用のサービス API](pm_service_api_spark.html) または [IBM® SPSS® モデル用のサービス API] (pm_service_api_spss.html) を参照してください。

IBM® SPSS® Modeler の概要と提供されるモデリング・アルゴリズムについて詳しくは、[IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/SS3RA7) を参照してください。

IBM Data Science Experience の概要と提供されるモデリング・アルゴリズムについて詳しくは、[https://datascience.ibm.com](https://datascience.ibm.com) を参照してください。
