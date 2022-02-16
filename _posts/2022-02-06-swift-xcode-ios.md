---
layout: post
---

# Swift

https://docs.swift.org/swift-book/GuidedTour/GuidedTour.html

```
let myConst = 42
var myVariable = 1
let implicitDobule = 70.0
let explicitDouble: Double = 70

// string interpolation using \()
var myStr = "My name is \(name)"
// indent is removed if it is the same as ending quotation marks
var myMultilineStr = """
I say "My name is \(name)"
"""
```

Arrays and dictionaries
```
var myArray = ["a", "b"]
var myDictionary = [
  "name": "Duke",
]
// to define empty arrays you need to declare type
let emptyArray: [String] = []
let emptyDictionary: [String: Float] = [:]
}

myArray.append("first")
```

Control flow for loops `for in`, `while`, `repeat while`
```
let scores = [1, 2]
var result = 0
for score in scores {
  if score > 1 {
    result += 2
  } else {
    result += 1
  }
}

// range of indexes
for i in 0..<4 { }
for i in 0...3 { }

for (_, name) in myDictionary {}

while n < 100 { n *= 2 }

repeat {
  m *= 2
} while m < 100
```

`if` statement requires boolean expression or using `let` with optional string
```
var optionalName: String? = "a"
if let name = optionalName {
}
optionalName = nil
```
Handle optional values with default value using `??` operator
```
let greeting = "Hi \(name ?? "guest")'
```
If class is optional than you can use `?` before calling a method or
property, `optionalSquare?.radius` returns an optional value.

Switch using case
```
let vegetable = "red pepper"
switch vegetable {
case "celery":
    print("Add some raisins and make ants on a log.")
case "cucumber", "watercress":
    print("That would make a good tea sandwich.")
case let x where x.hasSuffix("pepper"):
    print("Is it a spicy \(x)?")
default:
    print("Everything tastes good in soup.")
}
```

Functions and Closures
Arguments are parameter name and type, You can use custom argument label before
parameter name or `_` to use no argument label (otherwise you need to call a
function with parameter name like `greet(person: "John", day: "Wed")` if there
are more than one parameter)

```
func greet(_ person: String, on day: String) -> String {
  return "Hello \(person) today is \(day)"
}
print(greet("John", on: "Wed"))
```
Return value can be a tuple
```
func cal(scores: [Int]) -> (min: Int, max:  Int) {
  var min = scores[0]
  var max = scores[0]
  return (min, max)
}
```
You can nest functions.
Functions are first class type ie it can be return value from function.
```
func makeInc() -> ((Int) -> Int) {
  func add(number: Int) -> Int {
    return 1 + number
  }
  return add
}
var inc = makeInc()
print(inc(3))
```

Functions can take another function as one of its arguments
```
func hasAnyMatches(_ list: [Int], condition: (Int) -> Bool) -> Bool {
  for item in list {
    if condition(item) {
      return true
    }
  }
  return false
}

func lessThanTen(number: Int) -> Bool {
  return number < 10
}

var numbers = [30, 2, 12]
print(hasAnyMatches(numbers, condition: lessThanTen))
```

Functions are actually a special case of closures, blocks of code that can be
called later. Closure without a name is surrounded with braces, use `in` to
separate arguments and return type from the body
```
var numbers = [30, 2, 12]
print(numbers.map({ (number: Int) -> Int in
  return 3 * number
}))
```
When closure type is known (in callback for a delegate) you can omit return
type. Single line can omit return. You can also omit parameters and use `$0`
`$1`. You can omit parentheses if closure if the only argument.
```
var numbers = [30, 1, 12]
print(numbers.map({ number in 3 * number}))
print(numbers.map { 3 * $0})
var sorted = numbers.sorted { $0 > $1 }
print(sorted)
```

Classes uses same `let` for constants, `var` for instance property, except that
it's in the context of a class. Use `init()` for initializers (`deinit()` for
deinitializer for cleanup). If there is a parameter with same name, you can use
`self.name` (otherwise, it is not required when assigning instance property)
```
class NamedShape {
  var numberOfSides: Int = 0
  var name: String

  init(_ name: String) {
    self.name = name
  }

  func simpleDescription() -> String {
    return "The \(name) shape has \(numberOfSides) sides."
  }
}
var s = NamedShape("My")
s.numberOfSides = 2
print(s.simpleDescription())
```
Subclasses can `override` methods. You need to manually call `super.init()`.
```
class Circle: NamedShape {
  var radius: Int = 0

  init(radius: Int, name: String) {
    super.init(name)
    self.radius = radius
  }
  override func simpleDescription() -> String {
    return "The \(name) circle has \(radius) radius."
  }
}
```
In addition to simple instance properties, you can have a getter and a setter
(it uses `newValue` implicit name for setter). If you need to call a method when
changing the property, you can use `willSet` and `didSet`
```
class MyClass {
  var length: Int
  init(newLength length: Int) {
    length = newLength
  }

  var t: Int {
    get {
      return length
    }
    set {
      length = newValue / 3
    }
  }
}
```

