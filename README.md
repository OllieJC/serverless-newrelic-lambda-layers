# serverless-newrelic-lambda-layers

A [Serverless](https://serverless.com) plugin to add [New Relic](https://www.newrelic.com)
observability using [AWS Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) without requiring a code change.

## Requirements

- [serverless](https://github.com/serverless/serverless) >= 2.21.1

## Features

- Installs and configures the New Relic AWS Integration
- Supports Node.js, Python and Java runtimes (more runtimes to come)
- No code change required to enable New Relic
- Bundles New Relic's agent in a single layer
- Configures CloudWatch subscription filters automatically

## Install

With NPM:

```bash
npm install --save-dev serverless-newrelic-lambda-layers
```

With yarn:

```bash
yarn add --dev serverless-newrelic-lambda-layers
```

Add the plugin to your `serverless.yml`:

```yaml
plugins:
  - serverless-newrelic-lambda-layers
```

If you don't yet have a New Relic account, [sign up here](https://newrelic.com/products/serverless-aws-lambda).

Grab your [New Relic Account ID](https://docs.newrelic.com/docs/accounts/install-new-relic/account-setup/account-id),
your [New Relic Personal API Key](https://docs.newrelic.com/docs/apis/get-started/intro-apis/types-new-relic-api-keys#personal-api-key)
and plug them into your `serverless.yml`:

```yaml
custom:
  newRelic:
    accountId: your-new-relic-account-id-here
    apiKey: your-new-relic-personal-api-key-here
```

Deploy:

```bash
sls deploy
```

And you're all set.

## Usage

This plugin wraps your handlers without requiring a code change. If you're currently
using a New Relic agent, you can remove the wrapping code you currently have and this plugin will
do it for you automatically.

- [Node.js Instrumentation Guide](https://docs.newrelic.com/docs/agents/nodejs-agent/getting-started/introduction-new-relic-nodejs#extend-instrumentation)
- [Python Instrumentation Guide](https://docs.newrelic.com/docs/agents/python-agent/custom-instrumentation/python-custom-instrumentation)
- [Java Instrumentation Guide](https://docs.newrelic.com/docs/agents/java-agent/custom-instrumentation/java-custom-instrumentation)

## Config

The following config options are available via the `newRelic` section of the `custom` section of your `serverless.yml`:

#### `accountId` (required)

Your [New Relic Account ID](https://docs.newrelic.com/docs/accounts/install-new-relic/account-setup/account-id).

```yaml
custom:
  newRelic:
    accountId: your-account-id-here
```

#### `apiKey` (required)

Your [New Relic Personal API Key](https://docs.newrelic.com/docs/apis/get-started/intro-apis/types-new-relic-api-keys#personal-api-key).

```yaml
custom:
  newRelic:
    apiKey: your-api-key-here
```
If your function's source is committed to version control, you can avoid committing your license key by including it in your serverless.yml as a variable. See [the Serverless docs on template variables](https://www.serverless.com/framework/docs/providers/aws/guide/variables/) for more information.  

#### `nrRegion` (required for EU; optional for US)

If your New Relic account is based in the EU, make sure to specify your nrRegion in the custom block:

```yaml
custom:
  newRelic:
    nrRegion: 'eu'
```

#### `linkedAccount` (optional)

A label for the New Relic Linked Account. This is how this integration will
appear in New Relic. If not set, it will default to "New Relic Lambda Integration - 
<AWS ACcount ID>".

```yaml
custom:
  newRelic:
    linkedAccount: your-linked-account-name
```

#### `trustedAccountKey` (optional)

Only required if your New Relic account is a sub-account. This needs to be the account ID for the root/parent account.

```yaml
custom:
  newRelic:
    trustedAccountKey: your-parent-account-id
```

#### `debug` (optional)

Whether or not to enable debug mode. Must be a boolean value. This sets the log level to
debug.

```yaml
custom:
  newRelic:
    debug: true
```

#### `logEnabled` (optional)

Enables logging. Defaults to `false`

#### `enableExtension` (optional)

Allows your function to deliver its telemetry to New Relic via AWS Lambda Extension. Defaults to `true`, so it can be omitted. To avoid delivering your telemetry via the extension, set to `false`.

```yaml
custom:
  newRelic:
    enableExtension: true
```
#### `enableFunctionLogs` (optional)

Allows your function to deliver all of your function logs to New Relic via AWS Lambda Extension. This would eliminate the need for a CloudWatch log subscription + the NR log ingestion Lambda function. This method of log ingestion is lower-cost, and offers faster time to glass.  

```yaml
custom:
  newRelic:
    enableFunctionLogs: true
```

#### `enableIntegration` (optional)

Allows the creation of New Relic aws cloud integration when absent. Defaults to `false`

```yaml
custom:
  newRelic:
    enableIntegration: true
```

#### `logLevel` (optional)

Sets a log level on all functions. Possible values: `'fatal'`, `'error'`, `'warn'`, `'info'`, `'debug'`, `'trace'` or `'silent'`. Defaults to `'error'`

You can still override log level on a per function basis by configuring environment variable `NEW_RELIC_LOG_LEVEL`.

```yaml
custom:
  newRelic:
    logLevel: debug
```

Logging configuration is considered in the following order:

1. function `NEW_RELIC_LOG_LEVEL` environment
2. provider `NEW_RELIC_LOG_LEVEL` environment
3. custom newRelic `logLevel` property
4. custom newRelic `debug` flag

#### `customRolePolicy` (optional)

Specify an alternative IAM role policy ARN for this integration here if you do not want to use the default role policy.

```yaml
custom:
  newRelic:
    customRolePolicy: your-custom-role-policy-arn
```

#### `stages` (optional)

An array of stages that the plugin will be included for. If this key is not specified then all stages will be included.

```yaml
custom:
  stages:
      - prod
```

#### `include` (optional)

An array of functions to include for automatic wrapping. (You can set `include` or `exclude` options, but not both.)

```yaml
custom:
  newRelic:
    include:
      - include-only-func
      - another-included-func
```

#### `exclude` (optional)

An array of functions to exclude from automatic wrapping. (You can set `include` or `exclude` options, but not both.)

```yaml
custom:
  newRelic:
    exclude:
      - excluded-func-1
      - another-excluded-func
```

#### `layerArn` (optional)

Pin to a specific layer version. The latest layer ARN is automatically fetched from the [New Relic Layers API](https://layers.newrelic-external.com)

```yaml
custom:
  newRelic:
    layerArn: arn:aws:lambda:us-east-1:451483290750:layer:NewRelicPython37:2
```

#### `cloudWatchFilter` (optional)

Provide a list of quoted filter terms for the CloudWatch log subscription to the newrelic-log-ingestion Lambda. Combines all terms into an OR filter. Defaults to "NR_LAMBDA_MONITORING" if not set. Use "\*" to capture all logs

```yaml
custom:
  newRelic:
    cloudWatchFilter:
      - "NR_LAMBDA_MONITORING"
      - "trace this"
      - "ERROR"
```

If you want to collect all logs:

```yaml
custom:
  newRelic:
    cloudWatchFilter: "*"
```

Be sure to set the `LOGGING_ENABLED` environment variable to `true` in your log
ingestion function. See the [aws-log-ingestion documentation](https://github.com/newrelic/aws-log-ingestion) for details.

#### `prepend` (optional)

Whether or not to prepend the New Relic layer. Defaults to `false` which appends the layer.

```yaml
custom:
  newRelic:
    prepend: true
```

#### `logIngestionFunctionName` (optional)

Only required if your New Relic log ingestion function name is different from `newrelic-log-ingestion`.

```yaml
custom:
  newRelic:
    logIngestionFunctionName: log-ingestion-service
```

#### `disableAutoSubscription` (optional)

Only required if you want to disable auto subscription.

```yaml
custom:
  newRelic:
    disableAutoSubscription: true
```

#### `disableLicenseKeySecret` (optional)

Only required if you want to disable creating license key in AWS Secrets Manager. Setting this as `true` would create NEW_RELIC_LICENSE_KEY environment variable for the New Relic Lambda Extension to access.

```yaml
custom:
  newRelic:
    disableLicenseKeySecret: true
```

#### `javaNewRelicHandler` (optional)

**Java runtimes only**. Only required if you are implementing the `RequestStreamHandler` interface.
Defaults to `RequestHandler` interface.
#### Accepted inputs:
- handleRequest
- handleStreamsRequest

```yaml
custom:
  newRelic:
    javaNewRelicHandler: handleStreamsRequest
```

#### `proxy` (optional)

This plugin makes various HTTP requests to public APIs in order to retrieve data about the New Relic and cloud provider accounts. If you are behind a proxy when this plugin runs, the HTTP agent needs the proxy information to connect to those APIs. Use the given URL as a proxy for HTTP requests.

```yaml
custom:
  newRelic:
    proxy: http://yourproxy.com:8080
```

## Supported Runtimes

This plugin currently supports the following AWS runtimes:

- nodejs10.x
- nodejs12.x
- nodejs14.x
- python2.7
- python3.6
- python3.7
- python3.8
- python3.9
- java11
- java8.al2

## Contributing

### Testing

1. Make changes to `examples/nodejs/serverless.yml` based on what you are planning to test
2. Generate a test case by executing script `generate:test:case`

```shell
# Example
npm run generate:test:case
```

3. Rename generated file `tests/fixtures/example.service.input.json` to test case e.g. `tests/fixtures/log-level.service.input.json`
4. Create expected output file `tests/fixtures/example.service.output.json` for test case e.g. `tests/fixtures/log-level.service.output.json`
5. Run tests

```shell
# Example
npm run test
```
