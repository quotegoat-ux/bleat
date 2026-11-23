---
name: principle-test-behavior-not-implementation
description: "Apply when you write, change, or keep a test. Assert what the code's users can observe, so that the test fails when that behavior breaks, keeps passing when only the internals change, and gives the same answer however and wherever it runs."
user-invocable: false
---

# Test Behavior, Not Implementation

A test is a claim about what users of the code can observe through its interface: return values, errors, state they can read back, and effects on outside systems. Private helpers, internal call order, and intermediate data are implementation.

Judge each test against two imagined edits:

- **Break the behavior** (a wrong value, a skipped or doubled effect). If the test still passes, it proves nothing. The usual causes are a loose assertion, an expected value computed by the code under test, and an assertion on data the test built itself.
- **Change the internals** and keep every observable result. If the test fails, it is pinned to the implementation.

Keep a test that catches the first edit and survives the second. Rewrite or delete the rest. Drive the code through its public interface with a concrete input, and derive the expected value from the requirement. What an external service receives is behavior. A call to an internal helper is not. Pair "this never happens" with a case where it does happen. Assert the error type that callers branch on, not the message.

Then change nothing and run it again: alone, in a different order, under CPU load, and with the environment variables a developer or CI box sets. If the result changes, the test depends on something it does not own. The usual causes are an environment variable it reads but never sets, a timer, editor, or container it starts and never stops before teardown, a mock that returns a new object on each call where the real code returns the same one, and a click sent before the UI has wired its handler. Make the test own each one: stub the variable, destroy what it creates, keep the real code's identity guarantees in the mock, and wait for a ready state, not a fixed time.
