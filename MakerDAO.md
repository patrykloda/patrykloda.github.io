# MakerDAO Dai Technical Deepdive

MakerDAO is currently the #2 DeFi protocol by TVL, and this is with it only opertaing on the Ethereum blockchain. The main reason I chose to look into MakerDAO before Lido (the #1 protocol by TVL) is due to my personal interest in DAI, which is the #1 decentralised stablecoin that's soft-pegged to the US dollar. MakerDAO has also recently start to deploy smart contracts for their new version of the protocol, Multi Collateral Dai (MCD) which now allows the protocol to accept any Ethereum-based asset as collateral to generate Dai (as long as it is approved by MKR holders).

The main use cases for the protocol is to allow the borrowing/minting of Dai by providing ETH or approved ERC20 tokens. Users are also able to deposit their Dai into the Dai Savings Rate vault, allowing them to gain interest on their locked assets. 

## Why bother locking up collateral to borrow stablecoins?

Many users who are new to DeFi might be wondering why bother locking up one asset, to then borrow a different asset (and borrow less USD value than they are locking up!). The main use case that I've used similar protocols for (mainly compound due to the smaller minimum borrow amounts) is to purchase thing IRL using fiat, without having to sell my ETH. The benefit of this is that if the price of ETH goes up before you buy the ETH back, you'll end up paying more for the same amount of ETH, whilst borrowing against your ETH, mean you only have to pay the borrowing fee (which on MakerDAO is currently below 1.5% annually, however for small borrowing amounts the transaction fees of approving and locking up assets can be above 5%).

## Minting Dai

As mentioned before, the whole protocol is centered around locking up collateral, minting Dai and charging fees on the borrowed Dai (and generating interest to users who lock up their Dai). Dai is an ERC20 token that mostly complies to the standard, however it has a few differences that will be outlined in the next section. For most users the best way to obtain Dai is by purchasing it on a Dex or Cex, however the way new Dai is minted is outlined directly below.

To mint new Dai, a user can interact with the MakerDAO CDP manager, which is the public facing interface contracts that allows anyone to interact with the MakerDAO system. In order to fund the vault that will be created, you need to first approve the MCD adapters for the respective ERC20 tokens that will be used to fund the vaults. These MCD adapters will be interacted with through the above mentioned CDP manager

1. Call `CDP_MANAGER.open(bytes32 ilk, address usr)` to open an empty vault. This function will return `uint cdpId` which will be the id of the vault. 
   bytes32 ilk: Is the underlying asset that will be used within the vault.
   address usr: Is the address that will own the vault.

2. Call `CDP_MANAGER.urns(uint cdpId)` to need to retrieve the `urn` address.
   address urn: is where your collateral balance and outstanding stablecoin debt is registered.

3. Call `ERC20.approve(address MDC_JOIN_ASSET, uint dink)` to approve MDC_JOIN_ASSET's allowance to the amount you want to provide as collateral.
   address MDC_JOIN_ASSET: Is the adapter for the specific collateral token you're using.
   uint dink: The amount of collateral you want to lock up.

4. Call 'MCD_JOIN_ERC20_A.join(address urn, uint dink)` to send `dink` amount of collateral to the urn.

5. Call 'CDP_MANAGER.frob(uint cdpId, int dink, int dart)` to lock up `dink` collateral into your `urn` vault and create `dart` Dai.
   int dart: The value of Dai you want to borrow against your collateral.

6. Call `CDP_MANAGER.move(uint cdpId, address ETH_FROM, uint rad)` to move `rad` Dai from the `urn` to `ETH_FROM`.
   address ETH_FROM: Your personal address.
   uint rad: The amount of Dai you want to move from your `urn`. It is a 45 decimal high precision uint.

7. Call `MCD_VAT.hope(address MCD_JOIN_DAI)` to approve `MCD_JOIN_DAI` adapter in `MCD_VAT`.
   address MCD_JOIN_DAI: The adapter that manages the minting and burning of Dai.

8. Finally call `MCD_JOIN_DAI.exit(address ETH_FROM, uint dart)` to withdraw your Dai to your personal address.

### Dai differences from the ERC20 standard

- The `transferFrom` function allows for unlimited approvals by setting an addresses allowance to the maximum uint256 values, leading to an unlimited approval spending cap.
- Dai also uses aliases for different `transferFrom` calls:

   `push` for `transferFrom(msg.sender, usr, amount)` pushing amonunt from msg.sender to the usr.
   `pull` for `transferFrom(usr, msg.sender, amount)` pulling amount from usr to the msg.sender.
   `move` for `transferFrom(src, dst, amount)` moving amount from src to dst.
   Note: `pull` and `move` both require approvals to allow for msg.sender to initate these transfers.

- `permit` allows users to sign messages which can be relayed by another party to submit their approval. The benefit of this it that it allows users to sign approvals without having to send a transaction on the blockchain (meaning there are no fees), this allows users who only have access to Dai (and have no ETH to pay for a transaction) to sign a permit and a relayer can relay the transaction for them, taking a % fee for relaying their transaction (the fee would consists of the Eth gas cost and the relayer service fee). This can greatly increase the user experience, as users who only want to hold Dai can do so if they utilise permits.


_Note: Most of the information on this blog post has been taken from the MakerDAO docs and their github discussions. I have tried to add extra information I have found from other sources, to make it easier the blog easier to read and understand._