# ntp

TODO: what ntp does.

```clay
import("ntp")

Ntp greet("world") println      // Hello, world!
```

## Install

```sh
clay bricks add github.com/YOU/ntp
```

## Develop

```sh
clay tests/test_ntp.clay       # a brick can import itself by name
```

`.github/workflows/test.yml` runs the tests on Linux, macOS and Windows for every push and
pull request on GitHub (it builds Clay from source; pin `CLAY_REF` to a release once you
depend on one). On another Git host, delete it.

## Publish

1. Choose a license and add a `LICENSE` file.
2. Fill in `description` in `brick.json`, commit, and push to GitHub (or another Git host).
3. Tag a release: `git tag v0.1.0 && git push --tags`.
4. To let people find it, list it in the
   [clay-registry](https://github.com/mseminatore/clay-registry) with a pull request.
