Durable Functions is an extension of Azure Functions that lets you write stateful functions in a serverless compute environment. It enables you to define stateful workflows using an orchestrator function and stateful entities using entity functions.

#### Key Components:

1. **Orchestrator Functions**
    
    - Define the workflow steps and their order
    - Maintain state across function executions
    - Handle error conditions and compensation logic
    - Written using a special deterministic syntax
2. **Activity Functions**
    
    - Perform individual workflow steps
    - Can be long-running
    - May have side effects (I/O, random values, etc.)
    - Called from orchestrators
3. **Entity Functions**
    
    - Manage state for entities
    - Support for transactions
    - Concurrent operations
    - Called from orchestrators or directly
4. **Client Functions**
    
    - Start orchestrations or entity operations
    - Query status or terminate orchestrations
    - Send external events to running orchestrations

#### Durable Function Patterns:

##### 1. Function Chaining

- Execute a sequence of functions in a specific order
- Output of one function becomes input to the next
- Full workflow history maintained

```csharp
[FunctionName("SequentialWorkflow")]
public static async Task<object> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    try
    {
        // Get input
        var input = context.GetInput<string>();
        
        // Call activities in sequence
        var output1 = await context.CallActivityAsync<string>("Activity1", input);
        var output2 = await context.CallActivityAsync<string>("Activity2", output1);
        var output3 = await context.CallActivityAsync<string>("Activity3", output2);
        
        // Return final result
        return output3;
    }
    catch (Exception ex)
    {
        // Error handling/compensation
        await context.CallActivityAsync("CleanupActivity", null);
        throw;
    }
}

[FunctionName("Activity1")]
public static string Activity1([ActivityTrigger] string input) => $"Activity1 processed: {input}";

[FunctionName("Activity2")]
public static string Activity2([ActivityTrigger] string input) => $"Activity2 processed: {input}";

[FunctionName("Activity3")]
public static string Activity3([ActivityTrigger] string input) => $"Activity3 processed: {input}";
```

##### 2. Fan-Out/Fan-In

- Parallel execution of multiple functions
- Aggregate results when all functions complete
- Dynamic parallelism based on data

```csharp
[FunctionName("ParallelWorkflow")]
public static async Task<List<string>> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var parallelTasks = new List<Task<string>>();
    
    // Get data items to process
    string[] items = await context.CallActivityAsync<string[]>("GetWorkItems", null);
    
    // Fan out - create a task for each item
    foreach (string item in items)
    {
        Task<string> task = context.CallActivityAsync<string>("ProcessItem", item);
        parallelTasks.Add(task);
    }
    
    // Fan in - wait for all tasks to complete
    string[] results = await Task.WhenAll(parallelTasks);
    
    // Process aggregated results
    return results.ToList();
}

[FunctionName("GetWorkItems")]
public static string[] GetWorkItems([ActivityTrigger] object input)
{
    // Return items to process in parallel
    return new string[] { "item1", "item2", "item3", "item4", "item5" };
}

[FunctionName("ProcessItem")]
public static string ProcessItem([ActivityTrigger] string item)
{
    // Process individual item
    return $"Processed: {item}";
}
```

##### 3. Async HTTP API

- Long-running operations over HTTP
- Return immediate response with status URL
- Polling or webhook for completion notification

```csharp
[FunctionName("LongRunningOperation")]
public static async Task<IActionResult> RunHttpStart(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")] HttpRequest req,
    [DurableClient] IDurableOrchestrationClient starter,
    ILogger log)
{
    // Parse request
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    
    // Start orchestration
    string instanceId = await starter.StartNewAsync("LongRunningOrchestrator", null, requestBody);
    
    // Return status endpoints that can be used to monitor status
    return starter.CreateCheckStatusResponse(req, instanceId);
}

[FunctionName("LongRunningOrchestrator")]
public static async Task<object> LongRunningOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    // Get input
    var input = context.GetInput<string>();
    
    // Simulate long-running process with multiple steps
    await context.CallActivityAsync("Step1", input);
    await context.CallActivityAsync("Step2", null);
    await context.CallActivityAsync("Step3", null);
    
    return new { Result = "Completed successfully" };
}
```

##### 4. Monitor Pattern

- Recurring process that monitors a condition
- Continues until condition is met or timeout
- Implements polling logic with customizable frequency

```csharp
[FunctionName("JobStatusMonitor")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    // Get job ID to monitor
    string jobId = context.GetInput<string>();
    
    // Define timeout and polling interval
    DateTime expiryTime = context.CurrentUtcDateTime.AddHours(2);
    TimeSpan pollingInterval = TimeSpan.FromMinutes(5);
    
    // Monitor until job completes or timeout
    while (context.CurrentUtcDateTime < expiryTime)
    {
        // Check job status
        bool jobCompleted = await context.CallActivityAsync<bool>("CheckJobStatus", jobId);
        
        if (jobCompleted)
        {
            // Job completed successfully, send notification
            await context.CallActivityAsync("SendCompletionNotification", jobId);
            return;
        }
        
        // Not complete yet, schedule next check
        DateTime nextCheck = context.CurrentUtcDateTime.Add(pollingInterval);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }
    
    // Timeout reached, send timeout notification
    await context.CallActivityAsync("SendTimeoutNotification", jobId);
}

[FunctionName("CheckJobStatus")]
public static bool CheckJobStatus([ActivityTrigger] string jobId)
{
    // Check job status logic here
    // Return true if job is completed
    return false; // Simulated "not yet complete"
}
```

