# Troubleshooting

## Where to look first

- **The Log panel**, bottom left. Almost everything the app does reports there, including the exact
  reason a trade failed.
- **`logs\crash.log`** in the install folder. Always written, no setting needed. If a button appears
  to do nothing at all, this is where the reason is.
- **`logs\bundler.log`**, the full log panel written to a file. Off by default; turn it on in
  **Settings → Advanced → Diagnostics** if you want a record to keep.

Log lines have any URLs stripped out of them, so you can paste one somewhere without leaking your
RPC key.

---

## The app will not start

### "This folder is read only"

The app is installed somewhere it cannot write, almost always Program Files. It stops rather than
falling back somewhere else, because a fallback would split your install in two.

**Fix:** move the whole program folder somewhere you own, Desktop, Documents, or anywhere outside
Program Files, and start it again.

**Do not "run as administrator" to get past it.** That would write your wallets into a folder every
later normal launch still cannot read, making the app permanently admin only.

### It asks to set up a new wallet when you already have one

The app reads its data from its own folder. If you moved the `.exe` out of the install folder, or
copied only the shortcut, it is looking in the wrong place. Move it back next to the `settings`
folder.

### The passphrase is rejected

*"decryption failed, wrong passphrase or corrupted data"* means exactly that, and the app cannot
tell the two apart. Check caps lock and keyboard layout.

If you are certain the passphrase is right, the `seed.enc` file is damaged. Recover from your
written down 12 words: that rebuilds every wallet exactly as it was.

---

## Nothing on chain works

### "RPC is not configured, add one in Settings → RPC Endpoints"

