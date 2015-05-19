

# Compiler Invocation #

Suppose you have a file `protos/server/example.proto` that contains the following:

```
package server_protos;

message Foo {
  optional Bar bar = 1;
}

enum Bar {
  Crowbar = 1;
  Divebar = 2;
  Wunderbar = 3;
}

```

Suppose then that you invoke the compiler as follows:

```
hprotoc \
  --haskell_out=gen \
  --prefix=MyProtos \
  protos/server/example.proto
```

`hprotoc` generates one Haskell module for each message or enum:
  * `gen/MyProtos/Server_protos/Foo.hs`
  * `gen/MyProtos/Server_protos/Bar.hs`
and one Haskell module for each input `.proto` file:
  * `gen/MyProtos/Server_protos.hs`

The `--haskell_out` (or `-d`) option specifies the directory below which generated modules will be created. The directory must already exist. The `--prefix` (or `-p`) option specifies a prefix for the hierarchical name of each resulting module. This is useful for avoiding collisions with existing Haskell modules.

# Packages #

# Messages #

Given a message declaration:

```
message Foo {}
```

hprotoc generates a module `ModulePrefix.ProtoPackage.Foo`, where `ModulePrefix` is the prefix given with `--prefix` (if any) and `ProtoPackage` is derived from the `package` declaration (if any).

The module contains a data type `Foo`, with a single record-style constructor whose fields correspond to the fields of the source message. `Foo` is a member of the following standard type classes:
  * `Show`;
  * `Eq`;
  * `Ord`;
  * `Typeable`.
`Foo` is also a member of the following type classes:
  * [Text.ProtocolBuffers.Basic.Mergeable](http://hackage.haskell.org/packages/archive/protocol-buffers/latest/doc/html/Text-ProtocolBuffers-Basic.html#t:Mergeable) (providing `mergeAppend` for merging the contents of two messages);
  * [Text.ProtocolBuffers.Basic.Default](http://hackage.haskell.org/packages/archive/protocol-buffers/latest/doc/html/Text-ProtocolBuffers-Basic.html#t:Default) (providing `defaultValue`, a `Foo` whose fields all have their default values);
  * [Text.ProtocolBuffers.WireMessage.Wire](http://hackage.haskell.org/packages/archive/protocol-buffers/latest/doc/html/Text-ProtocolBuffers-WireMessage.html#t:Wire) (providing serialization and deserialization);
  * [Text.ProtocolBuffers.Extensions.GPB](http://hackage.haskell.org/packages/archive/protocol-buffers/latest/doc/html/Text-ProtocolBuffers-Extensions.html#t:GPB) (a convenient shorthand for all the above type classes);
  * [Text.ProtocolBuffers.Reflections.ReflectDescriptor](http://hackage.haskell.org/packages/archive/protocol-buffers/latest/doc/html/Text-ProtocolBuffers-Reflections.html#t:ReflectDescriptor) (allowing reflection of the structure of the data type, including the tag numbers of its fields).

## Message Names ##

A Haskell data type name is the same as the source message name, except:
  * if the first character of the name is alphabetic then it is forced to upper case;
  * if the first character of the name is an underscore then it is replaced with `U'`.
These rules ensure that the result is a valid Haskell name for a type.

# Fields #

Each field declared in a message has a label, a type and a name. Each corresponding Haskell field has a type and a name.

## Field Types ##

A generated Haskell record field has a type based on both the source type and the source label, as described by the following two tables:

| **Field Type** | **Haskell Type** |
|:---------------|:-----------------|
| double         | Double           |
| float          | Float            |
| int32          | Int32            |
| int64          | Int64            |
| uint32         | Word32           |
| uint64         | Word64           |
| sint32         | Int32            |
| sint64         | Int64            |
| fixed32        | Word32           |
| fixed64        | Word64           |
| sfixed32       | Int32            |
| sfixed64       | Int64            |
| bool           | Bool             |
| string         | Utf8             |
| bytes          | ByteString       |

| **Field Label** | **Haskell Type** |
|:----------------|:-----------------|
| required        | a                |
| optional        | Maybe a          |
| repeated        | Seq a            |

## Field Names ##

A Haskell field name is the same as the source field name, except:
  * if the name is a Haskell keyword, `'` is appended;
  * if the first character of the name is alphabetic then it is forced to lower case;
  * if the first character of the name is an underscore then it is replaced with `u'`.
These rules ensure that the result is a valid Haskell name for a value.

## Examples ##

| **Field Label and Type** | **Haskell Field Type** |
|:-------------------------|:-----------------------|
| `required bytes blob`    | `blob :: ByteString`   |
| `optional fixed64 hash`  | `hash :: Maybe Word32` |
| `repeated string instance` | `instance' :: Seq Utf8` |
| `optional bool CAPS`     | `cAPS :: Maybe Bool`   |

# Enumerations #

## Enumeration Names ##

Enumeration names are generated in the same way as [message names](#Message_Names.md).

# Extensions #

Given a message with an extension range:
```
message Foo {
  extensions 100 to 199;
}
```

`hprotoc` will declare `Foo` an instance of the type class [Text.ProtocolBuffers.Extensions.ExtendMessage](http://hackage.haskell.org/packages/archive/protocol-buffers/latest/doc/html/Text-ProtocolBuffers-Extensions.html#t:ExtendMessage).

Extensions can be declared nested inside of another type. For example, a common pattern is to do something like this:
```
message Baz {
  extend Foo {
    optional Baz foo_ext = 124;
  }
}
```
In this case, the extension key `foo_ext` is declared in the module for `Baz`.

Additionally `Baz` is an instance of one more type class:
  * [Text.ProtocolBuffers.Extensions.MessageAPI](http://hackage.haskell.org/packages/archive/protocol-buffers/latest/doc/html/Text-ProtocolBuffers-Extensions.html#t:MessageAPI).



# Services #