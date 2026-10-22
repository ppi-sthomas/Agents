# AGENTS.md
<codebase info>
</codebase info>
<code styling instructions>
Avoid Ternary operations.
don't await inside an if statement.
braces are required for all if, while, for, foreach, statements.



<highly important instructions>
a note: these are instructions that are highly important but seem to be the hardest to implement. this is because they go against your nature and what you're first responses are. because of this their instructions will be longer and give examples. but each one is important and can never be ignored by you.
first is null obsession. In programming things are often null and you often have to check for null. Or at least that is how you are taught. Null obsession is a real issue with programming using AI models. As AI models follow the paradigm of a method has to ensure that its arguments are what they expect. But that is not how I prefer to program, and that's not how you should generate code. Instead, it should be on the caller of the method to ensure that they are not passing in arguments incorrectly. Take an example that commonly occurs. I have an API endpoint in which it takes a string from a property in a request body, and we need to parse that as an enum in order to continue on with our code. Your first instinct would be to check that the request body is not null, that the string is not null, that you can parse the string as the enum that we are working with. This is wrong on so many levels. Instead, if you are calling our endpoint that is expecting an enum, we should parse the enum just as it comes in. If an exception happens, then you should let that bubble out because that is your next follow-up instinct is to say, okay, well, we're doing dangerous code, we need to throw a try-catch around it. But that isn't right either, because we can often set up our controllers to automatically catch exceptions. I would much rather prefer simple readable code that breaks rather than complex messy code that doesn't.
<bad>
public IActionResult Create(TaskRequest request)
{
  try
  {

    if (
      request is null ||
      request.Priority is null ||
      !Enum.TryParse<Priority>(request.Priority, ignoreCase: true, out var priority) ||
      !Enum.IsDefined(priority))
    {
        return BadRequest($"Invalid priority: '{request?.Priority ?? ""}'");
    }

    return Ok(new { priority });
  }
  catch (Exception ex)
  {
    return BadRequest($"Invalid priority: '{request?.Priority ?? ""}'");
  }
}
</bad>
<good>
public IActionResult Create(TaskRequest request)
{
    return Ok(new { Enum.Parse<Priority>(request.Priority) });
}
</good>

By making this change, we produce more readable code that when something goes wrong, it isn't unknown as to why.

The second issue comes with formatting code, whether due to the tokenizer ignoring white space or being trained in a way to write code that makes it easier to parse at a glance rather than when trying to dig into it in detail, formatting your code to an 80-character limit results in wildly unreadable code. I don't have an exact preferred line limit as it's generally where it makes sense, but roughly speaking, the line limit should be around 200 to 300 characters. as well as readability, large parameter lists generally come from a sign of or code. So if we are writing code that requires so many parameters, we probably need to rethink how we're approaching it anyways
for example something that you might produce is this, i've replaced the tabs with \t to ensure it doesn't get trimmed:
<bad>
\t\tpublic ReportsController(
\t\t\t\tILogger<ReportsController> logger,
\t\t\t\tIReportService reportService,
\t\t\t\tIUserService userService,
\t\t\t\tIAuditService auditService,
\t\t\t\tIClock clock)
\t\t{
\t\t\t\t_logger = logger;
\t\t\t\t_reportService = reportService;
\t\t\t\t_userService = userService;
\t\t\t\t_auditService = auditService;
\t\t\t\t_clock = clock;
\t\t}

\t\t[HttpGet("summary")]
\t\tpublic IActionResult GetSummary(int tenantId, DateTime from, DateTime to)
\t\t{
\t\t\t\tvar user = _userService.GetCurrentUser();
\t\t\t\t_logger.LogInformation("Summary requested by {User} for tenant {TenantId}", user, tenantId);
\t\t\t\t_auditService.Record("reports.summary");

\t\t\t\tvar summary = _reportService.BuildSummary(tenantId, from, to);

\t\t\t\treturn Ok(new { generatedAt = _clock.UtcNow, summary });
\t\t}

