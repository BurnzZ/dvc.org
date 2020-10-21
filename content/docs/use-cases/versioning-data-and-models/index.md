# Versioning Data and Models

Taking data science from conceptual to applied requires answering data
management questions around dataset and machine learning model versioning. How
to track the evolution of data and ML models? How to organize and store the
revisions of these data artifacts for easy access and sharing?

It turns out that we can find some answers in software engineering, like
[version control](https://en.wikipedia.org/wiki/Version_control). This means
adopting a standard file storage structure (cache) and keeping a change history
(commits), as well as further benefits like enabling parallel teamwork
(branching), peer-reviews (pull requests), etc. Imagine if we could do the same
with data!

![](/img/404) _Data as code_

Some advantages:

- Manage and share multiple versions of datasets and ML models easily,
  maintaining visibility over them.
- Codifying <abbr>data artifacts</abbr> (and processes) enables proven Git
  workflows such as branching, pull requests, release management, and even CI/CD
  for your data lifecycle.
- Reproducibility: Guarantee that anyone can rewind data pipelines exactly as
  they were created originally.
- Simple CLI: work with simple terminal commands like `dvc add` or
  `dvc checkout` (similar to familiar `git` commands).
- Security: [DVC metafiles](/doc/user-guide/dvc-files-and-directories) enable
  auditing data changes. And data access controls can be setup via storage
  integrations.

## Why DVC

Unfortunately, SCM tools like [Git](https://git-scm.com/) are designed to handle
small text files, so data scientists can only control part of their assets that
way. Storage itself is also severely limited by code hosting services
[like GitHub](https://docs.github.com/en/github/managing-large-files/what-is-my-disk-quota),
so transferring and managing data storage separately is a constant hurdle.

Rather than reinventing the wheel, DVC proposes to treat **data as code** in
order to manage it with existing engineering tools and best practices.

DVC enables [tracking](#how-it-looks) large datasets and other <abbr>data
artifacts</abbr> in Git by replacing them with small, human-readable
[metafiles](/doc/user-guide/dvc-files-and-directories). The data itself is
<abbr>cached</abbr> outside of Git (locally or externally), and can be
synchronized automatically with [dedicated storage](#versioned-storage). More
advanced features of DVC build upon this foundation.

> 💡 Please see [Get Started](/doc/start) for a primer on DVC's features.

## How it looks

It's easy to revisit previous versions of the code with Git, but the data also
changes along the way. With DVC, we can achieve true
[reproducibility](/doc/start/data-pipelines) by matching code commits with
snapshots of the data used at the time. DVC [commands](/doc/command-reference)
and metafiles aim to make this intuitive.

### Basics of tracking and caching data

The following project directory contains a couple of large data files in
subdirectories of `data/`, along with some source code to process it:

```dvc
.
├── data
│   ├── labels.csv        # Many MB of labeled data here
│   └── transactions.csv  # Several GB of raw historic data
├── training.py
```

The `dvc add` command captures the essence of existing files or directories in
the <abbr>workspace</abbr> as `.dvc` files, tiny placeholders for the data that
can be put in Git. Similarly, `dvc.lock` files can also refer to any data
<abbr>output</abbr> by running the project's code (see `dvc repro` and
`dvc run`). All tracked data contents are saved to the <abbr>DVC cache</abbr>,
and linked\* back to their original location:

> \* See
> [Large Dataset Optimization](/doc/user-guide/large-dataset-optimization) and
> `dvc config cache` for more info. on file linking.

```git
 .
+├── .dvc     # Hidden DVC internals
+│   ├── cache
+│   │   ├── b6...  # data/ contents moved here
+│   │   └── ed...  # model.pkl moved here
+│   ...
 ├── data           # Now a link to the cache
 │   ├── labels.csv
 │   └── transactions.csv
 ...
+├── data.dvc   # Points to data/
+├── dvc.lock   # Points to model.h5
+├── dvc.yaml   # Wrapper for running training.py
+├── model.h5   # Final result (also a link to the cache)
 ├── training.py
```

> See also `dvc.yaml`.

The `data.dvc` and `dvc.lock` metafiles connect workspace and cache. Let's see
their contents:

```yaml
outs:
  - md5: b6e29fb8740486c7e64a240e45505e41.dir
    path: data
```

```yaml
stages:
  train_model:
    cmd: python training.py data/ model.h5
    deps:
      - path: data
        md5: b6e29fb8740486c7e64a240e45505e41.dir
    outs:
      - path: model.h5
        md5: ede2872bedbfe10342bb1c416e2f049f
```

These metafiles, along with your code, can safely be put into a Git repo. The
tracked data in the workspace and the cache are separated by DVC (via
`.gitignore`).

### Version control

DVC doesn't replace Git. Having codified datasets, data pipelines, and their
outputs, use regular `git` commands to create versions (commits, tags, branches,
etc.):

```dvc
$ git add cleanup.sh
$ dvc add data/
...
$ git add data.dvc data/.gitignore

$ git commit -m 'Data cleanup v1.0'
```

Now that Git is tracking the code (including
[DVC metafiles](/doc/user-guide/dvc-files-and-directories)), and DVC is tracking
the data, we can repeat the procedure to generate more commits. Use
`dvc checkout` to go back:

```dvc
$ git checkout v1.0
$ dvc checkout
M       data\raw
```

![](/img/versioning.png) _Full project restoration_

> For more hands-on experience, we recommend following the
> [versioning tutorial](/doc/use-cases/versioning-data-and-models/tutorial).

## Versioned storage

What if we could **combine data and ML model versioning features with large file
storage** solutions like traditional hard drives, NAS, or cloud services such as
Amazon S3 and Google Drive? DVC brings together the best of both worlds by
implementing easy synchronization between the data <abbr>cache</abbr> and
on-premises or cloud storage for sharing.

![](/img/model-versioning-diagram.png) _DVC's hybrid versioned storage_

> Note that [remote storage](/doc/command-reference/remote) is optional in DVC:
> no server setup or special services are needed, just the `dvc` command-line
> tool.
