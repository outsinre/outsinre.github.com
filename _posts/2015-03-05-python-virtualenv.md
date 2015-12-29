---
layout: post
title: Python Virtualenv
---

> To create isolated Python development environment, each for a specific project.

1. Install *virtualenv*.

    ```bash
    # emerge -av dev-python/virtualenv
    ```
    `dev-python/pip` will also be emerged as a dependency.
2. **NEVER** use `pip` to install Python packages for system-wide usage. Otherwise, Gentoo portage might be ruined or tangled with those packages, making system difficult to maintain.

    For example, you `pip install foo` and forgot installing it through `pip`. Later on, you decide to uninstall it, and use `emerge -avc foo`. You can imagine how horrible/bad is that! *emerge* knows nothing about how `pip` installed *foo*, let alone *equery* tools. 

    Use `pip` within *virtualenv*; use `emerge` for real system.
3. Create *virtualenv*.

    Switch to normal user account:

    ```bash
    $ cd ~
    $ virtualenv --help
    $ virtualenv -p python2.7 --system-site-packages -v ~/workspace/python/venv-test
    $ ls workspace/python/venv-test/
    ```
    A new directory named *venv-test* under *~/workspace/python/* is created. The output will be like this:

    ```
    zachary@zhtux ~ $ virtualenv -p python2.7 --system-site-packages -v workspace/python/venv-test
    Already using interpreter /usr/bin/python2.7
    Creating workspace/python/venv-test/lib/python2.7
    Symlinking Python bootstrap modules
      Symlinking workspace/python/venv-test/lib/python2.7/config
      Symlinking workspace/python/venv-test/lib/python2.7/lib-dynload
      Symlinking workspace/python/venv-test/lib64/python2.7/os.py
      Ignoring built-in bootstrap module: posix
      Symlinking workspace/python/venv-test/lib64/python2.7/posixpath.py
      Cannot import bootstrap module: nt
      Symlinking workspace/python/venv-test/lib64/python2.7/ntpath.py
      Symlinking workspace/python/venv-test/lib64/python2.7/genericpath.py
      Symlinking workspace/python/venv-test/lib64/python2.7/fnmatch.py
      Symlinking workspace/python/venv-test/lib64/python2.7/locale.py
      Symlinking workspace/python/venv-test/lib64/python2.7/encodings
      Symlinking workspace/python/venv-test/lib64/python2.7/codecs.py
      Symlinking workspace/python/venv-test/lib64/python2.7/stat.py
      Symlinking workspace/python/venv-test/lib64/python2.7/UserDict.py
      Symlinking workspace/python/venv-test/lib64/python2.7/copy_reg.py
      Symlinking workspace/python/venv-test/lib64/python2.7/types.py
      Symlinking workspace/python/venv-test/lib64/python2.7/re.py
      Symlinking workspace/python/venv-test/lib64/python2.7/sre.py
      Symlinking workspace/python/venv-test/lib64/python2.7/sre_parse.py
      Symlinking workspace/python/venv-test/lib64/python2.7/sre_constants.py
      Symlinking workspace/python/venv-test/lib64/python2.7/sre_compile.py
      Symlinking workspace/python/venv-test/lib64/python2.7/warnings.py
      Symlinking workspace/python/venv-test/lib64/python2.7/linecache.py
      Symlinking workspace/python/venv-test/lib64/python2.7/_abcoll.py
      Symlinking workspace/python/venv-test/lib64/python2.7/abc.py
      Symlinking workspace/python/venv-test/lib64/python2.7/_weakrefset.py
    Creating workspace/python/venv-test/lib/python2.7/site-packages
    Writing workspace/python/venv-test/lib64/python2.7/site.py
    Writing workspace/python/venv-test/lib64/python2.7/orig-prefix.txt
    Creating parent directories for workspace/python/venv-test/include
    Symlinking workspace/python/venv-test/include/python2.7
    Creating workspace/python/venv-test/bin
    New python executable in workspace/python/venv-test/bin/python2.7
    Changed mode of workspace/python/venv-test/bin/python2.7 to 0755
    Also creating executable in workspace/python/venv-test/bin/python
    Changed mode of workspace/python/venv-test/bin/python to 0755
    Testing executable with workspace/python/venv-test/bin/python2.7 -c "import sys;out=sys.stdout;getattr(out, "buffer", out).write(sys.prefix.encode("utf-8"))"
    Got sys.prefix result: u'/home/zachary/workspace/python/venv-test'
    Creating workspace/python/venv-test/lib64/python2.7/distutils
    Writing workspace/python/venv-test/lib64/python2.7/distutils/__init__.py
    Writing workspace/python/venv-test/lib64/python2.7/distutils/distutils.cfg
    Installing setuptools, pip, wheel...
      Ignoring indexes: https://pypi.python.org/simple
      Collecting setuptools
      Collecting pip
      Collecting wheel
      Installing collected packages: setuptools, pip, wheel
      Successfully installed pip-7.1.2 setuptools-18.2 wheel-0.24.0
    ...Installing setuptools, pip, wheel...done.
    Writing workspace/python/venv-test/bin/activate
    Writing workspace/python/venv-test/bin/activate.fish
    Writing workspace/python/venv-test/bin/activate_this.py
    Writing workspace/python/venv-test/bin/activate.csh
    ```
    A new `python` executable is **copied** from system */usr/bin/python* to *~/workspace/venv-test/bin/python*, while `pip`, `setuptools`, `wheel` etc. executables are **installed** there as well.

    We can add many arguments to `virtualenv` command to change default behavior. For example `-p python3.4` to specify our desired Python version instead of system's default */usr/bin/python*. By default, global Python packages (i.e. */usr/lib/python2.7/site-packages/*) are beyond of *virtualenv*, `--system-site-packages` argument allows *virtualenv* to access those global site-packages.
