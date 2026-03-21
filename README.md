<!DOCTYPE html>
<html>
<head>
  <title>SVUH MUST Screening Tool</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    body { font-family: Arial; padding: 15px; max-width: 500px; margin: auto; background:#f7f9fc;}
    h2 { text-align: center; color:#003087;}
    label { font-weight:bold; }
    input, select, button {
      width:100%; padding:12px; margin:8px 0;
      border-radius:6px; border:1px solid #ccc;
    }
    button { background:#003087; color:white; border:none; }
    .box { padding:15px; border-radius:8px; margin-top:15px;}
    .warning { background:#e2e3e5; padding:10px; border-radius:6px; margin-top:10px;}
    .small { font-size:12px; color:#555; margin-bottom:8px; }
  </style>
</head>

<body>

<!-- SVUH Logo -->
<div style="text-align:center; margin-bottom:15px;">
  <img src="IMG_4549.jpeg" alt="SVUH Dietetics Logo" style="max-width:200px;">
</div>

<h2>SVUH MUST Screening Tool</h2>

<h3>Patient Considerations</h3>

<label>Age (years)</label>
<input type="number" id="age" placeholder="Enter patient age">

<label>Oedema?</label>
<select id="oedema">
  <option value="no">No</option>
  <option value="yes">Yes</option>
</select>

<label>Amputee?</label>
<select id="amputee" onchange="showAmputeeType()">
  <option value="no">No</option>
  <option value="yes">Yes</option>
</select>

<div id="amputeeDiv" style="display:none;">
  <label>Type of amputation</label>
  <select id="amputeeType">
    <option value="16">Entire leg</option>
    <option value="6">Lower leg</option>
    <option value="1.5">Foot</option>
    <option value="5">Entire arm</option>
    <option value="2.3">Forearm</option>
    <option value="0.7">Hand</option>
  </select>
</div>

<h3>1. Measurements</h3>
<input type="number" id="weight" placeholder="Current Weight (kg)">
<input type="number" id="height" placeholder="Height (cm)">
<input type="number" id="ulna" placeholder="Ulna length (cm, optional)">
<input type="number" id="calf" placeholder="Calf circumference (cm, optional)">

<h3>2. Weight Loss</h3>
<input type="number" id="prevWeight" placeholder="Previous Weight (kg)">
<select id="weightLossSelect">
  <option value="auto">Calculate automatically</option>
  <option value="0">&lt;5%</option>
  <option value="1">5–10%</option>
  <option value="2">&gt;10%</option>
  <option value="unknown">Unknown</option>
</select>

<h3>3. Acute Disease Effect</h3>
<select id="acute">
  <option value="0">No</option>
  <option value="2">Yes</option>
</select>
<div class="small">MUST: Consider patients with acute disease where there has been or is likely to be no nutritional intake for &gt;5 days.</div>

<h3>4. Intake</h3>
<select id="intake">
  <option value="good">&gt;50% meals</option>
  <option value="poor">&lt;50% meals</option>
</select>

<h3>5. ONS Intake</h3>
<select id="ons">
  <option value="100">Taking 100%</option>
  <option value="50">&le;50%</option>
  <option value="0">Charted but not taking</option>
  <option value="-1">Not charted</option>
</select>

<button onclick="calc()">Calculate MUST</button>

<div id="result"></div>

<script>
function showAmputeeType() {
  document.getElementById("amputeeDiv").style.display =
    document.getElementById("amputee").value === "yes" ? "block" : "none";
}

function calc() {
  let w = parseFloat(document.getElementById("weight").value);
  let hInput = parseFloat(document.getElementById("height").value);
  let ulnaInput = parseFloat(document.getElementById("ulna").value);
  let prevW = parseFloat(document.getElementById("prevWeight").value);
  let wlSelect = document.getElementById("weightLossSelect").value;
  let acute = parseInt(document.getElementById("acute").value);
  let intake = document.getElementById("intake").value;
  let ons = document.getElementById("ons").value;
  let oedema = document.getElementById("oedema").value;
  let amputee = document.getElementById("amputee").value;
  let calf = parseFloat(document.getElementById("calf").value);
  let age = parseInt(document.getElementById("age").value);

  let warnings = "";

  if(!w || !age){
    alert("Enter weight and age");
    return;
  }

  // Amputee adjustment
  if(amputee === "yes"){
    let adj = parseFloat(document.getElementById("amputeeType").value);
    w = w / (1 - adj/100);
    warnings += "⚠ Weight adjusted for amputation.<br>";
  }

  // Height estimation
  let h;
  if(!hInput && ulnaInput){
    h = ulnaInput*4.67 + 70.9;
    warnings += "⚠ Height estimated from ulna length.<br>";
  } else {
    h = hInput;
  }

  if(!h){
    alert("Enter height or ulna");
    return;
  }

  // BMI calculation
  let bmi = w / ((h/100)*(h/100));
  let bmiLabel = ""; // only show 'Underweight' when applicable
  let bmiScore = 0;

  if(age >= 65){
    if(bmi < 20){ bmiLabel = "Underweight"; bmiScore = 2; }
  } else {
    if(bmi < 18.5){ bmiLabel = "Underweight"; bmiScore = 2; }
  }

  // Add oedema score
  if(oedema === "yes"){ bmiScore += 1; warnings += "⚠ BMI score increased due to oedema.<br>"; }

  // Weight loss score
  let wlScore = 0;
  if(wlSelect === "auto" && prevW){
    let percentLoss = ((prevW - w)/prevW)*100;
    wlScore = percentLoss < 5 ? 0 : percentLoss < 10 ? 1 : 2;
    warnings += `Weight loss: ${percentLoss.toFixed(1)}%<br>`;
  } else if(wlSelect === "unknown"){
    wlScore = 1;
    warnings += "⚠ Weight loss unknown.<br>";
  } else {
    wlScore = parseInt(wlSelect);
  }

  // Total MUST score
  let total = bmiScore + wlScore + acute;

  let action = total === 0 ? "Routine care"
             : total === 1 ? "Observe, food chart, review"
             : "Refer to dietitian";

  // Intake warning
  if(intake === "poor"){ warnings += "⚠ Intake <50% meals.<br>"; }

  // ONS intake warning
  let onsText;
  if(ons === "100"){ onsText = "Taking 100%"; }
  else if(ons === "50"){ onsText = "≤50%"; warnings += "⚠ ONS intake ≤50%.<br>"; }
  else if(ons === "0"){ onsText = "Charted but not taking"; warnings += "⚠ Charted but not taking ONS.<br>"; }
  else{ onsText = "Not charted"; warnings += "⚠ ONS not charted.<br>"; }

  document.getElementById("result").innerHTML = `
    <div class="box">
      <b>MUST Score: ${total}</b><br><br>
      BMI: ${bmi.toFixed(1)} ${bmiLabel ? '('+bmiLabel+')' : ''}<br>
      BMI Score: ${bmiScore}<br>
      Weight Loss Score: ${wlScore}<br>
      Acute Disease Effect: ${acute === 2 ? "Yes" : "No"}<br>
      Intake: ${intake === "good" ? ">50% meals" : "<50% meals"}<br>
      ONS Intake: ${onsText}<br><br>
      <b>Action:</b> ${action}<br><br>
      <div class="warning">${warnings}</div>
    </div>
  `;
}
</script>

</body>
</html>
