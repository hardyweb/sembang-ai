AI coding assistants are getting very good at generating Laravel code.

Give an AI a feature request and it can probably create:

* migrations
* models
* controllers
* Form Requests
* Blade views
* routes
* tests
* JavaScript
* CSS

The problem is not whether the code works.

The problem is whether the code **belongs in the project**.

AI can easily generate Laravel code that works today but becomes a maintenance problem tomorrow:

* fat controllers
* duplicated business logic
* unnecessary service/repository layers
* `$request->all()`
* N+1 queries
* database queries inside Blade
* huge inline `<style>` blocks
* huge inline `<script>` blocks
* introducing another frontend framework
* ignoring existing components
* rewriting parts of the starter kit
* unrelated refactoring
* debug code left behind
* tests that were never actually executed

For me, the solution is not to tell the AI to "write better code".

The better approach is to **define the engineering rules and make the AI work inside the existing project architecture**.

My current workflow is built around:

```text
Laravel Project
      │
      ├── AGENTS.md
      │      └── Engineering rules
      │
      ├── skills.md
      │      └── Knowledge / examples / procedures
      │
      └── Kilo Code
             └── Implement within those rules
```

And for longer-running projects, there is another layer that can work alongside them:

```text
DSOM Brain
└── Project state, decisions, continuity and context
```

I'll come back to that near the end.

---

# 1. First Rule: Inspect Before You Code

Before asking an AI coding agent to implement anything, I want it to understand the project that already exists.

This is probably the most important rule in my workflow.

Don't start with:

> "Build a user management system."

Start with:

> "Inspect the existing project first."

The AI should inspect things such as:

* Laravel version
* `composer.json`
* existing application structure
* authentication system
* authorization approach
* existing routes
* controllers
* models
* Form Requests
* policies
* migrations
* tests
* frontend architecture
* starter kit
* CSS framework
* JavaScript/TypeScript structure
* existing components
* project conventions

The reason is simple.

**The existing project is already an architecture.**

If the application already uses Livewire, don't suddenly introduce Inertia and React just because the AI knows how to generate them.

If the application already uses React + Inertia, don't create a random Blade-based application UI.

If the application uses Tailwind, don't introduce Bootstrap.

If the project already has reusable components, don't create another component that does the same thing.

AI should extend the project, not accidentally redesign it.

---

# 2. Why AI-Generated Laravel Code Can Become Sloppy

AI optimizes heavily for producing a plausible solution.

That doesn't necessarily mean it optimizes for:

* consistency
* simplicity
* maintainability
* project conventions
* long-term architecture
* minimal changes

For example, imagine this request:

> Add an endpoint to update a user's profile.

A simplistic AI implementation might put everything inside the controller:

```php
public function update(Request $request, User $user)
{
    $user->update($request->all());

    // validation
    // authorization
    // business logic
    // logging
    // notifications
    // other operations...
    
    return redirect()->back();
}
```

It might work.

But the better Laravel implementation should consider:

* Form Request validation
* authorization
* mass-assignment rules
* route model binding
* business logic placement
* transactions if required
* testing
* security implications

The goal isn't to make the code more complicated.

The goal is to put each responsibility where it belongs.

---

# This Concept Is Not Kilo Code Specific

Although this workflow was developed and tested with Kilo Code, the underlying idea is not tied to Kilo Code.

The important part is not the tool name. The important part is giving an AI coding agent a clear set of engineering rules, project knowledge, and working context before asking it to write code.

If you use other coding agents such as OpenCode, Codex, or Claude Code, you can apply the same concept.

The exact mechanism may be different:

the instruction file may have a different name,
the location may be different,
the way instructions are loaded may be different,
and each tool may have its own instruction hierarchy or configuration.

But the architecture can remain the same:

                    AI Coding Agent
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        Engineering     Project      Working
           Rules       Knowledge      Context
              │            │            │
              └────────────┼────────────┘
                           ▼
                    Laravel Project

For example, the same principles can be adapted to:

Kilo Code    → AGENTS.md + skills
OpenCode     → project instructions / AGENTS.md
Codex        → project instructions / AGENTS.md
Claude Code  → project instructions / CLAUDE.md

