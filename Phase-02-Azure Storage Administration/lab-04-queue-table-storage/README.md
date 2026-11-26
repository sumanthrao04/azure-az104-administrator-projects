# **AZ-104 — Phase 02 | Lab 04: Azure Queue & Table Storage**

**Project: Asynchronous Application Communication & NoSQL Data Management**

In the previous labs, we covered Storage account security, Blob Storage, and Azure Files.

In this lab, we'll learn how enterprise applications use **Azure Queue Storage for asynchronous communication** and **Azure Table Storage for structured NoSQL data**.

We'll simulate an order-processing application without deploying application servers or virtual machines.

Duration

**2–3 hours**

Additional cost target

**Under $1**

Region

**Central India**

Subscription

**Azure Free Trial**

**Resource group:** `rg-az104-storage-cin-001`

**Storage account:** Reuse the StorageV2 account from Lab 01.

Cost is a planning estimate. We'll use a small number of messages and entities without deploying additional compute resources.

## **1. Real-world scenario**

Imagine you're working as an Azure Cloud Engineer for an e-commerce company.

Whenever a customer places an order, the application needs to perform several operations:

- Receive the customer order.
- Process the order asynchronously.
- Store order details.
- Track order processing status.
- Handle temporary application failures.
- Retry failed processing operations.

Instead of making the customer wait until every operation is completed, the application sends an order message to Azure Queue Storage.

A background worker processes the message and stores the order details in Azure Table Storage.

In this lab, we'll simulate the frontend application and background worker using Azure CLI commands.

## **2. Architecture — What we're building**

The frontend and background worker are simulated using CLI commands. No actual application servers will be deployed.

## **3. Topics we'll learn**

| Topic               | Description                                          |
| ------------------- | ---------------------------------------------------- |
| Azure Queue Storage | Asynchronous message storage                         |
| Message lifecycle   | Send, peek, receive, process and delete              |
| Visibility timeout  | Temporarily hides received messages                  |
| Message TTL         | Determines how long a message remains in the queue   |
| Dequeue count       | Number of times a message has been retrieved         |
| Pop receipt         | Required for deleting or updating a received message |
| Azure Table Storage | Schemaless NoSQL key-value storage                   |
| PartitionKey        | Groups related entities                              |
| RowKey              | Uniquely identifies an entity within a partition     |
| CRUD operations     | Create, Read, Update and Delete                      |
| Azure RBAC          | Queue and Table data-plane permissions               |

# **4. Azure Queue Storage fundamentals**

Azure Queue Storage allows applications to communicate asynchronously without requiring the producer and consumer to be available simultaneously.

For example, if a background worker is temporarily unavailable, messages can remain in the queue until a worker retrieves them or they expire.

### **Message processing lifecycle**

Receiving a message does not delete it. If processing fails and the message isn't deleted, it can become visible again after the visibility timeout.

### **Important Queue Storage properties**

| Property                           | Description                   |
| ---------------------------------- | ----------------------------- |
| Maximum message size               | 64 KiB                        |
| Default message TTL                | 7 days                        |
| Default receive visibility timeout | 30 seconds                    |
| Delivery behavior                  | At-least-once delivery        |
| Message ordering                   | Strict FIFO is not guaranteed |
| Protocol                           | HTTPS / REST API              |

Because messages may be delivered more than once, applications should implement **idempotent processing** to prevent duplicate business operations.

# **5. Hands-on implementation**

## **Task 1 — Verify the existing storage account**

Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com) and launch Cloud Shell in Bash mode.

```
RG_NAME="rg-az104-storage-cin-001"
STORAGE_NAME="<your-storage-account-name>"
QUEUE_NAME="order-processing-queue"
TABLE_NAME="Orders"
```

Replace the storage account placeholder with your actual account name.

```
az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query "{Name:name,Kind:kind,SKU:sku.name}" \
  -o table
```

Expected:

```
Kind        SKU
----------  ------------
StorageV2   Standard_LRS
```

**Networking reminder:** If your storage firewall allows only your laptop's public IP, execute data-plane commands using Azure CLI on that laptop rather than Cloud Shell.

