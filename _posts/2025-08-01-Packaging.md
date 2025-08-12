---
title: "Sharing Python Tools Inside Your Small Company"
published: false
---

* Your first instinct is to reach over to different file via your file system and read the Python code from there. It's actually easy to do that. You can find the `sys.path` hacks quickly.
* You have to copy-paste that hack into every new source file you write.
* Or you have to hack your `PYTHON_PATH` environment variable everywhere you go.

## Proposed Solution

* Use `pyproject.toml` and the recommended structure.
* Put your modules in their own repositories
* `python -m pip install        magic formula to access code from git repositories`
    * This requires your colleagues have access to your git repository. Alternatively, you could publish projects to less-restrictive places, like a network drive.
    * Then, your colleagues would have to `python -m pip install path-to-that-directory`.