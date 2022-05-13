# Overview

Integrating Dynatrace Cloud Automation with PagerDuty can enable you to mobilize responders whenever important problems occur in Dynatrace, while coordinating other actions in Cloud Automation.

This translator Lambda Function will receive PagerDuty events from the PagerDuty event rule set and translate and make the Cloud Automation API call to send back a Cloud Automation sequence task finished event.

# Install

1 . Create a Lambda in AWS

  * Log in to AWS Console and navigate to AWS Lambda. 
  * Click on the Create Function button in the top right. 
  * Choose Author from scratch, give the function a name like pd-to-cloudautomation, and choose the runtime Node.js 14.x. 
  * Click on the Create Function button at the bottom. 

2 . After your function has been created:

  * click on the Add Trigger button at the top. 
  * Choose API Gateway from the drop-down list, then choose Create API, then HTTP API, then Security > Open. 
  * Click on the Add button to add the trigger. 

3 . Once the trigger has been created, you will use this URL to configure the Event Rule in PagerDuty. The URL should look something like this `https://abc.execute-api.us-west-1.amazonaws.com/default/pd-to-cloudautomation`

4 . Upload the Translator Code

  * Download the pre-built Lambda code ZIP from [this link](https://github.com/martindstone/pd-to-cloudautomation-lambda/releases/) and save it to your computer.  
  * Now click on the Code tab in your Lambda, and click on Upload from > .zip file on the right. 
  * Click Upload, choose the zip file you downloaded, and click Save.

5 . Configure the Translator

In your Lambda, click on `Configuration > Environment variables > Edit`, and add the following 3 variables:

| Name | Value |
| ---- | ----- |
| KEPTN_BASE_URL | _Base URL to your Cloud Automation environment. For example: https://abcd.cloudautomation.live.dynatrace.com_ |
| KEPTN_API_TOKEN | _Cloud Automation API token_ |
| PD_API_ACCESS_KEY | _PagerDuty API Access Key_ |

Click on Save to save your environment variables.

# Development

- Clone this repo - `git clone https://github.com/martindstone/pd-to-cloudautomation-lambda.git`
- Change directory to the root of this project - `cd pd-to-cloudautomation-lambda`
- Install the dependencies - `npm install`
- zip up the contents of this directory (example on Mac: `zip -r myzip.zip *`)

# Testing the Lambda function

You can directly call the Lambda function as shown below. Just update the AWS and PagerDuty incident IDs from your environment.

```
export PD_URL=https://abc.execute-api.us-west-2.amazonaws.com/default/pd-to-cloudautomation

curl -X POST "$PD_URL" \
    -H "Content-Type: application/json" \
    --data-raw '
{
  "__pd_metadata": {
    "incident": {
      "id": "Q2N8IXDVXUUWC1"
    }
  }
}
'
```

# Testing the PageDuty event

You can directly call the PagerDuty event webhook with some hardcoded test values. Just update the PagerDuty URL and Dynatrace details from your environment.  Add debugging statements in the Lambda function as required.

```
PD_URL=https://events.pagerduty.com/x-ere/[YOUR ID such as KJD33KDK83LKNAER888]
ID=$(date +%s)

curl -X POST "$PD_URL" \
    -H "Content-Type: application/json" \
    --data-raw '
{
  "data": {
    "project": "incident-demo",
    "service": "casdemoapp",
    "stage": "production"
  },
  "shkeptncontext": "'$ID'",
  "id": "'$ID'",
  "type": "sh.keptn.event.openticket.triggered",
  "incident": {
        "PID": "5260361002895203404_1651766138342V2",
        "ProblemURL": "https://[YOUR-TENANT].live.dyntrace.com",
        "BridgeURL": "https://[YOUR-CA-TENANT].cloudautomation.live.dynatrace.com/bridge/project/incident-demo/sequence/0fcbddcb-6820-44fe-ad46-d5aaa16260e3",
        "ProblemID": "'$ID'",
        "ProblemTitle": "Simulated problem from script '$ID'",
        "Tags": "keptn_service:casdemoapp, [Environment]DT_RELEASE_PRODUCT:incident-demo, [Environment]DT_RELEASE_BUILD_VERSION:1.0.0, keptn_stage:production, [Environment]DT_RELEASE_VERSION:1.0.0, keptn_project:incident-demo, [Environment]DT_RELEASE_STAGE:production",
        "State": "OPEN"
    }
}
'
```