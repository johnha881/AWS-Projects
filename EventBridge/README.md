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
1. Navigate to EventBridge in your console
2. Select Event Buses under Buses
3. Create event bus

![eventbuscreation](https://github.com/user-attachments/assets/07c9ff5e-4221-425a-9640-da58000e96ba)

