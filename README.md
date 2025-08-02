<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>AI Medical Prescription Verification</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 800px;
      margin: 30px auto;
      padding: 10px;
    }
    h1 {
      text-align: center;
    }
    label {
      font-weight: bold;
      margin-top: 15px;
      display: block;
    }
    input[type="text"],
    input[type="number"],
    textarea,
    select {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
      box-sizing: border-box;
    }
    button {
      margin-top: 10px;
      padding: 10px 20px;
      font-size: 1em;
      cursor: pointer;
    }
    .output {
      margin-top: 15px;
      padding: 10px;
      border: 1px solid #ddd;
      background-color: #f9f9f9;
    }
    .warning {
      color: #b94a48;
      background-color: #f2dede;
      padding: 10px;
      margin: 10px 0;
    }
    .success {
      color: #3c763d;
      background-color: #dff0d8;
      padding: 10px;
      margin: 10px 0;
    }
  </style>
</head>
<body>
  <h1>AI Medical Prescription Verification</h1>

  <label for="task-select">Select Task:</label>
  <select id="task-select">
    <option value="interactions">Check Drug Interactions</option>
    <option value="dosage">Dosage Recommendation</option>
    <option value="alternatives">Alternative Medications</option>
    <option value="extract">Extract Drug Info</option>
  </select>

  <div id="task-container"></div>

  <script>
    const baseURL = "http://localhost:8000";

    const container = document.getElementById("task-container");
    const taskSelect = document.getElementById("task-select");

    function createInteractionsUI() {
      container.innerHTML = `
        <label for="drugs-input">Enter drugs separated by commas:</label>
        <input type="text" id="drugs-input" placeholder="aspirin, ibuprofen" />
        <button id="check-btn">Check Interactions</button>
        <div id="output"></div>
      `;

      document.getElementById("check-btn").onclick = async () => {
        const drugsRaw = document.getElementById("drugs-input").value.trim();
        const output = document.getElementById("output");
        output.innerHTML = "";

        if (!drugsRaw) {
          output.innerHTML = '<div class="warning">Please enter at least one drug.</div>';
          return;
        }

        const drugs = drugsRaw.split(",").map(d => d.trim());

        try {
          const res = await fetch(baseURL + "/check-interactions", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ drugs }),
          });

          if (!res.ok) throw new Error("Error from server");

          const data = await res.json();

          if (data.interactions.length === 0) {
            output.innerHTML = '<div class="success">No harmful interactions detected.</div>';
          } else {
            output.innerHTML = data.interactions
              .map(i => `<div class="warning">Interaction between <b>${i.drugs[0]}</b> and <b>${i.drugs[1]}</b>: ${i.interaction}</div>`)
              .join("");
          }
        } catch (error) {
          output.innerHTML = `<div class="warning">Error checking interactions.</div>`;
          console.error(error);
        }
      };
    }

    function createDosageUI() {
      container.innerHTML = `
        <label for="drug-input">Enter Drug Name:</label>
        <input type="text" id="drug-input" placeholder="aspirin" />
        <label for="age-input">Enter Patient Age:</label>
        <input type="number" id="age-input" min="0" max="120" />
        <button id="dosage-btn">Get Dosage</button>
        <div id="output"></div>
      `;

      document.getElementById("dosage-btn").onclick = async () => {
        const drug = document.getElementById("drug-input").value.trim();
        const age = parseInt(document.getElementById("age-input").value);
        const output = document.getElementById("output");
        output.innerHTML = "";

        if (!drug || isNaN(age)) {
          output.innerHTML = '<div class="warning">Please enter both drug name and age.</div>';
          return;
        }

        try {
          const res = await fetch(baseURL + "/dosage-recommendation", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ drug, age }),
          });

          if (!res.ok) {
            const errorData = await res.json();
            output.innerHTML = `<div class="warning">${errorData.detail || "Drug not found or error."}</div>`;
            return;
          }

          const data = await res.json();
          output.innerHTML = `<div class="success">Recommended dosage for <b>${data.age_category}</b> patient: <b>${data.recommended_dosage}</b></div>`;
        } catch (error) {
          output.innerHTML = `<div class="warning">Error getting dosage recommendation.</div>`;
          console.error(error);
        }
      };
    }

    function createAlternativesUI() {
      container.innerHTML = `
        <label for="drug-alt-input">Enter Drug Name:</label>
        <input type="text" id="drug-alt-input" placeholder="aspirin" />
        <button id="alt-btn">Get Alternatives</button>
        <div id="output"></div>
      `;

      document.getElementById("alt-btn").onclick = async () => {
        const drug = document.getElementById("drug-alt-input").value.trim();
        const output = document.getElementById("output");
        output.innerHTML = "";

        if (!drug) {
          output.innerHTML = '<div class="warning">Please enter a drug name.</div>';
          return;
        }

        try {
          const res = await fetch(baseURL + "/alternative-medications", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ drug }),
          });

          if (!res.ok) throw new Error("Error fetching alternatives");

          const data = await res.json();
          if (data.alternatives.length === 0) {
            output.innerHTML = `<div>No alternatives found for <b>${drug}</b>.</div>`;
          } else {
            output.innerHTML = `<div>Alternatives for <b>${drug}</b>:</div><ul>` + 
              data.alternatives.map(a => `<li>${a}</li>`).join("") + 
              `</ul>`;
          }
        } catch (error) {
          output.innerHTML = `<div class="warning">Error fetching alternatives.</div>`;
          console.error(error);
        }
      };
    }

    function createExtractUI() {
      container.innerHTML = `
        <label for="medical-text">Paste medical prescription text here:</label>
        <textarea id="medical-text" rows="6" placeholder="e.g. Take aspirin 100mg twice a day"></textarea>
        <button id="extract-btn">Extract Info</button>
        <div id="output"></div>
      `;

      document.getElementById("extract-btn").onclick = async () => {
        const medicalText = document.getElementById("medical-text").value.trim();
        const output = document.getElementById("output");
        output.innerHTML = "";

        if (!medicalText) {
          output.innerHTML = '<div class="warning">Please enter medical text.</div>';
          return;
        }

        try {
          const res = await fetch(baseURL + "/extract-drug-info", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ medical_text: medicalText }),
          });

          if (!res.ok) throw new Error("Error extracting info");

          const data = await res.json();
          const info = data.extracted_info;

          output.innerHTML = "<b>Extracted Information:</b><ul>" +
            Object.entries(info).map(([k, v]) => `<li><b>${k}</b>: ${v}</li>`).join("") +
            "</ul>";
        } catch (error) {
          output.innerHTML = `<div class="warning">Error extracting information.</div>`;
          console.error(error);
        }
      };
    }

    function loadTaskUI() {
      const task = taskSelect.value;
      switch(task) {
        case "interactions": createInteractionsUI(); break;
        case "dosage": createDosageUI(); break;
        case "alternatives": createAlternativesUI(); break;
        case "extract": createExtractUI(); break;
      }
    }

    taskSelect.addEventListener("change", loadTaskUI);

    // Load initial UI
    loadTaskUI();
  </script>
</body>
</html>
