# Introduction to conda environments

### Description

This short workshop aims to familiarize participants with `conda` environments. For many new and existing Python programmers, Anaconda/Miniconda has become the default way to install the language and manage third-party libraries. While the graphical user interface (Navigator) offers plenty of features, it pays off to learn to use the command-line, specially for those working with version control systems, such as git, or in CLI environments, like compute clusters.

### What to expect from the workshop?

Ideally, you will leave this workshop loving `conda` environments. More seriously, our main goal is to demonstrate the advantages of using different `conda` environments for separate projects. In addition, by the end of the workshop you will be able to install, upgrade, and remove packages using the command-line interface of `conda`.

### What should I know already?

There are no hard requirements to follow this workshop. It helps if you are familiar with command-line interfaces, but we will explain all the commands as they are introduced.

### Software Requirements

To follow this workshop, please install any version of [Anaconda](https://www.anaconda.com/products/individual) or [Miniconda](https://docs.conda.io/en/latest/miniconda.html). We _highly_ recommend installing one that supports Python 3.x, as Python 2 is no longer supported since January 2020. When prompted to let the installer initialize `conda`, click on yes. This setting will automatically enable `conda` on all terminal sessions by default.

We will also make use of the terminal. Terminal choices depend on your operative system. Mac OS and Linux users are lucky in that they already have one pre-installed. On Mac OS, you might want to look at [iTerm2](https://www.iterm2.com/) for a nicer terminal experience, but the default Terminal application is good enough for this tutorial. For Windows users, you can simply use the Anaconda Prompt.

# Ready? Let's go!

## What are conda environments?

After installing Anaconda or Miniconda and opening a terminal window, you should be greeted with a prompt that looks like this, minus the user and host names:

```bash
(base) joaor@desktop $
```

This prefix - `(base)` - is the way `conda`  tells us that we are in the _base_ environment. So what's an environment? An environment is a folder that contains a specific set of packages you have installed. In other words, an environment is an isolated installation of Python with some extra libraries installed. Since scientific packages often depend on very specific version of certain libraries, using dedicated environments helps minimize clashes between these dependencies. More importantly, dedicated environments makes your life easier if installing a package breaks ten others - worst case scenario, you just delete and recreate the environment. Without environments you'd have to reinstall Anaconda/Miniconda. So, as a rule of thumb, you __never__ want to install packages in the `base` environment unless you have a really good reason to do so.

The major downside of using environments is that you will need some additional disk space. Each environment should include its own Python interpreter, so if you have twenty environments, you will have twenty Python installations somewhere on your hard drive. Thankfully, computers today come with several hundred GBs of disk space, or even TBs, so this is really _not_ an issue in the vast majority of cases.

## Creating a new conda environment

Now that we understand what `conda` environments are, let's create our first. Type the following in your terminal:

```bash
(base) joaor@desktop $ conda create --name myenv python=3.8
```

This command will create a new environment called `myenv` with Python 3.8 as its default interpreter. Importantly, environment names cannot have spaces. The `python=3.8` part of the command instructs `conda` to install the python package, at version 3.8. This version specification is particularly useful. If you'd want to run a library that is only supported in Python 2.7, you would create an environment with `python=2.7`. There are plenty more options that you can pass to `conda create`, which you can find out about with the option `--help`.

Depending on your internet connection, `conda` might take a while to collect the necessary information to run the command. Sooner or later, you will be shown a list of packages that will be downloaded and installed, followed by a prompt asking you if you want to proceed. Type `y` and `conda` will download and install those packages, and setup your environment. When you repeat this operation later on, `conda` will be smart about which packages it needs to download. If you previously downloaded a version of a package you need, it will use that cached version, saving you the time (and trouble) to download it from a remote server.

Once the command finishes, notice that you remain on the `(base)` environment. Creating an environment does not automatically trigger a change in environment. To move to - or to activate - the `myenv` environment, you need to use the `conda activate` command, as follows:

```bash
(base) joaor@desktop $ conda activate myenv
(myenv) joaor@desktop $
```

The prompt prefix changes to indicate the environment we are in changed. If you were to run `python`, it would launch the interpreter stored in the `myenv` environment folder. If you wanted to return to the `base` environment, you would issue the command `conda deactivate`. As of now, every Python script runs through this version of the interpreter, having access only to the modules installed on this environment. So let's go ahead and install some packages.

## Installing packages via conda

Installing packages in an environment requires a bit information about `conda` as a package manager. Packages in the `conda` cloud are distributed through channels. Default installations, like yours, have access to the `defaults` channel which contains various common modules for scientific Python, such as `numpy`. To install a package, you use the following command:

```bash
(myenv) joaor@desktop $ conda install numpy
```

The resulting dialogue is similar to that when we first created the environment, and for a good reason: the process is essentially the same. When we created the environment, we included `python=3.8`, which is itself a package. More specifically, we asked for version 3.8 of the `python` package. Similarly, we could have asked for a _specific_ version of `numpy` using the same syntax, e.g. `numpy=1.15`. The `conda` manager will then compute whatever dependencies it needs to download and install to get the package we requested working, or die trying. Sometimes, packages have conflicting dependencies. For example, version 1.15 of `numpy` might ask for a particular version of `mkl` which is incompatible with the version of `scipy` you have installed. In this case, `conda` will prompt if you want to `downgrade` packages, if it found a way to solve the dependencies constraints. Otherwise, it might just quit with an error. Again, the good thing with having separate environments.

To install a package that is _not_ on the `defaults` channel, we have to specify which channel we want to search in. Let's try and install `biopython`, which is only available through the `conda-forge` channel:

```bash
(myenv) joaor@desktop $ conda install -c conda-forge biopython
```

You can verify that `bioptyhon` is indeed installed by trying to import it in your environment:

```bash
(myenv) joaor@desktop $ python -c 'import Bio; print(Bio.__version__)'
1.76
```

If you try and run this same command on the `base` environment, it will give an error because `biopython` is not installed there:

```bash
(myenv) joaor@desktop $ conda activate base
(base) joaor@desktop $ python -c 'import Bio; print(Bio.__version__)'
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'Bio'
```

Let us go back to our `myenv` environment before proceeding:

```bash
(base) joaor@desktop $ conda activate myenv
```

In addition to specifying channels directly when you install a package, you can also permanently add a channel to your `conda` configuration. You can either do this globally, affecting all environments, or per environment. We recommend the latter, as it is less likely to surprise you in the future. Let's add `conda-forge` as a permanent channel for our `myenv` environment.

```bash
(myenv) joaor@desktop $ conda config --env --add channels conda-forge
```

By default, using `--add` adds the specified channel at the top of the channel priority list. In short, this means that `conda` will look there first to see if a package is available. If you would rather add a channel at the bottom of the priority list, for instance if you only want to use that channel for a very specific package, then replace `--add` by `--append` in the command above. There are other, more advanced, ways of specifying channel priorities that go beyond the scope of the introduction. For more information, read [the documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-channels.html).

## Removing packages

Installing packages is useful, but so is removing them. To remove a package, e.g. `biopython`, you can run the following command:

```bash
(myenv) joaor@desktop $ conda remove biopython
```

Note that this removes that package alone, not its dependencies. Because of how the dependency solving process works, it is safer to let the user remove packages manually.

## Exporting an environment

The next command we will cover in this workshop lets us export the configuration of an environment to a file, so that we can share it with others. Instead of bundling the packages themselves, `conda` exports a list of the package names and their versions, as we have them on our system. In addition to package details, the file contains also the list of all channels we defined in our configuration, both globally and environment-specific. Finally, the file is written in `YAML`, a human-readable text format that we can inspect manually and edit if necessary. Let's export the `myenv` environment to a file:

```bash
(myenv) joaor@desktop $ conda env export --no-builds --file myenv.yaml
```

There are some options to unpack in this command. First, we do not use `conda` but the `conda env` subcommand, which is a more advanced script to manage environments. We also pass a `--no-builds` option, which tells `conda` to specify only the package names and versions. By default, `conda` would have exported the build information for each package, which you can think of as a very precise version that is sometimes specific to the operative system. While this is great for reproducibility, it is very likely that you will end up not being able to share your environment across different systems. 

Finally, it is recommended to open the resulting `yaml` file and keep only the packages you installed yourself. For example, we only installed `python` and `numpy`, so we would want to remove all but these two entries from the `yaml` file. Why? First, because these would be very likely enough to reproduce our results (to the numerical precision of the computers). And second, because the `conda` of whoever tries to replicate our environment from our file would solve dependencies on its own, as it sees best, without running into compatibility issues caused by machine-dependent package versions/settings. We might also want to remove the `prefix` line, although there are no real consequences if you do not. As a result, our `yaml` file would look like this:

```yaml
name: myenv
channels:
  - defaults
  - https://repo.anaconda.com/pkgs/main
  - https://repo.anaconda.com/pkgs/free
dependencies:
  - numpy=1.18.1
  - python=3.8.2
```

## Creating an environment from a YAML file

Now let's do the reverse operation and create an environment from a `yaml` file. You will find these files often in GitHub repositories, so it is handy to know how to use them. Let's open a text editor and make some changes to our `myenv.yaml` file, so that it looks like this:

```yaml
name: myenv
channels:
  - defaults
  - https://repo.anaconda.com/pkgs/main
  - https://repo.anaconda.com/pkgs/free
dependencies:
  - numpy
  - python=2.7
```

Then, run the following command on your terminal:

```bash
(myenv) joaor@desktop $ conda env create --name myenv_py27 --file myenv.yaml
```

The `--name` option defines the name of the environment we will create and overrides the setting on the `yaml` file. As you will see in the command's output, `conda` will fetch all the necessary dependencies to include `numpy` in our environment, which is based on `python` version 2.7.

## Removing environments

Finally, let's see how we remove environments. Removing environments is useful when you make mistakes and environments become unusable, or just because you finished a project and you need to clear some disk space. The command to remove an environment is the following:

```bash
(myenv) joaor@desktop $ conda env remove --name myenv_py27
```

Running this command will delete all packages installed in that environment and remove the environment folder. Naturally, you cannot delete the environment you are on - you must first use `conda deactivate` or move to another environment.

## The End

And that's it. There is much more to learn about `conda`, so go ahead and browse the [documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/index.html) for more information. However, these few commands we learned are sufficient to get you started and to help you manage your Python projects a tiny bit more efficiently.

Let us know if you have any questions or feedback, you can reach me at `joaor@stanford.edu`. Thank you for following along and good luck with your `conda` and Python adventures!

---

Content licensed under CC-BY-4.0. For details, see the LICENSE file on the repository.

Last updated: 20 May 2020
