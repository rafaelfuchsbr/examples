# Notifications

> :warning: Default channels in the code are temporarily set to `#sre-playground` to avoid sending test
messages to channels with broader audience.

Library to send notifications related to Jenkins jobs. There are different types of notifications, each one with its own
default messages for different scenarios.

The first notification will create a singleton for that type of message and subsequent messages will use this
singleton as the base to process the messages. So anything setup in the first message will be hold in this singleton,
except for `default messages` - they will be used unless a specific message is passed to each notification call.

Each type of notification and methods may have different and/or specific arguments based on the default message needs -

## Methods

:warning: **ATTENTION!!** :warning:

> Note that when a required argument is not passed, the default value will be used. So please pay attention to the default values so you don't end up sending the wrong notification to the wrong channel.

### setupSlackNotification

Create a SlackNotification object so generic **notifications** can be sent.
The arguments will be used to initialise the notification.
All arguments are optional, but `message`. The call will not fail, but a warning message will be logged and it will
return `null`.

#### Arguments

| name                  | type    | required | acceptable values                                 | default | examples                                   |
|-----------------------|---------|:--------:|---------------------------------------------------|---------|--------------------------------------------|
| message               | string  | :white_check_mark:      | any string message                                | depends on the type of notification, more detail below | ...                                        |
| channel               | string  | :x:      | any string that can identify a channel or DM      | `#sre-playground` | `#dev`, `#team-sre`                        |
| failOnError :warning: | boolean | :x:      | `true` / `false`                                  | `false` | `true`, `false`                            |
| timestamp             | string  | :x:      | timestamp from previous message                   | `null` | `response.ts`                              |
| color                 | string  | :x:      | `good`, `warning`, `danger` or any hex color code | depends on the notification, more detail below | `#439FE0`, `good`                          |
| replyBroadcast        | boolean | :x:      | `true` / `false`                                  | `false` | `true`, `false`                            |
| blocks                | map     | :x:      | map of blocks acceptable by Slack                 | `null` | [more details here](slack-plugin-docs)     |
| tokenCredentialId     | map     | :x:      | Slack tokens                                      | `slack-athena-bot-token` | Avoid using this, unless pretty necessary. |

> :warning: The `failOnError` argument will fail **the build** if it fails to send the message to Slack. Use it with care.

> Most (or all) arguments are optional as the usual scenario is to use predefined messages that will be sent to
  predefined channels. Anything different from this, you should use the parameters, specially `message` and `channel`.

### setupBuildNotification

Create a BuildNotification object so notifications related to builds can be sent.

#### Arguments

Same as `setupSlackNotification`, with the following differences.

| name | type | required | acceptable values | default | examples            |
|------|------|:--------:|-------------------|---------|---------------------|
| channel | string | :x: | any string that can identify a channel or DM | `#deployments` | `#dev`, `#team-sre` |

### setupDeploymentNotification

Create a DeploymentNotification object so notifications related to deployments can be sent.

#### Arguments

Same as `setupSlackNotification`, with the following differences.

| name | type | required | acceptable values | default | examples |
|------|------|:--------:|-------------------|---------|----------|
| channel | string | :x: | any string that can identify a channel or DM | `#deployments` | `#deployments-testing`, `#team-sre` |

### setupTerraformNotification

Create a TerraformNotification object so notifications related to Terraform can be sent.

#### Arguments

Same as `setupSlackNotification`, with the following differences.

| name | type | required | acceptable values | default | examples |
|------|------|:--------:|-------------------|---------|----------|
| channel | string | :x: | any string that can identify a channel or DM | `#terraform` | `#team-sre` |

## Generic notifications

Simple methods to send notifications. No default messages and the default channel is `#sre-playground` to avoid any noise in common channels.

- **`send`**: Send a message to a Slack channel. No color is used and there is a "warning" default message is case the message is empty.
  - Arguments:
    - message:
    - `color`: The color of "sidebar" of the notification.
    - `channel`: The channel to where the message will be posted.
    - `timestamp`: This parameter is used only to identify a specific message when we want to `update/edit` the message. No
      usage in other scenarios.