Enumerations `enum` can have methods. It starts from zero unless explicitly
specifying value. Access raw values using `.rawValue` property.
Use `init?(rawValue:)` initializer to make an instance of an enumeration from a
raw value.

```
enum Rank: Int {
    case ace = 1
    case two, three

    func simpleDescription() -> String {
        switch self {
        case .ace:
            return "ACE"
        default:
            return String(self.rawValue)
        }
    }
}
let ace = Rank.ace
// or let ace: Rank = .ace
print(ace)
print(ace.rawValue)
print(ace.simpleDescription())
```
You can use values associated with the case, which are determined when you make
an instance, so they can be different for each instance of enumeration case
(like stored properties of the enumeration case instance)
```
enum ServerResponse {
    case result(String, String)
    case failure(String)
}

let success = ServerResponse.result("6:00 am", "8:09 pm")
let failure = ServerResponse.failure("Out of cheese.")

switch success {
case let .result(sunrise, sunset):
    print("Sunrise is at \(sunrise) and sunset is at \(sunset).")
case let .failure(message):
    print("Failure...  \(message)")
}
// Prints "Sunrise is at 6:00 am and sunset is at 8:09 pm."
```

Structires `struct` are similar to `class` just they are always copied, but
classes are passed by reference

Protocols can be declared with `protocol`. use `mutating` to mark a method that
modifies the structure.

```
protocol ExampleProtocol {
  var simpleDescription: String { get }
  mutating func adjust()
}

class SimpleClass: ExampleProtocol {
  var simpleDescription: String = "A very simple class"
  func adjust() {
    simpleDescription += " now adjusted"
  }
}
```

Use `extension` to add functionality to an existing type.
```
extension Int: ExampleProtocol {
    var simpleDescription: String {
        return "The number \(self)"
    }
    mutating func adjust() {
        self += 42
    }
}
print(7.simpleDescription)
// Prints "The number 7"
```

Error Handling is using any type that adopts `Error` protocol
Use `throw` to raise an error and `throws` to mark a function that can throw an
error. Code that called a function (with `try` prefix) handles the error with:
`do catch`, catch is similar to `switch Error case ...` automatic
name is `error`.
```
enum PrinterError: Error {
  case outOfPaper
  case onFire
}
func send(job: Int, printerName: String) throws -> String {
  if printerName == "Never has paper" {
    throw PrinterError.outOfPaper
  }
  return "Job \(job) sent"
}

do {
  let result = try send(job: 42, printerName: "Never has paper")
  print(result)
} catch PrinterError.onFire {
  print("Call the fireman")
} catch let printerError as PrinterError {
  print("Printer error: \(printerError)")
} catch {
  print(error)
}
```
Another way to handle errors is to use `try?` to convert the result to an
optional.
```
let result = try? send(job: 42, printerName: "Never has paper")
```
Use `defer` to write a block of code that's executed after all other code in the
function, just before function returns, it is executed regardless of whether the
function throws an error. Please use this cleanup code after setup block
```
var fridgeIsOpen = false
let fridgeContent = ["milk", "eggs"]
func fridgeContains(_ food: String) -> Bool {
  fridgeIsOpen = true
  defer {
    fridgeIsOpen = false
  }
  let result = fridgeContent.contains(food)
  return result
}
print(fridgeContains("banana"))
print(fridgeIsOpen)
```

Generic function is created using `<Name>` which is populated when it is called
```
func makeArray<Item>(_ item: Item, numberOfTimes: Int) -> [Item] {
  var result: [Item] = []
  for _ in 0..<numberOfTimes {
    result.append(item)
  }
  return result
}
print(makeArray("asd", numberOfTimes: 3))
print(makeArray(1, numberOfTimes: 0))
```
You can make generic forms of functions, methods, classes, enumeration and
structures
```
// Reimplement the Swift standard library's optional type
enum OptionalValue<Wrapped> {
  case none
  case some(Wrapped)
}
var possibleInteger: OptionalValue<Int> = .none
possibleInteger = .some(100)
```
Use `where` right before the body to specify a list of requirements, for example
require type to implement a protocol, require a class to have superclass
```
func anyCommonElements<T: Sequence, U: Sequence>(_ lhs: T, _ rhs: U) -> Bool
    where T.Element: Equatable, T.Element == U.Element
{
    for lhsItem in lhs {
        for rhsItem in rhs {
            if lhsItem == rhsItem {
                return true
            }
        }
    }
    return false
}
anyCommonElements([1, 2, 3], [3])
```

# Ubuntu

Use official vim support
```
git clone git@github.com:apple/swift.git --depth 1
mkdir ~/.vim/pack/bundle/start -p
cp vim ~/.vim/pack/bundle/start/swift -r
```

Install from https://www.swift.org/download/
extract to Programs and add Path
```
echo "PATH=~/Programs/swift-5.5.3-RELEASE-ubuntu20.04/usr/bin:$PATH" >> ~/.bashrc
. ~/.bashrc
```
Build
```
touch Package.swift
mkdir Sources
echo print("Hello, world") >> Sources/main.swift
swift build
# .build/debug/Hello
```
