# props-model

A javascript package providing a model for properties, including change events and derived properties.

[![JavaScript Style Guide](https://cdn.rawgit.com/standard/standard/master/badge.svg)](https://github.com/standard/standard)

[![npm](https://img.shields.io/npm/v/props-model.svg)](https://libraries.io/github/mearns/props-model)
[![Stable Build](https://travis-ci.org/mearns/props-model.svg?branch=versions%2Fstable)](https://travis-ci.org/mearns/props-model)
[![node](https://img.shields.io/node/v/props-model.svg)](https://www.npmjs.com/package/props-model)
[![NpmLicense](https://img.shields.io/npm/l/props-model.svg)](https://spdx.org/licenses/MIT)

[![npm bundle size (minified)](https://img.shields.io/bundlephobia/min/props-model.svg)](https://www.npmjs.com/package/props-model)
[![Libraries.io for GitHub](https://img.shields.io/librariesio/github/mearns/props-model.svg)](https://libraries.io/github/mearns/props-model)
[![npm](https://img.shields.io/npm/dy/props-model.svg)](https://www.npmjs.com/package/props-model)
[![GitHub issues](https://img.shields.io/github/issues-raw/mearns/props-model.svg)](https://github.com/mearns/props-model/issues?q=is%3Aissue+is%3Aopen)
[![GitHub pull requests](https://img.shields.io/github/issues-pr-raw/mearns/props-model.svg)](https://github.com/mearns/props-model/pulls?q=is%3Apr+is%3Aopen)
[![GitHub last commit](https://img.shields.io/github/last-commit/mearns/props-model.svg)](https://github.com/mearns/props-model/commits/)


## Overview

This package provides the `PropertyModel` class which can serve as the M in an MVC pattern, or more generally
to track and manage a set of named properties. In addition to get and modify access for the properties, the
model also provides synchronous change event firing and listener creation for the managed properties. It
also allows you to define _derived properties_ whose values are automatically calculated anytime a property
it depends on changes.

## Install

With npm:

```console
npm install --save props-model
```

## Demo Usage

```javascript
import PropsModel from 'props-model' // or `const { PropsModel } = require('props-model')`
import EventEmitter from 'events'

class MutableRectangle {
  constructor (initialLength, initialWidth) {
    // Define the properties of this object.
    const propModel = new PropsModel(new EventEmitter())
      // These are "primary" properties, they are not calculated from other properties.
      .defineProp('length', initialLength, isValidDimension)
      .defineProp('width', initialWidth, isValidDimension)
      // These are "derived" properties, they are automatically updated when any of
      // the properties they depend on change.
      .defineDerivedProp('area', ['length', 'width'], (length, width) => length * width)
      .defineDerivedProp('perimeter', ['length', 'width'], (length, width) => (2 * length) + (2 * width))
      .defineDerivedProp('aspectRatio', ['length', 'width'], (length, width) => length / width)

    // We can use the propModel as an implementation detail, and expose classical getters and setters
    // for our props. This method on the propModel does that for us.
    propModel.installAccessors(this, {
      // getter and setters for our primary properties.
      length: 'readwrite',
      width: 'readwrite',
      // It's not usually a good idea to allow derived properties to be set directly,
      // it breaks coherency. Only getters will be provided for these properties.
      area: 'readonly',
      perimeter: 'readonly',
      aspectRatio: 'readonly'
    })

    // Our propModel also provides a convenient JSON representation of our properties,
    // which we will adopt as our own.
    this.toJSON = () => propModel.toJSON()

    // Note that we don't need to keep propModel around as an instance property, it's
    // attached to the accessors' closures as needed; it's _generally_ good practice to
    // set up all uses of the propModel in the constructor, and *not* use it directly
    // after that.
  }
}

// Optional property value validators can be provided for primary properties.
function isValidDimension (dim) {
  if (typeof dim !== 'number') {
    throw new Error('Invalid dimension, must be a number')
  }
  if (dim < 0) {
    throw new Error('Invalid dimension, must be non-negative')
  }
}

// Let's exercise our new class a bit:
function main () {
  const rect = new MutableRectangle(10, 20)

  // JSON.stringify uses the toJSON() method attached ot the object.
  console.log(JSON.stringify(rect)) // {"length":10,"width":20,"area":200,"perimeter":60,"aspectRatio":0.5}

  // When we set a property...
  rect.setLength(15)

  // ...that property is updated ...
  console.log(rect.getLength()) // 15

  // ... and so are derived properties ...
  console.log(rect.getArea()) // 300
  console.log(JSON.stringify(rect)) // {"length":15,"width":20,"area":300,"perimeter":70,"aspectRatio":0.75}
}

main()

```
