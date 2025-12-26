# Kaia Testnet Subgraph Deployment Guide

This guide will help you deploy the Uniswap V3 subgraph for Kaia testnet.

## Prerequisites

1. **Node.js and Yarn** installed
2. **The Graph CLI** installed: `npm install -g @graphprotocol/graph-cli`
3. **Access to The Graph Studio** or your own Graph Node
4. **Contract deployment information** from your `state.json`

## Step 1: Update Configuration Files

### 1.1 Update Start Block

You need to find the block number where the Factory contract was deployed. You can find this by:

1. Looking at the deployment transaction hash in your deployment logs
2. Querying the Kaia testnet explorer for the factory address: `0xb522cF1A5579c0EAe37Da6797aeBcE1bac2D4a29`

Once you have the block number, update `config/kaia-testnet/config.json`:

```json
{
  "network": "kaia-testnet",
  "factory": "0xb522cF1A5579c0EAe37Da6797aeBcE1bac2D4a29",
  "startblock": "YOUR_DEPLOYMENT_BLOCK_NUMBER"
}
```

### 1.2 Update Token Addresses

Update `config/kaia-testnet/chain.ts` with actual token addresses on Kaia testnet:

1. **REFERENCE_TOKEN**: The wrapped native token address (WKAI) on Kaia testnet
2. **STABLE_TOKEN_POOL**: A stable token pool address (can be updated after first pool is created)
3. **WHITELIST_TOKENS**: Add common tokens that should be tracked
4. **STABLE_COINS**: Add stablecoin addresses (USDC, USDT, etc.)

Example:
```typescript
export const REFERENCE_TOKEN = '0x...' // WKAI address
export const STABLE_TOKEN_POOL = '0x...' // Update after creating a pool
export const WHITELIST_TOKENS: string[] = [
  REFERENCE_TOKEN,
  '0x...', // USDC
  '0x...', // USDT
]
```

### 1.3 Update Subgraph Names (Optional)

If you want custom subgraph names, update `config/kaia-testnet/.subgraph-env`:

```
V3_TOKEN_SUBGRAPH_NAME="v3-tokens-kaia-testnet"
V3_SUBGRAPH_NAME="uniswap-v3-kaia-testnet"
```

## Step 2: Install Dependencies

```bash
yarn install
```

## Step 3: Build the Subgraph

Build the V3 subgraph:

```bash
yarn build --network kaia-testnet --subgraph-type v3
```

This will:
- Copy the chain configuration files
- Generate the subgraph manifest (`v3-subgraph.yaml`)
- Run code generation

## Step 4: Deploy to The Graph

### Option A: Deploy to The Graph Studio (Hosted Service)

1. **Create a subgraph in The Graph Studio**:
   - Go to https://thegraph.com/studio/
   - Create a new subgraph
   - Note: The Graph may not support "kaia-testnet" as a network name. You may need to:
     - Use a custom Graph Node (see Option B), or
     - Contact The Graph team to add Kaia testnet support

2. **Set up environment variables**:
   Create a `.env` file in the root directory:
   ```
   ALCHEMY_DEPLOY_KEY="your-deploy-key"
   ALCHEMY_DEPLOY_URL="https://api.studio.thegraph.com/deploy/"
   ALCHEMY_IPFS_URL="https://api.studio.thegraph.com/ipfs/"
   ```

   Also create `.subgraph-env` in the root (or it will be copied from config):
   ```
   V3_SUBGRAPH_NAME="your-username/uniswap-v3-kaia-testnet"
   ```

3. **Deploy**:
   ```bash
   yarn build --network kaia-testnet --subgraph-type v3 --deploy
   ```

### Option B: Deploy to Custom Graph Node

If The Graph doesn't support Kaia testnet, you'll need to run your own Graph Node:

1. **Set up a Graph Node** that connects to Kaia testnet RPC
2. **Update your `.env`**:
   ```
   ALCHEMY_DEPLOY_KEY="your-key"
   ALCHEMY_DEPLOY_URL="http://your-graph-node:8020"
   ALCHEMY_IPFS_URL="http://your-ipfs-node:5001"
   ```

3. **Deploy**:
   ```bash
   yarn build --network kaia-testnet --subgraph-type v3 --deploy
   ```

### Option C: Manual Deployment

If you prefer manual deployment:

1. **Build the subgraph** (from Step 3)
2. **Authenticate with The Graph**:
   ```bash
   graph auth --studio <your-deploy-key>
   ```
3. **Deploy manually**:
   ```bash
   graph deploy --studio <your-subgraph-name> v3-subgraph.yaml
   ```

## Step 5: Verify Deployment

1. Check The Graph Studio dashboard for indexing status
2. Query the subgraph using GraphQL:
   ```graphql
   {
     pools(first: 5) {
       id
       token0 {
         symbol
       }
       token1 {
         symbol
       }
     }
   }
   ```

## Troubleshooting

### Network Not Supported

If The Graph doesn't recognize "kaia-testnet", you have two options:

1. **Use a custom Graph Node**: Set up your own Graph Node that supports Kaia testnet
2. **Contact The Graph**: Request Kaia testnet support from The Graph team

### Start Block Issues

- Make sure the start block is correct (the block where Factory was deployed)
- If indexing is slow, you can use a later block number, but you'll miss earlier events

### Token Address Issues

- Ensure all token addresses in `chain.ts` are correct
- Use lowercase addresses
- Verify addresses on Kaia testnet explorer

## Additional Resources

- [The Graph Documentation](https://thegraph.com/docs/)
- [Uniswap V3 Subgraph Repository](https://github.com/Uniswap/v3-subgraph)
- [Graph Node Setup Guide](https://github.com/graphprotocol/graph-node)

## Your Deployed Contracts

From your `state.json`, here are the deployed contract addresses:

- **Factory**: `0xb522cF1A5579c0EAe37Da6797aeBcE1bac2D4a29`
- **Multicall2**: `0x2A2aDD27F8C70f6161C9F29ea06D4e171E55C680`
- **ProxyAdmin**: `0xC7a8B0c7e78807DD3C53D98AE8850D2932a0c4d8`
- **TickLens**: `0x56C8DAB2fFf78D49a76B897828E7c58896bA8b87`
- **NFTDescriptorLibrary**: `0x9cF0c0F4A41Ea5300c9D12D47643382289d901D9`
- **NonfungibleTokenPositionDescriptor**: `0x76aAbaC0f28FD64A3e884f6EFa680F72895e6A48`
- **DescriptorProxy**: `0xEDfB1b7d2591af9FB911Cc9C914A22F7f1A2B178`
- **NonfungibleTokenPositionManager**: `0x9546E23b2642334E7B82027B09e5c6c8E808F4E3`
- **V3Migrator**: `0xDab424Aba37f24A94f568Df345634d4B66830ebB`
- **V3Staker**: `0xc3cF7B37E5020f718aceE1f4e1b12bC7b1C6CE4B`
- **QuoterV2**: `0x56a4BD4a66785Af030A2003254E93f111892BfB5`
- **SwapRouter02**: `0xd28909Ef8bd258DCeFD8B5A380ff55f92eD8ae4b`

