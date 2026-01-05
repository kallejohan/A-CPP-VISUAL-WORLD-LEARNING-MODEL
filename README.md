This work is licensed under CC BY 4.0 and is free to use in academic,
educational, and commercial contexts with attribution.

A TWO-LAYER WORLD FOR MODERN C++
A MODEL DESCRIBING VALUES, OBJECTS, REFERENCES, POINTERS, AND LIFETIMES
(C++17 and later)

This text describes a mental model for modern C++.

It is not the C++ standard.
It is not an implementation model.

It is an educational model you can use to see how values, objects, references, pointers,
and lifetimes behave in C++17 and later.

Like all metaphors, this model has limits and should be used alongside
the C++ standard and other authoritative references.

--------------------------------------------------------------------------------
THE SHAPE OF THE WORLD
--------------------------------------------------------------------------------

The world has two layers:

- The Air
- The Ground

The Ground is a wide landscape.

To the left, there is a territory of Bedrock.
To the right, there is open land where objects may appear during execution.

Nothing runs this world.
There is no executor or agent.

Things simply appear, connect, and disappear according to the rules
of the language.

--------------------------------------------------------------------------------
1) THE AIR — WHERE PURE VALUES APPEAR
--------------------------------------------------------------------------------

Above the ground lies the Air.

Most of the time, it is empty.

Whenever an expression produces a pure value,
a bubble appears in the air.

A bubble:

- carries a value
- has no identity
- cannot be referred to later
- disappears when no longer needed

These bubbles correspond to prvalues.

Examples that create air bubbles:

42

i + 1

f()

(if f returns by value)

A bubble is not an object.
It is only a value.

Example:

42;

A bubble containing 42 appears momentaneously in the air and immediately pops.
Nothing ever reaches the ground.

--------------------------------------------------------------------------------
2) THE GROUND — WHERE IDENTITY EXISTS
--------------------------------------------------------------------------------

Below the air lies the Ground.

The ground is the only place where identity exists.

Anything on the ground:

- can be referred to again
- has storage
- has a lifetime

The ground has regions.

--------------------------------------------------------------------------------
3) BEDROCK — FIXED PROGRAM ENTITIES
--------------------------------------------------------------------------------

On the left side of the ground lies Bedrock.

Bedrock represents things that exist because the program is written
that way, not because an expression evaluated.

Bedrock entities:

- exist for the entire program
- are not created or destroyed by evaluation
- never change shape

Examples of bedrock entities:

int f();

void g();

constexpr int N = 10;

Functions and compile-time constants belong to bedrock.

They are part of the landscape.
They do not move or change.

--------------------------------------------------------------------------------
4) OBJECTS — FORMED IN THE LANDSCAPE
--------------------------------------------------------------------------------

To the right side of the ground, objects may appear during execution.

Objects are made of clay.

--------------------------------------------------------------------------------
4a) SOFT CLAY (GREEN) — MUTABLE OBJECTS
--------------------------------------------------------------------------------

Soft-clay objects:

- appear during execution
- have identity and lifetime
- can be reshaped
- remain the same object while changing value

Example:

int a = 1;

a = 4;

An air bubble with value 1 appears in the air.

A green soft-clay object is formed when the bubble pops over it.

Later, a new air bubble pops and reshapes the green clay object into 4.

It is still the same object.


--------------------------------------------------------------------------------
4b) HARD CLAY (GREY) — CONST OBJECTS
--------------------------------------------------------------------------------

Hard-clay objects:

- appear during execution

- have identity and lifetime

- are shaped once

- cannot be reshaped afterward

Example:

const int b = 3;

An air bubble with value 3 appears.
A hard-clay object is formed when the air bubble pops over b.
Its shape is fixed forever.

--------------------------------------------------------------------------------
5) EXPRESSIONS THAT DESIGNATE THE GROUND — VIEWS
--------------------------------------------------------------------------------

Some expressions do not produce values in the air.

Instead, they designate something that already exists on the ground.

When this happens, the model shows a momentaneous view (a hand)
touching that entity.

These hands touching the entities:

- are not objects
- do not persist
- exist only during expression evaluation

--------------------------------------------------------------------------------
6) VALUE CATEGORIES AS HANDS
--------------------------------------------------------------------------------

To make these momentaneous views visible, I represent the view as a hand.

A hand:

- is not part of the world
- is not an object
- is not a reference
- does not change the state of the object it touches
- exists only during the expression
- shows how the expression views what it designates

--------------------------------------------------------------------------------
7) HAND COLORS — HOW AN EXPRESSION VIEWS AN OBJECT
--------------------------------------------------------------------------------

Green hand touching object:
- non-const lvalue
- stable view
- reshaping of object allowed

