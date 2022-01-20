# combine_transforming_operators

### Collect

```swift
["A", "B", "C", "D"]
    .publisher
    .sink {
        // individual items
        print($0) // A, B, C, D
    }
        
["A", "B", "C", "D"]
    .publisher
    .collect()
    .sink {
        // array
        print($0) // [A, B, C, D]
    }

["A", "B", "C", "D"]
    .publisher
    .collect(2)
    .sink {
        // array
        print($0) // [A, B], [C, D]
    }
```

### Map

```swift
[123, 45, 67]
    .publisher
    .map {
        formatter.string(from: NSNumber(integerLiteral: $0)) ?? ""
    }
    .sink {
        print($0) // spelled out numbers
        // one hundred twenty-three
        // forty-five
        // sixty-sevent 
    }
```

### Map - key path structure

```swift
struct Point {
	  let x: Int
	  let y: Int
}

let publihser = PassthroughSubject<Point, Never>()
publihser.map(\.x, \.y)
    .sink { x, y in
        print("x is \(x), y is \(y)")
    }

publihser.send(Point(x: 2, y: 10))
```

### Flat Map

- Single down stream
- See internal internal publication events

```swift
import Combine

struct School {

    let name: String
    let noOfStudents: CurrentValueSubject<Int, Never> //Publisher, Returns Single Value
    
    init(name: String, noOfStudents: Int) {
        self.name = name
        self.noOfStudents = CurrentValueSubject(noOfStudents)
    }
    
}

let citySchool = School(name: "Fountain Head School", noOfStudents: 100)
let school = CurrentValueSubject<School, Never>(citySchool)

// this is fired when only school is changed
school.sink {
    print($0)
}

school
    // get internal changes
    .flatMap {
        $0.noOfStudents
        // 100
        // 45
        // 101
    }
    .sink {
        print($0)
    }

let townSchool = School(name: "Town Head School", noOfStudents: 45)

school.value = townSchool // calls school.sink

citySchool.noOfStudents.value += 1 // this doesn't fire anything
townSchool.noOfStudents.value += 10
```

### replaceNil

```swift
import Foundation
import Combine

// Optional A, B, *, nil
["A", "B", nil, "C"]
    .publisher
    .replaceNil(with: "*")
    .sink { print($0)}
```

### unwrapping optional

```swift
import Foundation
import Combine

// A, B, C, D
["A", "B", nil, "C"]
    .publisher
    .replaceNil(with: "*")
    .map { $0! }
    .sink { print($0)}
```

### replaceEmpty

```swift
import Foundation
import Combine

let empty = Empty<Int, Never>()

empty
    .replaceEmpty(with: 1)
    .sink(receiveCompletion: {print($0)}, receiveValue: {print($0)})
```

### scan

```swift
import Foundation
import Combine

let publisher = (1...10).publisher

publisher
// initial value
    .scan([]) { numbers, value -> [Int] in
        print("numbers: \(numbers)")
        print("value: \(value)")
        print("number + values: \(numbers + [value])")
    return numbers + [value] // adding array to an array
}.sink {
    print($0)
}

/*
 [1]
 [1, 2]
 [1, 2, 3]
 [1, 2, 3, 4]
...
 */
```
