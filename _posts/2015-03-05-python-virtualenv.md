---
layout: post
title: Python Virtualenv
---

1. toc
{:toc}

To create isolated Python development environment, each for a specific project.

1. Install *virtualenv*.

   ```bash
   # emerge -avt dev-python/virtualenv
   ```
   
   *dev-python/pip* will be installed as a dependency.
2. **NEVER** use *pip* to install system-wide packages, which may conflict with Gentoo portage, making the system difficult to maintain.

   Suppose you `pip install foo` and forget installing it through *pip*. Later on, you decide to uninstall it by `emerge -avc foo`. You can imagine how horrible that is! Portage *emerge* knows nothing about how *pip* installed the *foo* package, let alone *equery*.

   Use *pip* within virtual environment and use *emerge* for system-wide packages.
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
   tux@zhtux ~ $ virtualenv -p python2.7 --system-site-packages -v workspace/python/venv-test
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
   Got sys.prefix result: u'/home/tux/workspace/python/venv-test'
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
   
   A new *python* executable is *copied* from system */usr/bin/python* to *~/workspace/venv-test/bin/python*, while *pip*, *setuptools*, *wheel* etc. executables are *installed* there as well.

   We can supply many arguments to *virtualenv* command to change its default behavior. For example `-p python3.4` specifies the desired Python version instead of */usr/bin/python*. By default, global Python packages (i.e. */usr/lib/python2.7/site-packages/*) are beyond *virtualenv*. The `--system-site-packages` argument fix this issue.
3. Activate

   ```bash
   $ source workspace/python/venv-test/bin/activate
   $ pwd
   $ python --version
   $ which python pip
   ```

   Activation of *virtualenv* does *not* change the current working directory. That is to say, you are not constrained to that newly created directory. The new *venv-test* directory just holds virtual environment files.

   After activation, the created virtual environment name *venv-test* will be appended to *bash* prompt like `(venv-test)tux@zhtux ~ $`. This indicates and reminds that we are successfully switched to a virtual environment.
4. pip install

   As mentioned, we should not install system-wide packages by `pip install foobar`.

   Pakcages installed within *virtualenv* environment are bound to and only meaningful for the very environment! Those packages are placed under *~/workspace/python/venv-test/lib/python2.7/site-packages/*.
5. pip freeze

   Say, you want to creat a new project, and need to clone the current *virtualenv*. The *freeze* command helps.

   `pip freeze > requirement.txt` will  create file *requirements.txt* that contains a full list of packages within current virtual environment, along with their respective versions.  You can also use `pip list > requirement.txt`, but the output format is a little different.

   Within the new virtual environment, try `pip install -r requirements.txt` to install the same set of packages. This can help ensure consistency across installations, across deployments, and across developers.

   **attention**: if `--system-site-packages` was added, `pip freeze/list` includes system-wide Python packages. But don't worry, `pip install -r requirements.txt` won't pull in those packages as long as we stick to `--system-site-packages`.
6. Lastly, remember to exclude the virtual environment directory (i.e. *venv-test*) from source control system (i.e. add to *.gitignore).
7. Deactivate

   ```bash
   $ deactivate
   ```
   
8. Delete

   ```bash
   $ rm -r workspace/python/venv-test
   ```
   
   Just remove the created directory!
9. Run script without actication

   It's easy to run script of *virtualenv* without activating.

   ```bash
   $ /full/path/to/virtualenv/bin/python foobar.py
   # -or-
   $ /full/path/to/virtualenv/bin/foobar.py
   ```

   Use *full path* to *python* or the script within *virtualenv*, Python will add */full/path/to/virtualenv/lib/python2.7/site-packages/* to *sys.path* automatically. It is handy if we want to invoke the script from GUI. It also helps if we don't bother activating virtualenv each time.
1. As the word implies, *virtualenvwrapper* is a wrapper to *virtualenv* and helps manage multiple virtual environments. Especially, it places all your virtual environments in one place.
2. Emacs [pyvenv](https://github.com/jorgenschaefer/pyvenv) supports virtual environment. If you installed [elpy](https://github.com/jorgenschaefer/elpy), then it is probably brought in.

   To activate a virtual environment with Emacs, just `M-x: pyvenv-activate` and type the relevant directory.
3. Details
   1. [virtualenv](https://virtualenv.pypa.io/en/latest/)
   2. [virtual environment](http://docs.python-guide.org/en/latest/dev/virtualenvs/)
   3. [gentoo python pip and virtualenv](http://blog.samuelololol.org/2013/10/how-python-pip-and-virtualenv-go-along.html)
   4. [Virtualenv Tutorial Part 2](http://www.simononsoftware.com/virtualenv-tutorial-part-2/)
   5. [A Primer on virtualenv](http://iamzed.com/2009/05/07/a-primer-on-virtualenv/)
   6. [a useful video](http://showmedo.com/videotutorials/video?name=2910000&fromSeriesID=291)
