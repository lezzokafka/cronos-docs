# Band Protocol Price Feed on Cronos

## Introduction

Band Protocol is a secure, blockchain-agnostic decentralized oracle ([more detail here](https://docs.bandchain.org)).
It features a blockchain of its own (BandChain) that does the heavy-lifting job of pulling data from external sources,
aggregating all the results, and creating a verifiable proof of data integrity to be used in other blockchain
applications -- all with a secure crypto-economic which guarantees data availability and correctness.

Developers building on Cronos can now utilize Band’s decentralized oracle infrastructure. With Band’s oracle,
they now have access to various cryptocurrency price data to integrate into their applications.

## Supported Tokens

Currently, the following token symbols are supported. Going forward, this list will continue to expand based on
the developer needs and community feedback.

| Token Name      | Symbol |
| --------------- | ------ |
| Tether          | USDT   |
| USD Coin        | USDC   |
| Dai             | DAI    |
| Crypto.com Coin | CRO    |
| Ethereum        | ETH    |
| Wrapped Bitcoin | WBTC   |

### Price Pairs

The method provided below supports price query with any denomination as long as the base and quote symbols are supported
in the list above. For example,

- `CRO/USD`
- `WBTC/ETH`

## Querying Prices

Developers are able to query prices from Band’s oracle through Band’s `StdReference` smart contract on the Cronos chain.

### Solidity Smart Contract

To query prices from Band’s oracle through EVM smart contracts, the contract looking to use the price
values should reference Band’s `StdReference` contract. This contract exposes `getReferenceData` and
`getReferenceDataBulk` functions.

`getReferenceData` takes two strings as the inputs, the base and quote symbol respectively. It then queries the
`StdReference` contract for the latest rates for these two tokens, and returns a `ReferenceData` struct, shown below.

```solidity
struct ReferenceData {
    uint256 rate; // base/quote exchange rate, multiplied by 1e18.
    uint256 lastUpdatedBase; // UNIX epoch of the last time when base price gets updated.
    uint256 lastUpdatedQuote; // UNIX epoch of the last time when quote price gets updated.
}
```

`getReferenceDataBulk` instead takes two lists, one of the base tokens, and one of the quotes. It then proceeds to
similarly query the price for each base/quote pair at each index, and returns an array of `ReferenceData` structs.

For example, if we call `getReferenceDataBulk(['CRO','WBTC','ETH'], ['USD','ETH','DAI'])`, the returned `ReferenceData`
array will contain information regarding the pairs:

- `CRO/USD`
- `WBTC/ETH`
- `ETH/DAI`

### Example Usage

The contract code below demonstrates a simple usage of the new `StdReference` contract and the `getReferenceData`
function.

```solidity
pragma solidity 0.8.3;
pragma experimental ABIEncoderV2;

interface IStdReference {
    /// A structure returned whenever someone requests for standard reference data.
    struct ReferenceData {
        uint256 rate; // base/quote exchange rate, multiplied by 1e18.
        uint256 lastUpdatedBase; // UNIX epoch of the last time when base price gets updated.
        uint256 lastUpdatedQuote; // UNIX epoch of the last time when quote price gets updated.
    }

    /// Returns the price data for the given base/quote pair. Revert if not available.
    function getReferenceData(string memory _base, string memory _quote)
        external
        view
        returns (ReferenceData memory);

    /// Similar to getReferenceData, but with multiple base/quote pairs at once.
    function getReferenceDataBulk(string[] memory _bases, string[] memory _quotes)
        external
        view
        returns (ReferenceData[] memory);
}

contract DemoOracle {
    IStdReference ref;

    uint256 public price;

    constructor(IStdReference _ref) public {
        ref = _ref;
    }

    function getPrice() external view returns (uint256){
        IStdReference.ReferenceData memory data = ref.getReferenceData("CRO","USD");
        return data.rate;
    }

    function getMultiPrices() external view returns (uint256[] memory){
        string[] memory baseSymbols = new string[](2);
        baseSymbols[0] = "CRO";
        baseSymbols[1] = "ETH";

        string[] memory quoteSymbols = new string[](2);
        quoteSymbols[0] = "USD";
        quoteSymbols[1] = "USD";
        IStdReference.ReferenceData[] memory data = ref.getReferenceDataBulk(baseSymbols,quoteSymbols);

        uint256[] memory prices = new uint256[](2);
        prices[0] = data[0].rate;
        prices[1] = data[1].rate;

        return prices;
    }

    function savePrice(string memory base, string memory quote) external {
        IStdReference.ReferenceData memory data = ref.getReferenceData(base,quote);
        price = data.rate;
    }
}
```

## Contract Addresses

| Network        | StdReference Contract Address                                                                                                                               |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cronos Testnet | [`0xE0d9A9ABEaa8Ec7a1184c6Eeb2ecb641Ba214644`](https://cronos.crypto.org/explorer/testnet3/address/0xE0d9A9ABEaa8Ec7a1184c6Eeb2ecb641Ba214644/transactions) |
