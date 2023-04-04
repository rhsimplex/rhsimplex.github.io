---
title: Why I quit Numerai
date: 2023-01-23 10:15:56
tags: [ml, numerai, finance]
---
Last week I cashed out over [$20,000](https://etherscan.io/tx/0x69ea5ea4212746b17d0d4d89a4b9c06a906bb703db231a82377e4573a93fc83c) lifetime winnings from the Numerai tournament. The Numerai tournament is a data science competition where particpants are rewarded for quality stock market predictions. The rewards are paid out in Numerai's custom crypto token, Numeraire (NMR). Participants must "stake" NMR to earn NMR for their predictions, and you can roll your winnings back into your stake.

Despite having a [decently-performing model](https://numer.ai/600cell) that reliably had a positive return and earned NMR, I decided it was time to get out. Why? Because in real dollar (USD) terms, I haven't actually made any money since mid-2022. While my NMR stake kept increasing, the value of NMR steadily fell even faster, causing the overall value of my stake USD to decrease.

You can see this quite plainly in the graph below: the value of my stake in NMR trends steadily upwards, while the real value in dollars spikes up and trends gently downward.

<figure>
  <img src="/images/nmr_vs_usd.svg" alt="Cross fold validation of various models"/>
</figure>

Forcing you to stake your models with a custom token exposes you to the wild volatility of the cryptocurrency markets and the hassle and risk of participating in the digital currency economy. As you can see in the figure above, the volatility of NMR matters a lot more than my model performance.

NMR also seems to be suffering from inflation from participants winning an unsustainable amount of NMR. Numerai's addition of an [exponentially decaying payout factor](https://docs.numer.ai/tournament/learn#payouts) is a tacit admission of the problem. Coupled with the falling real value of NMR, you need to win more and more just to keep up. Numerai has started offering daily instead of weekly tournaments, presumably to help combat this problem. I did not participate in the daily tournament, but it will be interesting to see how this pans out.

I haven't kept up on the Numerai Forums lately, but it's clear [I'm not the only one who has a problem with NMR](https://forum.numer.ai/t/numeraire-worst-part-of-numerai-time-to-change/6017). If Numerai is unable to use a real currency for payouts (and to be fair, they did attempt this back in 2018), why not use a less volatile token like Tether?