##### 5. Human Interaction

- Workflow with human approval steps
- Wait for external input
- Timeout and reminder logic

```csharp
[FunctionName("ApprovalWorkflow")]
public static async Task<object> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    // Get request details
    var requestData = context.GetInput<RequestData>();
    
    // Send approval request
    await context.CallActivityAsync("RequestApproval", requestData);
    
    // Create a durable timer for timeout
    DateTime deadline = context.CurrentUtcDateTime.AddHours(24);
    Task timeoutTask = context.CreateTimer(deadline, CancellationToken.None);
    
    // Wait for approval or timeout
    Task<bool> approvalTask = context.WaitForExternalEvent<bool>("ApprovalEvent");
    
    // Also set up reminders
    bool reminderSent = false;
    DateTime reminderTime = context.CurrentUtcDateTime.AddHours(8);
    Task reminderTask = context.CreateTimer(reminderTime, CancellationToken.None);
    
    // Process approval, timeout, or send reminder
    while (!timeoutTask.IsCompleted && !approvalTask.IsCompleted)
    {
        Task winner = await Task.WhenAny(approvalTask, timeoutTask, reminderTask);
        
        if (winner == reminderTask && !reminderSent)
        {
            // Send reminder
            await context.CallActivityAsync("SendReminder", requestData);
            reminderSent = true;
            reminderTask = context.CreateTimer(context.CurrentUtcDateTime.AddHours(8), CancellationToken.None);
        }
    }
    
    if (approvalTask.IsCompleted)
    {
        // Process based on approval result
        bool approved = approvalTask.Result;
        if (approved)
        {
            await context.CallActivityAsync("ProcessApproved", requestData);
            return new { Result = "Approved" };
        }
        else
        {
            await context.CallActivityAsync("ProcessRejected", requestData);
            return new { Result = "Rejected" };
        }
    }
    else
    {
        // Timeout occurred
        await context.CallActivityAsync("ProcessTimeout", requestData);
        return new { Result = "Timeout" };
    }
}
```

#### Entity Functions

Entity functions provide a way to manage state for small pieces of data while allowing for reliable, concurrent operations on that data.

```csharp
// Entity definition
public class Counter
{
    [JsonProperty("value")]
    public int CurrentValue { get; set; }

    public void Add(int amount) => CurrentValue += amount;
    public void Reset() => CurrentValue = 0;
    public int Get() => CurrentValue;
}

// Entity function
[FunctionName("Counter")]
public static Task Run([EntityTrigger] IDurableEntityContext ctx)
{
    return ctx.DispatchAsync<Counter>();
}

// Client function to interact with entity
[FunctionName("CounterClient")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")] HttpRequest req,
    [DurableClient] IDurableEntityClient client)
{
    var entityId = new EntityId("Counter", "myCounter");
    
    // Perform operations on entity
    await client.SignalEntityAsync(entityId, "Add", 1);
    
    // Read current state
    var state = await client.ReadEntityStateAsync<Counter>(entityId);
    
    return new OkObjectResult(state);
}
```

#### Advanced Durable Functions Features:

##### 1. Sub-Orchestrations

- Orchestrators calling other orchestrators
- Hierarchical workflow composition
- Reusable workflow components

```csharp
[FunctionName("ParentOrchestrator")]
public static async Task<object> RunParentOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    // Call multiple child orchestrations
    Task<object> child1 = context.CallSubOrchestratorAsync<object>("ChildOrchestrator", "Input1");
    Task<object> child2 = context.CallSubOrchestratorAsync<object>("ChildOrchestrator", "Input2");
    
    // Wait for all to complete
    await Task.WhenAll(child1, child2);
    
    return new { Result1 = child1.Result, Result2 = child2.Result };
}

[FunctionName("ChildOrchestrator")]
public static async Task<object> RunChildOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string input = context.GetInput<string>();
    
    // Child orchestration logic
    await context.CallActivityAsync("ChildActivity", input);
    
    return new { ProcessedBy = "ChildOrchestrator", Input = input };
}
```

##### 2. Eternal Orchestrations

- Never-ending workflows
- Periodic tasks
- Continuous monitoring

```csharp
[FunctionName("EternalOrchestrator")]
public static async Task RunEternalOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    // Get configuration
    var config = context.GetInput<ScheduleConfig>();
    
    // Loop "forever"
    while (true)
    {
        // Get current time in orchestrator
        DateTime nextRun = context.CurrentUtcDateTime.Add(config.Interval);
        
        // Run scheduled task
        await context.CallActivityAsync("ScheduledTask", null);
        
        // Wait until next scheduled time
        await context.CreateTimer(nextRun, CancellationToken.None);
    }
}
```

