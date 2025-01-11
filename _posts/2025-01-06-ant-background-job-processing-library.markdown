---
layout: post
title:  "Introducing Ant: a simple background job processing library for Elixir"
date:   2025-01-06 08:00:00 +0200
categories: elixir
excerpt: "In this post, we will explore Ant, my new background job processing library for Elixir, designed with simplicity in mind. It is built on Mnesia as a storage solution and includes retry mechanisms out of the box."
---

In the Elixir community we are lucky to have a lot of great quality libraries backed by not only by core engineers, but also by the strong community. I use Elixir on a daily basis not only for my work, but also for my personal projects and never feel like I'm missing something crucial. At least in the web development area.

Nevertheless, recently I have decided to publish my new library called [Ant](https://github.com/MikeAndrianov/ant) for background job processing.

### Why another background processing library?

Elixir comes with OTP out of the box. It is one of the competitive advantages, which makes Elixir a great choice for working with multiple concurrent processes. It allows to supervise them and handle failures gracefully.
Using `Task` or `GenServer` is a go-to solution for asynchronous code execution. `Agent` is a simple way to store data and share it between processes. For a lot of cases these solutions are more than enough.

Sometimes, however, it's necessary to have more:
1. **Persist jobs between restarts** \
Storing them in order to process them later or for debugging purposes
2. **Retry failed jobs**
3. **Monitor the status of your jobs**

For such advanced cases the first what comes to my mind is [Oban](https://github.com/oban-bg/oban). It is a powerful, flexible and reliable library that has a lot of features. It is actively maintained. Many companies trust it and use it in production. Oban allows to choose between PostgreSQL and SQLite as a storage solution.

Then why did I decide to create my own library? Many years ago, when I first started to hear about Elixir, one of the main narratives was that Elixir, which is developed on top of the Erlang VM, inherited a rich Erlang ecosystem. You can use a lot of tools out of the box without any additional dependencies. One of these tools is Mnesia, a distributed database system. Unfortunately, I never saw it in a production environment in web development.

I was eager to try to apply it to my personal needs and decided to create a library for myself that would be simple and easy to use, with the ability to persist jobs and include retry mechanisms for failed jobs, without any additional dependencies. Thus, I built a proof of concept and released the first version of `ant`.


### How to use Ant

I always prefer to explain by real-world examples. So let's imagine that we have a with leads and we need to send an email to each of them.
This task can be broken down into several steps:
1. Create a file with `400_000` rows to **create leads**
2. **Parse the generated file**
3. **For each row, emulate lead creation**
4. **Send email for each lead**
5. Make a mailer to raise an exception randomly to **emulate possible real-world behavior**

First thing first, we need to create a new elixir application:

```bash
mix new ant_sandbox
```

The entry point may look like this:

```elixir
defmodule AntSandbox do
  alias AntSandbox.LeadReportGenerator
  alias AntSandbox.SendPromotionWorker

  def call() do
    :observer.start() # optionally you can start the observer to monitor the application

    :ok = LeadReportGenerator.call() # 1. Create a file with 400_000 rows to create leads

    File.stream!("leads.txt") # 2. Parse the generated file
    |> Stream.each(fn line ->
      line
      |> String.trim()
      |> send_promotion()
    end)
    |> Stream.run()
  end

  defp send_promotion(email) do # 3 and 4: Emulate lead creation and send email
    SendPromotionWorker.perform_async(%{email: email})
  end
end
```

Next, let's start with the creation of a module responsible for generating a file with leads, which will later be used as a source of leads. Each row in the file contains a randomly generated email:

```elixir
defmodule AntSandbox.LeadReportGenerator do
  def call do
    File.open!("leads.txt", [:write], fn file ->
      1..400_000
      |> Stream.map(&generate_email/1)
      |> Stream.each(&IO.write(file, &1 <> "\n"))
      |> Stream.run()
    end)
  end

  defp generate_email(id) do
    first_name = Enum.random(["john", "jane", "alex", "michael", "david", "lisa"])
    last_name = Enum.random(["smith", "williams", "brown", "jones", "garcia"])
    domain = Enum.random(["gmail.com", "yahoo.com", "hotmail.com", "example.com"])

    "#{first_name}.#{last_name}#{id}@#{domain}"
  end
end
```

A new file `leads.txt` will be generated with 400,000 emails inside. Let's pretend that we received this file from our marketing team.
So far so good. \
The next step is to implement the module responsible for creating leads and sending them emails. Sending emails to big amount of users might take some time, thus it is a good idea to make it asynchronous. Failure during the creation or sending of an email for a single lead should not affect others. That's why it is worth using separate workers for each lead.

Add `ant` to our dependencies:

```elixir
def deps do
  [
    {:ant, "~> 0.0.1"}
  ]
end
```
By default `ant` uses Mnesia with in-memory persistence strategy (`:ram_copies`) and a single queue named `default`. It is good enough for us at the moment. For more advanced cases you can consider changing default persistence strategy to `:disc_copies` or `:disc_only_copies` in order to save jobs on a disk.
For additional available configuration, please check [Configuration section](https://github.com/MikeAndrianov/ant?tab=readme-ov-file#configuration) in the GitHub repository.

The next step is to define a worker:

```elixir
defmodule AntSandbox.SendPromotionWorker do
  alias AntSandbox.Mailer
  alias AntSandbox.CreateLead

  use Ant.Worker, max_attempts: 3

  @impl Ant.Worker
  def perform(%{args: %{email: email}} = _worker) do
    with {:ok, _lead} <- CreateLead.call(email) do
      Mailer.send_promotion(email)
    end
  end
end
```

Worker module should include `use Ant.Worker` line and implement `perform` function with `%Ant.Worker{}` struct as an argument. It contains `args` field with the arguments passed to `perform_async` function, which schedules the job. Even though Mnesia can store not only basic types like atoms, integers, strings, but also complex data types like maps, lists, or tuples, it is recommended to pass only basic types as arguments.
For this worker I decided to increase retries to `3` by setting `max_attempts` option. Without it, the failed job will stay in `failed` state and will not be retried.

`CreateLead` module looks simple:
```elixir
defmodule AntSandbox.CreateLead do
  # emulate lead creation
  def call(email) do
    if String.contains?(email, "smith") and String.contains?(email, "yahoo") do
      {:error, "Invalid email"} # emulate validation error
    else
      Process.sleep(100)
      {:ok, %{email: email}}
    end
  end
end
```
It emulates validation error for emails containing `"smith"` at `yahoo` domain. For other cases it sleeps the process for 100 milliseconds and returns okay tuple.

Only `Mailer` is missing. It is responsible for sending emails with a promotion to leads. Here is the implementation:
```elixir
defmodule AntSandbox.Mailer do
  # emulate sending emails
  def send_promotion(email) do
    if :rand.uniform(10) == 1 do
      raise "Failed to send email to #{email}"
    else
      Process.sleep(100)
    end

    :ok
  end
end
```
Most of the time it sleeps a process for 100 milliseconds and later returns `:ok`, but may raise an exception randomly.
The last step is to test our application.

```elixir
iex(1)> AntSandbox.call()
:ok
```

Now we can check jobs and their statuses.
For fetching all workers, you can use `Ant.Workers.list_workers()`. It returns a list of workers`%Ant.Worker{}` regardless of their status:
```elixir
iex(2)> Ant.Workers.list_workers()
  {:ok,
   [
      %Ant.Worker{
        id: 4628067,
        worker_module: AntSandbox.SendPromotionWorker,
        queue_name: :default,
        args: %{email: "jane.jones248@yahoo.com"},
        status: :enqueued,
        attempts: 0,
        scheduled_at: ~U[2025-01-11 16:30:18.466259Z],
        updated_at: ~U[2025-01-11 16:30:18.466260Z],
        errors: [],
        opts: [max_attempts: 3]
      },
      ...
    ]
  }
```

It's possible to filter by worker attributes.

For example by status (`:enqueued`, `:running`, `:scheduled`, `:completed`, `:failed`, `:retrying`, `:cancelled`):

```elixir
iex(3)> Ant.Workers.list_workers(%{status: :completed})
  {:ok,
   [
      %Ant.Worker{
        id: 238946,
        worker_module: AntSandbox.SendPromotionWorker,
        queue_name: :default,
        args: %{email: "sarah.davis727@gmail.com"},
        status: :completed,
        attempts: 3,
        scheduled_at: nil,
        updated_at: ~U[2025-01-11 17:21:02.565686Z],
        errors: [
        %{
            error: "Failed to send email to sarah.davis727@gmail.com",
            attempt: 2,
            stack_trace: "(ant_sandbox 0.1.0) lib/mailer.ex:5: AntSandbox.Mailer.send_promotion/1\n...",
            attempted_at: ~U[2025-01-11 17:20:24.732696Z]
        },
        %{
            error: "Failed to send email to sarah.davis727@gmail.com",
            attempt: 1,
            stack_trace: "(ant_sandbox 0.1.0) lib/mailer.ex:5: AntSandbox.Mailer.send_promotion/1...",
            attempted_at: ~U[2025-01-11 17:20:08.150039Z]
        }
        ],
        opts: [max_attempts: 3]
    },
    ...
   ]}
```

Or by multiple attributes:
```elixir
iex(4)> Ant.Workers.list_workers(%{
...(4)>   queue_name: :default,
...(4)>   status: :failed,
...(4)>   args: %{email: "jane.smith734@yahoo.com"}
...(4)> })
  {:ok,
   [
      %Ant.Worker{
        id: 1150403,
        worker_module: AntSandbox.SendPromotionWorker,
        queue_name: :default,
        args: %{email: "jane.smith734@yahoo.com"},
        status: :failed,
        attempts: 3,
        scheduled_at: nil,
        updated_at: ~U[2025-01-11 17:31:29.615924Z],
        errors: [
        %{
            error: "Expected :ok or {:ok, _result}, but got {:error, \"Invalid email\"}",
            attempt: 3,
            stack_trace: nil,
            attempted_at: ~U[2025-01-11 17:31:29.615792Z]
        },
        %{
            error: "Expected :ok or {:ok, _result}, but got {:error, \"Invalid email\"}",
            attempt: 2,
            stack_trace: nil,
            attempted_at: ~U[2025-01-11 17:30:56.964108Z]
        },
        %{
            error: "Expected :ok or {:ok, _result}, but got {:error, \"Invalid email\"}",
            attempt: 1,
            stack_trace: nil,
            attempted_at: ~U[2025-01-11 17:30:32.909500Z]
        }
        ],
        opts: [max_attempts: 3]
      }
    ]}
```
If a worker fails during execution, data about failed attempts will be stored in the `errors` field. It contains the error message, stack trace, attempt number, and timestamp of the attempt. An example of such a worker can be observed above.

Also it is possible to fetch worker by its id:

```elixir
iex(5)> Ant.Workers.get_worker(1150403)
  {:ok, %Ant.Worker{...}}
```

### What's the catch?

It's definitely easy to use this library. It is simple and flexible. It doesn't require any additional database.
Nevertheless, there are two important topics to consider:
1. **Persistence in a cloud** \
When Mnesia is configured to store data on disk using `:disc_copies` (memory and disk) or `:disc_only_copies` (disk only), it writes to table files in a specified directory. \
Modern cloud platforms like Heroku or Fly.io can store only ephemeral data. That means that after the application is restarted, all data from the disk will be lost.
Luckily, you can use [Volumes](https://fly.io/docs/volumes/overview/) to create a persistent storage. It may require some additional time and effort to configure it properly, especially for larger applications, that require scalability. Even though it's solvable, it's becomes harder to set it up comparing with more popular solutions, like Oban backed by PostgreSQL.

2. **Mnesia is not widely used in the web development** \
It's significantly less popular storage solution. There are no nearly as many available resources and guides. I wish somebody experienced would write a book about it. I found difficult to comprehend the [documentation](https://www.erlang.org/doc/apps/mnesia/mnesia.html).


\
For further development I would like to hear your feedback. What you would like to see in the future versions of the library? What is crucial for you to start using it? [Please share your thoughts with me through the email](mailto:mikeandrianov@gmail.com).

\
You can find [Ant on the Github](https://github.com/MikeAndrianov/ant). Thank you for reading my article.
