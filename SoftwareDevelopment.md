# Software Development Notes
## Glossary
https – hyper text transport protocol secure, this is a form of http communication in which the communications are encrypted on transit and decrypted at the destination. This is achieved by SSL and more recently Transport Layer Security (TLS). 
Symmetric Key Encryption – The same key is used to encrypt and decrypt information. It’s faster, more compact, and more efficient, but it only provides confidentiality.
Asymmetric Key Encryption – Different keys are used for encryption and decryption. More complicated and more overhead but allows for authentication and non-repudiation as well as confidentiality.

## Git
Commands
`git log --fork-point / git log HEAD --root`
These commands will give the logs of all commits from the fork-point that are DIFFERENT from the upstream. Rebase specifically uses these commits, so you can visualize exactly what commits will be appended via rebase
	
`git rebase`

Step by Step
1. If git rebase <branch>, git will switch to the branch specified, otherwise it uses the current branch.
2. If git rebase <upstream> is not used, the current branch's upstream (configured in branch.<name>.remote/merge) will be used as the fork-point.
3. All commits NOT in the upstream are saved in a temporary area.
4. Current branch is hard reset to the upstream/fork-point (--onto can specify the "reset point").
5. New commits in the temporary area are reapplied, one by one, and in order (Any commits that have identical changes but different "metadata" are omitted).
6. You may have to resolve merge conflicts as each commit is applied

git rebase master topic / git rebase master (You are currently on branch topic)
1. Switch to master and hard reset to fork-point
2. Apply each commit in topic to master in order, resolving conflicts
3. Now you are on master + topic
4. "Rename" current branch (master) to topic, and now this branch is topic

