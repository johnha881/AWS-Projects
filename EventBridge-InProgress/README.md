# Event driven architecture with the use of EventBridge service pushing events out to SQS, SNS and Step Functions.

## Table of Contents
   - [Introduction](#introduction)
   - [Setup](#setup)
      - [EventBridge Event Bus](#eventbridge-event-bus)
      - [SNS Creation](#sns-creation)
      - [SQS](#sqs)
      - [Step Functions](#step-functions)
   - [Scaling](#scaling)
   - [Availability](#availability)
   - [Cost Optimization](#cost-optimization)
   - [Security](#security)

## Introduction
This project showcases how to use AWS EventBridge as an event bus to manage and route events, with Amazon SQS, AWS Step Functions, and Amazon SNS serving as consumers. It demonstrates the setup of EventBridge rules to capture and filter events, which are then forwarded to SQS for queuing, processed through Step Functions for workflow orchestration, and published to SNS for notifications. The project also discusses considerations for scaling, availability, cost optimization, and security across these services.

Advantage of this type of architecture:
1. Producers and Consumers are decoupled. By being independent of each other, scaling would be less of an issue.
2. Filtering and routing to the correct target depend on the rules specified.


## Setup

### EventBridge Event Bus
1. Navigate to EventBridge in your console.
2. Select Event Buses under Buses.
3. Create event bus.

![eventbuscreation](https://github.com/user-attachments/assets/07c9ff5e-4221-425a-9640-da58000e96ba)

4. In "create event bus" page create a name of your choice (my-event-bus will be mine).

![2nd event buss](https://github.com/user-attachments/assets/8a3b71af-e23f-4670-aae0-c78a298790f4)

5. Leave the rest as default. 
6. Select Create at bottom of page.

**Rules:**
1. Back at the EventBridge page, select rules and create rule.
2. Select "my-event-bridge" as the bus name the rule will apply to.

![13](https://github.com/user-attachments/assets/6f31e41b-74b7-40f8-9202-7d94e00ae509)

3. At the Define rule detail page:
   - Name the rule ( Mine will be rule-for-testing)
   - Description: Testing by catching all events
   - Rule type: Rule with an event pattern 

![4](https://github.com/user-attachments/assets/9fe4eb65-fb8f-4222-aa13-94477ded2cd1)

4. On the Build event pattern page:
   A. Select "Other" as Event source.

   ![14](https://github.com/user-attachments/assets/181cee96-454b-4fba-889d-615feaa6201e)

   B. Under Event pattern, enter: 
      ```
      {
        "source": ["com.aws.my-event-bus"]
      }
      ```
   ![6](https://github.com/user-attachments/assets/e4b3442f-708a-4304-8657-6904fce63198)

   C. Under Select targets page:
      - Select AWS Service as Target type.
      - Select CloudWatch Log Group under Select a target.
      - Enter "my-event-bus" as Log Group (/aws/events/my-event-bus).
      - Click Next until you are able to review and create rule.

![15](https://github.com/user-attachments/assets/54b43805-5205-4bca-b70e-d7fd23f6e5af)

**Testing Event**

1. Select Event Buses again and Click on Send events

![16](https://github.com/user-attachments/assets/b1a5ad80-7da0-42cd-a6fd-8b94217e70d5)

2. Select/Fill in the these values:
   - my-event-bus for Event Bus
   - com.aws.my-event-bus for Event source
   - Notification for detail type
   - For Event detail:
      ```
     {
        "category": "Colors",
        "color": "red",
        "location": "us-east"
       }
      ```
  
      
![21](https://github.com/user-attachments/assets/2a4c2959-f7ca-420c-956b-1997a3928524)


3. Click Send

4. Head over to CloudWatch page and find Logs on left pane
5. Select Log Groups and click on /aws/events/my-event-bus

![18](https://github.com/user-attachments/assets/f9dfa4ff-fd2f-4d31-bab5-576d5a6131e4)

6. Under Log streams click on the recently created stream

![19](https://github.com/user-attachments/assets/b3037f4a-99db-429f-88f4-53ce78aff9dc)

7. Here you will see the event details that were sent to CloudWatch Logs

![1](https://github.com/user-attachments/assets/07be9960-26cd-486c-914e-b52ac494f588)

**Rules Matching**

Rules determine which services an event will trigger. They are processed in parallel, without a specific order. JSON format will be used by EventBridge for rules to match event patterns. Patterns need to be precise(Uppercase/Lowercase/decimal/etc matter).

## SNS Creation

We will be using Simple Notification Service (SNS) to receive events from EventBridge. 

1. Navigate to SNS services on the console.
2. On the left pane of the page, select "Topics."
3. Click on the "Create topic" button.

![2](https://github.com/user-attachments/assets/2ca0c8df-beff-488c-900d-97a3d36f2f86)

4. Choose "Standard" as the Type.
5. Set the Name to "colors" for this topic.
6. Set the Display name to "Colors SNS."

![3](https://github.com/user-attachments/assets/9c1786b4-4af3-4fea-95a6-1a9bf8287fbf)

Once created, we will now need to provide a subscription for this Topic.
1. Go back to "Topics" and select "colors."
2. Click "Create subscription."

![10](https://github.com/user-attachments/assets/124e6399-e05a-48fb-bba9-cbdf7b0439ba)

3. For Protocol, select "Email" as the subscriber.
4.For Endpoint, enter your email (you will need to confirm the subscription via email).

![11](https://github.com/user-attachments/assets/1e44b2a2-6914-42d9-848f-2833da3fa9de)

5. Click "Create subscription."
6. Go to your email and confirm the subscription to this topic.

![12](https://github.com/user-attachments/assets/c982c63a-0649-467d-8e5d-4f7f9223b3e2)

**Add rule to route an event to SNS**

1. Head back to the EventBridge page and click on "Rules" -> "Create rule."

![4](https://github.com/user-attachments/assets/c4f4fe3f-75a5-4ae3-9b85-ed5c15da8f2d)

2. Under the "Define Rule detail" page, enter:
   - Name: "SendColorEventsToSN
   - Description: "Send event to SNS" 
   - Event bus: "my-event-bus"
  
![5](https://github.com/user-attachments/assets/17d88b6f-9be5-4e45-8cea-1d7dc0538843)

3. Click "Next."

4. On the "Build event pattern" page, scroll down to the "Create method" section:
   - Select "Custom pattern (JSON Editor)" and enter:

```
{
  "source": ["com.aws.my-event-bus"],
  "detail": {
    "category": ["Colors"],
    "color": ["red"],
    "location": ["us-east"]
  }
}
```
![6](https://github.com/user-attachments/assets/f0fd4dd7-3484-4639-a89e-290681bf8d82)

5. Click "Next."

6. On the "Select targets" page:
- Select "AWS Service" for the "Target type."
- For "Select a target," choose "SNS topic."
- Select "colors" from the "Topic" drop-down menu.

![14](https://github.com/user-attachments/assets/a9dbc48d-2c2a-437a-8827-caa8b45e2be3)

7. Click "Next" until you reach the "Review and create rule" section.
8. Click "Create rule."

**Test SNS**

1. Return to EventBridge to test the new rule.
2. Repeat these steps:
   - Event Bus: "my-event-bus"
   - Event source: "com.aws.my-event-bus"
   - Detail type: "Notification"
   - Event detail:
   ```
     {
        "category": "Colors",
        "color": "red",
        "location": "us-east"
     }
   ```

Sample of Email Received After Sending Test:

```
{
  "version": "0",
  "id": "XXXXXXXXXXXXXXXXXX",
  "detail-type": "notification",
  "source": "com.aws.my-event-bus",
  "account": "XXXXXXXXXXXX",
  "time": "2024-08-03T02:12:25Z",
  "region": "us-east-1",
  "resources": [],
  "detail": {
    "category": "Colors",
    "color": "red",
    "location": "us-east"
  }
}
```
![13](https://github.com/user-attachments/assets/45be14db-4798-4dd8-a6d8-8529438b72c8)


## SQS

1. Head to Simple Queue Service(SQS) page.
2. Select "Create queue"

![15](https://github.com/user-attachments/assets/cbed1f81-3518-47bb-a2a5-8a4fd86caa4d)

3. For Type choose "standard" as we do not need First-in-First-out delivery.
4. I will use "my-sqs-queue" as the name of the queue.
5. We will leave everthing else as default.

![16](https://github.com/user-attachments/assets/0fb15582-d3d8-4a55-a2bb-2d208bd4e13b)

**Create rule for SQS**

1. Return to EventBridge and create another rule for "my-event-bus".
2. Rule details:
   - Name: event-to-sqs-rule
   - Description: Rule to send events to SQS
   - Event bus: my-event-bus
   - Rule type: Rule with an event pattern
3. On the "Build event pattern" page, repeat the steps with on minor change:
   - The "color" value will be "blue" instead of "red"

![18](https://github.com/user-attachments/assets/d67e2cc6-3a3e-43b6-bbb1-92bc1c4de221)

4. On the "Select Targets" page, enter the following:
   - Target types: AWS service
   - Select a target: SQS queue
   - Queue: my-sqs-queue

![19](https://github.com/user-attachments/assets/8c78a0e6-1d9b-46d2-bb66-3927dda55a09)

5. Leave everything else as default and select "Create rule".

**Test SQS"

1. Return to EventBridge to test the new rule.
2. Repeat these steps:
   - Event Bus: "my-event-bus"
   - Event source: "com.aws.my-event-bus"
   - Detail type: "Notification"
   - Note the change of  color from red to blue
   - Event detail:
     
   ```
     {
        "category": "Colors",
        "color": "blue",
        "location": "us-east"
     }
   ```
3. Head back to SQS page and select "my-sqs-queue".
4. Expand the "more" button and you will see that one message is available.
 
![20](https://github.com/user-attachments/assets/e348ca51-21bb-4a6d-9e65-6cee0cd3dfc4)

5. To see the actual message, we will need to poll the queue:
   - Click on: Send and receive messsages
   
   ![21](https://github.com/user-attachments/assets/609b1c49-5852-4fc9-aedb-242fdc0c1623)

   - Click on: Poll for messages for the "Receive messages" section
    
  ![22](https://github.com/user-attachments/assets/a437e256-7c03-49b9-aadd-8eff82ed2616)

6. Under "Messages" you will a new message appear; select the item.

![23](https://github.com/user-attachments/assets/e84fc0b4-0815-4ec8-8745-bb914a00b54c)

7. Take not of the "color" "blue" that is shown in the message details. If we sent a message that kept the color "red", SQS would not have received this event.

![24](https://github.com/user-attachments/assets/9c429818-92d7-407a-9d12-d5866a56533d)


## Step Functions

**Create Step function**

1. Head over to "Step Functions" page.
2. Select "State machines"and click on "Create state machine".

![1 step](https://github.com/user-attachments/assets/3e61cb7e-f626-4b8d-9283-45898553321c)

3. When choosing a template, select "Blank"

![2](https://github.com/user-attachments/assets/7400880e-7bef-485a-8d84-f41fff308058)

4. The "Design" page will be displayed. We will use a simple "Pass" execution to make sure the step function works.

![3](https://github.com/user-attachments/assets/b0d425a9-1f6f-495a-beba-8f617edc68a6)

5. Click on "Flow" and drag the selection on to the "dragon first state here" template

![4](https://github.com/user-attachments/assets/4bb9dad6-5df0-4607-a3db-b8c8525faa6d)

6. Click on "Code" tab, which will bring you to the JSON format of what has been done.

![5](https://github.com/user-attachments/assets/8a603a5a-22a1-4aab-962d-eb41099a8deb)

7. Next, Click on Config and do the following:
   - State machine name: "ProcessEventStepFunction"
   - Type: "Standard"
   - Permissions: Execution role: "Create New Role"

![6](https://github.com/user-attachments/assets/75a17955-4b1c-4791-a315-c46295182734)

8. Click "Create" at the top right.
9. A pop-up to confirm the creation will appear. Note the role that will be created. Click "Confirm"

![7](https://github.com/user-attachments/assets/74e9bb60-aae5-4fcb-b8c4-136ca471ee93)

**Test Step Function**

1. After creating the Step function, you will be taken to its page named "ProcessEventStepFunction".
2. Click "Start execution"

![8](https://github.com/user-attachments/assets/d5c4ddc7-8c34-46cd-9e03-c24da95b45a7)

3. A pop-up will appear:
   - Name: Leave as default
   - input:  {
           "msg": ["Hello World!"]
             }

![9](https://github.com/user-attachments/assets/ed2ac619-417a-4519-8bb7-ce8d10f8252d)

4. Click on "Start execution"

5. If successful, you will see this output:

![11](https://github.com/user-attachments/assets/610d007a-b5b4-44fe-ac8e-04bf6b9f134d)

**Sending Event from EvenBridge to Step Functions**

1. Back at the EventBridge page, select rules and create rule.
2. Select "my-event-bridge" as the bus name the rule will apply to.
3. Create Rule.

4. Define rule detail:
   - Name: "event-to-stepFunction-rule"
   - Description: "Event sent to step function"
   - Event bus: "my-event-bus"
   - Rule type: "Rule with an event pattern"
   - Create

5. Build event pattern:
   - Event source: "Other"
   - Event Pattern:
```
{
   "source": ["com.aws.my-event-bus"],
   "detail": {
      "category": ["Colors"],
       "color": ["green"],
       "location": ["us-east"]
  }
}
```
   - Next

6. Select target:
   - Target type: "AWS service"
   - Select a target: "Step Functions state machine"
   - State machine: "ProcessEventStepFunction"
   - Execution role: "Create a new role for this specific resource"
   - Role name: "Amazon_EventBridge_Invoke_Step_Functions_824957223" (randomly created when create new role)

![14](https://github.com/user-attachments/assets/35f9164b-fc52-4982-b078-b2e3bfff53df)

7. Click next until you can "Create rule".

8. Return to EventBridge to test the new rule.
9. Repeat these steps:
   - Event Bus: "my-event-bus"
   - Event source: "com.aws.my-event-bus"
   - Detail type: "colors"
   - Note the change of color: "green"
   - Event detail:
   - Send
   ```
     {
        "category": "Colors",
        "color": "green",
        "location": "us-east"
     }
   ```

![15](https://github.com/user-attachments/assets/a7518a5f-2626-4a8e-9d52-0df1d5fee083)

10. Return to Step Functions page; find and select "ProcessEventStepFunction" function
11. If event was successfully sent to Step function, you will see the new entry for execution

![16](https://github.com/user-attachments/assets/ac8bb5f9-d4b9-468b-a249-ab031c193bea)

12. Select that execution and scroll down to "Event view"
13. Expand where it says "ExecutionSucceded" and you will see your Event information that was passed from EventBridge.

![17](https://github.com/user-attachments/assets/339f339c-323f-4aae-b9e4-976576bc4d8e)

## Scaling

### EventBridge

   - Scales automatically (there is a softcap on quotas).
   - Make sure consumers are able to handle the amount of events being sent.
   
### SNS

   - SNS has inherent scaling. It will automatically scale for you.
   - SNS will also scale with the amount of subscribers for the topic.

### SQS

   - SQS has inherent scaling. It will automatically scale for you.
   - Horizontal scaling with auto-scaling groups(EC2) and ECS to poll larger loads. (Visibility time out to avoid same messaging being processed multiple times).

### Step Functions

   - Scales automatically by adjusting operations and compute resources.
   - Map state processing to run multiple step functions in parallel

## Availability

### EventBridge

   - Highly available through multiple AZ deployments.
   - Option to enable cross-region event routing.
   - Guaranteed 99.99% availability for event delivery

### SNS

   - SNS is highly available through regions and Availability Zones(AZ).
   - 99.9% availability through multiple servers and data centers.
   - 2 weeks retention period for undelivered messages.
   - Cross-region subscription is available for pseudo-replication."

### SQS

   - SQS is highly available through regions and Availability Zones(AZ).
   - 99.9% availability through multiple servers and data centers.

### Step Functions

   - Highly available through multiple AZ deployments.

## Cost Optimization

Use CloudWatch Alarms to be alerted to unwanted actions which would increase cost.

### EventBridge

   - Be aware of payload size.
   - Proper event patterns to filtering.
   - Limit number of event buses.

### SNS

   - Standard VS FIFO(First in, First out).
     - Standard is used for non-critical scenarios where cost is lower and messages might arrive more than once.
     - FIFO is for order-critical, one message only delivery. Cost more than standard.
   - Be aware of payload size.
   - SMS/email subscribers are more expensive than lambda/http.

### SQS

   - Track duplication of messages.
   - Pay-as-you-go pricing for no upfront cost.
   - Short polling vs long polling (less amount of pull request)
   - Message size, smaller the better
   - Use visibility time out appropriately to avoid messages being processed more than once.

### Step Functions

   - Pay-per-use
   - 4,000 state transitions per month free, $0.025 per 1,000 state transitions after.

## Security 

All services can use VPC endpoints to limit access from outside environments.

### EventBridge

   - IAM policies for access.
   - Least privilege to send and receive events.
   - CloudWatch log to see user activity.
   - AWS encryption at rest and in transit.

### SNS

   - Topic policies to limit who can publish and subscribe.
   - Server-side encryption on topics.
   - IAM users, roles, and policies for access.

### SQS

   - Least privilege to send and receive events.
   - IAM users, roles, and policies for access.
   - SSL encryption of messages in transit.

### Step Functions

   - IAM users, roles, and policies for access. 

