# minecraft-overviewer-enhancements
> Automated docker images of the Minecraft Overviewer project

![Update Submodule](https://github.com/TheDruidsKeeper/minecraft-overviewer-enhancements/workflows/Update%20Submodule/badge.svg)

![Docker Build](https://github.com/TheDruidsKeeper/minecraft-overviewer-enhancements/workflows/Build%20and%20Push%20Docker/badge.svg)

[This repo](https://github.com/TheDruidsKeeper/minecraft-overviewer-enhancements) is to dockerize the [Minecraft Overviewer](https://github.com/overviewer/Minecraft-Overviewer) project. Why isn't this just part of the original project you ask? Because @CounterPillow [rejected it](https://github.com/overviewer/Minecraft-Overviewer/pull/1805) for not wanting to maintain a dockerfile ¯\\\_(ツ)\_/¯.


## Usage

See the Minecraft Overviewer documentation on [Running the Overviewer](http://docs.overviewer.org/en/latest/running/) for majority of information needed. The key differences here are that rather than running `overviewer.py` directly you will be running it via docker & will need to mount volumes to provide world data & retrieve rendered output. `overviewer.py` is the entrypoint for the docker image, and so any additional parameters provided to running the docker image will be passed along to that script.

### Example

The following steps will walk you through how to easily get set up with rendering a map of your Minecraft world(s). The example here is Linux-based, but the same can easily be accomplished in Windows or MacOS.

1. Create the directory `/var/minecraft-overviewer` with the following empty sub-folders:
    - versions
    - worlds
    - map
2. Download Minecraft's client jar that matches the version of your world and place into the versions directory following Minecraft Overviewer's [instructions for textures](http://docs.overviewer.org/en/latest/running/#installing-the-textures)
    - Ex: `wget https://overviewer.org/textures/${VERSION} -O /var/minecraft-overviewer/versions/${VERSION}/${VERSION}.jar`
3. Create the `config.py` to generate the renders:
```python
worlds["Vanilla World"] = "/var/minecraft/worlds/vanilla"
renders["vanilladay"] = {
        "world": "Vanilla",
        "title": "Daytime",
        "rendermode": smooth_lighting,
        "dimension": "overworld"
}
outputdir = "/var/minecraft/output"
```
:information_source: Note: The paths used for the worlds & output must match the paths used for volume mounts when running the docker container.

5. Copy the world you wish to render into the `worlds` directory:
```
rsync -az --progress --delete mineos:/var/games/minecraft/servers/vanilla/world/ /var/minecraft-overviewer/worlds/vanilla
```
6. Run the docker image, mounting the directories & config file used above:
```
docker run --rm \
       -v /var/minecraft-overviewer/config.py:/var/minecraft/config.py \
       -v /var/minecraft-overviewer/versions:/root/.minecraft/versions \
       -v /var/minecraft-overviewer/worlds:/var/minecraft/worlds \
       -v /var/minecraft-overviewer/map:/var/minecraft/output \
       thedruidskeeper/minecraft-overviewer:latest --config=/var/minecraft/config.py
```
7. Wait for the render to complete

The `/var/minecraft-overviewer/map` directory will now contain the output from Minecraft Overviewer and can be used however you wish, like mount it to an nginx container that's configured to host the static files (for example).

:star: Protip: Put the `rsync` & `docker run` commands into a script & run via cron job to keep your maps updated on a regular basis!

## Versions

A github action is configured to automatically update the overviewer submodule to the [latest release](https://github.com/overviewer/Minecraft-Overviewer/releases/latest). The docker image build will tag the image with the tag used on the current commit referenced by the submodule, along with the `latest` tag. See the [docker hub](https://hub.docker.com/repository/docker/thedruidskeeper/minecraft-overviewer/tags?page=1&ordering=last_updated&name=v) for the full list of versioned releases, or use the `latest` tag to stay updated.