The exact filenames and supported features should be checked against the documentation for the coding agent you are using.

The key idea

Don't think:

"I need to use Kilo Code because this workflow uses AGENTS.md."

Think instead:

"I need to give my coding agent an engineering contract."

That contract should tell the agent things such as:

how the project is structured,
which Laravel conventions to follow,
how authorization should be handled,
where validation belongs,
how database access should be designed,
what security practices are required,
how tests should be written,
what the agent must inspect before changing code,
and what it should not change unnecessarily.

So if you move from Kilo Code to OpenCode, Codex, or Claude Code, you don't have to throw away the workflow.

You adapt the instruction layer, while keeping the underlying engineering principles.

This makes the workflow tool-agnostic, even though my current implementation and examples are based on Kilo Code.

# 3. `AGENTS.md` as the Project Engineering Contract

This is where `AGENTS.md` becomes useful.

Instead of repeating the same instructions every time I ask the AI to implement something, I keep the important engineering rules inside the repository.

For example:

```text
my-laravel-app/
├── AGENTS.md
├── skills.md
├── composer.json
├── artisan
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── storage/
└── tests/
```

The important idea is that `AGENTS.md` is not documentation for humans only.

It becomes a **working contract between the project and the AI coding agent**.

For example:

```md
# Laravel Engineering Rules

- Follow the existing project architecture.
- Inspect existing code before implementation.
- Keep controllers thin.
- Use Form Requests for validation.
- Use policies for authorization.
- Prefer Eloquent over unnecessary repository abstractions.
- Do not put business logic in Blade.
- Avoid N+1 queries.
- Use migrations for database changes.
- Write tests for application behaviour.
- Do not introduce another frontend framework without a clear requirement.
- Reuse existing components.
- Do not add unnecessary abstractions.
- Do not refactor unrelated code.
```

Now the AI has something concrete to follow.

---

# 4. Follow the Existing Laravel Starter Kit

Laravel applications don't all have the same frontend architecture.

A project may use:

* Livewire
* React + Inertia
* Vue + Inertia
* Blade
* Tailwind
* another existing frontend setup

Laravel's current starter kits also provide different application stacks, so the starter kit already present in a project should be treated as part of that project's architecture.

My rule is:

> **Inspect the starter kit before creating new UI.**

If the project already has a component system, use it.

If the project already has layouts, use them.

If the project already has navigation components, don't create another navigation implementation.

The starter kit is a starting architecture, not something the AI should casually replace.

---

# 5. Don't Introduce Another Frontend Stack

This is one of the easiest ways for AI-generated code to become messy.

Imagine an existing Laravel project using:

```text
Blade + Alpine + Tailwind
```

Then an AI decides to add:

```text
React + Vite + another component library
```

Now the project has two frontend architectures.

That might be justified in some applications.

But it should never happen simply because the AI generated a React solution faster.

The rule should be:

> Follow the frontend architecture already used by the project unless there is an explicit reason to change it.

---

# 6. Reuse Existing Components

Before creating a new component, look for an existing one.

For example:

```text
resources/views/components/
```

might already contain:

```text
button.blade.php
input.blade.php
modal.blade.php
alert.blade.php
table.blade.php
```

If the application already has a button component, don't generate another button implementation.

The AI should inspect first.

This simple rule prevents a lot of duplication.

---

# 7. Don't Put Raw CSS Inside Blade

Another common AI-generated pattern is:

```blade
<style>
    .some-big-component {
        ...
    }

    .another-component {
        ...
    }

    /* 200 more lines */
</style>
```

This may work, but it can quickly become difficult to maintain.

If the project uses Tailwind, use Tailwind.

If the project uses Bootstrap, use Bootstrap.

If custom CSS is genuinely required, put it into the project's existing CSS structure.

For example:

```text
resources/
└── css/
    ├── app.css
    └── components/
        └── editor.css
```

The exact structure depends on the project.

The important rule is:

> **Follow the project's existing CSS architecture instead of dumping large amounts of CSS into Blade.**

