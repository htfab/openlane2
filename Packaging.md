# Packaging for Tiny Tapeout

Instructions to rebuild the Docker container & Python wheels used in the Tiny Tapeout GitHub actions.

## 1. Build & test the nix package

Run `nix-shell`. This will rebuild if necessary, then drop you into a shell with the openlane2 and its dependencies available via nix. Exit the shell when done testing.

## 2. Build the docker container

`IMAGE_PATH=$(nix build --extra-experimental-features "flakes nix-command" --print-out-paths --no-link --accept-flake-config --option system x86_64-linux --extra-platforms x86_64-linux '.#packages.x86_64-linux.openlane-docker')`

`cat $IMAGE_PATH | docker load`

## 3. Build wheels for `libparse-python`

(`libparse-python` doesn't change often, so you might reuse the wheels from the last `TinyTapeout/libparse-python` release.)

Clone `https://github.com/TinyTapeout/libparse-python` and change into the repo directory.

Build & install for the current python version:

- `pip install .`

Use `cibuildwheel` to build a manylinux wheel:

- `pip install cibuildwheel`, if necessary
  
- `CIBW_SKIP="pp*" python -m cibuildwheel --output-dir wheelhouse`
  
Built wheels are in the `wheelhouse` directory.

## 4. Build wheel for `openlane2`

Return to the `openlane2` directory.

Update the version in `pyproject.toml`.

Build & install `openlane2` for the current python version:

- `pip install .`

Make a redistributable wheel:

- Make a `wheelhouse` subdirectory and copy the contents of the `libparse-python/wheelhouse` (or the wheels downloaded from the github release) into it.
  
- `pip wheel -w wheelhouse -f wheelhouse .`

## 5. Test it

Run `OPENLANE_IMAGE_OVERRIDE=openlane:tmp-x86_64-linux openlane --dockerized`.

## 6. Upload the wheels

If `libparse-python` was changed, create a new tag & relase under `TinyTapeout/libparse-python` and upload all the wheels in the `libparse-python/wheelhouse` directory.

Create a new tag & relase under `TinyTapeout/openlane2`. Upload the newly created `openlane-<version>.whl` file from `openlane2/wheelhouse`. There will be other wheels for the dependencies (and possibly older versions of openlane2 built earlier), ignore those.

## 7. Upload the docker container

`docker tag openlane:tmp-x86_64-linux ghcr.io/tinytapeout/openlane2:<version>`

`docker push ghcr.io/tinytapeout/openlane2:<version>`

## 8. Update the action

Edit `action.yml` in `TinyTapeout/tt-gds-action` to change:

- Container version in `OPENLANE_IMAGE_OVERRIDE`
  
- Wheel versions in `pip install <whatever>.whl` lines
  
Check if an example project hardens successfully.
