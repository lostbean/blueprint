# blueprint

Blueprint is a Gleam library that simplifies JSON encoding and decoding while automatically generating JSON schemas for your data types.

[![Package Version](https://img.shields.io/hexpm/v/blueprint)](https://hex.pm/packages/blueprint)
[![Hex Docs](https://img.shields.io/badge/hex-docs-ffaff3)](https://hexdocs.pm/blueprint/)

```sh
gleam add blueprint
```

## Usage

Blueprint provides utilities for encoding and decoding JSON data, with special support for union types.

### Encoding Union Types

Here's an example of encoding a union type to JSON:

```gleam
import blueprint
import gleam/json
import gleam/io
import gleeunit/should

pub type Shape {
  Circle(Float)
  Rectangle(Float, Float)
}

fn encode_shape(shape: Shape) -> json.Json {
  blueprint.union_type_encoder(shape, fn(shape_case) {
    case shape_case {
      Circle(radius) -> #(
        "circle",
        json.object([#("radius", json.float(radius))]),
      )
      Rectangle(width, height) -> #(
        "rectangle",
        json.object([
          #("width", json.float(width)),
          #("height", json.float(height)),
        ]),
      )
    }
  })
}

fn shape_decoder() -> blueprint.Decoder(Shape) {
  blueprint.union_type_decoder([
    #(
      "circle",
      blueprint.decode1(Circle, blueprint.field("radius", blueprint.float())),
    ),
    #(
      "rectangle",
      blueprint.decode2(
        Rectangle,
        blueprint.field("width", blueprint.float()),
        blueprint.field("height", blueprint.float()),
      ),
    ),
  ])
}

fn simple_test() {
  let decoder = shape_decoder()

  // Test encoding a Circle
  let circle = Circle(5.0)
  encode_shape(circle)
  |> json.to_string
  |> blueprint.decode(using: decoder)
  |> should.equal(Ok(circle))

  // Test encoding a Rectangle
  let rectangle = Rectangle(10.0, 20.0)
  encode_shape(rectangle)
  |> json.to_string
  |> blueprint.decode(using: decoder)
  |> should.equal(Ok(rectangle))

  // Print JSON schema
  decoder
  |> blueprint.generate_json_schema()
  |> json.to_string
  |> io.println
}
```

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "oneOf": [
    {
      "required": ["type", "data"],
      "additionalProperties": false,
      "type": "object",
      "properties": {
        "type": {
          "enum": ["circle"]
        },
        "data": {
          "required": ["radius"],
          "additionalProperties": false,
          "type": "object",
          "properties": {
            "radius": {
              "type": "number"
            }
          }
        }
      }
    },
    {
      "required": ["type", "data"],
      "additionalProperties": false,
      "type": "object",
      "properties": {
        "type": {
          "enum": ["rectangle"]
        },
        "data": {
          "required": ["width", "height"],
          "additionalProperties": false,
          "type": "object",
          "properties": {
            "width": {
              "type": "number"
            },
            "height": {
              "type": "number"
            }
          }
        }
      }
    }
  ]
}
```

This will encode your union types into a standardized JSON format with `type` and `data` fields, making it easy to decode on the receiving end.

## Features

- 🎯 Type-safe JSON encoding and decoding
- 🔄 Support for union types with standardized encoding
- 📋 Automatic JSON schema generation
- ✨ Clean and intuitive API

Further documentation can be found at <https://hexdocs.pm/blueprint>.

## Development

```sh
gleam run   # Run the project
gleam test  # Run the tests
```
