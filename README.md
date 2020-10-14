# Safe Remote Purchase

### Authors  
1. Pavan R Shetty (4NM17CS122)
2. Mohammed Akzar A K (4NM17CS106)
3. Mohammed Sharhan Marakada (4NM17CS107)

## Introduction

Solidity is an object-oriented programming language for writing smart contracts.It is used for implementing smart contracts on various blockchain platforms, most notably, Ethereum. The Approved Account Transaction contract implements the simplest form of a cryptocurrency. The contract allows only its creator to create new coins (different issuance schemes are possible). Anyone can send coins to each other without a need for registering with a username and password, all you need is an Ethereum keypair.<br><br>
      A blockchain is a globally shared, transactional database. This means that everyone can read entries in the database just by participating in the network. If you want to change something in the database, you have to create a so-called transaction which has to be accepted by all others. The word transaction implies that the change you want to make (assume you want to change two values at the same time) is either not done at all or completely applied. Furthermore, while your transaction is being applied to the database, no other transaction can alter it.<br><br>
      If a transfer from one account to another is requested, the transactional nature of the database ensures that if the amount is subtracted from one account, it is always added to the other account. If due to whatever reason, adding the amount to the target account is not possible, the source account is also not modified. Furthermore, a transaction is always cryptographically signed by the sender (creator). This makes it straightforward to guard access to specific modifications of the database.A simple check ensures that only the person holding the keys to the account can transfer money from it.<br>
      
## Program

pragma solidity ^0.7.0;

contract Purchase {
    uint public value;
    address payable public seller;
    address payable public buyer;

    enum State { Created, Locked, Release, Inactive }
    // The state variable has a default value of the first member, `State.created`
    State public state;

    modifier condition(bool _condition) {
        require(_condition);
        _;
    }

    modifier onlyBuyer() {
        require(
            msg.sender == buyer,
            "Only buyer can call this."
        );
        _;
    }

    modifier onlySeller() {
        require(
            msg.sender == seller,
            "Only seller can call this."
        );
        _;
    }

    modifier inState(State _state) {
        require(
            state == _state,
            "Invalid state."
        );
        _;
    }

    event Aborted();
    event PurchaseConfirmed();
    event ItemReceived();
    event SellerRefunded();

    // Ensure that `msg.value` is an even number.
    // Division will truncate if it is an odd number.
    // Check via multiplication that it wasn't an odd number.
    constructor() payable {
        seller = msg.sender;
        value = msg.value / 2;
        require((2 * value) == msg.value, "Value has to be even.");
    }

    /// Abort the purchase and reclaim the ether.
    /// Can only be called by the seller before
    /// the contract is locked.
    function abort()
        public
        onlySeller
        inState(State.Created)
    {
        emit Aborted();
        state = State.Inactive;
        // We use transfer here directly. It is
        // reentrancy-safe, because it is the
        // last call in this function and we
        // already changed the state.
        seller.transfer(address(this).balance);
    }

    /// Confirm the purchase as buyer.
    /// Transaction has to include `2 * value` ether.
    /// The ether will be locked until confirmReceived
    /// is called.
    function confirmPurchase()
        public
        inState(State.Created)
        condition(msg.value == (2 * value))
        payable
    {
        emit PurchaseConfirmed();
        buyer = msg.sender;
        state = State.Locked;
    }

    /// Confirm that you (the buyer) received the item.
    /// This will release the locked ether.
    function confirmReceived()
        public
        onlyBuyer
        inState(State.Locked)
    {
        emit ItemReceived();
        // It is important to change the state first because
        // otherwise, the contracts called using `send` below
        // can call in again here.
        state = State.Release;

        buyer.transfer(value);
    }

    /// This function refunds the seller, i.e.
    /// pays back the locked funds of the seller.
    function refundSeller()
        public
        onlySeller
        inState(State.Release)
    {
        emit SellerRefunded();
        // It is important to change the state first because
        // otherwise, the contracts called using `send` below
        // can call in again here.
        state = State.Inactive;

        seller.transfer(3 * value);
    }
}

## Implementation

We have implemented a Ethereum Smart Contract using Solidity to create a Approved Account Transaction.<br>The line address public Creator; declares a state variable of type address that is publicly accessible. The next line, mapping (address => uint) public balances; also creates a public state variable. The type maps addresses to unsigned integers. Mappings can be seen as hash tables which are virtually initialized such that every possible key exists from the start and is mapped to a value whose byte-representation is all zeros.

function balances(address _account) external view returns (uint) {
    return balances[_account];
}

This function can be used to query the balance of a single account.<br>
The line event Sent(address from, address to, uint amount); declares a so-called “event” which is emitted in the last line of the function send. User interfaces can listen for those events being emitted on the blockchain without much cost. As soon as it is emitted, the listener will also receive the arguments from, to and amount, which makes it easy to track transactions.The constructor is a special function which is run during creation of the contract and cannot be called afterwards. It permanently stores the address of the person creating the contract. msg.sender is always the address where the current (external) function call came from.<br><br>
Finally, the functions that will actually end up with the contract and can be called by users and contracts alike are create and send. If cretae is called by anyone except the account that created the contract, nothing will happen. This is ensured by the special function require which causes all changes to be reverted if its argument evaluates to false. The second call to require ensures that there will not be too many coins, which could cause overflow errors later.<br><br>
On the other hand, send can be used by anyone (who already has some of these coins) to send coins to anyone else. If you do not have enough coins to send, the require call will fail and also provide the user with an appropriate error message string.<br>

### This program is executed in two environments.
1. JVM
2. Injected Web

## JVM
![](Jvm/1.program.gif)
![](Jvm/2.cmpilrLoading.gif)
![alt text](Jvm/3.AftrrCompilerLoadRun.jpg)
![](Jvm/4.SelectCreatorAcnt.gif)
![alt text](Jvm/5.CreatorAccount.jpg)
![alt text](Jvm/6.AfterDeploy.jpg)
![](Jvm/7.SelectingReceiverAddress.gif)
![](Jvm/8.SelecValueNTransctToChoosenRecieverFromCreator.gif)
![](Jvm/9.BalnceInReciveAcnt,CreatorNam,CreatorAddress.gif)
![](Jvm/10.SelectRecievrForSendFunc.gif)
![](Jvm/11.SelectSenderNValueForSendFunc.gif)
![](Jvm/12.SentValueUsingSendFunction.gif)
![alt text](Jvm/13.ValueInSenderAcntAftrSent.jpg)

## Injected Web
![](injectedWeb/1.SelectInjectedWeb.gif)
![](injectedWeb/2.DeploySuccesful.gif)
![alt text](injectedWeb/3.SuccessfulDeployLog.jpg)
![](injectedWeb/4.AfterDeployCreatorAddress.jpg)
![alt text](injectedWeb/5.RecieverAddresNCreatorName.gif)
![alt text](injectedWeb/6.SendingValueToReciever.gif)
![](injectedWeb/7.CheckingBalanceOfReciever.gif)
![](injectedWeb/8.CheckingBalanceOfCreator.gif)
![](injectedWeb/9.UsingSendErrorBecause0BALANCEWithSender.gif)

## Conclusion
Approved Account Transaction Implemented.
