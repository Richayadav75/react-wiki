- Category: JavaScript
- Difficulty: Beginner
- Related: object-methods

### What is JSON?
JSON (JavaScript Object Notation) is a lightweight format used for storing and transporting data, mostly when communicating between a web server and a browser.

---

### 1. JSON vs JS Object
**Theory**: JSON looks exactly like a JavaScript Object, but it is purely **Text (a String)**. All keys must be wrapped in double quotes.

**Working Flow**
```text
[ JS Object ] --( stringify )--> [ JSON String (Sent over network) ]
[ JSON String ] --( parse )----> [ JS Object (Usable in code) ]
```

**Key Methods**:
- `JSON.stringify(obj)`: Converts a JS Object into a JSON String.
- `JSON.parse(string)`: Converts a JSON String back into a JS Object.

**Step-by-Step Example**:
```javascript
// Step 1: A normal JavaScript Object
const user = { name: "Richa", age: 25 };

// Step 2: Convert to JSON to send to a server
const jsonString = JSON.stringify(user);
console.log(jsonString); 
// '{"name":"Richa","age":25}' (It is now a string!)

// Step 3: Receive JSON from a server and convert it back
const receivedData = '{"name":"John","age":30}';
const parsedUser = JSON.parse(receivedData);
console.log(parsedUser.name); // "John"
```