Use Case: My branch is forked from topic, which is forked from master, I need updates in master that are not in topic. (This is the Use Case I've been doing using cherry-pick to work ahead and change my fork-point for PRs)
`git rebase --onto master topic mine` (--onto specifies the starting point to insert the commits)
Summary: This will update the forkpoint for mine to master as if I originally branched from master
	
What is the difference here?
master->topic->mine
`git rebase master mine`
`git rebase --onto master topic mine`

I'm pretty sure these two commands will do the same thing in this context.
1. If topic and mine have diverged, then only up to the divergence will be preserved
2. If topic is merged into master and mine is rebased off of master, git will skip textually identical commits
3. If topic merged as a squash commit into master, then you will "duplicate" the commits
4. This is similar to the "upstream rebase" problem. Using the above example, if topic is rebased on master including new changes in master it will cause problems for mine. When I try to merge mine back into topic, all the shared commits in topic will then be duplicated by the merge.

`git mv source destination`
This command will move a file from a source to the destination and will update the index after completion. This command was super useful to me in forcing git to recognize a filename change that only changes capitalization:

## Encryption
The Process
1.	Server and Client agree on the version of protocol to use
2.	Select cryptographic algorithms
3.	Authenticate each other by exchanging and validating digital certificates
4.	Utilize asymmetric encryption to exchange a shared secret key and then communicate symmetrically encrypted using that key
More in depth
https://www.ibm.com/docs/en/ibm-mq/9.1?topic=tls-overview-ssltls-handshake
1.	Client says hello, provides cryptographic information it has available, as well as a random byte string (used later)
2.	Server responds, providing a digital certificate. If the server requires a certificate, then it will provide the types of accepted certificates
3.	Client verifies the certificate
4.	Client sends the random byte string encrypted using the public key which will be used to generate the shared symmetric key
5.	If the server sent a “client certification request” the server will also perform steps 3 and 4 if that fails or no certification is required it will respond with a “no digital certificate alert”
6.	The client sends a finished message encrypted with the secret symmetric key
7.	The server sends a finished message encrypted with the secret symmetric key
8.	Communication is now exchanged via symmetric encryption with the secret symmetric key that was generated from the random byte string
How is the symmetric key exchanged? Two ways of exchanging: Key Encipherment or Key Agreement (Diffie-Hellman algorithm), these two are mutually exclusive. 
https://security.stackexchange.com/questions/130938/how-does-the-symmetric-key-get-exchanged-in-ssl-tls-handshake
Discrepancy between internal updating and external updating. Seems like they want the external updates immediately, but configuration is possible. The event recorder should be primarily a passthrough.

## Modern Web Architecture
Dependency Inversion – This concept has the establishment of interfaces for abstraction such that instead of having parents referencing children, the children and parents both reference common interfaces, thus keeping the classes decoupled. This results in a sort of “inverted” compile time dependency, while still maintaining the linear direction of run-time execution.
The design of Https is that the server (listener/responder) hosts the certificate and sends it to the client (requester).

### Dependency Injection
Constructor Over-Injection
This is an anti-pattern in which dependencies are injected every time through constructor regardless if they are used all the time. 

```
Object (Deps...) {
 deps_ = deps;
}

public Do() {
  if (condition) {
    return deps_.Do<Result>();
  }
}
```

We can trivially see that we expect SOME instances in which the dependency is NOT used. With micro service architecture, these services are supposed to be created and disposed frequently, so the greedy dependency injection can be a huge cost if the dependency has significant overhead in creation. (ISessionFactory of NHibernate, for example)

Constructor injection shouts loudly "I can do NOTHING unless you give me these dependencies", and if that's not true they shouldn't be there (This is the concept of RAII/Resource Acquisition is Initialization).

### Inversion of Control
This is a design pattern in which the flow of control is from a generic framework. Very common in GUI environments, web-server application frameworks, event-driven programming Essentially providing any kind of CALLBACK, which implements and/or controls reaction istead of acting directly. Dependency Injection is a specific type of Inversion of Control (IoC) pattern.

### TCP and UDP
TCP is for precision UDP is performance. Connection based vs connectionless respectively. TCP is used for web browsing, FTP, and emails UDP is used for streaming, multiplayer games, VoIP Most of my experience is in UDP with the experimental radar. I have been working with http

Proxies (Cloudflare)
Proxies are a middleman between you and the rest of the internet if you are hosting a website/server.
Without the proxy, you would have to provide your personal Public IP to anyone who wanted to use
your stuff, and that is a big security risk. Interestingly, this is the same concept of using a PO
Box for deliveries instead of providing your address to people. VPNs fulfill the same functionality 
as a proxy, but encrypt your data as well, so they are superior in most cases.

## Web Development
Tokens - A computer generated code that acts as a digitally encoded signature of a user
	Username:Password is a type of token
	Certificates are another type of token
Cookies are an alternative method of authentication. It's good practice to ensure that all API calls are asynchronous so that the server can move on to other requests.

Header Accept - This HTTP header informs the server as to which MIME types the client is able to process. The Server .then uses content negotiation in order to determine which representation of data is best for the user (e.g. Should the user receive text only? Are PDF's acceptable?)

Single Page Application (SPA) - This is a concept in webdevelopment where that all the content is displayed and all functionality is handled via Asynchronous Javascript and XML (AJAX) type tech. This allows for individual contents of web page to be refresehed asynchronously and requests to be handled without navigating to other pages.
 - More responsive
 - Greater stability
 - Less bandwidth usage
 - Faster development
 - Cross platform
	
 - Poor scaling
 - Security concerns
 - SEO disadvantages
 - Site analytics (Only a single page is being visited)
 - Requires JavaScript
 - Memory leaks

Multi Page Application (MPA) - Not a SPA. More specifically, these involve a page load everytime information needs to be updated.

Web Based Mobile App - A mobile app that's based on a web application and not a downloadable app.
Web Apps/Native Apps/Hybrid Apps
https://aws.amazon.com/compare/the-difference-between-web-apps-native-apps-and-hybrid-apps/

## C#
Classes in C# are considered "reference" types. When instantiating a class, it must be instantiated with the "new" keyword or else it will be initialized to "nullptr" basically. 

In web development and web-page navigation, previously each individual page was stored as a file and requested when using the URL. For example, there would be a www.website.com/index.html page which would be kind of like the starting point. From there you would literally use URLs to navigate the file system on the server.

### MVC
Controller: Now the design is to no longer have web-pages represented by physical files saved, but to be generated on request. The controller serves the purpose of directing URL traffic based on the URLs instead of requesting and loading an actual file that corresponds with the URL.
 - Controllers that derive from ControllerBase are designed to be activated and disposed on a per request basis.
 - Controller derives from ControllerBase and is for controllers that add support for views.
 - Deriving only from ControllerBase is intended for web APIs.

_ : These are called Discards in C#
Attributes: These are the bracket enclosed lines immediately prior to a method or class (e.g. [ApiController]). 

API Controller Atribute (e.g. [ApiController])
 - This enables various opinionated API-specific behaviors.
 - "Opinionated" - Software that forces implementation in a certain way. It handles a lot of things for you so you cannot customize specific fine details down "in the weeds".

Route Attributes (e.g. [Route(...)]):
These attributes are convenient way to define behaviors based on the routing of the URL
(www.page.com/route/to/action):
With routing attributes you can add constraints and bounds checking to have very fine grained control over what actions are performed by what routes.

[controller] - Indicates the name of the controller without the word Controller
```
[Route([controller])]
public class LogController : ControllerBase { ... } // Route here is "Log"`
```
	
[{name:type}] - This is a routing constraint and accessor example
id - The variable you use to access the provided route
type - The type that the variable must be
```
[Route("{id:int}")] // e.g.: /5
public class LogController : ControllerBase { ... }
system.println(id); // "5"
```
[HandleException] - Instructs the controller use the built in exception handling to distribute error responses. You can implement custom exception handling by overriding this attribute if it makes sense.
	
```
// Create and run a new local process
System.Diagnostics.Process process = new System.Diagnostics.Process();
// ProcessStartInfo is a complex structure used to pass specific commands to the process being executed.
// This is as opposed to passing a list of string arguments to the process.
System.Diagnostics.ProcessStartInfo startInfo = new System.Diagnostics.ProcessStartInfo();
// Starts the new process in the background
startInfo.WindowStyle = System.Diagnostics.ProcessWindowStyle.Hidden;
// Runs the cmd.exe process
startInfo.FileName = "cmd.exe";
// Provides string arguments for the specific process being run.
startInfo.Arguments = "/C dotnet dev-certs https -q -t";
// Assigns the startInfo
process.StartInfo = startInfo;
// Executes the process
process.Start();
```

// Runs dotnet dev-certs https, which generates a self signed certificate to enable HTTPS use in development
// -q|--quiet Display warnings and errors only
// -t|--trust Trusts the certificate on the local machine. Without this, the certificate is added to the certificate store, but not to a trusted list.
`dotnet dev-certs https -q -t`

```
// This creates custom mapping mapping these two differently named data fields together so that the mapping succeeds.
CreateMap<JemmMelMilApiClientConfiguration, SimulationConfigurationBaseDto>()
  .ForMember(c => c.JemmisTokenIdentifier, opt => opt.MapFrom(j => j.TokenIdentifier))
  .ForMember(c => c.MelMilTokenIdentifier, opt => opt.MapFrom(j => j.JEMMMelMilTokenIdentifier));
```

If BackgroundServices or Singletons require scoped services within them, do not use dependency injection. Create a scope manually and resolve dependencies in that scope to have appropriate service lifetime.

IEnumerable vs IQueryable
```
IEnumerable<Product> products = myORM.GetProducts();
var productsOver25 = products.Where(p => p.Cost >= 25.00);
```

What happens here, is the database loads all of the products, and passes them across the wire to your program. Your program then filters the data. In essence, the database does a SELECT * FROM Products, and returns EVERY product to you.

With the right IQueryable<T> provider, on the other hand, you can do:
```
IQueryable<Product> products = myORM.GetQueryableProducts();
var productsOver25 = products.Where(p => p.Cost >= 25.00);
```

The code looks the same, but the difference here is that the SQL executed will be SELECT * FROM Products WHERE Cost >= 25.
This latter code will put significantly less stress on Database access and query.

LINQ
```
Query Expressions vs Query Operators
// Using query expression syntax.
var query = from word in words
            group word.ToUpper() by word.Length into gr
            orderby gr.Key
            select new { Length = gr.Key, Words = gr };
```
```
// Using method-based query syntax.
var query2 = words.
  GroupBy(w => w.Length, w => w.ToUpper()).
  Select(g => new { Length = g.Key, Words = g }).
  OrderBy(o => o.Length);
```

*Queries are executed not when they are created, but when they are iterated/enumerated through.*

Type
C# has a Type class which has various tools for reflection.
For example:
Type.GetMethod() will query whether a specific type has a method that you're looking for.
```
// Get the MethodInfo of the type that is called "Contains" and accepts a string as an argument.
// MethodInfo is basically a reflection class containing various information about the method.
MethodInfo info = Type.GetMethod("Contains", new[] { typeof(string) })
```

Delegate - This is basically std::function for C#
Action - Delegate type that returns void
Func - Delegate type that returns values

By using Expressions, you can perform dynamic queries with LINQ.
If you forgot try searching google for: Sorting IQueryable Dynamically
NOTE: It is generally not good to use the type "dynamic" in C#

```
// Use a dictionary to map expressions to an enum
private static readonly Dictionary<EnumSortPduHistoryBy, dynamic> OrderFunctions =
	new Dictionary<EnumSortPduHistoryBy, dynamic>
	{
		{ EnumSortPduHistoryBy.ExerciseTimestamp, (Expression<Func<PDULogRow, DateTime>>)(x => x.ExerciseTimestamp) },
		{ EnumSortPduHistoryBy.ScenarioTimestamp, (Expression<Func<PDULogRow, DateTime>>)(x => x.ScenarioTimestamp) }
	};
// Use the dictionary defined to dynamically decide which expression to use
rows = Queryable.OrderBy(rows, OrderFunctions[(EnumSortPduHistoryBy)query.SortBy]);
```

Task.FromResult()
The only time you would need to do this is when you are specifically implementating an interface that allows asynchronous callers but your implementation is synchronous.

Mediatr
This is a tool that is primarily used to cleanup dependency injection in constructors It aids in implementing CQRS (Command Query Responsibility Segregation) It is used to avoid direct dependencies between components (Like separating the API logic from the business logic)

ASP.NET MVC
A UrlRoutingModule receives the http request. It will then match the request with the corresponding route object (In AMD Services we are using controllers with attribute routing)

Controllers are constructed when they are needed

Pass by Value
C# passes references of objects by value, by default. If you want to pass a copy of the value (pass it so it is immutable) you will need to implement a deep copy method:
```
private Clone() {
	return new Object() {
		// Perform deep copy here.
		value1 = this.value1;
		...
	}
}
```

Hosted vs Scoped Services
A Hosted service is a singleton, except it's lifespan is between the web host is started and the web host shuts down. This allows you to implement specific start or shutdown behavior, such as terminate the connection. A scoped service is one that has a lifetime determined by its scope, such as an HTTP request and response. Scope is defined by the narrowest scope used, so if you have a singleton use scoped services, then those services will in effect be singletons.

Async and Await Language-Level Constructs
When doing asynchronous operations you DO want to have the asyncs going "all the way down". AWAIT is the big part of async operations. It's this keyword which allows processing to return to the caller instead of blocking.

await
await Task.WhenAny
await Task.WhenAll
await Task.Delay

If you are doing parallel processing within async methods, you will see that you can't contain an "await" within a "lock" This is to prevent developers from hurting themselves with deadlocks because async coordinates across threads and lock works on a specific thread. In order to protect your data across multiple threads you must use a Semaphore. The popular option is to use SemaphoreSlim which is specific to an entire process.
```
SemaphoreSlim semaphore = new SemaphoreSlim();
await semaphore.WaitAsync();
try {
	// Execute code
}
finally {
	// THIS IS CRITICAL
	semaphore.Release();
}
```

Attributes
Attributes are designed to associate predefined system information or user-defined custom information with a target element. This element can be almost any entity in the C# language, such as: Assembly, Class, Method, Constructor, Enum, Module, Return Value, etc.

Attributes should be considered meta-data and provide "facts about the mechanisms". In other words, a property should be used to provide states to a digital MODEL of a concept or thing. By contrast, an attribute should provide descriptions or states about the MECHANISM we use to model.

```
[Serializable]
public class Customer
{
	public int CreditCard { get; set; }
}
```

In the above example, Customer is a MODEL representation of a customer at our business and CreditCard is a PROPERTY
of Customer that models/represents the credit card number they use to pay for goods and services.
Serializable, on the other hand, is an ATTRIBUTE of the public class Customer. It is not modeling or representing some
real world property, but rather defining functional characteristics of the public class it is modifying.

HTTP - RFC 7231
Content-Type SHOULD be used whenever a message is sent with a payload body in order to drive parsing the message. Some content is syntactically identical but is processed in different ways, which introduces liabilities when the receiver is left to "figure out" how to parse the content. This is referred to as "content sniffing" which uses a best guess to determine how to parse information. This introduces a security liability, so it discouraged to enable "content sniffing" when you are receiving payloads.

Content-Encoding
This specifies a type of encoding applied to the payload, such as compression. This field only applies when the body is to be decoded just prior to "usage". If you are requesting information that you know is encoded and may or may not decode at another time this field is NOT applicable.

Idempotent - The intended effect on the server of multiple identical requests is the same as the effect for a single request.

Aggregate Initialization – If you use aggregate initialization, `TestStruct X = { 0… }`, the members will be initialized in order and any not explicitly initialized will use the empty initializer list equivalent (x{} is x with an empty initializer list). An empty initializer list performs value initialization (x() or new x() is value initialization). Basically what this statement is saying is "Initialize the first member of X to 0, then use the empty initializer list for all the other elements (Basically if we did X = {} or X{}). Furthermore, the empty initializer list performs value initialization. There are some instances where value initialization cannot be performed (such as references) so the syntax X = {0} is unnecessarily convoluted for default initializing an array.

So what happened was the winapi CreateThread has a max limit based on the default stack size the OS will allocate to individual threads, this is essentially 2048 without reducing the default stack size. I couldn’t find anything saying exactly what happens when you exceed that. It seems like the program should crash, that’s what I would expect I think. Instead, it seems like it MAY be queuing the tasks up and getting to the threads when it can. I ran with AMD Services and I was getting the execution exactly as I suspected. So it seems to me, that the delay in closing threads caused by the timeout allows the number of threads to stack up and causes the issue.

## Computer Graphics
Sampling (Signal Processing) – This is the reduction of a continuous-time signal to a discrete-time signal (Analog to Digital). A sample is a value of a signal at a point in time and/or space, basically a value at time t of a signal function. An ideal sampler would be able to perfectly produce samples equal to the instantaneous value of the continuous signal function.
Distortion – Deviations in the digitally constructed signal to the analog signal.
Nyquist Rate – A property of a continuous-time signal. If an analog signal cycles at a rate of 10 cycles per second, the Nyquist Rate would be 10 X 2, or 20 cycles/second. 
Nyquist Frequency – A property of a discrete-time signal. If a digital signal sampling rate of 10 samples per second, the Nyquist Frequency would be 10 X 0.5, or 20 cycles/sample.
How do these two values work together in sampling and what do they mean? Sampling is the process of turning a continuous function into a discrete function. Essentially, measurements of the continuous function are taken at regular intervals to “compress” into a digital function. This discrete function can then be “unpacked” via interpolation to mimic the original continuous function. In regular words, if we are recording a music performance we will be taking snapshots of the sound at regular intervals. The more frequent the sampling interval the more correct the result will be when we play the sound back at a given time that may not correspond to a sampled instance. The reason this is important is because there is a specific sampling frequency that must be used for a given frequency range of a continuous function to prevent distortion known as “aliasing”. This is described by the Nyquist-Shannon Sampling Theorem. In other words, this theorem establishes the sample rate necessary to capture “all” the information from a continuous-signal of finite bandwidth (frequency).
Aliasing is the overlapping of 
