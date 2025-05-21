<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>InvestApp Demo</title>
<style>
  body {
    margin: 0; padding: 0; font-family: Arial, sans-serif;
    background: #000;
    color: #eee;
    display: flex; flex-direction: column; height: 100vh;
  }
  header {
    background: #111;
    padding: 1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  header h1 {
    margin: 0;
    font-weight: normal;
  }
  nav button {
    background: #222;
    border: none;
    color: #eee;
    margin-left: 1rem;
    padding: 0.5rem 1rem;
    cursor: pointer;
    border-radius: 4px;
  }
  nav button.active, nav button:hover {
    background: #007acc;
  }
  main {
    flex: 1;
    overflow-y: auto;
    padding: 1rem;
  }
  section {
    display: none;
  }
  section.active {
    display: block;
  }
  /* Login form */
  #loginForm {
    max-width: 300px;
    margin: 3rem auto;
    background: #111;
    padding: 1rem 2rem;
    border-radius: 8px;
  }
  #loginForm input {
    width: 100%;
    padding: 0.5rem;
    margin: 0.5rem 0 1rem 0;
    border: none;
    border-radius: 4px;
  }
  #loginForm button {
    width: 100%;
    padding: 0.6rem;
    background: #007acc;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    color: white;
    font-weight: bold;
  }
  /* Business cards */
  .business-card {
    background: #111;
    border: 1px solid #222;
    border-radius: 8px;
    padding: 1rem;
    margin-bottom: 1rem;
  }
  .business-card img {
    max-width: 100%;
    border-radius: 4px;
  }
  .business-card .title {
    font-size: 1.2rem;
    font-weight: bold;
    margin: 0.5rem 0;
  }
  .business-card .phone {
    font-size: 0.9rem;
    color: #66ccff;
  }
  .business-card .description {
    margin: 0.5rem 0;
  }
  /* Upload form */
  #uploadForm input, #uploadForm textarea {
    width: 100%;
    background: #222;
    border: none;
    border-radius: 4px;
    padding: 0.5rem;
    color: #eee;
    margin-bottom: 1rem;
  }
  #uploadForm button {
    background: #007acc;
    border: none;
    padding: 0.7rem 1.5rem;
    color: white;
    border-radius: 4px;
    cursor: pointer;
    font-weight: bold;
  }
  /* Profile */
  #profile h2 {
    margin-top: 0;
  }
  /* Floating businesses */
  #homepage .business-list {
    max-width: 800px;
    margin: 0 auto;
  }
</style>
</head>
<body>

<header>
  <h1>InvestApp Demo</h1>
  <nav>
    <button id="homeBtn" class="active">Homepage</button>
    <button id="uploadBtn">Upload</button>
    <button id="profileBtn">Profile</button>
    <button id="logoutBtn" style="display:none;">Logout</button>
  </nav>
</header>

<main>
  <!-- Login form -->
  <div id="loginSection">
    <form id="loginForm">
      <h2>Login</h2>
      <input type="email" id="email" placeholder="Email" required />
      <input type="password" id="password" placeholder="Password" required />
      <button type="submit">Login / Register</button>
      <p style="font-size: 0.8rem; margin-top:1rem;">* Login or register with any email/password</p>
    </form>
  </div>

  <!-- Homepage -->
  <section id="homepage" class="active">
    <h2>Business Ideas</h2>
    <div class="business-list" id="businessList">
      <!-- Business cards get inserted here -->
    </div>
  </section>

  <!-- Upload -->
  <section id="uploadSection">
    <h2>Upload Business Idea</h2>
    <form id="uploadForm">
      <input type="text" id="title" placeholder="Business Title" required />
      <textarea id="description" placeholder="Business Description" rows="4" required></textarea>
      <input type="text" id="profitData" placeholder="Profits (comma separated numbers, e.g. 10,20,30)" />
      <input type="text" id="phone" placeholder="Business Phone Number" />
      <input type="url" id="imageURL" placeholder="Image URL" />
      <button type="submit">Upload</button>
    </form>
  </section>

  <!-- Profile -->
  <section id="profile">
    <h2>Your Profile & History</h2>
    <h3>Uploaded Businesses</h3>
    <div id="uploadedBusinesses"></div>
    <h3>Visited Businesses</h3>
    <div id="visitedBusinesses"></div>
  </section>
