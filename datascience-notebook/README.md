![docker pulls](https://img.shields.io/docker/pulls/jupyter/datascience-notebook.svg) ![docker stars](https://img.shields.io/docker/stars/jupyter/datascience-notebook.svg)

# Jupyter Notebook Data Science Stack

## What it Gives You

* Jupyter Notebook 4.2.x
* Conda Python 3.x and Python 2.7.x environments
* pandas, matplotlib, scipy, seaborn, scikit-learn, scikit-image, sympy, cython, patsy, statsmodel, cloudpickle, dill, numba, bokeh pre-installed
* Conda R v3.3.x and channel
* plyr, devtools, dplyr, ggplot2, tidyr, shiny, rmarkdown, forecast, stringr, rsqlite, reshape2, nycflights13, caret, rcurl, and randomforest pre-installed
* Julia v0.3.x with Gadfly, RDatasets and HDF5 pre-installed
* Unprivileged user `jovyan` (uid=1000, configurable, see options) in group `users` (gid=100) with ownership over `/home/jovyan` and `/opt/conda`
* [tini](https://github.com/krallin/tini) as the container entrypoint and [start-notebook.sh](../minimal-notebook/start-notebook.sh) as the default command
* A [start-singleuser.sh](../minimal-notebook/start-singleuser.sh) script for use as an alternate command that runs a single-user instance of the Notebook server, as required by [JupyterHub](#JupyterHub)
* Options for HTTPS, password auth, and passwordless `sudo`

## Basic Use

The following command starts a container with the Notebook server listening for HTTP connections on port 8888 without authentication configured.

```
docker run -d -p 8888:8888 jupyter/datascience-notebook
```

## Notebook Options

You can pass [Jupyter command line options](http://jupyter.readthedocs.io/en/latest/projects/config.html#command-line-options-for-configuration) through the [`start-notebook.sh` command](https://github.com/jupyter/docker-stacks/blob/master/minimal-notebook/start-notebook.sh#L15) when launching the container. For example, to set a password hashed using `IPython.lib.passwd()` to secure the notebook server:

```
docker run -d -p 8888:8888 jupyter/datascience-notebook start-notebook.sh --NotebookApp.password='sha1:74ba40f8a388:c913541b7ee99d15d5ed31d4226bf7838f83a50e'
```

Or to set the base URL of the notebook server:

```
docker run -d -p 8888:8888 jupyter/datascience-notebook start-notebook.sh --NotebookApp.base_url=/some/path
```

You can sidestep the `start-notebook.sh` script entirely by specifying a command other than `start-notebook.sh`. If you do, the `NB_UID` and `GRANT_SUDO` features documented below will not work. See the Docker Options section for details.

## Docker Options

You may customize the execution of the Docker container and the Notebook server it contains with the following optional arguments.

* `-e PASSWORD="YOURPASS"` - Configures Jupyter Notebook to require the given plain-text password. Should be combined with `USE_HTTPS` on untrusted networks. **Note** that this option is not as secure as passing a pre-hashed password on the command line as shown above.
* `-e USE_HTTPS=yes` - Configures Jupyter Notebook to accept encrypted HTTPS connections. If a `pem` file containing a SSL certificate and key is not provided (see below), the container will generate a self-signed certificate for you.
* `-e NB_UID=1000` - Specify the uid of the `jovyan` user. Useful to mount host volumes with specific file ownership. For this option to take effect, you must run the container with `--user root`. (The `start-notebook.sh` script will `su jovyan` after adjusting the user id.)
* `-e GRANT_SUDO=yes` - Gives the `jovyan` user passwordless `sudo` capability. Useful for installing OS packages. For this option to take effect, you must run the container with `--user root`. (The `start-notebook.sh` script will `su jovyan` after adding `jovyan` to sudoers.) **You should only enable `sudo` if you trust the user or if the container is running on an isolated host.**
* `-v /some/host/folder/for/work:/home/jovyan/work` - Host mounts the default working directory on the host to preserve work even when the container is destroyed and recreated (e.g., during an upgrade).
* `-v /some/host/folder/for/server.pem:/home/jovyan/.local/share/jupyter/notebook.pem` - Mounts a SSL certificate plus key for `USE_HTTPS`. Useful if you have a real certificate for the domain under which you are running the Notebook server.

## SSL Certificates

The notebook server configuration in this Docker image expects the `notebook.pem` file mentioned above to contain a base64 encoded SSL key and at least one base64 encoded SSL certificate. The file may contain additional certificates (e.g., intermediate and root certificates). 

If you have your key and certificate(s) as separate files, you must concatenate them together into the single expected PEM file. Alternatively, you can build your own configuration and Docker image in which you pass the key and certificate separately.

For additional information about using SSL, see the following:

* The [docker-stacks/examples](https://github.com/jupyter/docker-stacks/tree/master/examples) for information about how to use [Let's Encrypt](https://letsencrypt.org/) certificates when you run these stacks on a publicly visible domain.
* The [jupyter_notebook_config.py](jupyter_notebook_config.py) file for how this Docker image generates a self-signed certificate.
* The [Jupyter Notebook documentation](http://jupyter-notebook.readthedocs.io/en/latest/public_server.html#using-ssl-for-encrypted-communication) for best practices about running a public notebook server in general, most of which are encoded in this image.

## Conda Environments

The default Python 3.x [Conda environment](http://conda.pydata.org/docs/using/envs.html) resides in `/opt/conda`. A second Python 2.x Conda environment exists in `/opt/conda/envs/python2`. You can [switch to the python2 environment](http://conda.pydata.org/docs/using/envs.html#change-environments-activate-deactivate) in a shell by entering the following:

```
source activate python2
```

You can return to the default environment with this command:

```
source deactivate
```

The commands `jupyter`, `ipython`, `python`, `pip`, `easy_install`, and `conda` (among others) are available in both environments. For convenience, you can install packages into either environment regardless of what environment is currently active using commands like the following:

```
# install a package into the python2 environment
pip2 install some-package
conda install -n python2 some-package

# install a package into the default (python 3.x) environment
pip3 install some-package
conda install -n python3 some-package
```


## JupyterHub

[JupyterHub](https://jupyterhub.readthedocs.org) requires a single-user instance of the Jupyter Notebook server per user.   To use this stack with JupyterHub and [DockerSpawner](https://github.com/jupyter/dockerspawner), you must specify the container image name and override the default container run command in your `jupyterhub_config.py`:

```python
# Spawn user containers from this image
c.DockerSpawner.container_image = 'jupyter/datascience-notebook'

# Have the Spawner override the Docker run command
c.DockerSpawner.extra_create_kwargs.update({
	'command': '/usr/local/bin/start-singleuser.sh'
})
```
