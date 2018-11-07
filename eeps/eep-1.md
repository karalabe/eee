---
EEP:     1
Title:   Consensus update rollout
Authors: Péter Szilágyi <peter@ethereum.org>
Status:  Draft
Version: 2018-11-06
Discuss:
---

*Ethereum is a mission critical system, the long-term sustainability of which requires strict engineering practices over rock-star development. Part of that effort is ensuring we have well defined processes that all ecosystem participants understand, agree with and adhere to. Everyone hates checklists, but Ethereum will only reach its potential if we can regulate it with the gravity of flight engineering.*

*All amendments to this standardized workflow must be rationalized via real world failures, which must be included for future reference to avoid regressions. When adding a new item, please also include an inline explanation to help newcomers.*

## Abstract

The goal of this document is to act as a step-by-step guide for network participants on how to roll out consensus upgrades onto the Ethereum network in a reliable and reproducible fashion. It defines the timelines and deliverables different teams must follow, as well as the emergency procedures in case something foreseeable goes wrong.

The scope of this document is *not* the specification, implementation and preparation of forks, rather their safe rollout once the ecosystem gives the green light on deploying them.

## Preconditions

- All Ethereum Improvement Proposals (EIPs) aimed to be deployed in the hard fork have been implemented in all desired clients and a summary document has been produced with links to all the relevant code changes.
  - Beside tracking the implementation progress of EIPs, the goals of the summary document are to allow developers to cross validate other implementations; as well as to have a quick index into other code-bases in case of a consensus issue.
- All EIPs have every known corner-case covered with public consensus tests and a summary document has been produced with links from specific corner-cases in the EIP to test cases within the tests repository.
  - It makes absolutely no sense to go forward with a hard fork if the tests are incomplete. This is especially relevant for test networks where client developers are more probable to just not care, potentially breaking things for higher layer programmers.
- All relevant consensus tests are passing on desired clients using black-box testing (*hive*).
  - Although all clients generally have their own special way to run consensus tests fast as part of their CI workflow, these depend on internal assumptions or shortcuts that might not hold in live operation. Black box testing approaches can cover the full operational pipeline.

## Preparation

- Create a GitHub tracking issue with individual sections for all of the preconditions defined above, flattened instead of referencing external resources.
  - It's hard for client developers to keep up to date with many externally linked documents that are under 3rd party ownership and can change without notice. By creating a single *master document* that is considered the baseline truth, information won't fall through the cracks.
- Create a GitHub checklist based on the bullet points below, expanded appropriately with the individual ecosystem players that have a stake in the update.
  - It's important to be able to track the progress of the consensus rollout both globally as well as from the perspective of individual clients, which can also incentivize developers to put in the effort within a reasonable time frame to avoid public outcry.
- Configure a temporary ethstats server that can be used to track the upgrade status.
  - It's essential to see how each of the supported clients behave during a fork. If a network does not have a stats page, the rollout would be done blindly. If a network uses a public stats page, the information is lost among the noise of the general public.
- Configure and run a differential fuzzer throughout the test period (needs client support).
  - It's unrealistic for hand written consensus tests to cover and catch all possible issues since most stem from client optimizations. Although it's extra effort from client teams to support fuzzerz, long term they are immensely valuable **(see Byzantium post mortem)**.

## Smoke Testing

*The goal of smoke testing is to ensure that clients at least seem to be able to stay in consensus with a new set of rules, without causing unwarranted annoyances for higher layer developers.*

- Define a new Ethereum network genesis with all previous hard forks enabled and the new one forking after some blocks (get everyone onboard beforehand). The genesis number should be reasonably high to allow testing consensus updates dependent on chain depth (e.g. ice age).
  - Supporting genesis blocks with a number larger than 0 is essential for testing anything that depends on chain depth. Generally EIPs are created explicitly for mainnet, so smoke tests are the only opportunity to ensure depth dependent code works before going live.
- Every client team should run and add to the stats page at least one mining and one non-mining full nodes. Ideally add light nodes and other client specific infrastructure components too.
  - Different modes of operations have different behaviors under similar network conditions. Hard forks tend to affect both mining and validating code, but optimizations often cause different code paths to be taken.
- Keep the network running until all desired players are able to synchronize, mine and transact.
  - With more and more clients being developed, there's a high probability that some bugs will surface already at this point. Expect failures and only continue when consensus is reached on which implementations go forward.
- Tear down all the smoke-test configured clients and reset the stats page.

**This is not a game and we're not playing catch-up! Client implementations not functional at this point must be ignored onward! The network cannot afford partially tested clients and the ecosystem cannot afford to waste developers' time!**

## Testnet Rehearsal

*The goal of the testnet rehearsal is to create a temporary fork of a live but low difficulty network. This allows client developers to have real-world transaction traffic executed on top of an updated protocol, without breaking anything for users of said network.*

- Pick a rehearsal fork block number an ensure it can be configured in clients without hard coding, or at least without requiring a live release.
  - DApp developers rely even on the test networks. With more and more clients being added, testing might take more than expected. By supporting rehearsals without doing releases the community can properly address issues without the pressure of maintaining a live network.
