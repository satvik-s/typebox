<div align='center'>

<h1>TypeBox</h1>

<p>JSON Schema Type Builder with Static Type Resolution for TypeScript</p>
	
<img src="https://github.com/sinclairzx81/typebox/blob/master/typebox.png?raw=true" />

<br />
<br />

[![npm version](https://badge.fury.io/js/%40sinclair%2Ftypebox.svg)](https://badge.fury.io/js/%40sinclair%2Ftypebox)
[![Downloads](https://img.shields.io/npm/dm/%40sinclair%2Ftypebox.svg)](https://www.npmjs.com/package/%40sinclair%2Ftypebox)
[![GitHub CI](https://github.com/sinclairzx81/typebox/workflows/GitHub%20CI/badge.svg)](https://github.com/sinclairzx81/typebox/actions)

</div>

<a name="Install"></a>

## Install

#### Node

```bash
$ npm install @sinclair/typebox --save
```

#### Deno

```typescript
import { Static, Type } from 'https://deno.land/x/typebox/src/typebox.ts'
```

## Example

```typescript
import { Static, Type } from '@sinclair/typebox'

const T = Type.String()     // const T = { type: 'string' }

type T = Static<typeof T>   // type T = string
```

<a name="Overview"></a>

## Overview

TypeBox is a type builder library that creates in-memory JSON Schema objects that can be statically inferred as TypeScript types. The schemas produced by this library are designed to match the static type checking rules of the TypeScript compiler. TypeBox enables one to create unified types that can be statically checked by TypeScript and runtime asserted using standard JSON Schema validation.

TypeBox can be used as a simple tool to build up complex schemas or integrated into RPC or REST services to help validate data received over the wire. It offers a type system aligned to the capabilities of TypeScript and can be used equally well in JavaScript environments.

License MIT

## Contents
- [Install](#Install)
- [Overview](#Overview)
- [Usage](#Usage)
- [Types](#Types)
- [Modifiers](#Modifiers)
- [Options](#Options)
- [Extended Types](#Extended-Types)
- [Reference Types](#Reference-Types)
- [Recursive Types](#Recursive-Types)
- [Generic Types](#Generic-Types)
- [Unsafe Types](#Unsafe-Types)
- [Conditional Types](#Conditional-Types)
- [Values](#Values)
- [Guards](#Guards)
- [Strict](#Strict)
- [Validation](#Validation)
- [Compiler](#Compiler)
- [Benchmark](#Benchmark)
- [Contribute](#Contribute)

<a name="Example"></a>

## Usage

The following demonstrates TypeBox's general usage.

```typescript

import { Static, Type } from '@sinclair/typebox'

//--------------------------------------------------------------------------------------------
//
// Let's say you have the following type ...
//
//--------------------------------------------------------------------------------------------

type T = {
  id: string,
  name: string,
  timestamp: number
}

//--------------------------------------------------------------------------------------------
//
// ... you can express this type in the following way.
//
//--------------------------------------------------------------------------------------------

const T = Type.Object({                              // const T = {
  id: Type.String(),                                 //   type: 'object',
  name: Type.String(),                               //   properties: { 
  timestamp: Type.Integer()                          //     id: { 
})                                                   //       type: 'string' 
                                                     //     },
                                                     //     name: { 
                                                     //       type: 'string' 
                                                     //     },
                                                     //     timestamp: { 
                                                     //       type: 'integer' 
                                                     //     }
                                                     //   }, 
                                                     //   required: [
                                                     //     'id',
                                                     //     'name',
                                                     //     'timestamp'
                                                     //   ]
                                                     // } 

//--------------------------------------------------------------------------------------------
//
// ... then infer back to the original static type this way.
//
//--------------------------------------------------------------------------------------------

type T = Static<typeof T>                            // type T = {
                                                     //   id: string,
                                                     //   name: string,
                                                     //   timestamp: number
                                                     // }

//--------------------------------------------------------------------------------------------
//
// ... then use the type both as JSON schema and as a TypeScript type.
//
//--------------------------------------------------------------------------------------------

function receive(value: T) {                         // ... as a Type

  if(JSON.validate(T, value)) {                      // ... as a Schema
  
    // ok...
  }
}
```

<a name="Types"></a>

## Types

The following table outlines the TypeBox mappings between TypeScript and JSON schema.

```typescript
┌────────────────────────────────┬─────────────────────────────┬────────────────────────────────┐
│ TypeBox                        │ TypeScript                  │ JSON Schema                    │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Any()           │ type T = any                │ const T = { }                  │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Unknown()       │ type T = unknown            │ const T = { }                  │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.String()        │ type T = string             │ const T = {                    │
│                                │                             │   type: 'string'               │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Number()        │ type T = number             │ const T = {                    │
│                                │                             │   type: 'number'               │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Integer()       │ type T = number             │ const T = {                    │
│                                │                             │   type: 'integer'              │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Boolean()       │ type T = boolean            │ const T = {                    │
│                                │                             │   type: 'boolean'              │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Null()          │ type T = null               │ const T = {                    │
│                                │                             │    type: 'null'                │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.RegEx(/foo/)    │ type T = string             │ const T = {                    │
│                                │                             │    type: 'string',             │
│                                │                             │    pattern: 'foo'              │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Literal(42)     │ type T = 42                 │ const T = {                    │
│                                │                             │    const: 42                   │
│                                │                             │    type: 'number'              │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Array(          │ type T = number[]           │ const T = {                    │
│   Type.Number()                │                             │   type: 'array',               │
│ )                              │                             │   items: {                     │
│                                │                             │     type: 'number'             │
│                                │                             │   }                            │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Object({        │ type T = {                  │ const T = {                    │
│   x: Type.Number(),            │   x: number,                │   type: 'object',              │
│   y: Type.Number()             │   y: number                 │   properties: {                │
│ })                             │ }                           │      x: {                      │
│                                │                             │        type: 'number'          │
│                                │                             │      },                        │
│                                │                             │      y: {                      │
│                                │                             │        type: 'number'          │
│                                │                             │      }                         │
│                                │                             │   },                           │
│                                │                             │   required: ['x', 'y']         │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Tuple([         │ type T = [number, number]   │ const T = {                    │
│   Type.Number(),               │                             │   type: 'array',               │
│   Type.Number()                │                             │   items: [{                    │
│ ])                             │                             │      type: 'number'            │
│                                │                             │    }, {                        │
│                                │                             │      type: 'number'            │
│                                │                             │    }],                         │
│                                │                             │    additionalItems: false,     │
│                                │                             │    minItems: 2,                │
│                                │                             │    maxItems: 2                 │
│                                │                             │ }                              │
│                                │                             │                                │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ enum Foo {                     │ enum Foo {                  │ const T = {                    │
│   A,                           │   A,                        │   anyOf: [{                    │
│   B                            │   B                         │     type: 'number',            │
│ }                              │ }                           │     const: 0                   │
│                                │                             │   }, {                         │
│ const T = Type.Enum(Foo)       │ type T = Foo                │     type: 'number',            │
│                                │                             │     const: 1                   │
│                                │                             │   }]                           │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.KeyOf(          │ type T = keyof {            │ const T = {                    │
│   Type.Object({                │   x: number,                │   anyOf: [{                    │
│     x: Type.Number(),          │   y: number                 │     type: 'string',            │
│     y: Type.Number()           │ }                           │     const: 'x'                 │
│   })                           │                             │   }, {                         │
│ )                              │                             │     type: 'string',            │
│                                │                             │     const: 'y'                 │
│                                │                             │   }]                           │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Union([         │ type T = string | number    │ const T = {                    │
│   Type.String(),               │                             │   anyOf: [{                    │
│   Type.Number()                │                             │      type: 'string'            │
│ ])                             │                             │   }, {                         │
│                                │                             │      type: 'number'            │
│                                │                             │   }]                           │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Intersect([     │ type T = {                  │ const T = {                    │
│   Type.Object({                │   x: number                 │   type: 'object',              │
│     x: Type.Number()           │ } & {                       │   properties: {                │
│   }),                          │   y: number                 │     x: {                       │
│   Type.Object({                │ }                           │       type: 'number'           │
│     y: Type.Number()           │                             │     },                         │
│   })                           │                             │     y: {                       │
│ ])                             │                             │       type: 'number'           │
│                                │                             │     }                          │
│                                │                             │   },                           │
│                                │                             │   required: ['x', 'y']         │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Record(         │ type T = Record<            │ const T = {                    │
│   Type.String(),               │   string,                   │   type: 'object',              │
│   Type.Number()                │   number,                   │   patternProperties: {         │
│ )                              │ >                           │     '^.*$': {                  │
│                                │                             │       type: 'number'           │
│                                │                             │     }                          │
│                                │                             │   }                            │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Partial(        │ type T = Partial<{          │ const T = {                    │
│   Type.Object({                │   x: number,                │   type: 'object',              │
│     x: Type.Number(),          │   y: number                 │   properties: {                │
│     y: Type.Number()           | }>                          │     x: {                       │
│   })                           │                             │       type: 'number'           │
│ )                              │                             │     },                         │
│                                │                             │     y: {                       │
│                                │                             │       type: 'number'           │
│                                │                             │     }                          │
│                                │                             │   }                            │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Required(       │ type T = Required<{         │ const T = {                    │
│   Type.Object({                │   x?: number,               │   type: 'object',              │
│     x: Type.Optional(          │   y?: number                │   properties: {                │
│       Type.Number()            | }>                          │     x: {                       │
│     ),                         │                             │       type: 'number'           │
│     y: Type.Optional(          │                             │     },                         │
│       Type.Number()            │                             │     y: {                       │
│     )                          │                             │       type: 'number'           │
│   })                           │                             │     }                          │
│ )                              │                             │   },                           │
│                                │                             │   required: ['x', 'y']         │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Pick(           │ type T = Pick<{             │ const T = {                    │
│   Type.Object({                │   x: number,                │   type: 'object',              │
│     x: Type.Number(),          │   y: number                 │   properties: {                │
│     y: Type.Number()           | }, 'x'>                     │     x: {                       │
│   }), ['x']                    │                             │       type: 'number'           │
│ )                              │                             │     }                          │
│                                │                             │   },                           │
│                                │                             │   required: ['x']              │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Omit(           │ type T = Omit<{             │ const T = {                    │
│   Type.Object({                │   x: number,                │   type: 'object',              │
│     x: Type.Number(),          │   y: number                 │   properties: {                │
│     y: Type.Number()           | }, 'x'>                     │     y: {                       │
│   }), ['x']                    │                             │       type: 'number'           │
│ )                              │                             │     }                          │
│                                │                             │   },                           │
│                                │                             │   required: ['y']              │
│                                │                             │ }                              │
│                                │                             │                                │
└────────────────────────────────┴─────────────────────────────┴────────────────────────────────┘
```
<a name="Modifiers"></a>

## Modifiers

TypeBox provides modifiers that can be applied to an objects properties. This allows for `optional` and `readonly` to be applied to that property. The following table illustates how they map between TypeScript and JSON Schema.

```typescript
┌────────────────────────────────┬─────────────────────────────┬────────────────────────────────┐
│ TypeBox                        │ TypeScript                  │ JSON Schema                    │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Object({        │ type T = {                  │ const T = {                    │
│   name: Type.Optional(         │   name?: string             │   type: 'object',              │
│     Type.String()              │ }                           │   properties: {                │
│   )                            │                             │      name: {                   │
│ })  	                         │                             │        type: 'string'          │
│                                │                             │      }                         │
│                                │                             │   }                            │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Object({        │ type T = {                  │ const T = {                    │
│   name: Type.Readonly(         │   readonly name: string     │   type: 'object',              │
│     Type.String()              │ }                           │   properties: {                │
│   )                            │                             │     name: {                    │
│ })  	                         │                             │       type: 'string'           │
│                                │                             │     }                          │
│                                │                             │   },                           │
│                                │                             │   required: ['name']           │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Object({        │ type T = {                  │ const T = {                    │
│   name: Type.ReadonlyOptional( │   readonly name?: string    │   type: 'object',              │
│     Type.String()              │ }                           │   properties: {                │
│   )                            │                             │     name: {                    │
│ })  	                         │                             │       type: 'string'           │
│                                │                             │     }                          │
│                                │                             │   }                            │
│                                │                             │ }                              │
│                                │                             │                                │
└────────────────────────────────┴─────────────────────────────┴────────────────────────────────┘
```

<a name="Options"></a>

## Options

You can pass additional JSON schema options on the last argument of any given type. The following are some examples.

```typescript
// string must be an email
const T = Type.String({ format: 'email' })

// number must be a multiple of 2
const T = Type.Number({ multipleOf: 2 })

// array must have at least 5 integer values
const T = Type.Array(Type.Integer(), { minItems: 5 })
```

<a name="Extended-Types"></a>

## Extended Types

In addition to JSON schema types, TypeBox provides several extended types that allow for `function` and `constructor` types to be composed. These additional types are not valid JSON Schema and will not validate using typical JSON Schema validation. However, these types can be used to frame JSON schema and describe callable interfaces that may receive JSON validated data. These types are as follows.

```typescript
┌────────────────────────────────┬─────────────────────────────┬────────────────────────────────┐
│ TypeBox                        │ TypeScript                  │ Extended Schema                │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Constructor([   │ type T = new (              │ const T = {                    │
│   Type.String(),               │  arg0: string,              │   type: 'constructor'          │
│   Type.Number()                │  arg1: number               │   parameters: [{               │
│ ], Type.Boolean())             │ ) => boolean                │     type: 'string'             │
│                                │                             │   }, {                         │
│                                │                             │     type: 'number'             │
│                                │                             │   }],                          │
│                                │                             │   return: {                    │
│                                │                             │     type: 'boolean'            │
│                                │                             │   }                            │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Function([      │ type T = (                  │ const T = {                    │
|   Type.String(),               │  arg0: string,              │   type : 'function',           │
│   Type.Number()                │  arg1: number               │   parameters: [{               │
│ ], Type.Boolean())             │ ) => boolean                │     type: 'string'             │
│                                │                             │   }, {                         │
│                                │                             │     type: 'number'             │
│                                │                             │   }],                          │
│                                │                             │   return: {                    │
│                                │                             │     type: 'boolean'            │
│                                │                             │   }                            │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Uint8Array()    │ type T = Uint8Array         │ const T = {                    │
│                                │                             │   type: 'object',              │
│                                │                             │   specialized: 'Uint8Array'    │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Promise(        │ type T = Promise<string>    │ const T = {                    │
│   Type.String()                │                             │   type: 'promise',             │
│ )                              │                             │   item: {                      │
│                                │                             │     type: 'string'             │
│                                │                             │   }                            │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Undefined()     │ type T = undefined          │ const T = {                    │
│                                │                             │   type: 'object',              │
│                                │                             │   specialized: 'Undefined'     │
│                                │                             │ }                              │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Type.Void()          │ type T = void               │ const T = {                    │
│                                │                             │   type: 'null'                 │
│                                │                             │ }                              │
│                                │                             │                                │
└────────────────────────────────┴─────────────────────────────┴────────────────────────────────┘
```

<a name="Reference-Types"></a>

## Reference Types

Use `Type.Ref(...)` to create referenced types. The target type must specify an `$id`.

```typescript
const T = Type.String({ $id: 'T' })                  // const T = {
                                                     //    $id: 'T',
                                                     //    type: 'string'
                                                     // }
                                             
const R = Type.Ref(T)                                // const R = {
                                                     //    $ref: 'T'
                                                     // }
```

<a name="Recursive-Types"></a>

## Recursive Types

Use `Type.Recursive(...)` to create recursive types.

```typescript
const Node = Type.Recursive(Node => Type.Object({    // const Node = {
  id: Type.String(),                                 //   $id: 'Node',
  nodes: Type.Array(Node)                            //   type: 'object',
}), { $id: 'Node' })                                 //   properties: {
                                                     //     id: {
                                                     //       type: 'string'
                                                     //     },
                                                     //     nodes: {
                                                     //       type: 'array',
                                                     //       items: {
                                                     //         $ref: 'Node'
                                                     //       }
                                                     //     }
                                                     //   },
                                                     //   required: [
                                                     //     'id',
                                                     //     'nodes'
                                                     //   ]
                                                     // }

type Node = Static<typeof Node>                      // type Node = {
                                                     //   id: string
                                                     //   nodes: Node[]
                                                     // }

function test(node: Node) {
  const id = node.nodes[0].nodes[0].nodes[0]         // id is string
                 .nodes[0].nodes[0].nodes[0]
                 .nodes[0].nodes[0].nodes[0]
                 .id
}
```

<a name="Generic-Types"></a>

## Generic Types

Use functions to create generic types. The following creates a generic `Nullable<T>` type. 

```typescript
import { Type, Static, TSchema } from '@sinclair/typebox'

const Nullable = <T extends TSchema>(type: T) => Type.Union([type, Type.Null()])

const T = Nullable(Type.String())                    // const T = {
                                                     //   anyOf: [{
                                                     //     type: 'string'
                                                     //   }, {
                                                     //     type: 'null'
                                                     //   }]
                                                     // }

type T = Static<typeof T>                            // type T = string | null

const U = Nullable(Type.Number())                    // const U = {
                                                     //   anyOf: [{
                                                     //     type: 'number'
                                                     //   }, {
                                                     //     type: 'null'
                                                     //   }]
                                                     // }

type U = Static<typeof U>                            // type U = number | null
```

<a name="Unsafe-Types"></a>

## Unsafe Types

Use `Type.Unsafe(...)` to create custom schemas with user defined inference rules.

```typescript
const T = Type.Unsafe<string>({ type: 'number' })    // const T = {
                                                     //   type: 'number'
                                                     // }

type T = Static<typeof T>                            // type T = string
```

The `Type.Unsafe(...)` function can be used to create schemas for validators that require specific schema representations. An example of this would be OpenAPI's `nullable` and `enum` schemas which are not provided by TypeBox. The following demonstrates using `Type.Unsafe(...)` to create these types.

```typescript
import { Type, Static, TSchema } from '@sinclair/typebox'

//--------------------------------------------------------------------------------------------
//
// Nullable<T>
//
//--------------------------------------------------------------------------------------------

function Nullable<T extends TSchema>(schema: T) {
  return Type.Unsafe<Static<T> | null>({ ...schema, nullable: true })
}

const T = Nullable(Type.String())                    // const T = {
                                                     //   type: 'string',
                                                     //   nullable: true
                                                     // }

type T = Static<typeof T>                            // type T = string | null

//--------------------------------------------------------------------------------------------
//
// StringEnum<string[]>
//
//--------------------------------------------------------------------------------------------

function StringEnum<T extends string[]>(values: [...T]) {
  return Type.Unsafe<T[number]>({ type: 'string', enum: values })
}

const T = StringEnum(['A', 'B', 'C'])                // const T = {
                                                     //   enum: ['A', 'B', 'C']
                                                     // }

type T = Static<typeof T>                            // type T = 'A' | 'B' | 'C'
```

<a name="Conditional-Types"></a>

## Conditional Types

Use `Conditional.Extends(...)` to create conditional mapped types.

```typescript
import { Conditional } from '@sinclair/typebox/conditional'
```
The following table shows the TypeBox mappings between TypeScript and JSON schema.

```typescript
┌────────────────────────────────┬─────────────────────────────┬────────────────────────────────┐
│ TypeBox                        │ TypeScript                  │ JSON Schema                    │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Conditional.Extends( │ type T =                    │ const T = {                    │
│   Type.String(),               │  string extends number      │   const: false,                │
│   Type.Number(),               │  true : false               │   type: 'boolean'              │
│   Type.Literal(true),          │                             │ }                              │
│   Type.Literal(false)          │                             │                                │
│ )                              │                             │                                │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Conditional.Extract( │ type T = Extract<           │ const T = {                    │
│   Type.Union([                 │   'a' | 'b' | 'c',          │   anyOf: [{                    │
│     Type.Literal('a'),         │   'a' | 'f'                 │     const: 'a'                 │
│     Type.Literal('b'),         │ >                           │     type: 'string'             │
│     Type.Literal('c')          │                             │   }]                           │
│   ]),                          │                             │ }                              │
│   Type.Union([                 │                             │                                │
│     Type.Literal('a'),         │                             │                                │
│     Type.Literal('f')          │                             │                                │
│   ])                           │                             │                                │
│ )                              │                             │                                │
│                                │                             │                                │
├────────────────────────────────┼─────────────────────────────┼────────────────────────────────┤
│ const T = Conditional.Exclude( │ type T = Exclude<           │ const T = {                    │
│   Type.Union([                 │   'a' | 'b' | 'c',          │   anyOf: [{                    │
│     Type.Literal('a'),         │   'a'                       │     const: 'b',                │
│     Type.Literal('b'),         │ >                           │     type: 'string'             │
│     Type.Literal('c')          │                             │   }, {                         │
│   ]),                          │                             │     const: 'c',                │
│   Type.Union([                 │                             │     type: 'string'             │
│     Type.Literal('a')          │                             │   }]                           │
│   ])                           │                             │ }                              │
│ )                              │                             │                                │
│                                │                             │                                │
└────────────────────────────────┴─────────────────────────────┴────────────────────────────────┘
```

<a name="Values"></a>

## Values

Use `Value.Create(...)` to generate values from types.

```typescript
import { Value } from '@sinclair/typebox/value'
import { Type } from '@sinclair/typebox'

const T = Type.Object({
  x: Type.Number({ default: 1 }),
  y: Type.Number()
})

const V = Value.Create(T)                            // const V = {
                                                     //   x: 1,
                                                     //   y: 0
                                                     // }
```
Use `Value.Cast(...)` to cast a value into a given type.
```typescript
import { Value } from '@sinclair/typebox/value'
import { Type } from '@sinclair/typebox'

const T = Type.Object({
  x: Type.Number(),
  y: Type.Number()
})

const A = Value.Cast(T, null)                          // const A = { x: 0, y: 0 }

const B = Value.Cast(T, { x: 1 })                      // const B = { x: 1, y: 0 }

const C = Value.Cast(T, { x: 1, y: 2, z: 3 })          // const C = { x: 1, y: 2 }
```

<a name="Guards"></a>

## Guards

Use a `TypeGuard` to test if a value meets a TypeBox type specification. Guards can be helpful when reflecting types.

```typescript
import { TypeGuard } from '@sinclair/typebox/guard'
import { Type }  from '@sinclair/typebox'

const T = Type.String()

if(TypeGuard.TString(T)) {
    
  // T is TString
}

```

<a name="Strict"></a>

## Strict

TypeBox schemas contain the `Kind` and `Modifier` symbol properties. These properties are provided to enable runtime type reflection on schemas, as well as helping TypeBox internally compose types. These properties are not strictly valid JSON schema; so in some cases it may be desirable to omit them. TypeBox provides a `Type.Strict()` function that will omit these properties if necessary.

```typescript
const T = Type.Object({                              // const T = {
  name: Type.Optional(Type.String())                 //   [Kind]: 'Object',
})                                                   //   type: 'object',
                                                     //   properties: {
                                                     //     name: {
                                                     //       [Kind]: 'String',
                                                     //       type: 'string',
                                                     //       [Modifier]: 'Optional'
                                                     //     }
                                                     //   }
                                                     // }

const U = Type.Strict(T)                             // const U = {
                                                     //   type: 'object', 
                                                     //   properties: { 
                                                     //     name: { 
                                                     //       type: 'string' 
                                                     //     } 
                                                     //   } 
                                                     // }
```

<a name="Validation"></a>

## Validation

TypeBox schemas target JSON Schema draft 6 so any validator capable of draft 6 should be fine. A good library to use for validation in JavaScript environments is [Ajv](https://www.npmjs.com/package/ajv). The following example shows setting up Ajv to work with TypeBox.

```bash
$ npm install ajv ajv-formats --save
```

```typescript
//--------------------------------------------------------------------------------------------
//
// Import TypeBox and Ajv
//
//--------------------------------------------------------------------------------------------

import { Type }   from '@sinclair/typebox'
import addFormats from 'ajv-formats'
import Ajv        from 'ajv'

//--------------------------------------------------------------------------------------------
//
// Setup Ajv validator with the following options and formats
//
//--------------------------------------------------------------------------------------------

const ajv = addFormats(new Ajv({}), [
  'date-time', 
  'time', 
  'date', 
  'email',  
  'hostname', 
  'ipv4', 
  'ipv6', 
  'uri', 
  'uri-reference', 
  'uuid',
  'uri-template', 
  'json-pointer', 
  'relative-json-pointer', 
  'regex'
])

//--------------------------------------------------------------------------------------------
//
// Create a TypeBox type
//
//--------------------------------------------------------------------------------------------

const T = Type.Object({
  x: Type.Number(),
  y: Type.Number(),
  z: Type.Number()
})

//--------------------------------------------------------------------------------------------
//
// Validate Data
//
//--------------------------------------------------------------------------------------------

const R = ajv.validate(T, { x: 1, y: 2, z: 3 })      // const R = true
```

Please refer to the official Ajv [documentation](https://ajv.js.org/guide/getting-started.html) for additional information on using Ajv.

<a name="Compiler"></a>

## Compiler

TypeBox provides an optional high performance runtime type checker that can be used in applications that require extremely fast validation. This type checker is optimized for TypeBox types only whose schematics are known in advance. If defining custom schemas with `Type.Unsafe<T>` please consider Ajv.

The following demonstrates its use.

```typescript
import { TypeCompiler } from '@sinclair/typebox/compiler'
import { Type } from '@sinclair/typebox'

const C = TypeCompiler.Compile(Type.Object({         // const C: TypeCheck<TObject<{
  x: Type.Number(),                                  //     x: TNumber;
  y: Type.Number(),                                  //     y: TNumber;
  z: Type.Number()                                   //     z: TNumber;
}))                                                  // }>>

const R = C.Check({ x: 1, y: 2, z: 3 })              // const R = true 
```

Validation errors can be read with the `Errors(...)` function.

```typescript
const C = TypeCompiler.Compile(Type.Object({         // const C: TypeCheck<TObject<{
  x: Type.Number(),                                  //     x: TNumber;
  y: Type.Number(),                                  //     y: TNumber;
  z: Type.Number()                                   //     z: TNumber;
}))                                                  // }>>

const value = { }

const errors = [...C.Errors(value)]                  // const errors = [{
                                                     //   schema: { type: 'number' },
                                                     //   path: '/x',
                                                     //   value: undefined,
                                                     //   message: 'Expected number'
                                                     // }, {
                                                     //   schema: { type: 'number' },
                                                     //   path: '/y',
                                                     //   value: undefined,
                                                     //   message: 'Expected number'
                                                     // }, {
                                                     //   schema: { type: 'number' },
                                                     //   path: '/z',
                                                     //   value: undefined,
                                                     //   message: 'Expected number'
                                                     // }]
```

Compiled routines can be inspected with the `.Code()` function.

```typescript
const C = TypeCompiler.Compile(Type.String())        // const C: TypeCheck<TString>

console.log(C.Code())                                // return function check(value) {
                                                     //   return (
                                                     //     (typeof value === 'string')
                                                     //   )
                                                     // }
```

## Benchmark

This project maintains benchmarks that measure Ajv and TypeCompiler validate and compile performance. These benchmarks can be run locally by cloning this repository and running `npm run benchmark`. Results show against Ajv version 8.11.0.

### Validate

This benchmark measures validation performance for varying types. You can review this benchmark [here](https://github.com/sinclairzx81/typebox/blob/master/benchmark/check.ts).

```typescript
┌──────────────────┬────────────┬──────────────┬──────────────┬──────────────┐
│     (index)      │ Iterations │     Ajv      │ TypeCompiler │ Performance  │
├──────────────────┼────────────┼──────────────┼──────────────┼──────────────┤
│           Number │  4000000   │ '     17 ms' │ '     16 ms' │ '    1.06 x' │
│           String │  4000000   │ '     74 ms' │ '     37 ms' │ '    2.00 x' │
│          Boolean │  4000000   │ '     68 ms' │ '     36 ms' │ '    1.89 x' │
│             Null │  4000000   │ '     68 ms' │ '     36 ms' │ '    1.89 x' │
│            RegEx │  4000000   │ '    170 ms' │ '    143 ms' │ '    1.19 x' │
│          ObjectA │  4000000   │ '    122 ms' │ '     84 ms' │ '    1.45 x' │
│          ObjectB │  4000000   │ '    182 ms' │ '    137 ms' │ '    1.33 x' │
│            Tuple │  4000000   │ '     82 ms' │ '     48 ms' │ '    1.71 x' │
│            Union │  4000000   │ '     84 ms' │ '     52 ms' │ '    1.62 x' │
│        Recursive │  4000000   │ '   1526 ms' │ '    631 ms' │ '    2.42 x' │
│          Vector4 │  4000000   │ '     80 ms' │ '     42 ms' │ '    1.90 x' │
│          Matrix4 │  4000000   │ '    157 ms' │ '    113 ms' │ '    1.39 x' │
│   Literal_String │  4000000   │ '     69 ms' │ '     36 ms' │ '    1.92 x' │
│   Literal_Number │  4000000   │ '     69 ms' │ '     35 ms' │ '    1.97 x' │
│  Literal_Boolean │  4000000   │ '     68 ms' │ '     35 ms' │ '    1.94 x' │
│     Array_Number │  4000000   │ '    119 ms' │ '     60 ms' │ '    1.98 x' │
│     Array_String │  4000000   │ '    119 ms' │ '     76 ms' │ '    1.57 x' │
│    Array_Boolean │  4000000   │ '    131 ms' │ '     88 ms' │ '    1.49 x' │
│    Array_ObjectA │  4000000   │ '  10615 ms' │ '   7053 ms' │ '    1.51 x' │
│    Array_ObjectB │  4000000   │ '  11305 ms' │ '   8128 ms' │ '    1.39 x' │
│      Array_Tuple │  4000000   │ '    355 ms' │ '    271 ms' │ '    1.31 x' │
│      Array_Union │  4000000   │ '    910 ms' │ '    340 ms' │ '    2.68 x' │
│  Array_Recursive │  4000000   │ '  29101 ms' │ '  10837 ms' │ '    2.69 x' │
│    Array_Vector4 │  4000000   │ '    381 ms' │ '    212 ms' │ '    1.80 x' │
│    Array_Matrix4 │  4000000   │ '   1534 ms' │ '   1344 ms' │ '    1.14 x' │
└──────────────────┴────────────┴──────────────┴──────────────┴──────────────┘
```

### Compile

This benchmark measures compilation performance for varying types. You can review this benchmark [here](https://github.com/sinclairzx81/typebox/blob/master/benchmark/compile.ts).

```typescript
┌──────────────────┬────────────┬──────────────┬──────────────┬──────────────┐
│     (index)      │ Iterations │     Ajv      │ TypeCompiler │ Performance  │
├──────────────────┼────────────┼──────────────┼──────────────┼──────────────┤
│           Number │    2000    │ '    384 ms' │ '      7 ms' │ '   54.86 x' │
│           String │    2000    │ '    311 ms' │ '      6 ms' │ '   51.83 x' │
│          Boolean │    2000    │ '    304 ms' │ '      5 ms' │ '   60.80 x' │
│             Null │    2000    │ '    252 ms' │ '      5 ms' │ '   50.40 x' │
│            RegEx │    2000    │ '    480 ms' │ '      7 ms' │ '   68.57 x' │
│          ObjectA │    2000    │ '   2786 ms' │ '     27 ms' │ '  103.19 x' │
│          ObjectB │    2000    │ '   2946 ms' │ '     22 ms' │ '  133.91 x' │
│            Tuple │    2000    │ '   1277 ms' │ '     16 ms' │ '   79.81 x' │
│            Union │    2000    │ '   1288 ms' │ '     16 ms' │ '   80.50 x' │
│          Vector4 │    2000    │ '   1628 ms' │ '     12 ms' │ '  135.67 x' │
│          Matrix4 │    2000    │ '    912 ms' │ '      7 ms' │ '  130.29 x' │
│   Literal_String │    2000    │ '    344 ms' │ '      5 ms' │ '   68.80 x' │
│   Literal_Number │    2000    │ '    432 ms' │ '      5 ms' │ '   86.40 x' │
│  Literal_Boolean │    2000    │ '    407 ms' │ '      4 ms' │ '  101.75 x' │
│     Array_Number │    2000    │ '    788 ms' │ '      9 ms' │ '   87.56 x' │
│     Array_String │    2000    │ '    823 ms' │ '      7 ms' │ '  117.57 x' │
│    Array_Boolean │    2000    │ '    816 ms' │ '      5 ms' │ '  163.20 x' │
│    Array_ObjectA │    2000    │ '   4270 ms' │ '     24 ms' │ '  177.92 x' │
│    Array_ObjectB │    2000    │ '   4008 ms' │ '     28 ms' │ '  143.14 x' │
│      Array_Tuple │    2000    │ '   2243 ms' │ '     13 ms' │ '  172.54 x' │
│      Array_Union │    2000    │ '   1734 ms' │ '     16 ms' │ '  108.38 x' │
│    Array_Vector4 │    2000    │ '   2348 ms' │ '     14 ms' │ '  167.71 x' │
│    Array_Matrix4 │    2000    │ '   1608 ms' │ '     11 ms' │ '  146.18 x' │
└──────────────────┴────────────┴──────────────┴──────────────┴──────────────┘
```

<a name="Contribute"></a>

## Contribute

TypeBox is open to community contribution. Please ensure you submit an open issue before submitting your pull request. The TypeBox project preferences open community discussion prior to accepting new features.
