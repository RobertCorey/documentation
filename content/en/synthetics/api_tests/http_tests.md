---
title: HTTP Tests
kind: documentation
description: Simulate HTTP requests to monitor public and internal API endpoints
aliases:
  - /synthetics/http_test
  - /synthetics/http_check
further_reading:
- link: "https://www.datadoghq.com/blog/introducing-synthetic-monitoring/"
  tag: "Blog"
  text: "Introducing Datadog Synthetic Monitoring"
- link: 'https://learn.datadoghq.com/course/view.php?id=39'
  tag: 'Learning Center'
  text: 'Introduction to Synthetic Tests'
- link: "/getting_started/synthetics/api_test"
  tag: "Documentation"
  text: "Get started with HTTP tests"
- link: "/synthetics/private_locations"
  tag: "Documentation"
  text: "Run HTTP tests on internal endpoints"
---
## Overview

HTTP tests allow you to send HTTP requests to your applications' API endpoints to verify responses and defined conditions, such as overall response time, expected status code, header, or body content.

HTTP tests can run from both [managed][1] and [private locations][2] depending on your preference for running the test from outside or inside your network. HTTP tests can run on a schedule, on-demand, or directly within your [CI/CD pipelines][3].

## Configuration

After choosing to create an `HTTP` test, define your test's request.

### Define request

1. Choose the **HTTP Method** and specify the **URL** to query. Available methods are: `GET`, `POST`, `PATCH`, `PUT`, `HEAD`, `DELETE`, and `OPTIONS`. Both `http` and `https` URLs are supported.
2. Enrich your HTTP request with **Advanced Options** (optional):

   {{< tabs >}}

   {{% tab "Request Options" %}}

   * **Follow redirects**: Select to have your HTTP test follow up to ten redirects when performing the request.
   * **Timeout**: Specify the amount of time in seconds before the test times out.
   * **Request headers**: Define headers to add to your HTTP request. You can also override the default headers (for example, the `user-agent` header).
   * **Cookies**: Define cookies to add to your HTTP request. Set multiple cookies using the format `<COOKIE_NAME1>=<COOKIE_VALUE1>; <COOKIE_NAME2>=<COOKIE_VALUE2>`.

   {{% /tab %}}

   {{% tab "Authentication" %}}

   * **HTTP Basic Auth**: Add HTTP basic authentication credentials.
   * **AWS Signature V4**: Enter your Access Key ID and Secret Access Key. Datadog generates the signature for your request.
   This option uses the basic implementation of SigV4. Specific signatures such as AWS S3 are not implemented.
   * **NTLM v1**: Add NTLM authentication credentials.

   {{% /tab %}}

   {{% tab "Request Body" %}}

   * **Body type**: Select the type of the request body (`text/plain`, `application/json`, `text/xml`, `text/html`, `application/x-www-form-urlencoded`, or `None`) you want to add to your HTTP request.
   * **Request body**: Add the content of your HTTP request body. **Note**: The request body is limited to a maximum size of 50 kilobytes.

   {{% /tab %}}

   {{% tab "Certificate" %}}

   * **Ignore server certificate error**: Select to have your HTTP test go on with connection even if there are errors when validating the SSL certificate.
   * **Client certificate**: Authenticate through mTLS by uploading your client certificate (`.crt`) and the associated private key (`.key`) in `PEM` format. You can use the `openssl` library to convert your certificates. For example you can convert a `PKCS12` certificate to `PEM` formatted private keys and certificates.

     ```
     openssl pkcs12 -in <CERT>.p12 -out <CERT_KEY>.key -nodes -nocerts
     openssl pkcs12 -in <CERT>.p12 -out <CERT>.cert -nokeys
     ```

   {{% /tab %}}

   {{% tab "Proxy" %}}

   * **Proxy URL**: Specify the URL of the proxy the HTTP request should go through (`http://<YOUR_USER>:<YOUR_PWD>@<YOUR_IP>:<YOUR_PORT>`).
   * **Proxy Header**: Add headers to include in the HTTP request to the proxy.

   {{% /tab %}}

   {{% tab "Privacy" %}}

   * **Do not save response body**: Select this option to prevent response body from being saved at runtime. This can be helpful to ensure no sensitive data gets featured in your test results. Use mindfully as it can make failures troubleshooting more difficult. For more security recommendations, see [Synthetic Monitoring Security][1].


