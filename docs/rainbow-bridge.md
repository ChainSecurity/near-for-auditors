The rainbow bridge is a bridge between Ethereum and NEAR. In this document we'll give a detailed description of the various parts of the bridge.

The overview is based on https://near.org/blog/eth-near-rainbow-bridge/?utm_source=pocket_mylist.

Trust assumptions:
    * On Ethereum, finality is achieved after X blocks.
    * On NEAR, 2/3 of the validators are honest.
    * A signature check on Ethereum fits into one block?.

The rainbow bridge is:
    * trustless: since most of its parts are on blockchain
    * rapid: With the exception of checking some signatures on-chain, all the other transactions do not require long time intervals to be executed.
    * decentralized: the bridge can be deployed by anyone
    * generic: it can be used for different tasks sicne ...

Design

The system consists of the following smart contracts:

On Ethereum:
    
    NearBridge: this contract is also known as the NEAR light client. The main purpose of the contract is to track the state of NEAR on Ethereum. Any user who has deposited 32ETH? can submit a new NEAR block (``addLightClientBlock``). At least one block per epoch must be submitted. At the beginning of each epoch the public keys of the validators are saved. To reduce the costs, only the header of the block is added.

    The blocks are sent using the NEAR2ETH relay.

    NEAR uses Ed25519 to sign messages. There is no available precompiled contract for EVM to verify the signature, thus, the verification should be using normal contracts. This verification is computationally expensive. To avoid using it, an optimistic approach is chosen. This means any user can challenge a submitted signature by calling ``challenge``. Only the last submitter can be challenged. Currently the challenge window is set to be 4 hours.

    NEAROnEthereumProver: It proves the specific events are included?

    ERC20 Locker: It's responsible for locking and unlocking ERC20 tokens.


On NEAR:
    Ethereum Light Client (eth-client): this contract serves the same role as the near light client on Ethereum. A block is added by calling ``add_block_header``. The ethereum client is more complicated since it has to restrict the storage it occupies on the blockchain. That's why hashes that are too old are removed. 


Execution Flow

From Ethereum to NEAR

    1. Alice calls ERC20Locker.lockTokens after she has approved the contract. This way she transfers an amount of an ERC20 to the ERC20Locker contract. A locked event is emitted.

    2. The relay waits for enough blocks to be confirmed and then pushes the header of the block that contains the event of interest (not entirely precise)

    3. RainbowLib computes the proof of the event

    4. MintableFungibleToken contract verifies the proof (?)

    5. MintableFungibleToken mints the ERC20 token (?)

    


