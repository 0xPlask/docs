---
id: liscense-modifications
title: License Modifications
---

## Licensing

Please note that Uniswap V3 is under [BUSL license](https://github.com/Uniswap/v3-core#licensing) until the Change Date, currently 2023-04-01. Exceptions to the license may be specified by Uniswap Governance via Additional Use Grants, which can, for example, allow V3 to be deployed on new chains. Please follow the [Uniswap Governance process](https://gov.uniswap.org/t/community-governance-process/7732) to request a DAO vote for exceptions to the license, or to move up the Change Date.

License changes must be enacted via the [ENS domain](https://ens.domains/) uniswap.eth, which is controlled by Uniswap Governance. This means (among other things) that Governance has the power to associate arbitrary text with any subdomain of the form X.uniswap.eth. Modifications of the Change Date should be specified at v3-core-license-date.uniswap.eth, and Additional Use Grants should be specified at v3-core-license-grants.uniswap.eth. The process for associating text with a subdomain is detailed below:

<details>
<summary> ENS Subdomain Details & Process </summary>

If the subdomain does not already exist which can be checked [here](https://app.ens.domains/name/uniswap.eth/subdomains), the [`setSubnodeRecord`](https://docs.ens.domains/contract-api-reference/ens#set-subdomain-record) function of the ENS registry should be called with the following arguments:

- `node`: `namehash('uniswap.eth')` (`0xa2a03459171c76bff45817330c10ef9f8af07011a33005b73b50189bbc7e7132`)
- `label`: `keccak256('v3-core-license-date')` (`0xee55740591b0fd5d7a28a6edc49567f6ff3febbe942ec0e2fa49ee536595085b`) or `keccak256('v3-core-license-grants')` (`0x15ff9b5bd7642701a10e5ea8fb29c957ffda4854cd028e9f6218506e6b509af2`)
- `owner`: [`0x1a9C8182C09F50C8318d769245beA52c32BE35BC`](https://etherscan.io/address/0x1a9c8182c09f50c8318d769245bea52c32be35bc), the Uniswap Governance Timelock
- `resolver`: [`0x4976fb03c32e5b8cfe2b6ccb31c09ba78ebaba41`](https://etherscan.io/address/0x4976fb03c32e5b8cfe2b6ccb31c09ba78ebaba41), the public ENS resolver.
- `ttl`: `0`

2. Then, the [`setText`](https://docs.ens.domains/contract-api-reference/publicresolver#set-text-data) function of the public resolver should be called with the following arguments:

- `node`: `namehash('v3-core-license-date.uniswap.eth')` (`0x0505ec7822d61b4cfb294f137d1a7f0ceedf162f555a4bf2f4be58a07cf266c5`) or `namehash('v3-core-license-grants.uniswap.eth')` (`0xa35d592ec6e5289a387cba1d5f82be794f495bd5a361a1fb314687c6aefea1f4`)
- `key`: A suitable label, such as `notice`.
- `value`: The text of the change. Note that text may already be associated with the subdomain in question. If it does, it can be reviewed at the following URLs for either [v3-core-license-date](https://app.ens.domains/name/v3-core-license-date.uniswap.eth/details) or [v3-core-license-grants](https://app.ens.domains/name/v3-core-license-grants.uniswap.eth/details), and appended to as desired.

Note: [`setContentHash`](https://docs.ens.domains/contract-api-reference/publicresolver#set-content-hash) may also be used to associate text with a subdomain, but `setText` is presented above for simplicity.

These contract function calls should then be encoded into a governance proposal, and approved by Uniswap Governance.

</details>

## Proposals

Proposals are submitted to Uniswap Governance via the [governance contract](https://etherscan.io/address/0x8d12a197cb00d4747a1fe03395095ce2a5cc6819#readContract). NPM packages for consuming the contract ABI, and details on previous versions, are available [here](https://docs.uniswap.org/protocol/concepts/governance/overview).

<details>
    <summary> Governor Bravo #propose Parameters </summary>

Deployment reference available [here.](https://etherscan.io/address/0x53a328f4086d7c0f1fa19e594c9b842125263026#code)

```solidity
/**
    * @notice Function used to propose a new proposal. Sender must have delegates above the proposal threshold
    * @param targets Target addresses for proposal calls
    * @param values Eth values for proposal calls
    * @param signatures Function signatures for proposal calls
    * @param calldatas Calldatas for proposal calls
    * @param description String description of the proposal
    * @return Proposal id of new proposal
    */
function propose(
    address[] memory targets,
    uint[] memory values,
    string[] memory signatures,
    bytes[] memory calldatas,
    string memory description
) public returns (uint)

```

</details>

## Populating Proposal Calldata

Below is an example of using a scripting environment to generate a proposal. This is for educational purposes only - that example assumes access to a private key with a sufficient amount of delegated UNI to submit a proposal, which may be an insecure practice. There are several ways to generate a proposal transaction and submit it to Ethereum; this example should only be used for reference and not in production.

<details>
<summary> Populating `Propose` Calldata </summary>

```typescript
import { ethers } from "hardhat";
import { waffle } from "hardhat";
import { Contract } from "ethers";
import {
  GOVERNOR_BRAVO_ABI,
  ENS_REGISTRY_ABI,
  ENS_PUBLIC_RESOLVER_ABI,
} from "./utils";
import { namehash } from "@ethersproject/hash";
import { keccak256 } from "@ethersproject/keccak256";
import { utils } from "ethers";
import { Interface } from "@ethersproject/abi";
import "hardhat";

const { provider } = waffle;

// get the governor bravo contract
const governorBravoAddress = "0x408ED6354d4973f66138C91495F2f2FCbd8724C3";
const governorBravo = new Contract(
  governorBravoAddress,
  GOVERNOR_BRAVO_ABI,
  provider
);

const NODE_TOP_LEVEL: string = namehash("uniswap.eth");
const LABEL: string = keccak256(utils.toUtf8Bytes("v3-core-license-grants"));
const OWNER_UNISWAP_GOVERNANCE_TIMELOCK: string =
  "0x1a9C8182C09F50C8318d769245beA52c32BE35BC";
const RESOLVER_PUBLIC_ENS_RESOLVER: string =
  "0x4976fb03c32e5b8cfe2b6ccb31c09ba78ebaba41";
const TTL: number = 0;

const ensRegistryInterface = new Interface(ENS_REGISTRY_ABI);
const setSubnodeRecordCalldata = ensRegistryInterface.encodeFunctionData(
  "setSubnodeRecord",
  [
    NODE_TOP_LEVEL,
    LABEL,
    OWNER_UNISWAP_GOVERNANCE_TIMELOCK,
    RESOLVER_PUBLIC_ENS_RESOLVER,
    TTL,
  ]
);

const ensPublicResolverInterface = new Interface(ENS_PUBLIC_RESOLVER_ABI);

const NODE: string = namehash("v3-core-license-grants.uniswap.eth");
const KEY: string = "[your-project's-additional-use-grant-title]";
const VALUE: string = "[your-additional-use-grant-description]";
const setTextCalldata = ensPublicResolverInterface.encodeFunctionData(
  "setText",
  [NODE, KEY, VALUE]
);

const ENS_REGISTRY_ADDRESS: string =
  "0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e";

const PUBLIC_ENS_RESOLVER_ADDRESS: string =
  "0x4976fb03c32e5b8cfe2b6ccb31c09ba78ebaba41";

/// @notice the ordered list of target addresses for calls to be made
const targets = [ENS_REGISTRY_ADDRESS, PUBLIC_ENS_RESOLVER_ADDRESS];

/// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made.
/// as this example does not include the transfering of any tokens, this list is empty.
const values = [0, 0];

/// @notice The ordered list of function signatures to be called. The signatures arguments
/// are optional, if not provided, the function signature will be inferred from the calldata
const signatures = ["", ""];

/// @notice The ordered list of calldata to be passed to each call in the proposal. The calldata
/// in this example takes the place of the function signature arguments.
const calldatas = [setSubnodeRecordCalldata, setTextCalldata];

/// @notice the description of the proposal.
const description = "Your Additional Use Grant Proposal Description";

async function main() {
  const yourSigner = await ethers.getSigner("YOUR_SIGNER_ADDRESS_HERE");

  return Promise.all(
    await governorBravo
      .connect(yourSigner)
      .propose(targets, values, signatures, calldatas, description)
  );
}

main();
```

</details>

## Helpful Links

- [Governor Bravo Proxy](https://etherscan.io/address/0x408ED6354d4973f66138C91495F2f2FCbd8724C3#readProxyContract)
- [Governor Bravo Delegate](https://etherscan.io/address/0x53a328f4086d7c0f1fa19e594c9b842125263026#code)
- [ENS Subnode Record Update Details](https://github.com/Uniswap/deploy-v3#licensing)