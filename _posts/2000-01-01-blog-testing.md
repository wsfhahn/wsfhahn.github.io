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