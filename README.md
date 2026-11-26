1. Choosing the Right SDLC Model
(B) Food-delivery startup app: Agile/Scrum, or RAD.
•	Requirements are unstable because the market, competitors and user feedback keep changing.
•	Risk tolerance is higher. A bug in a delivery app is an inconvenience, not a catastrophe.
•	Short sprints produce working releases quickly, so you get real user feedback early and testing happens continuously.
How the three factors decide it: stable requirements favor Waterfall, while changing ones favor Agile/RAD. Low risk tolerance favors Spiral or Waterfall, while high tolerance allows Agile. Late, one-time testing only works when requirements won't change. Continuous testing is needed when they will.

2. The Hidden Trade-offs of RAD
RAD trades depth for speed, and the cost shows up later:
•	Prototypes become the product. Without extensive upfront planning, there is no solid architecture. The quick prototype gets patched and extended instead of redesigned, which creates architectural debt: tight coupling, duplicated logic and no clear layers.
•	Reduced scalability. A prototype built for a few users and a small dataset was never designed for high load, large data volumes or growth. Retrofitting scalability later means rewriting core parts.
•	Reduced reliability. Time-boxed delivery cuts scope and testing to hit the deadline. Because there is no deep risk analysis, failure modes in mission-critical systems (security, concurrency, recovery) are never examined.
•	Heavy dependence on users and modularity. RAD needs high user availability for constant feedback. It also needs a system that splits cleanly into modules, which large, tightly integrated systems often don't.
That is why RAD suits small to medium, non-critical systems and not large-scale or mission-critical ones.


