---
layout: post
title:  "Recreating the Bitcoin blockchain with Rust"
date:   2022-10-11 14:35:50 -0300
categories: ["Blockchain"]

---
In this article, I will recreate the Bitcoin blockchain using the Rust programming language, with the goal of learning more about how the features and structures of the world's most famous blockchain are implemented in practice.

*As the purpose of this article is to only demonstrate the structure of Bitcoin's blockchain, I will not delve into the concepts surrounding the transaction lifecycle, such as UTXOs, lock and unlock scripts, etc. However, if you're interested in learning more about these topics, I recommend this article: https://www.oreilly.com/library/view/mastering-bitcoin/9781491902639/ch05.html*


## History leading up to Bitcoin:

In 1991, researchers <b>Stuart Haber</b> and <b>W. Scott Stornetta</b>  introduced the idea of a cryptographically secured chain of blocks to store timestamped documents.

In 2004, computer scientist <b>Hal Finney</b> introduced a system called <b>Reusable Proof of Work(RPOW)</b> which solved the double-spending problem by maintaining token ownership records on a trusted server that allowed users to verify its integrity in real-time.

In 2008 <b>Satoshi Nakamoto</b> conceptualized the theory <b>distributed blockchain</b> which essentially allowed blocks to be added to the initial chain without needing to be signed by trusted parties, using a peer-to-peer network for timestamping and verifying each data exchange. This allowed the blockchain to be managed autonomously without the need for a central authority. These changes proposed by Satoshi, using previous concepts, enabled the use of blockchains for cryptocurrencies.


# Code Implementation:


## Blockchain structures:
- <b>Blockchain</b>:
```rust
pub struct Blockchain {
  pub blocks: Vec<Block>,
  pub transactions_pool: Vec<Transaction> 
}
/* transactions_pool stores transactions that have not yet been added to any block.*/
```
- <b>Block</b>:
```rust
pub struct Block {
  pub header: Block_Header,
  pub size: f32,
  pub transactions_counter: usize,
  pub transactions: Vec<Transaction>
}
```
- <b>Block_Header</b>:
```rust
pub struct Block_Header {
  pub version: usize,
  pub previous_hash: String,
  pub nonce: i32,
  pub timestamp: String,
  pub merkle_root: String,
  pub difficulty: i32
}
```
- <b>Transactions</b>:
```rust
pub struct Transaction {
  pub version: usize,
  pub input_counter: usize,
  pub signature: String,
  pub inputs: Vec<Inputs>,
  pub outputs: Vec<Outputs>, 
  pub locktime: Datransactions_poolte
}
```

## Proof Of Work
Perhaps the biggest factor in Bitcoin's success was its consensus protocol, **Proof of Work**. Before Bitcoin, there were attempts to create digital money systems, such as e-Gold and b-Money, but they failed to solve the double-spending problem effectively and in a decentralized manner

In short, the Proof of Work algorithm requires computational effort to validate a block. This is done by searching for a specific number called a **nonce**, which, when combined with other specific information and hashed, results in a certain value.

Next, we will implement this algorithm and explain step by step how it works in practice.

```rust
pub fn proof_of_work(&mut self, previous_hash: String, version: usize) -> i32 {
  let mut nonce: i32 = 0;

  loop {
    let prefix = String::from("0");
    let header = &Header { 
      version, 
      previous_hash: previous_hash.clone(),
      nonce
    };

    let hashed_with_nonce = utils::hash_block(header);

    if validated_hash(hashed_with_nonce.clone(), 3, prefix) {
      println!("nonce {} validated: {}", nonce, hashed_with_nonce);
      return nonce;
    }
    nonce = nonce + 1;
  }
}

pub fn validated_hash(hash: String, difficulty: usize, prefix: String) -> bool {
  let check = prefix.repeat(difficulty);

  return hash.starts_with(&check);
}

pub fn hash_block(block_header: &Header) -> String {
  let nonce = block_header.nonce.to_string();
  let previous_hash = block_header.previous_hash.clone();
  let version = block_header.version.to_string();

  let mut to_be_hashed = concat_strings(version, previous_hash);
  to_be_hashed = concat_strings(to_be_hashed, nonce);

  return hex::encode(hash(&to_be_hashed));
}
```

The  function **validated_hash** takes three arguments, including the **hash** to be validated, **difficulty** referring to the number of times the **prefix** appears at the beginning of the hash.
To perform the check, the function declares a variable check that receives a string containing the character prefix multiplied by the difficulty. For example, if the prefix is **0** and the difficulty is **5**, the check will be: **00000**.
It then returns a boolean checking if the provided hash string starts with the check string.

The function **hash_block** receives a header struct and uses the provided information to create a hash.

The **proof_of_work** function implements an infinite loop, and inside this loop, it constructs a header using information from the previous block and a nonce with an initial value of 0, passes it to the **hash_block** function that returns a hash to be verified by the **validated_hash** function. If the hash is valid, it returns the current nonce value; if not, it increments the nonce by one and repeats the process until it finds the nonce that, along with the other header information, makes the block valid. This process is called **mining**, and the higher the **difficulty** value, the longer it takes to find the nonce value, which requires more computational power. This verification makes it nearly impossible to alter anything in the Bitcoin blockchain, as it would require finding all the nonces of all the blocks that have already been validated.
*If you are interested in knowing the current difficulty defined in the Bitcoin blockchain, you can run a Bitcoin node and type the opcode **OP_GETDIFFICULTY**.*

## Inserting a block into the blockchain:
To insert a block into the blockchain, we must first ensure that the blockchain contains at least one block. If not, we need to create and manually insert the first block, known as the **genesis block**.

Next, we need to construct the block header, taking the values as follows:
 
```rust
let previous_hash = self.get_last_block_hash();
let version = 1; // block version number indicates which set of block validation rules to follow.
let nonce = self.proof_of_work(previous_hash.clone(), version.clone());

let header = &Header {
  previous_hash: previous_hash.clone(),
  nonce,
  version
};
```
After constructing the header, we can take the transactions in the transaction pool (also called **mempool**) and add them to the block payload.
*Keep in mind that transactions must be validated before being added to the block, but for example purposes, we will consider that all transactions in the pool are already validated.*

```rust
let mut copy_vec: Vec<Transaction> = Vec::new();
let mut i = 0;

while i < self.transactions_pool.len() {
  let copy_transaction: Transaction = Transaction {
    input_counter: self.transactions_pool[i].input_counter,
    signature: self.transactions_pool[i].signature.clone(),
    version: self.transactions_pool[i].version
  };
  copy_vec.push(copy_transaction);

  i = i + 1;
}
```

Next, we create a new block, pass it to the **hash_block** function, and validate it with the **validated_hash** function, passing the hash, difficulty, and prefix as parameters. In this case, we will use the prefix 0 and the difficulty 2.
And if the block is valid, we will add it to the Blockchain.

```rust
let new_block = Block::new(header, copy_vec);
let new_block_hash = utils::hash_block(header);
let block_validated = utils::validated_hash(new_block_hash.clone(), 2, String::from("0"));

if block_validated {
  self.blocks.push(new_block);
} else {
  println!("Failed to validate block: {}", new_block_hash);
}
```

With this, we conclude the main structure of this blockchain. You can interact with it by cloning the [project repository](https://github.com/alk0x1/Ritcoin) and running it with cargo run. This way, you can insert and better visualize the blocks.