Grey hand:
- const lvalue
- stable view
- read-only

Red hand:
- non-const xvalue
- expiring view
- reshaping allowed

Grey/red hand:
- const xvalue
- expiring view
- read-only

Value category belongs to the expression, not to the object.

--------------------------------------------------------------------------------
8) PRVALUES DO NOT PRODUCE HANDS
--------------------------------------------------------------------------------

A prvalue produces no hand.

It produces only an air bubble.

Hands appear only when identity already exists on the ground.

--------------------------------------------------------------------------------
9) MATERIALIZATION — WHEN AIR MUST BECOME GROUND
--------------------------------------------------------------------------------

Some operations cannot work with pure values alone.

They require an object with identity.

Examples:

- binding a reference to an object
- accessing a member
- passing by reference
- initializing a class object

Temporary materialization:

1) An air bubble representing the value descends toward the ground
2) A ground bubble forms
3) Inside that ground bubble, a temporary green object arises
4) The temporary object gets a momentaneous red hand indicating that object is expiring. 

This object:

- is a real object on the ground
- has identity and storage
- exists only because the prvalue was forced to materialize

The object is kept alive by the surrounding ground bubble.
If object is not used the bubble pops, the object inside is destroyed.

If an object is initialized by a pure value, the bubble descends to ground where the object shall exist. The bubble pops and a green or grey object is formed on ground. 

--------------------------------------------------------------------------------
10) REFERENCES — WIRES SHOT FROM PORTS
--------------------------------------------------------------------------------

A reference binding is represented as a wire that fastens to a hand.

Wires are not part of the world.
They are visual aids.

A wire is always shot from a port.

Ports exist only at reference declarations.

Examples of ports:

const int& r =   // r is a port

int&& rr = ...   // rr is a port

Ports can exist in bedrock when they represent reference bindings
inside functions.

Ports can also exist in objects representing reference members.
However, ports are not objects and do not belong to the world.

--------------------------------------------------------------------------------
11) WIRE, PORT COLORS AND BINDING RULES
--------------------------------------------------------------------------------

Wire color matches the port it comes from.

When a wire binds to a hand, the hand + wire represent a reference:

- Green wire + hand   -> T&
- Grey wire + hand    -> const T&
- Red wire + hand     -> T&&
- Grey/red wire + hand-> const T&&

Binding rules:

- Green wire binds only to green hands on soft-clay objects
- Grey wire binds to green, grey, red, and grey/red hands
- Red wire binds only to red hands
- Grey/red wire binds to red or grey/red hands


When a wire binds to a hand, the hand takes the color of the wire.
The hand and wire persist until reference goes out of scope.

--------------------------------------------------------------------------------
12) LIFETIME EXTENSION
--------------------------------------------------------------------------------

When a wire binds to a temporary green object with momentaneous red hand inside a ground bubble:

- the wire persists
- the hand holding it persists
- the ground bubble remains
- the temporary object remains alive

When the wire disappears:

- the hand disappears
- the ground bubble pops
- the temporary object is destroyed

The wire that binds to the temporary (momentaneous red or hand on green temporary) gets a special seal marker where it goes through the bubble. This seal marker at entering the bubble shows that this reference keeps the bubble and object alive. 

New wires and hands can bind to object inside bubble but these references without seal markers can disappear without destroying the temporary object in ground bubble. 

--------------------------------------------------------------------------------
PART II — DETAILED WORLD EXAMPLES
--------------------------------------------------------------------------------

Example 1 — Returning by value, unused

int f() { return 42; }

f();

Calling f() produces an air bubble.

No identity is required.

The bubble pops in the air.

Nothing reaches the ground.

--------------------------------------------------------------------------------
Example 2 — Returning by value, initializing an object

int x = f();

f() produces an air bubble.

Initialization requires identity.

The value bubble pops into a new soft-clay object x.

--------------------------------------------------------------------------------
Example 3 — Binding const lvalue reference

const int& r = f();

f() produces an air bubble.

A port exists at r.

Identity is required.

The bubble descends.

A ground bubble forms.

A temporary int object arises inside it.

A grey wire is shot and binds with a grey hand.

The ground bubble and object persist as long as the reference exists.

--------------------------------------------------------------------------------
Example 4 — Binding rvalue reference

int&& r = f();

f() produces an air bubble.

A port exists at r.

The bubble descends.

A ground bubble forms.

A temporary int object arises.

A red wire and hand bind.

Lifetime is extended.

--------------------------------------------------------------------------------
Example 5 — Named rvalue reference and std::move

int&& r = f();

std::move(r);

f() produces an air bubble.

The bubble descends.

A ground bubble forms.

A temporary object arises.

