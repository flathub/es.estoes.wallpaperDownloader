### Generating maven-dependencies.yaml

Here's a *very* hacky way to generate the `maven-dependencies.yaml` file until [maven support](https://github.com/flatpak/flatpak-builder-tools/pull/253) is added to `flatpak-builder-tools`:

Temporarily enable network access at build time by adding the following key under `build-options:` in the manifest file (`es.estoes.wallpaperDownloader.yaml`):

```yaml
      build-args:
        - --share=network
```

Now, *delete* the following line from the manifest file, found under the `sources:` key:

```yaml
      - maven-dependencies.yaml
```

Now, invoke `flatpak-builder`:

```bash
flatpak-builder --build-only --force-clean --keep-build-dirs --install-deps-from=flathub builddir/ es.estoes.wallpaperDownloader.yaml
```

After the build succeeds, make sure you have the [`fd` tool](https://github.com/sharkdp/fd) installed.

Then, run these two commands to finally generate the `maven-dependencies.yaml` file:

```bash
cd .flatpak-builder/builddir/wallpaperdownloader/.m2/repository
fd '\.(jar|pom)$' | sort -V | xargs -rI '{}' bash -c 'echo -e "- type: file\n  dest: .m2/repository/$(dirname {})\n  url: https://repo.maven.apache.org/maven2/{}\n  sha256: $(sha256sum {} | cut -c 1-64)"' > ../../../../../maven-dependencies.yaml
```

To test if this generated file actually works, first *revert* the changes you did to the manifest file in the previous steps.

Then, invoke `flatpak-builder` with the `--sandbox` option:

```bash
flatpak-builder --force-clean --sandbox builddir/ es.estoes.wallpaperDownloader.yaml
```

The build should succeed if the file was generated correctly.