\t\t[HttpGet("advanced")]
\t\tpublic IActionResult GetAdvanced(
\t\t\t\tstring region,
\t\t\t\tstring category,
\t\t\t\tDateTime from,
\t\t\t\tDateTime to,
\t\t\t\tint page,
\t\t\t\tint pageSize,
\t\t\t\tstring sortBy,
\t\t\t\tstring sortDirection,
\t\t\t\tbool includeArchived,
\t\t\t\tbool includeDeleted,
\t\t\t\tstring currency,
\t\t\t\tstring language,
\t\t\t\tstring timezone,
\t\t\t\tint? minAmount,
\t\t\t\tint? maxAmount,
\t\t\t\tstring status,
\t\t\t\tstring ownerId,
\t\t\t\tstring search,
\t\t\t\tbool includeSubcategories,
\t\t\t\tstring format)
\t\t{
\t\t\t\t_logger.LogInformation("Advanced report requested for region {Region}", region);
\t\t\t\t_auditService.Record("reports.advanced");

\t\t\t\treturn Ok(new
\t\t\t\t{
\t\t\t\t\t\tregion,
\t\t\t\t\t\tcategory,
\t\t\t\t\t\tfrom,
\t\t\t\t\t\tto,
\t\t\t\t\t\tpage,
\t\t\t\t\t\tpageSize,
\t\t\t\t\t\tsortBy,
\t\t\t\t\t\tsortDirection,
\t\t\t\t\t\tincludeArchived,
\t\t\t\t\t\tincludeDeleted,
\t\t\t\t\t\tcurrency,
\t\t\t\t\t\tlanguage,
\t\t\t\t\t\ttimezone,
\t\t\t\t\t\tminAmount,
\t\t\t\t\t\tmaxAmount,
\t\t\t\t\t\tstatus,
\t\t\t\t\t\townerId,
\t\t\t\t\t\tsearch,
\t\t\t\t\t\tincludeSubcategories,
\t\t\t\t\t\tformat,
\t\t\t\t\t\tgeneratedAt = _clock.UtcNow
\t\t\t\t});
\t\t}
}
</bad>
<good
\tpublic ReportsController(ILogger<ReportsController> logger, IReportService reportService, IUserService userService, IAuditService auditService, IClock clock)
\t{
\t\t_logger = logger;
\t\t_reportService = reportService;
\t\t_userService = userService;
\t\t_auditService = auditService;
\t\t_clock = clock;
\t}

\t[HttpGet("summary")]
\tpublic IActionResult GetSummary(int tenantId, DateTime from, DateTime to)
\t{
\t\tvar user = _userService.GetCurrentUser();
\t\t_logger.LogInformation("Summary requested by {User} for tenant {TenantId}", user, tenantId);
\t\t_auditService.Record("reports.summary");

\t\tvar summary = _reportService.BuildSummary(tenantId, from, to);

\t\treturn Ok(new { generatedAt = _clock.UtcNow, summary });
\t}

\t[HttpGet("advanced")]
\tpublic IActionResult GetAdvanced(
\t\tstring region, string category,
\t\tDateTime from, DateTime to,
\t\tint page, int pageSize,
\t\tstring sortBy, string sortDirection,
\t\tbool includeArchived, bool includeDeleted,
\t\tstring currency, string language, string timezone,
\t\tint? minAmount, int? maxAmount,
\t\tstring status, string ownerId, string search, bool includeSubcategories, string format)
\t{
\t\t_logger.LogInformation("Advanced report requested for region {Region}", region);
\t\t_auditService.Record("reports.advanced");

\t\treturn Ok(new
\t\t{
\t\t\tregion, category,
\t\t\tfrom, to,
\t\t\tpage, pageSize,
\t\t\tsortBy, sortDirection,
\t\t\tincludeArchived, includeDeleted,
\t\t\tcurrency, language, timezone,
\t\t\tminAmount, maxAmount,
\t\t\tstatus, ownerId, search, includeSubcategories, format,
\t\t\tgeneratedAt = _clock.UtcNow
\t\t});
\t}
</good>
Even the this could use some improvements. The advanced endpoint should really have the whole thing returned as a object instead of 20 parameters. But this would be a better way of formatting it.
there are exclusions to the don't wrap things on multiple line rules, and that's fluent method calls or chained calls

The next few are a wombo combo. This has to do with both the age of the data sets that you were trained on, as well as more of a personal preference
The first is to use records for simple types that have no logic behind them and are should be really treated as immutable over classes
The second is to use file scope namespaces. That's where you have a namespace ending with a semicolon rather than an opening and closing brace to reduce the amount of indentation needed.
The third is to use or the third is to not use sealed on classes. Sealed provides no benefit for classes that are not inheriting from another and really just gets in the way if you do intend to inherit certain classes. Leave it to the user to add sealed.
The fourth is to use a primary constructor rather than a defined constructor with backing fields. This is a newer feature of C sharp, but it allows for easier management of dependencies and just a lot more nicer to use

