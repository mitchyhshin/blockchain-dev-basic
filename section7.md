# **Section 7 - ERC721** :monkey_face:

# ERC721 소개

- ERC-721은 증서라고 알려진 NFT의 표준안입니다. NFT는 대체불가토큰(Non Fungible Token)의 약자로 대체 불가능한 토큰이라는 의미입니다. 따라서 ERC-721로 발행되는 토큰은 대체 불가능하여 모두 제 각각의 가치(Value)를 갖고 있습니다.

- ERC20(Fungible Token, FT)과 ERC721(Non Fungible Token, NFT)이 있습니다. 보통 FT를 토큰 또는 코인이라고 말하고 NFT는 NFT라고 말합니다.
둘다 토큰이기 때문에 FT는 토큰이라고 말하기 보다는 코인이라고 말하는게 좋을거 같습니다.

- openzeppelin ERC721폴더를 찾습니다. (/node_modules/@openzeppelin/contracts/token/ERC721)

- IERC721.sol 살펴보기

    - function balanceOf(address owner) external view returns (uint256 balance);

    - function ownerOf(uint256 tokenId) external view returns (address owner);

    - function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId,
        bytes calldata data
    ) external;

    - function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId
    ) external;

    - function transferFrom(
        address from,
        address to,
        uint256 tokenId
    ) external;

    - function approve(address to, uint256 tokenId) external;

    - function setApprovalForAll(address operator, bool _approved) external;

    - function getApproved(uint256 tokenId) external view returns (address operator);

    - function isApprovedForAll(address owner, address operator) external view returns (bool);

- ERC721.sol 살펴보기

# ERC721 My NFT 만들기

- /contracts/Monkey.sol 파일 추가

- Monkey.sol 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```solidity
        //SPDX-License-Identifier: MIT

        pragma solidity 0.8.17;

        import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
        import "hardhat/console.sol";

        import "@openzeppelin/contracts/access/Ownable.sol";
        import "@openzeppelin/contracts/access/AccessControl.sol";
        import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";

        import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
        import "@openzeppelin/contracts/utils/Counters.sol";
        import "@openzeppelin/contracts/utils/Strings.sol";
        import "@openzeppelin/contracts/utils/Base64.sol";

        contract Monkey is Ownable, ERC721Enumerable, ERC721URIStorage {
            using Strings for uint256;
            using Counters for Counters.Counter;
            Counters.Counter private _tokenIds;

            mapping(uint256 => uint256) public tokenIdToLevels;
            mapping(uint256 => string) public tokenIdToNames;

            constructor(string memory name, string memory symbol) ERC721(name, symbol) {}

            function getLevels(uint256 tokenId) public view returns (string memory) {
                uint256 levels = tokenIdToLevels[tokenId];
                return levels.toString();
            }

            function generateCharacter(uint256 tokenId) public view returns (string memory) {
                bytes memory svg = abi.encodePacked(
                    '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
                    "<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>",
                    '<rect width="100%" height="100%" fill="black" />',
                    '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">',
                    tokenIdToNames[tokenId],
                    "</text>",
                    '<text x="50%" y="50%" class="base" dominant-baseline="middle" text-anchor="middle">',
                    "Level: ",
                    getLevels(tokenId),
                    "</text>",
                    "</svg>"
                );
                return string(abi.encodePacked("data:image/svg+xml;base64,", Base64.encode(svg)));
            }

            function getTokenURI(uint256 tokenId) public view returns (string memory) {
                bytes memory dataURI = abi.encodePacked(
                    "{",
                    '"name": "Monkey #',
                    tokenId.toString(),
                    '",',
                    '"description": "Monkey created by Mo Young Chul",',
                    '"image": "',
                    generateCharacter(tokenId),
                    '",',
                    '"attributes": [{"trait_type": "Level", "value": ',
                    tokenIdToLevels[tokenId].toString(),
                    " }] ",
                    "}"
                );
                return string(abi.encodePacked("data:application/json;base64,", Base64.encode(dataURI)));
            }

            function mint(string memory name) external {
                _tokenIds.increment();
                uint256 tokenId = _tokenIds.current();
                _safeMint(msg.sender, tokenId);
                tokenIdToLevels[tokenId] = 1;
                tokenIdToNames[tokenId] = name;
                _setTokenURI(tokenId, getTokenURI(tokenId));
            }

            function train(uint256 tokenId) public {
                require(_exists(tokenId), "Please use an existing token");
                require(ownerOf(tokenId) == msg.sender, "You must own this token to train it");
                tokenIdToLevels[tokenId] += 1;
                _setTokenURI(tokenId, getTokenURI(tokenId));
            }

            function _beforeTokenTransfer(
                address from,
                address to,
                uint256 firstTokenId,
                uint256 batchSize
            ) internal virtual override(ERC721, ERC721Enumerable) {
                super._beforeTokenTransfer(from, to, firstTokenId, batchSize);
            }

            function supportsInterface(
                bytes4 interfaceId
            ) public view virtual override(ERC721, ERC721Enumerable) returns (bool) {
                return super.supportsInterface(interfaceId);
            }

            function _burn(uint256 tokenId) internal virtual override(ERC721, ERC721URIStorage) {
                super._burn(tokenId);
            }

            function tokenURI(uint256 tokenId) public view virtual override(ERC721, ERC721URIStorage) returns (string memory) {
                return super.tokenURI(tokenId);
            }
        }

        ```
    
    </details>

    - 출처 : https://docs.alchemy.com/docs/how-to-make-nfts-with-on-chain-metadata-hardhat-and-javascript
