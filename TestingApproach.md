```markdown
# 🧪 Testing Approach in Gibber

## 🎯 Overview
Gibber integrates testing directly into the development process by allowing **LLMs to write, run, and validate tests** in real-time while generating code. This ensures that:
- **Code and unit tests** are either in the same file or separate files as needed.
- **Tests are enforced before compilation**—compilation fails if tests do not pass.
- LLMs can **automatically generate, run, and validate** test cases during development.

---

## 🏗 How Testing Works in Gibber

1. **LLM generates code and tests together**.
2. **Tests are executed in real-time** as the code is being written.
3. **Code cannot compile unless all tests pass**.
4. **Testing can be inline (same file) or separate** depending on project structure.

---

## 🔍 Example: Code and Tests in the Same File
```gibber
@explain("Computes the square of a number.")
@pure
def square(x: int) -> int:
    return x * x

@test
@explain("Tests the square function.")
def test_square():
    assert square(2) == 4
    assert square(3) == 9
    assert square(-2) == 4
```
### ✅ Key Features:
- **`@test`** annotation tells Gibber that the function is a test case.
- **Tests run immediately**—if they fail, the code won’t compile.
- **LLM auto-generates the test cases** alongside the function.

---

## 🔍 Example: Code and Tests in Separate Files
### **`mathlib.gibber`** (Main Code File)
```gibber
@explain("Implements basic math functions.")
@module("mathlib")
class MathLib:
    @explain("Finds the maximum of two numbers.")
    def max(a: int, b: int) -> int:
        return a if a > b else b
```

### **`mathlib_test.gibber`** (Test File)
```gibber
@import("mathlib")

@test
@explain("Tests max function.")
def test_max():
    assert MathLib.max(3, 5) == 5
    assert MathLib.max(10, 2) == 10
    assert MathLib.max(-1, -5) == -1
```
### ✅ Key Features:
- **Tests are in a separate file** but automatically linked to `mathlib.gibber`.
- **Gibber enforces that all modules have test coverage**.

---

## 🚀 Enforcing Tests Before Compilation
Gibber **prevents compilation if any test fails**. The command-line workflow looks like this:

```sh
$ gibber build
❌ Compilation failed: Test test_square failed (expected 9, got 8)
```

Once all tests pass, compilation succeeds:
```sh
$ gibber build
✅ Compilation successful.
```

---

## 📌 Summary
- **LLMs generate and run tests while writing code**.
- **Tests can be inline (same file) or separate**.
- **Tests are required before compilation succeeds**.
- **Automated AI-driven TDD (Test-Driven Development)** ensures **higher code reliability**.

---

Gibber's built-in test enforcement **eliminates AI-generated bugs** before code reaches production. What features should we add next? 🚀
```