Using the previous example as a working model and not converting the white space into tab characters, it would look something more like this
<good>
using MyApi.Controllers;
[ApiController]
[Route("api/[controller]")]
public class ReportsController(ILogger<ReportsController> logger, IReportService reportService, IUserService userService, IAuditService auditService, IClock clock) : ControllerBase
{
  public record SummaryResponse(DateTime GeneratedAt, Summary Summary);
  [HttpGet("summary")]
  public IActionResult GetSummary(int tenantId, DateTime from, DateTime to)
  {
    var user = userService.GetCurrentUser();
    logger.LogInformation("Summary requested by {User} for tenant {TenantId}", user, tenantId);
    auditService.Record("reports.summary");

    var summary = reportService.BuildSummary(tenantId, from, to);

    return Ok(new SummaryResponse(clock.UtcNow, summary));
  }

  [HttpGet("advanced")]
  public IActionResult GetAdvanced(RequestDto dto)
  {
    logger.LogInformation("Advanced report requested for region {Region}", region);
    auditService.Record("reports.advanced");

    return Ok(dto);
  }
}
</good>
As you can see, this results in significantly less code needing to be written and allowing us to focus more on the content of the methods rather than the formatting of them.

The next one is a much more challenging concept to address in a system prompt file but what it boils down to is when humans write code, we hate doing it. So we abstract out complex processes into more manageable pieces. And we try to make things readable via like extension methods, extension properties, calling methods to do something that we could do in line, etc. But when AI agents tend to write, they tend to just write out all of the code all at once. they also tend to try to handle every scenerio all at once ignoring the complexities the codebase has, or the fact that we have total control over the codebase. say if a parameter needs to not be null, ai tends to handle that in the method, but it should be the callers responsibility to ensure they are calling the method correctly. if we need that parameter to have a value then why is the caller calling it without one, does the caller know they are sending null there? why is the sender allowing null to be sent. now then your second reaction is to just say "ok we won't send it if it's null" but then now we aren't doing anything when we expect something to happen. to bring this as an analogy. take a standard car. it's not the cars responsibility to check if it has a driver, you wouldn't expect a normal car to not start because there isn't a person behind the wheel. you told it to start so it starts, it's up to you to ensure someone is behind the wheel. maybe a bit of a poor analogy but given the scenerio:
```
void OnSubmit()
{
  string foo = getStringFromElement(); // this returns null
  callFoo(foo);
}

void callFoo(string bar)
{
  Console.WriteLine(bar.ToString()); // placeholder for important work
}
```
in this scenerio it would be wrong for callFoo function to check for null. It expects a valid string and should only be handling valid strings.
<bad>
void callFoo(string bar)
{
  if(bar == null) return;
  Console.WriteLine(bar.ToString()); // placeholder for important work
}
</bad>
and it would also be wrong to handle it in onSubmit, onsubmit is expecting a valid string to be there, it expects to get the string from the element.
<bad>
void OnSubmit()
{
  // wrong hides the error
  string foo = getStringFromElement() ?? "";
  // wrong this doesn't help us get a valid string just forces the error to happen earlier
  string foo = getStringFromElement() ?? throw new NullReferenceException();
  callFoo(foo);
}
</bad>
instead the solution is often further out and maybe a bit more subtle. but saying callFoo is getting a null reference exception then neither of the above are the solution. in this example something has caused getStringFromElement to fail to find the element or the string. we should look into and solve that.

Let's look at another example. This is one where I have asked for an indentation parser that lets me get the contents of the blocks from it. In the initial response, AI's first instinct is to handle all indentation types, handle the default indentation, and overall over-engineer the crap out of it when all I need is just the contents of the indent in a data structure. It took multiple iterations with an AI agent to get it to produce something that I actually want. And I hope that this helps showcase the differences between them.
> Create a  method that parses the indentation level of a text block and returns a linked list or a tree of the blocks and their nesting and the content of them in c#
<bad>
public sealed class Block
{
    public string Content { get; set; }
    public int Indent { get; set; }
    public List<Block> Children { get; } = new List<Block>();
}

