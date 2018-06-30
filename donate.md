## Please Support This Work {#donate}

I have produced this work using my own personal resources and my own time. No-one has paid me for this. Please sponsor my work! I intend to make free videos and tour the world presenting this information live. I have produced hundreds of instructional videos at [ScalaCourses.com](https://www.ScalaCourses.com), and I am ready to do the same for Ethereum projects. If your organization would like to become a sponsor, and get a lot of exposure to Ethereum developers world-wide, please email me at [mslinn@micronauticsresearch.com](mailto:mslinn@micronauticsresearch.com). You can also contribute below. Every contribution helps!

<div style='display: inline-block; padding: 0.5em; width: 23%; text-align: center; vertical-align: top;'>
<b>ETH</b><br>
<a href="https://etherscan.io/address/0x844FdA3cD125df8464574977169DE4ecEa31aA65" title="See this Ethereum address on the blockchain explorer" style="font-size: 0.35vw;"><img src='images/0x844FdA3cD125df8464574977169DE4ecEa31aA65.png' style='width: 100%; height: auto;' /><br>0x844FdA3cD125df8464574977169DE4ecEa31aA65</a>
</div>

<div style='display: inline-block; padding: 0.5em; width: 23%; text-align: center; vertical-align: top;'>
<b>BTC</b><br>
<a href="https://explorer.bitcoin.com/bch/address/3DzwmiMnfK5JbNGVb7CXHydGdifGVm72Gc" title="See this address on the BitCoin explorer" style="font-size: 0.35vw;"><img src='/images/3DzwmiMnfK5JbNGVb7CXHydGdifGVm72Gc.png' style='width: 100%; height: auto;' /><br>3DzwmiMnfK5JbNGVb7CXHydGdifGVm72Gc</a>
</div>

<div style='display: inline-block; padding: 0.5em; width: 23%; text-align: center; vertical-align: top;'>
<b>BCH</b><br>
<a href="https://explorer.bitcoin.com/bch/address/bitcoincash:qqhxm8wdl3k9fnjj6hhhs366ucfpq8qj8quq8p2d42" title="See this Bitcoin Cash address on the Bitcoin explorer" style="font-size: 0.35vw;"><img src='/images/qqhxm8wdl3k9fnjj6hhhs366ucfpq8qj8quq8p2d42.png' style='width: 100%; height: auto;' /><br>qqhxm8wdl3k9fnjj6hhhs366ucfpq8qj8quq8p2d42</a>
</div>

<div style='display: inline-block; padding: 0.5em; width: 23%; text-align: center; vertical-align: top;'>
<b>LTC</b><br>
<a href="http://explorer.litecoin.net/address/MKNtixLfWGvFQxMyLF2eDV1Dwv5TP25Fi1" title="See this LiteCoin  address on the LiteCoin explorer" style="font-size: 0.35vw;"><img src='/images/MKNtixLfWGvFQxMyLF2eDV1Dwv5TP25Fi1.png' style='width: 100%; height: auto;' /><br>MKNtixLfWGvFQxMyLF2eDV1Dwv5TP25Fi1</a>
</div>

<style>
.tip-button {
  width: 304px;
  height: 89px;
  background-size: 100%;
  background-image: url('images/1_pay_mm_off.png');
  cursor: pointer;
}

.tip-button:hover {
  background-image: url('images/1_pay_mm_over.png');
}

.tip-button:active {
  background-image: url('images/1_pay_mm_off.png');
}
</style>
<script>
var tipButton = document.querySelector('.tip-button')
tipButton.addEventListener('click', function() {
  if (typeof web3 === 'undefined') {
    return renderMessage('You need to install MetaMask to use this feature.  https://metamask.io')
  }

  var user_address = web3.eth.accounts[0]
  web3.eth.sendTransaction({
    to: YOUR_ADDRESS,
    from: user_address,
    value: web3.toWei('1', 'ether'),
  }, function (err, transactionHash) {
    if (err) return renderMessage('Oh no!: ' + err.message)

    // If you get a transactionHash, you can assume it was sent,
    // or if you want to guarantee it was received, you can poll
    // for that transaction to be mined first.
    renderMessage('Thanks!')
  })
})
</script>
<div class="tip-button"></div>