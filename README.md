# Redwood build system

Redwood is an experimental build system based around Datalog. The project
goal is to explore whether a scalable build system can be made as powerful as
existing tools like Bazel without the burdens they bring.

You should be able to:
* Override Redwood's opinions easily
* Learn only one simple & consistent language

And Redwood should:
* start fast
* plan fast
* build fast
* Be correct

## Why It's Cool
----

Don't know where something is defined
```bash

$ redwood query 'where_is(//foo:bar)'
BUILD.datalog:56: cc_binary("//foo:bar")
```



Force rebuild all the targets with MIT licenses

```bash
$ redwood build //myapp:server --with 'force_rebuild(X), license(_, "MIT").'
building //myapp/lib:transitive_dep ...
linking //myapp:server ...
Target //myapp:server successfully built!
```



Check if any dependencies are unused

```bash
$ redwood query 'is_orphaned(T).' --with 'is_orphaned(T) :- is_dep(T), not is_dependency_of_anything(T).'
dep("//little/orphan:annie")
```



Determine which targets changed between `7f3a2c1` and `b8e91d4`

```bash
$ redwood query 'affected_by_changes("7f3a2c1", "b8e91d4")'
target("//myapp/libs:backend")
target("//myapp/libs:database")
```



Find a PR reviewer

```bash
$ redwood query 'target_owner("//myapp/bin/http:server", Owner)'
target_owner("//myapp/bin/http", "charles")
```



Display licenses for every target in the repo:

```bash
$ redwood query "has_license(Target, L)"
license("//myapp:server", "PROPRIETARY")
license("//myapp/lib:gpl_lib", "GPL-3.0")
```



Profile your build path

```bash
$ redwood build //myapp:server 'explain_critical_path("//myapp:server")'
building //myapp/lib:transitive_dep ...
linking //myapp:server ...
Target //myapp:server successfully built!
=== Critical Path ===
//myapp/lib:transitive_dep (30s)
  //myapp/libs:backend (4s)
     //myapp:server (15s)
```



You can see these recipes and more in the [Redwood Cookbook](docs/cookbook/intro.md), or learn the basics in the [Redwood Tutorial](docs/tutorial/intro.md)

## Conceptual Model

----

Redwood treats your repository as a datalog program. You describe your projects as a series of facts -- sources, dependencies, licenses, etc -- and the system lazily executes that program with `redwood build` by "proving" those facts are sufficient to build the goal, executing any tools necessary to actually build it along the way as side effects.

When you `redwood query` instead or pass the `--dry-run` flag, the side effects simply aren't executed.

The power of the system comes from everything this model implies.

If your goal is fully satisfied by the facts Redwood already knows, it doesn't need to spend time downloading external modules resolve the entire dependency graph like Bazel.

Because Redwood is just proving goals are satisfied, by facts, it's not limited only to building code. You can prove anything you can express as basic facts and relations between them, like whether your targets only use dependencies with compatible licenses or solve for version constraints. It can even execute in different directions, an advanced feature better documented in the [Cookbook](docs/cookbook/executing_backwards.md)





