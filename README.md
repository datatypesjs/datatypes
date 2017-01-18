# Datatypes

Collection of custom data-types with a consistent API.

<!-- toc -->

- [Modules](#modules)
  * [Geometry/Graphics](#geometrygraphics)
  * [Time](#time)
  * [Data structures](#data-structures)
- [Principles](#principles)
  * [No inheritance](#no-inheritance)
  * [Normalized Object Schema](#normalized-object-schema)
- [Usage](#usage)

<!-- tocstop -->


## Modules

### Geometry/Graphics

- [circle-sector] - Class for Circle sector with SVG support.
- [face] - Class for 2-faces to use as facets of polyhedrons and polygon meshes.
- [matrix] - Class for 4x4 row-major matrices for transformations in 3D.
- [point] - Class for points in 3D space.
- [vector] - Class for 3D vectors.

[circle-sector]: https://github.com/datatypesjs/circle-sector
[face]: https://github.com/datatypesjs/face
[matrix]: https://github.com/datatypesjs/matrix
[point]: https://github.com/datatypesjs/point
[vector]: https://github.com/datatypesjs/vector


### Time

- [duration] - Full ISO 8601 compatible duration class.
- [interval] - ISO 8601 based interval class.
- [moment] - ISO 8601 based time and date class.

[duration]: https://github.com/datatypesjs/duration
[interval]: https://github.com/datatypesjs/interval
[moment]: https://github.com/datatypesjs/moment


### Data structures

- [octree] - Class for octrees.

[octree]: https://github.com/datatypesjs/octree


## Principles

- One class per file
- Class name must be the same as file name

Each datatype is implemented by an ES2015 class.

```js
export default class Color {
  …
}
```
---

Each class must be instantiated with the `new` keyword.

```js
const color = new Color()
```
---

The constructor always expects exactly one object as the argument
or no argument at all.

Example:
```js
const color = new Color({red: 250, green: 35, blue: 129})
```

All object keys must be assigned to the object instance
(e.g. through `Object.assign(this, optionsObject)`).
This allows the user of the class to easily assign custom object properties
in addition to the designated ones.

To provide other constructor methods use the `from<datatype>` pattern.

Where `<datatype>` must contain the native JavaScript data type
(e.g. `String`, `Number`, `Array`, `Object`, `Date`, …)
if it's not a custom datatype.

Examples:

```js
const color = Color.fromHexString('#FA2381')
```
```js
const color = Color.fromRgbArray([250, 35, 129])
```
```js
const color = Color.from(car)
```
---

`.from()` is a special constructor which accesses the relevant Data
through duck typing. So as long as an object has the properties
`red`, `green` and `blue` it can be be used with the `.from()` constructor.
(This is derived from the native `Array.from`)


Each public computed property must have a setter and a getter.
Always using getters (e.g. `hsl` instead of `toHsl`)
avoids premature optimization and a ensures a consistent API.
It also hides the implementation detail
if a property is static or is dynamically created.

```js
export default class Color {
  …
  get red () { return this._red }
  set red (value) {
    this._red = value
    // To other related computation
    return this
  }
  …
}
```
---

Additionally to the setter there must be a `.set<property>` method
to enable chaining.

Example:
```js
const color = new Color()
  .setRed(250)
  .setGreen(35)
  .setBlue(129)
```


Evaluate getters lazily.
(E.g. don't recalculate the internal HSL value on setting red
but rather when the new HSL value is requested)
Furthermore a `.computeHsl()` method can be provided to let
the user decide when to recalculate values.

```js
// When computation time is abundantly available
const hslObject = new Color()
  .setRed(250)
  .computeHsl()

// Later, when computation time is limited
doSomething(hslObject.hsl)
```


---

All methods must accept one argument at most.
For more arguments an object must be used.

- Ensures backwards compatible extensibility
- Increases readability

Example:

```js
const color = new Color({red: 250, green: 35, blue: 129})
```

instead of

```js
const color = new Color(250, 35, 129)
```
---

Mandatory instance methods:

- `toJSON` - Must return a plain JavaScript object
  which can be JSON stringified.
  For example `color.toJSON()` could yield `{red: 250, green: 35, blue: 129}`
  `.toJSON()` is an alias for `.object`

- `toString` - Must return a string representation of the object.
  For example `color.toString()` could yield `'rgb(250, 35, 129)'`
  `.toString()` is an alias for `.string`

---

Mandatory class methods:

- `fromObject` - Create instance from plain JavaScript object.
- `schema` - Returns JSON schema for the `fromObject` argument.
- `validate` - Validates an object against the JSON schema.

---

Private methods must start with an underscore `_`

---

Datatypes classes shall be preferred over custom implementations.


---
Attention: `color instanceof Color` is anti-pattern in most cases!

Instead of `if (color instanceof Color) return color.red`
use `if (color.hasOwnProperty('red')) return color.red`

---

Use "synchronous usage of asynchronous code" pattern for asynchronous code.



### No inheritance

```js
class Person extends Mammal extends Animal extends Organism extends …
```

Although correct, it's not really helpful,
consumes a lot of memory and method lookup hurts performance.

Better:
Explicitly set a class as prototype if methods and properties of
a possible base class are needed.


### Normalized Object Schema

The root element must be an object:

```yaml
users:
  - id: 1
  - id: 2
```

instead of

```yaml
- id: 1
- id: 2
```

Avoid singular terms.

Instead of

```yaml
email: john@work.com
```

use

```yaml
emails:
  - …
  - …
```

Objects may not be misused as arrays

Instead of similar keys

```yaml
privateEmail: john@home.com
workEmail: john@work.com
```

use arrays

```yaml
emails:
  - value: john@home.com
    type: private
  - value: john@work.com
    type: work
```

Avoid primitives in arrays:

Instead of

```yaml
emails:
  - john@home.com
  - john@work.com
```

use

```yaml
emails:
  - value: john@home.com
    type: private
  - value: john@work.com
    type: work
```


Make sure the primitive value can not be further specialized:

```js
{
  email: 'john.doe@example.com'
}

```

should be

```js
{
  email: new Email('john.doe@example.com')
}
```

This, on the other hand, is OK:

```js
{
  id: 2342
}
```


## Usage

Don't use primitives when you need to perform computation on that data.
E.g. You can use `'2003-09-23'` to store a birthday, but you should use
`new Day('2003-09-23')` if you need to calculate the age from it.