## **Task 2 — Configure Azure RBAC permissions**

Your existing Blob and Azure Files roles do not automatically grant access to Queue and Table data.

Navigate to:

**Storage account → Access control (IAM) → Add role assignment**

Reuse the storage security groups from Lab 01.

| Security group          | Additional role                |
| ----------------------- | ------------------------------ |
| `az104-storage-admins`  | Storage Queue Data Contributor |
| `az104-storage-admins`  | Storage Table Data Contributor |
| `az104-storage-readers` | Storage Queue Data Reader      |
| `az104-storage-readers` | Storage Table Data Reader      |

For the core lab, ensure your executing identity has the necessary Contributor data roles. Subscription Owner access alone does not automatically grant these data-plane permissions when using Entra authentication.

### **Validate role assignments**

```
STORAGE_SCOPE=$(az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query id -o tsv)
```

```
az role assignment list \
  --scope "$STORAGE_SCOPE" \
  --include-inherited \
  --query "[].{Role:roleDefinitionName,Principal:principalName}" \
  -o table
```

Allow time for role assignments to propagate.

**Screenshot:** `01-queue-table-rbac.png`

## **Task 3 — Create Azure Queue Storage**

Navigate to:

**Storage account → Data storage → Queues → + Queue**

Configure:

| Field      | Value                         |
| ---------- | ----------------------------- |
| Queue name | `order-processing-queue`      |
| Purpose    | Asynchronous order processing |

### **Azure CLI**

```
az storage queue create \
  --account-name "$STORAGE_NAME" \
  --name "$QUEUE_NAME" \
  --auth-mode login
```

### **List queues**

```
az storage queue list \
  --account-name "$STORAGE_NAME" \
  --auth-mode login \
  -o table
```

Expected: `order-processing-queue` appears in the output.

**Screenshot:** `02-queue-created.png`

## **Task 4 — Send messages to the queue**

We'll simulate three customers placing orders.

### **Message 1**

```
az storage message put \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --content '{"orderId":"ORD001","product":"Laptop","quantity":1}' \
  --auth-mode login
```

### **Message 2**

```
az storage message put \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --content '{"orderId":"ORD002","product":"Monitor","quantity":2}' \
  --auth-mode login
```

### **Message 3**

```
az storage message put \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --content '{"orderId":"ORD003","product":"Keyboard","quantity":1}' \
  --auth-mode login
```

### **Validate messages**

Open:

**Storage account → Queues → order-processing-queue**

Inspect the available messages.

**order-processing-queue**

Illustrative queue contents

**ORD001**

Laptop · Quantity: 1

Pending

**ORD002**

Monitor · Quantity: 2

Pending

**ORD003**

Keyboard · Quantity: 1

Pending

**Screenshot:** `03-queue-messages.png`

## **Task 5 — Peek, receive and delete messages**

This is the most important part of Queue Storage.

### **Step 1 — Peek at messages**

```
az storage message peek \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --num-messages 3 \
  --auth-mode login \
  -o json
```

**What does peek do?**

It reads visible messages without making them invisible or deleting them.

### **Step 2 — Receive a message**

```
az storage message get \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --num-messages 1 \
  --visibility-timeout 60 \
  --auth-mode login \
  -o json
```

The response contains information such as:

| Property          | Purpose                                |
| ----------------- | -------------------------------------- |
| `id`              | Unique message identifier              |
| `content`         | Message payload                        |
| `popReceipt`      | Receipt required for message deletion  |
| `dequeueCount`    | Number of retrieval attempts           |
| `timeNextVisible` | When the message becomes visible again |

The message becomes temporarily invisible for 60 seconds.

### **Step 3 — Understand visibility timeout**

### **Interactive Queue Processing Simulation**

ReceiveSuccessFailure

**ORD001**



Invisible



Worker receives the message. Other workers temporarily cannot retrieve it.

Visibility timeout: 60 seconds in this example

Conceptual simulation, not connected to your Azure account.

### **Step 4 — Delete a processed message**

Receive a message and capture its current ID and pop receipt:

