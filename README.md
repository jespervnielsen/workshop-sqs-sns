# SNS + SQS + Lambda Workshop

This workshop provides hands-on experience with Amazon SNS, SQS, and Lambda.
We use a pre-built CloudFormation stack that sets up a simple flow:

**Producer Lambda → SNS → SQS → Consumer Lambda**

The goal is to explore different scenarios and gain a deeper understanding of how SNS and SQS handle messages in practice.

---

## Exercises

### Exercise 1 – Send many messages
- **Step:** Modify the producer code to send **1000 messages in a loop**.
- **Observation:** Check **CloudWatch (Lambda & SQS metrics)** to see:
  - SNS handles all messages without blocking
  - SQS queue fills up and drains
  - Consumer Lambda scales out with multiple invocations
- *(Extra: Add `await new Promise(r => setTimeout(r, 500))` in the consumer to simulate a slow API call and watch the queue grow)*

---

### Exercise 2 – Add a custom attribute
- **Step:** Add a custom attribute in the producer:
  ```js
  MessageAttributes: {
    itemId: { DataType: "String", StringValue: `xyz-${i}` }
  }
  ```

### Exercise 2 – Read custom attribute
- **Step:** Read the attribute in the consumer Lambda and log it.
- **Observation:** Check **CloudWatch Logs** → the metadata follows each message.

---

### Exercise 3 – Limit concurrency
- **Step:** In the Lambda consumer → Configuration → Concurrency → set **Reserved concurrency = 2**.
- **Observation:**
  - Messages pile up in SQS since only 2 invocations can run at once
  - **CloudWatch metrics** show queue length and processing time increasing

---

### Exercise 4 – Throttling
- **Step:** Enable **"Throttle"** in the consumer Lambda configuration.
- **Observation:**
  - Messages stop being processed → the queue grows
  - Once throttling is removed, the consumer starts draining the queue again
- **Learning goal:** Understand how throttling protects downstream systems and how SQS acts as a buffer.

---

### Exercise 5 – Multiple subscribers
- **Step:** Create an additional SQS queue and subscribe it to the same SNS topic.
- **Step:** Deploy a simple Lambda that logs messages from this new queue.
- **Observation:** Each message is delivered to **all subscribers** (fan-out).
- **Learning goal:** Understand the publish/subscribe model with SNS.

---

### Exercise 6 – Dead-letter Queue (DLQ)
- **Step:** Create a DLQ and attach it to the consumer SQS.
- **Step:** Simulate failures in the consumer Lambda (e.g. `throw new Error("Simulated failure")`).
- **Observation:** After multiple retries, failed messages are moved to the DLQ.
- **Learning goal:** Learn how to handle failures without losing messages.

---

### Tips
- Use **CloudWatch Metrics** (SQS and Lambda) to track throughput, queue length, and errors.
- Inspect **CloudWatch Logs** to view message payloads and attributes.
- If time allows, experiment with both **Standard** and **FIFO queues**.