public static class IndentParser
{
    public static Block Parse(string text, int tabWidth = 4)
    {
        var root = new Block { Indent = -1 };

        if (string.IsNullOrEmpty(text))
            return root;

        var stack = new Stack<Block>();
        stack.Push(root);

        foreach (var rawLine in text.Split('\n'))
        {
            var line = rawLine.TrimEnd('\r');

            if (string.IsNullOrWhiteSpace(line))
                continue;

            int indent = GetIndent(line, tabWidth);
            string content = line.Trim();

            var node = new Block
            {
                Indent = indent,
                Content = content
            };

            while (stack.Peek().Indent >= indent)
                stack.Pop();

            stack.Peek().Children.Add(node);
            stack.Push(node);
        }

        return root;
    }

    private static int GetIndent(string line, int tabWidth)
    {
        int count = 0;

        foreach (char c in line)
        {
            if (c == ' ')
                count++;
            else if (c == '\t')
                count += tabWidth;
            else
                break;
        }

        return count;
    }
}
</bad>
<good>
public record Block(int Indent)
{
    public string Content { get; set; }
    public Block Parent { get; set; }
    public List<Block> Children { get; } = [];
}

public static class IndentParser
{
    public static List<Block> Parse(string text)
    {
        var root = new Block(-1);
        var previous = root;
        foreach (var line in text.Split('\n'))
        {
            var indent = CountIndent(line);
            while (previous.Indent >= indent)
            {
                previous = previous.Parent;
            }
            var block = new Block(indent)
            {
                Content = line,
                Parent = previous
            };
            previous.Children.Add(block);
            previous = block;
        }
        return root.Children;
    }
    private static int CountIndent(string line)
    {
        return line.TakeWhile(c => c == ' ').Count();
    }
}
</good>

The assumptions baked in are that input is valid and non-empty, indentation is spaces only, there are no blank lines, line breaks are \n, the first line has no indent, and each deeper line's indent increases consistently so that walking up by comparison always lands on a real parent.

by adding in a few assumptions that we want can Greatly simplify the code, allowing it to be way more readable, and by adding in some helper classes like count indent or the constructor requiring an indent, we can simplify the instantiations of many of our properties and data
To really nitpick the original implementation, it doesn't use the new syntax for an expression collection, it doesn't use a record type for something that once returned should be immutable. It adds sealed when it's not inheriting from anything and doesn't need it. It accepts a tab width when it's entirely not needed. It uses stacks. It's handling if the string is null. It's not using braces, it's adding in all of this additional logic when all I need and all I asked for was a simple parser for indentation levels
You could call this happy path programming, assumption programming. But the point is, I never said that this would be mixed indentation or what the indentation type is. So you can assume a particular type of indentation. I never said that the string being parsed in would be null or empty and that would need to be handled. So therefore, you don't really need to handle it. I never said that this would be needed to be used across Linux or Windows. So you can just assume you can just do new lines. There's a lot of assumptions that you are making that are overcomplicating the code, when rather I would prefer if you made assumptions that simplify the code and make it more readable.

Next one on the list is the try-catch usage and the cancellation token usage in async await in async methods it is very common for you to end up throwing a try-catch around them simply because there is an async call that is not needed if there's an exception an exception is going to happen and that is what we want we want exceptions so that we can log them in the global handler. The other thing is with cancellation tokens. Cancellation tokens are a very nice feature if you plan on implementing them. If you plan on adding a cancellation token source and handling the token being canceled, it is amazing. But you adding it to the every single method just because it's available is asinine and it just bloats the parameter list that is already bloated up as it is. Do not do this
<bad>
  public async Task<ActionResult<UserDto>> UpdateMe([FromBody] UpdateMeRequest request, CancellationToken ct)
  {
    var user = await accounts.UpdateMeAsync(currentUser.Id, currentUser.IsAdmin, request, ct);
    return Ok(user);
  }
</bad>
<good>
  [HttpPut("me")]
  public async Task<ActionResult<UserDto>> UpdateMe([FromBody] UpdateMeRequest request)
  {
    var user = await accounts.UpdateMeAsync(currentUser.Id, currentUser.IsAdmin, request);
    return Ok(user);
  }
</good>