```
MESSAGE=$(az storage message get \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --num-messages 1 \
  --visibility-timeout 120 \
  --auth-mode login \
  -o json)
```

```
MESSAGE_ID=$(echo "$MESSAGE" | jq -r '.[0].id')

POP_RECEIPT=$(echo "$MESSAGE" | jq -r '.[0].popReceipt')
```

Verify that a message was actually returned before continuing. If the queue is empty or all messages are invisible, wait and retry.

After simulating successful processing, delete it before the visibility timeout expires:

```
az storage message delete \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --id "$MESSAGE_ID" \
  --pop-receipt "$POP_RECEIPT" \
  --auth-mode login
```

### **Peek vs Get vs Delete**

| Operation | Reads message | Hides message | Removes message |
| --------- | ------------- | ------------- | --------------- |
| Peek      | Yes           | No            | No              |
| Get       | Yes           | Temporarily   | No              |
| Delete    | No            | —             | Yes             |

**Screenshot:** `04-message-processing.png`

## **Task 6 — Simulate a worker failure and retry**

**Real-world scenario:** The background worker receives an order, but the application crashes before completing the operation.

Receive a message:

```
az storage message get \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --num-messages 1 \
  --visibility-timeout 30 \
  --auth-mode login \
  -o json
```

Do not delete the message.

Wait for 35 seconds:

```
sleep 35
```

Retrieve messages again:

```
az storage message get \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --num-messages 3 \
  --visibility-timeout 30 \
  --auth-mode login \
  --query "[].{ID:id,Content:content,DequeueCount:dequeueCount}" \
  -o table
```

Compare the message IDs and dequeue counts to identify the retried message.

### **Expected outcome**

The unprocessed message can become visible again and be retrieved by another worker.

**Interview takeaway:** Azure Queue Storage supports at-least-once delivery. Applications must handle duplicate messages and implement retry logic. Repeatedly failing messages can be moved to an application-managed poison queue.

**Screenshot:** `05-message-retry-validation.png`

# **6. Azure Table Storage fundamentals**

Azure Table Storage is a schemaless NoSQL data store used for storing structured, non-relational data.

Common use cases include application metadata, device information, inventory records, and order-processing status.

### **Table Storage hierarchy**

```
Storage Account
    |
    └── Table: Orders
          |
          ├── Entity: ORD001
          ├── Entity: ORD002
          └── Entity: ORD003
```

### **Important entity properties**

| Property          | Description                                |
| ----------------- | ------------------------------------------ |
| PartitionKey      | Groups related entities                    |
| RowKey            | Unique identifier within a partition       |
| Timestamp         | System-managed last modification timestamp |
| Custom properties | Application-specific attributes            |

The combination of **PartitionKey + RowKey** uniquely identifies an entity.

### **Azure Table Storage vs Azure SQL**

| Feature       | Table Storage                | Azure SQL                    |
| ------------- | ---------------------------- | ---------------------------- |
| Database type | NoSQL                        | Relational                   |
| Schema        | Flexible                     | Defined                      |
| Query model   | Key and property filters     | SQL                          |
| Joins         | Not supported                | Supported                    |
| Relationships | Application-managed          | Relational constraints       |
| Typical use   | Metadata and simple entities | Complex relational workloads |

## **Task 7 — Create an Azure Table**

Navigate to:

**Storage account → Data storage → Tables → + Table**

Enter:

`Orders`

### **Azure CLI**

```
az storage table create \
  --account-name "$STORAGE_NAME" \
  --name "$TABLE_NAME" \
  --auth-mode login
```

### **Verify**

```
az storage table list \
  --account-name "$STORAGE_NAME" \
  --auth-mode login \
  -o table
```

**Screenshot:** `06-table-created.png`

## **Task 8 — Insert entities into Table Storage**

We'll store the three sample orders.

### **Insert Order 001**

```
az storage entity insert \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --entity PartitionKey=Orders \
           RowKey=ORD001 \
           Product=Laptop \
           Quantity=1 \
           Status=Pending \
  --auth-mode login
```

### **Insert Order 002**

