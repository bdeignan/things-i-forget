## Sources
Read:
https://discuss.python.org/t/specification-of-editable-installation/1564
https://snarky.ca/what-the-heck-is-pyproject-toml/
https://packaging.python.org/guides/distributing-packages-using-setuptools/
https://realpython.com/python-wheels/#telling-pip-what-to-download
https://martin-thoma.com/pyproject-toml/
3-part series:
https://www.bernat.tech/pep-517-and-python-packaging/
https://www.bernat.tech/pep-517-518/
https://www.bernat.tech/growing-pain/
Not read:
https://packaging.python.org/tutorials/packaging-projects/
https://realpython.com/courses/python-modules-packages/
https://realpython.com/courses/how-to-publish-your-own-python-package-pypi/

Very different from conda: Python Packaging Authorities systems' pip and setuptools

This post has it all, may just want to refer to it and then show what works (for now) https://stefanoborini.com/current-status-of-python-packaging/


Say you have this simple project:
```bash
pugs-project
├── README.md
├── setup.cfg
├── setup.py
├── LICENSE.txt
├── my_lib
│   ├── __init__.py
│   └── logic.py
├── tests
│   ├── test_init.py
│   └── test_logic.py
├── tox.ini
└── azure-pipelines.yml
```

The focus here will be on the **packaging code and metadata**: setupy.py, setup.cfg, LICENSE and README, and I'll just lightly touch on the other stuff.

