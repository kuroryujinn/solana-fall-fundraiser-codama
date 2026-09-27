# NOTES

## Versions
anchor-cli 1.1.2 · solana-cli 3.1.10 · node v24.20.0 · yarn 1.22.22 · surfpool installed
@codama/cli 1.6.3 · @codama/renderers-js 2.5.0 · @codama/nodes-from-anchor 1.5.6 · @solana/kit 8.3.0 · @coral-xyz/anchor 0.32.1

## TODO 3
Required: `contributor` (the signer), `mintToRaise`, `fundraiser`, `vault`.
Optional (derived by the generated client): `contributorAccount`, `contributorAta`,
`tokenProgram`, `systemProgram`.

All four PDAs have seeds in the IDL, but only the two that were optional have
seeds the client can compute from inputs it already holds. `contributorAccount`
is seeded on `[b"contributor", fundraiser, contributor]` and `contributorAta`
on `[contributor, TOKEN_PROGRAM_ID, mintToRaise]` — plain accounts from the
instruction's own input. `fundraiser`, though, is seeded on
`[b"fundraiser", fundraiser.maker]` and the vault on
`[fundraiser, TOKEN_PROGRAM_ID, fundraiser.mint_to_raise]`: both read fields
*inside the Fundraiser account itself*, which is the account being derived. To
derive an address from the data an account stores, the client would have to
fetch and decode that account first — and the very same problem shows up when
comparing with `initialize`: there `fundraiser` is seeded on plain `maker` (an
input the caller always has), so in `InitializeAsyncInput` it is optional and
derived, while in `ContributeAsyncInput` the identical account is required.
So the difference is the program's seed design, not a Codama limitation: the
renderer only made optional the derivations its seed data could actually
resolve.

## Bonus
Attempted and green: built the instruction with `getContributeInstructionAsync`,
converted it with `tests/helpers/kit-adapter.ts`, sent it through
`provider.sendAndConfirm`, and the vault grew by exactly AMOUNT. The noop
signer's address is the provider wallet, so the provider signed for real.

## One thing that surprised me
`anchor test` ran zero tests at first: the shipped script used
`tests/**/*.ts`, and bash only recurses `**` with `globstar` enabled, so the
glob silently matched just `tests/helpers/kit-adapter.ts`. I named the test
files explicitly in Anchor.toml. Second surprise: ts-mocha typechecked
`clients/js/src/generated` (tsconfig has no `include`), which forced the
tsconfig from `module: commonjs` to `module: node16` so TypeScript could
resolve Kit's `exports` maps.
