# Building your own crypto-currency using Solana programs

Welcome to the Solana crypto-currency quest. With this quest you'll get upto speed with the most rapidly rising blockchain in the market: *Solana*. It would be awesome if you know a bit 
of Rust (or even C++ concepts) already and are familiar with how blockchains work, but even if you do not have any specific background of Rust or Solana development, we will have all bases covered. If 
you have a high level of interest and motivation, we should be good to go ahead.

In this quest, we will be developing our own crypto-currency on the Solana blockchain or our own `spl-token` in the Solana lingo. This essentially means that once you are done with this quest, 
you will be able to make your crypto-currency using Solana programs and use that to do whatever you can think of, including using it as a fan token, a social token, a governance token, a utility token
or a coin.

# Setting up the Environment: 

There are a few things that we need to get up and running before we move forward in this quest. Before we move forward make sure you've a working NodeJS environment set up. We need rust, Solana, Mocha(a JS testing framework), Anchor and Phantom wallet for this quest.
To install rust, run
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustup component add rustfmt
```

To install Solana, run
```
sh -c "$(curl -sSfL https://release.solana.com/v1.8.0/install)"
```

To install mocha globally, run
```
npm install -g mocha
```

Now we'll be installing Anchor.
If you're on a linux system, run
```
# Only on linux systems
npm i -g @project-serum/anchor-cli
```

**Fair Warning** : If you are using a Windows system, we highly suggest using [WSL2](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) (Windows sub-system for Linux) or switching to a Linux environment. Setting up WSL is also quick and easy. A 
good walk-through can be found [here](https://www.youtube.com/watch?v=X3bPWl9Z2D0&ab_channel=BeachcastsProgrammingVideos)
For any other OS, you need to build from source. Run the following command
```
cargo install --git https://github.com/project-serum/anchor --tag v0.18.0 anchor-cli --locked
```

To verify that Anchor is installed, run
```
anchor --version
```

Since Solana is still a pretty new blockchain compared to the establised ones out there, it's developer tooling too is pretty limited and cumbersome as of now. However, it is rapidly improving 
and it does so on a daily basis. At the forefront of this development is Anchor, by [Armani Ferrante](https://twitter.com/armaniferrante). You can think of it like the Ruby on Rails framework for
Ruby, that means yes, you can develop things on vanilla Ruby, but Ruby on Rails makes your life much much easier, right? That's the same with Anchor and Solana development. Anchor is the Hardhat of 
Solana development plus much more. It offers a Rust DSL (basically, an easier Rust) to work with along with IDL, CLI and workspace management. Anchor abstracts away a lot of potential security holes
from a conventional Solana 


# Running configurations on Solana CLI

The first command you should run on your terminal (assuming Solana CLI was properly installed in the last quest) is:

```
solana config get
```

This should throw up a result similar to something like: 

![](img/1.png)

If you didnot set up your keypair earlier, then you won't be having the `Keypair Path` in your results. To set that up, follow the instructions over [here](https://docs.solana.com/wallet-guide/paper-wallet#seed-phrase-generation)

We would want to remain on the local network for building our program and later shift to the devent or mainnet-beta if required. If the `RPC URL` field of your last result did not show `localhost`, you can set it to localhost using the following command:

```
solana config set --url localhost
```

Next, we would want to know our account/wallet address and airdrop some SOL tokens into it, to handle all the deployment, transactions etc costs that come with interacting with and using a Solana program. To do that first let's find our address. The command to do that is:
```
solana address
```

This would result into something like this:

![](img/2.png)

Then, for more comprehensive details of your account, use the following command with the address that you got from the last command
```
solana account <your address from the last command>
```

This would result into something like this:

![](img/3.png)

Next, we want to spin up our local network. Think of this local network as a mock Solana blockchain running on your own single system. This network would be required for development and testing of our program. To spin it up, in a separate tab, use the following command:
```
solana-test-validator
```

Once you get an image, like the one below, you know that your local validator (local network) is now up and running

![](img/4.png)

Now, our last task is to top up our account with some SOL, which you can do by using:

```
solana airdrop 100
```

This should result in something like:

![](img/5.png)

# Setting up our Anchor project

In this sub-quest all we would do is initialize an Anchor project and see whether everything's there and working fine or not and after move on ahead to make our own changes.
Head over to your preferred destination for the project using your terminal and then type the following command:
```
anchor init mymoneydapp

