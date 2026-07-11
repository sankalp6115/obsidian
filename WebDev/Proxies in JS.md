A Proxy sits _in front of_ an object. Anyone interacting with the proxy thinks they're talking to the real object, but every interaction gets intercepted first. The original object is untouched and unaware.

```js
const user = {name, age, nationality}
```

Need rules, like age be number, name be string and no special chars etc

**Naive Approach ->** 
whenever object used, immediately using if checks, 
Results in scattered if checks and a lot of places for changes if object properties changed.

**Proxies** 
Simplify by centrally handling these in one place, define rules in one place, and each time the object is used, rules are already applied.
Proxy wraps an object in an invisible control layer. Hits every time object is interacted with.

Takes two things : target and trap
Target is the object.
Trap is a plain object with methods like:

**get()** trap fired every time object is read.
**set()** trap fires every time object is changed.

Trap container is a plain object with method names like `get`, `set`, `has`, `delete` etc. as keys. Each method is one trap.

**Code**
```js
const user = {name: "John", age: 42};

const proxy = new Proxy(user, {
	get(obj, prop){
		if(!(prop in obj)) return `no ${prop}`;
		return Reflect.get(obj,prop);
	},
	set(obj,prop,value){
		if(prop == "age" && typeof value !== "number"){
			throw TypeError("bad age");
		}
		return Reflect.set(obj,prop,value);
	}
})
```

**Reflect API**
- `Reflect.get(obj, prop)` is just the "default, normal" way to read a property
- Inside a trap, you can't just do `return obj[prop]` — that can cause bugs with `this` binding and inheritance
- `Reflect` is basically saying _"now do what you would've done normally"_ after your custom logic runs
Works like this - intercept, do your checks, then hand back control to the default behaviour.

### Usage Areas

**Vue3 Reactivity System** 
Tracks changes using proxy. When you change a value, Vue knows because the Proxy trap fired, which then triggers re-renders.
**Information validation to guard any inputs**
**Logging**

### Always a catch
Every access now runs trap function
Slower on hot paths, overhead per object access
Hidden indirection, looks like plain object
Not perf-critical code, skip it in tight loops
