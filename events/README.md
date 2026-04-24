- Category: DOM Manipulation
- Difficulty: Beginner
- Related: dom

### What are Events?
Events are actions or occurrences that happen in the system you are programming, which the system tells you about so your code can react to them (like a user clicking a button).

---

### 1. Event Listeners
**Theory**: You attach an "Event Listener" to a DOM element. It waits patiently until the specified event happens, then runs a callback function.

**Working Flow**
```text
[ Button ] --( User Clicks )--> [ Event Listener ] --( Triggers )--> [ Callback Function Runs ]
```

**Key Methods**:
- `addEventListener(event, function)`: The modern way to attach events.
- `removeEventListener(event, function)`: Removes an attached event.

**Step-by-Step Example**:
```javascript
// Step 1: Select the element
const button = document.getElementById("submit-btn");

// Step 2: Define what should happen
function handleClick() {
  console.log("Button was clicked!");
}

// Step 3: Attach the listener to the 'click' event
button.addEventListener("click", handleClick);
```

---

### 2. Event Bubbling
**Theory**: When an event happens on an element, it first runs the handlers on it, then on its parent, then all the way up on other ancestors.

**Working Flow**
```text
[ div ] <-- ( Event bubbles up here )
   |
[ button ] <-- ( User clicks here )
```

**Step-by-Step Example**:
```javascript
const div = document.querySelector("div");
const button = document.querySelector("button");

// If you click the button, BOTH events will fire!
div.addEventListener("click", () => console.log("Div Clicked"));
button.addEventListener("click", (event) => {
  console.log("Button Clicked");
  
  // Stop the event from bubbling up to the div!
  event.stopPropagation(); 
});
```