##### 3. Error Handling and Compensation

- Try/catch pattern for workflow error handling
- Compensation logic for rolling back partial work
- Retry policies for transient failures

```csharp
[FunctionName("ReliableWorkflow")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    try
    {
        // Step 1 - Reserve resources
        await context.CallActivityAsync("ReserveResources", null);
        
        try
        {
            // Step 2 - Perform work (might fail)
            await context.CallActivityWithRetryAsync("PerformWork", 
                new RetryOptions(TimeSpan.FromSeconds(5), 3), null);
            
            // Step 3 - Commit work
            await context.CallActivityAsync("CommitWork", null);
        }
        catch (Exception)
        {
            // Compensation - Release resources on failure
            await context.CallActivityAsync("ReleaseResources", null);
            throw;
        }
    }
    catch (Exception ex)
    {
        // Handle all unhandled exceptions
        await context.CallActivityAsync("LogFailure", ex.Message);
        throw;
    }
}
```

##### 4. External Events

- Receiving events from external sources
- Integration with other systems
- Long-running approvals

```csharp
[FunctionName("OrderProcessingOrchestrator")]
public static async Task<object> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    // Process order
    var order = context.GetInput<Order>();
    await context.CallActivityAsync("ProcessOrder", order);
    
    // Wait for payment (external event)
    using (var timeoutCts = new CancellationTokenSource())
    {
        DateTime expiration = context.CurrentUtcDateTime.AddDays(3);
        Task timeoutTask = context.CreateTimer(expiration, timeoutCts.Token);
        Task<PaymentInfo> paymentTask = context.WaitForExternalEvent<PaymentInfo>("PaymentReceived");
        
        Task winner = await Task.WhenAny(paymentTask, timeoutTask);
        if (winner == paymentTask)
        {
            // Cancel the timeout
            timeoutCts.Cancel();
            
            // Complete the order with payment info
            PaymentInfo payment = paymentTask.Result;
            await context.CallActivityAsync("CompleteOrder", new { Order = order, Payment = payment });
            return new { Status = "Completed", OrderId = order.Id };
        }
        else
        {
            // Timeout occurred
            await context.CallActivityAsync("CancelOrder", order);
            return new { Status = "Cancelled", OrderId = order.Id, Reason = "Payment timeout" };
        }
    }
}

// HTTP Endpoint to send payment event
[FunctionName("ReceivePayment")]
public static async Task<IActionResult> ReceivePayment(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    [DurableClient] IDurableOrchestrationClient client,
    ILogger log)
{
    // Get order ID and payment info
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    var data = JsonConvert.DeserializeObject<PaymentRequest>(requestBody);
    
    // Raise external event to the orchestration
    await client.RaiseEventAsync(data.OrderId, "PaymentReceived", data.PaymentInfo);
    
    return new OkResult();
}
```

#### Important Implementation Details:

##### 1. Determinism in Orchestrator Functions

- Orchestrator code must be deterministic (produce same output for same input)
    
- Cannot use:
    
    - Random numbers: `Random`, `Guid.NewGuid()`
    - Current date/time: `DateTime.Now`, `DateTime.UtcNow`
    - Environment variables that might change
    - Async I/O operations
- Must use:
    
    - `context.CurrentUtcDateTime` for time
    - Activity functions for non-deterministic operations
    - `IDurableOrchestrationContext` methods for timers

##### 2. Storage Providers

- Azure Storage (default)
- SQL Server (for higher throughput)
- Configuration in host.json

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "type": "AzureStorage"
      }
    }
  }
}
```

##### 3. Instance Management APIs

- Start new orchestration: `StartNewAsync`
- Query status: `GetStatusAsync`
- Terminate: `TerminateAsync`
- Purge: `PurgeInstanceHistoryAsync`
- Send events: `RaiseEventAsync`

##### 4. Versioning and Deployment

- Handle versioning carefully to avoid breaking running instances
- Use versioned orchestration names
- Consider using custom status for application state

#### Performance Considerations:

1. **Storage Performance**:
    
    - Durable Functions use storage for state
    - High-throughput scenarios may require SQL provider
    - Be mindful of storage transaction costs
2. **Replay Overhead**:
    
    - Orchestrator code replays on each resume
    - Keep orchestrator code light and simple
    - Move complex logic to activity functions
3. **Fan-out Scale Limits**:
    
    - Avoid extremely large fan-out (thousands of parallel activities)
    - Use batching for very large workloads
4. **Entity Concurrency**:
    
    - Entity operations are serialized by default
    - Can impact throughput for high-contention entities
    - Consider partitioning strategies for high-volume entities

### 💡 Exam Tip:

> Focus on understanding the five main Durable Functions patterns and when to use each one. Know the key differences between orchestrator, activity, entity, and client functions. Be ready to identify which pattern solves a given scenario in the exam. Remember the determinism requirements for orchestrator functions.

### ⚠️ Gotcha:

> Durable Functions orchestrator code must be deterministic. Any non-deterministic operations (like random number generation or datetime operations) should be moved to activity functions.