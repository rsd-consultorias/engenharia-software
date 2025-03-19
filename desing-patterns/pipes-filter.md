## Pipes & Filter

```csharp
// C#
// salvar em um arquivo PipesFilters.csx
// para executar abra o terminal e rode o arquivo com o comando "csi PipesFilters.csx"

using System;

public static class Pipeline
{
    public static T ApplyPipeline<T>(T input, params Func<T, T>[] filters)
    {
        return filters.Aggregate(input, (current, filter) => filter(current));
    }
}

public static class Filters
{
    public static double Receiver(double eventValue)
    {
        Console.WriteLine($"received {eventValue}");
        return eventValue * 100;
    }

    public static double Transformer(double eventValue)
    {
        Console.WriteLine($"transformed {eventValue}");
        return eventValue * 10;
    }

    public static double Adder(double eventValue, double constant)
    {
        Console.WriteLine($"added {eventValue}");
        return eventValue + constant;
    }

    public static double Send(double eventValue)
    {
        Console.WriteLine($"sent {eventValue}");
        return eventValue / 1000;
    }
}

public class PipesFilters
{
    public static void Teste()
    {
        var input = 10.0;

        var result = Pipeline.ApplyPipeline(input,
            Filters.Receiver,
            Filters.Transformer,
            e => Filters.Adder(e, 20),
            Filters.Send);

        Console.WriteLine(result);
    }
}

PipesFilters.Teste();
```

```java
// Java
// salvar em um arquivo PipesFilters.java
// para rodar execute o comando "jshell PipesFilters.java", quando o shell abrir
// execute "PipesFilters.main(null);"

import java.util.function.Function;

class PipesFilters {
    public static void main(String[] args) {
        var input = 10.0;

        var result = Pipeline.applyPipeline(input,
                Filters::receiver,
                Filters::transformer,
                e -> Filters.adder(e, 20),
                Filters::send);

        System.out.println(result);
    }

    public class Pipeline {

        @SafeVarargs
        private static <T> T applyPipeline(T input, Function<T, T>... filters) {
            for (Function<T, T> filter : filters) {
                input = filter.apply(input);
            }
            return input;
        }
    }

    public class Filters {
        public static double receiver(double event) {
            System.out.printf("received %s\n", event);
            return event * 100;
        }

        public static double transformer(double event) {
            System.out.printf("transformed %s\n", event);
            return event * 10;
        }

        public static double adder(double event, double constant) {
            System.out.printf("added %s\n", event);
            return event + constant;
        }

        public static double send(double event) {
            System.out.printf("sent %s\n", event);
            return event / 1000;
        }
    }
}
```

Explicação do Pattern
 - Pipeline Centralizado: Ambos os exemplos implementam pipelines, onde cada filtro processa e encaminha os dados para o próximo.
 - Filtros Reutilizáveis: Transformações e persistência são encapsuladas como filtros.
 - Flexibilidade: Novos filtros podem ser facilmente adicionados ao pipeline.