The same principle applies to JavaScript.

Don't create a huge:

```blade
<script>
    // 300 lines of application logic
</script>
```

inside a Blade view if the project already has a proper JavaScript/TypeScript structure.

---

# 8. My Basic Laravel Architecture Rules

These are the rules I want an AI coding agent to understand before it starts changing my Laravel application.

---

## Thin Controllers

Controllers should coordinate the request.

They shouldn't become the entire application.

Bad:

```php
public function store(Request $request)
{
    // validation

    // authorization

    // database queries

    // business rules

    // notifications

    // logging

    // calculations

    // 100 more lines
}
```

Better:

```php
public function store(StoreOrderRequest $request)
{
    $this->authorize('create', Order::class);

    $order = $this->orderService->create(
        $request->validated()
    );

    return redirect()->route('orders.show', $order);
}
```

The exact architecture depends on the project.

The important part is keeping responsibilities separated.

---

# 9. Form Requests Instead of `$request->all()`

I don't want AI-generated Laravel code to casually do this:

```php
$data = $request->all();
```

Especially when the data is going into a model.

Use a Form Request when validation and authorization belong there:

```php
$data = $request->validated();
```

Or:

```php
$data = $request->safe()->only([
    'name',
    'email',
]);
```

This makes the input boundary explicit.

It also reduces the chance of accidentally passing unexpected fields into application logic.

---

# 10. Authorization Is Not Authentication

Another common mistake is confusing:

```text
Who are you?
```

with:

```text
Are you allowed to do this?
```

Authentication identifies the user.

Authorization determines whether that user can perform the operation.

Laravel provides policies and gates for this.

For example:

```php
$this->authorize('update', $post);
```

The AI should not assume that:

```php
auth()->check()
```

means the user is allowed to modify the resource.

---

# 11. Eloquent First

Laravel already provides a powerful ORM.

I don't want an AI to create:

```text
Repository
    ↓
Interface
    ↓
Repository Implementation
    ↓
Manager
    ↓
Service
    ↓
Model
```

for a simple CRUD operation.

If Eloquent is enough:

```php
$user = User::findOrFail($id);
```

use Eloquent.

Repositories can be useful in the right context.

But abstraction should solve a problem, not create one.

---

# 12. Check for N+1 Queries

AI-generated Laravel code can easily create N+1 queries.

For example:

```php
$posts = Post::all();

foreach ($posts as $post) {
    echo $post->author->name;
}
```

The correct implementation may need:

```php
$posts = Post::with('author')->get();
```

The AI should inspect relationships and query behaviour rather than assuming that syntactically valid Eloquent code is automatically efficient.

---

# 13. Don't Put Business Logic in Blade

Blade should primarily handle presentation.

I don't want this:

```blade
@php
    $orders = Order::where('user_id', auth()->id())->get();

    // business logic
    // calculations
    // database operations
@endphp
```

The view should receive the data it needs.

For example:

```php
$orders = $user->orders()->latest()->get();

return view('orders.index', compact('orders'));
```

Then Blade renders it.

---

# 14. Services and Actions Only When Needed

I'm not against service classes.

I'm against creating them automatically.

A service or action can make sense when there is:

* complex business logic
* reusable application behaviour
* multiple steps
* transaction boundaries
* difficult-to-test logic
* meaningful separation of responsibilities

But this:

```text
CreateUserService
```

for a three-line CRUD operation may simply add another layer.

The rule is:

> **Introduce abstractions because the problem requires them, not because AI likes generating classes.**

---

# 15. Database Changes Through Migrations

Don't manually modify production database structures.

Use migrations.

For example:

```bash
php artisan make:migration add_status_to_orders_table
```

Then define the change.

Database design should also consider:

* foreign keys
* indexes
* unique constraints
* nullable columns
* data types
* relationships

AI should inspect the existing schema before creating another migration.

---

# 16. Transactions When Atomicity Matters

If an operation modifies several related records and they must succeed or fail together, consider a transaction.

For example:

```php
DB::transaction(function () {
    // create order

    // create order items

    // update inventory
});
```