```
az storage entity insert \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --entity PartitionKey=Orders \
           RowKey=ORD002 \
           Product=Monitor \
           Quantity=2 \
           Status=Pending \
  --auth-mode login
```

### **Insert Order 003**

```
az storage entity insert \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --entity PartitionKey=Orders \
           RowKey=ORD003 \
           Product=Keyboard \
           Quantity=1 \
           Status=Pending \
  --auth-mode login
```

### **Expected table contents**

**Orders**

3 entities

| **PartitionKey** | **RowKey** | **Product** | **Qty** | **Status** |
| ---------------- | ---------- | ----------- | ------- | ---------- |
| Orders           | ORD001     | Laptop      | 1       | Pending    |
| Orders           | ORD002     | Monitor     | 2       | Pending    |
| Orders           | ORD003     | Keyboard    | 1       | Pending    |

**Screenshot:** `07-table-entities.png`

## **Task 9 — Perform CRUD operations**

CRUD stands for Create, Read, Update and Delete.

### **Read all entities**

```
az storage entity query \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --auth-mode login \
  -o json
```

### **Read a specific order**

```
az storage entity show \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --partition-key Orders \
  --row-key ORD001 \
  --auth-mode login \
  -o json
```

### **Filter pending orders**

```
az storage entity query \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --filter "PartitionKey eq 'Orders' and Status eq 'Pending'" \
  --auth-mode login \
  -o json
```

### **Update order status**

We'll simulate successful processing of Order 001.

```
az storage entity merge \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --entity PartitionKey=Orders \
           RowKey=ORD001 \
           Status=Processed \
  --auth-mode login
```

### **Verify the update**

```
az storage entity show \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --partition-key Orders \
  --row-key ORD001 \
  --auth-mode login \
  --query Status \
  -o tsv
```

Expected:

```
Processed
```

### **Delete an entity**

Delete only the synthetic Order 003 record.

```
az storage entity delete \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --partition-key Orders \
  --row-key ORD003 \
  --auth-mode login
```

### **Merge vs Replace**

| Merge                               | Replace                                      |
| ----------------------------------- | -------------------------------------------- |
| Updates specified properties        | Replaces the entity's custom properties      |
| Preserves other existing properties | Unspecified custom properties may be removed |
| Suitable for partial updates        | Suitable for full entity replacement         |

**Screenshot:** `08-table-crud-validation.png`

## **Task 10 — Simulate an end-to-end order-processing workflow**

Now we'll combine Queue Storage and Table Storage.

### **Order Processing Workflow**

1. **Customer places order**

   Producer sends order details to Queue Storage.
2. **Worker receives message**

   Message becomes temporarily invisible.
3. **Worker processes order**

   Application validates and processes the order.
4. **Update Table Storage**

   Order status changes to Processed.
5. **Delete queue message**

   Worker acknowledges successful processing.

### **Create a dedicated workflow queue**

We'll use a separate queue so earlier test messages don't interfere.

```
WORKFLOW_QUEUE="order-workflow-demo"

az storage queue create \
  --account-name "$STORAGE_NAME" \
  --name "$WORKFLOW_QUEUE" \
  --auth-mode login
```

### **Send a new order**

```
az storage message put \
  --account-name "$STORAGE_NAME" \
  --queue-name "$WORKFLOW_QUEUE" \
  --content '{"orderId":"ORD004","product":"Mouse","quantity":2}' \
  --auth-mode login
```

### **Create the worker script**

Create a file named `order-worker.sh`.

