# Gated token

The scaffold that `npm create redbelly-dapp` gives you, taken all the way to testnet: deployed from a verified wallet, source verified on Routescan, with the exact commands and the exact costs.

<!-- assets/deploy.gif goes here: pre-flight, ship report and deploy as they ran, cut to the receipts -->

2 contracts · 86 tests · 1 block · 2,675,097 gas · 815.53 test RBNT · about US$1.27 · verified on Routescan

```sh
npm run preflight                                      # nine checks, read-only, before you sign anything
npm run ship -- --chain 153 --account <keystore-name>  # gas, tests, analysers, one report; the deploy script wants it dated today
cd contracts && forge script script/Deploy.s.sol --rpc-url redbelly_testnet --account <keystore-name> --sender <your-address> --broadcast
                                                       # the one place a key is used: yours, in your keystore, never read by anything here
```

**The gate is in the contract, not the wallet.** Every gated function on the token asks the network's
identity verifier about the caller against request 18, the over-18 recipe, and the test base proves each
one refuses a credential that was never issued, has expired, was revoked or is from the wrong
jurisdiction.

**The check will run every time.** Subscribing, transferring, receiving: each is its own question to the
verifier at the moment it happens. A wallet that passed yesterday and was revoked today is refused today.

**Nothing deploys without the ship report.** Pre-flight, the gas report, the 86 tests and the Slither
check write one file. The deploy script will not broadcast on the mainnet without that file dated today, and
will not take a mock verifier there at all. On testnet it warns and lets you learn.

## Live on testnet

Chain 153, deployed and verified on 23 October 2026.

