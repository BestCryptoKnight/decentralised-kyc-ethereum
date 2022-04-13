# Decentralised KYC Etherum

<p align="center">
  <img width="100%"  src="https://github.com/GeekyAnts/sample-decentralised-kyc-ethereum/blob/main/assets/1.png?raw=true">
</p>

KYC is a process by which banks obtain information about the identity and address of the purchasers. It’s a regulator governed process of performing due diligence for verifying the identity of clients. This process helps to make sure that banks’ services aren’t misused. The banks are responsible for completing the KYC procedure while opening accounts. Banks also are required to periodically update their customers’ KYC details. KYC may be a manual, time-consuming, and redundant across institutions. Sharing KYC information on Blockchain would enable financial institutions to deliver better compliance outcomes, increase efficiency, and improve customer experience.

## Problem

Each company has to verify your identity somehow, and it’s particularly important for financial institutions. From this ‘know your customer,’ or KYC protocols was the rise to assist companies to ensure they know who they’re doing business with. Typically, this involves an extended, drawn-out practice where certain documents are shown, and a few kinds of background checks or verification takes place.
In the traditional KYC system, each bank will conduct its identity check i.e. each user is checked individually by an individual organization or government structure. Hence, there is a waste of time for checking each identity from scratch.

## Solution

The blockchain architecture and the DLT allow us to collect information from various service providers into one cryptographically secure and unchanging database that does not need a third party to verify the authenticity of the knowledge. It makes it possible to form a system where the user will only need to undergo the KYC procedure once to verify his/her identity.

## Required:

- Different Roles: Admin Financial Institution(e.g RBI), Financial Institutions, & Customers
- Smart Contract consisting of all the rules and protocols required for KYC document flow. We have created 2 contacts for Banks and Customers, and inherited those contract KYC contract.
- Blockchain Network to deploy the Contract. We have used Rinkeby for our contract.
- Website for user Interface where Users according to their role can access informfation. We have created webpage with React & Native Base.

## Assumptions:

- Admin Financial Institution can add verified FIs.
- Admin can make FIs active/inactive w.r.t to any actions.
- FIs can add Customers and request for KYC from Customers.
- Customer can approve/reject the KYC request from FIs.
- If Customer approve the KYC request, a notiifcation (via email/phone no.) will be sent to FI and FI can access the Customer's KYC documents such as Aadhar Card, Pancard, Photo Id, Signature, etc for verification.
- FIs can approve/reject the Customer's data after verifing.
- If FI rejects Customer's KYC verification, a notiifcation (via email/phone no.) will be sent to Customer with the reason.
- Customer can update the KYC documents and the update notification will get trigger to all the connected FIs.
- All the user roles should have mandatory metamask address on the main network.
- The User who deploy the contract to the main net will be considered as Admin FI.

## How it works

<p align="center">
  <img width="100%"  
      alt='KYC'
       src="https://github.com/GeekyAnts/sample-decentralised-kyc-ethereum/blob/main/assets/kyc-small.jpg?raw=true">
</p>

### Modifiers

```c++
/**
     * @notice Checks whether the requestor is admin
     */
    modifier isAdmin() {
        require(
            msg.sender == admin,
            "Only admin is allowed to operate this functionality"
        );
        _;
    }

    /**
     * @notice Checks whether the requestor is bank & is active
     * @param id_ Metamask address of the bank
     */
    modifier isValidBank(address id_) {
        require(banks[id_].id_ != address(0), "Bank not added by admin");
        require(
            banks[id_].id_ == id_,
            "Unauthenticated requestor! Bank not added by admin"
        );
        require(
            banks[id_].status == BankStatus.Active,
            "Bank is not active! Contact Admin"
        );
        _;
    }

    /**
     * @notice Checks whether customer already exists
     * @param id_ Metamask address of the customer
     */
    modifier isValidCustomer(address id_) {
        require(id_ != address(0), "Id is empty");
        Customer memory user_ = customers[id_];
        require(user_.id_ != address(0), "User Id is empty");
        require(!compareStrings(user_.email, ""), "User Email is empty");
        _;
    }
```

### KYC Objects

```c++
    enum Role {
        Admin, // 0
        Bank, // 1
        Customer // 2
    }

    enum BankStatus {
        Active, // 0
        Inactive // 1
    }

    enum KycStatus {
        Pending, // 0
        KYCVerified, // 1
        KYCFailed // 2
    }

    enum DataHashStatus {
        Pending, // 0
        Approved, // 1
        Rejected // 2
    }

    struct User {
        string name;
        string email;
        address id_;
        Role role;
        BankStatus status;
    }

    struct Customer {
        string name;
        string email;
        uint256 mobileNumber;
        address id_;
        address kycVerifiedBy; // Address of the bank only if KYC gets verified
        string dataHash; // Documents will be stored in decentralised storage & a hash will be created for the same
        uint256 dataUpdatedOn;
    }

    struct Bank {
        string name;
        string email;
        address id_;
        string ifscCode;
        uint16 kycCount; // How many KCY's did this bank completed so far
        BankStatus status; // RBI, we call "admin" here can disable the bank at any instance
    }

    struct KycRequest {
        string id_; // Combination of customer Id & bank is going to be unique
        address userId_;
        string customerName;
        address bankId_;
        string bankName;
        bytes32 dataHash;
        uint256 updatedOn;
        KycStatus status;
        DataHashStatus dataRequest; // Get approval from user to access the data
        string additionalNotes; // Notes that can be added if KYC verification fails  OR
        // if customer rejects the access & bank wants to re-request with some message
    }
```