cd mymoneydapp
``` 

This would result in a screen somewhat similar to this:

![](img/6.png)

First we check whether we can see the *programs*, *app*, *programs*, *migrations* directory among others or not. If we can, we would head over to *programs/messengerapp/src/lib.rs* to see the default program that Anchor provides us. This is the most basic example possible on Anchor and what's happening here is simply that a user-defined function `Initialize` whenever called would successfully exit the program. That's all, nothing fancy. Now, let's try to compile this program using the following command:

```
anchor build
```

This would trigger a build function and would something like this upon completion:

![](img/7.png)

This build creates a new folder in your project directory called, `target`. This `target` folder contains the `idl` directory, which would also contain the `idl` for our program. The `IDL` or Interface Description Language describes the instructions exposed by the contract and is very similar to ABI in Solidity and user for similar purposes, ie, for tests and front-end integrations. Next, we can move onto testing this program, so that we can get familiar with how testing is done in Anchor. Head to `tests/messengerapp.js`. Here, you'll see a test written in javascript to interact and test the default program. There are a lot of things in the test, that may not make sense to you right now, but stick around and we'll get to those shortly. The test would look something like this:

![](img/8.png)

Next, to actually run these tests, first head over to the tab where you ran the solana-test-validator command and kill that process (using Ctrl-C). Now, use the following command:

```
anchor test
```

The passing tests should result in the following screen:

![](img/9.png)

Now, let's head over to the `programs` directory and start importing some cool Rust crates provided by Anchor which will help us build our money app.

## Importing the Anchor SPL Crates

Head over to `programs/mymoneydapp/Cargo.toml` and under dependencies, add the two following lines:

```
anchor-spl = "0.17.0"
spl-token = { version = "3.1.1", features = ["no-entrypoint"] }
```

Make sure that the version of the `anchor-spl` you write here, matches the version of `anchor-lang` that you had installed. It would be much more convenient for you if install the same version that we are using in the quest. After these changes, the `Cargo.toml` file would look something like this:

![](img/11.png)

Now that we are done adding the dependencies in the Cargo file, let's call it inside our program too. Write down the following `use` statments at the very top of your `programs/mymoneydapp/src/lib.rs` file:

```
use anchor_spl::token::{self, Burn, MintTo, SetAuthority, Transfer};
```

After this, clear all the default code that we were provided with, this would make your coding screen look something like:

![](img/12.png)

## Writing our first program function

Head over to `programs/mymoneydapp/src/lib.rs` and clear the code written there apart from the macro declarations and crates (libraries) that we will be using. After the clearning, your coding screen should look something like the last screen of the last quest.

Now let us simply define four functions that we will be using in our Solana program. The bulk of the program goes in a module under the `#[program]` macro. We'll just define them under the `pub mod mymoneydapp` and write the logic later. These four function definitions would look like this:
```
    pub fn proxy_transfer(ctx: Context<ProxyTransfer>, amount: u64) -> ProgramResult {

    }
    
    pub fn proxy_mint_to(ctx: Context<ProxyMintTo>, amount: u64) -> ProgramResult {
    
    }
    
    pub fn proxy_burn(ctx: Context<ProxyBurn>, amount: u64) -> ProgramResult {
    
    }
    
    pub fn proxy_set_authority(
        ctx: Context<ProxySetAuthority>,
        authority_type: AuthorityType,
        new_authority: Option<Pubkey>,
    ) -> ProgramResult {
    
    }
```

Notice the use of the word `proxy` in each function name? That is because we would be doing `Cross Program Invocation` or CPI for short in this program of ours. This essentially means that we would be calling functions of other Solana programs from our program. As you might have guessed, we would be calling the functions from the `anchor_spl` programs, which is the (abstracted) Anchor implementation of the spl-programs that you can find in the Solana Rust SDK. To look at the structure and explore the `anchor_spl` programs more, visit https://github.com/project-serum/anchor/tree/v0.17.0/spl. Reading through that would also help us understand the *implementations* later on.

Here `pub` means public and `fn` means function, implying that they are public functions that can be invoked from our program, ie it becomes a client-callable program function. The first argument of these functions is always `Context<T>` which consist of the solana accounts array and the program ID, which in essence is the required data to call just about any progarm on Solana. The next parameter of these functions is a `u64` or an unsigned integer named *amount*, which we will be using as our the amount of our tokens in different functions. The `ProgramResult` is the return type of both these functions, which actually is just an easier method to serve function results and/or errors.

As discussed earlier, the `Context` parameter in each function is essentially a list of all the accounts that must be passed for the function to work as expected and the different structs you see in all `Contexts` such as `ProxyTransfer`, `ProxyMintTo`, etc will be defined later. 

After defining the above four functions, your code should look something like this:

![image](https://user-images.githubusercontent.com/32522659/141686353-c526a7d5-930f-427d-8ba2-74f2dc6d8f33.png)

## Writing the logic for our functions

The logic of all of our functions would be very straight-forward. We would simply call the functions provided by `anchor_spl` with the correct parameters. That's it. Simple. First update your code as the following:
```
    pub fn proxy_transfer(ctx: Context<ProxyTransfer>, amount: u64) -> ProgramResult {
        token::transfer(ctx.accounts.into(), amount);
    }
    
    pub fn proxy_mint_to(ctx: Context<ProxyMintTo>, amount: u64) -> ProgramResult {
        token::mint_to(ctx.accounts.into(), amount);
    }
    
    pub fn proxy_burn(ctx: Context<ProxyBurn>, amount: u64) -> ProgramResult {
        token::burn(ctx.accounts.into(), amount);
    }
    
    pub fn proxy_set_authority(
        ctx: Context<ProxySetAuthority>,
        authority_type: AuthorityType,
        new_authority: Option<Pubkey>,
    ) -> ProgramResult {
        token::set_authority(ctx.accounts.into(), authority_type.into(), new_authority);
    }
```

With this, your coding screen would look something like: 

![image](https://user-images.githubusercontent.com/32522659/141686661-65bdf7e1-44ac-4668-a8ce-962094805b63.png)

If you want to further investigate the functions that we called here, you can head over to the [official docs of the `anchor_spl` crate](https://docs.rs/anchor-spl/0.17.0/anchor_spl/index.html).

## Let's learn to serialize and deserialize
