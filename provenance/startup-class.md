# Supported startup class receipt

This slice replaces only `cocotex.cls`, generating the supported CoCoTeX 0.4.0
class from [the maintained source change](https://github.com/cartavelio/CoCoTeX/pull/1).
It loads `kvoptions-patch` only on LaTeX formats before `2020/10/01`, retaining
the existing class and language option processing.

## Source and runtime pairing

| Role | Revision |
| --- | --- |
| Original CoCoTeX source | `9c71be6ff1075b9535d13ddacc894a11e236b1af` |
| Maintained CoCoTeX source | `f76c3103f2ce500a89ee3bc6a125409bbcfd68e9` |
| Runtime base and retained helper tree | `ed41b568accfed686f30d3e0d83324e59c0235f9` |
| Effective runtime | The commit containing this receipt; identify it with `git rev-parse HEAD` in the paired checkout |

The original runtime contains 78 tracked files. Its other 77 files remain at
the base revision's bytes and modes, and the class retains mode `100755`.
In particular, `coco-floats.sty` remains at SHA-256
`86e51b17da90f8881c584c086d14da94d6b52b3acbe9aa03f51dbecd463cd8f1`.
The release installer emits thirteen files; only the class is overlaid here.
All twelve companion outputs match the runtime base and are left untouched.
This receipt and its source licence notice are metadata additions. Float-anchor migration and Scripello repinning are separate changes.

## SHA-256 receipt

| Artifact | SHA-256 |
| --- | --- |
| Original runtime `cocotex.cls` | `92d12d8996fa239f3db8f9169427da2bb87ca7164236c9142636d22201e9737e` |
| Generated and installed-worker `cocotex.cls` | `29ac17c9be114cec94af6d661d01eae0e4ea732ab8c1a106f36b909dbf17cfc2` |
| Maintained `src/cocotex.dtx` | `3efbc954f36a8a6575245a5bf2ce401763100a9195ad838ff8c4ac157d6c64ce` |
| `lib/install/cocotex.ins` | `dccfa0563b9575e553f036e42da45d959f5802c5baa999f2efdb78f3da3e0a23` |
| Ruby-builder merged `cocotex.dtx` | `544b70adb498b0e0cfee0140ad225bc7c3f7a62c5caa655ba88826b21beb99de` |

The source version remains `0.4.0`, dated `2024/01/29`. The output includes
the explanatory comment emitted by the source's DocStrip `class` guard.

## Reproduction

Use Ruby 3.1.7 with Bundler and a working `latex` command, and set `runtime`
to this checkout's absolute path. Work in a disposable source checkout:

```bash
set -euo pipefail
: "${runtime:?Set runtime to the paired xerif-latex checkout}"
scratch=$(mktemp -d)
git clone https://github.com/cartavelio/CoCoTeX.git "$scratch/source"
git -C "$scratch/source" checkout --detach f76c3103f2ce500a89ee3bc6a125409bbcfd68e9
(
  cd "$scratch/source"
  # The release front end sources this optional integration environment hook.
  touch source.sh
  export BUNDLE_APP_CONFIG="$scratch/bundle-config-build"
  export BUNDLE_PATH="$scratch/gems"
  ( cd lib/ruby && BUNDLE_FROZEN=false bundle install )
  ./build.sh build --keep-temp --no-color
  echo '29ac17c9be114cec94af6d661d01eae0e4ea732ab8c1a106f36b909dbf17cfc2  build/cocotex.cls' | sha256sum --check
  cmp build/cocotex.cls "$runtime/cocotex.cls"
  for file in build/coco-*.sty build/coco-*.lua; do
    cmp "$file" "$runtime/${file##*/}"
  done
)
# The source also supplies a standalone class-checksum gate:
BUNDLE_APP_CONFIG="$scratch/bundle-config-check" \
  bash "$scratch/source/lib/install/check-docstrip.sh"
```

The builder merges the ordered DTX modules using `VERSION`, then runs the
unchanged installer through DocStrip. Do not replace this runtime's helper
tree with the source repository or copy the other generated outputs as part
of this slice. Preserve the existing class mode when installing its bytes.

## Validation evidence

The unchanged Ruby release builder and standalone checksum gate passed with
Ruby 3.1.7, Bundler 2.3.27, `thor` 1.5.0, `tomlib` 0.7.3 and `bigdecimal`
4.1.3. Extraction used pdfTeX 1.40.24 / TeX Live 2022 (Debian), LaTeX
`2022-11-01` patch level 1, and DocStrip v2.6b (`2022-09-03`). A separate
Bundler configuration was used for each check; frozen mode was disabled only
for the retained checkout's initial dependency install, before running the
unchanged builder with its normal frozen behavior.

A second extraction of the retained merged DTX with LuaHBTeX 1.24.0 / TeX
Live 2026, LaTeX `2026-06-01` and DocStrip v2.6c (`2024-12-23`) reproduced
all thirteen outputs byte for byte. The twelve companion outputs also matched
the runtime base. All 77 retained runtime files were compared byte for byte,
and all 78 original file modes were preserved.

The generated class was compared directly with `/opt/transpect/latex/cocotex.cls`
in the cached worker image, local image ID
`sha256:a93d6294503ed722d37ae371a0472e6ef61a79a1284f78b19eb0784c4564008a`.
Both have the full class digest recorded above.

The established [worker startup check](https://github.com/keethburu/scripello/blob/b7c6a024c957a5b8b1dd129fd546671500970ae2/worker/check_startup.py)
was rerun on LaTeX `2026-06-01`, mounting the candidate runtime at the worker's
actual `/opt/transpect/latex` path. The candidate and existing installed runtime
both exited 0: default layout, book/monograph selection, main English, French
text and Hebrew script options were retained without startup compatibility
warnings. The unchanged runtime base, tested as a negative control, exited 1
on the incompatible `kvoptions-patch` warning and aborted package loading.
These fresh checks used the shared renderer lock, a 3 CPU / 4 GiB cap and
bounded commands with network disabled.

The earlier source Test evidence is reused by the exact generated class
SHA-256 above, rather than described as new historical-format runs:

| Actual LaTeX format | Observed startup behavior |
| --- | --- |
| `2020-02-02`, pdfLaTeX / TeX Live 2019 | Default book, Spanish and endnotes retained; real legacy patch loaded |
| `2020-10-01`, pdfLaTeX / TeX Live 2020 | Requested article, Spanish and endnotes retained; recorder confirms patch skipped |
| `2024-11-01`, LuaLaTeX / TeX Live 2024 | Requested article, Spanish and endnotes retained; patch skipped, no compatibility warning |

Those source tests built class digest `29ac17c9...` at source revision
`ad1c6f7d62f7f97e5a10dcc810ffcd8b4f981418`; later source changes retained that
class digest. The pre-guard modern control loaded the patch and emitted its
incompatibility warning. The merged source's public
[DocStrip check](https://github.com/cartavelio/CoCoTeX/actions/runs/44592333051/job/135801334290)
also passed on the approved head `c38f5a768ac01f1640d3f49b296256331dbd63e4`.

These checks cover startup, output bytes and preservation of the bundle.
They do not establish float behavior, a full manuscript regression or PDF/UA
conformance. Existing Scripello renders used the original component pins with
their separately recorded startup and float patches; this runtime commit does
not change those pins or remove those patches.

## Source and notices

Matching source, including its original notices, is available at
[CoCoTeX `f76c3103f2ce500a89ee3bc6a125409bbcfd68e9`](https://github.com/cartavelio/CoCoTeX/tree/f76c3103f2ce500a89ee3bc6a125409bbcfd68e9).
Keep that source with distributions of the generated runtime. The generated
class's existing source/distribution and modified-filename notice is preserved
verbatim. The source repository's licence is reproduced in
[CoCoTeX-LICENSE](CoCoTeX-LICENSE). This receipt records the class's provenance;
it does not resolve the separate naming/distribution review or relicense the
helper bundle. The existing derivative licence assessment remains
`NOASSERTION`, under temporary acceptance pending the independent follow-up.
