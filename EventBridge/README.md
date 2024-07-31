Event driven architecture with the use of EventBridge service.

Services used:
EventBridge
SQS
Step Functions
SNS

Advantage of this type of architecture:
1. Producers and Consumers are decoupled. By being independant of each other, scaling would be less of an issue.
2. Filtering and Routing to the correct target depending not the rules stated.


**Setup**

**Create EventBridge**
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

   B. Under Even pattern enter.  
   {
     "source": ["com.aws.my-event-bus"]
   }

![6](https://github.com/user-attachments/assets/e4b3442f-708a-4304-8657-6904fce63198)

   C. Under Select targets page:
      -Select AWS Service as Target type.
      -Select CloudWatch Log Group under Select a target.
      -Enter "my-event-bus" as Log Group(/aws/events/my-event-bus).
      -Click Next until you are able to review and create rule.

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
        "value": 101,
        "location": "us-east"
       }
      for Event detail