A red wire binds to it.



Evaluating std::move(r):

r designates the ground object.

A red hand appears on the object.

The object is viewed as expiring.

No new object is created.

The red hand disappears, but the ground bubble, object,
and wire persist.

--------------------------------------------------------------------------------
Example 6 — Using a reference as an expression

int&& r = f();

r;

The red wire and hand hold the object inside the ground bubble.

Evaluating expression r produces a green hand on the object.

The object is viewed as a stable lvalue.

The green hand disappears after the expression.

The red wire, ground bubble, and object persist until r goes out of scope.

--------------------------------------------------------------------------------
Example 7 — Built-in assignment

int a;

a = f();

f() produces an air bubble.

Assignment consumes the value.

The shape of a changes.

The bubble pops.

No ground bubble forms.

--------------------------------------------------------------------------------
Example 8 — Arithmetic expressions

int x = 1;

int y = x + 2;

x produces a green hand.

2 produces an air bubble.

The addition produces an air bubble.

The resulting value initializes and forms y on the ground.

--------------------------------------------------------------------------------
Example 9 — Returning by reference

int x = 7;

int& f() { return x; }

int& r = f();

f() designates x.

A green hand appears on x.

A green wire binds directly to the green hand on x.

--------------------------------------------------------------------------------
PART III — POINTERS
--------------------------------------------------------------------------------

--------------------------------------------------------------------------------
13) POINTERS — OBJECTS THAT EMIT ARROWS
--------------------------------------------------------------------------------

Pointers are objects on the ground.

Unlike references, a pointer:

- has identity
- has storage
- contains a value (an address or null)
- may be reshaped unless const-qualified

A pointer may be initialized from an air bubble containing an address,
but once initialized, the pointer itself exists on the ground.

In this world:

- The pointer is a green or grey clay object
- From the pointer, an arrow extends
- The arrow indicates which object is being pointed to

The arrow is not a wire.
It does not bind.
It does not affect lifetime.
The arrow is always green or grey. 

--------------------------------------------------------------------------------
14) POINTER INITIALIZATION — ADDRESS BECOMES AN OBJECT
--------------------------------------------------------------------------------

int x = 10;

int* p = &x;

- x is a soft-clay object on the ground
- The expression &x produces an air bubble containing x’s address
- Initialization of p requires identity
- The air bubble pops and forms a pointer object p on the ground
- From p, a green arrow points to x

The arrow represents direction only.

--------------------------------------------------------------------------------
15) POINTER REASSIGNMENT — MOVING THE ARROW
--------------------------------------------------------------------------------

int y = 20;

p = &y;

- &y produces a new air bubble containing y’s address
- Assignment reshapes the pointer object p
- The arrow moves from x to y
- Neither x nor y is affected

--------------------------------------------------------------------------------
16) CONSTNESS — OBJECT VS ARROW
--------------------------------------------------------------------------------

int* const p2 = &x;

- p2 is a grey object on the ground
- the arrow is green
- the pointer cannot be reshaped
- the pointed object may be reshaped

const int* p3 = &x;

- p3 is a green object
- the arrow is grey
- the pointer may be reshaped
- the pointed object is read-only through the arrow

const int* const p4 = &x;

- p4 is a grey object
- the arrow is grey
- neither pointer nor pointed object may be reshaped

--------------------------------------------------------------------------------
17) POINTERS AND LIFETIME
--------------------------------------------------------------------------------

Pointers do not create ground bubbles.
Pointers do not extend lifetime.

If the object an arrow points to disappears:

- the pointer object remains
- the arrow remains
- the arrow dangles

The world allows this.
The model makes it visible.

To be added:

Non void pointers always propagate in ground layer. 

A void pointer is a dashed line going under the ground layer and points to a location on ground but never touches anything. It can not be dereferenced. 

A pointer with null value goes up through air layer and into empty space above air layer. It doesnt point to anything. 

Add symbol for arrays, its like a roof symbol (horizontal bracket)
above objects (elements of array)

Non-void pointer can point to roof (array) or to element in array.

Reference can bind to roof or to element. 




--------------------------------------------------------------------------------
CLOSING
--------------------------------------------------------------------------------

In this world:

- The air holds values
- The ground holds identity
- Bedrock fixes structure
- Objects arise from clay
- Hands show expression views
- Wires show reference binding
- Arrows show pointer direction
- Ground bubbles make materialization visible
- Lifetime extension becomes explicit

--------------------------------------------------------------------------------
END
--------------------------------------------------------------------------------

Feedback, questions, and alternative viewpoints are welcome via GitHub Discussions.

If you reuse or adapt this work, please credit:
kallejohan — "A Two-Layer World for Modern C++"










