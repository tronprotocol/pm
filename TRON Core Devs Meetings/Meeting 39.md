# Core Devs Community Call 39
### Meeting Date/Time: June 6th, 2025, 6:00-6:30 UTC
### Meeting Duration: 30 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/140)
### Agenda

*  Progress of [Proposal: Reduce TRX block rewards](https://github.com/tronprotocol/tips/issues/738)


## Details

**Progress of Proposal: Reduce TRX block rewards**

* Jake

  Let's start now without waiting any longer to save everyone's time. We only have one agenda today: discussing the TRX production reduction proposal. Murphy will share the latest updates, and we'll dive straight in.

* Murphy

  Can everyone see the screen?


  Alright, I’ll sync you with the latest developments and analyze the TRX production reduction plan. We’ll look at how the inflation rate is expected to change and update you on the proposal details. First, the proposal was released on March 19th, and we’ve invited community discussion for over two months. After receiving mostly positive feedback, we first discussed the proposal at the 37th developers’ meeting, where there were both supportive and opposing voices. We summarized the results and communicated them to the community through marketing and operations. Later, we launched the plan on the testnet and promoted the production reduction proposal, which didn’t cause significant community resistance.

  At last week’s developers’ meeting, we further confirmed the specific reduction parameters: the block reward will decrease from 16 TRX to 8 TRX, and the voting reward from 160 TRX to 128 TRX, with a total reduction ratio of approximately 77%. Today’s meeting focuses on sharing the analysis and impacts of the reduction.

  The proposal compares on-chain data from when it was initiated and predicts outcomes for 20%, 30%, and 50% production reductions. According to the analysis, the annual inflation rate after a 50% reduction would be -1.78% (a deflation rate of 1.78%), all within the range of -2%, which is a healthy metric compared to other mainstream public chains, indicating all options are viable.

  The main argument from opponents is that reducing rewards will affect stakers’ returns. We analyzed the specific impacts on stakers’ returns at different reduction ratios. Due to TRON’s unique energy model, we compared two dimensions: without energy leasing income, the annual staking yield of TRON would decrease from a maximum of 4.15%, dropping to approximately half (2.08%) at a 50% reduction. However, considering the active energy leasing market, where staked energy can be fully utilized in the energy market, the current comprehensive annual yield with leasing is as high as 9.15%. Even after a 50% reduction, the comprehensive annual yield would remain at 7.08%, still competitive compared to staking projects on other chains. This is the impact on staking returns.

  Regarding the impact on long-term staking rates, since there’s no direct formulaic relationship between production reduction and staking rates, we can only analyze qualitatively. Short-term after the proposal takes effect, some profit-driven funds may withdraw due to reduced returns. However, as analyzed, TRON’s staking returns remain competitive, so the withdrawal is expected to be limited. In the medium to long term, sustained deflation will increase TRX’s scarcity and value, creating a potential expectation of price appreciation. With a reduced TRX supply, the cost of obtaining energy (with a fixed energy supply) may rise, making energy more valuable. To compete for energy for transactions, more users may be incentivized to stake TRX for resources, which will impact staking rates.

  For the impact on burning, also a qualitative analysis: ignoring uncontrollable price factors, TRX production reduction doesn’t directly affect transaction fees. Short-term, with some stakers withdrawing, lower energy acquisition costs may lead to more transactions via staking, potentially reducing burning volume. In the medium to long term, rising transaction fees (due to potential price increases) may decrease transaction volume, also reducing burning volume. Thus, this proposal aims to adapt to TRON’s current transaction scale and activity, while the core for long-term benefits still requires finding new burning growth points and stimulating transaction scale and activity.

  Below is a comparison with historical token economic model changes on other public chains:

  BTC’s reward adjustment is pre-fixed, halving every 4 years, which has driven price increases after multiple halvings. And for ETH, it initially had a high inflation rate, about 90%; after EIP-1234 and EIP-1559 launched in 2021, it reduced rewards significantly, and post-Merge, with Layer 2 integration, ETH’s supply rate dropped from about 4.5% to 0.4%. Our reduction is more moderate by comparison. Solana on the other hand, uses a preset rule with a current 5% inflation rate and a 15% annual reduction. TRON last adjusted its supply in 2021, and after five years of ecosystem maturity and changing on-chain data, adjusting TRX rewards is now reasonable.

  Let’s talk about the proposal’s launch. After last week’s meeting, the parameters are confirmed as 8 (block reward) and 128 (voting reward). The proposal will launch on June 10th, 2025 (next Tuesday), and today’s meeting may be the final developers’ meeting on this proposal. The proposal is still open for feedback—please share any questions or comments via comments. Based on current on-chain data, the annual inflation rate is -0.85%, which will drop to -1.29% after the proposal takes effect, returning to the 2021-2022 level (before the energy unit price increased from 210 SUN to 420 SUN and the dynamic energy model was activated). That’s all for now. Any questions?

  We’ve updated the proposal with these parameters, the launch time, and relevant data. Feel free to review the proposal for details.

* Neo

  Could you explain how the annual yields with and without energy leasing were calculated in that table? The percentages specifically.

* Murphy

  One second, let me find that sheet for you. For the annual yield without energy leasing, we calculated it based on data: 176 TRX block rewards per block multiplied by one year, divided by the current total staked amount, resulting in a yield of 4.15% per TRX staked. The JustLend platform’s data includes energy leasing income; my calculations via other methods had slight deviations, but still around 9%. Therefore, for the "without leasing" yield, we subtracted the 5% annual energy leasing income from the comprehensive yield.

* Neo

  Got it. Have any SRs participated in the discussion?

* Murphy

  In the comments section, I believe no SRs have joined the discussion yet. The big whales from the community have been paying much attention to the issue, and some SRs have asked about it but do not have any feedback yet, and I haven’t seen direct SR participation in the GitHub issue.

* Neo

  Understood.

* Jake

  If there are no further questions, that is a wrap for today. Goodbye!





  

### Attendance
* Daniel
* Gordan
* Sunny
* Neo
* Blade
* Leem
* Brown
* Mia
* Wayne
* Federico
* Murphy
* Jake