# Asset Pipeline And Resource Integration

Use this workflow whenever a GameMaker project receives generated or externally
produced sprites, sounds, fonts, or other content. Keep game names, rosters,
prompts, machine paths, and exact asset counts in the project's own docs.

## Choose the runtime contract first

Decide whether each asset is a **native GameMaker resource** that must appear in
the IDE and be addressable by a stable resource name, or a **runtime-loaded
file** intentionally kept outside the resource tree.

These are different designs. A PNG copied into the project and loaded with
`sprite_add()` is not a native Sprite resource. It may render at runtime while
remaining absent from the Sprites folder and the `.yyp` resource list. When
editor visibility, resource compilation, or stable asset references matter,
create native `.yy` resources, add them to the `.yyp`, and verify resource-order
metadata. Keep source images in a separate provenance folder when useful.

The same rule applies to audio. Copying an MP3/WAV into a project and opening it
through a runtime stream API is not the same as creating a GameMaker Sound
resource. If the game is expected to use the IDE's Sounds folder, create a
native Sound resource with the correct file, add its `.yy` entry to the `.yyp`,
and reference that resource from the game. Runtime streams are valid only when
the project explicitly chooses an external-file audio contract. Never claim an
asset is integrated until it is visible in the expected IDE resource folder and
survives a clean compile/package.

## Use a stable manifest

Represent every requested visual variant with a stable key such as
`(asset_key, view_id)`. A manifest should record the key, source/model, prompt
or input reference, dimensions, transparency expectation, review status, and
destination resource.

Before generation, compare requested keys with accepted files and native
resources. Generate only missing pairs. Afterward, compare again and report
base-catalog and expansion-batch counts separately when they differ. Do not
infer completeness from a folder listing or a success message.

## Validate both sides of the pipeline

Asset verification needs two checks:

1. **Resource/build:** the native resource exists, is referenced by the `.yyp`,
   has expected dimensions and origin, and compiles.
2. **Runtime mapping:** the lookup from stable key to resource ID resolves the
   expected asset, with an intentional documented fallback for a missing key.

For native resources, a lookup such as `asset_get_index("spr_" + asset_key)` is
useful only after the naming convention is validated. A square placeholder can
make a test look successful while the catalog or mapping is broken.

Run the asset compiler/build, headless checks, and a visual smoke test. Record
resource counts and the final manifest comparison in a small machine-readable
status file. Compiled native resources may be inside game data rather than
visible as individual files in a package.

## Keep generated and source artifacts separated

Use separate directories for source inputs, generated intermediates, accepted
deliverables, and runtime package output. Make the commit boundary explicit so
unrelated generated output is not accidentally added. Preserve license,
attribution, seed, model, prompt, and review metadata wherever required.

For machine-reviewed art or audio, keep review state and criteria in the
manifest. A local visual review may reject and regenerate an asset, but it
should not silently change the stable key or overwrite an accepted revision.

## Safe integration sequence

1. Read the project's asset contract and current manifest.
2. Fetch the exact producer revision or export and verify its license.
3. Diff requested keys against accepted files and native resources.
4. Generate or copy only missing assets.
5. Validate dimensions, transparency, naming, and provenance.
6. Create/update native resources and `.yyp` references when that is the chosen
   contract.
7. Compile, run automated checks, and perform a visual smoke test.
8. Update the status manifest and project progress notes.
9. Review the diff for unrelated generated output, then commit and push.

If a tool rewrites unrelated `.yy`, `.yyp`, or resource-order serialization,
restore that noise before committing. Never hide a real resource change in a
large generated diff.
