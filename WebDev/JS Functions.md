# 🔹 1. String Functions (most used)

### Basic
```javascript
let str = "Hello World";
str.length              // 11
str.toUpperCase()       // "HELLO WORLD"
str.toLowerCase()       // "hello world"
```

### Searching
```javascript
str.indexOf("o")        // 4
str.lastIndexOf("o")    // 7
str.includes("World")   // true
```

### Extracting
```javascript
str.slice(0, 5)         // "Hello"
str.substring(0, 5)     // "Hello"
```

### Replace & Split
```javascript
str.replace("World", "JS")   // "Hello JS"
str.split(" ")               // ["Hello", "World"]
```

### Trim
```javascript
"  hi  ".trim()        // "hi"
```

---

# 🔹 2. Array Functions (very important)

### Basic
```javascript
let arr = [1, 2, 3];
arr.length          // 3
arr.push(4)         // [1,2,3,4]
arr.pop()           // removes last
arr.shift()         // removes first
arr.unshift(0)      // add at start
```

### Looping
```javascript
arr.forEach(x => console.log(x));
```

### Transform
```javascript
arr.map(x => x * 2)        // [2,4,6]
arr.filter(x => x > 1)     // [2,3]
```

### Searching
```javascript
arr.find(x => x > 1)       // 2
arr.includes(2)            // true
arr.indexOf(2)             // 1
```

### Other useful
```javascript
arr.join("-")              // "1-2-3"
arr.slice(1, 3)            // [2,3]
arr.splice(1, 1)           // removes element
```

---

# 🔹 3. Math Functions

### Basic
```javascript
Math.round(4.6)      // 5
Math.floor(4.6)      // 4
Math.ceil(4.2)       // 5
```

### Min / Max
```javascript
Math.max(1, 5, 3)    // 5
Math.min(1, 5, 3)    // 1
```

### Power & Root
```javascript
Math.pow(2, 3)       // 8
Math.sqrt(16)        // 4
```

### Random
```javascript
Math.random()        // 0 to 1
// Random number from 1 to 10
Math.floor(Math.random() * 10) + 1;
```

### Constants
```javascript
Math.PI
Math.E
```