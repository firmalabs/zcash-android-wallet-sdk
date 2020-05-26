Wallet App Threat Model
========================

This threat model is intended for curious technical users of the ECC wallet
apps as well as developers making use of the SDK in their own apps. See the
[Invariant-Centric Threat Modeling](https://github.com/defuse/ictm) for a
complete explanation of the threat modeling methodology we use; a short
summary of the methodology is:

- This document lists security invariants that the apps and SDK should
currently provide. Users and developers *should not* rely on any security or
privacy properties that are not explicitly listed here. If there's a security
or privacy property that you would like to be able to rely on, but isn't
listed here, then please raise an issue on GitHub.
- We aim to state "security invariants" in a language that end-users would
understand. If a security invariant uses technical language or involves
complicated concepts then users are unlikely to understand it, which could
lead to them being overconfident in their use of the software.

*If you are a security auditor, please try to break one of the security
invariants and think about which important security invariants might be
missing from this list!*

The security properties that the apps and SDKs can provide depend on how
powerful the adversary trying to attack the user is. This threat model's
security invariants are organized into sections for each kind of adversary.
We start with the most poweful kind of adversary, one who has completely
compromised the lightwalletd server the wallet connects to, can intercept
network traffic, install apps on the user's phone, etc., and end with the
weakest kind of adversary, one who simply knows the user's address and can
observe the public blockchain. For simplicity of explanation, the stronger
adversaries have all of the capabilities of the weaker adversaries. For each
kind of adversary, we list:

- Which security invariants we expect are satisfied against that adversary
(and all weaker ones).
- Which security invariants we know are *not* satisfied against that
adversary (and all stronger ones).

There are several weakneses in the current implementation that we beleive
most users would find counter-intuitive, or at least not expect to be the
case based on their experience using other cryptocurrency wallets. We've
highlighted those ones in bold. Brief technical details of how the adversary
could exploit the weakness are included with each one.

Let's begin with the most powerful type of adversary considered by our model.

## Lightwalletd-Compromising Adversary

**Description:** The lightwalletd service the user connects to is compromised
*or outright malicious. In addition to being in complete control over
*lightwalletd, the adversary can intercept all of the app's network traffic
*and can run code as another app on the user's phone (e.g. a fake calculator
*app). The adversary knows some addresses belonging to the user. Accidental
*reorgs happen regularly.

The security invariants we expect to be satisfied against this adversary are
all the ones we expect to be satisfied against weaker adversaries in the
section below, and in addition, that the adversary...

- can't tell what the user's current shielded balance is (aside from it being
zero when the wallet is created).
- can't learn information about the user's shielded balance over time (aside
from the assumption that it must be nonzero after they've received
transactions).
- can't cause a transaction the user is receiving to fail on the Zcash
network.
- can't learn information about the value, memo field, etc. of shielded
transactions the user receives.
- can't learn information about the value, memo field, etc. of shielded
transactions the user sends.
- can't learn who the user is sending funds to in fully-shielded transactions
as long as the other user isn't using the same lightwalletd service provider
and there is no collusion with that other service provider.
    - This is not a useful security invariant, because it is hard for users to understand.
- can't learn who the user is receiving funds from in fully-sheilded
transactions as long as the other user isn't using the same lightwalletd
service provider and there is no collusion with that other service provider.
    - This is not a useful security invariant, because it is hard for users to understand.
- can't find out one of the user's wallet addresses unless they've given it out.
- can't tell how many distinct people have sent the user funds (all transactions coming from one vs. many).
- can't tell how many distinct people the user has sent funds to (all transactions going to one vs. many).
- can't steal the user's funds.
- can't burn the user's funds or otherwise make them unspendable.
- can't make the user send funds to the wrong address.
- can't make the user think person X has sent them funds when it was actually someone different.
- can't make the funds go to someone else when someone was trying to send the user funds.
- can't take execute arbitrary code on the user's phone.
- can't make the user send the wrong amount of funds.
- can't make the user send a transaction with a memo field they did not intend.
- can't make the user send funds when they did not intend to.
- can't make it look like a payment the user received came from someone who it actually did not.
- can't make it look (to someone else) like the user is sending funds to somewhere they are not.
- can't make it look (to someone else) like the user is receiving money from somewhere that they are not.
- can't send money to the user at any address of theirs that the adversary did not already know about.
- can't cause the app to display an false official-looking message.
- can't learn any of the user's cryptographic key material (spending keys, viewing keys, seed phrase, etc.)
- can't see any of the user's wallet history when they connect to this lightwalled instance for the first time (after previously using a different one).
- can't cause the funds the user sends to someone else to be gone from their wallet but be unspendable by the recipient.

