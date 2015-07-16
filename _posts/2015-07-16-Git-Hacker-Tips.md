### Commands to explore Git Objects

Git stores only three kind of objects: commit, tree, blob.

Objects are stored objects using the first two characters of hash to place the folder and the next characters to place the file. Ex: an object with hash `bd9dbf5aae1a3862dd1526723246b20206e5fc37` is stored in `.git/objects/bd/9dbf5aae1a3862dd1526723246b20206e5fc37`.

(Write abouth SHA1 hash and zip with zlib)

A commit has important metadata about autor, a tree hash, and commit fathers' hash.

- `git cat-file <option> hash` display information about an object
  - `-p` prints in pretty output format the object's content
  - `-t` returns the type
  - `-s` returns the content's size

- `git ls-tree [<options>] tree-hash` diysplay a table with permisons, type, hash, path for each object in tree.
  - `-r` recurse into subtrees
  - [more](git-ls-tree link)
