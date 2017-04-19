---
layout: post
title: Inside Git
description: Git backend exploration
date: 2015-07-16
categories: technology update
img: git-objects.png
author: Carlos Raffellini
---

### Commands to explore Git Objects

Git stores only three kind of objects: commit, tree, blob.

Objects are stored objects using the first two characters of hash to place the folder and the next characters to place the file. Ex: an object with hash `bd9dbf5aae1a3862dd1526723246b20206e5fc37` is stored in `.git/objects/bd/9dbf5aae1a3862dd1526723246b20206e5fc37`.

(Write abouth SHA1 hash and zip with zlib)

A commit has important metadata about autor, a tree hash, and commit fathers' hash.

- `git cat-file <option> hash` display information about an object
  - `-p` prints in pretty output format the object's content
  - `-t` returns the type
  - `-s` returns the content's size

  ```bash
	> $ git cat-file -t 39f25aaa
	commit

	> $ git cat-file -p 39f25aaa
	tree 48a8ddfdc7bffc37658f32424e767ea453ce3e7a
	parent ed2a1be40c0eaef5a27fa366c7fcc6412fd628b2
	author charlyraffellini <del0al99@gmail.com> 1437353110 -0300
	committer charlyraffellini <del0al99@gmail.com> 1437353110 -0300

	Update API example.
  ```

- `git ls-tree [<options>] tree-hash` diysplay a table with permisons, type, hash, path for each object in tree.
  - `-r` recurse into subtrees
  - [more](git-ls-tree link)

  ```bash
  > $ git ls-tree -r 48a8ddfdc                                                                                
  100644 blob a7874e28b9115e8a1160961fea9cf37e10caa632	LICENSE
  100644 blob 2032652a29555f5e7a5c1b0e60980bbfbc9f5858	Makefile
  100644 blob ef687bf87cadaa84c5b82bf5a155af43a3eb8a62	README.md
  100644 blob 8a74d87deb13997b6972ea9d3bc1e204ea7b83f7	TUTORIAL.md
  100644 blob 273a6c6513b5dd42c8db583be4b7a3a377e4f663	alert.js
  100644 blob 72e8ffc0db8aad71a934dd11e5968bd5109e54b4	deps/.gitignore
  100644 blob 72e8ffc0db8aad71a934dd11e5968bd5109e54b4	obj/.gitignore
  100644 blob 1379b5f29c1ccd949f5fb58286614f7c472d53a2	src/jsnotify.cpp
  ```


