### **Tables Overview**

1. **OrderRequisition**
2. **RequisitionItem**

---

### **1. OrderRequisition**

- **Description**: This represents the requisition attached to a work order
- **Columns**:
  - `RequisitionID`(PK): Unique identifier for the requisition
  - `WorkOrderID` (FK): Refers to the work order that this requisition is based off of, in `WorkOrders.WorkOrderID`
  - `UnitID` (FK): Refers to the work order unit that this requisition is generated from, in `WorkOrderUnits.UnitID`
  - `Status`: Status of the requisition (e.g. 'Draft','Requested', 'In Progress', 'Partially Fulfilled', 'Fulfilled', 'Cancelled')

### **2. RequisitionItem**

- **Description**: Represents an item stock that is requested in as part of an requisition
- **Columns**:
  - `RequisitionItemID` (PK): Unique identifier for the requisition item
  - `RequisitionID` (FK): The requisition that this item is a part of, referencing `OrderRequisition.RequisitionID`
  - `StockID` (FK): The item stock that this requisition item is based on, referencing `InventoryStocks.StockID`
  - `OperationID` (FK): The operation that this item is required for
  - `AmountRequired`: The amount of this item that is requested in the requisition
  - `AmountFilled`: The amount of this item that has been fulfilled by warehouse tech
  - `Status`: The status of this requisition item (`Requested`, `Reserved`, `Retrieving`, `Partially Retrieved`, `Retrieved`, `Delivering`, `Delivered`, `In Operation`, `Cancelled`)
  - `LastActionBy` (FK): The ID of the last person that performed an action on this item, refers to `RequisitionUser.UserID`

### **2. RequisitionUser**

- **Description**: Represents a user of the Requisition System
- **Columns**:
  - `UserID` (PK): Unique identifier for the user
  - `Role`: The role of the user (e.g. 'CAM Programmer', `Manufacturing Engineer`, `Warehouse Technician`)
  - `LocationID`(FK): This should ideally refer to a warehouse or location ID, which is typically used in the IMS InventoryLocation table.

---

### **Entity Relationships**

- **OrderRequisition** ➔ **WorkOrders**: One-to-One
  - One order requisition should be based off of one work order
- **OrderRequisition** ➔ **WorkOrderUnits**: One-to-One
  - One order requisition should be generated by one work order unit
- **OrderRequisition** ➔ **RequisitionItem**: One-to-Many
  - An order requisition can have many requisition items that are being requested

---

### **Simplified SQL Schema**

```sql
-- Table: OrderRequisition
CREATE TABLE OrderRequisition (
RequisitionID INT PRIMARY KEY
FOREIGN KEY (WorkOrderID) REFERENCES WorkOrders(WorkOrderID)
FOREIGN KEY (UnitID) REFERENCES WorkOrderUnits(UnitID)
Status VARCHAR(20) NOT NULL
)

-- Table: RequisitionItem
CREATE TABLE RequisitionItem (
RequisitionItemID INT PRIMARY KEY
FOREIGN KEY (RequisitionID) REFERENCES OrderRequisition(RequisitionID)
FOREIGN KEY (StockID) REGERENCES InventoryStocks(StockID)
FOREIGN KEY (OperationID) REFERENCES Operations(OperationID)
AmountRequired
AmountFilled
Status
)
```