| | |
|---|---|
| `GatedERC20`, "Gated Token" (GTOK) | [`0x26ab267769C249a007e2f1869272bf30fd169BaC`](https://redbelly.testnet.routescan.io/address/0x26ab267769C249a007e2f1869272bf30fd169BaC/contract/153/code) |
| its transaction | `0xcb49c398ff671cb5566521f256825984b6d89b371caba0551d86eabde326f4a1`, 1,972,293 gas, 601.27 test RBNT |
| `ReceptorMock`, the verifier | [`0x2E652B946DC303d8f53bC92D0787e13B109003e5`](https://redbelly.testnet.routescan.io/address/0x2E652B946DC303d8f53bC92D0787e13B109003e5/contract/153/code) |
| its transaction | `0x5b394dc1f4ba9bdb361c087bda11a5f2b77b730a9fa8fd096c9d06d4651d29f9`, 702,804 gas, 214.26 test RBNT |
| block | 3,055,910, both in the one block, which arrived within seconds |
| cost | 815.53 test RBNT at a base fee of 304,859 gwei, about US$1.27 at that day's feed price |
| eligibility request | 18, the over-18 recipe in `recipes/over-18/` |
| records and reports | `contracts/deployments/153-GatedERC20.json`, `ship-153-2026-10-25.md` written before and again after the deploy |

Also in this repository: `RecipientGuard` at
[`0x017b98c2b39E26435f9ad3880101F9f1c5FDBE73`](https://redbelly.testnet.routescan.io/address/0x017b98c2b39E26435f9ad3880101F9f1c5FDBE73/contract/153/code),
deployed on 17 October 2026 for 503,685 gas, verified, with one real send through it and the refusal of an
unverified recipient proven on the network. It is a project of its own on the builder path. Its files
live here because this was the first scaffold I had a funded wallet for.

The verifier is a mock. Testnet has no real verifier for the over-18 recipe, so every wallet starts
ineligible and the mock's admin decides. The token has no supply yet; the first subscription and transfer
through the live gate are the next step, and this page will carry them when they are done. Nothing here
is on mainnet.

## What the deploy taught me

Measured, not estimated.

- The gas report's figure for the token, 1,971,486, was within 0.05% of the receipt. Trust it.
- The network prices gas in US dollars and converts to RBNT at the time of the call. The dollar column is
  the stable one; the RBNT column moves with the price.
- Forge's estimate on the way in shows about double the real cost, because it prices at twice the base
  fee as a ceiling. Its closing "Total Paid" line can disagree with its own receipts too. The receipts and
  your balance are the truth.
- A deploy command needs `--sender <your address>` beside `--account <keystore-name>`. Without it Foundry
  rehearses the script as its own default sender and the pre-flight rightly refuses it.
- A dry run without `--broadcast` writes a deployment record that looks exactly like a real one. Check the
  address for code before you trust a record.
- The ship report's verify command is built from whatever record exists when the report is written. Run
  the report again after the deploy, or build the verify command from the fresh record.

## Install

| You need | Why |
|---|---|
| Node 20 or later | The scripts and the web app |
| [Foundry](https://getfoundry.sh) | `forge`, `anvil` and `cast`: build, test, fork, deploy |
| A wallet verified on the network, for step 4 only | The network refuses any other sender, and so does the script's own pre-flight |

`npm run doctor` checks the first two and tells you what is missing, one line each with the fix.

1. **Install**, once per clone:

   ```sh
   npm install
   npm run contracts:install
   ```

2. **Test**:

   ```sh
   npm run test        # 86 tests in four suites on the five-state base, plus a fuzz and an invariant suite
   ```

3. **Run the local loop**, one command:

   ```sh
   npm run dev
   ```

   Forks testnet on Anvil, deploys the token and a mock, seeds five wallets into the five credential
   states, and starts the web app with a panel that flips the connected wallet between them. Ctrl-C stops
   everything.

4. **Deploy** to testnet with your own keystore. Make it once with `cast wallet import <keystore-name>
   --interactive`, verify that wallet on the network, fund it from the faucet, then:

   ```sh
   npm run preflight
   npm run ship -- --chain 153 --account <keystore-name>
   cd contracts && forge script script/Deploy.s.sol --rpc-url redbelly_testnet --account <keystore-name> --sender <your-address> --broadcast
   ```

   Then verify, with the constructor arguments from `contracts/deployments/153-GatedERC20.json`:

   ```sh
   cd contracts && forge verify-contract <address> src/GatedERC20.sol:GatedERC20 --verifier etherscan --verifier-url https://api.routescan.io/v2/network/testnet/evm/153/etherscan --etherscan-api-key verifyContract --chain 153 --compiler-version 0.8.30 --evm-version prague --num-of-optimizations 200 --constructor-args <from the record> --watch
   ```

## The flow

| When | Command | What you get |
|---|---|---|
| once per machine | `npm run doctor` | What is missing, with the fix |
| once per clone | `npm install && npm run contracts:install` | The contracts and the web app ready to build |
| every change | `npm run test` | 86 tests, every gated path in all five credential states |
| while you build | `npm run dev` | A fork, the token, five seeded wallets and the web app, in one command |
| before a deploy | `npm run gas -- --chain 153` | Gas per function in RBNT and US cents at the chain's own feed price |
| before a deploy | `npm run ship -- --chain 153 --account <keystore-name>` | One report: pre-flight, gas, tests, analysers. On mainnet, the gate |
| the deploy | `forge script ... --broadcast` | Here: one block, 2,675,097 gas across two contracts, 815.53 test RBNT |
| after the deploy | `forge verify-contract ...` | Source on Routescan for both contracts |

<details>
<summary>What the token does</summary>

A gated ERC-20 from the contract kit: an issuer registry with windows and allowances, compliance transfers
that ask the verifier about both sides, a pause with its own role, a subscription that mints 100 tokens to a
wallet that passes the gate, and the Safe plus timelock pattern for mainnet. Roles are split (admin,
pauser, compliance, issuer admin) and on testnet all four are the deployer, which the deploy says out
loud. The web app has one gated action and a `/fees` page that prices a transaction on this network from
the chain's own feed, read in the browser with no wallet.

</details>

<details>
<summary>Why this network</summary>

A token can ask the chain itself whether a wallet has passed an identity gate, at every transfer. On a
chain without that layer the same token keeps its own allow-list, off chain or on, and somebody has to
keep it true. Here the list is the network's and the token only asks.

</details>

## Docs

| | |
|---|---|
| 01. The deploy | `contracts/deployments/ship-153-2026-10-25.md`: the report that gated it, every check and figure |
| 02. The records | `contracts/deployments/`: the token, its mock and the guard, with constructor arguments encoded |
| 03. The threat model | `THREAT-MODEL.md` |
| 04. The recipes | `recipes/README.md`: the eligibility recipes the kit ships; this token uses `over-18` |
| 05. The guard | `contracts/src/RecipientGuard.sol` and its script, deployed 17 October 2026 |

## Development

```sh
git clone https://github.com/gatedpath/gated-token && cd gated-token
npm install && npm run contracts:install
npm run test
```

Issues and pull requests are welcome; `CONTRIBUTING.md` is short. If you have found a security problem,
please read `SECURITY.md` first. This is my kit's own scaffold at `npm create redbelly-dapp` and it is an
independent project: Redbelly did not make it and is not associated with it. Their developer portal is
[Vine](https://vine.redbelly.network), and that is where the network's own facts live. MIT.

<details>
<summary>The scaffold's own README, as generated</summary>

<!-- the template README goes here, unchanged except the title -->

</details>
