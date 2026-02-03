# N2BIN Containers

Build apptainer containers for the N2BIN courses.

# Using the containers

The containers are published to GHCR and can be pulled and run with apptainer. For example, to run the samtools container:

```bash
apptainer run oras://ghcr.io/dodslaser/n2bin-samtools:latest <SAMTOOLS ARGS>
```

# Building the containers

> [!NOTE]
> These instructions assume you have cloned the repository and navigated to it.


The easiest way to build the containers is to use [Pixi](https://pixi.sh):


If you don't have pixi, it can be installed by running:

```bash
curl -fsSL https://pixi.sh/install.sh | sh
```

To build a specific container (e.g. samtools), run:

```bash
pixi run build samtools
```

To build all containers, run:

```bash
pixi run build-all
```

You can also build the containers directly with `apptainer`:

```
apptainer build --build-arg environment=samtools --force samtools.sif base.def
```

# Running without containers

The tools can also be run without containers using `pixi run`

```bash
pixi run -e samtools samtools --version
```

# Adding new containers

To add a new container, create a new feature `[feature.<container_name>]` to `pixi.toml`, specifying the required packages and dependencies.

```bash
pixi add --feature <container_name> <package1> <package2> ...
```

Then add an environment under the `[environments]` section, referencing the feature you created (and possibly more features if you want a multi-tool container).

```bash
pixi workspace environment add <container_name> --feature <container_name> --no-default-feature
```
Pixi won't solve dependencies for the feature until it is referenced by an environment, so after adding the environment we need to upgrade the feature to get the latest package versions.

```bash
pixi upgrade --feature <container_name>
```

Set the `APP_ENTRYPOINT` and `APP_TEST_CMD` environment variables for the container in the `[environments.<container_name>]` section. APP_ENTRYPOINT should be set to the main executable of the tool, and APP_TEST_CMD should be a command that can be used to test if the tool is working (e.g. `my_tool --version`).

> [!IMPORTANT]
> The test command should include the full command to run the executable, including the executable name.

```toml
[environments.<container_name>]
...
activation.env.APP_ENTRYPOINT = "<main_executable>"
activation.env.APP_TEST_CMD = "<main_executable> --version"
```

Finally, add the container name to the `depends-on` list in the `[tasks.build-all]` section.

```toml
[tasks.build-all]
depends-on = [
    ...
    { task = "build", args = ["<container_name>"] },
]
```