- Every client team should run and add to the stats page at least one mining and one non-mining full nodes. Mining nodes should have approximately equal capacity across clients, but below the live networks hash-rate. Add archive, light nodes and other nodes too.
  - The rehearsal fork will actively compete with the live non-forked chain. The goal is to test, not to break, so the total hash-rate should be below the live network to avoid accidentally surfacing sync issues into the old chain **(see Ropsten Constaninople post mortem)**.
- Every client team should run and add to the stats page at least one non-forking full node.
  - Things will get messy in the testnet rehearsal and we need all the information we can in a centralized location. By having both old and new nodes in the same stats page developers can a lot better track how the two chains are evolving.
- Keep the network running until all desired players are able to synchronize, mine and transact.
  - The testnet rehearsal has a nice side effect that it produces an extremely unhealthy network as there's a longer competing chain. This can surface a significant number of client issues around synchronization and chain reorganizations **(see Ropsten Constaninople post mortem)**.
- Tear down all the clients and the custom stats page too.

**As previously, any client unwilling to keep up needs to be left behind, otherwise we'll never reach the end of the rollout flow!**

## Testnet Rollout

*The goal of the testnet rollout is to enable new protocol features for all network participants and let them try to break things. This phase already has serious implications as failures can potentially break higher layer use cases. Consider accepting bounties at this point!*

- Pick a testnet fork block number and ship a live, stable release with it.
  - It's unrealistic to expect DApp developers to run custom built or custom configured nodes. The only meaningful way to fork a test network is to ship a proper client and leave enough time for everyone to update to it.
- Every client team should run and add to the stats page at least one mining and one non-mining full nodes. Ideally add light nodes and other client specific infrastructure components too.
  - Without running mining nodes, client developers risk the fork failing due to testnet miners not updating and thus stalling the new chain **(see Ropsten Constaninople post mortem)**.
- Keep the network running until the entire ecosystem is confident things can't go wrong.

## Mainnet Rehearsal

*I'm not yet fully sure this phase is needed. It can mirror the testnet rehearsal but for mainnet to stress test the code a bit more thoroughly. There might be annoying difficulties related to hashing power, so this might need some community altruism to pull off.*

## Mainnet Rollout

*This is as dangerous as it gets. Doing a live update has serious consequences on all participants. Code at this point must already be battle tested and focus should be on damage control.*

**This section should be written up perhaps by Hudson or some other hard fork coordinator. Things to cover are coordination between exchanges, mining pools, handling of emergency situations, paging setups, call duty, etc.**

## Post Mortems

The goal of this section is to both describe past issues that influenced the evolution of the proposed consensus update workflow, as well as to act as a regression prevention mechanism for contributors. Only add details relevant to this process.

### 2017 Oct 15 - Byzantium

During the Byzantium mainnet hard fork, the Parity team made 3 emergency releases 4, 3 and 2 days respectively before the hard fork, whereas the Geth team made 1 emergency release 1 day before the hard fork. Both clients were passing all consensus tests.

The reason for the last minute hotfixes was the creation and introduction of a differential fuzzer a week before the scheduled hard fork, which proved immensely effective at finding issues. Although the fork was a success, it's obvious that fuzzers need to be an ongoing part of consensus testing.

### 2018 Oct 17 - Ropsten Constantinople

During the Constantinople hard fork of the Ropsten test network, Geth and Parity got stuck at the fork block. Later both moved a few hundred blocks, after which they split into two chains and Geth got stuck for a while before continuing. Some Geth nodes were on the same chain as Parity, some rejected it. For a couple days multiple chains were competing and clients in general were extremely unstable with regard to sync.

The reason for getting stuck in the hard fork originally was that both Geth and Parity teams assumed some DApp developer will be mining, so neither team started miners. Turned out nobody was mining. The takeaway is that clients must not rely on community mining for testing a hard fork on a testnet.

The reason Geth and Parity split into two after getting a miner onboard was a consensus issue in Parity, but since the miner was backed by a Parity node, there was nothing to push the Geth chain forward. The takeaway is that while developers figure out which client is correct on a fork, both chains must be able to go forward, so all client teams must have their own miners, one is not enough.

While trying to fix the stuck chains and get some new mining nodes onboard, it became apparent that both Geth's fast sync and Parity's warp sync is useless as they sync to the heaviest chain and ignore state transitions (by design). This was the reason why Geth and Parity nodes intermingled on chains they otherwise considered invalid. The takeaway is that once something goes wrong, it can be hard to get new clients on the correct chains, so it's better to have all nodes deployed before the fork hits. Clients also need to support rolling back chains and blacklisting certain blocks.

After identifying the consensus issue in the Parity code and stopping mining on that chain, Geth still had a hard time to sync as the longest chain was invalid. Some internal optimizations were not really properly handling long competing but invalid chains. The takeaway is that we need to be able to test scenarios where the consensus rules conflict with the GHOST protocol reorg rules. See next part on how to do it.

Since the chain split into many forks and clients had a hard time picking any of those and remaining up to date with them, we essentially destroyed the utility of the Ropsten testnet for a few days. The Ropsten transaction load was however needed to trigger the issue int he first place. The takeaway is that instead of doing an irreversible fork for the stress test, a "private" rehearsal one should be done with a limited number of nodes. Doing a private fork, all the transactions get replayed and consensus issues are hit, but we're not crippling higher layer developers if things go bad. Care must be taken to keep the hash-rate below the live network to avoid accidentally getting newly syncing nodes over to the fork.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
