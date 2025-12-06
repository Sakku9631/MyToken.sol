# MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

/// @title MyToken - Simple ERC-20 ready for Remix deployment on Ethereum
/// @notice Owner-mintable, burnable, pausable ERC-20 with optional cap.
/// @dev Imports OpenZeppelin from raw GitHub URLs so Remix can fetch them.
import "https://raw.githubusercontent.com/OpenZeppelin/openzeppelin-contracts/v4.9.3/contracts/token/ERC20/ERC20.sol";
import "https://raw.githubusercontent.com/OpenZeppelin/openzeppelin-contracts/v4.9.3/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "https://raw.githubusercontent.com/OpenZeppelin/openzeppelin-contracts/v4.9.3/contracts/security/Pausable.sol";
import "https://raw.githubusercontent.com/OpenZeppelin/openzeppelin-contracts/v4.9.3/contracts/access/Ownable.sol";

contract MyToken is ERC20, ERC20Burnable, Pausable, Ownable {
    /// @notice cap in base units (includes decimals). If 0 => no cap.
    uint256 public immutable cap;

    /// @param name_ token name
    /// @param symbol_ token symbol
    /// @param initialSupply_ initial supply minted to deployer (in base units: amount * 10**decimals)
    /// @param cap_ maximum total supply in base units. Set to 0 for no cap.
    constructor(
        string memory name_,
        string memory symbol_,
        uint256 initialSupply_,
        uint256 cap_
    ) ERC20(name_, symbol_) {
        cap = cap_;
        if (initialSupply_ > 0) {
            require(cap == 0 || initialSupply_ <= cap, "initial exceeds cap");
            _mint(msg.sender, initialSupply_);
        }
    }

    /// @notice Mint tokens to an address. Only owner.
    /// @param to recipient
    /// @param amount amount in base units
    function mint(address to, uint256 amount) external onlyOwner {
        require(amount > 0, "amount 0");
        require(cap == 0 || totalSupply() + amount <= cap, "cap exceeded");
        _mint(to, amount);
    }

    /// @notice Pause transfers. Only owner.
    function pause() external onlyOwner {
        _pause();
    }

    /// @notice Unpause transfers. Only owner.
    function unpause() external onlyOwner {
        _unpause();
    }

    /// @dev Enforce pausability on transfers
    function _beforeTokenTransfer(address from, address to, uint256 amount)
        internal
        override
    {
        super._beforeTokenTransfer(from, to, amount);
        require(!paused(), "token transfer while paused");
    }

    // decimals() defaults to 18 in OpenZeppelin ERC20. Override if you want different decimals.
}
MyToken.sol
```markdown
# Deploy MyToken to Ethereum using Remix

This README shows how to compile and deploy `MyToken.sol` from Remix to the Ethereum network (mainnet or a testnet) using MetaMask.

Files
- MyToken.sol — ERC-20 token: owner-mintable, burnable, pausable, optional cap.

Overview
- Paste `MyToken.sol` into Remix and compile with a Solidity 0.8.x compiler that matches the imports.
- Use MetaMask (Injected Provider) to deploy to Ethereum Mainnet (default in MetaMask) or a testnet (e.g., Sepolia).
- Verify source on Etherscan after deployment (flatten if required).

Preparation
1. MetaMask:
   - Install MetaMask and unlock it.
   - For mainnet, MetaMask's "Ethereum Mainnet" is preconfigured.
   - For testing, switch MetaMask to a supported testnet (Sepolia is commonly used). Check current recommended testnets in official docs.
   - Ensure the account has ETH on the chosen network (use faucets for testnet).

2. RPC / Provider:
   - You can use MetaMask's default provider or add a custom RPC (Alchemy, Infura, Blast). For heavier use prefer a reputable RPC provider.
   - If using a custom RPC, add it in MetaMask network settings.

Compile in Remix
1. Open https://remix.ethereum.org
2. Create a new file named `MyToken.sol` and paste the contract above.
3. Open the "Solidity Compiler" tab:
   - Select a compiler version compatible with pragma (e.g., 0.8.17).
   - Enable Optimization (recommended) and set runs = 200.
   - Click "Compile MyToken.sol".
4. If Remix fails to fetch OpenZeppelin imports:
   - Pick a compiler version matching the OpenZeppelin tag used in imports, or
   - Copy the required OpenZeppelin contracts into separate files in Remix.

Constructor arguments (deploy)
- name_ (string) — token name, e.g. "MyToken"
- symbol_ (string) — token symbol, e.g. "MTK"
- initialSupply_ (uint256) — initial minted amount to deployer, expressed in base units (includes decimals). Example:
  - For 1,000 tokens with 18 decimals: 1000 * 10**18 = `1000000000000000000000`
- cap_ (uint256) — maximum total supply in base units. Use `0` for no cap. If set, initialSupply_ must be <= cap_.

Example constructor values:
- name_: "MyToken"
- symbol_: "MTK"
- initialSupply_: `1000000000000000000000` (1,000 tokens, 18 decimals)
- cap_: `10000000000000000000000` (10,000 tokens cap) or `0` for no cap

Deploy with MetaMask (Injected Provider)
1. In Remix, open the "Deploy & Run Transactions" panel.
2. Set Environment to "Injected Provider - MetaMask".
3. Connect Remix to MetaMask when prompted; ensure MetaMask is set to the correct network (Mainnet or selected testnet).
4. Select the `MyToken` contract from the dropdown.
5. Enter constructor arguments as described above.
6. Click "Deploy". MetaMask will show a confirmation; review gas estimate and confirm.
7. Wait for the transaction to confirm on the network.

Post-deploy
- Copy the deployed contract address from Remix.
- In Remix (Deployed Contracts) you can call:
  - mint(to, amount) — owner only
  - burn(amount) — token holder
  - pause() / unpause() — owner only
  - standard ERC-20 functions: transfer, approve, allowance, etc.
- Amounts are in base units (include decimals).

Verify on Etherscan
1. Go to Etherscan (or the testnet explorer for your network).
2. Use "Verify & Publish" to submit the source code.
3. Choose the same compiler version and optimizer settings you used in Remix.
4. If Etherscan asks for a flattened file, I can generate one for you.

Security & recommendations
- Test first on a testnet (Sepolia) before mainnet.
- Use hardware wallets or multisig (Gnosis Safe) for significant deployments.
- Consider adding: timelock/multisig for admin functions, permit (EIP-2612), and thorough testing/audit for production.
- Keep private keys and RPC credentials secure.

Troubleshooting
- "Wrong Network" in MetaMask: check selected network and chainId.
- Remix import errors: choose a compiler matching the OpenZeppelin tag or paste OZ sources locally.
- Insufficient gas / RPC errors: ensure enough ETH for gas and consider using a reliable RPC provider.

If you want I can:
- Generate a flattened contract for Etherscan verification.
- Produce a Hardhat deploy script (mainnet or Sepolia) with example .env and RPC settings.
- Add permit (EIP-2612), metadata, or other token features.
- Walk you step-by-step through a testnet deployment while you follow along in Remix/MetaMask.
```