- **`start`**: Send a message with neutral (light blue) color indicator.
  - Arguments:
    - `color`: Optional. Default is `Formatter.NEUTRAL_COLOR`.
- **`success`**: Send a message with green color indicator.
  - Arguments:
    - `color`: Optional. Default is `Formatter.SUCCESS_COLOR`.
- **`failure`**: Send a message with red color indicator.
  - Arguments:
    - `color`: Optional. Default is `Formatter.FAILURE_COLOR`.
- **`reply`**: Reply to the previous message in form of a thread.
- **`updateMessage`**: Update a given message. The `channelId` and `timestamp` fields define which message to update. If they are not set, the last message will be updated.
  - Arguments:
    - `channelId`: Optional. `channelId` attribute from the `response` of the message you are updating.
    - `timestamp`: Optional. `ts` attribute from the `response` of the message you are updating.
- **`react`**: Add reaction (emoji) to the _last message_. It reacts to the `response` object, so if you want to react to a message different from the last one, you have to save the `response` related to that message.
  - Arguments:
    - `emoji`: Required. The argument is not actually named, so you just have to pass a string with the emoji code.

> The `message` and `color` arguments can always be defined in the arguments of all methods above (except for `react`, which is not sending any message).

## Common arguments for `start`, `success` and `failure` methods

These are common arguments for all types of notifications with predefined messages.
The predefined message will look like the following one, so these arguments will be used to build those messages.
They are all optional.

- **`changeId`**: It's actually the branch name. Default is `env.GIT_BRANCH`, the branch of the current build.
- **`commitLinkText`**: Text that will be used for the link to the commit. Default is the short commit id (first 7 characters).
- **`commit`**: The commit id that will be shown in the message. The default is `env.GIT_COMMIT`, the current build's commit id.
- **`logsLinkText`**: Text that will be shown as the link to the Jenkins build logs. The default is `Jenkins logs`.

> These arguments are not applicable to the methods `react`, `updateMessage`, `reply` and `send`.

## Common arguments for `reply`, `react` and `updateMessage`

These are common arguments for all types of notifications.
They are all optional.

- **`response`**: The response object related to the message you want to interact with. The default is to use the response
  from the last message sent, except for `reply`, which defaults to the first message sent.

## Build notifications

Methods to send messages related to generic builds.
There are predefined messages in each method, which can beoverwritten by passing arguments.

> :warning: There is no default channel for this type of notification. You have to specify it through `environment` named parameter, otherwise no notification will be sent.

- **`buildStart`**: Same as `start` but with predefined messages.
- **`buildSuccess`**: Same as `success` but with predefined messages.
- **`buildFailure`**: Same as `failure` but with predefined messages.
- **`buildReply`**: Same as `reply` but with predefined messages.
- **`buildUpdateMessage`**: Same as for generic messages.
- **`buildReact`**: Same as for generic messages.

> If `message` argument is passed, then the predefined message will be ignored.
> If `color` argument is passed, then the predefined color will be ignored.

## Deployment notifications

> **TODO**: add arguments like commit, deployer name/id, etc - improve predefined messages.

Methods to send messages related to deployments.
The default channel is `#deployments` for `production` environment and `#deployments-testing` for any
other environment. There are predefined messages in each method, which can be overwritten by passing
arguments.

- **`deploymentStart`**: Same as `start` but with predefined messages.
- **`deploymentSuccess`**: Same as `success` but with predefined messages.
- **`deploymentFailure`**: Same as `failure` but with predefined messages.
- **`deploymentReply`**: Same as `reply` but with predefined messages.
- **`deploymentUpdateMessage`**: Same as for generic messages.
- **`deploymentReact`**: Same as for generic messages.

> If `message` argument is passed, then the predefined message will be ignored.
> If `color` argument is passed, then the predefined color will be ignored.

