# CyberLab Wiki
---

This is a repository containing the wiki page for Adelaide University's open source CyberLab.
This is built with the `hugo-book` theme and is being updated.

## Running the wiki locally

Prerequisites:
 - `hugo`
 - `git`

1. Clone and `cd` into the repository

```bash
git clone https://github.com/au-cyberlab/wiki
cd wiki
```

2. Update the submodules

```bash
git submodule update --init --remote
```

This will update the submodules and allow `hugo` to compile and run the wiki.

3. Build/Run the server

To build:
```bash
hugo build
```

To build and run:
```bash
hugo server
```
