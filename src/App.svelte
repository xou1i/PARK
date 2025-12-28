<script>
  let token = "";
  let user = null;
  let plate = "";
  let duration = 60;
  let loading = false;
  let price = 0;

  const durations = [
    { label: "30 Mins", value: 30 },
    { label: "1 Hour", value: 60 },
    { label: "2 Hours", value: 120 },
  ];

  $: price = (duration / 30) * 1.5;

  async function login() {
    loading = true;
    try {
      const authCode = await new Promise((resolve, reject) => {
        my.getAuthCode({
          scopes: ["auth_base", "USER_ID"],
          success: (res) => resolve(res.authCode),
          fail: (err) => reject(err),
        });
      });

      const res = await fetch(
        "https://its.mouamle.space/api/auth-with-superQi",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ token: authCode }),
        },
      );

      const data = await res.json();

      if (data.token) {
        token = data.token;
        user = data.record;
        my.alert({ content: "Login successful" });
      } else {
        my.alert({ content: "Login failed" });
      }
    } catch (err) {
      console.error(err);
      my.alert({ content: "Auth error" });
    } finally {
      loading = false;
    }
  }

  async function pay() {
    if (!token || loading) return;
    loading = true;

    try {
      const res = await fetch("https://its.mouamle.space/api/payment", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: token,
        },
      });

      const data = await res.json();

      if (data.url) {
        my.tradePay({
          paymentUrl: data.url,
          success: () => {
            my.alert({ content: "Payment successful" });
          },
          fail: (err) => {
            console.error(err);
            my.alert({ content: "Payment failed" });
          },
        });
      } else {
        my.alert({ content: "No payment URL returned" });
      }
    } catch (err) {
      console.error(err);
      my.alert({ content: "Payment error" });
    } finally {
      loading = false;
    }
  }
</script>

<main>
  <header>
    <h1>ParkUp</h1>
    <p>Smart Parking Mini App</p>
  </header>

  <section class="card auth-section">
    {#if !token}
      <button class="btn-primary" on:click={login} disabled={loading}>
        {loading ? "Connecting..." : "Login with SuperQi"}
      </button>
    {:else}
      <div class="user-info">
        <span class="label">Logged in as</span>
        <span class="user-id">ID: {user?.id || "Unknown"}</span>
      </div>
    {/if}
  </section>

  <section class="card parking-section">
    <h2>Book Parking</h2>

    <div class="input-group">
      <label for="plate">Plate Number</label>
      <input
        id="plate"
        type="text"
        bind:value={plate}
        placeholder="ABC-1234"
        class="input-field"
      />
    </div>

    <div class="input-group">
      <label>Duration</label>
      <div class="duration-options">
        {#each durations as opt}
          <button
            class="duration-btn"
            class:active={duration === opt.value}
            on:click={() => (duration = opt.value)}
          >
            {opt.label}
          </button>
        {/each}
      </div>
    </div>

    <div class="price-display">
      <span>Total</span>
      <span class="amount">${price.toFixed(2)}</span>
    </div>

    <button
      class="btn-pay"
      disabled={!token || !plate || loading}
      on:click={pay}
    >
      {#if !token}
        Login to Pay
      {:else}
        {loading ? "Processing..." : "Pay & Park"}
      {/if}
    </button>
  </section>
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
      Helvetica, Arial, sans-serif;
    background-color: #f5f7fa;
    color: #333;
  }

  main {
    max-width: 480px;
    margin: 0 auto;
    padding: 20px;
  }

  header {
    margin-bottom: 24px;
    text-align: center;
  }

  h1 {
    margin: 0;
    font-size: 28px;
    font-weight: 700;
  }

  p {
    margin: 4px 0 0;
    color: #666;
    font-size: 14px;
  }

  .card {
    background: white;
    border-radius: 16px;
    padding: 20px;
    margin-bottom: 20px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  }

  .btn-primary {
    width: 100%;
    background: #007bff;
    color: white;
    border: none;
    padding: 14px;
    border-radius: 12px;
    font-weight: 600;
    font-size: 16px;
  }

  .user-info {
    display: flex;
    justify-content: space-between;
    background: #f8f9fa;
    padding: 12px;
    border-radius: 8px;
  }

  .input-group {
    margin-bottom: 20px;
  }

  .input-field {
    width: 100%;
    padding: 12px;
    border: 1px solid #ddd;
    border-radius: 8px;
  }

  .duration-options {
    display: flex;
    gap: 8px;
  }

  .duration-btn {
    flex: 1;
    padding: 10px;
    background: #f0f2f5;
    border-radius: 8px;
    border: none;
  }

  .duration-btn.active {
    background: #e6f0ff;
    color: #007bff;
  }

  .price-display {
    display: flex;
    justify-content: space-between;
    margin: 24px 0;
  }

  .amount {
    font-size: 24px;
    font-weight: 700;
  }

  .btn-pay {
    width: 100%;
    background: #10b981;
    color: white;
    border: none;
    padding: 16px;
    border-radius: 12px;
    font-size: 18px;
  }

  .btn-pay:disabled {
    background: #ccc;
  }
</style>
