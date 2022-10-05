<script>
  import Modal from "../components/modal.svelte";
  const arweave = window.Arweave.init({
    host: "arweave.net",
    port: 443,
    protocol: "https",
  });

  const { WarpWebFactory, LoggerFactory } = window.warp;
  LoggerFactory.INST.logLevel("fatal");

  const warp = WarpWebFactory.memCached(arweave);

  let contract = "";
  let target = "";
  let quantity = 0;
  let addr = "";
  let balance = 0;
  let showConfirm = false;

  async function loadData() {
    if (!window.arweaveWallet) {
      alert("arweave wallet required!");
    }
    await window.arweaveWallet.connect(
      ["ACCESS_ADDRESS", "SIGN_TRANSACTION", "DISPATCH"],
      { name: "quick transfer" }
    );
    addr = await window.arweaveWallet.getActiveAddress();

    const state = await fetch(`https://cache.permapages.app/${contract}`).then(
      (res) => res.json()
    );
    balance = state.balances[addr];
  }
  async function transfer() {
    if (balance < quantity) {
      alert("you do not have enough units to transfer");
    }
    if (quantity < 1) {
      alert("quantity must be greater than 1");
    }
    if (target.length !== 43) {
      alert("wallet addresses are always 43 characters");
    }
    const result = await warp
      .contract(contract)
      .connect("use_wallet")
      .bundleInteraction({
        function: "transfer",
        target,
        qty: quantity,
      });

    await new Promise((r) => setTimeout(r, 500));
    const state = await fetch(`https://cache.permapages.app/${contract}`).then(
      (res) => res.json()
    );
    if (state.balances[target] === quantity) {
      balance = state.balances[addr];
      showConfirm = true;
    } else {
      alert("Error Occurred! Could not transfer");
    }
  }
</script>

<main>
  <section class="hero min-h-screen bg-base-100 items-start">
    <div class="hero-content flex-col">
      <h1 class="text-4xl font-bold">Quick Transfer</h1>
      <p>Quickly Transfer a Percent of your PST to another user</p>
      <form on:submit|preventDefault={transfer} class="w-[400px]">
        <div class="form-control">
          <label for="contract" class="label"> Asset: </label>
          <input
            type="text"
            id="contract"
            class="input input-bordered"
            bind:value={contract}
            required
          />
          <label class="label">
            Asset Identifier
            <button type="button" class="link" on:click={loadData}
              >Load Asset Data</button
            >
          </label>
        </div>
        {#if balance}
          <div class="my-4">Balance: {balance}</div>
        {/if}
        <div class="form-control">
          <label for="target" class="label">Target:</label>
          <input
            type="text"
            id="target"
            class="input input-bordered"
            bind:value={target}
            required
          />
          <label class="label">Target Wallet</label>
        </div>
        <div class="form-control">
          <label for="quantity" class="label">Quantity</label>
          <input
            type="number"
            id="quantity"
            class="input input-bordered"
            bind:value={quantity}
            required
          />
        </div>
        <div class="my-4">
          <button class="btn rounded-none">Submit</button>
        </div>
      </form>
    </div>
  </section>
</main>
<Modal bind:open={showConfirm}>
  <h1>You transfered ownership!</h1>
</Modal>
