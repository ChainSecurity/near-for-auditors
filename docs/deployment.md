Deploying a contract takes a few steps. These can either be done in a batch transaction or individually.

# CreateAccount
First, we need to create an account for our contract. We can do this by submitting a CreateAccount transaction. Additionally, we must either deploy a contract in the same batch transaction, or create an full access key so we can deploy it later.

After creating a new account, all subsequent actions in the same batch transaction will be executed on behalf of the new account.

# DeployContract
In order to deploy code to an account, you must have a full access key. Then, you submit a DeployContract action containing the contract's bytecode. This will replace the contract's code with the submitted bytecode. Indeed, code can be deployed multiple times to the same contract in order to upgrade it to new versions. If a contract needs to be non-upgradeable (trustless), all full access keys need to be removed (and the contract should not be able to deploy code to itself in a trustless fashion).

# Initialization
All contracts must implement the `Default` trait. This is because whenever any method is called on the contract, if the contract state does not already exist a default state will be created. If this is not desired, one can instead implement the `PanicOnDefault` trait which, as the name suggests, panics when `default` is called.
The `#[init]` decorator allows you to write a function that returns an instance of the contract state, which is then written to storage. This allows you to initialize the contract. By default, `#[init]` panics if the state already exists, but you can instead use `#[init(ignore_state)]` if the function should be able to be called multiple times.
In order to call an init method, you just submit a transaction with a FunctionCall action. An init method may take arguments.