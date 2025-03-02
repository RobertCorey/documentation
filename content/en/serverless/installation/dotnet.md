---
title: Instrumenting .NET Serverless Applications
kind: documentation
further_reading:
- link: 'serverless/serverless_tagging/'
  tag: "Serverless"
  text: 'Tagging Serverless Applications'
- link: 'serverless/distributed_tracing/'
  tag: "Serverless"
  text: 'Tracing Serverless Applications'
- link: 'serverless/custom_metrics/'
  tag: "Serverless"
  text: 'Submitting Custom Metrics from Serverless Applications'
---

## Prerequisites

To instrument your .NET serverless application, you must install **either** the [Datadog Forwarder Lambda function][1] or (beta) the [Datadog Lambda Extension][2].

## Configuration

{{< tabs >}}
{{% tab "Extension" %}}

<div class="alert alert-warning">
Instrumenting .NET applications with the Datadog Lambda Extension is in beta.
</div>

Add the [Datadog Lambda Extension][1] layer for your Lambda function using the appropriate ARN for your desired processor:

**x86**: `arn:aws:lambda:<AWS_REGION>:464622532012:layer:Datadog-Extension:<EXTENSION_VERSION>`

**ARM**: `arn:aws:lambda:<AWS_REGION>:464622532012:layer:Datadog-Extension-ARM:<EXTENSION_VERSION>`

Set the environment variable `DD_API_KEY` to your [Datadog API key][2].


[1]: /serverless/libraries_integrations/extension/
[2]: https://app.datadoghq.com/organization-settings/api-keys
{{% /tab %}}
{{% tab "Forwarder" %}}

### Install

1. Enable [AWS X-Ray active tracing][1] for your Lambda function.
2. Install the [AWS X-Ray SDK for .NET][2].

### Subscribe

Subscribe the Datadog Forwarder Lambda function to each of your function's log groups to send metrics, traces, and logs to Datadog.

1. [Install the Datadog Forwarder if you haven't][3].
2. [Subscribe the Datadog Forwarder to your function's log groups][4].


[1]: https://docs.aws.amazon.com/xray/latest/devguide/xray-services-lambda.html
[2]: https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-dotnet.html
[3]: /serverless/forwarder/
[4]: /serverless/libraries_integrations/extension/
{{% /tab %}}
{{< /tabs >}}

### Tag

Although it's optional, Datadog recommends tagging your serverless applications with the `env`, `service`, and `version` tags following the [unified service tagging documentation][3].

### Collect logs from AWS serverless resources

Serverless logs generated by managed resources besides AWS Lambda functions can be hugely valuable in helping identify the root cause of issues in your serverless applications. Datadog recommends you forward logs from the following managed resources in your environment:
- APIs: API Gateway, AppSync, ALB.
- Queues & Streams: SQS, SNS, Kinesis.
- Data Stores: DynamoDB, S3, RDS, etc.

To collect logs from non-Lambda AWS resources, install and configure the [Datadog Forwarder][9] to subscribe to each of your managed resource CloudWatch log groups.

### Monitor custom business logic

If you would like to submit a custom metric, see the sample code below:

```csharp
var myMetric = new Dictionary<string, object>();
myMetric.Add("m", "coffee_house.order_value");
myMetric.Add("v", 12.45);
myMetric.Add("e", (int)(DateTime.UtcNow - new DateTime(1970, 1, 1)).TotalSeconds);
myMetric.Add("t", new string[] {"product:latte", "order:online"});
LambdaLogger.Log(JsonConvert.SerializeObject(myMetric));
```

For more information on custom metric submission, see the [Serverless Custom Metrics documentation][3].

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: /serverless/forwarder/
[2]: /serverless/libraries_integrations/extension/
[3]: /serverless/custom_metrics?tab=otherruntimes
