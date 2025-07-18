# Technical Design Document Template

---

## Author

- **Name**: _Ming Yue _

---

## Status

- **Draft**

---

## Summary

We are proposing building a Requisition System that can act as a bridge between the existing MES and IMS services. This system will automate item/part requisition based off of work order units from MES and IMS inventory data. The system will communicate with the existing MES and IMS using both gRPC when making traditional API calls, and Apache Kafka to broadcast events and updates in real-time. Ideally, the Requisition System will help solve frictions that arise during the item requisition stage of the manufacturing process.

---

## Motivation

The current work flow of warehouse techs involving the MES and IMS is somewhat error prone. Warehouse workers have to read through work orders and try and fill required parts and items needed for the production of a work order unit. This error-prone system can make it hard to track item stock levels, as well as keeping track of where items are exactly.

---

## Goals

We want to create a service that can:

- Integrate with the existing MES and IMS services and leverage existing
- Automate item/part requisitions based off of work order units.
- Using heuristics and inventory data from the IMS, properly allocate and figure out where items should be picked up from
- Provide a clear UI that helps warehouse techs identify parts/items for pickup, as well as update item requisition/delivery status in a simpler, less error prone fashion. (They can mark and update line items as they pick them up and deliver them).
- Provide updates to both MES and IMS in real time, enabling users of MES to track the progress of item/part allocation, and enabling IMS to also update item/part inventory level quicker
- Improve the traceability of of items/parts.
- Reduces errors in the overall item requisition process.

---

## Non-Goals

As part of this proposal, we will not:

- Aim to fully replace or alter the core functionalities of the MES or IMS system.
- Handle supplier management or item/part procurement
- Have financial integration for cost tracking/budgeting
- Have any advanced analytics or predictive modeling
- Automate inventory replenishment

---

## Proposal

In order to fill the item requisition gap that exists between MES and IMS, we will create a Requisition System that sits in between the two. The Requisition system will handle automatic RequisitionOrder creation, based off of work orders from MES.

This process will first kick-off when work orders are created in MES. MES will produce an event for an Apache Kafka broker, of which the proposed Requisition System is a subscriber to. Once the Requisition System consumes this event, it will automatically make requests via gRPC to MES for more information regarding the work order created (e.g. what operations are involved, what items are required for this operations). Requisition System will then make requests to IMS via gRPC to ascertain item stock information (e.g. how much of an item stock is available, at which locations are they located, etc.). Using the gathered information and heuristics written in business logic, Requisition System will then create a RequisitionOrder. Each RequisitionOrder can then have multiple RequisitionItem associations, which represent line-item parts/items required for the complete work order. Based on how many units exist for this work order, RequisitionItem could additionally update IMS inventory for stocks that will be reserved.

The Requisition System will also include UI for warehouse technicians to access. This UI will more clearly list out items that need to be retrieved by the warehouse tech, ideally at their warehouse location only. As technicians retrieve and deliver items, they can update their progress. These updates will trigger Requisition System in publishing events for the Kafka broker, and subscribers (IMS and MES in this case) will be notified. MES can update work order progress if needed, and IMS can properly update inventory stock information.

---

## Design and Implementation Details

_Provide technical details of your design. Include diagrams, API specifications, database schemas, or code snippets as appropriate._