But again, don't wrap every single database query inside a transaction just because the AI knows the API exists.

Use transactions when atomicity matters.

---

# 17. Security Is Part of the Coding Standard

Security shouldn't be an afterthought.

For Laravel development, I want the AI to consider:

* authentication
* authorization
* validation
* mass assignment
* CSRF
* XSS
* SQL injection
* file uploads
* path traversal
* rate limiting
* secrets
* logging
* access control

For example:

```php
Log::info('User updated profile', [
    'user_id' => $user->id,
]);
```

is useful.

But:

```php
Log::info($request->all());
```

could expose sensitive information.

The AI should understand that logging itself can become a security problem.

---

# 18. Testing Is Part of the Workflow

I don't want AI to say:

> Tests should be added later.

If a feature changes application behaviour, testing should be part of the implementation.

For example:

```text
Feature
├── implementation
├── validation
├── authorization
└── tests
```

Feature tests are useful for application behaviour.

Unit tests are useful for isolated logic.

And one important rule:

> Never claim that tests passed unless they were actually executed.

An AI saying:

```text
All tests pass.
```

without running them is not useful.

---

# 19. The "No Sloppy AI Code" Checklist

Before accepting AI-generated code, I can ask:

```text
- Did you inspect the existing architecture first?
- Did you follow the existing starter kit?
- Did you reuse existing components?
- Did you follow the existing CSS/JS structure?
- Are controllers still thin?
- Is validation handled properly?
- Is authorization handled separately?
- Did you use Eloquent before introducing abstractions?
- Did you check for N+1 queries?
- Is business logic outside Blade?
- Are services/actions actually justified?
- Are database changes done through migrations?
- Are transactions used only where needed?
- Did you consider security?
- Did you add appropriate tests?
- Did you avoid unrelated refactoring?
- Did you remove debug code?
```

This checklist is often more valuable than telling an AI:

> "Write clean code."

Because "clean" is subjective.

Rules are more explicit.

---

# 20. `skills.md` Is Different From `AGENTS.md`

I also like separating **rules** from **knowledge**.

`AGENTS.md` answers:

> **How should the AI work on this project?**

While `skills.md` can contain:

* useful procedures
* project-specific knowledge
* examples
* commands
* troubleshooting
* reusable implementation patterns

For example:

```text
AGENTS.md
└── Rules

skills.md
└── Knowledge / procedures / examples
```

This keeps the main engineering contract focused.

The AI doesn't need every piece of project knowledge to become a hard rule.

---

# 21. Why I Use Kilo Code

My current workflow uses **Kilo Code** as the coding agent.

I don't need to run four different coding agents for every Laravel project.

The important part isn't how many AI tools are involved.

The important part is whether the AI is working inside a clear engineering environment.

My basic setup is:

```text
Kilo Code
    │
    ├── Inspect project
    │
    ├── Read AGENTS.md
    │
    ├── Load relevant skills
    │
    ├── Plan
    │
    ├── Implement
    │
    ├── Test
    │
    └── Review changes
```

Another developer may prefer another coding agent.

That's fine.

The principles remain the same.

---

# 22. Windows, Linux and WSL

My development environment is not necessarily a traditional Linux workstation.

I work across:

```text
Windows
   │
   └── WSL2
          │
          ├── Linux
          ├── Laravel
          ├── Git
          └── Kilo Code
```

This is another reason why project-local instructions are useful.

The AI shouldn't assume a generic environment.

If the project has specific commands, paths, services or development procedures, document them.

---

# 23. Don't Put Secrets in `AGENTS.md`

`AGENTS.md` belongs in the project.

That means it may eventually be committed to Git.

Therefore:

**Don't put secrets inside it.**

Never store:

```text
API keys
passwords
tokens
private credentials
production secrets
```

Instead document the expected configuration:

```text
MAIL_USERNAME=<configured through environment>
MAIL_PASSWORD=<configured through environment>
```

The actual secret belongs in the appropriate secret/configuration mechanism.

---

# 24. A Practical Project Structure

A simple Laravel project can look like:

```text
my-laravel-app/
├── AGENTS.md
├── skills.md
├── composer.json
├── artisan
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── storage/
└── tests/
```