```

#!/usr/bin/env bash
set -euo pipefail

STORAGE_NAME="${1:?Provide storage account name}"
QUEUE_NAME="order-workflow-demo"
TABLE_NAME="Orders"

echo "Checking queue for messages..."

MESSAGE=$(az storage message get \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --num-messages 1 \
  --visibility-timeout 120 \
  --auth-mode login \
  -o json)

if [ "$(echo "$MESSAGE" | jq 'length')" -eq 0 ]; then
  echo "No messages available."
  exit 0
fi

MESSAGE_ID=$(echo "$MESSAGE" | jq -r '.[0].id')
POP_RECEIPT=$(echo "$MESSAGE" | jq -r '.[0].popReceipt')
CONTENT=$(echo "$MESSAGE" | jq -r '.[0].content')

ORDER_ID=$(echo "$CONTENT" | jq -er '.orderId')
PRODUCT=$(echo "$CONTENT" | jq -er '.product')
QUANTITY=$(echo "$CONTENT" | jq -er '.quantity')

echo "Processing order: $ORDER_ID"

az storage entity insert \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --entity PartitionKey=Orders \
           RowKey="$ORDER_ID" \
           Product="$PRODUCT" \
           Quantity="$QUANTITY" \
           Status=Processed \
  --if-exists merge \
  --auth-mode login \
  -o none

echo "Order stored successfully."

az storage message delete \
  --account-name "$STORAGE_NAME" \
  --queue-name "$QUEUE_NAME" \
  --id "$MESSAGE_ID" \
  --pop-receipt "$POP_RECEIPT" \
  --auth-mode login

echo "Queue message deleted."
echo "Order processing completed."

```

### **Execute the script**

```
chmod +x order-worker.sh
```

```
./order-worker.sh "$STORAGE_NAME"
```

### **Expected output**

```
Checking queue for messages...
Processing order: ORD004
Order stored successfully.
Queue message deleted.
Order processing completed.
```

### **Validate the processed order**

```
az storage entity show \
  --account-name "$STORAGE_NAME" \
  --table-name "$TABLE_NAME" \
  --partition-key Orders \
  --row-key ORD004 \
  --auth-mode login \
  -o json
```

Expected status: `Processed`.

### **Why is this script important?**

It demonstrates several real-world Cloud and DevOps concepts:

- Asynchronous application communication.
- Message-driven architecture.
- JSON parsing using `jq`.
- NoSQL entity management.
- Azure Entra authentication without hard-coded storage keys.
- Failure handling using Bash.
- Basic idempotent storage updates using the order ID.

The script deletes the queue message only after the table write succeeds. If the script fails earlier, the message can be retried after its visibility timeout.

**Production limitation:** This is an educational worker, not a production-ready application. A production worker would need stronger input validation, retry limits, visibility renewal, poison-message handling, concurrency controls, and monitoring.

**Screenshot:** `09-end-to-end-order-processing.png`

# **7. Azure Queue Storage vs Azure Service Bus**

This is a frequently asked Azure interview topic.

| Feature             | Queue Storage                               | Azure Service Bus                  |
| ------------------- | ------------------------------------------- | ---------------------------------- |
| Purpose             | Simple asynchronous messaging               | Advanced enterprise messaging      |
| Strict FIFO         | Not guaranteed                              | Supported using sessions           |
| Dead-letter queue   | Application-managed                         | Built-in                           |
| Duplicate detection | Application-managed                         | Supported in applicable tiers      |
| Publish/subscribe   | No native topics                            | Topics and subscriptions           |
| Transactions        | No comparable messaging transaction support | Supported                          |
| Complexity          | Lower                                       | Higher                             |
| Use case            | Background tasks and simple workloads       | Enterprise application integration |

**When to choose Queue Storage:** Simple background processing, cost-sensitive workloads, and basic asynchronous communication.

**When to choose Service Bus:** Enterprise messaging requiring advanced delivery guarantees, sessions, dead-lettering, or publish/subscribe.

Service Bus is an additional interview topic; we won't deploy it in this lab.

# **8. Storage monitoring**

Navigate to:

**Storage account → Monitoring → Metrics**

Explore the Queue and Table service metric namespaces.

| Metric                 | Purpose                           |
| ---------------------- | --------------------------------- |
| Transactions           | Number of storage requests        |
| Availability           | Service availability              |
| Success E2E Latency    | End-to-end request latency        |
| Success Server Latency | Server-side processing latency    |
| Queue Message Count    | Approximate queued message volume |
| Table Entity Count     | Number of stored entities         |

Some capacity and count metrics are collected less frequently than transaction metrics, so results may not appear immediately.

We'll configure more detailed monitoring and alerts in Lab 05.