- setup.py serves two primary functions:
    - It’s the file where various aspects of your project are configured using keyword arguments to the setup() function
    - It’s the command line interface for running various commands that relate to packaging tasks (although I've seen this discouraged in the case of testing). Namely, you can setup the project in editable mode which makes changes to your library immediately available for use. I've seen this ability be especially important to the scientific Python community where you're working on a library and then use it in a data analysis Jupyter notebook, for example.
- setup.cfg is an ini file that contains option defaults for setup.py commands.

The de facto standard was setuptools which uses the setup.py and setup.cfg. But setuptools isn't guaranteed to be on the end user's machine if you ship your library as an "sdist" as opposed to the newer standard "wheel".

- A “source distribution” (sdist) is unbuilt and requires a build step when installed by pip. Even if the distribution is pure Python (i.e. contains no extensions), it still involves a build step to build out the installation metadata from setup.py.
- A wheel is a built package that can be installed without needing to go through the “build” process.

Basically, sdist means that when you run pip install, the user's machine is responsible for building the library and the wheel is when the library developer builds it on their machine. More importantly Wheels cut setup.py execution out of the equation. Installing from a source distribution runs whatever is contained in that project’s setup.py. As pointed out by PEP 427, this amounts to arbitrary code execution. Wheels avoid this altogether.

The last part is important. Reading the PEP's, the Python commuinty is trying to move away from reliance on setuptools (and the older distutils) or at least making it _an_ option as opposed to _the_ option. PEP-518 specified a pyproject.toml file to specify packaging metadata and PEP517 builds on it to allow specification of build backends, of which setuptools is just one now.

So, TL;DR pyproject.toml is trying to become the new standard for packaging up your code for others. It doesn't assume the user has a specific version of setuptools installed, it doesn't require the user to execute an arbitrary setup.py on their machine - which besides a security risk also leads to weird to debug errors according to some.

## Distribute your code as PEP-517/518
#### pyproject.toml
The pyproject.toml file is a solution for defining the build system as a dependency. Also, other kinds of meta-data and install requirements can be defined in it.

So now your project above will look like this:
```bash
packaging_tutorial
├── LICENSE
├── README.md
├── example_pkg
│   └── __init__.py
├── pyproject.toml
├── setup.cfg
├── setup.py
└── tests
```

This is if you're using setuptools. If you're using Poetry, for example, then you don't need the setup.py or setup.cfg.

This causes an issue raised by many that I mentioned above. How to do an editable install without setuptools? pip install -e requires setup.py. People are still discussing a way to do this without setuptools (CITE), but a stanard hasn't been settled upon. Specifically, some people want to: The issue is not about a local development of a package, but about installing the package in the editable mode in another project.

Why is editable mode important to some? If you’re using a non-pure python package (e.g., any thing with a build step like cythonize, extension module compilation, webpack for building web assets), it’s extremely important for efficient development, since this requires that a build be run at some point. And if this build is not editable (i.e., just pip install .), then every single change you make from that point requires another pip install ..

https://github.com/python-poetry/poetry/discussions/1135

So the problem with not using a setup.py (say if you're using Poetry) is the following:
- The current project is installed in development mode within the venv with poetry install by default.
- You can add other poetry and setup.py managed project as dependencies in development mode with poetry add ../relative_path/to/package_folder
- Installing a poetry managed project - or better: any project managed by pyproject.toml instead of setup.py - in development mode by pip, is currently not possible. But that's on pip's side and not on poetry's. See also: pypa/pip#6434 https://github.com/pypa/pip/issues/6434

So, if you're making libA with poetry, and then you have another project, call it libB that is built in a conda environment (or any tool besides Poetry), then "pip install -e libA" in libB's environment won't work apparently.

What is _not_ a problem is in libA, you launch a Jupyter notebook (poertry run jupyter notebook) and want updates to libA's modules and packages picked up in the notebook immediately. This is not a problem and doesn't reuquore pip install -e.

So, if pip ever gets around to not relying on setuptools to make a package install available in editable mode, all of this conversation will be moot. In the meantime, if you're using poetry you can make a setup.py with the following work around:
```bash
poetry build --format sdist
tar -xvf dist/*-`poetry version -s`.tar.gz -O '*/setup.py' > setup.py
```


Cross-posted from here, for better visibility.)

tl;dr I think we need editable installs (or something of similar simplicity) in order to ease the entry pathway for novice Python package developers.

Long version Something I don’t think I’ve seen in the discussion so far is this: I think it’s also important to keep in mind that editable installs represent by far one of the easiest, most-minimal-config approaches for beginning package developers to get their work under reliable tests. @pfmoore’s comment https://github.com/pypa/packaging.python.org/issues/320#issuecomment-487309079 about running everything through tox, including local dev testing, got me thinking about how I might rework my project setup along those lines… but properly configuring tox to cleanly handle even just one environment for a local test suite is not trivial, and I figure it’s not going to be obvious to a new package developer why the best practice is wrapping the primary testing tool with tox. pip install -e . followed by pytest is simple, quick, and obvious, even if it’s not the most robust approach.

So, I think that editable install support should be eliminated only if another alternative is identified that is comparably simple for new developers. (It seems like the consensus is moving toward retaining editable install support, so this extra argument may be somewhat superfluous, but I wanted to make sure it was made regardless

always do python -m pip install https://snarky.ca/why-you-should-use-python-m-pip

This generally agrees with what I've read:
> As far as I can tell (and I haven't re-read everything in a while), how to do editable install with pip and pepe517 is not quite certain yet. There are complication depending of what "editable install" requires (pure python vs do you need to compile code); so the current state is the status quo.

pip is trying verry hard to control how files are going from the source tarball to within the site-directory. Which editable install are really trying to avoid. The the concept of editable install via pip is kind-of weird.

You don't have to get back to setup.py, you just can't do editable install with pip install -e .. For example, when I use flit for my packages I know I can do flit install --symlink for editable install. I'm guessing you can do the same by using --develop with poetry, but I may be wrong; you will just have to invoke poetry directly instead of pip.

The packaging ecosystem in Python is slow to adapt because of backward compat; but it is getting there. Keep in mind there is a current discussion to update the official packaging guide to stop telling people to call setup.py to install stuff.

So don't be in any hurry; things are getting better; slowly; you are on the bleeding edge so yes seem are rarely obvious. Next progress will be done when someone sheperd a pep and likely the code change to support editable install with pyproject.toml; it may just take some time.
Source: https://www.reddit.com/r/Python/comments/cjcanc/could_someone_shed_a_light_on_whats_up_with_pip/

Not packaging, but dev, here's alot of ways to maintain dev env https://dev.to/bowmanjd/python-tools-for-managing-virtual-environments-3bko

More social proof for poetry https://news.ycombinator.com/item?id=25253236 and https://www.reddit.com/r/Python/comments/l7qase/virutualenvs_which_one_in_2021_venv_virtualenv_or/
TRUE: Pip + setuptools is "it's working, no need to change" in the same way that "SVN works so why bother with Git?". It seems fine until you try the alternative, and then the thing it replaces just feels painful to use.

Setup.py can lead to shit when making sdists: Truth is: as long as there is an executable setup.py file in the sdist, then the resulting metadata can not be guaranteed until after it is indeed executed. We are talking about setuptools here. Other build backends (such as poetry) do not directly rely on an executable script, so things are much more static in the sdist.


More on pyproject.toml and pep518
https://github.com/carlosperate/awesome-pyproject/


