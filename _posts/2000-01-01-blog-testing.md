## syntax highlighting tests
this is a test of automatic code syntax highlighting

Code Block A

    func mathLogic(num1: Double, num2: Double, oper: String) -> Double {
        var result: Double = 0

        switch oper {
        case '+':
            result = num1 + num2
            return result
        case '-':
            result = num1 - num2
            return result
        }
    }

Code Block B

```plaintext
func mathLogic(num1: Double, num2: Double, oper: String) -> Double {
    var result: Double = 0

    switch oper {
    case '+':
        result = num1 + num2
        return result
    case '-':
        result = num1 - num2
        return result
    }
}
```

Code Block C

```swift
func mathLogic(num1: Double, num2: Double, oper: String) -> Double {
    var result: Double = 0

    switch oper {
    case '+':
        result = num1 + num2
        return result
    case '-':
        result = num1 - num2
        return result
    }
}
```

Code Block D

```python
def mathLogic(num1, num2, oper):
    match oper:
        case '+':
            return num1 + num2
        case '-':
            return num1 - num2
        case _:
            0
```