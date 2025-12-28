<script>
  // @ts-nocheck

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

  /* =====================
     LOGIN (SUPERQI AUTH)
     ===================== */
  async function login() {
    if (typeof my === "undefined") {
      alert("❌ This MiniApp must be opened inside SuperQi");
      return;
    }

    if (loading) return;
    loading = true;

    try {
      // 1. Get Auth Code
      const authCode = await new Promise((resolve, reject) => {
        my.getAuthCode({
          scopes: ["auth_base", "USER_ID"],
          success: (res) => resolve(res.authCode),
          fail: (err) => reject(err),
        });
      });

      // 2. Send to Backend (EXACT API)
      const res = await fetch(
        "https://its.mouamle.space/api/auth-with-superQi",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ token: authCode }),
        }
      );

      const data = await res.json();

      if (!data.token) {
        throw new Error("No token returned");
      }

      token = data.token;
      user = data.record;

      my.alert({ content: "✅ Login successful" });
    } catch (err) {
      console.error(err);
      my.alert({ content: "❌ Login failed" });
    } finally {
      loading = false;
    }
  }

  /* =====================
     PAYMENT
     ===================== */
  async function pay() {
    if (!token) {
      my.alert({ content: "⚠️ Please login first" });
      return;
    }

    if (!plate) {
      my.alert({ content: "⚠️ Enter plate number" });
      return;
    }

    if (loading) return;
    loading = true;

    try {
      const res = await fetch("https://its.mouamle.space/api/payment", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: token, // ❗ EXACT like trainer
        },
      });

      const data = await res.json();

      if (!data.url) {
        throw new Error("No payment URL");
      }

      my.tradePay({
        paymentUrl: data.url,
        success: () => {
          my.alert({ content: "✅ Payment successful" });
        },
        fail: () => {
          my.alert({ content: "❌ Payment failed" });
        },
      });
    } catch (err) {
      console.error(err);
      my.alert({ content: "❌ Payment error" });
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

  <!-- AUTH CARD -->
  <section class="card">
    {#if !token}
      <button class="btn-primary" on:click={login} disabled={loading}>
        {loading ? "Connecting..." : "Login with SuperQi"}
      </button>
    {:else}
      <div class="user-info">
        <span>Logged in</span>
        <strong>ID: {user?.id || "N/A"}</strong>
      </div>
    {/if}
  </section>

  <!-- PARKING CARD -->
  <section class="card">
    <h2>Book Parking</h2>

    <div class="input-group">
      <label>Plate Number</label>
      <input
        type="text"
        placeholder="ABC-1234"
        bind:value={plate}
        class="input-field"
      />
    </div>

    <div class="input-group">
      <label>Duration</label>
      <div class="duration-options">
        {#each durations as d}
          <button
            class="duration-btn"
            class:active={duration === d.value}
            on:click={() => (duration = d.value)}
          >
            {d.label}
          </button>
        {/each}
      </div>
    </div>

    <div class="price">
      <span>Total</span>
      <strong>${price.toFixed(2)}</strong>
    </div>

    <button
      class="btn-pay"
      on:click={pay}
      disabled={!token || loading}
    >
      {loading ? "Processing..." : "Pay & Park"}
    </button>
  </section>
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: system-ui, sans-serif;
    background: #f5f7fa;
  }

  main {
    max-width: 480px;
    margin: auto;
    padding: 20px;
  }

  header {
    text-align: center;
    margin-bottom: 20px;
  }

  h1 {
    margin: 0;
    font-size: 28px;
  }

  p {
    color: #666;
    font-size: 14px;
  }

  .card {
    background: white;
    padding: 20px;
    border-radius: 16px;
    margin-bottom: 20px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  }

  .btn-primary {
    width: 100%;
    padding: 14px;
    background: #007bff;
    color: white;
    border: none;
    border-radius: 12px;
    font-size: 16px;
    font-weight: 600;
  }

  .user-info {
    display: flex;
    justify-content: space-between;
    background: #f0f2f5;
    padding: 12px;
    border-radius: 8px;
  }

  .input-group {
    margin-bottom: 16px;
  }

  .input-field {
    width: 100%;
    padding: 12px;
    border-radius: 8px;
    border: 1px solid #ddd;
  }

  .duration-options {
    display: flex;
    gap: 8px;
  }

  .duration-btn {
    flex: 1;
    padding: 10px;
    border-radius: 8px;
    border: none;
    background: #f0f2f5;
  }

  .duration-btn.active {
    background: #e6f0ff;
    color: #007bff;
  }

  .price {
    display: flex;
    justify-content: space-between;
    margin: 20px 0;
    font-size: 18px;
  }

  .btn-pay {
    width: 100%;
    padding: 16px;
    background: #10b981;
    color: white;
    border: none;
    border-radius: 12px;
    font-size: 18px;
    font-weight: 600;
  }

  .btn-pay:disabled {
    background: #ccc;
  }
</style>
