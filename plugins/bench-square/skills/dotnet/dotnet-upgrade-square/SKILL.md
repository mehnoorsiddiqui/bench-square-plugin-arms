---
name: "dotnet-upgrade-square"
description: "Upgrading existing C# code to a new version of the Square .NET SDK — a package version bump on code that already calls Square. Load it before editing any file for the upgrade; it sets the upgrade workflow, the companion skill to load for each kind of change, and how to find the changes that still compile. .NET/C# only; never load it for another language."
---

# Square .NET SDK — Upgrading to a new SDK version

An upgrade skips the plan-first flow in `dotnet-integrate-square` (no `square-planner`, no `square-plan.md`). Work through these steps in order.

## 1. Find every change your code meets

Compile errors show only the members whose type changed or that were removed. Other changes compile and still break the code: a validation attribute (`[StringLength]`, `[MaxLength]`, `[Range]`, `[RegularExpression]`, `[Minimum]`, `[Maximum]`) added or tightened, a default changed, an enum value added or dropped, a wire name changed. The SDK never checks validation attributes; the provider refuses the request at runtime (see `dotnet-models`).

- Get both versions: the SDK source at both tags, or both package assemblies. Listing members from them is inspection.
- Print every changed line of every model in one command, and read all of it. Doc comments are filtered out, so what is left is real:

  ```
  git diff -U0 <old-tag> <new-tag> -- src/Square/Models | grep -E '^[+-]' | grep -vE '^[+-]\s*///'
  ```

  Do not choose which models to read by the size of their diff: a one-line attribute change breaks code as surely as a removed member. An attribute applies to the member declared below it, so read the two together.
- Start from your code. Before editing, write a table with one row for every SDK member it sets or reads and one for every value it builds itself (keys, ids, references, joined strings): the member, its type, attributes and default in the old and new versions, and, for a value the code builds, the longest value the code can produce. Fill every row.

## 2. Load the companion for each kind of change

Before editing, load the companion for each kind of change the new version brings:

| Change | Load |
| --- | --- |
| models, enums or a field whose type changed | `dotnet-models` |
| operations or request records | `dotnet-calling-endpoints` |
| environments, retries, timeouts or logging | `dotnet-configuration-resilience` |
| client setup | `dotnet-client-initialization` |
| credentials | `dotnet-authentication` |
| errors | `dotnet-error-handling` |

## 3. Rewrite each use on the new version's terms

Write each use of a changed member the way its companion describes using it on the new version, as you would in a fresh integration. Code that compiles only through a string comparison on a typed member, a cast, a serializer, a `[JsonConverter]`, reflection or a file under `src/Square/Core/` is a defect.

Change only what the new version forces. Keep the signature of every method, record and type in your own code unless the new SDK makes the old one impossible; convert between your types and the SDK's where your code calls the SDK. Before changing a signature, search for every caller, tests included.

## 4. Decide what the new version cannot hold

The new version may be unable to hold something the old code produced: a value its type does not declare, no value for a member that became required, a value over a new or tighter limit. Decide what the code does in each such case and report the decision. Check each value your code builds against its limit for the longest input the code can produce, row by row from the table in step 1.

## 5. Remove what the new version removed

When the new version removes a member and names nothing in its place, delete the code that set it, change any test that asserts it in the same way, and report the behaviour that is lost. A model's extension-data bag carries fields the API defines that this SDK never modelled; a member the new version removed stays removed.

## 6. Finish

Keep all other behaviour the same. In the summary, say how you compared the two versions, list each change you found that still compiles and what you did about it, and report any behaviour the new version cannot keep. Claim a check only if you can name the command that made it.
