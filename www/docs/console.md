---
title: Console
description: "The SST Console is a web based dashboard for managing your SST apps with your team."
---

import config from "../config";
import Calculator from "../src/components/PricingCalculator";
import HeadlineText from "@site/src/components/HeadlineText";

export const ConsoleUrl = ({url}) =>
<a href={url}>{url.replace("https://","").replace(/\/$/, "")}</a>;

<HeadlineText>

The <a href={config.console}>SST Console</a> is a web based dashboard to manage your SST apps.

</HeadlineText>

---

![SST Console homescreen](/img/console/sst-console-logs.png)

With the SST Console you can invoke functions, view and search logs, and manage all your apps with your team — **<ConsoleUrl url={config.console} />**

:::info
Looking for the Old Console? You can still access it here — <ConsoleUrl url="https://old.console.sst.dev" />
:::

---

## Quick start

Here's how to get started. <a href={config.console}>Head over to the Console</a> and create an account with your email.

1. **Create a workspace**

   ![SST Console create a workspace](/img/console/sst-console-create-new-workspace.png)
   
   You can add your apps and invite your team to a workspace. A workspace can be for a personal project or for your team at work. You can create as many workspaces as you want.

   :::tip
   Create a workspace for your organization. You can use it to invite your team and connect all your AWS accounts.
   :::

2. **Connect your AWS account**

   ![SST Console connect an AWS account](/img/console/sst-console-connect-aws-account.png)

   This will ask you to create a CloudFormation stack in your AWS account.

   ![SST Console connect an AWS account](/img/console/sst-console-create-cloudformation-stack.png)

   Make sure that this stack is being added to **us-east-1**. Scroll down and click **Create stack**.

   :::caution
   The CloudFormation stack needs to be created in **us-east-1**. If you create it in the wrong region by mistake, remove it and create it again.
   :::

   This stack will scan all the regions in your account for SST apps and subscribe to them. Once created, you'll see all your apps, stages, and the functions in the apps.

   ![SST Console app resources](/img/console/sst-console-resources.png)

3. **Invite your team**

   ![SST Console invite team](/img/console/sst-console-invite-user.png)

   Use the email address of your teammates to invite them. They just need to login with the email you've used and they'll be able to join your workspace.

---

## Requirements

**SST apps v2.19.2 or newer** are supported by the Console. Note that, apps **older than v2** won't be detected by the Console.

:::note
The SST Console is optional to use. It simply compliments the SST CLI.
:::

---

## How it works

At a high level, here's how the Console works.

1. It's hosted on our side

   - It stores some data about your account and your logs on our side.
   - We'll have a version that can be self-hosted in the future.

2. You can view all your apps and stages

   - Once you've connected your AWS accounts, it'll deploy a separate CloudFormation stack and connect to any SST apps in it.
   - And all your apps and stages will show up automatically.

3. You can manage your apps

   - You can view all the SST Functions in your app.
   - You can view their logs, invoke them, or replay invocations
   - You can also save event payloads to your workspace.
   - For your local [`sst dev`](live-lambda-development.md) stage, the logs will be streamed in real-time from your local machine.