[1]: /security/synthetics
   {{% /tab %}}

   {{< /tabs >}}

<br/>

3. **Name** your HTTP test.

4. Add `env` **Tags** as well as any other tag to your HTTP test. You can then use these tags to quickly filter through your Synthetic tests on the [Synthetic Monitoring homepage][4].

{{< img src="synthetics/api_tests/http_test_config.png" alt="Define HTTP request" style="width:90%;" >}}

Click **Test URL** to try out the request configuration. A response preview is displayed on the right side of your screen.

### Define assertions

Assertions define what an expected test result is. When hitting `Test URL` basic assertions on `response time`, `status code`, and `header` `content-type` are added based on the response that was obtained. You must define at least one assertion for your test to monitor.

| Type          | Operator                                                                                               | Value type                                                      |
|---------------|--------------------------------------------------------------------------------------------------------|----------------------------------------------------------------|
| body          | `contains`, `does not contain`, `is`, `is not`, <br> `matches`, `does not match`, <br> [`jsonpath`][5], [`xpath`][6] | _String_ <br> _[Regex][7]_ <br> _String_, _[Regex][7]_ |
| header        | `contains`, `does not contain`, `is`, `is not`, <br> `matches`, `does not match`                       | _String_ <br> _[Regex][7]_                                      |
| response time | `is less than`                                                                                         | _Integer (ms)_                                                  |
| status code   | `is`, `is not`                                                                                         | _Integer_                                                      |

**Note**: HTTP tests can decompress bodies with the following `content-encoding` headers: `br`, `deflate`, `gzip`, and `identity`.

You can create up to 20 assertions per API test by clicking on **New Assertion** or by clicking directly on the response preview:

{{< img src="synthetics/api_tests/assertions.png" alt="Define assertions for your HTTP test" style="width:90%;" >}}

### Select locations

Select the **Locations** to run your HTTP test from. HTTP tests can run from both [managed][1] and [private locations][2] depending on your preference for running the test from outside or inside your network.

### Specify test frequency

HTTP tests can run:

* **On a schedule** to ensure your most important endpoints are always accessible to your users. Select the frequency at which you want Datadog to run your HTTP test.
* [**Within your CI/CD pipelines**][3] to start shipping without fearing faulty code might impact your customers experience.
* **On-demand** to run your tests whenever makes the most sense for your team.

### Define alert conditions

Set alert conditions to determine the circumstances under which you want a test to fail and trigger an alert.

#### Alerting rule

When you set the alert conditions to: `An alert is triggered if any assertion fails for X minutes from any n of N locations`, an alert is triggered only if these two conditions are true:

* At least one location was in failure (at least one assertion failed) during the last *X* minutes;
* At one moment during the last *X* minutes, at least *n* locations were in failure.

#### Fast retry

Your test can trigger retries `X` times after `Y` ms in case of a failed test result. Customize the retry interval to suit your alerting sensibility.

Location uptime is computed on a per-evaluation basis (whether the last test result before evaluation was up or down). The total uptime is computed based on the configured alert conditions. Notifications sent are based on the total uptime.

### Notify your team

