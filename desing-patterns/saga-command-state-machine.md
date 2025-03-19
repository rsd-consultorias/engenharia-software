## Saga, command e state machine

**Saga Pattern (muito usado em arquiteturas de microsserviços):**
 - Esse padrão é excelente para gerenciar transações distribuídas e processos longos. Cada etapa do fluxo (captura do cartão, criação da assinatura, ativação do serviço, etc.) seria uma transação ou tarefa separada, e o Saga orquestraria ou coreografaria essas etapas.
 - Ele permite executar compensações caso uma das etapas falhe, como reverter a criação da assinatura caso a ativação do serviço não seja bem-sucedida.

```java
// Java
public class SagaExample {
    public static void main(String[] args) {
        try {
            new Saga()
                .step(() -> System.out.println("Capturing Credit Card..."))
                .step(() -> System.out.println("Creating Subscription..."))
                .step(() -> System.out.println("Activating Service..."))
                .execute();
        } catch (Exception e) {
            System.out.println("Transaction failed. Compensating...");
        }
    }
}

class Saga {
    private final List<Runnable> steps = new ArrayList<>();

    public Saga step(Runnable step) {
        steps.add(step);
        return this;
    }

    public void execute() {
        for (Runnable step : steps) {
            step.run();
        }
    }
}
```

```csharp
// C#
using System;
using System.Collections.Generic;

class SagaExample
{
    static void Main(string[] args)
    {
        try
        {
            new Saga()
                .Step(() => Console.WriteLine("Capturing Credit Card..."))
                .Step(() => Console.WriteLine("Creating Subscription..."))
                .Step(() => Console.WriteLine("Activating Service..."))
                .Execute();
        }
        catch (Exception)
        {
            Console.WriteLine("Transaction failed. Compensating...");
        }
    }
}

class Saga
{
    private readonly List<Action> steps = new List<Action>();

    public Saga Step(Action step)
    {
        steps.Add(step);
        return this;
    }

    public void Execute()
    {
        foreach (var step in steps)
        {
            step();
        }
    }
}
```

**Command Pattern:**
 - Ideal se você precisa encapsular cada etapa do processo como um comando discreto. Por exemplo, você pode ter comandos como CaptureCreditCard, CreateSubscription e ActivateService.
 - Isso é útil para manter responsabilidades bem separadas, facilitando o teste e a manutenção do código.

```csharp
// C#
interface ICommand
{
    void Execute();
}

class CaptureCreditCard : ICommand
{
    public void Execute()
    {
        Console.WriteLine("Capturing Credit Card...");
    }
}

class CreateSubscription : ICommand
{
    public void Execute()
    {
        Console.WriteLine("Creating Subscription...");
    }
}

class ActivateService : ICommand
{
    public void Execute()
    {
        Console.WriteLine("Activating Service...");
    }
}

class CommandExample
{
    static void Main(string[] args)
    {
        ICommand[] commands = {
            new CaptureCreditCard(),
            new CreateSubscription(),
            new ActivateService()
        };

        foreach (var command in commands)
        {
            command.Execute();
        }
    }
}
```

```java
// Java
interface Command {
    void execute();
}

class CaptureCreditCard implements Command {
    public void execute() {
        System.out.println("Capturing Credit Card...");
    }
}

class CreateSubscription implements Command {
    public void execute() {
        System.out.println("Creating Subscription...");
    }
}

class ActivateService implements Command {
    public void execute() {
        System.out.println("Activating Service...");
    }
}

public class CommandExample {
    public static void main(String[] args) {
        Command[] commands = {
            new CaptureCreditCard(),
            new CreateSubscription(),
            new ActivateService()
        };

        for (Command command : commands) {
            command.execute();
        }
    }
}
```

**State Machine Pattern:**
 - Se o processo de captura de cartão e ativação de serviços envolver múltiplos estados (como "captura pendente", "assinatura criada", "serviço ativado"), uma máquina de estados pode ajudar a organizar e controlar essas transições de maneira clara.

```csharp
// C#
enum State
{
    Captured,
    Subscribed,
    Activated
}

class StateMachineExample
{
    static void Main(string[] args)
    {
        State state = State.Captured;

        switch (state)
        {
            case State.Captured:
                Console.WriteLine("Credit Card Captured.");
                state = State.Subscribed;
                break;
            case State.Subscribed:
                Console.WriteLine("Subscription Created.");
                state = State.Activated;
                break;
            case State.Activated:
                Console.WriteLine("Service Activated.");
                break;
        }
    }
}
```

