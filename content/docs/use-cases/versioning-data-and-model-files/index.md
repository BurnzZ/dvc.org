# Versioning Data and Models

Data science teams today face data management questions around versioning
datasets, data artifacts, and machine learning models. How do we keep track of
changes in data, code, and ML models? What’s the best way to organize and store
multiple versions of data files for safe, persistent access? How can the
intertwined lifecycles of data and models be followed and enforced?

![](/img/data_ver_complex.png) _Exponential complexity of DS projects_

DVC proposes to _codify_ data projects in order to adopt existing engineering
tools like Git
[version control](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control),
continuous integration (CI), and other best practices that improve productivity.
This means writing a description of which data, ML artifacts, etc. should be in
the environment at a given time. DVC can restore the <abbr>workspace</abbr>
files and directories (from a separate data storage) to match that description.

Data versioning lets us identify exact research inputs later, so anyone can
understand and reproduce results. DVC brings additional benefits on top of that:
make projects easier to work on by separating code from data (which also
increases data persistence); enforce lifecycle and security policies by having
all changes go though a Git repo (immutable history); collaborate with a toolset
everyone is familiar with (data scientists, engineers, managers), and others
(see [Get Started](/doc/start)).

But how does DVC look & feel? First we replaces large data files and directories
with tiny, human-readable _metafiles_ that can be versioned using Git.

**Before**: manual filename-based versioning mess with ad hoc conventions —
prone to human error

```dvc
 .
 ├── data
 │   ├── 2019-04
 │   │   └── raw
 │   ├── 2019-10
 │   │   ├── raw
 │   │   └── clean.txt
 │   ├── 2020-03
 │   │   ├── raw.txt
 │   │   ├── labels.csv
 │   │   ├── labels-2.csv
...
 ├── model.h5
 ├── model_final.h5
 ├── training.py  # Only file in Git
```

**After**: Lean DVC workspace that is easy to navigate. Only one version of the
data is shown along with the current code version. `dvc.yaml` and `.dvc`
metafiles replace all the complexity.

```git
 .
 ├── data
 │   ├── raw.txt
 │   ├── labels.csv
 ...
+├── data.dvc     # in Git
+├── dvc.yaml     # in Git
 ├── model.h5
 ├── training.py  # in Git
```

The bottom line is that your code doesn't need to read or write complicated
output file paths like `data/2019-10...`. Leave it to DVC to match the right
versions of code and data later 💘.

A regular `git` workflow can be used to create project
[snapshots](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)
(commits), branches, etc. that encompass all the things (source code, data, and
models):

```dvc
$ git add data.dvc data.yaml ... training.py
$ git commit -m "First modeling experiment"
$ git tag -a v1 -m "Model v1"
```

DVC uses the metadata we put in Git as a proxy to the actual data
([stored separately](/doc/command-reference/cache)), and can rewind ⏪ or
fast-forward ⏩ the entire project based on it (see `dvc checkout`).

![](/img/versioning.png) _Full project restoration_

> For more hands-on experience, please follow the
> [versioning tutorial](/doc/use-cases/versioning-data-and-model-files/tutorial).

In summary, data science and machine learning are iterative processes where the
lifecycles of data, code, and ML models occur at different paces. DVC's unique
_data-as-code_ approach helps integrate and manage them effectively.