There are some known weaknesses this adversary can exploit. In addition to
all of the known weaknesses from the sections below (which also apply to this
adversary, because this one is capable of all the same things and more), the
adversary can...

- **make the user think they have (or will have) spendable funds when they don't.**
    - They can repeat a transaction to the user's wallet many times to cause the wallet to think it has more balance than it can actually spend.
- **make the wallet display the wrong memo for a given transaction.**
    - They can return the wrong note ciphertext (e.g. one from a different transaction) when the wallet requests the full one to obtain the memo field.
- **make the user think their balance is lower than it actually is.**
    - They can omit transactions destined to user, so that the user's wallet can have spendable funds that it isn't aware of.
- **make the user think a transaction they sent succeeded when it actually failed.**
    - If the user's transaction gets reorged-away or never mined, they could make it look like it was actually mined.
- **make the user think a transaction they sent failed when it actually succeeded.**
    - They could omit the transaciton, making it look like it failed when in fact it was mined on the Zcash network.
- **make the user think a transaction they received succeeded when it actually failed.**
    - Similarly, if the transaction got reorged-away or never mined, they could make it look like it was actually mined.
- **make the user think a transaction they received failed when it actually succeeded.**
    - They could omit the transaction, making it look like it failed when it in fact was mined on the Zcash network.

Let's move on to the second-most powerful kind of adversary we will consider.

## Typical Adversary

**Description:** There is a trust relationship between the user and the
lightwalletd server operator. Lightwalletd only ever provides valid
information coming from a consistent Zcash blockchain state. The information
is not guaranteed to be recent, and part of it may change (e.g. after a
reorg) and even revert to old state. The connection to lightwallletd is
protected by TLS, which we assume to be secure. The adversary (a) can
intercept all of the app's network traffic, (b) can run code as an app on the
user's phone (e.g. fake calculator app), and (c) can read (but not write to)
the lightwalletd server's private memory. The adversary knows some addresses
belonging to the user. Accidental reorgs happen regularly. **NB:** *If the
lightwalletd server gets compromised temporarily, which we should assume will
eventually happen, the security properties degrade to the column to the left
for the duration of the compromise, and possibly longer if the effects of the
attack persist.*

The security invariants we expect to be satisfied against this adversary are
all the ones we expect to be satisfied against weaker adversaries in the
section below, plus that the adversary...

- can't make the wallet display the wrong memo field for a given transaction.
- can't make the user think their balance is lower than it actually is.
- can't make the user think they have (or will have) spendable funds when they don't.
- can't make the user think a transaction they sent succeeded when it actually failed.
- can't make the user think a transaction they sent failed when it actually succeeded.
- can't make the user think a transaction they received succeeded when it actually failed.
- can't make the user think a transaction they received failed when it actually succeeded.

There are no known weaknesses that apply to this adversary specifically. All
of the known weaknesses in the sections below for weaker adveraries apply to
this one as well.

## Network- and Lightwalletd-Surveiling Adjacent-App Adversary

**Description:** The adversary can only (a) intercept all network traffic
between the app and the internet, (b) can run code as an app on the user's
phone (e.g. fake calculator app), and (c) intercept the traffic between the
lightwalletd server and the Internet. The adversary knows some addresses
belonging to the user. Accidental reorgs happen regularly. We assume defenses
are in place to detect eclipsing of the lightwalletd node.

There are no security invariants that we expect to be satisfied for this
adversary but not for the next-weaker one.

There are several known weaknesses that this kind of adversary can exploit.
The adversary can...

- **...tell *that* and *when* the user received a fully-shielded transaction.**
  - The wallet fetches the memo from lightwalletd separately, which uses
  more bandwidth, and that is visible even though the connection is
  encrypted.
