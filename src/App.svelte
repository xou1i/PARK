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

  <section class="card">
    <h2>Book Parking</h2>

    <input
      type="text"
      placeholder="Plate Number"
      bind:value={plate}
      class="input"
    />

    <div class="durations">
      {#each durations as d}
        <button on:click={() => (duration = d.value)}>
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
  body {
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

  .card {
    background: white;
    padding: 20px;
    border-radius: 16px;
    margin-bottom: 20px;
  }

  .btn-primary,
  .btn-pay {
    width: 100%;
    padding: 14px;
    border-radius: 12px;
    border: none;
    font-size: 16px;
    margin-top: 10px;
  }

  .btn-primary {
    background: #007bff;
    color: white;
  }

  .btn-pay {
    background: #10b981;
    color: white;
  }

  .user-info {
    display: flex;
    justify-content: space-between;
  }

  .input {
    width: 100%;
    padding: 12px;
    margin-bottom: 10px;
  }

  .durations button {
    margin-right: 5px;
  }
</style>