**Screenshot:** `10-storage-metrics.png`

# **9. Real-world troubleshooting scenarios**

| Problem                        | Possible cause                    | Resolution                                         |
| ------------------------------ | --------------------------------- | -------------------------------------------------- |
| Queue access returns 403       | Missing Queue data role           | Verify RBAC                                        |
| Table access denied            | Missing Table data role           | Assign appropriate data permissions                |
| Message disappears temporarily | Visibility timeout                | Wait for timeout or complete processing            |
| Duplicate message processing   | At-least-once delivery            | Implement idempotency                              |
| Message deletion fails         | Invalid or outdated pop receipt   | Use the latest receipt                             |
| Entity already exists          | Duplicate PartitionKey and RowKey | Use an appropriate update or upsert operation      |
| Table query is slow            | Inefficient partition design      | Query using PartitionKey and RowKey where possible |
| Storage commands fail          | Firewall restrictions             | Check permitted client network                     |
| Queue backlog increases        | Workers cannot keep up            | Investigate processing rate and failures           |

## **Production interview scenario**

**Incident:** An application has 10,000 pending messages in Azure Queue Storage, and the number continues increasing.

How would you troubleshoot?

1. Check queue message count and transaction metrics.
2. Verify whether producer traffic has increased.
3. Check whether background workers are running.
4. Investigate worker errors and processing latency.
5. Review authentication and storage firewall restrictions.
6. Inspect visibility timeout and dequeue counts.
7. Identify repeatedly failing messages.
8. Scale workers if processing capacity is insufficient.
9. Implement idempotency and appropriate retry handling.
10. Configure backlog alerts and monitor recovery.

# **10. GitHub repository structure**

```
phase-02-storage/
│
└── lab-04-queue-table-storage/
    ├── README.md
    └── screenshots/
        ├── 01-queue-table-rbac.png
        ├── 02-queue-created.png
        ├── 03-queue-messages.png
        ├── 04-message-processing.png
        ├── 05-message-retry-validation.png
        ├── 06-table-created.png
        ├── 07-table-entities.png
```

### **Git commit**

After completing and validating the lab:

```
git add phase-02-storage/lab-04-queue-table-storage/

git commit -m "feat(az104): implement Azure Queue and Table Storage with asynchronous order processing"

git push origin main
```

# **11. AZ-104 & DevOps interview questions**

1\. What is Azure Queue Storage?

2\. What is the maximum Azure Queue message size?

3\. What is visibility timeout?

4\. What is a pop receipt?

5\. Does receiving a queue message delete it?

6\. Does Azure Queue Storage guarantee FIFO ordering?

7\. What is Azure Table Storage?

8\. What are PartitionKey and RowKey?

9\. What is the difference between Queue Storage and Service Bus?

10\. What is idempotency in message processing?

11\. What is a poison message?

12\. How would you troubleshoot a growing queue backlog?

# **12. Cleanup and cost management**

After capturing your screenshots, delete the temporary queues and table if they are no longer required.

These commands permanently remove the corresponding lab data. Verify that they contain only your synthetic test records.

```
az storage queue delete \
  --account-name "$STORAGE_NAME" \
  --name order-processing-queue \
  --auth-mode login
```

```
az storage queue delete \
  --account-name "$STORAGE_NAME" \
  --name order-workflow-demo \
  --auth-mode login
```

```
az storage table delete \
  --account-name "$STORAGE_NAME" \
  --name Orders \
  --auth-mode login
```

Review your remaining trial credit and expiration date.

**Do not delete the storage account.** We still need it for Lab 05.

# **13. Lab 04 completion checklist**

**Lab 04 Progress**



Verified existing StorageV2 account

Configured Queue and Table data-plane RBAC

Created order-processing-queue

Sent sample order messages

Tested Peek and Get operations

Understood visibility timeout and pop receipt

Deleted a processed message

Simulated worker failure and message retry

Created Orders table

Inserted and queried entities

Performed CRUD operations

Created and executed order-worker.sh

Validated end-to-end order processing

Explored Queue and Table metrics

Completed cleanup and GitHub documentation
