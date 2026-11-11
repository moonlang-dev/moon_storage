# moon_storage

A keyed filesystem over one or more rooted directories.

```toml
[[deps]]
name = "moon_storage"
path = "../packages/moon_storage"
```

A key is a relative path, and every operation resolves it inside a root and
refuses to leave: `..`, an absolute key, and a symlink pointing outward are
containment failures rather than reads of somebody else's file. The check is
segment-aware, because a prefix test alone accepts `/videos-evil` for a root of
`/videos`. Multi-root because a library outgrows a disk — reads search every
root in order, writes go to the first, and the caller sees one namespace.

```moon
use moon_storage.storage
use moon_storage.storage.{Adapter, MultiAdapter}

let thumbs = storage.create("/var/lib/service/thumbs").unwrap()
let _ = thumbs.write("a/b.jpg", bytes)
let found = thumbs.read("a/b.jpg")            // Optional<string>
let escaped = thumbs.read("../../etc/passwd") // none: outside the root

// One namespace over several disks: reads search each root in order, writes
// land in the first.
let library = storage.create_multi(["/mnt/a/videos", "/mnt/b/videos"]).unwrap()
let path = library.path_of("clips/x.mp4")     // Optional<string>, whichever root has it
let space = storage.disk_space_of("/mnt/a/videos")
```

`contains_path(root, path)` is the containment check on its own, and
`absolute(value)` is `path.resolve` — for a caller holding a path from
somewhere else that has to know whether it is inside a root before touching
it.

This was extracted from `examples/moonvids` (plan 116 phase 4, its own package
since plan 138). The containment check is the valuable part and it is
`cap-std`-shaped: a capability to a directory, not a path you promise to be
careful with.
