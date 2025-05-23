# .NET Coding Conventions

## Solution and Project

### Nullability

For projects targeting C# 8 or later, nullability must always be enabled using the project property `Nullable`.

*<u>Interpretation</u>: Nullability is crucial for correctly and reliably handling `null` cases in reference types. It also prevents redundant booleans used as nullability flags for associated `struct` values.*

## General

### Using Directives Order

Namespace imports via the `using` keyword are placed at the top of the file and ordered alphabetically in the following sequence:

* All base .NET library imports (those beginning with `System`)
* Imports from the current project
* Third-party library imports

Flexibility is allowed regarding this order.

*<u>Interpretation</u>: The flexible enforcement reflects the mutable and significant nature of imports. Moreover, these are often managed by IDEs that impose their own order.*

### Local Variable Types

The `var` keyword must be used wherever possible, without exception.

### Current Object References

The `this` and `base` keywords must be used for intra-type references whenever available, regardless of context.

## UI

### Naming UI Elements

UI elements that can be named must follow this convention: the chosen name is prefixed by the element type (abbreviated clearly if too lengthy).

Examples:

| UI Element Type | Example Element Name |
| --------------- | -------------------- |
| `Button`        | `ButtonBack`         |
| `Label`         | `LabelCopyright`     |

*<u>Interpretation</u>: For UI elements, unlike standard fields or properties, including the type name in the variable name is often essential for semantic clarity. Thus, standardization is necessary.*

## Class Separation

User-defined separations (using the `partial` keyword) at the `class`, `struct`, or `interface` level should be used only in targeted cases to improve code readability and comprehension.

### File Naming

Files containing parts of partial classes end with a `.` followed by the context (excluding the extension). The main part does not bear this suffix.

Example for the split `WebView` class:

`WebView.cs`
`WebView.Navigation.cs`
`WebView.Render.cs`

All files containing the split type must reside in the same folder.

## Methods

### Naming

Methods follow the standard PascalCase naming convention, with exceptions:

#### *Events*

Methods subscribed to events are named as follows:

`OnEventName`
`ContextName_OnEventName`

Where `EventName` is the name of the event (abbreviated clearly if too long) and `ContextName` is the object or type emitting the event, or omitted if it’s `this`.

Examples:

| Context Name   | Event Name    | Method Name                  |
| -------------- | ------------- | ---------------------------- |
| `ButtonBack`   | `Click`       | `ButtonBack_OnClick`         |
| `TextBoxTitle` | `Initialized` | `TextBoxTitle_OnInitialized` |

#### *Asynchronous*

An asynchronous method (returning `Task`, `Task<TResult>`, or derived) must include the suffix `Async`.

Examples (partial method definitions):

```csharp
int Push
Task RegisterUserAsync
Task<bool> DeleteItemAsync
```

*<u>Interpretation</u>: Naming async methods with `Async` ensures `await` is always used, avoiding execution inconsistencies and improving debugging clarity.*

### Extension Methods

Extension methods must name their first parameter (the object to extend, marked with `this`) as `self` in all cases.

*<u>Interpretation</u>: The `self` identifier substitutes `this`, avoiding naming conflicts with other local variables.*

## Type Parameters

### Naming

Type parameters must always be in PascalCase prefixed by `T`.

Examples:

`TWorkload`
`TServiceFactory`

### Indentation

The `where` keyword must always start on a new line. Each `where` clause is on its own line, indented one block right from the start of the parent declaration.

Example:

```csharp
public class DummyClass<TParam>
    where TParam : TParamBase
{
    public void TestAction<TMethodParam1, TMethodParam2>()
        where TMethodParam1 : TMethodParam1Base
        where TMethodParam2 : TMethodParam2Base
    {
    }
}
```

### Dependency Injection

Constructor arguments in DI-based classes should follow this order:

1. Logging tools, e.g. `ILogger<T>`, `ILoggerFactory<T>`
2. Dependency container-related services, e.g. `IServiceProvider`, `IServiceScopeFactory`
3. Other unclassified dependencies
4. Options and option generators, e.g. `IOptions<T>`
5. Non-container injectable parameters specific to the implementation (e.g. from factories)

## Asynchronous Programming

Async methods (except rare utility cases like `Task.WhenAll`) must include a final `CancellationToken` parameter named `cancellationToken` or `ct`, defaulted to `default` (preferred over `CancellationToken.None`).

*<u>Interpretation</u>: Since the `CancellationToken` type is clear from the context, it's not mandatory to restate it.*

Option-like types for async execution management should also be final, optional parameters (nullable if reference types, `default` otherwise), optionally abbreviated (e.g., `ao` for `AsyncOptions`).

## Extensions

Extension methods must reside in dedicated `static` classes called extension classes.
There must be a one-to-one relationship between the extended type and its extension class within the same Assembly.

### Naming Extension Classes

Extension classes are named after the extended class with `Extension` appended. For generics, if homonyms exist, the name includes a generic summary with `_T` and the number of type parameters.

Examples:

| Extended Class       | Extension Class Name          |
| -------------------- | ----------------------------- |
| `DateTime`           | `DateTimeExtension`           |
| `Queue`              | `QueueExtension`              |
| `IServiceCollection` | `IServiceCollectionExtension` |
| `IList`              | `IListExtension`              |
| `IList<T>`           | `IList_T1Extension`           |

### File Location

