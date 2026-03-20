<!DOCTYPE html>
<html>
<head>
  <title>MUST Calculator</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    body { font-family: Arial; padding: 15px; max-width: 500px; margin: auto; background:#f7f9fc;}
    h2 { text-align: center; color:#003087;}
    h3 { margin-bottom:5px; }
    label { font-weight:bold; }
    input, select, button {
      width:100%; padding:12px; margin:8px 0; font-size:16px;
      border-radius:6px; border:1px solid #ccc;
    }
    button { background:#003087; color:white; border:none; }
    .box { padding:15px; border-radius:8px; margin-top:15px;}
    .low { background:#d4edda;}
    .medium { background:#fff3cd;}
    .high { background:#f8d7da;}
    .warning { background:#e2e3e5; padding:10px; border-radius:6px; margin-top:10px;}
  </style>
</head>

<body>

<!-- SVUH Dietetics Logo -->
<div style="text-align:center; margin-bottom:15px;">
  <img src="PASTE_FULL_URL_HERE" alt="SVUH Dietetics" style="max-width:200px;">
</div>

<h2>MUST Screening Tool</h2>

<h3>Patient Considerations</h3>
<label>Oedema / fluid overload?</label>
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

<label>Unable to weigh/measure height?</label>
<select id="muac">
  <option value="no">No</option>
  <option value="yes">Yes</option>
</select>

<h3>1. Weight & Height</h3>
<label>Current Weight (kg)</label>
<input type="number" id="weight">

<label>Height (cm)</label>
<input type="number" id="height">

<label>Optional: Ulna length (cm) if height not measurable</label>
<input type="number" id="ulna" placeholder="e.g., 25.5">

<label>Optional: Calf Circumference (cm) if weight/height not measurable</label>
<input type="number" id="calf" placeholder="e.g., 31">

<h3>2. Weight Loss</h3>
<label>Previous Weight (kg) (3–6 months ago)</label>
<input type="number" id="prevWeight">

<label>Or select if known:</label>
<select id="weightLossSelect">
  <option value="auto">Calculate automatically</option>
  <option value="0">&lt;5%</option>
  <option value="1">5–10%</option>
  <option value="2">&gt;10%</option>
  <option value="unknown">Unknown</option>
</select>

<h3>3. Acute Illness</h3>
<select id="acute">
  <option value="0">No</option>
  <option value="2">Yes (no intake &gt;5 days)</option>
</select>

<button onclick="calc()">Calculate MUST</button>

<div id="result"></div>

<script>
function showAmputeeType(){
  let a = document.getElementById("amputee").value;
  document.getElementById("amputeeDiv").style.display = (a==="yes") ? "block" : "none";
}

function calc(){
  let w = parseFloat(document.getElementById("weight").value);
  let hInput = parseFloat(document.getElementById("height").value);
  let ulnaInput = parseFloat(document.getElementById("ulna").value);
  let prevW = parseFloat(document.getElementById("prevWeight").value);
  let wlSelect = document.getElementById("weightLossSelect").value;
  let acute = parseInt(document.getElementById("acute").value);
  let oedema = document.getElementById("oedema").value;
  let amputee = document.getElementById("amputee").value;
  let muac = document.getElementById("muac").value;
  let calfInput = parseFloat(document.getElementById("calf").value);

  let warnings = "";

  if(oedema === "yes"){
    warnings += "⚠ Oedema may falsely increase weight.<br>";
  }

  // MUAC or calf circumference pathway
  if(muac === "yes" || calfInput){
    let riskText = "";
    if(calfInput){
      riskText = calfInput >= 31 ? "Low risk" : "Increased risk";
      warnings += "⚠ Calf circumference used as alternative pathway.<br>";
    } else {
      riskText = ">23.5 cm = Low risk<br>≤23.5 cm = Increased risk";
      warnings += "⚠ MUAC used as alternative pathway.<br>";
    }

    document.getElementById("result").innerHTML =
      `<div class="box medium">
        <b>Alternative Screening Pathway</b><br><br>
        ${riskText}<br><br>
        <b>Action:</b> Consider dietitian referral if concerns
        <div class="warning">${warnings}</div>
      </div>`;
    return;
  }

  if(!w){
    alert("Enter weight");
    return;
  }

  // Amputee adjustment
  if(amputee === "yes"){
    let adjustment = parseFloat(document.getElementById("amputeeType").value);
    w = w / (1 - adjustment/100);
    warnings += "⚠ Weight adjusted for amputation.<br>";
  }

  // Estimate height from ulna
  let h;
  if(!hInput && ulnaInput){
    h = ulnaInput*4.67 + 70.9;
    warnings += "⚠ Height estimated from ulna length.<br>";
  } else {
    h = hInput;
  }

  if(!h){
    alert("Enter height or ulna length");
    return;
  }

  // BMI
  let bmi = w / ((h/100)*(h/100));
  let bmiScore = bmi < 18.5 ? 2 : (bmi < 20 ? 1 : 0);

  if(oedema === "yes" && bmiScore < 2){
    bmiScore += 1;
    warnings += "⚠ BMI score increased due to presence of oedema.<br>";
  }

  // Weight loss scoring
  let wlScore = 0;
  if(wlSelect === "auto" && prevW){
    let percentLoss = ((prevW - w) / prevW) * 100;
    if(percentLoss < 5) wlScore = 0;
    else if(percentLoss < 10) wlScore = 1;
    else wlScore = 2;
    warnings += `Weight loss: ${percentLoss.toFixed(1)}%<br>`;
  } else if(wlSelect === "unknown"){
    wlScore = 1;
    warnings += "⚠ Weight loss unknown — assume moderate risk<br>";
  } else {
    wlScore = parseInt(wlSelect);
  }

  // Total MUST score
  let total = bmiScore + wlScore + acute;

  let text = "";
  let style = "";

  if(total === 0){
    style="low";
    text = "Routine care<br>Rescreen weekly";
  } else if(total === 1){
    style="medium";
    text = "Start food chart<br>Encourage intake<br>Repeat screening";
  } else {
    style="high";
    text = "<b>Refer to dietitian TODAY</b><br>Food + fluid chart<br>Consider supplements<br>Escalate if intake poor";
  }

  document.getElementById("result").innerHTML =
    `<div class="box ${style}">
      <b>MUST Component Scores:</b><br>
      BMI Score: ${bmiScore}<br>
      Weight Loss Score: ${wlScore}<br>
      Acute Illness Score: ${acute}<br>
      <b>Total MUST Score:</b> ${total}<br><br>
      ${text}
      <div class="warning">${warnings}</div>
    </div>`;
}
</script>

</body>
</html>