A notification is sent by your test based on the [alerting conditions](#define-alert-conditions) previously defined. Use this section to define how and what message to send to your teams.

1. [Similar to how you configure monitors][8], select **users and/or services** that should receive notifications either by adding a `@notification`to the message or by searching for team members and connected integrations with the drop-down box.

2. Enter the notification **message** for your test. This field allows standard [Markdown formatting][9] and supports the following [conditional variables][10]:

    | Conditional Variable       | Description                                                         |
    |----------------------------|---------------------------------------------------------------------|
    | `{{#is_alert}}`            | Show when the test alerts.                                          |
    | `{{^is_alert}}`            | Show unless the test alerts.                                        |
    | `{{#is_recovery}}`         | Show when the test recovers from alert.                             |
    | `{{^is_recovery}}`         | Show unless the test recovers from alert.                           |

3. Specify how often you want your test to **re-send the notification message** in case of test failure. To prevent renotification on failing tests, leave the option as `Never renotify if the monitor has not been resolved`.

Click **Save** to save and start your test.

## Variables

### Create local variables

You can create local variables by clicking on **Create Local Variable** at the top right hand corner of your test configuration form. You can define their values from one of the below available builtins:

`{{ numeric(n) }}`
: Generates a numeric string with `n` digits.

`{{ alphabetic(n) }}`
: Generates an alphabetic string with `n` letters.

`{{ alphanumeric(n) }}`
: Generates an alphanumeric string with `n` characters.

`{{ date(n, format) }}`
: Generates a date in one of our accepted formats with a value of the date the test is initiated + `n` days.

`{{ timestamp(n, unit) }}` 
: Generates a timestamp in one of our accepted units with a value of the timestamp the test is initiated at +/- `n` chosen unit.

### Use variables

You can use the [global variables defined in the `Settings`][11] and the [locally defined variables](#create-local-variables) in the URL, advanced options, and assertions of your HTTP tests.

To display your list of variables, type `{{` in your desired field:

{{< img src="synthetics/api_tests/use_variable.mp4" alt="Using Variables in API tests" video="true" width="90%" >}}

## Test failure

A test is considered `FAILED` if it does not satisfy one or more assertions or if the request prematurely failed. In some cases, the test can fail without testing the assertions against the endpoint. 

These reasons include the following:

`CONNREFUSED`
: No connection could be made because the target machine actively refused it.

`CONNRESET`
: The connection was abruptly closed by the remote server. Possible causes include the web server encountering an error or crashing while responding, or loss of connectivity of the web server.

`DNS`
: DNS entry not found for the test URL. Possible causes include misconfigured test URL or the wrong configuration of your DNS entries.

`INVALID_REQUEST` 
: The configuration of the test is invalid (for example, a typo in the URL).

`SSL`
: The SSL connection couldn't be performed. [See the dedicated error page for more information][12].

`TIMEOUT`
: The request couldn't be completed in a reasonable time. Two types of `TIMEOUT` can happen:
  - `TIMEOUT: The request couldn’t be completed in a reasonable time.` indicates that the request duration hit the test defined timeout (default is set to 60s). 
  For each request only the completed stages for the request are displayed in the network waterfall. For example, in the case of `Total response time` only being displayed, the timeout occurred during the DNS resolution.
  - `TIMEOUT: Overall test execution couldn't be completed in a reasonable time.` indicates that the test duration (request + assertions) hits the maximum duration (60.5s).

`MALFORMED_RESPONSE` 
: The remote server responded with a payload that does not comply with HTTP specifications.

## Permissions

By default, only users with the [Datadog Admin and Datadog Standard roles][13] can create, edit, and delete Synthetic HTTP tests. To get create, edit, and delete access to Synthetic HTTP tests, upgrade your user to one of those two [default roles][13].

If you have access to the [custom role feature][14], add your user to any custom role that includes `synthetics_read` and `synthetics_write` permissions.

### Restrict access

<div class="alert alert-warning">
Access restriction is available for customers with <a href="https://docs.datadoghq.com/account_management/rbac/?tab=datadogapplication#create-a-custom-role">custom roles</a> enabled on their accounts.</div>

You can restrict access to an HTTP test based on the roles in your organization. When creating an HTTP test, choose which roles (in addition to your user) can read and write your test. 

{{< img src="synthetics/settings/restrict_access.png" alt="Set permissions for your test" style="width:70%;" >}}

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: /api/v1/synthetics/#get-all-locations-public-and-private
[2]: /synthetics/private_locations
[3]: /synthetics/cicd_integrations
[4]: /synthetics/search/#search
[5]: https://restfulapi.net/json-jsonpath/
[6]: https://www.w3schools.com/xml/xpath_syntax.asp
[7]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions
[8]: /monitors/notify/#notify-your-team
[9]: https://www.markdownguide.org/basic-syntax/
[10]: /monitors/notify/?tab=is_recoveryis_alert_recovery#conditional-variables
[11]: /synthetics/settings/#global-variables
[12]: /synthetics/api_tests/errors/#ssl-errors
[13]: /account_management/rbac/
[14]: /account_management/rbac#custom-roles