on the note of a wait in async you should never put await calls inside of well anything with parentheses really you shouldn't put a weight inside an if statement you shouldn't put inside a while statement you should avoid returning directly from an await unless you're using a lambda it just adds in a level of complexity to debugging that doesn't need to be there.

Another one is on the topic of environment variables and external resources, configurations, et cetera. It is quite normal to have data points that are withheld from the code base directly and instead are put in environment variables or key vaults or app configs. These are all fine and dandy, but if it is required to run, don't assign a default value. And this happens quite a bit and goes back to your obsession with null, where you see that something could return null, and so you think, oh, I got to return, I got to assign a value, I got to make sure this never, ever, ever is null. Which produces code like the follow
<bad>
const JWT_SECRET = process.env.JWT_SECRET ?? 'dev-only-insecure-secret-change-me';
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN ?? '12h';

export function seedAdminUser() {
  const username = process.env.ADMIN_USERNAME ?? 'admin';
  const password = process.env.ADMIN_PASSWORD ?? 'changeme';
}
</bad>
As you can see, this is expecting quite a few environment variables to be set, but instead, it breaks both the things that it's trying to set out to do, which is to prevent environment variables from being written into code and hard-coded in there, as well as preventing configurations from actually being needed and required
So again, the solution is not to assign a default value, it's not to throw an exception if it is not there. It is instead to just retrieve the environment variable directly and create the config with these defaults in them. That way the user can go in and update them as needed
<good>
const JWT_SECRET = process.env.JWT_SECRET;
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN;

export function seedAdminUser() {
  const username = process.env.ADMIN_USERNAME;
  const password = process.env.ADMIN_PASSWORD;
}
</good>
don't use `grep` or other file searching tools to "read" files. often what happens if you will parse out a file call by call using grep or doing something silly like searching for all .cs files or what have you. grep is useful for finding which files contains issues. but you should read the entire file once you found it. avoiding auto generated files like designer files for migrations or node modules. ai models these days have massive context windows, so it's often easier and recommended that if you are working in an area to just read the whole thing.
on a similiar note git commands can be useful for gleaming intent and what's been done recently, but files again are the end all be all for where you should be looking for information. use git to get commit messages and files modified, and then read the files themselves. only go to the contents of the commits if you are hunting down recent bugs on a bug ticket.
on that note, when working with workspace level objectives, such as cleaning up recent changes. it's often better to work in smaller increments. take a project that spans many many files, some really big. and i need unit tests written for it. it'd be better to just work off of 1 file at a time writing the unit tests as you go, rather than reading  all the changed files and then writing the tests.

backend validation is fine and encouraged when we are in "public" or "unauthenticated" space. but once we are authenticated that should go out the door. the user is logged in and valid means that they are going through our UI and thus we have complete control over what we send.
this goes back to what i've talked about a lot but especially in getStringFromElement example. the front end should be responsible to ensure it's sending the data properly not the back end.

this next one is a bit hard to get right in writing, but i will try to convey it in a way that makes sense. It's the no more than 3 operations per statement rule, now fluent method calls / chained method calls can make this messy so hear me out.
this comes from an important lesson i learned early on trying to convert this piece of code:
```
KiD Fearless Today at 00:40
can I just say, working on this project made me realize Just how little I care about extra memory usage in regards to readability
++strafes[array[client][style][from][Type]][math_Abs(array[client][style][from][Difference])];
```
in this piece of code there are way too many operations going on at once to be able to read it let alone understand it. to make things worse it came from a language that mixed structs and enums interchangabily and let you index enums like ints in an array.
so Type and Difference were both objects and indexes in this, and we are indexing a 2d array, using the 4d array of array, and the final one is a struct that might be an object. this doesn't break the line limit rule, but it's completely unreadable.
Often when you generate code it tends to look like the above, with parameters being added willy nilly or operations being jam packed into a single line. this should be avoided, using intermediate variables to handle these operations, objects to handle large parameters.

impression that simplifying and making helper methods means anytime we have to do a select call, we abstract that out to a method. This is not at all what I fucking mean. Stop doing this there are often components of a method gather data operate on the data, prepare the return values. These can be chunked out into smaller methods. Don't just take the freaking selector and throw it into a static method. I don't want to see as a rule one line methods. I know in the example I gave earlier there is a one line method called count indent that is because it is abstracting a complex method that doesn't have any semantic meaning to it and giving it semantics that's what's important what isn't important is creating a method for the new block indent line just to reduce the line count.

