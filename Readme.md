# JavaScript Prototypes — A Complete Guide for Beginners

> **Who is this for?**
> You know basic JavaScript. You've written functions, objects, maybe some arrays. But when someone says "prototype chain" or "prototypal inheritance" — you zone out. This guide is written for you.
>
> We'll go slow. We'll use plain English before any code. And we'll use **one story throughout** so your brain doesn't have to keep resetting.

---

## Table of Contents

1. [The Story We'll Use Throughout This Guide](#1-the-story-well-use-throughout-this-guide)
2. [First — What is an Object? (Real quick)](#2-first--what-is-an-object-real-quick)
3. [The Problem That Prototypes Solve](#3-the-problem-that-prototypes-solve)
4. [What is a Prototype? (Plain English First)](#4-what-is-a-prototype-plain-english-first)
5. [Constructor Functions — Creating Many Objects](#5-constructor-functions--creating-many-objects)
6. [The `prototype` Property on Functions](#6-the-prototype-property-on-functions)
7. [The `__proto__` Link on Objects](#7-the-__proto__-link-on-objects)
8. [How JavaScript Looks Up a Property](#8-how-javascript-looks-up-a-property)
9. [The Prototype Chain — The Full Picture](#9-the-prototype-chain--the-full-picture)
10. [Prototypal Inheritance — One Object Inheriting from Another](#10-prototypal-inheritance--one-object-inheriting-from-another)
11. [Object.create() — The Cleanest Way](#11-objectcreate--the-cleanest-way)
12. [ES6 Classes — Same Thing, Nicer Syntax](#12-es6-classes--same-thing-nicer-syntax)
13. [Adding Methods to Built-in Types](#13-adding-methods-to-built-in-types)
14. [Practical Real-World Patterns](#14-practical-real-world-patterns)
15. [Interview Questions — Scenario Based](#15-interview-questions--scenario-based)
16. [Common Mistakes and Bugs](#16-common-mistakes-and-bugs)
17. [Quick Reference Cheatsheet](#17-quick-reference-cheatsheet)

---

## 1. The Story We'll Use Throughout This Guide

Imagine you're building software for a **school**. This school has students. Lots of them.

Every student has:
- A name
- A roll number
- The ability to study
- The ability to give an exam

We'll use this school example for the **entire guide**. Every new concept will be explained using students. This way you won't have to mentally reset every time a new idea comes in.

Let's start.

---

## 2. First — What is an Object? (Real quick)

In JavaScript, an object is just a **container of related data and actions**.

```js
const student = {
  name: "Ravi",
  rollNo: 101,
  study: function() {
    console.log(name + " is studying");
  }
};
```

That's it. `student` is an object with two pieces of data (`name`, `rollNo`) and one action (`study`).

You access them like this:

```js
student.name;      // "Ravi"
student.rollNo;    // 101
student.study();   // "Ravi is studying"
```

Simple. You already know this. Now let's move forward.

---

## 3. The Problem That Prototypes Solve

Now, our school doesn't have 1 student. It has **500 students**.

The obvious approach: create 500 objects.

```js
const student1 = {
  name: "Ravi",
  rollNo: 101,
  study: function() { console.log(this.name + " is studying"); },
  giveExam: function() { console.log(this.name + " is giving exam"); }
};

const student2 = {
  name: "Priya",
  rollNo: 102,
  study: function() { console.log(this.name + " is studying"); },
  giveExam: function() { console.log(this.name + " is giving exam"); }
};

// ... 498 more students
```

Do you see the problem?

**The `study` and `giveExam` functions are identical in every object.** But JavaScript is storing a separate copy of each function for every single student.

500 students × 2 functions = **1000 function copies in memory**.

They all do the exact same thing. This is pure waste.

---

### What We Actually Want

We want:
- Each student to have their **own** `name` and `rollNo` (because those are different)
- All students to **share** `study` and `giveExam` (because those are the same)

```
student1 ──→ { name: "Ravi",  rollNo: 101 }
student2 ──→ { name: "Priya", rollNo: 102 }
student3 ──→ { name: "Amit",  rollNo: 103 }
                  ↓ all three point to ↓
             SHARED: { study: fn, giveExam: fn }
```

This is exactly what **prototypes** do.

---

## 4. What is a Prototype? (Plain English First)

> **A prototype is just an object that other objects can borrow things from.**

That's it.

Think of it like a **template shelf** in a school office. The shelf has the stamp, the letterhead, the attendance sheet format. Every teacher borrows from that shelf — they don't each carry their own personal copy.

In JavaScript:
- The "shelf" = the **prototype object**
- The "borrowing" = the **prototype link** (`__proto__`)
- The teachers = the **instances** (individual student objects)

When you ask `student1.study()`, JavaScript first checks: "Does `student1` itself have a `study` method?" If not, it goes to the prototype and borrows it from there.

Now let's see this in actual code — step by step.

---

## 5. Constructor Functions — Creating Many Objects

Instead of writing 500 objects by hand, we use a **constructor function**.

A constructor function is just a regular function that you call with the `new` keyword to create objects.

```js
function Student(name, rollNo) {
  this.name = name;
  this.rollNo = rollNo;
}
```

> **What is `this` here?**
> When you call a function with `new`, JavaScript creates a fresh empty object and assigns it to `this`. So `this.name = name` means "put `name` on this new object".

Now let's create students:

```js
const student1 = new Student("Ravi", 101);
const student2 = new Student("Priya", 102);
const student3 = new Student("Amit", 103);
```

Check what each one looks like:

```js
console.log(student1); // Student { name: 'Ravi', rollNo: 101 }
console.log(student2); // Student { name: 'Priya', rollNo: 102 }
```

Each student has their own `name` and `rollNo`. 

But where do we put the shared methods? This is where `prototype` comes in.

---

### What `new` Does Behind the Scenes

Before moving on, let's understand exactly what `new` does. This is important and comes up in interviews.

When you write `new Student("Ravi", 101)`, JavaScript does these 4 things for you:

```
Step 1: Create an empty object {}
Step 2: Link that object to Student.prototype (we'll explain this soon)
Step 3: Run the Student function with `this` = that empty object
Step 4: Return the object
```

In "manual code" it would look like this:

```js
// This is what `new` does internally — you don't write this, JS does it
const obj = {};                          // Step 1
obj.__proto__ = Student.prototype;       // Step 2
Student.call(obj, "Ravi", 101);          // Step 3
// Step 4: return obj
```

You don't need to memorize this code right now. Just understand: **`new` creates an object and links it to the function's prototype**. We'll come back to this.

---

## 6. The `prototype` Property on Functions

Every function in JavaScript automatically gets a property called `prototype`.

```js
function Student(name, rollNo) {
  this.name = name;
  this.rollNo = rollNo;
}

console.log(Student.prototype); // { constructor: [Function: Student] }
```

Right now it just has `constructor` (which points back to the `Student` function itself). But we can add our shared methods here:

```js
Student.prototype.study = function() {
  console.log(this.name + " is studying");
};

Student.prototype.giveExam = function() {
  console.log(this.name + " is giving the exam");
};
```

Now create students and call the methods:

```js
const student1 = new Student("Ravi", 101);
const student2 = new Student("Priya", 102);

student1.study();    // "Ravi is studying"
student2.study();    // "Priya is studying"
student2.giveExam(); // "Priya is giving the exam"
```

It works! But here's the magic part — **`study` is not stored on `student1` or `student2`**. Let's verify:

```js
console.log(student1.hasOwnProperty("name"));   // true  ← on student1 itself
console.log(student1.hasOwnProperty("rollNo"));  // true  ← on student1 itself
console.log(student1.hasOwnProperty("study"));   // false ← NOT on student1!
```

So where is `study`? It's on `Student.prototype`. And `student1` can access it because of the invisible link called `__proto__`.

---

### So in simple words:

> `prototype` is a property **on the constructor function**. It's the shared shelf where all instances go to borrow methods from.

---

## 7. The `__proto__` Link on Objects

Every object in JavaScript has a hidden property called `__proto__`. It points to the prototype of whoever created it.

```js
const student1 = new Student("Ravi", 101);

console.log(student1.__proto__ === Student.prototype); // true
```

Let's read that line in plain English:

> "The `__proto__` of `student1` is the same object as `Student.prototype`"

This is the **link**. This is how `student1` finds `study` even though it doesn't have `study` itself.

```
student1 object:
  ├── name: "Ravi"
  ├── rollNo: 101
  └── __proto__ ────────→ Student.prototype:
                               ├── study: function
                               ├── giveExam: function
                               └── constructor: Student
```

When you call `student1.study()`, JavaScript does this:

1. Look for `study` on `student1` itself → not found
2. Follow `__proto__` to `Student.prototype` → found! Run it.

That's called **property lookup** and it's the core of how prototypes work.

---

### `__proto__` vs `prototype` — The Confusion Cleared

This is where most beginners get confused. Let's kill this confusion right now.

| | `prototype` | `__proto__` |
|---|---|---|
| **What is it?** | A property on **functions** | A property on **objects** |
| **What does it hold?** | The shared object where methods live | A link to the parent object |
| **Who has it?** | Only functions (constructors) | Every object instance |

```js
// prototype is on the FUNCTION:
Student.prototype   // ✅ makes sense
student1.prototype  // undefined ❌ (student1 is not a function)

// __proto__ is on the OBJECT (instance):
student1.__proto__  // ✅ makes sense → points to Student.prototype
Student.__proto__   // also exists but points to Function.prototype (advanced, ignore for now)
```

---

### Modern way to read `__proto__`:

`__proto__` is older syntax. The modern, cleaner way:

```js
Object.getPrototypeOf(student1); // same as student1.__proto__
```

Both work. `__proto__` is more readable for learning, `Object.getPrototypeOf` is preferred in real code.

---

## 8. How JavaScript Looks Up a Property

This is a critical concept. When you access any property on an object, JavaScript follows a specific process. Let's walk through it.

```js
student1.study();
```

**JavaScript's thought process:**

```
"Does student1 have a property called 'study' directly on itself?"
  → Check student1's own properties: name, rollNo
  → No 'study' found.

"Okay, follow student1.__proto__"
  → We're now on Student.prototype
  → Does Student.prototype have 'study'?
  → YES! Found it. Run it with this = student1.
```

One more example:

```js
student1.toString(); // works! But where is toString?
```

```
"Does student1 have toString?"  → No
"Does Student.prototype have toString?" → No
"Does Student.prototype.__proto__ have toString?" → YES
  → Student.prototype.__proto__ is Object.prototype
  → Object.prototype has toString
  → Found it!
```

If nothing is found anywhere, JavaScript returns `undefined`.

---

## 9. The Prototype Chain — The Full Picture

The chain doesn't stop at `Student.prototype`. It keeps going up.

```
student1
  └── __proto__ → Student.prototype
                    └── __proto__ → Object.prototype
                                      └── __proto__ → null
                                                       (END OF CHAIN)
```

`Object.prototype` is the **top of the chain** for almost everything in JavaScript. It contains methods like:
- `toString()`
- `hasOwnProperty()`
- `valueOf()`

That's why every object in JavaScript has these methods — because they all eventually reach `Object.prototype` through the chain.

Let's verify this:

```js
console.log(student1.__proto__ === Student.prototype);            // true
console.log(Student.prototype.__proto__ === Object.prototype);    // true
console.log(Object.prototype.__proto__);                          // null — end of chain
```

---

### The `in` operator vs `hasOwnProperty`

Now that you understand the chain, these two make perfect sense:

```js
// `in` checks the ENTIRE chain (own + prototype + Object.prototype + ...)
console.log("name" in student1);       // true (own property)
console.log("study" in student1);      // true (on Student.prototype)
console.log("toString" in student1);   // true (on Object.prototype)

// hasOwnProperty checks ONLY the object itself
console.log(student1.hasOwnProperty("name"));     // true
console.log(student1.hasOwnProperty("study"));    // false (it's on prototype, not own)
console.log(student1.hasOwnProperty("toString")); // false
```

---

### `instanceof` — checking the chain

```js
student1 instanceof Student; // true
student1 instanceof Object;  // true (because Object.prototype is in the chain)
```

`instanceof` checks: "Is `Student.prototype` anywhere in the prototype chain of `student1`?"

---

## 10. Prototypal Inheritance — One Object Inheriting from Another

So far we've seen one level of inheritance (Student → Student.prototype).

Now let's go one level deeper: what if we want a **special type of student**?

Our school has regular students and **monitor students**. A monitor student can do everything a regular student can, but also has an extra ability: `conductClass()`.

We don't want to rewrite `study` and `giveExam` for monitors. We want monitors to **inherit** from Student.

### The goal:

```
monitor1
  └── __proto__ → Monitor.prototype
                    └── __proto__ → Student.prototype
                                      └── __proto__ → Object.prototype
                                                          └── null
```

Now `monitor1` has access to both Monitor methods AND Student methods.

### Step 1: Create the Monitor constructor

```js
function Monitor(name, rollNo, section) {
  // Call the Student constructor to set name and rollNo
  Student.call(this, name, rollNo);
  // Add monitor-specific data
  this.section = section;
}
```

> **What is `Student.call(this, name, rollNo)`?**
> It's like saying: "Hey Student function, run yourself, but use MY `this` instead of your own. And here are the arguments."
> This makes sure `name` and `rollNo` get set on the new Monitor object.

### Step 2: Link Monitor.prototype to Student.prototype

```js
Monitor.prototype = Object.create(Student.prototype);
```

> **What does `Object.create(Student.prototype)` do?**
> It creates a new empty object whose `__proto__` is `Student.prototype`. We'll explain `Object.create` fully in the next section.

### Step 3: Fix the constructor reference

```js
Monitor.prototype.constructor = Monitor;
```

Why? Because Step 2 replaced `Monitor.prototype` entirely, which lost the `constructor` reference. We put it back.

### Step 4: Add Monitor-specific methods

```js
Monitor.prototype.conductClass = function() {
  console.log(this.name + " is conducting the class");
};
```

### Using it:

```js
const monitor1 = new Monitor("Sneha", 105, "A");

monitor1.study();         // "Sneha is studying"        ← from Student.prototype
monitor1.giveExam();      // "Sneha is giving the exam" ← from Student.prototype
monitor1.conductClass();  // "Sneha is conducting the class" ← from Monitor.prototype

console.log(monitor1.name);    // "Sneha"   ← own property
console.log(monitor1.section); // "A"       ← own property
```

### Checking the inheritance:

```js
monitor1 instanceof Monitor;  // true
monitor1 instanceof Student;  // true ← inherits from Student!
monitor1 instanceof Object;   // true
```

### What just happened? (The chain visualized)

```
monitor1
  ├── name: "Sneha"       ← own
  ├── rollNo: 105         ← own
  ├── section: "A"        ← own
  └── __proto__ ──────────→ Monitor.prototype
                               ├── conductClass: fn  ← Monitor's own method
                               ├── constructor: Monitor
                               └── __proto__ ────────→ Student.prototype
                                                          ├── study: fn
                                                          ├── giveExam: fn
                                                          └── __proto__ → Object.prototype
```

When `monitor1.study()` is called:
1. Not on `monitor1` itself → go to `Monitor.prototype`
2. Not on `Monitor.prototype` → go to `Student.prototype`
3. Found `study` here! → Run it with `this = monitor1`

---

## 11. Object.create() — The Cleanest Way

`Object.create(someObject)` creates a **new empty object** whose `__proto__` is set to `someObject`.

```js
const studentMethods = {
  study: function() {
    console.log(this.name + " is studying");
  },
  giveExam: function() {
    console.log(this.name + " is giving the exam");
  }
};

const student1 = Object.create(studentMethods);
student1.name = "Ravi";
student1.rollNo = 101;

student1.study(); // "Ravi is studying" ✅
```

What happened:
- `student1` is a new empty object
- `student1.__proto__` is `studentMethods`
- So `student1` can access `study` and `giveExam` through the chain

```js
console.log(student1.__proto__ === studentMethods); // true
```

### Why use `Object.create` over `new`?

With `new`, the constructor function runs. Sometimes you just want to set up the prototype link **without running any constructor logic**. That's when `Object.create` is perfect.

We already used it in the inheritance setup:

```js
// This just sets up the link — no constructor called, no side effects
Monitor.prototype = Object.create(Student.prototype);
```

If we had written `Monitor.prototype = new Student()` instead:

```js
// ❌ Don't do this:
Monitor.prototype = new Student(); // Student() runs! name = undefined, rollNo = undefined
```

That would call `Student` without arguments, which is wrong. `Object.create` is the clean, correct way.

---

### `Object.create(null)` — An object with NO prototype

```js
const pureObj = Object.create(null);
pureObj.key = "value";

console.log(pureObj.toString); // undefined — no Object.prototype in chain!
```

This creates an object with absolutely no prototype chain. Used for pure data storage (like a dictionary) where you don't want any inherited methods interfering.

---

## 12. ES6 Classes — Same Thing, Nicer Syntax

In 2015, JavaScript introduced `class` syntax. A lot of people think classes are something new. They are NOT. Classes are just a cleaner way to write the same prototype-based code we've been writing.

Let's rewrite our Student example using class:

```js
class Student {
  constructor(name, rollNo) {
    this.name = name;
    this.rollNo = rollNo;
  }

  study() {
    console.log(this.name + " is studying");
  }

  giveExam() {
    console.log(this.name + " is giving the exam");
  }
}
```

That's it. Clean and readable. Let's use it:

```js
const student1 = new Student("Ravi", 101);
student1.study();    // "Ravi is studying"
student1.giveExam(); // "Ravi is giving the exam"
```

### Proving it's still prototype under the hood:

```js
console.log(typeof Student); // "function" ← it's still a function!

console.log(student1.__proto__ === Student.prototype); // true ← same as before
console.log(student1.hasOwnProperty("study")); // false ← study is on prototype, not own
```

Identical behavior. Classes are **100% syntax sugar** over the prototype system.

---

### Inheritance with classes — `extends` and `super`

```js
class Monitor extends Student {
  constructor(name, rollNo, section) {
    super(name, rollNo); // ← calls Student's constructor
    this.section = section;
  }

  conductClass() {
    console.log(this.name + " is conducting the class");
  }
}
```

> **What is `super()`?**
> `super(name, rollNo)` calls the parent class constructor (`Student`). It's the cleaner version of `Student.call(this, name, rollNo)` that we used earlier.
> In a class that extends another, you **must** call `super()` before using `this`. Otherwise JavaScript throws an error.

```js
const monitor1 = new Monitor("Sneha", 105, "A");
monitor1.study();        // "Sneha is studying" ✅
monitor1.conductClass(); // "Sneha is conducting the class" ✅
```

### The proof that it's still prototype:

```js
console.log(Monitor.prototype.__proto__ === Student.prototype); // true
```

`extends` literally just sets up `Monitor.prototype.__proto__ = Student.prototype`. Same thing we did manually.

---

### `static` methods — on the class, not on instances

```js
class Student {
  constructor(name, rollNo) {
    this.name = name;
    this.rollNo = rollNo;
  }

  static totalStudents() {
    return "We have many students!";
  }
}

Student.totalStudents(); // "We have many students!" ✅

const s = new Student("Ravi", 101);
s.totalStudents(); // ❌ TypeError — static methods are NOT on instances
```

Static methods live on the class (constructor function) itself, not on `prototype`. So instances can't access them.

---

## 13. Adding Methods to Built-in Types

JavaScript's built-in types like `Array`, `String`, `Number` also use prototypes. That means you can **add your own methods** to them.

### Adding a method to ALL arrays:

```js
Array.prototype.last = function() {
  return this[this.length - 1];
};

const marks = [85, 90, 78, 95];
marks.last(); // 95
```

### How does this work?

When you write `marks.last()`:
1. `marks` doesn't have `last` → follow `__proto__`
2. `marks.__proto__` is `Array.prototype`
3. We just added `last` to `Array.prototype` → Found! Run it.

### Adding a method to ALL strings:

```js
String.prototype.shout = function() {
  return this.toUpperCase() + "!!!";
};

"hello".shout(); // "HELLO!!!"
"ravi".shout();  // "RAVI!!!"
```

### Adding a method to ALL numbers:

```js
Number.prototype.isEven = function() {
  return this % 2 === 0;
};

(4).isEven(); // true
(7).isEven(); // false
```

> **Why does `"hello".shout()` work on a primitive string?**
> Strings are primitives, not objects. But when you call a method on a string, JavaScript **temporarily wraps it** in a `String` object, finds the method on `String.prototype`, runs it, then throws the wrapper away. This is called "autoboxing".

### ⚠️ Important Warning

Adding methods to built-in prototypes in **library/package code** is risky. If two libraries both add `Array.prototype.last`, they'll conflict. In your own personal projects or for learning, it's fine.

---

## 14. Practical Real-World Patterns

### Pattern 1: Mixin — Giving objects abilities from multiple sources

JavaScript's prototype chain is single — an object can only have one direct `__proto__`. But sometimes you want to give an object abilities from multiple places.

A **mixin** is just copying methods from one object onto another's prototype.

```js
// Mixin 1: Can serialize to JSON
const Serializable = {
  toJSON() {
    return JSON.stringify(this);
  }
};

// Mixin 2: Can validate itself
const Validatable = {
  isValid() {
    return this.name && this.name.length > 0;
  }
};

class Student {
  constructor(name, rollNo) {
    this.name = name;
    this.rollNo = rollNo;
  }
}

// Mix both into Student
Object.assign(Student.prototype, Serializable, Validatable);

const s = new Student("Ravi", 101);
s.toJSON();   // '{"name":"Ravi","rollNo":101}'
s.isValid();  // true
```

`Object.assign` just copies properties from one object to another. Here we're copying them directly onto `Student.prototype`, so all students get these abilities.

---

### Pattern 2: Method Chaining using prototype

A clean pattern where each method returns `this` so you can chain calls.

```js
function StudentBuilder(name) {
  this.name = name;
  this.courses = [];
}

StudentBuilder.prototype.addCourse = function(course) {
  this.courses.push(course);
  return this; // ← key! return this to allow chaining
};

StudentBuilder.prototype.setRollNo = function(rollNo) {
  this.rollNo = rollNo;
  return this;
};

StudentBuilder.prototype.build = function() {
  console.log(`${this.name} | Roll: ${this.rollNo} | Courses: ${this.courses.join(", ")}`);
  return this;
};

const s = new StudentBuilder("Priya")
  .setRollNo(102)
  .addCourse("Math")
  .addCourse("Science")
  .addCourse("English")
  .build();
// "Priya | Roll: 102 | Courses: Math, Science, English"
```

You see this pattern in jQuery, Mongoose, and many libraries.

---

### Pattern 3: Extending a class and overriding methods

Sometimes a child class wants to do what the parent does, plus a little extra.

```js
class Student {
  introduce() {
    return `I am a student named ${this.name}`;
  }
}

class Monitor extends Student {
  constructor(name, rollNo, section) {
    super(name, rollNo);
    this.section = section;
  }

  introduce() {
    // Call parent's introduce first, then add more
    const base = super.introduce();
    return `${base} and I am the monitor of section ${this.section}`;
  }
}

const m = new Monitor("Sneha", 105, "A");
m.introduce();
// "I am a student named Sneha and I am the monitor of section A"
```

`super.introduce()` calls the parent class version of `introduce`. Then we extend it.

---

### Pattern 4: Polyfill — How libraries add missing features

A **polyfill** is code that adds a feature to an environment that doesn't support it natively. Most polyfills work by adding to built-in prototypes.

```js
// Adding Array.prototype.includes if it doesn't exist
if (!Array.prototype.includes) {
  Array.prototype.includes = function(item) {
    for (let i = 0; i < this.length; i++) {
      if (this[i] === item) return true;
    }
    return false;
  };
}

[1, 2, 3].includes(2); // true
```

This is literally how polyfill libraries like `core-js` work — they check if a method exists on the prototype, and add it if it doesn't.

---

### Pattern 5: Config inheritance with Object.create

A clean way to have default settings that child configs can override:

```js
const defaultConfig = {
  theme: "light",
  language: "en",
  fontSize: 14,
  notifications: true
};

// Student-specific config — inherits defaults, overrides only what's needed
const studentConfig = Object.create(defaultConfig);
studentConfig.theme = "dark"; // override just this

console.log(studentConfig.theme);         // "dark"        ← own property
console.log(studentConfig.language);      // "en"          ← from defaultConfig
console.log(studentConfig.fontSize);      // 14            ← from defaultConfig
console.log(studentConfig.notifications); // true          ← from defaultConfig

// Change a default — all configs that inherit see the change
defaultConfig.fontSize = 16;
console.log(studentConfig.fontSize); // 16 ← updated automatically
```

---

## 15. Interview Questions — Scenario Based

These are the kinds of questions companies actually ask. Read each one, understand the question, think about it, then see the answer.

---

### Q1: What will this output and why?

```js
function Student(name) {
  this.name = name;
}
Student.prototype.greet = function() {
  return "Hi, I am " + this.name;
};

const s1 = new Student("Ravi");
const s2 = new Student("Priya");

Student.prototype.greet = function() {
  return "Hello, I am " + this.name + "!";
};

console.log(s1.greet());
console.log(s2.greet());
```

**Think:** `s1` and `s2` were created before `greet` was changed. Do they remember the old version?

**Answer:**
```
"Hello, I am Ravi!"
"Hello, I am Priya!"
```

Both print the **new** version. Because `s1` and `s2` don't store `greet` themselves — they look it up on `Student.prototype` every time they call it. When you change `Student.prototype.greet`, both see the updated version immediately.

---

### Q2: What will this output?

```js
function Student(name) {
  this.name = name;
}
Student.prototype.marks = [];

const s1 = new Student("Ravi");
const s2 = new Student("Priya");

s1.marks.push(90);

console.log(s1.marks);
console.log(s2.marks);
```

**Answer:**
```
[90]
[90]
```

**Why?** `marks` is an array on `Student.prototype`. Both `s1` and `s2` share the **same** array. When `s1.marks.push(90)`, it mutates the shared array — so `s2.marks` also shows `[90]`.

This is a classic bug. **Never put mutable data (arrays, objects) on prototype.**

**Fix:**
```js
function Student(name) {
  this.name = name;
  this.marks = []; // each student gets their OWN array
}
```

---

### Q3: What is the output?

```js
function Animal(sound) {
  this.sound = sound;
}
Animal.prototype.speak = function() {
  console.log(this.sound);
};

function Dog(name) {
  this.name = name;
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

const d = new Dog("Bruno");
d.speak();
```

**Answer:** `undefined`

**Why?** `d.speak()` runs. Inside `speak`, `this` is `d`. But `d` never set `this.sound` — the `Animal` constructor was never called! We set up the prototype link but forgot `Animal.call(this, sound)` in `Dog`'s constructor.

**Fix:**
```js
function Dog(name, sound) {
  Animal.call(this, sound); // ← must call parent constructor!
  this.name = name;
}
```

---

### Q4: What will this output?

```js
const obj = {
  name: "Ravi",
  greet: () => {
    console.log("Hello, I am " + this.name);
  }
};

obj.greet();
```

**Answer:** `"Hello, I am undefined"`

**Why?** Arrow functions don't have their own `this`. `this` inside an arrow function refers to whatever `this` was in the surrounding scope at the time the arrow function was **defined** — which here is the global scope (or `undefined` in strict mode).

**Fix:** Use a regular function:
```js
greet: function() {
  console.log("Hello, I am " + this.name);
}
// or
greet() {
  console.log("Hello, I am " + this.name);
}
```

---

### Q5: Detached method — what happens?

```js
function Student(name) {
  this.name = name;
}
Student.prototype.study = function() {
  console.log(this.name + " is studying");
};

const s = new Student("Ravi");

const studyFn = s.study; // take the method out
studyFn(); // call it separately
```

**Answer:** `"undefined is studying"` (or TypeError in strict mode)

**Why?** When you do `s.study()`, JavaScript calls `study` with `this = s`. But when you do `studyFn()`, there's no context — `this` becomes `window` (or `undefined` in strict mode). `window.name` is `""` in browser, `undefined` in Node.

**Fix:** Use `bind`:
```js
const studyFn = s.study.bind(s); // lock `this` to s
studyFn(); // "Ravi is studying" ✅
```

---

### Q6: Build this (common coding round question)

> Without using the `class` keyword, create a `Vehicle` → `Car` → `ElectricCar` hierarchy.
> - `Vehicle`: has `brand`, method `start()`
> - `Car` extends `Vehicle`: has `model`, method `honk()`
> - `ElectricCar` extends `Car`: has `batteryLevel`, method `charge()`

**Answer:**

```js
// Vehicle
function Vehicle(brand) {
  this.brand = brand;
}
Vehicle.prototype.start = function() {
  console.log(this.brand + " starts");
};

// Car extends Vehicle
function Car(brand, model) {
  Vehicle.call(this, brand);
  this.model = model;
}
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;
Car.prototype.honk = function() {
  console.log(this.model + " honks!");
};

// ElectricCar extends Car
function ElectricCar(brand, model, batteryLevel) {
  Car.call(this, brand, model);
  this.batteryLevel = batteryLevel;
}
ElectricCar.prototype = Object.create(Car.prototype);
ElectricCar.prototype.constructor = ElectricCar;
ElectricCar.prototype.charge = function() {
  this.batteryLevel = 100;
  console.log(this.model + " is fully charged!");
};

// Test
const tesla = new ElectricCar("Tesla", "Model 3", 60);
tesla.start();  // "Tesla starts"      ← from Vehicle
tesla.honk();   // "Model 3 honks!"    ← from Car
tesla.charge(); // "Model 3 is fully charged!" ← from ElectricCar

console.log(tesla instanceof ElectricCar); // true
console.log(tesla instanceof Car);         // true
console.log(tesla instanceof Vehicle);     // true
```

---

### Q7: `hasOwnProperty` vs `in` — difference?

```js
function Student(name) {
  this.name = name;
}
Student.prototype.study = function() {};

const s = new Student("Ravi");

// What is the difference between these?
console.log("name" in s);
console.log("study" in s);
console.log(s.hasOwnProperty("name"));
console.log(s.hasOwnProperty("study"));
```

**Answer:**
```
true   ← `in` checks entire chain
true   ← `in` checks entire chain (finds on prototype)
true   ← hasOwnProperty: name IS on s itself
false  ← hasOwnProperty: study is NOT on s itself
```

**Rule:**
- `in` → searches the entire prototype chain
- `hasOwnProperty` → only checks the object itself, ignores chain

---

### Q8: What is prototype pollution and why is it dangerous?

```js
const payload = '{"__proto__": {"isAdmin": true}}';
const parsed = JSON.parse(payload);

// Attacker's trick:
Object.assign({}, parsed);

// What happens to a completely new empty object?
const user = {};
console.log(user.isAdmin); // ???
```

**Answer:** In older/misconfigured environments, this can set `isAdmin = true` on `Object.prototype`, meaning **every object** in the program now has `isAdmin: true`. This is a real security vulnerability called **prototype pollution**.

**Protection:** Never blindly merge untrusted user input into objects. Use `Object.create(null)` for data that shouldn't inherit from Object.prototype.

---

## 16. Common Mistakes and Bugs

### Mistake 1: Putting arrays/objects on prototype (shared state bug)

```js
// ❌ Wrong
function Student(name) {
  this.name = name;
}
Student.prototype.subjects = []; // shared by ALL students!

const s1 = new Student("Ravi");
s1.subjects.push("Math");

const s2 = new Student("Priya");
console.log(s2.subjects); // ["Math"] ← bug!

// ✅ Correct
function Student(name) {
  this.name = name;
  this.subjects = []; // each student gets their own
}
```

---

### Mistake 2: Overwriting prototype after creating instances

```js
function Student(name) { this.name = name; }
const s = new Student("Ravi");

// Now you overwrite prototype:
Student.prototype = { study: function() {} };

console.log(s.__proto__ === Student.prototype); // false ❌
// `s` still points to the OLD prototype object
// New instances will get the new prototype, old ones still have the old one
```

Add methods to `prototype`, don't replace the whole thing.

---

### Mistake 3: Forgetting to call parent constructor

```js
// ❌ Wrong
function Monitor(name, rollNo, section) {
  // Forgot Student.call(this, name, rollNo)!
  this.section = section;
}
Monitor.prototype = Object.create(Student.prototype);

const m = new Monitor("Sneha", 105, "A");
console.log(m.name); // undefined ❌

// ✅ Correct
function Monitor(name, rollNo, section) {
  Student.call(this, name, rollNo); // ← always call parent!
  this.section = section;
}
```

---

### Mistake 4: Forgetting `new` — `this` becomes global!

```js
function Student(name) {
  this.name = name;
}

const s = Student("Ravi"); // ← forgot `new`!
console.log(s);        // undefined (function returns nothing)
console.log(window.name); // "Ravi" ← name was set on global object!
```

Always use `new` with constructor functions. Or add a guard:

```js
function Student(name) {
  if (!(this instanceof Student)) {
    return new Student(name); // auto-fix if called without `new`
  }
  this.name = name;
}
```

---

### Mistake 5: Arrow functions as methods (lose `this`)

```js
class Student {
  constructor(name) {
    this.name = name;
  }

  // ❌ Arrow function as method
  study = () => {
    console.log(this.name + " studies"); // `this` works here because arrow captures from constructor
  }

  // But if you put arrow on prototype:
}

Student.prototype.greet = () => {
  console.log(this.name); // ❌ `this` is NOT the instance — it's outer scope!
};
```

**Rule:** Never use arrow functions directly on `prototype`. Use regular functions for prototype methods.

---

## 17. Quick Reference Cheatsheet

```
CONCEPT              PLAIN ENGLISH                           SYNTAX
─────────────────────────────────────────────────────────────────────────────
prototype            Shared shelf of methods for a           Fn.prototype
                     constructor function

__proto__            Invisible link from object to           obj.__proto__
                     its prototype                           Object.getPrototypeOf(obj)

Prototype Chain      The lookup path: obj → proto →          automatic by JS
                     proto's proto → ... → null

new keyword          Create instance + link to prototype     new Constructor()

Object.create(x)     New empty object with x as __proto__    Object.create(x)

instanceof           Is X.prototype in obj's chain?          obj instanceof X

hasOwnProperty       Does obj have this property itself?     obj.hasOwnProperty("key")
                     (no chain lookup)

in operator          Is this property anywhere in chain?     "key" in obj

constructor          Reference from prototype back           Fn.prototype.constructor
                     to the function

Prototypal           Child's __proto__ chain includes        Object.create(Parent.prototype)
Inheritance          Parent's prototype                      Child.call(this, ...)

Mixin                Copy methods from multiple              Object.assign(Fn.prototype, mixin)
                     sources onto a prototype

class                Clean syntax for the same               class Foo extends Bar {}
                     prototype system
```

---

```
FULL CHAIN PICTURE:

instance
  └──__proto__──→ Constructor.prototype
                    └──__proto__──→ Object.prototype
                                      └──__proto__──→ null
```

---

```
INHERITANCE RECIPE (without class):

function Child(args) {
  Parent.call(this, parentArgs);  // Step 1: call parent constructor
  this.ownProp = value;           // Step 2: own properties
}
Child.prototype = Object.create(Parent.prototype); // Step 3: link chains
Child.prototype.constructor = Child;               // Step 4: fix constructor
Child.prototype.ownMethod = function() {};         // Step 5: add own methods
```

---

```
INHERITANCE RECIPE (with class):

class Child extends Parent {
  constructor(args) {
    super(parentArgs);  // must come first
    this.ownProp = value;
  }
  ownMethod() {}
}
```

---

> **Final thought:**
> Every time you use a method on a string, array, or any object — you're using the prototype system. It's not an advanced topic. It's how JavaScript works at its core. Now you understand it from the ground up.

---

*Written for JS learners preparing for placements. Covers fundamentals → inheritance → real patterns → interview scenarios.*
