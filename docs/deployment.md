Deploying a contract takes a few steps. These can either be done in a batch transaction or individually.

## CreateAccount
First, we need to create an account for our contract. We can do this by submitting a CreateAccount transaction. Additionally, we must either deploy a contract in the same batch transaction, or create an full access key so we can deploy it later.

After creating a new account, all subsequent actions in the same batch transaction will be executed on behalf of the new account.

    CreateAccountAction {
    }

The CreateAccount action doesn't contain any data, as it uses the receiver address of the Transaction or ActionReceipt within which it is contained.

(?) Add predecessors and registrar https://nomicon.io/RuntimeSpec/Actions#createaccountaction

## DeployContract
In order to deploy code to an account, you must have a full access key or deploy the code from the account itself. Then, you submit a DeployContract action containing the contract's bytecode. This will replace the contract's code with the submitted bytecode. Indeed, code can be deployed multiple times to the same contract in order to upgrade it to new versions. If a contract needs to be non-upgradeable (trustless), all full access keys need to be removed (and the contract should not be able to deploy code to itself in an untrusted manner). Note that there is a maximum contract code size determined by the genesis configuration (see ``max_contract_size``) (?)

    DeployContractAction {
        code: Vec<u8>
    }

The contract is deployed and ready to use as soon as the action finishes.

There are no limitations on how many ``DeployContract`` actions exist in a batch. For example we could have a batch that looks like: 
    1. Deploy a contract
    2. Function call on that contract
    3. Deploy a new contract to that location

## Initialization
All contracts must implement the ``Default`` trait. This is because whenever any method is called on the contract, if the contract state does not already exist a default state will be created. If this is not desired, one can instead implement the ``PanicOnDefault`` trait which, as the name suggests, panics when ``default`` is called.
The ``#[init]`` decorator allows you to write a function that returns an instance of the contract state, which is then written to storage. This allows you to initialize the contract. By default, ``#[init]`` panics if the state already exists, but you can instead use ``#[init(ignore_state)]`` if the function should be able to be called multiple times.
In order to call an init method, you just submit a transaction with a FunctionCall action. An init method may take arguments and is not called automatically.

If you call a different function before the ``#[init]`` function, the contract's ``Default`` implementation will be called. Therefore, if you want to enforce calling of the ``#[init]`` function, you should derive the ``PanicOnDefault`` trait, or simply replace the ``Default`` implementation with your desired functionality.

## All together now
Putting it all together, we could submit a transaction containing a batch of actions as follows:

    actions: [
        CreateAccountAction {},
        AddKeyAction { "public_key": "...", "access_key": "..." },
        DeployContractAction { "code": "..." },
        FunctionCall { gas: 100000, deposit: 0, method_name: "init", args: "{'hello_world'}" }],

This batch would then execute atomically, so even if the init method were to fail, we wouldn't be left with a misconfigured contract. Later, we can upgrade the contract by using the access key we created, assuming we configured it to have full access.