Extension class files reside in `Utils/Extensions/`, or a subdirectory indicating context.

<pre><b>Root/</b>
 └─ Utils/
     └── Extensions/    
          ├── Collections/
          │    └── IEnumerableExtension.cs
          └── IServiceCollectionExtension.cs
</pre>

### Splitting Extension Classes

Splitting is allowed and useful for heavy types like `System.Collections.Generic.IEnumerable<T>`.

Example:

| Partial Class File              | Purpose                        |
| ------------------------------- | ------------------------------ |
| `IEnumerableExtension.cs`       | Synchronous extension methods  |
| `IEnumerableExtension.Async.cs` | Asynchronous extension methods |

*<u>Interpretation</u>: Since a type can have only one extension class in an Assembly, splitting it into main contexts improves organization.*

#### *Extending `IServiceCollection`*

Splitting is also useful for organizing `IServiceCollection` extensions by registration context.

Examples:

`IServiceCollectionExtension.cs`
`IServiceCollectionExtension.ViewModel.cs`
`IServiceCollectionExtension.Background.cs`

## Utility Classes

Utility functions for specific types must reside in files under the `Utils` folder at the project root, or a context-specific subfolder.

Example:

<pre><b>Root/</b>
 └─ Utils/
     ├── Json/    
     │    └── Converters/
     │         └── ResponseConverter.cs
     └── DateTimeUtils.cs
</pre>

## Namespaces

### Extension Classes

The namespace for extension classes is typically the project root namespace.

*<u>Interpretation</u>: Unless logically grouped otherwise, extension methods should be easily accessible on base types.*

## Code Bodies

### Method Bodies

Where possible, and excluding constructors and finalizers, method bodies must use expression-bodied syntax with `=>`.

*<u>Interpretation</u>: Constructors and finalizers have special roles, frequent changes, and should retain full method bodies for clarity and visibility.*

#### *Type Member Methods*

The expression is placed on the next line, indented one block right.

```csharp
public class DummyClass
{
    public void TestMethod()
        => AnotherTestMethod();
}
```

#### *Properties*

Single-expression properties use:

<pre><b>Property</b> => <b>Value computation expression</b></pre>

`=>` may be placed on the next line if too long.

For manually implemented getters/setters, the expression goes inline after the definition:

```csharp
public class DummyClass
{
    public string NormalManualGetProperty => "Computed value with a single expression";

    public string VeryVeryVeryVeryVeryVeryVeryVeryLongManualGetProperty
        => "Computed value with a single expression but on the next line";

    public string TestGetSetProperty
    {
        get => AnotherGetMethod();
        set => AnotherSetMethod(value);
    }

    public string TestGetInitProperty
    {
        get => AnotherGetMethod();
        init => AnotherSetMethod(value);
    }
}
```

## LINQ Queries and Method Chains

### LINQ Queries

LINQ queries are indented one block right. Each clause gets its own line, including the first.

Examples:

```csharp
public class DummyClass
{
    public int TestLinqRequest()
    {
        var numbers = new int[] {0, 1, 2, 3, 4};
        
        var queryResult =
            from number in numbers
            where (num % 2) == 0
            select num;

        return queryResult;
    }
}
```

```csharp
public class DummyClass
{
    public int TestLinqRequest()
    {
        var numbers = new int[] {0, 1, 2, 3, 4};
        
        var queryResult = TransformQuery(
                from number in numbers
                where (num % 2) == 0
                select num,
            true);

        return queryResult;
    }
}
```

### Method Chains

#### *Definition*

Method chains are sequences of method calls separated by `.`, commonly used for configuration (also called *Fluent* syntax).

#### *Indentation*

Each chained method indents one block right. The chain source aligns with the previous expression.

Examples:

```csharp
public class DummyClass
{
    public TestChainedMethods()
    {
        var result = this._source.NewQuery()
            .Where((i) => i < 5)
            .Select((i) => i * 3)
            .ToArray();
    }
}
```

```csharp
public static class IServiceCollectionExtension
{
    public static IServiceCollection AddViewModels(this IServiceCollection self)
        => self.AddViewModel<LoadingPageViewModel>()
               .AddViewModel<MainPageViewModel>()
               .AddViewModel<MainSettingsPageViewModel>();
}
```

## Asynchronous Code

### ConfigureAwait

Use `.ConfigureAwait(false)` when `await`-ing a `Task`, unless resuming on the calling context is needed.

*<u>Interpretation</u>: This secures the code against potential deadlocks.*

Example:

```csharp
public class DummyClass
{
    public async Task TestAsync()
    {
       await Task.WhenAll(
           AnotherFirstTestAsync(),
           AnotherSecondTestAsync()
       ).ConfigureAwait(false);

       await AnotherThirdTestAsync().ConfigureAwait(false);
    }
}
```

## Additional Recommendations

### LINQ Query Alignment

Align the `from` clause with the preceding expression only if unique; otherwise, start on a new line.

### Primary Constructors

Avoid using primary constructors for non-`record` types.

*<u>Interpretation</u>: Primary constructors in non-record types are limited, don't allow `readonly` fields, and their parameter ambiguity reduces clarity and usability.*

### Null Reference Validation

Avoid null reference checks at the start of methods unless the code is third-party and widely deployed (e.g., part of .NET runtime or core libraries).

*<u>Interpretation</u>: The caller is responsible for proper usage. Nullable types and documentation should suffice.*
