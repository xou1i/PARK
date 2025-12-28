<script>
  let token = "";
  let user = null;
  let plate = "";
  let duration = 60; // default 1 hour
  let loading = false;
  let price = 0;

  const durations = [
    { label: "30 Mins", value: 30 },
    { label: "1 Hour", value: 60 },
    { label: "2 Hours", value: 120 },
  ];

  $: price = (duration / 30) * 1.5; // $1.50 per 30 mins

  async function login() {
    loading = true;
    try {
      // 1. Get Auth Code
      const authCode = await new Promise((resolve, reject) => {
        if (typeof my !== "undefined") {
          my.getAuthCode({
            scopes: ["auth_base", "USER_ID"],
            success: (res) => resolve(res.authCode),
            fail: (err) => reject(err),
          });
        } else {
          console.warn("SuperQi env not found, mocking auth");
          setTimeout(() => resolve("mock-auth-code"), 500);
        }
      });

      // 2. Backend Auth
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
      } else {
        console.error("Auth failed", data);
      }
    } catch (err) {
      console.error("Login error", err);
    } finally {
      loading = false;
    }
  }

  async function pay() {
    if (!token || loading) return;
    loading = true;
    try {
      // 3. Payment Request
      const res = await fetch("https://its.mouamle.space/api/payment", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: token,
        },
      });

      const data = await res.json();

      // 4. Open Payment Window
      if (data.url) {
        if (typeof my !== "undefined") {
          my.tradePay({
            paymentUrl: data.url,
          });
        } else {
          console.log("Opening mock payment:", data.url);
          alert(`Opening Payment: ${data.url}`);
        }
      }
    } catch (err) {
      console.error("Payment error", err);
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
      <label for="duration">Duration</label>
      <div class="duration-options">
        {#each durations as opt}
          <button
            class:active={duration === opt.value}
            on:click={() => (duration = opt.value)}
            class="duration-btn"
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
    color: #1a1a1a;
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

  h2 {
    margin: 0 0 16px;
    font-size: 18px;
    font-weight: 600;
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
    cursor: pointer;
    transition: background 0.2s;
  }

  .btn-primary:active {
    background: #0056b3;
  }

  .user-info {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #f8f9fa;
    padding: 12px;
    border-radius: 8px;
  }

  .label {
    color: #666;
    font-size: 14px;
  }

  .user-id {
    font-weight: 600;
    color: #333;
  }

  .input-group {
    margin-bottom: 20px;
  }

  label {
    display: block;
    margin-bottom: 8px;
    font-size: 14px;
    font-weight: 500;
    color: #555;
  }

  .input-field {
    width: 100%;
    padding: 12px;
    border: 1px solid #ddd;
    border-radius: 8px;
    font-size: 16px;
    box-sizing: border-box;
    transition: border-color 0.2s;
  }

  .input-field:focus {
    border-color: #007bff;
    outline: none;
  }

  .duration-options {
    display: flex;
    gap: 8px;
  }

  .duration-btn {
    flex: 1;
    padding: 10px;
    background: #f0f2f5;
    border: 2px solid transparent;
    border-radius: 8px;
    font-size: 13px;
    font-weight: 500;
    cursor: pointer;
    color: #555;
  }

  .duration-btn.active {
    background: #e6f0ff;
    border-color: #007bff;
    color: #007bff;
  }

  .price-display {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin: 24px 0;
    padding-top: 20px;
    border-top: 1px solid #eee;
  }

  .price-display span {
    font-size: 16px;
    color: #666;
  }

  .price-display .amount {
    font-size: 24px;
    font-weight: 700;
    color: #1a1a1a;
  }

  .btn-pay {
    width: 100%;
    background: #10b981;
    color: white;
    border: none;
    padding: 16px;
    border-radius: 12px;
    font-weight: 600;
    font-size: 18px;
    cursor: pointer;
  }

  .btn-pay:disabled {
    background: #ccc;
    cursor: not-allowed;
  }

  .btn-pay:not(:disabled):active {
    background: #059669;
  }
</style>
