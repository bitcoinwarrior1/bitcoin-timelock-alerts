# bitcoin-timelock-alerts

Receive alerts for when CSVs and CLTVs time conditions are met.

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the
result.

You can start editing the page by modifying `app/page.tsx`. The page
auto-updates as you edit the file.

This project uses
[`next/font`](https://nextjs.org/docs/basic-features/font-optimization) to
automatically optimize and load Inter, a custom Google Font.

## Use cases

### Green app 2FA expiry & other security services that leverage timelocks

Blockstream green has a 2FA multisig feature that leverages CSVs. The UTXO unlocking script requires two signatures (one from you, and one from blockstream), until the CSV timelock is reached and the funds become spendable with just your key. Users need to create new UTXOs to reset the multisig, and forgetting to do so removes the protection that the multsig provides.

Users of this feature can use this service to remind them that they need to create new timelocks.

### Escrow, delayed payments & financial services

Many services leverage timelocks, such as escrows, delayed payments and estate planning. This service can provide alerts to let users know when the timelock payment is due to become valid.

### Lightning

Layer 2 protocols like the Lightning Network rely on relative timelocks (CSV) to manage closing channels or dispute periods. This service can be used to remind users when a timelock becomes due, enabling them to prevent force closures.

### HODLing

Some users lock up their coins to prevent them from premature selling. This service can remind them when the timelock is set to expire. Users may then wish to reset the timelock, or spend the funds.

## Business model

### Free

The free tier will allow users to download ical files for a particular transaction.

### Paid

The paid tier will allow users to track particular addresses by sending in an `xPub` or array of `addresses`. The paid tier will include the following premium features:

1. Email alerts
2. Dynamic tracking of their wallets, allowing new transactions to be automatically tracked
3. The ability to add `xPubs`, enabling new addresses to be derived and tracked for new transactions

Users will pay a monthly subscription, the price will be determined by the amount of inputs to track.

## Requirements 

### MVP
1. Create a web interface that lets a user paste an address or UTXO 
2. Show an error if the UTXO or address does not contain any timelock scripts
2. Grab all the timelock conditions from the UTXO(s), including the timestamp and unlock conditions
3. Create an iCal file for each timelock & its condition(s)
4. Download the calendar file 

### Post MVP premium requirements 
1. Email alerts (including alerts for new inputs, and spending of tracked inputs) 
2. Real time monitoring of addresses/xPubs for new inputs

## Architecture

### Backend

#### getTimeLocks

This function gets the CSVs or CLTVs for a particular transaction.

<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Type</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td><code>transactionHash</code>
   </td>
   <td><code>string</code>
   </td>
   <td>The transaction hash of a Bitcoin transaction
   </td>
  </tr>
</table>

This function takes a transaction hash and searches the blockchain for the locking script. If no timelock is found, it throws an error.

Returns:

<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Type</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td><code>timelocks</code>
   </td>
   <td><code>number[]</code>
   </td>
   <td>The timelock timestamp for the transaction 
   </td>
  </tr>
<tr>
   <td><code>conditions</code>
   </td>
   <td><code>string[]</code>
   </td>
   <td>The conditions to spend the transaction, once the timelock(s) are reached
   </td>
  </tr>
</table>

#### getCalendarInvite

This function gets an iCal file for a particular timelock. Each transaction could have multiple timelocks, and thus multiple calendar files.

<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Type</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td><code>timelock</code>
   </td>
   <td><code>Date</code>
   </td>
   <td> The reminder date 
   </td>
  </tr>
<tr>
   <td><code>condition</code>
   </td>
   <td><code>string</code>
   </td>
   <td>The conditions to spend the transaction once the timelock is reached
   </td>
  </tr>
<tr>
   <td><code>transactionHash</code>
   </td>
   <td><code>string</code>
   </td>
   <td>The transaction hash for the particular transaction 
   </td>
  </tr>
</table>

Returns: 

<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Type</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td><code>iCal</code>
   </td>
   <td><code>string</code>
   </td>
   <td>The iCal file data
   </td>
  </tr>
</table>

### getTimeLocksFromAddressUTXOs
This function gets UTXOs from an address and finds the timelock conditions. 

<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Type</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td><code>address</code>
   </td>
   <td><code>string</code>
   </td>
   <td>The bitcoin address
   </td>
  </tr>
</table>


```ts
type UTXOsWithTimeLocks = {
    txHashes: string[],
    timelocks: number[],
    conditions: string[],
}
```

Returns:
<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Type</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td><code>UTXOsWithTimeLocks</code>
   </td>
   <td><code>UTXOsWithTimeLocks</code>
   </td>
   <td>An object of type UTXOsWithTimeLocks
   </td>
  </tr>
</table>

### Database
This project uses MongoDB.