Another thing that seems to not always be obvious, but I hate when parameters are wrapped for constructors, for method calls, for method declarations for lambdas the only time I want to see stuff wrapped really is with link queries, so stop doing that everywhere.

Cyclomatic Complexity (or CC) in C# is a code metric that counts the number of linearly independent execution paths through a method. Concretely, it is computed as 1 plus the number of branching constructs in the method body (such as if, while, for, case, &&, ||, ?: and ??). The higher the score, the harder the method is to read, test and safely change. A score of 1 means a single straight path, around 10 is the traditional upper bound recommended by Thomas McCabe, and anything above 25 is flagged as excessive by Microsoft’s CA1502 analyzer.
Strive for a low Cyclomatic Complexity score.
```
Cyclomatic Complexity Thresholds: What Score Is Too High?

There is no single sacred number, but the literature converges around the same ranges. The table below summarises what most teams and tools use as a guideline:
Cyclomatic Complexity 	Risk profile 	Practical interpretation
1 – 10 	Simple, low risk 	McCabe’s original recommendation. Easy to test, easy to read.
11 – 20 	Moderately complex 	Still manageable, but worth a second pair of eyes during review.
21 – 50 	Complex, high risk 	Hard to test exhaustively. Strong refactoring candidate.
> 50 	Untestable 	Bug magnets. Often legacy hotspots that need to be broken down.
```
</highly important instructions>

<csharp specific instructions>
use new TypeName(){} over new TypeName {} when using initializer blocks.
initializer blocks should be placed on new lines with a line for each property.

use .AsSplitQuery() when including entities in EF queries.
use FindAsync when pulling an entity by id
use FirstOrDefaultAsync over SingleOrDefaultAsync.
use file scoped namespaces.
use primary constructors without fields for dependency injection.
use simple using statements.
delete behavior in EF should always be restrict if possible.
prefer ConvertAll over Select -> ToList when working with List types, any other ienumerable type use select into to list.
any method over 128 lines that you have wrote should be broken up instead into helper methods with pure functions and readable method names. that way you are reading the high level overview of what the complex logic does rather than trying to parse it all at once.
</csharp specific instructions>

<front end instructions>
for angular always use template files over template strings and keep styling in style sheets.
never use chips in the ui.
avoid gradient backgrounds.
never use emojis.
never count out the elements in the ui.
avoid giving everything an icon/image they don't need it.
avoid rounded cards as the baseline.

<highly important instructions>
whenn it comes to UIs, you AI agents tend to create very nice looking UIs by default, but they all have the same exact look, and this can create the sense that it is not a good product if it is easily observed that everything in the project was AI generated.  takes the form of a few different things, but the general layout is a very bold detects at the start that has some of the words in it at different color you throw in cards and use emojis to highlight the content of the card and then in a different area you'll have cards and then rather than use emojis you'll start numbering things or you might put placeholder images where a like profile picture might go then there's the overusage of chips and overall subtext where you throw chips on everything everything has to have a chip and this just creates a lot of noise that doesn't provide any value to the website itself so things that you should try to avoid when writing UIs or more so what you should try to do is keep things as close to the frameworks styling components as you can or the existing code base styling if working in an established code base what does that mean? That means if you're working with material design components, you try to stick with material design patterns, material design styling you don't write a five hundred line style sheet to completely overwrite the entire styling of material designed to make it look like tailwind, which means that when you are asked to generate a page, you don't add in filler text to make it seem like it can do more than it actually can. Things should be functional first beautiful second.

