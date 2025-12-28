<script>
  // @ts-nocheck

  let authCode = "";
  let token = "";
  let user = null;

  let plate = "";
  let duration = 60;
  let loading = false;

  const durations = [
    { label: "30 Mins", value: 30 },
    { label: "1 Hour", value: 60 },
    { label: "2 Hours", value: 120 },
  ];

  function login() {
    loading = true;

    my.getAuthCode({
      scopes: ["auth_base", "USER_ID"],
      success: (res) => {
        authCode = res.authCode;

        fetch("https://its.mouamle.space/api/auth-with-superQi", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            token: authCode, // نفس مشروع المدرّب
          }),
        })
          .then((res) => res.json())
          .then((data) => {
            token = data.token;
            user = data.record;

            my.alert({
              content: "Login successful",
            });
          })
          .catch((err) => {
            my.alert({
              content: "Login failed",
            });
            console.error(err);
          })
          .finally(() => {
            loading = false;
          });
      },
      fail: (err) => {
        loading = false;
        my.alert({
          content: "Auth failed",
        });
        console.error(err);
      },
    });
  }

  function pay() {
    if (!token) {
      my.alert({ content: "Please login first" });
      return;
    }

    fetch("https://its.mouamle.space/api/payment", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: token, // نفس المدرّب
      },
    })
      .then((res) => res.json())
      .then((data) => {
        my.tradePay({
          paymentUrl: data.url, // نفس المدرّب
          success: () => {
            my.alert({
              content: "Payment successful",
            });
          },
        });
      })
      .catch((err) => {
        my.alert({
          content: "Payment failed",
        });
        console.error(err);
      });
  }
</script>

<main>
  <header>
    <h1>ParkUp</h1>
    <p>Smart Parking Mini App</p>
  </header>

  <section class="card">
    {#if !token}
      <button class="btn-primary" on:click={login} disabled={loading}>
        {loading ? "Connecting..." : "Login with SuperQi"}
      </button>
    {:else}
      <div class="user-info">
        <span>Logged in</span>
        <strong>ID: {user?.id}</strong>
      </div>
    {/if}
  </section>

  <section class="card booking-card">
    <h2>Book Parking</h2>

    <input
      type="text"
      placeholder="Plate Number"
      bind:value={plate}
      class="input"
    />

    <div class="durations">
      {#each durations as d}
        <button
          class:selected={duration === d.value}
          on:click={() => (duration = d.value)}
        >
          {d.label}
        </button>
      {/each}
    </div>

    <button class="btn-pay" on:click={pay}>
      Pay & Park
    </button>
  </section>
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: "Inter", "Segoe UI", system-ui, -apple-system, sans-serif;
    background: #f5f7fb;
    color: #0b1b3a;
  }

  main {
    max-width: 520px;
    margin: 32px auto;
    padding: 20px;
  }

  header {
    text-align: center;
    margin-bottom: 24px;
  }

  h1 {
    margin: 0;
    font-size: 32px;
    letter-spacing: -0.02em;
  }

  p {
    margin: 6px 0 0;
    color: #4b5563;
    font-weight: 500;
  }

  .card {
    background: #ffffff;
    padding: 22px 22px 18px;
    border-radius: 18px;
    margin-bottom: 20px;
    box-shadow: 0 20px 60px rgba(11, 27, 58, 0.08);
    border: 1px solid #e3e8ef;
  }

  .booking-card {
    display: flex;
    flex-direction: column;
    gap: 14px;
    background: linear-gradient(180deg, #f8fbff 0%, #ffffff 100%);
    border: 1px solid #e3e8ef;
  }

  .booking-card h2 {
    margin: 0;
    color: #0b1b3a;
    letter-spacing: -0.01em;
  }

  .btn-primary,
  .btn-pay {
    width: 100%;
    padding: 14px;
    border-radius: 12px;
    border: none;
    font-size: 16px;
    margin-top: 10px;
    font-weight: 600;
    cursor: pointer;
    transition: transform 150ms ease, box-shadow 150ms ease, background 150ms ease;
    font-family: "Inter", "Segoe UI", system-ui, -apple-system, sans-serif;
  }

  .btn-primary {
    background: linear-gradient(135deg, #2563eb, #1d4ed8);
    color: white;
    box-shadow: 0 10px 25px rgba(37, 99, 235, 0.2);
  }

  .btn-primary:hover:not(:disabled),
  .btn-pay:hover {
    transform: translateY(-1px);
  }

  .btn-primary:active:not(:disabled),
  .btn-pay:active {
    transform: translateY(0);
  }

  .btn-primary:disabled {
    opacity: 0.65;
    cursor: not-allowed;
    box-shadow: none;
  }

  .btn-pay {
    background: linear-gradient(135deg, #f97316, #f59e0b);
    color: white;
    box-shadow: 0 10px 25px rgba(249, 115, 22, 0.25);
  }

  .user-info {
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #1e293b;
  }

  .input {
  width: 100%;
  box-sizing: border-box; /* ⭐ الحل الأساسي */
  padding: 13px 14px;
  border-radius: 12px;
  border: 1px solid #d7deeb;
  background: #f8fafc;
  font-size: 15px;
  font-family: "Inter", "Segoe UI", system-ui, -apple-system, sans-serif;

  outline: none;
  box-shadow: none;
}

.input:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.18);
  background: #ffffff;
}


/* نحذف أي focus افتراضي من المتصفح */
.input:focus-visible {
  outline: none;
}


  .durations {
    display: flex;
    gap: 10px;
    padding: 2px;
    background: #f0f4fa;
    border-radius: 14px;
    border: 1px solid #e3e8ef;
  }

  .durations button {
    flex: 1;
    padding: 12px;
    border-radius: 10px;
    border: 1px solid transparent;
    background: #ffffff;
    font-weight: 600;
    color: #0b1b3a;
    cursor: pointer;
    transition: all 150ms ease;
    font-family: "Inter", "Segoe UI", system-ui, -apple-system, sans-serif;
  }

  .durations button:hover {
    border-color: #2563eb;
    background: #eef2ff;
  }

  .durations button.selected {
    background: linear-gradient(135deg, #2563eb, #1d4ed8);
    color: white;
    border-color: #1d4ed8;
    box-shadow: 0 10px 25px rgba(37, 99, 235, 0.25);
  }
</style>
