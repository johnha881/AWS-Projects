Event driven architecture with the use of EventBridge service.

Services used:
EventBridge
SQS
Step Functions
SNS

Advantage of this type of architecture:
1. Producers and Consumers are decoupled. By being independant of each other, scaling would be less of an issue.
2. Filtering and Routing to the correct target depending not the rules stated.


Setup

Create EventBridge
1. Navigate to EventBridge in your console.
2. Select Event Buses under Buses.
3. Create event bus.

![eventbuscreation](https://github.com/user-attachments/assets/07c9ff5e-4221-425a-9640-da58000e96ba)

1.In "create event bus" page create a name of your choice (my-event-bus will be mine)
![2nd event buss](https://github.com/user-attachments/assets/8a3b71af-e23f-4670-aae0-c78a298790f4)

2. Leave the rest as default. 
3. Select Create at bottom of page.

Rules:
1. Back at the EventBridge page, select rules and create rule.
2. Select "my-event-bridge" as the bus name the rule will apply to

![13](https://github.com/user-attachments/assets/6f31e41b-74b7-40f8-9202-7d94e00ae509)

3. At the Define rule detail page:
   - Name the rule ( Mine will be rule-for-testing)
   - Description: Testing by catching all events
   - Rule type: Rule with an event pattern 
![4](https://github.com/user-attachments/assets/9fe4eb65-fb8f-4222-aa13-94477ded2cd1)

4. On the Build event pattern page:
   -Select "Other" as Event source

![5](https://github.com/user-attachments/assets/687b4440-34a0-4cee-8b9f-41489308c30d)





