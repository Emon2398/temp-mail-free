<!DOCTYPE html>
<html>
<head>
  <title>Advanced Temp Mail</title>
  <style>
    body { font-family: Arial; padding: 20px; text-align: center; background: #f9f9f9; }
    #email { font-size: 18px; color: blue; margin: 20px 0; }
    button { padding: 10px 15px; font-size: 16px; margin: 5px; }
    #inbox { margin-top: 30px; text-align: left; max-width: 600px; margin-left: auto; margin-right: auto; }
    .mail { padding: 10px; border: 1px solid #ccc; margin: 10px 0; background: #fff; }
  </style>
</head>
<body>
  <h1>Temp Mail Service</h1>
  <p id="email">Generating email...</p>
  <button onclick="copyEmail()">Copy Email</button>
  <div id="inbox"><h2>Inbox</h2><p>Loading...</p></div>

  <script>
    let emailAddress = "";
    let sessionId = "";
    const inboxDiv = document.getElementById("inbox");

    async function generateTempEmail() {
      const res = await fetch("https://dropmail.me/api/graphql", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          query: `
            mutation {
              introduceSession {
                id
                addresses {
                  address
                }
              }
            }
          `
        })
      });
      const data = await res.json();
      emailAddress = data.data.introduceSession.addresses[0].address;
      sessionId = data.data.introduceSession.id;
      document.getElementById("email").innerText = emailAddress;
      loadInbox();
    }

    async function loadInbox() {
      inboxDiv.innerHTML = "<h2>Inbox</h2><p>Checking mail...</p>";
      const res = await fetch("https://dropmail.me/api/graphql", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          query: `
            query {
              session(id: "${sessionId}") {
                mails {
                  fromAddr
                  subject
                  text
                }
              }
            }
          `
        })
      });
      const data = await res.json();
      const mails = data.data.session.mails;
      if (mails.length === 0) {
        inboxDiv.innerHTML = "<h2>Inbox</h2><p>No messages yet.</p>";
        return;
      }
      inboxDiv.innerHTML = "<h2>Inbox</h2>";
      mails.forEach(mail => {
        inboxDiv.innerHTML += `
          <div class="mail">
            <strong>From:</strong> ${mail.fromAddr}<br>
            <strong>Subject:</strong> ${mail.subject}<br>
            <p>${mail.text}</p>
          </div>
        `;
      });
    }

    function copyEmail() {
      navigator.clipboard.writeText(emailAddress);
      alert("Email copied!");
    }

    // Run at start
    generateTempEmail();
    setInterval(loadInbox, 10000); // auto-refresh inbox every 10 sec
  </script>
</body>
</html>