3. Activate

    ```bash
    $ source workspace/python/venv-test/bin/activate
    $ pwd
    $ python --version
    $ which python pip
    ```
    Activation of *virtualenv* does NOT change working directory. That is to say, you are not constrained within that newly created directory. Newly created *venv-test* directory is just for holding *virtualenv* files and executables.

    After activation, the created virtual environment name *venv-test* will be appended to the `bash` command prompt: `(venv-test)zachary@zhtux ~ $`. This indicates and reminds that you're in *virtualenv* instead of system Python environment.
4. `pip install`

    As mentioned, we should not install system-wide packages by `pip install xxx`. However, it can be used in *virtualenv*. Pakcages installed under *virtualenv* environment are bound to and constrained in that environment only!

    Installed packages will be located under *~/workspace/python/venv-test/lib/python2.7/site-packages/* instead of the system-wide site-packages director (i.e. */usr/lib/python2.7/site-packages/*).
5. `pip freeze`

    Say, you need to clone the current *virtualenv* for another project. So you have to create a new one and install the same set of packages with the same versions.

    `pip freeze > requirement.txt` will  create a requirements.txt file, which contains a simple list of all the packages in the current environment, and their respective versions.  You can also use `pip list > requirement.txt`, but the output format is a little different.

    Later on, when re-creating a new *virtualenv*, just use `pip install -r requirements.txt` to install the same set of packages. This can help ensure consistency across installations, across deployments, and across developers.

    Attention: if `--system-site-packages` was added, `pip freeze` and `pip list` include system-wide site-packages. But don't worry, `pip install -r requirements.txt` won't pull in those packages into *virtualenv* as long as we keep consistent on argument `--system-site-packages`.
6. Lastly, remember to exclude the virtual environment folder *venv-test* from source control by adding it to the ignore list.
7. Deactivate

    ```bash
    $ deactivate
    ```
8. Delete

    ```bash
    $ rm -r workspace/python/venv-test
    ```
    Just remove the created directory!
9. *virtualenvwrapper* is an alternative to *virtualenv* providing a set of commands which makes working with virtual environments much more pleasant and easier. It also places all your virtual environments in one place.
10. Details refer to:
    1. [virtualenv](https://virtualenv.pypa.io/en/latest/)
    2. [virtual environment](http://docs.python-guide.org/en/latest/dev/virtualenvs/)
    3. [gentoo python pip and virtualenv](http://blog.samuelololol.org/2013/10/how-python-pip-and-virtualenv-go-along.html)
    4. [Virtualenv Tutorial Part 2](http://www.simononsoftware.com/virtualenv-tutorial-part-2/)
    5. [A Primer on virtualenv](http://iamzed.com/2009/05/07/a-primer-on-virtualenv/)
    6. [a useful video](http://showmedo.com/videotutorials/video?name=2910000&fromSeriesID=291)