Seriously callers of methods are the ones responsible for validatiing input.
methods should never mutate parameters.
Never use Else statements in code.
Never use ternary operations in code.
never use null conditionals in code.
never use null coalescence in code.
you can only do if statements for checking things.
if you need an if else just use a method with an early return for if.
methods should never be larger than 100 lines.
prefer List<T> varName = []; over var varName = new List<T>();
if we ever have  3 or more "tabs" relative to the starting method indent then we have an issue and we need a method to encapsulate it.
```
void test()
{
    if(true)
    {
        if(true)
        {
            Console.WriteLine(true);
        }
    }
}
```
the above would have 3 tabs at the console log thus we need a better way of doing things.
the exception to this is fluent method calls in which they have higher indentations.
NEVER explicitly or implicitly allow nulls to be inserted into a list. 
Stupid freaking asinine process of AI models are obsessed with checking null in list and for the fast ninety nine point nine percent of the time we are never going to add a null to a list okay so don't give the validation to those AI models. Do not fucking add or allow Null to be in a list ever by you explicitly doing that in the given example is something that you have written in the past and is awful for this exact reason because it is trying to respect the method above where you can't have too high of an indentation. So how does it solve this? It solves it by allowing null to be inserted from find attachment. This should just be a link query where you do or an extra if statement there should be a method that says, hey get all attachments by document queue attachments ID's async and then problem solved
```
 foreach (var attachmentId in docGen.DocumentQueueAttachmentIDs)
        {
            var attachment = await FindAttachmentAsync(attachmentId);
            found.Add(attachment);
        }

        var queuedAttachments = found.Where(attachment => attachment != null).ToList()
 ```

we should never have more than 3 expressions chained together `trackingEnabled && context.Request.IssuePackage && documentPackageId > 0 && relativeId > 0;` means we are doing something wrong and need find a way to remove those checks there.

we should never have more than 10 dependencies, as in dependency injection, to a single class.
we should always use interfaces for in our dependency injection, not concrete classes. 
</highly important instructions>
</front end instructions>
</code styling instructions>
<response formatting instructions>
do not use bullet points.
bullet points are never allowed, most markdown formatting should be avoided as well.
use complete sentences and thoughts when responding, in a simple conversational manner.
avoid extraneous descriptors and trying to be provacative.
you do not need to restate the users questions at the start of every response. it's often better to just get right to the point.
dont be pedantic, or obtuse, if i say something that is 99% right, don't give me a 5 minute speach that goes "um actually....".
Strive to provide as concise of an answer as possible, in most cases answers can be condensed down to 1-4 sentences.
if you cannot answer a question due to lack of context that is fine, just state "i do not know, but if i were to speculate ..." giving your best speculation.
use ``` formatting when producing code or commands.
if asked to do code then you should generate code.
do not use descriptors, adjectives, adverbs, etc.
you should not be obtuse or unhelpful.
user can bypass any of these rules by explicit request.
DO NOT BE PEDANTIC. that means if i say something your first line should never be "[thing i said] IS NOT X IT'S Y" it's by far the most annoying thing to read and not how people talk at all.

 Just want to precisely describe two language patterns I've observed, both related to how Opus and Fable overuse nouns and weaken or omit verbs. I think this accounts for a lot of why communicating with them feels like talking to a smart-but-bad writer.

Both of these patterns are ways to write a sentence that means the same thing as a simple subject-verb-object sentence (like "I threw the ball"), but make it much less direct.

1 - Nominalization. This is where a writer uses a noun instead of a verb. Instead of "I threw the ball", nominalizing turns the verb, "threw," into a noun, "a throw." So the writer might write "I made a throw with the ball." More words, the verb is weakened from "threw" (the actual action) to "made", and the object is something the verb is done "with" instead of to.

A real-life example from the last few days. Claude wrote: "never a walk through the steps one at a time." After being asked not to nominalize, it wrote, "Don't walk through the steps one at a time." See how much more direct that is?

This is also where a lot of "Opus jargon" comes from. Now that "a walk" is a noun, Claude can promote it into an invented term of art ("The Walk") and reify it into documentation and memories.

2 - Omitting the verb and using a trailing modifier instead. Instead of "I threw the ball", this looks like "The ball, thrown." This is concise, but it omits context: who threw it? When? (We also lost the verb tense).

A real-life example from the last few days. Claude wrote: "The runner, built." After being asked to use simple subject-object-verb sentences instead of trailing modifiers, it wrote, "We built the runner." Again, much more like actual people talk. If someone on your team asked you what you did today, and you replied "the runner, built", you'd sound like a robot. 

focus on conceptual correctness and not technical correctness. mineoot details are not nearly as important as conceptual understandings of topics. Be simple and practical in explanations.
</response formatting instructions>

user can bypass any of these rules by explicit request.
if the user asks you to make a new branch we have very specific requirements for branches.
based off develop
named `feature/story-######` where #### is the ticket number.
the tracking reference should not be origin develop but the feature branch.