### Common Method(s)

| **Function Name**    | **Input Params**                   | **Return value** | **Description**                                                                                     |
| -------------------- | ---------------------------------- | ---------------- | --------------------------------------------------------------------------------------------------- |
| whoAmI()             | -                                  | User             | To get the details of the logged-in user (To decide wether I am a bank or admin or normal customer) |
| getCustomerDetails() | metamask `address` of the customer | Customer         | To get details of a single                                                                          |
| getBankDetails()     | metamask `address` of the bank     | Bank             | To get details of a single bank                                                                     |

### Admin Interface Methods

| **Function Name**        | **Input Params**                                     | **Return value**              | **Description**                                             |
| ------------------------ | ---------------------------------------------------- | ----------------------------- | ----------------------------------------------------------- |
| getAllBanks()            | pageNumber                                           | `totalPages` count,<br>Bank[] | To get all the list of banks that were added by admin (RBI) |
| addBank()                | Bank                                                 | -                             | To add new bank                                             |
| updateBankDetails()      | metamask `address` of bank,<br>email,<br>name        | -                             | To update the name & email of the bank                      |
| activateDeactivateBank() | metamask `address` of bank,<br>`isActivated` boolean | -                             | To activate & deactivate bank from the toggle               |

### Bank Interface Methods

| **Function Name**        | **Input Params**                                                             | **Return value**                                                 | **Description**                                                                                                     |
| ------------------------ | ---------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| getCustomersOfBank()     | pageNumber                                                                   | `totalPages` count,<br>KycRequest[]                              | To get all the list of customer kyc requests that were added by the current bank                                    |
| addKycRequest()          | Customer,<br>current `time` in epoch,<br>additional `notes`                  | -                                                                | To add new KYC request by adding new customer                                                                       |
| reRequestForKycRequest() | metamask `address` of the customer,<br>additional `notes`                    | -                                                                | If bank rejects the KYC verification or If customer rejects permission issue then this request can be rasided again |
| updateKycVerification()  | metamask `address` of customer,<br>`verified` boolean,<br>additional `notes` | -                                                                | To mark a customer as KYC verified                                                                                  |
| searchCustomers()        | metamask `address` of customer                                               | `bool` to say customer exists or not,<br>Customer,<br>KycRequest | To fetch customer by metamask address                                                                               |

### Customer Interface Methods

| **Function Name**           | **Input Params**                                                            | **Return value**                                         | **Description**                                                                           |
| --------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| getBankRequests()           | pageNumber                                                                  | `totalPages` count,<br>KycRequest[]                      | To get all the list of banks that requested KYC & were associated to the current customer |
| actionOnKycRequest()        | metamask `address` of the bank,<br>boolean `bankId_`,<br>additional `notes` | -                                                        | To update the KYC request (Either Approves or Rejects)                                    |
| updateProfile()             | name,<br>email,<br>mobile number                                            | -                                                        | To update the profile details like name, email, mobile number                             |
| updateDatahash()            | hash,<br>current `time` in epoch                                            | -                                                        | To update the Data hash of the documents                                                  |
| removerDatahashPermission() | metamask `address` of bank,<br>additional `notes`                           | -                                                        | To remove documents read permission to a specific bank                                    |
| searchBanks()               | metamask `address` of bank                                                  | `bool` to say bank exists or not,<br>Bank,<br>KycRequest | To get a specific bank details via address search                                         |

### Events

| **Event Name**            | **Params**                                                                                                                              | **Description**                                                             |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| bankAdded                 | metamask `address`,<br>name,<br>email,<br>ifsc code                                                                                     | Triggered when a new bank is added by admin (RBI)                           |
| bankUpdated               | metamask `address`,<br>name,<br>email                                                                                                   | Triggered when bank updates it's details                                    |
| bankActivated             | metamask `address`,<br>name                                                                                                             | Triggered when admin activates bank                                         |
| bankDeactivated           | metamask `address`,<br>name                                                                                                             | Triggered when admin de-activates bank                                      |
|                           |                                                                                                                                         |                                                                             |
| customerAdded             | metamask `address`,<br>name,<br>email                                                                                                   | Triggered when bank adds new customer                                       |
| customerDataUpdated       | metamask `address`,<br>name,<br>email                                                                                                   | Triggered when customer updates his/her profile                             |
| dataHashUpdated           | metamask `address`,<br>name,<br>datahash                                                                                                | Triggered when customer updates his/her KYC documents                       |
| dataHashPermissionChanged | Request ID,<br>Bank Name,<br>Customer Name,<br>Customer metamask `address`,<br>Bank metamask `address`,<br>Datahash permission `status` | Triggered when customer revokes/grants permission to his/her KYC documents. |
|                           |                                                                                                                                         |                                                                             |
| kycRequestAdded           | Request ID,<br>Bank Name,<br>Customer Name                                                                                              | Triggered when banks adds new KYC request                                   |
| kycReRequested            | Request ID,<br>Bank Name,<br>Customer Name                                                                                              | Triggered when bank re-requests for KYC documents permission.               |
| kycStatusChanged          | Request ID,<br>Bank Name,<br>Customer Name,<br>Customer metamask `address`,<br>Bank metamask `address`,<br>KYC verification `status`    | Triggered when bank rejects/verifies KYC documents                          |
