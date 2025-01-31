# Scenario Outline Support

Scenario outline is a way to express the same scenario execution many times. This helps you avoid copy-pasting the same scenario several times to only replace specific values in it. Instead, you define a single scenario outline, and define all varying values as examples for that scenario outline.

## Scenario Outline in Gherkin Feature File

```Gherkin
Scenario Outline: Add two numbers with examples
	Given I chose <a> as first number
	And I chose <b> as second number
	When I press add
	Then the result should be <sum> on the screen

	Examples:
		| a   | b   | sum |
		| 0   | 1   | 1   |
		| 1   | 9   | 10  |

	Examples: of bigger numbers
		| a   | b   | sum |
		| 99  | 1   | 100 |
		| 100 | 200 | 300 |

	Examples: of large numbers
		| a    | b | sum   |
		| 999  | 1 | 1000  |
		| 9999 | 1 | 10000 |
```

In the example above, just to demonstrate what all you can do, we have split the examples into separate tables, each having a name (optional - first one does not have a name).

When you define such scenario outline, it will turn into 6 different tests after compilation (2 example rows for each table).

## DataTable and DocString in Scenario Outline

Both DataTable and DocString will receive example values if they contain placeholders. In other words, the DataTable and DocString values your methods will receive during execution will be after substituting placeholders with example values.

For clarity, example of a placeholder is `<a>`, where "a" is the placeholder name that will be substituted with the value from an example row that has the same name (example column name being "a"):

## Step Methods to Execute for Scenario Outline Steps

Importantly (and hopefully this is a convenience, too), you don't have to define step methods in any new way compared to the regular scenario steps. You define step methods which match the scenario steps when the placeholders (i.e. `<a>`, `<b>` and `<sum>`) are replaced with values from examples. This is aligned with how the framework interprets scenario outlines - it turns outline into a regular scenario by replacing placeholders with example values; then it executes the scenario, just as always.

So, your step method attributes should match the steps with actual values in them. i.e. don't define a step with `[Given(@"I chose <a> as first number")]`. Instead, define a step with `[Given(@"I chose (\d+) as first number")]`. That's because we expect that `<a>` will be replaced with digits from the outline examples.

Here is the class that handles steps for this scenario outline:

```C#
[FeatureFile("./Addition/AddTwoNumbers.feature")]
public sealed class AddTwoNumbers : Feature
{
    private readonly Calculator _calculator = new Calculator();

    [Given(@"I chose (\d+) as first number")]
    public void I_chose_first_number(int firstNumber)
    {
        _calculator.SetFirstNumber(firstNumber);
    }

    [And(@"I chose (\d+) as second number")]
    public void I_chose_second_number(int secondNumber)
    {
        _calculator.SetSecondNumber(secondNumber);
    }

    [When(@"I press add")]
    public void I_press_add()
    {
        _calculator.AddNumbers();
    }

    [Then(@"the result should be (\d+) on the screen")]
    public void The_result_should_be_z_on_the_screen(int expectedResult)
    {
        var actualResult = _calculator.Result;

        Assert.Equal(expectedResult, actualResult);
    }
}
```

In this class, all methods will execute in the order as if each example row is a scenario of its own (just as you would naturally expect).

### Resharper

You can use Resharper test runner to execute the tests but it has it's limitations. These limitations are not limited to Xunit.Gherkin.Quick but also apply to some core Xunit features.

For Xunit.Gherkin.Quick the limitations are around Scenario Outline and Background usage. This is easily noticed when you run all tests with the Visual Studio harness and compare the output to Resharper's test harness. Resharper appears to have run fewer tests - in fact it will only run a single test per Scenario Outline, regardless of how many Examples are defined.