- **...tell *that* and *when* the user sends a fully-sheilded transaction.**
  - The act of sending a transaction uses more bandwidth, which is visible
  even though the connection is encrypted.
- **...tell how many transactions the user has sent or received over time.**
  - For the same reasons as above.
- **...tell who the user is.**
  - The adversary knows the user's IP address, which could lead them to the user's real identity.
- **..tell where the user is.**
  - The adversary could look up the user's IP address in a geolocation database to approximate their location.
- **...learn who the user is sending/receiving funds to/from in fully-shielded transactions.**
  - When both users are using the same lightwalletd instance, even though the connections are encrypted, the adversary will be able to correlate bandwidth spikes that look like sends from one user with bandwidth spikes that look like receives from another user.
- **...determine whether or not the user's wallet owns a particular address.**
  - The adversary could send funds to that address and watch to see if there are bandwidth spikes that look like the wallet fetched a memo around the same time.
- **...tell that the user spends or receives money according to a certain pattern (e.g. recurring payments) using fully-shielded transacitons.**
  - They would observe bandwidth spikes that look like sends/receives following the same pattern.
- ...tell which of many users of the lightwalletd instance the user is.
  - If the user reconnects from the same IP address, they can usually assume it is the same user.
- ...silently prevent the user from receiving wallet security updates or security notices
  - By blocking the wallet's connection to the internet.
- ...tell when the user's wallet was created.
    - The wallet only downloads blocks starting with its birthday. By
    observing how much bandwidth gets used during the initial download, the
    adversary can determine roughly how many blocks it downloaded, and thus
    roughtly what its birthday is.
- ...tell which cryptocurrencies the user is using if the SDK is used in a multi-currency wallet.
    - The adversary would be able to see that the wallet is connecting to a lightwalletd instance, which reveals they are using Zcash.
- ...tell when the user is actively using the wallet.
    - They can see that the wallet is communicating with lightwalletd when it's in active use.
- ...cause a transaction the user sends to fail.
    - They can block the connection between the wallet and lightwalletd.
- ...make the user think outdated information (transactions, balance, etc.) is up to date.
    - By blocking the connection to lightwalletd.

These weaknesses also apply to the stronger adversaries in the sections above.

## Address-Knowing Adversary

**Description:** The adversary only knows some of the users' z-addresses and
t-addresses, and has no other special capabilities.

This is the weakest adversary in our model. The security invariants we expect
to be satisfied against this adversary are that the adversary...

- can't silently prevent the user from receiving wallet security updates or security notices.
- can't tell *that* or *when* the user has received a fully-shielded transaction.
- can't tell *that* or *when* the user sends a fully-shielded transaction.
- can't tell how many transactions the user has sent or received over time.
- can't tell precisely when the user's wallet was created.
- can't tell which cryptocurrencies the user is using if the SDK is in use in a multi-currency wallet.
- can't tell when the user is actively using the wallet.
- can't cause a transaction the user sends to fail.
- can't tell who the user is (i.e. their real name).
- can't tell where the user is.
- can't tell which of the many users of the lightwalletd instance the user is.
- can't make the user think outdated information (transactions, balance, etc.) is up to date.
- can't learn who the user is sending/receiving funds to/from in fully-shielded transactions.
- can't determine whether or not the user's wallet owns a particular address.
- can't tell that the user spends money according to a certain pattern (e.g. recurring payments) using fully-shielded transacitons.

There are several known weaknesses against this adversary. The adversary can...

- tell when the user sends a t-address transaction.
- tell when the user receives a t-address transaction
- tell what the user's current transparent balance is.
- tell that the user is using this particular wallet app.
    - differneces in note selection might distinguish it from zcashd.
- tell when the user spends shielded funds sent to them by the adversary versus someone else.
    - dust attack: send many low-value notes, which will need to be aggregated to be spent (visible on the blockchain).

These weaknesses also apply to all of the stronger adversaries in the sections above.

## Limitations

This threat model is missing important details, e.g. about:

- Adversaries that have physical access to the device the SDK is running on.
- Secure usability of the SDK's API.
- Implicit assumptions about how the SDK is being used.

These shortcomings will be addressed in future updates to the threat model.