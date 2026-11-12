# rebar — SafeRE benchmark fork

This is the [SafeRE project's fork](https://github.com/safere/rebar) of
[BurntSushi/rebar](https://github.com/BurntSushi/rebar), a biased barometer for
gauging the relative speed of regex engines on a curated set of tasks.
The benchmark framework, original workloads, and methodology come from upstream
rebar.

We maintain this fork to benchmark [SafeRE](https://github.com/safere/safere)
against other regex engines. It adds SafeRE and RE2/J runners alongside the
engines already supported upstream, with JMH measurements for both additions.

## Changes from upstream

### Added engines

The added engines are registered in
[`benchmarks/engines.toml`](benchmarks/engines.toml):

| Engine | API and measurement mode |
| --- | --- |
| `safere/string` | SafeRE over Java strings, with UTF-16 match offsets. |
| `safere/utf8` | SafeRE directly over UTF-8 bytes, with byte offsets. |
| `safere/utf8-vector` | SafeRE's UTF-8 API with its experimental Vector API scanner enabled. |
| `re2j` | RE2/J's Java String API, with UTF-16 match offsets. |

Both runners use Maven and JMH, with explicit JVM warmup and a separate
measurement JVM. They support matching and compatible single-pattern compilation
workloads. Compilation measurements report warmed compilation costs; search
measurements exclude pattern compilation and input preparation. See the
[SafeRE runner](engines/safere/README.md) and
[RE2/J runner](engines/re2j/README.md) for toolchains, measurement details,
supported workloads, and semantic limitations.

### Workload and harness changes

* Extend the existing benchmark definitions to select the added engines where
  supported, with engine-specific expected counts for differences such as
  UTF-16 versus byte offsets and Unicode semantics. Unsupported syntax and
  incompatible workloads are excluded; the runner READMEs explain the limits.
* Terminate runner process trees on timeout or interruption so child JVMs
  launched by JMH do not continue running after the runner stops. The SafeRE
  runner also cleans up its JMH temporary files on termination.
* Recognize OpenJDK's HotSpot VM name as well as Oracle's in the existing
  `java/hotspot` engine's version check.
* Skip the unused PCRE2 POSIX compatibility source when building the vendored
  PCRE2 runner, since its optional header is absent from the vendored sources.

## Branches and upstream contributions

| Branch | Purpose |
| --- | --- |
| `safere` | The fork's default branch. Integrates our runners, harness changes, and fork-specific documentation. Previously named `personal`. |
| `master` | Tracks upstream `BurntSushi/rebar`'s `master`, without fork-specific changes. The starting point for upstream contributions. |
| Topic branches | Isolate individual changes for review or contribution, such as adding an engine or fixing a runner. |

For a change intended for upstream, create a topic branch from an up-to-date
`master` and include only the relevant code, benchmark definitions, and
documentation. Open the upstream PR from that topic branch, and merge the same
branch into `safere` to use the change in this fork. If a change already exists
only on `safere`, cherry-pick its relevant commits onto a fresh topic branch
from `master`.

When upstream advances, fast-forward `master` to upstream and merge `master`
into `safere`. Keep this fork's README changes on `safere` so they stay out of
upstream contributions.

## Build and run

Clone this fork to get the added runners on the default `safere` branch:

```sh
git clone https://github.com/safere/rebar
cd rebar
cargo build --release
```

Building the added runners requires JDK 21 or newer and Maven 3.9 or newer.
Building `safere/string` also builds the two UTF-8 modes; running
`safere/utf8-vector` requires a JDK with the `jdk.incubator.vector` module.

For example, build the SafeRE and RE2/J runners and compare their String APIs
on the curated literal workloads:

```sh
./target/release/rebar build -e '^(safere/string|re2j)$'
./target/release/rebar measure -e '^(safere/string|re2j)$' \
  -f '^curated/01-literal/' --timeout 60s > results.csv
./target/release/rebar cmp results.csv
```

Other engines have their own build requirements. See [BUILD](BUILD.md) and the
[runner documentation](engines/README.md); when following upstream cloning
instructions, use this fork's URL to get these additions.

## Documentation

* [METHODOLOGY](METHODOLOGY.md): motivation, design, workload selection, and
  evaluation protocol.
* [BUILD](BUILD.md): building rebar and its engine runners.
* [TUTORIAL](TUTORIAL.md): exploring benchmarks and comparing measurements.
* [CONTRIBUTING](CONTRIBUTING.md): adding benchmarks and engine runners.
* [MODELS](MODELS.md): the types of workloads measured.
* [FORMAT](FORMAT.md): benchmark directory structure and TOML definitions.
* [KLV](KLV.md): the protocol between rebar and its runners.
* [BIAS](BIAS.md): discussion of benchmark bias.
* [WANTED](WANTED.md): upstream ideas for additional engines.
* [BYOB](BYOB.md): using your own engines and benchmark definitions.

## Upstream results

The [upstream README](https://github.com/BurntSushi/rebar#results) presents
upstream's curated benchmark results and analysis. The
[recorded measurements](record/README.md), including the
[2025-12-19 curated report](record/curated/2025-12-19/README.md), are retained
here as upstream results. They do not include this fork's SafeRE or RE2/J
additions. Use the runners above to collect comparisons for those engines.

## License

See [UNLICENSE](UNLICENSE). Bundled engine sources retain their own licenses
and notices.