The important part isn't the number of files.

It is that the project contains a small, visible place where the AI can learn:

```text
Rules
Knowledge
Code
Tests
```

---

# 25. A Real Example: My Kilo Code Configuration

The examples above describe the workflow conceptually.

If you want to see how I actually structure these files in a working repository, I keep a practical example here:

GitHub:
https://github.com/hardyweb/kilo

The repository contains the project-level instructions and skills I use with Kilo Code.

The important part is not to copy the repository blindly.

Instead, look at how the responsibilities are separated:
```bash
kilo/
├── AGENTS.md
│
└── skills/
    └── laravel/
        └── SKILL.md
```
The idea is simple:

```bash
AGENTS.md
    │
    └── Project-wide engineering rules
             │
             ▼
skills/
    │
    └── Domain-specific knowledge
             │
             ▼
       Coding Agent
             │
             ▼
       Laravel Project
```
For example, AGENTS.md can define rules such as:

```text
inspect before changing code
follow the existing architecture
keep controllers thin
use Form Requests for validation
use policies for authorization
avoid unnecessary abstractions
reuse existing components
test behaviour
don't claim tests passed unless they were actually run
```

The Laravel skill can then contain more detailed Laravel-specific knowledge, procedures, conventions and implementation guidance.

This separation is useful because not everything needs to become a global rule.

# 26. What If You Want More Than Coding Rules?

This is where another interesting layer comes in.

`AGENTS.md` is useful for defining **how the AI should work**.

`skills.md` is useful for giving the AI **knowledge, procedures and examples**.

But long-running projects can have another problem:

> **What does the AI need to remember about the project over time?**

Architecture decisions.

Previous work.

Current project state.

Session handovers.

Decisions that were already made.

Things that should not be repeated.

This is a different problem from coding rules.

One project I came across is **Deep State of Mind (DSOM)**, which approaches this problem as a persistent AI context and governance layer. Its DSOM architecture includes persistent project state, decision/context artifacts, session continuity and a Git-oriented workflow.

Conceptually, I see the layers like this:

```text
                    AI Coding Workflow
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
     AGENTS.md         skills.md        DSOM Brain
          │                │                │
       Rules          Knowledge       Project State
     & standards       & examples     & continuity
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                       Kilo Code
                           │
                           ▼
                    Laravel Project
```

The distinction is important:

```text
AGENTS.md
"What rules should the AI follow?"

skills.md
"What knowledge/procedures can help the AI?"

DSOM Brain
"What does the AI need to remember about this project?"
```

I don't see DSOM Brain as something that replaces `AGENTS.md` or `skills.md`.

Instead, it can **run alongside them**.

For a small Laravel project, `AGENTS.md` and a small set of skills may already be enough.

For a long-running project, however, persistent project state and context can become useful.

That's where **DSOM Brain** becomes interesting.

I'm keeping the DSOM Brain implementation out of this article because it deserves its own discussion.

I'll cover that separately in another article.

For reference:

[Deep State of Mind (DSOM) For My AI](https://linuxmalaysia.github.io/deep-state-of-mind-for-my-ai/?utm_source=chatgpt.com)

---

# 27. The Real Goal

The goal isn't to make AI write more code.

The goal is to make AI work **inside an engineering system**.

My current thinking is:

```text
AGENTS.md
    │
    └── Engineering rules

skills.md
    │
    └── Knowledge and procedures

DSOM Brain
    │
    └── Persistent project context

Kilo Code
    │
    └── Implementation

Git
    │
    └── History and audit trail

Laravel
    │
    └── The actual application
```

This changes the relationship with an AI coding agent.

Instead of:

> "AI, build this feature."

The workflow becomes:

> "Here is the project. Inspect it first. Here are the engineering rules. Here are the relevant skills. Here is the project context. Now implement the feature within the existing architecture."

That is a much more useful way for me to think about AI-assisted Laravel development.

**Don't try to teach the AI everything. Define the rules that matter, inspect the existing project first, keep those rules close to the project, and make the AI work within the architecture that already exists.**
