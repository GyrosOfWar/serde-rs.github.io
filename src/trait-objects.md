# Using Serde trait objects

The usual Serde `Serialize` and `Serializer` traits cannot be used as [trait
objects](https://doc.rust-lang.org/book/trait-objects.html) like `&Serialize` or
boxed trait objects like `Box<Serialize>` because of Rust's ["object safety"
rules](http://huonw.github.io/blog/2015/01/object-safety/). In particular, both
traits contain generic methods which cannot be made into a trait object.

The [`erased-serde`](https://github.com/dtolnay/erased-serde) crate provides
versions of the `Serialize` and `Serializer` traits that are object-safe and can
be used as `&Serialize`/`&Serializer` and `Box<Serialize>`/`Box<Serializer>`.

The traits in `erased-serde` can be used seamlessly with any existing Serde
`Serialize` type and any existing Serde `Serializer` format.

```rust
extern crate erased_serde;
extern crate serde_json;
extern crate serde_cbor;

use std::collections::BTreeMap as Map;
use std::io::stdout;

use erased_serde::{Serialize, Serializer};

fn main() {
    // The values in this map are boxed trait objects. Ordinarily this would not
    // be possible with serde::Serializer because of object safety, but type
    // erasure makes it possible with erased_serde::Serializer.
    let mut formats: Map<&str, Box<Serializer>> = Map::new();
    formats.insert("json", Box::new(serde_json::ser::Serializer::new(stdout())));
    formats.insert("cbor", Box::new(serde_cbor::ser::Serializer::new(stdout())));

    // These are boxed trait objects as well. Same thing here - type erasure
    // makes this possible.
    let mut values: Map<&str, Box<Serialize>> = Map::new();
    values.insert("vec", Box::new(vec!["a", "b"]));
    values.insert("int", Box::new(65536));

    // Pick a Serializer out of the formats map.
    let format = formats.get_mut("json").unwrap();

    // Pick a Serialize out of the values map.
    let value = values.get("vec").unwrap();

    // This line prints `["a","b"]` to stdout.
    value.erased_serialize(format).unwrap();
}
```
