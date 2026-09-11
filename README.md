# mousecoinrh

A real connectome, simulated neuron by neuron, driving the
[pons launchpad](https://www.ponsfamily.com/launchpad) on Robinhood Chain —
presented as a **mouse**.

165,122 neurons. 10,228,000 signed synaptic connections. Every one of them
measured from an actual male *Drosophila melanogaster* by electron microscopy —
not invented, not sampled from a distribution, not a neural network "inspired
by" a brain.

> **On the mouse, stated plainly.** This is a re-brand of the original
> [fruitflydev/flycoinrh](https://github.com/fruitflydev/flycoinrh). The
> connectome under the hood is unchanged: it is the **FlyEM Drosophila male CNS
> v1.0** (© HHMI Janelia FlyEM, the Cambridge Connectomics Group and Google
> Research, CC-BY). "Mouse" is the brand, the agent and the on-screen animal —
> the neurons, the anatomy and the data are a fruit fly's. Wherever this README
> says "the mouse", the connectome the mouse runs on is a *Drosophila
> melanogaster*. The frontend shows the descending neurons under mouse-flavoured
> labels (M1, MLR, CST, PAG); the names in the data are DNa02, DNa01, MDN and
> DNp09.

## What it actually does

Press START and a real Chromium opens ponsfamily.com/launchpad. Its screenshots
are sampled through the **892 retinotopic hex columns** into L1 and L2 — the
lamina monopolar cells that are the direct postsynaptic targets of
photoreceptors R1–R6. 165,122 neurons integrate. The cursor comes back out of
the descending neurons the animal actually walks with:

| neuron | what it does in the fly | what it does here |
|---|---|---|
| **DNa02** left vs right | steering — the fly turns by left/right asymmetry | cursor x |
| **DNa01** | forward walking | cursor y |
| **MDN** | the Moonwalker descending neuron — walking backwards | reverse |
| **DNp09** | stopping | the click |

Then it connects the wallet, accepts the launchpad's terms, uploads the token
image, fills the form — name, ticker, description and the `x.com/` handle —
picks the paired asset out of a 57-item list of tokenised equities, opens
**Advanced**, sets the creator tax, and launches.

## The mouse is loose on the internet

`roam.py` gives it a browser and no instructions. A page is screenshotted,
sampled through the 892 hex columns, and 165,122 neurons decide where the
cursor goes. If a click lands on a link, the mouse is somewhere new. When its
forward drive pushes past the bottom of the window the page scrolls, so it
walks down a page the way it walks across one.

The live view is served by `site/` — a static page plus one serverless function
that proxies the chain (see below). It shows the page the mouse is looking at,
the neurons firing, and what its descending neurons are doing, all read out of
the running simulation.

```bash
py roam.py            # http://localhost:4660, no start button
```

There is no start and no stop. It roams when the process is up, and if a run
dies it waits six seconds and starts another life.

### The rails, and why each one is there

A random clicker on the open internet, streamed publicly, from a machine that
also holds a funded wallet, is a genuinely bad idea unless it is fenced.

- **No wallet.** This browser gets no key, no provider and no extension. The
  roaming browser and the launching browser share nothing but the brain.
- **No keyboard.** The mouse cannot type, so it cannot fill a field, write a
  message or answer a prompt.
- **Every click is checked before it lands.** Anything that reads as a submit,
  an upload, a payment or a sign-in is vetoed, and the veto count is on screen.
- **A domain fence.** The first thing built was a keyword blocklist; the first
  thing tested was `p0rn.com`, which walked straight through it. Keyword
  filters do not hold, so the real control is an allowlist of link-rich
  domains. `MOUSE_ROAM_OPEN=1` removes the fence and should not be left on for
  an unattended public stream.
- No downloads, no popups, no dialogs, and a hop budget so a dead end does not
  become a permanent home.

### What the readouts actually are

Nothing on that page is decoration. `mousesim` was extended to report it:

| on screen | what it is |
|---|---|
| neurons firing | a per-neuron bit set on every spike in the window |
| spikes/sec | total spikes divided by simulated time |
| membrane | mean membrane potential at the end of the window |
| visual / motor | firing neurons intersected with L1/L2 and the DN groups |
| the scatter | those neurons at their **measured soma coordinates** |
| DNa02 / DNa01 / MDN / DNp09 | the recorded rates already driving the cursor |

`MousePilot.step(detail=True)` returns the extra readout as a fifth value, so
every existing caller still unpacks four and is unaffected.

The site itself is in `site/` — a static page on Vercel plus one serverless
function that proxies the chain, because the public Robinhood node
intermittently answers `Access-Control-Allow-Origin: *,*`, which browsers
refuse. The mouse publishes its latest frame and summary to a public object
store, so nothing about the feed needs the machine it runs on to be reachable.


## Why Robinhood Chain is the better half of this project

pump.fun's backend answers `401 Unauthorized` to an injected wallet, so its own
Create button can never complete a launch — the Solana repo has to bypass the
site entirely and mint through a signed transaction.

The pons launchpad **accepts the injected EIP-1193 wallet directly**. It
auto-connects with no modal, `eth_chainId` returns `0x1237`, `personal_sign`
round-trips. So here the site's own button is the real path.

## Not launched yet

The launch path is proven — the original project drove this exact flow to a
mined block five times. **This fork has not launched a token.** The token and
wallet addresses in `site/` are zero-address placeholders:

- `site/server/main.py` — `MOUSE_WALLET`, `MOUSE_TOKEN`, `MOUSE_TOKEN_BLOCK`
- `site/api/state.js` — `MOUSE_WALLET`, `MOUSE_TOKEN`, `MOUSE_TOKEN_BLOCK`
- `site/web/index.html` — `const TOKEN`, `const BIRTH`

After your first launch, set those to the deployed contract, its birth block,
and the wallet that signed it. Until then the live panel reads `—` rather than
pretending to point at a token.

### The paired asset

pons pairs a new token against something already on Robinhood Chain, and what
is on Robinhood Chain is mostly tokenised equities — the menu is **57 assets**:
NVDA, SPCX, GOOGL, TSLA, GME, AAPL, SPY, and so on down to gold, oil and
Treasuries. The default is ETH. `set_pair_asset()` changes it, and the choice
is real: graduation goes from `4.2 ETH` to `24.2 GOOGL`.

The launch fee stays in ETH either way — the page says `GOOGL pair, ETH 0.0005
due` — so pairing against an equity needs none of that equity in the wallet.

```bash
MOUSE_RH_PAIR=GOOGL MOUSE_RH_TAX=2 py rhlive.py --port 4651
```

Two things about that menu are worth knowing if you automate it. It is a 262px
window onto a 2,064px list with **its own** scrollbar, so page scrolling cannot
reach inside it — `smooth_scroll_in()` eases the container's own `scrollTop`,
because `scrollIntoView` teleports and the list would cut from ETH to GOOGL
between two frames. And Playwright's `get_by_role("button", name="GOOGL")`
never resolves against it; the rows have to be found by `textContent` and
clicked at coordinates.

### Step 06: it learns now

`mushroom.py` is the one place in this project a weight is allowed to move,
and it moves where the fly's weights actually move — the Kenyon cell to MBON
synapse, under dopamine.

The rule is the measured one. A Kenyon cell active shortly before a
dopaminergic neuron fires has **that synapse depressed**, not strengthened.
Learning in a fly is subtraction: the mushroom body starts able to drive every
response and experience carves away the ones that did not pay. So there is no
potentiation here, only depression with a floor and a slow drift back toward
baseline standing in for forgetting.

Which MBONs count as reward-side and which as punishment-side is **not
hardcoded from a table**. For each MBON, total PAM input weight is compared
against total PPL1 input and the stronger wins. That split puts MBON01, 02 and
03 on the reward side and MBON04, 10 and 11 on the punishment side, which is
where the literature puts them — a good sign it is finding real structure
rather than noise.

```
44,042 KC->MBON synapses     27,939 reward-side     14,349 punish-side
```

Measured, with controls — twenty rewarded encounters with one view:

| population | change | expected |
|---|---|---|
| reward-side MBONs | **−6.0%** | depressed, it was the addressed compartment |
| punishment-side MBONs | −0.9% | ~0, never addressed |
| Kenyon cells | +0.3% | ~0, upstream of the synapse that changed |

**The reward signal is not real, and the module says so in as many words.** A
fly is rewarded by sugar, not by reaching a web page. Novelty stands in for it
here, and that is a modelling choice made by a person — the fly has no say in
it. The circuit, the plasticity site and the direction of the rule are the
parts that are real.

### Where it roams, and where it does not

The open web, plus the chain it launches its own token on: Wikipedia,
Wikimedia Commons, Wikisource, Hacker News, Project Gutenberg, Open Library,
xkcd, arXiv — and the pons launchpad and Blockscout.

Four places were tried and dropped, all for the same reason: a real browser
gets nothing usable from them.

| tried | what a headless browser actually gets |
|---|---|
| Google | 335 chars behind an "unusual traffic" wall |
| X | 623 chars of "Continue with phone" |
| old.reddit | blocked outright |
| archive.org | 0 chars — paints nothing headless, even at `networkidle` |

Open Library stands in for the Internet Archive, since it renders. The list is
what it is because a retina needs a page that exists, not because those sites
were uninteresting.

Keeping it on two domains was tried too, and the failure was quieter: 93 pages
and 110 clicks in 49 minutes across **6 unique pages**. It was moving the whole
time and going nowhere.

Trade controls are blocked and vetoed now that it roams a launchpad. It has no
wallet and a trade is impossible, but the claim was that every click is
checked, and it did once reach a "Buy token" page before that was tightened.

### Why there is no object store in the path

The public feed used to push a frame and a summary to a blob store twice a
second. That suspended the store on operation count — thirteen megabytes held,
every read answering `403` — and took the live feed down with it. Before that
it had a subtler problem: the CDN answered `X-Vercel-Cache: HIT` with an `Age`
of twenty seconds on a fixed pathname no matter what cache headers went with
the upload, because public blobs are treated as immutable.

So the mouse opens a cloudflared quick tunnel instead and the socket carries
frames, telemetry and events for free. The only thing published anywhere is
where the tunnel is — `site/web/live.json`, written when the address changes,
which the page reads. Quick tunnel addresses are random and change every run,
so nothing is hardcoded.


### Waiting for the chain, not for the click

A launch that had already succeeded looked exactly like a hang. The run loop
broke as soon as the page *asked* for a signature, so the rig declared itself
done about two seconds later — with the launchpad still showing "Confirming",
the recording cut, and the token appearing on-chain a few seconds after
everything had stopped.

The loop now waits for the receipt, on camera, and `send_transaction` takes
`wait_receipt=False` so the page gets its hash immediately: it is blocked on
that call and cannot render its own confirming state until the hash returns.

Then it ends where pump.fun would. pons does **not** redirect after a launch —
it leaves you on the empty create form — so `show_coin_page()` reads the token
address out of the receipt logs (the contract that answers `name()` and
`symbol()`), opens that token's page, accepts the terms gate that navigating
re-arms, and scrolls down it. The last thing on screen is the coin.

### The socials field

The X handle is `input[placeholder="handle"]`, `aria-label="X profile handle"`,
behind an `x.com/` prefix; Telegram sits next to it as `community`. Neither has
a name or an id, so the placeholder is the only stable handle on them. It is
typed like every other field and set by `MOUSE_RH_X`.

It is also optional on the form, and the rig treats it that way — a missing
field logs and the run continues rather than dying on a selector.

> The default is `elonmusk`, which is a **test value**. It has only ever been
> typed in dry runs. Putting a real person's handle on a live token presents
> that token as theirs, which is impersonation and gets both the token and the
> creator wallet flagged. Set `MOUSE_RH_X` to something you own before any live
> launch, or clear it.

### Dark mode was a flip, not a set

`go_dark()` clicks the site's theme *toggle*. That turned `/create` dark, and
then the coin page — which the site already remembered as dark — got flipped
back to **light** for the closing shot. It now measures the body background's
luma first and only flips when it has to, so the run both starts and ends dark.

### The bug that made the first attempt look like a success

The first live run reported a signed transaction and then nothing: the balance
never moved. `hexbytes >= 1.0` changed `.hex()` to return the raw hex *without*
the `0x` prefix, so `eth_sendRawTransaction` rejected the payload — and because
the failure came back through `page.expose_function`, the launchpad swallowed
it and the page just sat there. `send_transaction` now re-prefixes, logs a
rejection loudly, and polls for the receipt so a silent failure is not
possible.

## Check it yourself

**The chain.** Robinhood Chain is an Arbitrum Nitro L2 — chain id **4663**, RPC
`https://rpc.mainnet.chain.robinhood.com`, gas in ETH at about 0.13 gwei.

**The wallet.** `py rhwallet.py new` generates a secp256k1 keypair, writes the
secret straight into gitignored `.env`, and prints only the address. It is never
printed, never returned, never passed through a chat window.

**The transaction path.** `py rhdryrun.py` exercises every step except the
broadcast: chain id, nonce, gas price, a real signed transaction, and — the
part that matters — recovering the signature and checking it against the
wallet's own address. Nothing is broadcast; there is no code path in that file
that can send.

```
chain id        4663 ok
nonce           0
gas price       0.1358 gwei
signed tx       110 bytes
recovered from  <your address>  MATCHES
estimateGas     21000 units
```

**The connectome.** CC-BY, from a public bucket, no account and no key:

```
https://storage.googleapis.com/flyem-male-cns/v1.0/connectome-data/
  body-annotations-male-cns-v1.0-minconf-0.5.feather      14 MB
  body-neurotransmitters-male-cns-v1.0.feather            42 MB
  connectome-weights-male-cns-v1.0-minconf-0.5.feather   1.1 GB
```

`py build_graph.py` turns those into 165,122 traced neurons and 10,228,000
signed edges. If your numbers differ from mine, one of us has a bug.

## What is NOT real, stated plainly

- **The mouse does not fill the whole form.** It reliably gets the description,
  often the ticker, rarely the name, and does not navigate to the launch button
  on its own. The rig completes whatever it misses, and the on-screen log says
  which fields were which — `by MOUSE: …  ·  by rig: …`.
- **Light mode breaks it.** The retina is a luminance map and every bit of
  tuning it has was done against dark UI. On the launchpad's default light theme
  it filled **0 of 3** fields; in dark mode, **2 of 3**. The rig switches the
  theme before it starts, and that gap is the clearest evidence its vision is
  doing real work.
- **The idle animation is decoration.** During a run every dot is a neuron at a
  measured soma coordinate. While idle it is a mouse-shaped scatter, and the
  panel label changes to say so.
- **The launch is one signature, and the rig arms it.** The launchpad asks for
  a single `eth_sendTransaction`; there is no second approval and no separate
  ERC-20 allowance. But the rig is what presses **Confirm** in the dialog, not
  the mouse — the mouse's contribution ends at the form and the launch button.
- **The mouse does not choose the paired asset.** It cannot read `GOOGL` at 892
  columns; picking a row out of a 57-item list is the rig following
  `MOUSE_RH_PAIR`. The same goes for the creator tax and the X handle.
- **The connectome is a fruit fly's.** The mouse identity is a shell. The
  neurons, anatomy and measurements are the FlyEM Drosophila male CNS, kept
  intact and re-branded.

## Running it

```bash
py rhwallet.py new              # create the wallet, then fund it (~0.002 ETH)
py rhdryrun.py                  # prove the signing path, spend nothing
py rhlive.py                    # http://localhost:4651, press START
py record.py --port 4651        # record the run to build/recordings/
```

A launch costs about **0.001 ETH** all in, so ~0.002 ETH in the wallet is
enough for a first one with room for the gas estimate to be wrong.

Two flags gate everything, both in `.env`, both off by default:

- `MOUSE_ALLOW_BROWSER=1` — required before a browser will open against a real
  site at all. A button in a web page is not a strong enough guard for that.
- `MOUSE_RH_LIVE=1` — required before any transaction is signed. Funding the
  wallet does not, on its own, arm anything.

## Fork it

```bash
git clone https://github.com/Cryptosheep/mousecoinrh
cd mousecoinrh
pip install -r requirements.txt
python -m playwright install chromium

# the connectome itself - 1.1 GB, CC-BY, no account and no key
# (URLs under "Check it yourself" below), into data/
py build_graph.py            # -> build/graph.npz, 165,122 neurons

cp .env.example .env         # then set MOUSE_ALLOW_BROWSER=1
py roam.py                   # http://localhost:4660
```

MIT for the code. The connectome is **not ours to license** and stays CC-BY
wherever it goes — keep the attribution, it is the whole reason any of this is
real.

Things worth pointing it at that we have not:

- **A different readout.** The cursor comes out of DNa02, DNa01, MDN and DNp09
  because those are what the fly walks with. Nothing says the output has to be
  a cursor.
- **The olfactory channel.** 2,635 ORNs across 53 receptor types are sitting
  there unused. cVA through ORN_DA1 drives pC1 at 222 Hz untrained, so the
  pathway works — it just has nothing plugged into it.
- **A real reward.** Ours is novelty, which is invented. Anything measurable
  and honest would be better.
- **Somewhere else entirely.** A game, a robot, a microscope. The brain does
  not know it is on a launchpad.

If you build something with it, open an issue — we would rather see it than
not.


## Credits

Connectome data © HHMI Janelia FlyEM, the Cambridge Connectomics Group and
Google Research, released CC-BY. Simulation approach after Shiu et al. 2024 and
Lappalainen et al. 2024. Not affiliated with any of them, nor with pons or
Robinhood.