## Terraform notifications

Methods to send messages related to Terraform steps.
The default channel is `#terraform`. There are predefined messages in each method, which can be overwritten by passing arguments.

- **`terraformStart`**: Same as `start` but with predefined messages.
  - Arguments:
    - `environment`: Required. String with list of environments. You can just pass whatever the job is using.
    - `tfStep`: Optional. It will show the string passed. It's recommended to stick with actual Terraform commands, like `init`, `plan`, `apply`, etc. The default value is `command`.
- **`terraformSuccess`**: Same as `success` but with predefined messages.
  - Arguments:
    - `tfStep`: Optional. It will show the string passed. It's recommended to stick with actual Terraform commands, like `init`, `plan`, `apply`, etc. The default value is `command`.
- **`terraformFailure`**: Same as `failure` but with predefined messages.
  - Arguments:
    - `tfStep`: Optional. It will show the string passed. It's recommended to stick with actual Terraform commands, like `init`, `plan`, `apply`, etc. The default value is `command`.
- **`terraformReply`**: Same as `reply` but with predefined messages.
- **`terraformUpdateMessage`**: Same as for generic messages.
- **`terraformReact`**: Same as for generic messages.

> If `message` argument is passed, then the predefined message will be ignored.
> If `color` argument is passed, then the predefined color will be ignored.

## Examples

```groovy
def config = [
        channel: '#dev',
        message: 'This is my example message',
        color: 'good'
]
notifications.send(config)
//no need to send the `config` again, except for a custom message
notifications.success([ message: "All commands are good!" ])
notifications.failure([ message: "OoOops, something went wrong!" ])
```

```groovy
def config = [
        channel: '#dev',
        message: 'This is my example message',
        color: 'good'
]
notifications.deploymentStart(config)
//no need to send the `config` again, except for a custom message
notifications.deploymentSuccess()
```

```groovy
//all default options will be used, but the `channel`
//an inline map can be used as well
notifications.deploymentStart([ channel: '#dev' ])
```

```groovy
//all default options will be used
notifications.buildStart()
```

```groovy
//updating the last message
notifications.deploymentStart() //this message will be updated
notifications.deploymentUpdateMessage([ message: "Deployment for ABC started!"] )
```

```groovy
//updating the last message
notifications.deploymentStart()
notifications.deploymentSuccess() //this message will be updated
notifications.deploymentUpdateMessage([ message: "Deployment for ABC started!"] )
```

```groovy
//updating the a specific message
def myResponse = notifications.deploymentStart() //this message will be updated
notifications.deploymentSuccess()
notifications.deploymentUpdateMessage([ message: "Deployment for ABC started!", response: myResponse] )
```

```groovy
//replying to the original/first message
notifications.deploymentStart()  //reply message will be added to this message
notifications.deploymentReply([ message: "Deployment done!"] )
```

```groovy
//replying to the original/first message
notifications.deploymentStart()  //reply message will be added to this message
notifications.deploymentSuccess()
notifications.deploymentReply([ message: "Deployment done!"] )
```

```groovy
//replying to a specific message
notifications.deploymentStart()
def myResponse  = notifications.deploymentSuccess() //reply message will be added to this message
notifications.deploymentReply([ message: "That's a good example!", response: myResponse ])
```

```groovy
//reacting to the first/original message
notifications.deploymentStart()  //reaction will be added to this message
notifications.deploymentReact([ emoji: "nopain" ])
```

```groovy
//reacting to the last message
notifications.deploymentStart()
notifications.deploymentSuccess() //reaction will be added to this message
notifications.deploymentReact([ emoji: "nopain" ])
```

```groovy
//reacting to a specific message
def myResponse = notifications.deploymentStart() //reaction will be added to this message
notifications.deploymentSuccess()
notifications.deploymentReact([ emoji: "nopain", response: myResponse ])
```

[salck-plugin-docs]:https://github.com/jenkinsci/slack-plugin