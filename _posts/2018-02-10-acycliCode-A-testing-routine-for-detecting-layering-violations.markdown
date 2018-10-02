---
layout: post
title:  "acycliCode: A testing routine for detecting layering violations"
date:   2018-02-10 00:05:20 +0200
categories:
---

> This blog post is reposted from my original article [on Medium](https://medium.com/@papachristoumarios/acyclicode-a-testing-routine-for-detecting-layering-violations-2fbb725a70e0).



Layered architectures were first proposed by Dijkstra and Parnas [1]. They suggested the organization of the program modules into layers and establish a hierarchy between them such that modules can call functions from modules lying in adjoining layers. However, they did not devise systematic ways of analyzing such layered architectural schemata. There has been a research interest around layering violations in the recent years since codebases have started becoming huge.

Violations of the layered architecture can occur primarily due to lack of experience from the part of the contributors or the reviewers of the pull-requests. Besides lack of experience, a time shortage or a huge volume of pull-requests in order to meet certain deadlines such as releases can also be a reason for committing code that invokes violating calls. Sadly enough, modern professional continuous integration (CI) platforms, like Travis CI or CircleCI, do not offer features for the detections of layering violations on pull-requests.

Consider that you are starting a project and you want to adopt a layered architecture. Your aim is to have always modules that lie in their respective layers and do not violate the existing architecture as described above. You have decided the source code files are part of each layer. For example, consider a simple project with two files demo/foo.c and demo/boo.c:

```
{
	"demo/foo.c": 0, 
	"demo/boo.c": 1
}
```

This definition tells us that demo/foo.c lies in Layer 0 and demo/boo.c lies in Layer 1.

The contents of the files are:

```
// demo/foo.c
#include "boo.h"
```

```
int f(int x) {
	return g(x);
}
```

and

```
// demo/boo.c
#include "foo.h"
```

```
int g(int x) {
	return x; 
}
```

```
int h(int x) {
	return f(x);  
}
```

Its obvious that there is a layering violation since f(x) calls g(x) and thus there’s a call from Layer 0 to Layer 1. That, of course, should not be allowed in a layered architecture.

The main idea here is that *we can prevent this violation from happening via a failing test at the commit it caused it*. That is that for every commit h each and for every layer u and v each u -> v call must satisfy that `depth[u] >= depth[v]`. It is therefore easy to show that if the edge set for `HEAD^1` contains no violations and the added function calls do not introduce any cycles then `HEAD` must contain no cycles.

For example in a scheme with 3 layers (the colored edges correspond to the calls that were introduced in the current commit):

![img](https://cdn-images-1.medium.com/max/880/0*WIpjEWhKA9L0NSiR.png)

The first of the three options depicted above is a valid option whereas the second and third options should not be allowed. This differential approach allows for preventing the problem from happening and can be integrated into CI.

These edges can be generated using GNU Cflow via the command:

```
cflow <changed-files> <direct-dependencies> -d 2
```

Where `<changed-files>` correspond to the changed files and `<direct-dependencies>` correspond to the *direct includes* of the changed files. The output of `cflow` is then parsed to detect cycles in a single iteration.

**acycliCode** is a library for detecting Layering Violations as part of Continuous Integration (CI) checks projects *with a C codebase*. It is built on *Python 3.x and* [*GNU Cflow*](https://www.gnu.org/software/cflow/) and follows a “differential” approach of cycle detections keeping layers in disjoint sets. It uses this approach to prevent the problem of layering violations from happening. It features checking for layering violations using Cflow static flow graph generator. It is released under the *MIT License*.

Run on the above example it detects the violation, and the CI routine test fails, disallowing the violation from happening.

You can supply it with the commit hash you want with the `-c` flag and the layer definition files via `-l` . Finally, an `--asert` option is available for making passing/failing tests for CI just like you did with `pytest` on Python files in other projects. For example:

```
acyclicode -l demo/layers.json -c <commit-hash> --assert
```

The results are:

```
WARNING : Back Calls
WARNING : demo/foo.c (Layer 0) -> demo/boo.c (Layer 1)
WARNING : Average BCVI: 1.0
INFO : No Skip Calls Found
```

In this way, if integrated with the project from early on, acycliCode solves the problem of layering violations in software architectures with this “preventive manner”.

### References

GitHub Repository: <https://github.com/papachristoumarios/acycliCode>

Contributing: <https://github.com/papachristoumarios/acycliCode/issues>

[1] Sarkar, Santonu, Girish Maskeri Rama, and R. Shubha. “A method for detecting and measuring architectural layering violations in source code.” *Software Engineering Conference, 2006. APSEC 2006. 13th Asia Pacific*. IEEE, 2006.