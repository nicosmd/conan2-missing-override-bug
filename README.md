# Introduction

This Repo demonstrates a bug found in conan version 2.0.3. The bug is caused by an override statement in a downstream package to a dependency which is never
required as a real dependency in the result graph. 

The follwing diagram illustrates the dependency graph:

![image](www.plantuml.com/plantuml/png/SoWkIImgAStDuOhEoKnAZbNGrRLJ036JOj554Y4YIIWY44f1JmyeKWLanrefv9Ub5XLbfgHoEQJcfG1D1W00)

# Reproduction

In order to reproduce the bug, you need to build all three packages in the following order: 
    1. liba
    2. libb
    3. libc

using `conan create .`. Make sure you have conan 2.0.3 installed. The issue might exists in older versions as well but it was not tested.

During the build of libc you shoud receive the following error message:

```
Traceback (most recent call last):
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conan/cli/cli.py", line 272, in main
    cli.run(args)
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conan/cli/cli.py", line 171, in run
    command.run(self._conan_api, self._commands[command_argument].parser, args[0][1:])
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conan/cli/command.py", line 157, in run
    info = self._method(conan_api, parser, *args)
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conan/cli/commands/create.py", line 69, in create
    deps_graph = conan_api.graph.load_graph_requires(requires, tool_requires,
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conan/api/subapi/graph.py", line 123, in load_graph_requires
    deps_graph = self.load_graph(root_node, profile_host=profile_host,
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conan/api/subapi/graph.py", line 172, in load_graph
    deps_graph = builder.load_graph(root_node, profile_host, profile_build, lockfile)
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conans/client/graph/graph_builder.py", line 53, in load_graph
    new_node = self._expand_require(require, node, dep_graph, profile_host,
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conans/client/graph/graph_builder.py", line 100, in _expand_require
    node.propagate_closing_loop(require, prev_node)
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conans/client/graph/graph.py", line 82, in propagate_closing_loop
    prev_node.propagate_downstream(transitive.require, transitive.node, self)
  File "/home/sin7hi/miniconda3/envs/conan2/lib/python3.10/site-packages/conans/client/graph/graph.py", line 86, in propagate_downstream
    assert node is not None
AssertionError

ERROR:
```
# Remarks

It is interesting that the error occurs only in that specific dependency graph pattern. When removing libb or liba from libc, the package creation is fine.
I'm not sure what the right behvior should look like. Maybe I would expect some warning but the build should succeed anyway even if the override 
dependency was never *materialized*.
