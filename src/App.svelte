<script>
	import { onMount } from 'svelte';
	import * as nearAPI from "near-api-js";
	import { WalletConnection } from "near-api-js";
	import Counter from "./Counter.svelte";
	import Header from "./Header.svelte";

	//might need to be popped out to separate component
	//separate view and call functionality
	const { keyStores } = nearAPI;
	const keyStore = new keyStores.BrowserLocalStorageKeyStore();
	const { connect } = nearAPI;
	
	//hardcoded contract info and default account name
	//consider .env
	const accountId = process.env.DEFAULT_ACCOUNT;
	const contractName = process.env.CONTRACT_ACCOUNT;
	const appName = process.env.APP_NAME;
	const viewMethods = process.env.VIEW_METHODS.split(' ');
	const changeMethods = process.env.CHANGE_METHODS.split(' ');

	//pseudo store of all near objects defined here
	let near;
	let wallet;
	let account;
	let contract;

	let config = {
		networkId: process.env.NETWORK_ID,
		keyStore,
		nodeUrl: process.env.NODE_URL,
		walletUrl: process.env.WALLET_URL,
		helperUrl: process.env.HELPER_URL,
		explorerUrl: process.env.EXPLORER_URL 
	};

	//logic to connect to the chain and initialize account/contract
	$: connectToNear(config)
		.then( Near => {
			near = Near;
			wallet = new WalletConnection(Near, 'creaky')
		});
    //is run whenever a new account is signed into
	$: near && near.account(wallet.getAccountId() || accountId)
		.then( Account => {
			account = Account;
		});
	$: account && initContract(account)
		.then( Contract => contract = Contract );


	onMount(async () => {
	})

	async function connectToNear(config){
		return await connect(config);
	}

	async function initContract(account) {
		return new nearAPI.Contract(
			account,
			contractName, {
				viewMethods,
				changeMethods,
			}
		);
	}
	
</script>

<main class="xl:text-lg 2xl:text-xl">
	<Header bind:wallet {contractName} {appName}></Header>
	<div class="font-mono h-3/4 md:h-auto">
		<div class="text-center flex justify-evenly items-center flex-col flex-grow-0 md:flex-row p-4 md:p-9 xl:p-14 h-full">
			<div class="p-4 md:px-4">
				<h1 class="font-bold text-2xl xl:text-4xl 2xl:text-5xl pb-6 xl:pb-9">
					Hello {(wallet && wallet.getAccountId()) || 'Friend'}!
				</h1>
				This is a simple counter dapp running on the NEAR testnet. 
				{#if !wallet || !wallet.isSignedIn()}
					<br> Sign in to your Near account to increase or decrease the count!
				{:else}
					<br> You can increase or decrease the count with some gas!
				{/if}
			</div>

			<div class="w-3/4 md:w-auto">
				{#if contract}
					<Counter {contract} {wallet}></Counter>
				{:else}
					<p>...Connecting to Near...</p>
				{/if}
			</div>
		</div>
	</div>
</main>

<style global lang="postcss">
	@tailwind base;
	@tailwind components;
	@tailwind utilities;

	body {
		padding: 0;
	}

	.btn-purple {
		@apply transition-colors rounded p-2 bg-purple-700 text-purple-100 hover:bg-purple-100 hover:border-purple-700 hover:text-purple-700;
	}
</style>