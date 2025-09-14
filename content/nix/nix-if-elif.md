---
aliases:
  - /2016/06/12/if-then-else/
title: "Understanding Nix if-then-else Expressions, Why There's No elseif and How to Work Around It"
summary: "Learn why Nix uses if-then-else expressions instead of if-elseif-else statements, and discover practical patterns for handling multiple conditions in expression-based languages."
tags: ["nix", "expressions", "conditionals"]
date: "2016-06-12"
mod: "2025-09-12"
---

## Updates Since Original Publication

Language Stability: Nix's core expression system and if-then-else behavior remain unchanged since 2016. The fundamental concepts in this article are still accurate.

Ecosystem Changes:
- Nixpkgs has grown to over 80,000 packages with improved library functions
- Flakes introduced as modern alternative to channels (doesn't affect conditional logic)
- Enhanced documentation at nix.dev replaces older resources
- Better IDE support and tooling available

Code Updates: All examples work with current Nix versions. Consider checking nixpkgs lib documentation for latest helper functions when implementing the patterns shown.

---

The following is the original post content with some updates to the writing style to (hopefully) make it easier to read:

A teammate recently asked me: _"I understand how to use if-else in Nix, but how do I create if-elseif-else logic?"_ This question reveals a common misunderstanding that trips up developers transitioning from imperative languages to expression-based systems like Nix.

The answer lies in understanding a fundamental difference: **Nix uses expressions, not statements.**

## Expressions vs. Statements: The Core Difference

Most mainstream languages (Perl, Bash, JavaScript) use statements for control flow. Statements execute actions but don't necessarily return values. This flexibility allows constructs like:

```bash
if [ -d ./path ]; then
  declare key="directory"
elif [ -x ./path ]; then
  echo "executable file found"
else  
  declare key="something else"
fi
```

Notice how the middle branch performs a side effect (printing) without setting the `key` variable. This creates an inconsistent state where `key` might remain undefined.

Nix, as an expression language, requires every expression to evaluate to a value. This constraint eliminates the inconsistency problem but also eliminates traditional elseif syntax.

## How Nix if-then-else Works

In Nix, every conditional must return a value:

```nix
{
  key = if builtins.pathExists ./path then "exists" else "missing";
}
```

This expression always evaluates to either:
```nix
{ key = "exists"; }
# OR  
{ key = "missing"; }
```

The variable `key` is guaranteed to have a value because both branches of the expression return one.

## Why This Design Prevents Bugs

Expression-based conditionals eliminate a entire class of bugs. Consider this problematic Bash code:

```bash
if [ -d ./path ]; then
  declare result="directory"  
elif [ -x ./path ]; then
  echo "Found executable"  # Oops! No assignment
else
  declare result="file"
fi
echo "Result: ${result}"  # May be undefined!
```

Nix's expression system makes this impossible. Every path through an if-then-else must produce a value, ensuring consistency.

## Practical Solutions for Multiple Conditions

### Pattern 1: Nested if-then-else Expressions

For complex conditions, nest expressions:

```nix
let
  fileType = if builtins.pathExists ./path then
    if builtins.readDir ./path != {} then "directory"
    else if lib.isExecutable ./path then "executable"  
    else "regular-file"
  else "missing";
in { inherit fileType; }
```

### Pattern 2: Attribute Set Lookups

When matching exact values, use attribute sets as lookup tables:

```nix  
{ envType, ... }:
let
  configs = {
    "dev" = devConfig;
    "test" = testConfig; 
    "prod" = prodConfig;
  };
  
  getConfig = env: default:
    if configs ? ${env} then configs.${env} else default;
    
in getConfig envType defaultConfig
```

This pattern is cleaner and more maintainable than nested conditionals for simple mappings.

### Pattern 3: Helper Functions

Create reusable conditional logic:

```nix
let
  selectByCondition = conditions: default:
    if conditions == [] then default
    else 
      let first = builtins.head conditions;
      in if first.condition then first.value 
         else selectByCondition (builtins.tail conditions) default;
         
  result = selectByCondition [
    { condition = builtins.pathExists ./dev; value = "development"; }
    { condition = builtins.pathExists ./prod; value = "production"; }
  ] "unknown";
in result
```

## The Mental Model: Think Functions, Not Control Flow

You can conceptualize if-then-else as a function:

```nix
ifThenElse = condition: trueValue: falseValue:
  if condition then trueValue else falseValue;
```

This function always returns exactly one of its two value parameters. There's no room for side effects or undefined states.

## Key Takeaways

**Expression languages like Nix prioritize predictability over flexibility.** While you lose the ability to mix side effects with conditional logic, you gain:

- **Guaranteed value assignment** - variables are never unexpectedly undefined
- **Referential transparency** - expressions evaluate consistently  
- **Easier debugging** - no hidden state changes in conditionals
- **Better composability** - expressions nest cleanly without side effects

*Important caveat:* Not all if/elif/else implementations in other languages require an else clause or guarantee the same return type across branches. In languages without exhaustivity checking or sum type support, there's minimal functional difference between nested if-then-else expressions and attribute set lookups - only syntax differs.

Nix's strength lies in its **mandatory else clause** and **expression semantics**. While Nix itself is dynamically typed, the expression-based design enforces that both branches must return *some* value. In statically typed expression languages like Dhall or Nickel, this constraint goes further - both branches must return values of the same type, providing compile-time guarantees about program behavior.

This expression-based approach eliminates undefined states that plague optional-else languages, whether through Nix's runtime value requirement or through static type checking in more strongly typed functional languages.

When transitioning from statement-based languages, embrace the constraint. The "limitation" of always returning values becomes a strength that prevents entire categories of bugs.

The next time you reach for elseif, remember: you're thinking in statements. Switch to expressions, and discover patterns that are often cleaner and more reliable than their imperative counterparts.
