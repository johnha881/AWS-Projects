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
