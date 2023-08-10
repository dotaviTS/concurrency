


## Theories and Concepts

### 1. Supervision Trees

Coursework: https://elixirschool.com/en/lessons/advanced/otp_supervisors
Elixir Lang - https://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html
Hex Docs: https://hexdocs.pm/elixir/1.12/Supervisor.html
Supervisors Article: https://medium.com/learn-elixir/supervisors-and-workers-in-10-minutes-83fbad6f16d1
Article on Supervision Trees: https://kodius.com/blog/elixir-supervision-tree
Good Article with Diagrams: https://www.educative.io/answers/what-are-the-strategies-available-to-supervisors-in-elixir

Find relevant Youtube videos to further your understanding


A supervision tree is a hierarchical structure of supervisors and worker processes. Supervisors can oversee other supervisors, creating a tree-like structure that enhances fault tolerance.

#### Example
Consider a web application with a database connection pool, cache, and background job processing. The structure might look like:

https://ibb.co/SB98HLM

 **Restart Strategies**
These define how the supervisor will react to child failures.
 - :one_for_one: If a child process terminates, only that process is    
   restarted.     
 - :one_for_all: If a child process terminates, all other
   child processes are terminated and restarted.      
 - :rest_for_one: If a child process terminates, the rest of the children started after it are terminated and restarted.

### Exercise
Identify suitable scenarios for each restart strategy.

#### 1. `:one_for_one` Strategy

This strategy restarts only the failed child process. It is suitable for scenarios where child processes are independent of one another.

**Example Scenario:**

-   In a web server, where each worker process handles an independent client request. If one request fails, it should not affect the other requests.

#### 2. `:one_for_all` Strategy

This strategy restarts all child processes if one fails. It's suitable for scenarios where all child processes are tightly coupled and dependent on one another.

**Example Scenario:**

-   In a distributed database cluster, if one node fails, it might cause inconsistency across other nodes. Restarting all nodes ensures that they are synchronized and consistent.

#### 3. `:rest_for_one` Strategy

This strategy restarts the failed child process and any process started after it. It's suitable for scenarios where some child processes depend on others, but not all of them.

**Example Scenario:**

-   In a multi-stage data processing pipeline, if one stage fails, all subsequent stages that depend on it must be restarted, but previous stages can continue to run.


## 2. Dynamic Supervision
Dynamic supervision allows you to manage child processes that are created at runtime. This is useful when you don't know the children beforehand.

**Example**
A chat application where each user connection spawns a new supervised process:
```
options = [name: ChatApp.Supervisor, strategy: :one_for_one]
DynamicSupervisor.start_link(options)

To start a new chat handler dynamically:

{:ok, pid} = DynamicSupervisor.start_child(ChatApp.Supervisor, ChatHandler)
```
## 3. Task Supervision
The Task.Supervisor specializes in supervising short-lived tasks.

Example
Running a background job to send email notifications:
```
{:ok, pid} = Task.Supervisor.start_child(MyApp.TaskSupervisor, fn -> send_emails end)
```
**Exercise: Building a Supervised Application**

Provide a hands-on exercise for students to create a supervised application, such as a simple queue system or a chat server. Provide step-by-step instructions, guiding them through defining child specifications, choosing appropriate restart strategies, and implementing dynamic supervision if needed.


Creating a Project
Create a new project using mix new simple_queue --sup.
Place the SimpleQueue module code in lib/simple_queue.ex.
Add the supervisor code in lib/simple_queue/application.ex.
Children can be defined using a list of module names or a list of tuples if you want to include configuration options:

Example 1:
```
defmodule SimpleQueue.Application do
  use Application

  def start(_type, _args) do
    children = [SimpleQueue]
    opts = [strategy: :one_for_one, name: SimpleQueue.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

Example 2:
```
defmodule SimpleQueue.Application do
  use Application

  def start(_type, _args) do
    children = [{SimpleQueue, [1, 2, 3]}]
    opts = [strategy: :one_for_one, name: SimpleQueue.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```
Running iex -S mix will automatically start SimpleQueue with [1, 2, 3]. If it crashes, the Supervisor would restart it as if nothing had happened.


DynamicSupervisor
For cases where supervised children will not be known at startup, a Dynamic Supervisor is used.

Example:
```
options = [name: SimpleQueue.Supervisor, strategy: :one_for_one]
DynamicSupervisor.start_link(options)
{:ok, pid} = DynamicSupervisor.start_child(SimpleQueue.Supervisor, SimpleQueue)
```
Task Supervisor
Tasks have their specialized Supervisor, the Task.Supervisor. It's designed for dynamically created tasks.

Example:
```
children = [{Task.Supervisor, name: ExampleApp.TaskSupervisor, restart: :transient}]
{:ok, pid} = Supervisor.start_link(children, strategy: :one_for_one)
{:ok, pid} = Task.Supervisor.start_child(ExampleApp.TaskSupervisor, fn -> background_work end)
```
Supervision Trees
Explore the hierarchical structure of supervisors and workers, and how it enhances fault tolerance.


## Exercise: Building a Supervised Customer Support Ticket System

### Objective

Create a system that allows customer support tickets to be managed, including the creation, assignment to support agents, and resolution of tickets, using Elixir's supervision capabilities.

### Step-by-Step Instructions

#### 1. **Create a New Project**
```
mix new support_system --sup
```

#### 2. **Create the Ticket Worker Module**

Create a file named `ticket_worker.ex` in the `lib/support_system` directory:

```
defmodule SupportSystem.TicketWorker do
  use GenServer

  def start_link(ticket) do
    GenServer.start_link(__MODULE__, ticket)
  end

  def init(ticket) do
    {:ok, ticket}
  end

  def handle_call(:assign_agent, _from, ticket) do
    assigned_ticket = Map.put(ticket, :status, :assigned)
    {:reply, :ok, assigned_ticket}
  end

  def handle_call(:resolve, _from, ticket) do
    resolved_ticket = Map.put(ticket, :status, :resolved)
    {:reply, :ok, resolved_ticket}
  end
end
```

#### 3. **Create the Dynamic Supervisor for Tickets**

Create a file named `ticket_supervisor.ex`:
```
defmodule SupportSystem.TicketSupervisor do
  use DynamicSupervisor

  def start_link(options \\ []) do
    DynamicSupervisor.start_link(__MODULE__, options, name: __MODULE__)
  end

  def init(_options) do
    DynamicSupervisor.init(strategy: :one_for_one)
  end

  def create_ticket(ticket_details) do
    spec = {SupportSystem.TicketWorker, ticket_details}
    DynamicSupervisor.start_child(__MODULE__, spec)
  end
end
```
#### 4. **Update the Application Supervisor**

Edit the `application.ex` file to start the TicketSupervisor:

```
defmodule SupportSystem.Application do
  use Application

  def start(_type, _args) do
    children = [
      SupportSystem.TicketSupervisor
    ]

    opts = [strategy: :one_for_one, name: SupportSystem.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

#### 5. **Test the Ticket System**

Start an interactive Elixir shell:
```
iex -S mix
```

Create a ticket:
```
ticket_details = %{id: 1, description: "Login issue", status: :open}
{:ok, pid} = SupportSystem.TicketSupervisor.create_ticket(ticket_details)` 
```
Assign an agent to the ticket:
```
GenServer.call(pid, :assign_agent)
```

Resolve the ticket:

```
GenServer.call(pid, :resolve)
```
