# Datatypes

Collection of custom data-types with a consistent API.


## Modules

- [circle-sector] - Class for Circle sector with SVG support.
- [duration] - Full ISO 8601 compatible duration class.
- [face] - Class for 2-faces to use as facets of polyhedrons and polygon meshes.
- [interval] - ISO 8601 based interval class.
- [matrix] - Class for 4x4 row-major matrices for transformations in 3D.
- [moment] - ISO 8601 based time and date class.
- [octree] - Class for octrees.
- [point] - Class for points in 3D space.
- [vector] - Class for 3D vectors.

[circle-sector]: https://github.com/datatypesjs/circle-sector
[duration]: https://github.com/datatypesjs/duration 
[face]: https://github.com/datatypesjs/face 
[interval]: https://github.com/datatypesjs/interval 
[matrix]: https://github.com/datatypesjs/matrix 
[moment]: https://github.com/datatypesjs/moment 
[octree]: https://github.com/datatypesjs/octree 
[point]: https://github.com/datatypesjs/point 
[vector]: https://github.com/datatypesjs/vector


## Principles

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

The constructor always expects exactly one object as the argument or no argument at all.

Example:
```js
const color = new Color({red: 250, green: 35, blue: 129})
```

To provide other constructor methods use the `from<datatype>` pattern.

Examples:

```js
const color = Color.fromHexString('#FA2381')
```
```js
const color = Color.fromRgbArray([250, 35, 129])
```
---

Each public property must have a setter and a getter.

```js
export default class Color {
  …
  set red () { … }
  get red () { return this._red }
  …
}
```
---

Additionally to the setter there must be a `.set<property>` method to enable chaining.

Example:
```js
const color = new Color()
  .setRed(250)
  .setGreen(35)
  .setBlue(129)
```
---

All methods must accept one argument at most. For more arguments an object must be used.

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

- `toJSON` - Must return a plain JavaScript object which can be JSON stringified.
  For example `color.toJSON()` could yield `{red: 250, green: 35, blue: 129}`
- `toString` - Must return a string representation of the object.
  For example `color.toString()` could yield `'rgb(250, 35, 129)'`

---

Mandatory class methods:

- `fromObject` - Create instance from plain JavaScript object.
- `schema` - Returns JSON schema for the `fromObject` argument.
- `validate` - Validates an object against the JSON schema.

---

Private methods must start with an underscore `_`
---

Datatypes classes shall be preferred over custom implementations.