- Typechain 만들기
    ```
    npx hardhat typechain
    ```

# Monkey NFT UnitTest 만들기

- /test/Monkey.test.ts 파일 추가

- Monkey.test.ts 스크립트 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import { expect } from 'chai';
        import { BigNumber } from 'ethers';
        import { ethers, waffle } from 'hardhat';
        import MonkeyArtifact from '../artifacts/contracts/Monkey.sol/Monkey.json';
        import { Monkey } from '../typechain';

        describe('Monkey', () => {
            let monkey: Monkey;

            const [admin, other0, other1, other2, receiver] =
                waffle.provider.getWallets();

            before(async () => {});

            beforeEach(async () => {
                monkey = (await waffle.deployContract(admin, MonkeyArtifact, [
                'Monkey',
                'Mon',
                ])) as Monkey;
            });

            it('mint', async () => {
                await monkey.mint('brown');
                await monkey.mint('james');
                const balance = await monkey.balanceOf(admin.address);
                expect(balance).to.be.equal(BigNumber.from(2));
                const totalSupply = await monkey.totalSupply();
                expect(totalSupply).to.be.equal(BigNumber.from(2));
            });

            it('train', async () => {
                await monkey.mint('brown');
                const level = await monkey.getLevels(1);
                expect(level).to.be.equal(BigNumber.from(1));

                await monkey.train(1);

                const afterLevel = await monkey.getLevels(1);
                expect(afterLevel).to.be.equal(BigNumber.from(2));
            });
        });

        ```
    
    </details>

- Unit Test 실행
    ```
    npx hardhat test .\test\Monkey.test.ts
    ```

# Deploy Monkey NFT to Klaytn

- /src/Monkey 폴더추가

- /src/Monkey/deploy.ts 파일 추가

- deploy.ts 스크립트 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import hre, { ethers } from 'hardhat';
        import MonkeyArtifact from '../../artifacts/contracts/Monkey.sol/Monkey.json';
        import { getGasOption } from '../utils/gas';
        import * as fs from 'fs';

        async function main() {
            const [admin] = await hre.ethers.getSigners();

            const chainId = hre.network.config.chainId || 0;

            const factory = await ethers.getContractFactory(MonkeyArtifact.contractName);
            const contract = await factory.deploy('Monkey', 'Mon', getGasOption(chainId));
            const receipt = await contract.deployTransaction.wait();

            const deployedContract = {
                address: contract.address,
                blockNumber: receipt.blockNumber,
                chainId: hre.network.config.chainId,
                abi: MonkeyArtifact.abi,
            };

            const filename = __dirname + `/monkey.deployed.json`;

            const deployedContractJson = JSON.stringify(deployedContract, null, 2);
            fs.writeFileSync(filename, deployedContractJson, {
                flag: 'w',
                encoding: 'utf8',
            });

            console.log(deployedContractJson);
        }

        main()
        .then(() => process.exit(0))
        .catch(error => {
            console.error(error);
            process.exit(1);
        });

        ```
    
    </details>

- [Opensea metadata standards](https://docs.opensea.io/docs/metadata-standards)

- deploy.ts 실행
    ```
    npx hardhat run --network baobab .\src\monkey\deploy.ts
    ```

# Mint Monkey NFT

- /src/Monkey/mint.ts 파일 추가

- mint.ts 스크립트 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import hre, { ethers } from 'hardhat';
        import { getGasOption } from '../utils/gas';
        import * as fs from 'fs';
        import { Monkey } from '../../typechain';

        async function main() {
            const [admin] = await hre.ethers.getSigners();

            const chainId = hre.network.config.chainId || 0;

            const deployedContractJson = fs.readFileSync(
                __dirname + '/monkey.deployed.json',
                'utf-8',
            );
            const deployedContract = JSON.parse(deployedContractJson);
            const monkey = (await ethers.getContractAt(
                deployedContract.abi,
                deployedContract.address,
            )) as Monkey;

            const transaction = await monkey.mint('brown', getGasOption(chainId));
            await transaction.wait();
        }

        main()
        .then(() => process.exit(0))
        .catch(error => {
            console.error(error);
            process.exit(1);
        });

        ```
    
    </details>

- mint.ts 실행
    ```
    npx hardhat run --network baobab .\src\monkey\mint.ts
    ```

- Klaytn Scope 확인