You have not added an endpoint yet, or the one you added is not saved and enabled. See
[Getting started](getting-started.md#2-add-an-rpc-endpoint).

### Balances refresh but the "holds other tokens" dots never fill in

You will see this in the log:

```
Token holdings stopped early — the RPC endpoint is rate limiting this lookup.
Public endpoints throttle this lookup hardest — it is two calls per wallet.
A free Helius key handles a full pool: Settings → RPC Endpoints → "New here?"
```

The full token scan is two requests per wallet, around 44 on a full pool, and it is the call free
public nodes throttle hardest. The app stops rather than hammering the endpoint, keeps what it
managed to read, and leaves everything else working: **SOL balances, trading and the Focus Coin
column are unaffected.**

What you lose is the dot showing a wallet holds other tokens, and the token list you get by clicking
a wallet's name.

**Fix:** get a free Helius or QuickNode key. Or set a Focus Coin, with one set, Refresh Balances
reads SOL plus that one coin in a single request and never trips the limit.

### An action just hangs

Check the endpoint with **Test** on its row in Settings → RPC Endpoints. It probes ten methods and
gives you a verdict in about a second.

If one method fails on an otherwise good endpoint, press the **✕** next to that method in the test
popup. The app will route that one method to your next enabled endpoint instead.

---

## A launch failed

### The Launch button is greyed out

Hover it. The tooltip names the one thing left to fix. In order: no mint armed, no dev wallet, name
and symbol missing, no image, no buy set up, or a wallet below the 0.01 SOL minimum.

The last one catches people out: **a launch is one atomic bundle, so a single order below the
minimum stops all of them.**

### "insufficient funds for rent"

A wallet does not have enough SOL to open the accounts a first buy needs. A wallet buying a
brand new coin spends roughly **0.007 SOL on account setup before any of its money reaches the
coin**, its own rent exempt minimum, the transaction fee, a token account, a wrapped SOL account,
and a one off pump.fun account it pays once ever.

**Fix:** fund the wallet with more SOL. Do not lower the sell buffer below 0.0069 in `config.toml`
to squeeze more in, that is what the buffer is protecting.

Because the launch is atomic, one wallet hitting this kills the whole launch, not just that wallet.

### The bundle did not land

Nothing was created and nothing was charged. The usual causes:

- **The tip was too low.** Raise the Jito tip and try again, the app shows you the current floor.
- **The blockhash went stale** because the launch sat too long between building and sending. Just
  retry.
- **A transaction in the bundle would have failed.** The log says which, and why.

Your form, your buy selection and your mint address are all kept, because the next step is usually
a retry.

### "this RPC endpoint returned no program logs"

Your endpoint stripped the on chain error detail out of its reply, so the app can tell you a
transaction failed but not why. That is a fact about the endpoint, not about your transaction.
Retry on a different endpoint and the real reason appears.

### "simulateBundle" is not available

You will see two lines in the log saying the endpoint does not support bundle simulation, and the
launch carries on. **That is not an error.** Bundle simulation is a preflight nicety that only
some endpoints offer. A Jito bundle is atomic, so a bundle that would have failed simply never
lands, and costs you nothing.

---

## A sell did not fire, or failed

### "Max 5 sell jobs already running, stop or remove one first"

Five is the limit across every strategy, and market cap positions count too. Stop or remove a
finished one.

### "{wallet} already has an active {strategy} job for this coin, skipped"

You cannot run two strategies against the same wallet and the same coin at once, they would sell
the same tokens twice. Stop the existing job first.

### "Live mcap for the focus coin isn't available yet, wait for the next poll"

You are trying to add a ladder level before the app has read a market cap for the coin. Wait for the
*"Current mcap"* line to show a figure. If it never does, your RPC endpoint is the problem.

### A wallet was excluded from the batch

Two common reasons, both logged with the wallet's name:

- **Its reserve is bigger than its balance.** A job that must keep back more than the wallet holds
  can never sell anything, so it is not created.
- **Its live balance could not be read.** The rest of the batch still starts.

### The sell reverts

Raise **Slippage**. On a fast moving coin, or when several wallets sell at once against the same
reserves, the later fills land under the price they were quoted from. Raising the **Priority fee**
helps them land sooner, which is a different fix for the same symptom.

### Smart Sell never receives a price

Smart Sell is the one part of the app on a live WebSocket stream. If your endpoint's free tier has
no WebSocket access, it will not run. Helius and QuickNode include it free; some others do not.
Check with **Test** on the endpoint's row.

### My sell jobs disappeared

Sell jobs run inside the app. **Closing the window stops them and nothing resumes on restart.** If
you need something to run unattended, leave the app open and unlocked.

---

## Other things

### The Update button refuses

Something is running, a launch in flight, a sell job, an armed Bunch run. The update waits rather
than interrupting it. Let it finish, or stop it, then update.

### "This version can no longer trade"

A minimum version has been set and this build is below it, so **buying and selling are switched
off** until you update. It is not a fault and nothing has happened to your funds.

Everything else still works, your wallets, your balances, Refresh Balances, Sweep to Funder,
Withdraw SOL and key export are all unaffected. Only the two trading actions are disabled.

The notice carries an **↑ Update required** button. Press it and the app downloads the new version,
installs it and restarts. If that fails for any reason, get the installer from the
[releases page](https://github.com/Ashkiln/pumpfun-launcher/releases/latest) and install over the
top, your wallets are in the `settings` folder and an install does not touch them.

### The app asks you to reinstall

**Do it.** Download a fresh copy from the
[releases page](https://github.com/Ashkiln/pumpfun-launcher/releases/latest) and install it over
your existing folder. Your wallets live in `settings` and are not touched by an install.

### A button does nothing at all

Read `logs\crash.log`. Frontend errors are written there directly and never reach the Log panel, so
a control that appears dead usually has its reason sitting in that file. Include it if you report
the problem.

### Something is still wrong

Open a thread in
**[GitHub Discussions](https://github.com/Ashkiln/pumpfun-launcher/discussions)**. The useful things
to include:

- The app version (Settings → About).
- The relevant lines from the Log panel, and from `logs\crash.log` if a control did nothing.
- Which RPC provider you are on.

**Never post your recovery phrase, your passphrase, a private key, or an unredacted RPC URL with a
key in it.** Nobody legitimate will ask you for any of them.