3. Balancing DRY, KISS, YAGNI and Separation of Concerns
Yes, trying too hard to follow DRY or Separation of Concerns can violate KISS and YAGNI. These principles are trade-offs to weigh, not rules to apply mechanically.
Hypothetical over-engineered example: a program that reads two numbers and prints their sum. A developer, determined never to repeat code and to separate every concern, builds:
•	an IOperation interface, an OperationFactory, and AddOperation, SubtractOperation and MultiplyOperation classes
•	a separate InputReader service, an OutputFormatter service and a Logger
•	a dependency-injection container, plus separate UI, Business Logic and Data Access projects
•	a config file that selects the operation at runtime
The result is about 10 files and several hundred lines for a task that needs 5 lines.
•	KISS is violated: the design is far more complex than the problem.
•	YAGNI is violated: nobody asked for subtraction, multiplication, pluggable logging or a data layer. They were built "just in case" and may never be needed.
•	DRY was misapplied: nothing was actually duplicated, so the abstractions removed no repetition.
Takeaway: abstract when real duplication or real change pressure appears (for example, the "rule of three"), not before.
4. The Economics of Testing and Code Quality
Why error-correction cost is lowest early and highest after deployment:
•	A defect found in requirements or design only needs a document or diagram to be changed.
•	A defect found during coding means rewriting the affected code.
•	A defect found in late testing means rework in design, code and tests, because other code has been built on top of the flawed part.
•	A defect found after deployment adds patches, redistribution, downtime, data repair, support costs, lost customers and reputation damage.
Defects compound: a flawed requirement becomes a flawed design, which becomes flawed code, which becomes flawed tests. Cost often grows roughly 10x or more per phase (Boehm's cost curve).
How these practices keep cost low:
•	TDD: writing tests before code forces clear requirements and testable design. Each bug is caught minutes after it is written, and the test suite acts as a regression safety net.
•	Continuous Integration: code is merged often and built and tested automatically, so integration defects surface within hours, not at the end of the project.
•	Peer code reviews: other developers catch logic errors, design flaws and standard violations before code is merged. Reviews also spread knowledge across the team.
All three shift defect detection left, to the cheap end of the cost curve, unlike Waterfall's late testing.
5. MSIL, JIT and the Platform Independence Paradox
MSIL is independent of the CPU, but not of the operating system, which is why .NET Framework apps are still tied to Windows.
The flow:
1.	C# source code goes through the compiler.
2.	The compiler produces MSIL, CPU-independent intermediate code with metadata.
3.	The JIT compiler converts the MSIL at run time.
4.	The result is native machine code for the current CPU.
Why .NET Framework apps are still platform-dependent:
•	The JIT and the CLR runtime that run the MSIL exist only for Windows.
•	The Framework Class Library depends on Windows-specific APIs: Win32, WinForms, the registry, GDI+ and COM.
•	So the same assembly runs on any CPU, but only on an OS that has a compatible runtime and libraries.
How Mono changed that: Mono is a third-party, open-source implementation of the CLR and the class libraries for Linux, macOS and other systems. It supplies its own JIT and cross-platform equivalents of the libraries (for example GTK# instead of WinForms). The same MSIL assemblies can therefore run on non-Windows systems. The code didn't change, but the runtime underneath did.
6. Managed vs. Unmanaged Code and Language Interoperability
Managed code is protected by the CLR at run time, and C# and VB.NET can share one application because both compile to MSIL and share the CTS.
How managed code in the CLR is protected, compared with unmanaged Win32 code:
Problem in unmanaged (Win32) code	How the CLR protects managed code
Manual memory management (malloc/free) causes leaks, dangling pointers and double frees	Garbage collection reclaims unused objects automatically
Raw pointers and unchecked casts allow memory corruption	Type safety: verified IL, no arbitrary pointer arithmetic, no unsafe casts
Buffer overruns, a classic security hole	Array bounds checking throws an exception instead of overwriting memory
Weak error handling	Structured exception handling and runtime security checks
In unmanaged code, one mistake can crash the whole process or open a security hole. The CLR catches the same mistake as a controlled exception.
How C# and VB.NET share one application without data conflicts:
•	MSIL: both compilers produce the same intermediate language, so the CLR sees no difference between them.
•	CTS (Common Type System): it defines one shared set of types and rules. C# int and VB.NET Integer both map to System.Int32. C# string and VB.NET String both map to System.String. Every type ultimately derives from System.Object.
•	Metadata: each assembly describes its types, so one language can use classes written in the other directly. For example, a VB.NET class can inherit from a C# class.
Because both sides use the same underlying types, no data conversion or marshalling is needed between them.

7. Solving "DLL Hell"
.NET avoids DLL Hell by giving every assembly its own identity and version, so applications stop fighting over one shared copy.
The old problem: applications shared DLLs in a common folder (for example System32) and relied on the registry. Installing App B could overwrite a shared DLL with a different version and break App A, because only one version could exist at a time.
How .NET solves it:
•	Assemblies with manifests and metadata: each assembly describes itself with its name, version, culture and the exact versions of the assemblies it references. No registry entry is needed.
•	Private assemblies: these live in the application's own folder, so each app uses its own copy and nothing is shared. Deployment is just a copy (xcopy).
•	Strong names: name + version + culture + public key + digital signature give an assembly a unique, tamper-proof identity.
•	GAC (Global Assembly Cache): this is the central store for shared, strong-named assemblies. Several versions can sit side by side, and each application binds to the version recorded in its manifest.
Why Namespace and Assembly are not interchangeable:
•	A namespace is a logical grouping of types. It organizes code and prevents name collisions (for example System.Collections).
•	An assembly is a physical unit of deployment, a .exe or .dll file with a manifest.
•	One assembly can contain several namespaces, and one namespace can span several assemblies. A using statement imports a namespace but does not tell the runtime which assembly file to load.
8. Abstraction in Data Access: ADO.NET vs. Entity Framework and LINQ
Entity Framework and LINQ raise productivity and safety, at the cost of control and sometimes speed.
Why raw DataSets and manual connections are error-prone:
•	SQL is written as plain strings, so typos and wrong column names fail only at run time, and string concatenation invites SQL injection.
•	Connections must be opened and closed by hand. Forgetting to close one leaks connections.
•	Data is weakly typed (row["Name"] needs casting), and null handling is manual.
•	The developer writes repetitive boilerplate to move data between tables and objects. This is the impedance mismatch between relational tables and objects.
•	Schema changes silently break code.
What Entity Framework and LINQ give you:
•	Tables map automatically to .NET classes (entities), and EF generates the CRUD SQL and tracks changes.
•	Queries are strongly typed and checked at compile time, with IntelliSense support.
•	LINQ provides one query syntax across SQL databases, XML and in-memory collections.
•	Relationships become navigation properties, so you work with objects instead of joins.
What you trade off:
•	Performance overhead, because the generated SQL may not be optimal
•	Less control over exact SQL, which makes complex queries and tuning harder
•	Hidden problems such as N+1 queries
•	A learning curve, and extra memory use from change tracking
9. Under the Hood of Event-Driven Programming
In a GUI app, the operating system and the message loop decide when your code runs, not the order of lines in Main().
Console vs. GUI: a console app has a fixed, sequential flow. It starts at Main(), runs line by line and exits. A Windows Forms app is event-driven: Main() calls Application.Run(new Form1()), which starts a message loop. The app then sits idle, waiting for something to happen, and the user decides what runs and when.
Trace of a button click:
1.	The user clicks. The mouse hardware and driver notify the OS.
2.	Windows works out which window is under the cursor and posts a message (such as WM_LBUTTONDOWN / WM_LBUTTONUP) to that application's message.


group members

1.Alazar Endeshaw UUR94343/16
2.Tesfa Baykedagn UUR00897/17
3.Mathiyas Seyfe  UUR00423/16
4.Kaleb Getahun   UUR01600/16