```java
// Java
enum State {
    CAPTURED, SUBSCRIBED, ACTIVATED
}

public class StateMachineExample {
    public static void main(String[] args) {
        State state = State.CAPTURED;

        switch (state) {
            case CAPTURED:
                System.out.println("Credit Card Captured.");
                state = State.SUBSCRIBED;
            case SUBSCRIBED:
                System.out.println("Subscription Created.");
                state = State.ACTIVATED;
            case ACTIVATED:
                System.out.println("Service Activated.");
        }
    }
}
```

## Sobre combinar mais de um pattern em uma classe

**Saga Pattern:**
 - As etapas (como CapturePaymentStep, CreateSubscriptionStep e ActivateServiceStep) encapsulam a lógica de cada ação como partes do Saga.
 - A execução do Saga percorre as etapas de forma sequencial.

**State Machine:**
 - Usamos estados (State) para controlar onde no fluxo estamos e verificar a validade das transições.
 - Se algo der errado, o estado é atualizado para FAILED.


**Vantagens:**
 - Resiliência: Facilita o rastreamento de falhas e o controle de estados.
 - Separação de Responsabilidades: Cada etapa é isolada e só cuida da sua lógica específica.
 - Escalabilidade: Fácil de adicionar novas etapas ou estados, se necessário.

```csharp
// C#
using System;

enum State
{
    Started,
    PaymentCaptured,
    SubscriptionCreated,
    ServiceActivated,
    Failed
}

interface IStep
{
    State Execute(State currentState);
}

class CapturePaymentStep : IStep
{
    public State Execute(State currentState)
    {
        Console.WriteLine("Capturing payment...");
        if (currentState == State.Started)
            return State.PaymentCaptured;
        else
            throw new InvalidOperationException("Invalid state for payment capture");
    }
}

class CreateSubscriptionStep : IStep
{
    public State Execute(State currentState)
    {
        Console.WriteLine("Creating subscription...");
        if (currentState == State.PaymentCaptured)
            return State.SubscriptionCreated;
        else
            throw new InvalidOperationException("Invalid state for subscription creation");
    }
}

class ActivateServiceStep : IStep
{
    public State Execute(State currentState)
    {
        Console.WriteLine("Activating service...");
        if (currentState == State.SubscriptionCreated)
            return State.ServiceActivated;
        else
            throw new InvalidOperationException("Invalid state for service activation");
    }
}

class SagaWithStateMachine
{
    static void Main(string[] args)
    {
        State state = State.Started;
        IStep[] steps = {
            new CapturePaymentStep(),
            new CreateSubscriptionStep(),
            new ActivateServiceStep()
        };

        foreach (var step in steps)
        {
            try
            {
                state = step.Execute(state);
                Console.WriteLine($"Current State: {state}");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
                state = State.Failed;
                break;
            }
        }

        Console.WriteLine($"Final State: {state}");
    }
}
```

```java
// Java
enum State {
    STARTED, PAYMENT_CAPTURED, SUBSCRIPTION_CREATED, SERVICE_ACTIVATED, FAILED
}

interface Step {
    State execute(State currentState) throws Exception;
}

class CapturePaymentStep implements Step {
    @Override
    public State execute(State currentState) throws Exception {
        System.out.println("Capturing payment...");
        if (currentState == State.STARTED) {
            return State.PAYMENT_CAPTURED;
        } else {
            throw new Exception("Invalid state for payment capture");
        }
    }
}

class CreateSubscriptionStep implements Step {
    @Override
    public State execute(State currentState) throws Exception {
        System.out.println("Creating subscription...");
        if (currentState == State.PAYMENT_CAPTURED) {
            return State.SUBSCRIPTION_CREATED;
        } else {
            throw new Exception("Invalid state for subscription creation");
        }
    }
}

class ActivateServiceStep implements Step {
    @Override
    public State execute(State currentState) throws Exception {
        System.out.println("Activating service...");
        if (currentState == State.SUBSCRIPTION_CREATED) {
            return State.SERVICE_ACTIVATED;
        } else {
            throw new Exception("Invalid state for service activation");
        }
    }
}

public class SagaWithStateMachine {
    public static void main(String[] args) {
        State state = State.STARTED;
        Step[] steps = {
            new CapturePaymentStep(),
            new CreateSubscriptionStep(),
            new ActivateServiceStep()
        };

        for (Step step : steps) {
            try {
                state = step.execute(state);
                System.out.println("Current State: " + state);
            } catch (Exception e) {
                System.out.println("Error: " + e.getMessage());
                state = State.FAILED;
                break;
            }
        }

        System.out.println("Final State: " + state);
    }
}
```