</main>

<script>
  // Simple localStorage "database"
  const STORAGE_USERS = "investapp_users";
  const STORAGE_BUSINESSES = "investapp_businesses";
  const STORAGE_LOGGED_IN = "investapp_loggedInEmail";
  const STORAGE_VISITS = "investapp_visits";

  // Current user
  let currentUser = null;

  // Load data from localStorage or init empty arrays
  let users = JSON.parse(localStorage.getItem(STORAGE_USERS)) || [];
  let businesses = JSON.parse(localStorage.getItem(STORAGE_BUSINESSES)) || [];
  let visits = JSON.parse(localStorage.getItem(STORAGE_VISITS)) || [];

  // Elements
  const loginSection = document.getElementById("loginSection");
  const loginForm = document.getElementById("loginForm");
  const emailInput = document.getElementById("email");
  const passwordInput = document.getElementById("password");
  const logoutBtn = document.getElementById("logoutBtn");

  const homeBtn = document.getElementById("homeBtn");
  const uploadBtn = document.getElementById("uploadBtn");
  const profileBtn = document.getElementById("profileBtn");

  const homepage = document.getElementById("homepage");
  const uploadSection = document.getElementById("uploadSection");
  const profileSection = document.getElementById("profile");

  const businessList = document.getElementById("businessList");
  const uploadedBusinessesDiv = document.getElementById("uploadedBusinesses");
  const visitedBusinessesDiv = document.getElementById("visitedBusinesses");

  // Navigation handlers
  homeBtn.onclick = () => showSection("homepage");
  uploadBtn.onclick = () => {
    if (!currentUser) return alert("Please login first");
    showSection("uploadSection");
  };
  profileBtn.onclick = () => {
    if (!currentUser) return alert("Please login first");
    showSection("profile");
    renderProfile();
  };
  logoutBtn.onclick = () => {
    logout();
  };

  // Show section
  function showSection(sectionId) {
    [homepage, uploadSection, profileSection].forEach(s => s.classList.remove("active"));
    if(sectionId === "homepage") homepage.classList.add("active");
    else if(sectionId === "uploadSection") uploadSection.classList.add("active");
    else if(sectionId === "profile") profileSection.classList.add("active");
  }

  // Login/Register handler
  loginForm.onsubmit = (e) => {
    e.preventDefault();
    const email = emailInput.value.trim().toLowerCase();
    const password = passwordInput.value.trim();

    if (!email || !password) return alert("Please enter email and password");

    // Check if user exists
    let user = users.find(u => u.email === email);

    if (!user) {
      // Register new user (store password as plain text - not secure, just demo)
      user = { email, password, uploaded: [], visited: [] };
      users.push(user);
      saveUsers();
      alert("Registered new user!");
    } else {
      // Check password (plain text, demo only)
      if (user.password !== password) {
        return alert("Wrong password!");
      }
    }

    currentUser = user;
    localStorage.setItem(STORAGE_LOGGED_IN, email);
    loginSection.style.display = "none";
    logoutBtn.style.display = "inline-block";
    renderBusinessList();
  };

  // Logout
  function logout() {
    currentUser = null;
    localStorage.removeItem(STORAGE_LOGGED_IN);
    loginSection.style.display = "block";
    logoutBtn.style.display = "none";
    showSection("homepage");
  }

  // Load logged in user if any
  function loadLoggedInUser() {
    const email = localStorage.getItem(STORAGE_LOGGED_IN);
    if (!email) return;
    const user = users.find(u => u.email === email);
    if (user) {
      currentUser = user;
      loginSection.style.display = "none";
      logoutBtn.style.display = "inline-block";
    }
  }

  // Save users to localStorage
  function saveUsers() {
    localStorage.setItem(STORAGE_USERS, JSON.stringify(users));
  }

  // Save businesses to localStorage
  function saveBusinesses() {
    localStorage.setItem(STORAGE_BUSINESSES, JSON.stringify(businesses));
  }

  // Save visits to localStorage
  function saveVisits() {
    localStorage.setItem(STORAGE_VISITS, JSON.stringify(visits));
  }

  // Render business list on homepage sorted by popularity (visit count)
  function renderBusinessList() {
    if (businesses.length === 0) {
      businessList.innerHTML = "<p>No business ideas uploaded yet.</p>";
      return;
    }
    // Sort by visit count desc (popularity)
    const sorted = [...businesses].sort((a,b) => (b.visitCount || 0) - (a.visitCount || 0));
    businessList.innerHTML = "";
    sorted.forEach(biz => {
      const card = createBusinessCard(biz);
      businessList.appendChild(card);
    });
  }

  // Create business card element
  function createBusinessCard(biz) {
    const card = document.createElement("div");
    card.className = "business-card";

    let imgHTML = "";
    if(biz.imageURL) {
      imgHTML = `<img src="${biz.imageURL}" alt="Business Image" />`;
    }
    card.innerHTML = `
      ${imgHTML}
      <div class="title">${biz.title}</div>
      <div class="phone">Phone: ${biz.phone || "N/A"}</div>
      <div class="description">${biz.description}</div>
      <div>Profits (last periods): ${biz.profitData || "N/A"}</div>
      <button>View Details & Invest</button>
    `;

    // On clicking the button, record visit and alert chat/contact info
    card.querySelector("button").onclick = () => {
      recordVisit(biz.id);
      alert(`Contact the businessman at phone: ${biz.phone}\n\n(Here chat feature can be implemented)`);
    };

    return card;
  }

  // Record visit for current user and increment business visit count
  function recordVisit(bizId) {
    if (!currentUser) return;

    // Add to user's visited array if not already there
    if (!currentUser.visited.includes(bizId)) {
      currentUser.visited.push(bizId);
      saveUsers();
    }

    // Increase visit count for business
    let biz = businesses.find(b => b.id === bizId);
    if (biz) {
      biz.visitCount = (biz.visitCount || 0) + 1;
      saveBusinesses();
    }
  }

  // Upload form handler
  const uploadForm = document.getElementById("uploadForm");
  uploadForm.onsubmit = (e) => {
    e.preventDefault();
    if (!currentUser) return alert("Please login first to upload!");

    const title = document.getElementById("title").value.trim();
    const description = document.getElementById("description").value.trim();
    const profitData = document.getElementById("profitData").value.trim();
    const phone = document.getElementById("phone").value.trim();
    const imageURL = document.getElementById("imageURL").value.trim();

    if (!title || !description) return alert("Title and description are required");

    const newBiz = {
      id: Date.now().toString(),
      title,
      description,
      profitData,
      phone,
      imageURL,
      visitCount: 0,
      ownerEmail: currentUser.email,
    };

    businesses.push(newBiz);
    currentUser.uploaded.push(newBiz.id);
    saveBusinesses();
    saveUsers();

    alert("Business uploaded!");
    uploadForm.reset();
    showSection("homepage");
    renderBusinessList();
  };

  // Render profile section
  function renderProfile() {
    // Uploaded Businesses
    uploadedBusinessesDiv.innerHTML = "";
    if (currentUser.uploaded.length === 0) {
      uploadedBusinessesDiv.innerHTML = "<p>You haven't uploaded any business ideas yet.</p>";
    } else {
      currentUser.uploaded.forEach(id => {
        const biz = businesses.find(b => b.id === id);
        if (biz) {
          const card = createBusinessCard(biz);
          uploadedBusinessesDiv.appendChild(card);
        }
      });
    }

    // Visited Businesses
    visitedBusinessesDiv.innerHTML = "";
    if (currentUser.visited.length === 0) {
      visitedBusinessesDiv.innerHTML = "<p>You haven't visited any business ideas yet.</p>";
    } else {
      currentUser.visited.forEach(id => {
        const biz = businesses.find(b => b.id === id);
        if (biz) {
          const card = createBusinessCard(biz);
          visitedBusinessesDiv.appendChild(card);
        }
      });
    }
  }

  // On page load
  window.onload = () => {
    loadLoggedInUser();
    renderBusinessList();
    if(currentUser){
      loginSection.style.display = "none";
      logoutBtn.style.display = "inline-block";
    }
  };
</script>

</body>
</html>
