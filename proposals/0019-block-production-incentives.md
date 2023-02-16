---
simd: '0019'
title: Incentives for good block production behaviour
authors:
  - Michael Hubbard (Laine)
  - Ben Hawkins (Cogent Crypto)
category: Economics
type: Economics/Rewards/Fees
status: Draft
created: 2023-01-19
feature: TBC
---

## Summary

Block rewards form a smaller part of validator revenues and there exist insufficient incentives for validators to produce blocks or even good blocks. This is exacerbated by vote lagging and other modifications to improve credits for the sake of APY (share of inflationary rewards), though that is partially dealt with in the timely vote credits proposal.

We propose a method that provides direct incentives to validators to produce good blocks by applying a negative weighting on their points used in staking reward calculation, thereby creating a tie-in between block production and staking rewards.

## Motivation

Over the past year more and more validators have begun modifying their voting behaviour to achieve higher vote credits and thus higher share of inflationary rewards. At the same time poor hardware or networking causes some validators to produce smaller or even vote-only blocks. 

Because of the skewed scales between block rewards and inflationary rewards (and the potential future stake from having higher inflationary rewards and thus APY) validators currently don't care sufficiently about block production.

## Alternatives Considered

Measuring block production purely by blocks produced vs skipped without consideration for CUs

## Detailed Design

1. While replaying a block tally the CUs and increment a counter in the vote account, similar to the credits counter.
2. During rewards calculation currently the validator's epoch credits are stake-weighted to obtain "points" which are used to apportion the total epoch inflation to the validator's stake accounts. We propose the addition of another multiplier to this weighting by adding in the VCUR as a weighting.
3. While CUs are a second order factor of stake, and thus could ostensibly replace stake weight as a factor in points calculation, there are many validators with no or few leader slots, where this would result in a multiplier of 0 and thus no rewards, despite some small amount of stake.
4. There needs to be some normalization to work against, an expected value of CUs per block, this can be a magic constant for now as it is applied equally to all validators. The validator's ratio of mean CU/leader slot vs expected CU/leader slot becomes their points multiplier.

Current formula

$$V_{Rewards} = E_{Inflation} * [(V_{Credits} * V_{Stake}) / (C_{Credits} * C_{Stake})]$$

where $V$ = validator, $E$ = Epoch, $C$ = Cluster

The vote account needs to be modified to allow for the storage of the ECUP value, this value could be an integer measuring tens of billions, possibly hundreds of billions, the value could be divided by 1,000 or 100,000 to allow a smaller data structure to be used.

The benefit of this design is that there is no modification to the rewards calculation other than the formula for points calculation.

The maximum penalty derived from this value should be capped to 10% of rewards initially, i.e. a validator can lose up to 10% of their inflationary rewards (and their stakers consequently) per epoch, but not more. Stake already accrued is never lost and stakers will always earn at least 90% of the rewards they would earn. With time this percentage share of rewards can be changed as needed.

## Impact

Validators

## Security Considerations

Need to ensure this doesn't lead to rewards that exceed 100% or fall below 90% of what validators would receive currently, and can't be gamed, e.g. by packing self-transfers into blocks to boost CUs.

## Drawbacks *(Optional)*

CUs are dependant on network demand and this might penalize validators who produce blocks while there is little demand for block space.

Can be gamed by packing circular transfers into blocks though this is likely not economical (transfer fee).

## Backwards Compatibility *(Optional)*

Vote account structure must be modified.