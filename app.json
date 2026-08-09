```javascript
let bankData = null;
let currentUser = null;
let transactions = [];
let balanceHistory = [];

const $ = id => document.getElementById(id);

document.addEventListener("DOMContentLoaded", startApp);

async function startApp() {
  try {
    const response = await fetch("data.json");

    if (!response.ok) {
      throw new Error("Unable to load bank data.");
    }

    bankData = await response.json();

    setupEvents();

    const savedUser = localStorage.getItem("pibUser");

    if (savedUser) {
      currentUser = JSON.parse(savedUser);

      transactions = JSON.parse(
        localStorage.getItem("pibTransactions") || "[]"
      );

      balanceHistory = JSON.parse(
        localStorage.getItem("pibBalanceHistory") || "[]"
      );

      showBank();
    }
  } catch (error) {
    console.error(error);

    $("loginMessage").textContent =
      "Unable to load the demo banking system.";
  }
}


/* EVENTS */

function setupEvents() {

  $("showPassword").addEventListener("click", togglePassword);

  $("loginButton").addEventListener("click", login);

  $("logoutButton").addEventListener("click", logout);

  $("addBalanceButton").addEventListener(
    "click",
    () => openPanel("addBalancePanel")
  );

  $("confirmAddBalance").addEventListener(
    "click",
    addBalance
  );

  $("closeBalance").addEventListener(
    "click",
    () => closePanel("addBalancePanel")
  );

  $("transferButton").addEventListener(
    "click",
    () => openPanel("transferPanel")
  );

  $("sendButton").addEventListener(
    "click",
    sendMoney
  );

  $("closeTransfer").addEventListener(
    "click",
    () => closePanel("transferPanel")
  );

  $("transactionsButton").addEventListener(
    "click",
    () => {
      loadTransactions();
      openPanel("transactionsPanel");
    }
  );

  $("viewAllButton").addEventListener(
    "click",
    () => {
      loadTransactions();
      openPanel("transactionsPanel");
    }
  );

  $("closeTransactions").addEventListener(
    "click",
    () => closePanel("transactionsPanel")
  );

  $("securityButton").addEventListener(
    "click",
    () => openPanel("securityPanel")
  );

  $("closeSecurity").addEventListener(
    "click",
    () => closePanel("securityPanel")
  );

  $("settingsButton").addEventListener(
    "click",
    () => openPanel("settingsPanel")
  );

  $("closeSettings").addEventListener(
    "click",
    () => closePanel("settingsPanel")
  );

  $("printReceipt").addEventListener(
    "click",
    () => window.print()
  );

  $("closeReceipt").addEventListener(
    "click",
    () => closePanel("receiptPanel")
  );

  $("language").addEventListener(
    "change",
    changeLanguage
  );

  $("password").addEventListener(
    "keydown",
    event => {
      if (event.key === "Enter") {
        login();
      }
    }
  );
}


/* LOGIN */

function togglePassword() {
  const input = $("password");
  const button = $("showPassword");

  if (input.type === "password") {
    input.type = "text";
    button.textContent = "Hide";
  } else {
    input.type = "password";
    button.textContent = "Show";
  }
}


function login() {

  const account = $("account").value.trim();
  const password = $("password").value;

  $("loginMessage").textContent = "";

  if (!account || !password) {
    $("loginMessage").textContent =
      "Please enter your account number and password.";

    return;
  }

  const accountData = bankData.accounts.find(
    user =>
      user.account === account &&
      user.password === password
  );

  if (!accountData) {
    $("loginMessage").textContent =
      "Invalid account number or password.";

    return;
  }

  currentUser = {
    account: accountData.account,
    name: accountData.name,
    balance: accountData.balance,
    currency: accountData.currency
  };

  transactions = JSON.parse(
    localStorage.getItem(
      "pibTransactions_" + currentUser.account
    ) || "[]"
  );

  balanceHistory = JSON.parse(
    localStorage.getItem(
      "pibBalanceHistory_" + currentUser.account
    ) || "[]"
  );

  if (balanceHistory.length === 0) {
    balanceHistory.push({
      label: "Opening",
      balance: currentUser.balance
    });
  }

  saveUserData();

  $("account").value = "";
  $("password").value = "";

  showBank();

  showToast("Signed in successfully.");
}


/* DASHBOARD */

function showBank() {

  $("loginPage").classList.add("hidden");
  $("bankPage").classList.remove("hidden");

  $("userName").textContent = currentUser.name;

  $("accountNumber").textContent =
    maskAccount(currentUser.account);

  updateBalance();

  loadRecentTransactions();

  drawChart();

  $("language").value =
    localStorage.getItem("pibLanguage") || "en";
}


function updateBalance() {

  $("balance").textContent =
    formatMoney(currentUser.balance);
}


function maskAccount(account) {

  if (!account) return "••••";

  return "•••• " + account.slice(-4);
}


/* ADD BALANCE */

function addBalance() {

  const amount = Number($("addAmount").value);

  if (!Number.isFinite(amount) || amount <= 0) {
    $("balanceMessage").textContent =
      "Enter a valid amount.";

    return;
  }

  if (amount > 1000000) {
    $("balanceMessage").textContent =
      "Demo limit exceeded.";

    return;
  }

  currentUser.balance += amount;

  addTransaction(
    "Cash In",
    amount,
    "PIB Demo Account",
    "in"
  );

  addBalancePoint("Cash In");

  saveUserData();

  updateBalance();

  $("addAmount").value = "";
  $("balanceMessage").textContent = "";

  loadRecentTransactions();
  drawChart();

  showReceipt(
    "Cash In",
    amount,
    "PIB Demo Account"
  );

  showToast("Balance added successfully.");

  closePanel("addBalancePanel");
}


/* TRANSFER */

function sendMoney() {

  const service = $("transferType").value;
  const target = $("transferTarget").value.trim();
  const amount = Number($("transferAmount").value);

  $("transferMessage").textContent = "";

  if (!target) {
    $("transferMessage").textContent =
      "Enter the destination.";

    return;
  }

  if (!Number.isFinite(amount) || amount <= 0) {
    $("transferMessage").textContent =
      "Enter a valid amount.";

    return;
  }

  if (amount > currentUser.balance) {
    $("transferMessage").textContent =
      "Insufficient demo balance.";

    return;
  }

  currentUser.balance -= amount;

  addTransaction(
    "Transfer",
    amount,
    service + " • " + target,
    "out"
  );

  addBalancePoint(service);

  saveUserData();

  updateBalance();

  $("transferTarget").value = "";
  $("transferAmount").value = "";

  loadRecentTransactions();
  drawChart();

  showReceipt(
    "Transfer",
    amount,
    service + " • " + target
  );

  showToast(
    "Transfer to " + service + " completed."
  );

  closePanel("transferPanel");
}


/* TRANSACTIONS */

function addTransaction(
  type,
  amount,
  destination,
  direction
) {

  const now = new Date();

  const transaction = {
    id: createReference(),
    type: type,
    amount: amount,
    destination: destination,
    direction: direction,
    date: now.toLocaleDateString(),
    time: now.toLocaleTimeString()
  };

  transactions.unshift(transaction);

  if (transactions.length > 50) {
    transactions = transactions.slice(0, 50);
  }
}


function loadRecentTransactions() {

  const box = $("recentTransactions");

  box.innerHTML = "";

  if (transactions.length === 0) {
    box.innerHTML =
      '<p class="empty">No transactions yet.</p>';

    return;
  }

  transactions
    .slice(0, 5)
    .forEach(transaction => {

      box.appendChild(
        createTransactionElement(transaction)
      );

    });
}


function loadTransactions() {

  const box = $("transactionList");

  box.innerHTML = "";

  if (transactions.length === 0) {
    box.innerHTML =
      '<p class="empty">No transactions yet.</p>';

    return;
  }

  transactions.forEach(transaction => {

    box.appendChild(
      createTransactionElement(
        transaction,
        true
      )
    );

  });
}


function createTransactionElement(
  transaction,
  detailed = false
) {

  const item = document.createElement("div");

  item.className = "transaction-item";

  const info = document.createElement("div");

  info.className = "transaction-info";

  const title = document.createElement("strong");

  title.textContent = transaction.type;

  const small = document.createElement("small");

  small.textContent =
    transaction.destination +
    " • " +
    transaction.date +
    " " +
    transaction.time;

  info.appendChild(title);
  info.appendChild(small);

  if (detailed) {

    const reference = document.createElement("small");

    reference.textContent =
      "Ref: " + transaction.id;

    reference.style.display = "block";
    reference.style.marginTop = "4px";

    info.appendChild(reference);
  }

  const amount = document.createElement("div");

  amount.className =
    "transaction-amount " +
    transaction.direction;

  const sign =
    transaction.direction === "in"
      ? "+"
      : "-";

  amount.textContent =
    sign + formatMoney(transaction.amount);

  item.appendChild(info);
  item.appendChild(amount);

  return item;
}


/* BALANCE GRAPH */

function addBalancePoint(label) {

  balanceHistory.push({
    label: label,
    balance: currentUser.balance
  });

  if (balanceHistory.length > 12) {
    balanceHistory.shift();
  }
}


function drawChart() {

  const canvas = $("balanceChart");

  if (!canvas) return;

  const wrapper = canvas.parentElement;

  const width = wrapper.clientWidth;
  const height = wrapper.clientHeight;

  const ratio = window.devicePixelRatio || 1;

  canvas.width = width * ratio;
  canvas.height = height * ratio;

  canvas.style.width = width + "px";
  canvas.style.height = height + "px";

  const ctx = canvas.getContext("2d");

  ctx.scale(ratio, ratio);

  ctx.clearRect(0, 0, width, height);

  if (balanceHistory.length < 2) {

    ctx.fillStyle = "#8fa3bb";
    ctx.font = "12px Arial";
    ctx.textAlign = "center";

    ctx.fillText(
      "Add or transfer money to see activity.",
      width / 2,
      height / 2
    );

    return;
  }

  const padding = {
    top: 20,
    right: 15,
    bottom: 30,
    left: 15
  };

  const chartWidth =
    width - padding.left - padding.right;

  const chartHeight =
    height - padding.top - padding.bottom;

  const values =
    balanceHistory.map(item => item.balance);

  let min = Math.min(...values);
  let max = Math.max(...values);

  if (min === max) {
    min -= 100;
    max += 100;
  }

  const range = max - min;

  min -= range * 0.1;
  max += range * 0.1;

  /* GRID */

  ctx.strokeStyle = "#193654";
  ctx.lineWidth = 1;

  for (let i = 0; i < 4; i++) {

    const y =
      padding.top +
      (chartHeight / 3) * i;

    ctx.beginPath();

    ctx.moveTo(
      padding.left,
      y
    );

    ctx.lineTo(
      width - padding.right,
      y
    );

    ctx.stroke();
  }


  /* LINE */

  ctx.beginPath();

  balanceHistory.forEach((item, index) => {

    const x =
      padding.left +
      (index /
        (balanceHistory.length - 1)) *
        chartWidth;

    const y =
      padding.top +
      chartHeight -
      ((item.balance - min) / (max - min)) *
        chartHeight;

    if (index === 0) {
      ctx.moveTo(x, y);
    } else {
      ctx.lineTo(x, y);
    }

  });

  ctx.strokeStyle = "#2f80ed";
  ctx.lineWidth = 2.5;
  ctx.stroke();


  /* POINTS */

  balanceHistory.forEach((item, index) => {

    const x =
      padding.left +
      (index /
        (balanceHistory.length - 1)) *
        chartWidth;

    const y =
      padding.top +
      chartHeight -
      ((item.balance - min) / (max - min)) *
        chartHeight;

    ctx.beginPath();

    ctx.arc(
      x,
      y,
      3,
      0,
      Math.PI * 2
    );

    ctx.fillStyle = "#ffffff";
    ctx.fill();

    ctx.strokeStyle = "#2f80ed";
    ctx.lineWidth = 2;
    ctx.stroke();

  });
}


/* RECEIPT */

function showReceipt(
  type,
  amount,
  destination
) {

  const reference = transactions[0]
    ? transactions[0].id
    : createReference();

  const now = new Date();

  $("receiptContent").innerHTML = "";

  addReceiptRow(
    "Transaction",
    type
  );

  addReceiptRow(
    "Amount",
    formatMoney(amount)
  );

  addReceiptRow(
    "Destination",
    destination
  );

  addReceiptRow(
    "Date",
    now.toLocaleDateString()
  );

  addReceiptRow(
    "Time",
    now.toLocaleTimeString()
  );

  addReceiptRow(
    "Reference",
    reference
  );

  addReceiptRow(
    "Status",
    "Completed"
  );

  openPanel("receiptPanel");
}


function addReceiptRow(label, value) {

  const row = document.createElement("div");

  const strong = document.createElement("strong");

  strong.textContent = label + ": ";

  const span = document.createElement("span");

  span.textContent = value;

  row.appendChild(strong);
  row.appendChild(span);

  $("receiptContent").appendChild(row);
}


/* SETTINGS */

function changeLanguage() {

  const language =
    $("language").value;

  localStorage.setItem(
    "pibLanguage",
    language
  );

  applyLanguage(language);

  showToast("Language updated.");
}


function applyLanguage(language) {

  if (!bankData || !bankData.languages) {
    return;
  }

  const words =
    bankData.languages[language];

  if (!words) return;

  document.querySelector(".welcome p").textContent =
    words.welcome;

  document.querySelector(".balance-top span").textContent =
    words.balance;

  $("addBalanceButton").textContent =
    "+ " + words.addBalance;

  $("transferButton").textContent =
    words.transfer;

  $("transactionsButton").textContent =
    words.transactions;

  $("securityButton").textContent =
    words.security;

  $("settingsButton").textContent =
    words.settings;

  $("logoutButton").textContent =
    words.logout;
}


/* SECURITY */

function openSecurity() {

  openPanel("securityPanel");

  showToast(
    "Security center opened."
  );
}


/* PANEL CONTROL */

function openPanel(id) {

  const panel = $(id);

  if (!panel) return;

  panel.classList.remove("hidden");

  panel.scrollIntoView({
    behavior: "smooth",
    block: "nearest"
  });
}


function closePanel(id) {

  const panel = $(id);

  if (!panel) return;

  panel.classList.add("hidden");
}


/* LOGOUT */

function logout() {

  localStorage.removeItem("pibUser");

  currentUser = null;
  transactions = [];
  balanceHistory = [];

  $("bankPage").classList.add("hidden");
  $("loginPage").classList.remove("hidden");

  showToast("You have been logged out.");
}


/* STORAGE */

function saveUserData() {

  localStorage.setItem(
    "pibUser",
    JSON.stringify(currentUser)
  );

  localStorage.setItem(
    "pibTransactions_" + currentUser.account,
    JSON.stringify(transactions)
  );

  localStorage.setItem(
    "pibBalanceHistory_" + currentUser.account,
    JSON.stringify(balanceHistory)
  );
}


/* HELPERS */

function formatMoney(amount) {

  return new Intl.NumberFormat(
    "en-PH",
    {
      style: "currency",
      currency: "PHP",
      minimumFractionDigits: 2
    }
  ).format(amount);
}


function createReference() {

  const date =
    Date.now().toString(36).toUpperCase();

  const random =
    Math.random()
      .toString(36)
      .substring(2, 7)
      .toUpperCase();

  return "PIB-" + date + "-" + random;
}


let toastTimer;

function showToast(message) {

  const toast = $("toast");

  toast.textContent = message;

  toast.classList.add("show");

  clearTimeout(toastTimer);

  toastTimer = setTimeout(() => {

    toast.classList.remove("show");

  }, 2500);
}


/* REDRAW GRAPH */

window.addEventListener(
  "resize",
  () => {

    if (currentUser) {
      drawChart();
    }

  }
);
```
