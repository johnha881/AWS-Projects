Event driven architecture with the use of EventBridge service.

Services used:
EventBridge
SQS
Step Functions
SNS

Advantage of this type of architecture:
1. Producers and Consumers are decoupled. By being independent of each other, scaling would be less of an issue.
2. Filtering and Routing to the correct target depending on the rules stated.


**Setup**

**EventBridge Event Bus**
1. Navigate to EventBridge in your console.
2. Select Event Buses under Buses.
3. Create event bus.

![eventbuscreation](https://github.com/user-attachments/assets/07c9ff5e-4221-425a-9640-da58000e96ba)

1.In "create event bus" page create a name of your choice (my-event-bus will be mine)

![2nd event buss](https://github.com/user-attachments/assets/8a3b71af-e23f-4670-aae0-c78a298790f4)

2. Leave the rest as default. 
3. Select Create at bottom of page.

**Rules:**
1. Back at the EventBridge page, select rules and create rule.
2. Select "my-event-bridge" as the bus name the rule will apply to.

![13](https://github.com/user-attachments/assets/6f31e41b-74b7-40f8-9202-7d94e00ae509)

3. At the Define rule detail page:
   - Name the rule ( Mine will be rule-for-testing).
   - Description: Testing by catching all events.
   - Rule type: Rule with an event pattern .

![4](https://github.com/user-attachments/assets/9fe4eb65-fb8f-4222-aa13-94477ded2cd1)

4. On the Build event pattern page:
   A. Select "Other" as Event source.

   ![14](https://github.com/user-attachments/assets/181cee96-454b-4fba-889d-615feaa6201e)

   B. Under Event pattern, enter: 
      {
        "source": ["com.aws.my-event-bus"]
      }

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
   - {
        "category": "Colors",
        "color": "red",
        "location": "us-east"
       }
      for Event detail

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

**SNS Creation**

We will be using Simple Notification Service (SNS) to receive events from EventBridge. 

1. Navigate to SNS services on the console
2. On the left pane of the page select topic
3. Click on create topic button
4. Choose Standard as Type
5. Name will be colors for this topic
6. Display name will be Colors SNS

Once created we will now need to provide subscription for this Topic.
1. Go back to topic and select color under topics
2. Click Create subscription

image

3. I will select Email as my subscriber.
4. Endpoint will be my email( I will need to go to my email and confirm the subscription)

image



5. Create subscription
6. Go to email and subscribe to this topic

image

**Add rule to route and event to SNS**

Head Back to EventBridge page and click on rule -> create rule
image
Under Define Rule datail page enter:
"SendColorEventsToSNS" for the name
"Send event to SNS" for description 
Select "my-event-bus" for event bus
Click next

image

On the "build event pattern" page scroll down to "Create method" section
Select "Custom pattern (JSON Editor) and enter

{
  "source": ["com.aws.my-event-bus"],
  "detail": {
    "category": ["Colors"],
    "color": ["red"],
    "location": ["us-east"]
  }
}

Click next


Select targets page select "AWS service".
For Select a target chooose " SNS topic
Select colors from Topic drop down menu
Click next until you are able to click on create rule

**Test SNS**

Go back to Eventbridge to test the new rule
Repeat these steps:

Select/Fill in the these values:
   - my-event-bus for Event Bus
   - com.aws.my-event-bus for Event source
   - Notification for detail type
   - {
        "category": "Colors",
        "color": "red",
        "location": "us-east"
       }
      for Event detail

Sample of email recieved after sending test:

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