![Requisition System Architectual Diagram](https://lucid.app/lucidchart/84bdb830-4419-4d4c-ba79-229a953285ff/edit?viewport_loc=-1175%2C-212%2C1673%2C913%2C0_0&invitationId=inv_b352d669-0c48-4f5c-a7ab-bee12c283c6a)

![Requisition System Schemas and Models]()

![Requisition System Protobuf definitions]()

- **System Architecture**:

The archecture is fairly simple, it mainly consists of the Requisition System core service, which contains the business logic. An Apache Kafka broker will be used as an event bus, to notify MES, IMS, and Requisition System of events such as work order creation, or requisition updates. Requisition System will also connect to IMS and MES via gRPC. We will connect a Redis cache to the Requisition System, for faster access to requisition orders and items. A SQL database will also exist, for more persistent storage. Requisition System will ideally only read from this database. All writes will be performed to Redis, which will periodically flush updates to database. To make the system more robust, we could consider containerizing the Requisition System and running multiple docker containers.

- **Components**:

  - Requisition System Service: This is a backend service that contains the business logic, including RequisitionOrder and RequisitionItem CRUD operations, MES and IMS integrations, and other relevant APIs. Should Ideally should be containerized for scalibility and fault tolerance.
  - Requisition Front-end: This serves as a UI, mainly intended for warehouse techs, streamlining part acquisition.
  - Database: Stores RequisitionOrder and RequisitionItem tables. Should have replication/backups for fault tolerance.
  - Redis cache: Caches RequisitionOrder and RequisitionItems for faster access, can have a cache replacement policy like least recently used. Should periodically flush to database, and have snapshot backup
  - Apache Kafka: Kafka broker will act as an event bus and notify subscibers asynchronously of various events, like work order creation, item pickup/dropoff, etc.

- **Data Flow**:
  ![Requisition System Sequence Diagram](https://mermaid.ink/img/pako:eNqVk01P4zAQQP_KaK6kVdw0SeMDF-CAVpVgASGtuFjJtLXa2N2xDQtV_zuuS2FXsAhyc-z35sveYGs7QomOfgcyLZ1qNWfV3xmI3_TsajA4Pj76oWZLJeGCbRdaAron4yUYeoAHy0uw3BFDy6Q8dXsyEYn9GcXaaa-tuSK-1y1JOLHGhf5Lovd4lMa8JFwG4se_uAzsmljtTroMlOlAe-qBdwKmPkZynyjPp6_KHebAedsuQZuZ5T5JP4FvFdPCBkdwTe1Cwql265U6mLyNSXjWsVrYS_49HwUfNelm3cU2HIpIvFrFtJQP_y8kuj4e1pvmzQIhhXg3stSLw4zOvs-n8XyLxwznrDuUngNl2FPs-W6Jd2azjZtrZX5Z2x_22Yb5AuVMrVxc7S0vF_f1L5OJl-LEBuNRFnXeJAvKDf5BORDVsM5HIm-EqIqyqooMH1GKST0UYjwZj_MmbyYjUW0zfEqBxVDUZVnXTV4UdTlqykmG1Glvebp_QK01Mz3H7TOlryE5?type=png)](https://mermaid.live/edit#pako:eNqVk01P4zAQQP_KaK6kVdw0SeMDF-CAVpVgASGtuFjJtLXa2N2xDQtV_zuuS2FXsAhyc-z35sveYGs7QomOfgcyLZ1qNWfV3xmI3_TsajA4Pj76oWZLJeGCbRdaAron4yUYeoAHy0uw3BFDy6Q8dXsyEYn9GcXaaa-tuSK-1y1JOLHGhf5Lovd4lMa8JFwG4se_uAzsmljtTroMlOlAe-qBdwKmPkZynyjPp6_KHebAedsuQZuZ5T5JP4FvFdPCBkdwTe1Cwql265U6mLyNSXjWsVrYS_49HwUfNelm3cU2HIpIvFrFtJQP_y8kuj4e1pvmzQIhhXg3stSLw4zOvs-n8XyLxwznrDuUngNl2FPs-W6Jd2azjZtrZX5Z2x_22Yb5AuVMrVxc7S0vF_f1L5OJl-LEBuNRFnXeJAvKDf5BORDVsM5HIm-EqIqyqooMH1GKST0UYjwZj_MmbyYjUW0zfEqBxVDUZVnXTV4UdTlqykmG1Glvebp_QK01Mz3H7TOlryE5)

  1). A work order is created in MES.
  2). MES produces an event notifying the Kafka Broker that a work order is created.
  3). The Requisition System, which is subscribed the work order created topic, is notified of the new work order.
  4). Requisition System then queries MES via gRPC for operations associated with the new work order, and all items required by these operations. It then queries for work order units associated with the work order.
  5). Requisition System queries IMS via gRPC for item and stock information required by the work order
  6). Based off of information queried, Requisition System will create at least one RequisitionOrder, and associated RequisitionItems. These will be stored in database and Redis cache.
  7). Warehouse tech will access Requisiton System UI to view items needed to be picked, and update statuses as they are picked.
  8). As statuses are updated, Requisition system will update the RequisitionItem in Redis, (Redis will flushed to database on a set schedule to ensure data consistency).
  8). As statuses are updated, Requisition system will produce an "item picked" event to the Kafka broker.
  9). MES and IMS will consume this event and update their records accordingly.

- **Integration Strategy**:

MES: To communicate with MES, the Requisiton system will leverage both Kafka and gRPC. Kafka will be used to asynchronous notify Requisition System that work orders are created, and also for Requisition System to broadcast updates to the requisition progress. To actually query Work Order and Operation information, Requisition System will have to use gRPC. An additional API must be added to MES to help query Work Order Unit information.

IMS: To commnicate with IMS, Requisition system will leverage both Kafka and gRPC as well. To query inventory stock information, it will use standard gRPC communcation. To notify IMS of items being picked up and inventory data needs updating, Kafka will act as the broker.

- **Development and Rollout Strategy**:
  Phase 1: Build and implement Requisition System core services/API, database schema, UI.
  Phase 2: Implement Kafka and Redis services.
  Phase 3: Integrate and test with IMS and MES.
  Phase 4: Write unit and integration tests for business logic and APIs, and end-to-end testing
  for entier system
  Deployment: Containzerize services and deploy.
  Rollout: Gradually and partially rollout to warehouse technicians/warehouses.

---

## Assumptions and Dependencies

- There exists some procedures to handle partially filled requisitions, when there are only a certain number of the required items are available.
- There exists some form of user authorization/authentication, such that we can build role based access for the Requisition System around this.
- Warehouse techs will be able have access to devices that can utilize the Requisition System. For results closer to real-time, ideally this would be a mobile device.
- There will be tolerable connections between MES, Requisition System, and IMS.

---

## Alternatives Considered

One alternative considered intially, but not really, was to forgo the Requisition System entirely, and just build additional functionality within MES. This was not fully considered because MES and IMS would probably end up very tightly coupled, and this functionality would probably be out of MES's domain.

---

## Conclusion

The Requisition System we proposed here will provide a smoother, less error prone experience for the manufacturing process. By utilizing real-time communication between the new system and the existing systems, we can hopefully address the pain points and fill in gaps that that previously existed.

---

_Note: Focus on the key aspects of your design due to time constraints. Be clear and concise in your explanations._

---
