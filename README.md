<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->

<a name="readme-top"></a>

<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Don't forget to give the project a star!
*** Thanks again! Now go create something AMAZING! :D
-->

<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->

[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/Aboudoc/AU-proxies">
    <img src="images/logo.png" alt="Logo" width="80" height="60">
  </a>

<h3 align="center">Proxies</h3>

  <p align="center">
    project_description
    <br />
    <a href="https://github.com/Aboudoc/AU-proxies"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/Aboudoc/AU-proxies">View Demo</a>
    ·
    <a href="https://github.com/Aboudoc/AU-proxies/issues">Report Bug</a>
    ·
    <a href="https://github.com/Aboudoc/AU-proxies/issues">Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->

## About The Project

[![Product Name Screen Shot][product-screenshot]](https://example.com)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Built With

-   [![Hardhat][Hardhat]][Hardhat-url]
-   [![Ethers][Ethers.js]][Ethers-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->

## Getting Started

To get a local copy up and running follow these simple example steps.

### Prerequisites

-   npm

    ```sh
    npm install npm@latest -g
    ```

-   hardhat

    ```sh
    npm install --save-dev hardhat
    ```

    ```sh
    npm install @nomiclabs/hardhat-ethers @nomiclabs/hardhat-waffle
    ```

    run:

    ```sh
    npx hardhat
    ```

### Installation

1. Clone the repo
    ```sh
    git clone https://github.com/Aboudoc/AU-proxies.git
    ```
2. Install NPM packages
    ```sh
    npm install
    ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- USAGE EXAMPLES -->

## Usage

If you need testnet funds, use the [Alchemy testnet faucet](https://goerlifaucet.com/).

This project demonstrates some basic proxy use cases. It comes with four different proxy contracts, a test for proxy v2 contract, and a library from openZeppelin called StorageSlot using assembly code.

For testing purposes, get contract (ABI) Logic1 and Logic2 with proxy address

```js
const proxyAsLogic1 = await ethers.getContractAt("Logic1", proxy.address)
const proxyAsLogic = await ethers.getContractAt("Logic2", proxy.address)
```

Using two different ways to check x value:

```js
assert.equal(await logic1.x(), 0)
assert.equal(await ethers.provider.getStorageAt(logic1.address, 0x0), 0)
```

By using eth_getStorageAt we are bypassing the public getter, we can remove public viewer on x variable

# Proxy contract

We're not using `delegatecall`, a massive migration will be needed when changing implementation. The fallback forwards the call to the implementation contract. the implementation works on its own storage.

# Proxy V2 contract

The storage values are inside of the proxy. This is where the `delegatecall` comes in. No data migration needed in case of ugrading to logic2. For more details, check for [Transparent Proxy Pattern](https://blog.openzeppelin.com/the-transparent-proxy-pattern/)

# Generic Proxy contract

Using EIP1967 to modify an arbitrary storage slot that we create to put away the implementation to make sure it doesn't collate with any storage variables => [library StorageSlot](https://eips.ethereum.org/EIPS/eip-1967) from EIP-1967

```js
    /**
     * @dev Returns an `AddressSlot` with member `value` located at `slot`.
     */
    function getAddressSlot(bytes32 slot) internal pure returns (AddressSlot storage r) {
        assembly {
            r.slot := slot
        }
```

Generic Proxy should only be used for learning purposes! One thing that it does not do is return the return value in the fallback function. This can only be done by dropping down into assembly code, as shown by the [OpenZeppelin proxy logic](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#proxy-forwarding). In general, you should try to stick to using proxies that are audited and battle tested!

_For audited examples, please refer to the [OpenZeppelin Docs](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable)_

# Open Zeppelin Proxy

An example of how upgrading actually works using openZeppelin's `Proxy.sol`contract

`SmallProxy`contract inherits from [Proxy.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/Proxy.sol), set the implementation slot as specified in [EIP-1967](https://eips.ethereum.org/EIPS/eip-1967)

```js
bytes32 private constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
```

The implementation slot is the the `keccak-256` hash of **"eip1967.proxy.implementation"** subtracted by 1

To test this proxy, you can use helper functions as `getDataToTransact` or `readStorage`. Feel free to test on Remix, seing is believing.

Note how `readStorage` function uses assembly:

```js
    function readStorage() public view returns (uint256 valueAtStorageSlotZero) {
        assembly {
            valueAtStorageSlotZero := sload(0)
        }
    }
```

Keep in mind that **if a contract can be updated by only one person, you have a single centralized point of failure. Technically the contract isn't even decentralized**

The main issue is **function selector clashes**. Transparent Proxy and Universal Upgradable Proxy (UUPS) are the solution

# Transparent Proxy Pattern

You can find a basic example of Transparent Proxy Pattern in this repo: (coming soon...)

# Universal Upgradable Proxy Standard (UUPS)

(coming soon...)

[EIP-1822](https://eips.ethereum.org/EIPS/eip-1822)

All the logic of upgrading in the implementation

-   Allows Solidity to detect 2 functions with the same selector (while compiling)
-   Gas saver: we have to do lesser read. No need to check in the proxy contract if someone is an Admin

The main issue is **if you deploy an implementation contract without an upgradable functionality, back to the YEET method**

# Diamond Pattern

(coming soon...)

-   Allows for multiple implementation contracts
-   Allows to make more granular upgrades

# Metamorphic Contract

(coming soon...)

# Proxy exploit

Some proxy's vulnerabilities can be found in this repo: [Proxy-exploit](https://github.com/Aboudoc/Proxy-exploit)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ROADMAP -->

## Roadmap

-   [ ] Test Generic Proxy
-   [ ] Write scripts to deploy and interact with proxy
-   [-] Explore proxie's vulnerabilities
-   [ ] Proxy patterns
    -   [ ] Transparent Proxy Pattern
    -   [ ] Universal Upgradable Proxy (UUPS)

See the [open issues](https://github.com/Aboudoc/AU-proxies/issues) for a full list of proposed features (and known issues).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTRIBUTING -->

## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- LICENSE -->

## License

Distributed under the MIT License. See `LICENSE.txt` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTACT -->

## Contact

Reda Aboutika - [@twitter_AboutikaR](https://twitter.com/AboutikaR) - reda.aboutika@gmail.com

Project Link: [https://github.com/Aboudoc/AU-proxies](https://github.com/Aboudoc/AU-proxies)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ACKNOWLEDGMENTS -->

## Acknowledgments

-   [AlchemyUniversity](https://university.alchemy.com/)
-   [Dan-Nolan](https://github.com/Dan-Nolan)
-   [Patrick Collins](https://github.com/PatrickAlphaC)
-   [OpenZeppelin](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies)
-   [SolidityByExample](https://solidity-by-example.org/)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->

[contributors-shield]: https://img.shields.io/github/contributors/Aboudoc/AU-proxies.svg?style=for-the-badge
[contributors-url]: https://github.com/Aboudoc/AU-proxies/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/Aboudoc/AU-proxies.svg?style=for-the-badge
[forks-url]: https://github.com/Aboudoc/AU-proxies/network/members
[stars-shield]: https://img.shields.io/github/stars/Aboudoc/AU-proxies.svg?style=for-the-badge
[stars-url]: https://github.com/Aboudoc/AU-proxies/stargazers
[issues-shield]: https://img.shields.io/github/issues/Aboudoc/AU-proxies.svg?style=for-the-badge
[issues-url]: https://github.com/Aboudoc/AU-proxies/issues
[license-shield]: https://img.shields.io/github/license/Aboudoc/AU-proxies.svg?style=for-the-badge
[license-url]: https://github.com/Aboudoc/AU-proxies/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/r%C3%A9da-aboutika-34305453/?originalSubdomain=fr
[product-screenshot]: https://res.cloudinary.com/divzjiip8/image/upload/c_scale,w_239/v1587421101/mascots_dge1th.png
[Hardhat]: https://img.shields.io/badge/Hardhat-20232A?style=for-the-badge&logo=hardhat&logoColor=61DAFB
[Hardhat-url]: https://hardhat.org/
[Ethers.js]: https://img.shields.io/badge/ethers.js-000000?style=for-the-badge&logo=ethersdotjs&logoColor=white
[Ethers-url]: https://docs.ethers.org/v5/
[Vue.js]: https://img.shields.io/badge/Vue.js-35495E?style=for-the-badge&logo=vuedotjs&logoColor=4FC08D
[Vue-url]: https://vuejs.org/
[Angular.io]: https://img.shields.io/badge/Angular-DD0031?style=for-the-badge&logo=angular&logoColor=white
[Angular-url]: https://angular.io/
[Svelte.dev]: https://img.shields.io/badge/Svelte-4A4A55?style=for-the-badge&logo=svelte&logoColor=FF3E00
[Svelte-url]: https://svelte.dev/
[Laravel.com]: https://img.shields.io/badge/Laravel-FF2D20?style=for-the-badge&logo=laravel&logoColor=white
[Laravel-url]: https://laravel.com
[Bootstrap.com]: https://img.shields.io/badge/Bootstrap-563D7C?style=for-the-badge&logo=bootstrap&logoColor=white
[Bootstrap-url]: https://getbootstrap.com
[JQuery.com]: https://img.shields.io/badge/jQuery-0769AD?style=for-the-badge&logo=jquery&logoColor=white
[JQuery-url]: https://jquery.com

```

```
