Event driven architecture with the use of EventBridge service.

Services used:
- EventBridge
- SQS
- Step Functions
- SNS

Advantage of this type of architecture:
1. Producers and Consumers are decoupled. By being independent of each other, scaling would be less of an issue.
2. Filtering and routing to the correct target depend on the rules specified.


**Setup**

**EventBridge Event Bus**
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
   - Name the rule ( Mine will be rule-for-testing).
   - Description: Testing by catching all events.
   - Rule type: Rule with an event pattern .

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

**SNS Creation**

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

1. Go back to EventBridge to test the new rule.
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

