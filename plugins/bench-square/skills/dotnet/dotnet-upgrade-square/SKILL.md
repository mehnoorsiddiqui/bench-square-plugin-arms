---
name: "dotnet-upgrade-square"
description: "Upgrading existing C# code to a new version of the Square .NET SDK — a package version bump on code that already calls Square. Load it before editing any file for the upgrade; it sets the upgrade workflow, the companion skill to load for each kind of change, and how to find the changes that still compile. .NET/C# only; never load it for another language."
---

# Square .NET SDK — Upgrading to a new SDK version

An upgrade skips the plan-first flow in `dotnet-integrate-square` (no `square-planner`, no `square-plan.md`). Work through these steps in order.

## 1. Find every change your code meets

Compile errors show only the members whose type changed or that were removed. Other changes compile and still break the code: a validation attribute (`[StringLength]`, `[MaxLength]`, `[Range]`, `[RegularExpression]`, `[Minimum]`, `[Maximum]`) added or tightened, a default changed, an enum value added or dropped, a wire name changed, or a member the API sends back that was always present can now be missing (required to optional, non-nullable to nullable), an enum it sends back gains values, or a list can now be empty. The SDK never checks validation attributes; the provider refuses the request at runtime (see `dotnet-models`). A value that comes back differently compiles and runs, but the old code was written assuming it could never occur.

- Get both versions: the SDK source at both tags, or both package assemblies. Listing members from them is inspection.
- Print every changed line of every model in one command, and read all of it. Doc comments are filtered out, so what is left is real:

  ```
  git diff -U0 <old-tag> <new-tag> -- src/Square/Models | grep -E '^[+-]' | grep -vE '^[+-]\s*///'
  ```

  Do not choose which models to read by the size of their diff: a one-line attribute change breaks code as surely as a removed member. An attribute applies to the member declared below it, so read the two together.
- Start from your code. Before editing, write a table with one row for every SDK member it sets or reads and one for every value it builds itself (keys, ids, references, joined strings): the member, its type, attributes and default in the old and new versions; for a value the code builds, the longest value the code can produce; and, for a member the code reads, what the new version can send back that the old one never did (missing, null, empty, a new enum value). Fill every row.

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

## 4. Check what you send and what comes back

**What you send.** The new version may be unable to hold something the old code produced: a value its type does not declare, no value for a member that became required, a value over a new or tighter limit. Decide what the code does in each such case and report the decision. Check each value your code builds against its limit for the longest input the code can produce, row by row from the table in step 1.

**What comes back.** For every row in the table where the new version can send back a value the old one never did:

1. Write one concrete example: a response that carries that value (the record with the member left out, the new enum value, the empty list).
2. Trace it through every place your code reads that member, to what the code finally returns, stores or decides (a total, a balance, a status, an allow or refuse).
3. Ask whether that result is still correct, not whether the code still runs. Comparisons are where it usually goes wrong: `==`, `Equals`, a dictionary lookup, a `switch`, a `Where`. A missing value matches nothing, so the record quietly drops out of a sum, a count or a check, and the result is wrong with no error. If the result is still correct, change nothing and say so.
4. If the result is wrong, fix what the code means, not only its types. Find out what the API means when it sends that value (the member's doc comment, the provider's docs). A record with a member left out usually still exists and still affects the result; you only know less about it. Handle it on those terms. Do not leave the comparison because it compiles, filter out records that lack the value, substitute `""`, a default or a placeholder that matches nothing, or throw where the old code returned an answer. Throw only when no correct answer exists, and report it.
5. If the project has tests, add one that feeds the example in and checks the final result.

## 5. Remove what the new version removed

When the new version removes a member and names nothing in its place, delete the code that set it, change any test that asserts it in the same way, and report the behaviour that is lost. A model's extension-data bag carries fields the API defines that this SDK never modelled; a member the new version removed stays removed.

## 6. Finish

Keep all other behaviour the same. In the summary, say how you compared the two versions, list each change you found that still compiles and what you did about it, list each member that can now come back differently with the example you traced and the result before and after your change, and report any behaviour the new version cannot keep. Claim a check only if you can name the command that made it.
