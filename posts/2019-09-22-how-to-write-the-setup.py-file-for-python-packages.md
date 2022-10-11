---
aliases:
- /Programming/2019/09/22/how-to-write-the-setup.py-file-for-python-packages
author: Ziyue Li
categories:
- Programming
date: '2019-09-22'
description: An example and explanation of some minimum things to configure in the
  setup.py
hide: false
image: images/posts/setup_python.png
layout: post
search_exclude: false
summary: The documentation for the setup.py file is not very clear. And I think the
  quickest way to learn this is through an example, so here is an example of some
  minimum things to configure in the setup.py.
tags:
- python
title: How to Write the setup.py File for Python Packages
toc: true

---

## Basic Setup

The documentation for the setup.py file is not very clear, and I think the quickest way to learn this is through an example, so here is an example of some of the minimum things to configure in the setup.py:

If your folder structure is the following (see [here](https://blog.ionelmc.ro/2014/05/25/python-packaging/#id13) for why this structure is preferred):

```plain
- Project_folder
- src
- PACKAGE_NAME
      - __init__.py
      - data
      - ...
- test
- setup.py
```

we can use the following setup.py.

```python
from setuptools import setup, find_packages

setup(name=PACKAGE_NAME,
      maintainer=AUTHOR,
      version=PACKAGE_VERSION,
      install_requires=['pandas', 'numpy'],
      package_dir={'': 'src'},
      packages=find_packages('src'),
      package_data={PACKAGE_NAME: ['data/*']},
      include_package_data=True,
      description=DESCRIPTION,
      python_requires='>=3.6')
```

- The name, maintainer, and version are self-explanatory.

- `install_requires` specifies the dependent python packages so that when installing your package, if some of these packages are not already installed in the user's system, they will be downloaded and installed. One can [specify version requirements for each package here too](https://packaging.python.org/discussions/install-requires-vs-requirements/).

- `packages` specifies all packages to include[^1].
  They should be specified in the form of `[foo, foo.bar, foo.tar, foo.bar.tool, ...]`, where foo.bar, foo.tar and foo.bar.tool are sub-packages. All sub-packages need to be listed explicitly.

  If you put all packages under one folder, you can use the find_packages function to automatically find all packages under that folder.

- `package_dir` specifies where your package resides. The keys to this dictionary are package names, and an empty package name stands for the root package. In the above example, your package directly resides below the `src` folder.

  If we have `package_dir={'foo': 'lib'}`, then it means the package foo is the folder lib. If we then have `packages = ['foo', 'foo.bar']`, Distutils will be looking for `lib/__init__.py` and `lib/bar/__init__.py` for the foo and foo.bar packages respectively.


- `package_data` specifies data files to be included for each package. The key is the package name and the value is a list of paths to the data files that relative to the directory containing the package (information from the package_dir mapping is used if appropriate).


## Why do we choose the "src" structure?
1. We could've put our package directly under the project folder:

   ```plain
   - Project_folder
   - PACKAGE_NAME
         - __init__.py
         - data
         - ...
   - test
   - setup.py
   ```

   But then when we write our tests, the local package with the same PACKAGE_NAME will be imported instead of the installed package in our system.

2. We could've put the content of our package directly under the `src` folder instead of having another folder with our package name:

   ```plain
   - Project_folder
     - src
       - __init__.py
       - data
       - ...
     - test
     - setup.py
   ```

   This way, when we import PACKAGE_NAME, it won't import the local package since they are under the folder named `src`.
   However, if we had done so, we would not only need to specify that `package_dir={PACKAGE_NAME: 'src'}`, but because the package name wouldn't be picked up by the find_packages function, we would also need to manually list all packages in `packages`.

3. We could've put the test inside the package:

   ```plain
   - Project_folder
     - src
       - __init__.py
       - data
       - test
       - ...
     - setup.py
   ```

   But then we would be distributing the test as part of the package, which is not ideal.

In comparison, the structure we mentioned in the very beginning avoids all these problems and is therefore what I consider the best solution I've found so far.


[^1]: see here for [discussions on the difference between packages and modules](https://stackoverflow.com/questions/7948494/whats-the-difference-between-a-python-module-and-a-python-package).