4. It doesn't support all the features of the [Old Console](#old-console)

   - We are starting with just functions and logs for now. We might add the other [Explorers](#explorers) in the future.

5. It's open-source, built with SST, and deployed with [Seed](https://seed.run)

   - The Console is a full-stack SST app. You can view the <a href="https://github.com/sst/console">source on GitHub</a>.

---

### AWS account access

The SST Console needs access to your AWS account to do the following things:

1. Discover all the SST apps and stages across the various AWS regions.
2. Access the resources created while deploying your SST apps.
3. Access the CloudWatch APIs to let you view your logs in production.
4. Invoke the Lambda functions in your SST apps when you invoke them in the Console.

---

## Local logs

When the Console starts up, it checks if you are running `sst dev` locally. If so, then it'll show you real-time logs from your local terminal.

![SST Console tailing local logs](/img/console/sst-console-tailing-local-logs.png)

This works by connecting to a local server that's run as a part of the SST CLI.

:::info
The local server only allows access from `localhost` and `console.sst.dev`.
:::

The local logs works in all browsers and environments. But for certain browsers like Safari or Brave, and Gitpod, it needs some additional configuration.

---

### Safari & Brave

Certain browsers like Safari and Brave require the local connection between the browser and the `sst dev` CLI to be running on HTTPS.

SST integrates with [mkcert](https://github.com/FiloSottile/mkcert) to automatically generate a self-signed certificate. To set this up:

1. Follow the mkcert [installation steps](https://github.com/FiloSottile/mkcert#installation).
2. Run `mkcert -install` in your terminal.
3. Restart your browser.
4. Restart `sst dev` and navigate to <ConsoleUrl url={config.console} /> in the browser.

---

### Gitpod

If you are using [Gitpod](https://www.gitpod.io/), you can use the Gitpod Local Companion app to connect to the `sst dev` or `sst console` process running inside your Gitpod workspace.

To get started:

1. [Install Gitpod Local Companion app](https://www.gitpod.io/blog/local-app#installation).
2. [Run the Companion app](https://www.gitpod.io/blog/local-app#running).
3. Navigate to <ConsoleUrl url={config.console} /> in the browser.

The companion app runs locally and creates a tunnelled connection to your Gitpod workspace.

---

## Security

The CloudFormation stack that the SST Console uses creates an IAM Role in your account to manage your resources. If this is a concern for your production environments, we have a couple of options.

By default, this role is granted `AdministratorAccess`, but you can customize it to restrict access. We'll look at this below. Additionally, if you'd like us to sign a BAA, feel free to <a href={`mailto:${config.email}`}>contact us</a>.

There maybe we cases where you don't want any data leaving your AWS account. For this, we'll be supporting self-hosting the Console in the future. You can let us know if this is a priority for you by sending us a message over on Discord.

---

### IAM permissions

Permissions for the SST Console fall into two categories: read and write.

- **Read Permissions** — The Console needs specific permissions to display information about resources within your SST apps:

  | Purpose                                | AWS IAM Action                   |
  |----------------------------------------|----------------------------------|
  | Fetch stack outputs                    | `cloudformation:DescribeStacks`  |
  | Retrieve function runtime and size     | `lambda:GetFunctionCommand`      |
  | Access stack metadata                  | `s3:GetObject`<br/>`s3:ListObjectsV2`|
  | Display function logs                  | `logs:DescribeLogStreams`<br/>`logs:FilterLogEvents`<br/>`logs:GetLogEvents`<br/>`logs:StartQuery`|
  | Monitor invocation usage               | `cloudwatch:GetMetricData`       |

  Attach the `arn:aws:iam::aws:policy/ReadOnlyAccess` AWS managed policy to the IAM Role for comprehensive read access.

- **Write Permissions** — The Console requires the following write permissions:

  | Purpose                                          | AWS IAM Action                                                               |
  |-----------------------------------------------------|------------------------------------------------------------------------------|
  | Forward bootstrap bucket events to event bus     | `s3:PutBucketNotification`                                      |
  | Send events to SST Console                       | `events:PutRule`<br/>`events:PutTargets`                           |
  | Grant event bus access for SST Console           | `iam:CreateRole`<br/>`iam:DeleteRole`<br/>`iam:DeleteRolePolicy`<br/>`iam:PassRole`<br/>`iam:PutRolePolicy` |
  | Invok Lambda functions or replaying invocations  | `lambda:InvokeFunction` |

---

### Customize policy

To customize IAM permissions for the CloudFormation stack:

1. On the CloudFormation create stack page, download the default `template.json`.

   ![SST Console template URL](/img/console/sst-console-template-url.png)

2. Edit the template file with necessary changes.

  <details>
  <summary>View the template changes</summary>
  
    ```diff
        "SSTRole": {
          "Type": "AWS::IAM::Role",
          "Properties": {
            ...
            "ManagedPolicyArns": [
    -         "arn:aws:iam::aws:policy/AdministratorAccess"
    +         "arn:aws:iam::aws:policy/ReadOnlyAccess"
    +       ],
    +       "Policies": [
    +         {
    +           "PolicyName": "SSTPolicy",
    +           "PolicyDocument": {
    +             "Version": "2012-10-17",
    +             "Statement": [
    +               {
    +                 "Effect": "Allow",
    +                 "Action": [
    +                   "s3:PutBucketNotification"
    +                 ],
    +                 "Resource": [
    +                   "arn:aws:s3:::sstbootstrap-*"
    +                 ]
    +               },
    +               {
    +                 "Effect": "Allow",
    +                 "Action": [
    +                   "events:PutRule",
    +                   "events:PutTargets"
    +                 ],
    +                 "Resource": {
    +                   "Fn::Sub": "arn:aws:events:*:${AWS::AccountId}:rule/SSTConsole*"
    +                 }
    +               },
    +               {
    +                 "Effect": "Allow",
    +                 "Action": [
    +                   "iam:CreateRole",
    +                   "iam:DeleteRole",
    +                   "iam:DeleteRolePolicy",
    +                   "iam:PassRole",
    +                   "iam:PutRolePolicy"
    +                 ],
    +                 "Resource": {
    +                   "Fn::Sub": "arn:aws:iam::${AWS::AccountId}:role/SSTConsolePublisher*"
    +                 }
    +               },
    +               {
    +                 "Effect": "Allow",
    +                 "Action": [
    +                   "lambda:InvokeFunction"
    +                 ],
    +                 "Resource": {
    +                   "Fn::Sub": "arn:aws:lambda:*:${AWS::AccountId}:function:*"
    +                 }
    +               }
    +             ]
    +           }
    +         }
            ]
          }
        }
    ```
  
  </details>

3. Upload your edited `template.json` file to an S3 bucket.

4. Return to the CloudFormation create stack page and replace the template URL in the page URL.

:::tip
The SST Console is constantly evolving. As new features are added, additional permissions might be required. It's good practice to periodically review and update the IAM policy.
:::

---

## Pricing

The SST Console pricing is based on the number of times the Lambda functions in your SST apps are invoked per month and it uses the following tiers.

| Invocations | Rate (per invocation) |
|-------------|------|
| First 1M    | Free |
| 1M - 10M    | $0.00002 |
| 10M+        | $0.000002 |

These are calculated for a given workspace on a monthly basis. For volume pricing, feel free to <a href={`mailto:${config.email}`}>contact us</a>.

---

#### Pricing calculator

You can use this pricing calculator to estimate what your monthly cost will be.

<Calculator />

For further details, check out the [Pricing FAQ below](#pricing-faq).

---

## Getting help

If you have any questions or if you need help, <a href={config.discord}>**join us in #console on Discord**</a>.

---

## Old Console


The Old SST Console is a static single-page app hosted at <ConsoleUrl url="https://old.console.sst.dev" />

:::info
We'll be moving away from the Old Console in the future.
:::

#### Explorers

The Old Console has separate tabs or _explorers_ for managing the different parts of your application.

- Logs

  View **real-time logs** from your [Live Lambda Dev](live-lambda-development.md) environment.

- Stacks

  View all the deployed **stacks** and **resources** in your app.

- Functions

  **Invoke** the functions in your app and **replay** invocations.

- API

  The API explorer lets you **make HTTP requests** to any of the routes in your [`Api`](constructs/Api.md) and [`ApiGatewayV1Api`](constructs/ApiGatewayV1Api.md) constructs.

  Set the headers, query params, request body, and view the function logs in the response.

- RDS

  The RDS explorer allows you to manage the RDS instance created with the [`RDS`](constructs/RDS.md) constructs in your app.

  You can use the **query editor** to run queries. You can also use the migrations panel to view all of your **migrations and apply them**.

- Buckets

  The Buckets explorer allows you to manage the S3 Buckets created with the [`Bucket`](constructs/Bucket.md) constructs in your app.

  It allows you to **upload**, **delete**, and **download** files. You can also create and delete folders.

- GraphQL

  The GraphQL explorer lets you **query GraphQL endpoints** created with the [`Api`](constructs/Api.md) and [`AppSyncApi`](constructs/AppSyncApi.md) constructs in your app.

- Cognito

  The Cognito explorer allows you to manage the User Pools created with the [`Cognito`](constructs/Cognito.md) constructs in your app.

  It allows you to **create** new users and **delete** existing **users**.

- DynamoDB

  The DynamoDB explorer lets you **query the DynamoDB** tables in the [`Table`](constructs/Table.md) constructs in your app.

  You can scan the table, query specific keys, create and edit items.

---

## FAQ

- Do I need to use the Console to use SST?

  You don't need the Console to use SST. It displays the local logs from your terminal in a UI that's more convenient.

- What if I don't want to pay for the Console?

  You can still invite your team and use it to view your local logs and stages.

- Why did we move away from the Old Console?

  It required you to run a command when you wanted to view logs for a specific stage. It also was purely a client-side app, this made it very limited for viewing or searching production logs.

- What will happen to the [Old Console](#old-console)?

  It'll be available at <ConsoleUrl url="https://old.console.sst.dev" /> for some time but we'll be moving away from it.

- What will happen to Seed?

  [Seed](https://seed.run) also lets you view logs for your SST apps, so there is some overlap between the two products. But Seed will continue to work just as before.

---

### Pricing FAQ

- Do I need a credit card to get started?

  The Console is free to get started and **doesn't need a credit card**.

- Which Lambda functions are included in the number of invocations?

  The number of invocations are only counted for the **Lambda functions in your SST apps**. Other Lambda functions in your AWS accounts are not included.

- Do the functions in my local stages count as a part of the invocations?

  Lambda functions that are invoked **locally are not included**.

- Can I access the local stages if I'm above the free tier?

  If you go above the free tier in your _production_ stages, you **can still access your local stages**. You'll just need to add your billing information to access the _production_ ones.

- My invocation volume is far higher than the listed tiers. Are there any other options?

  Feel free to <a href={`mailto:${config.email}`}>contact us</a> and we can figure out a pricing plan that works for you.

If you have any further questions, feel free to ask us on Discord or <a href={`mailto:${config.email}`}>send us an email</a>.
