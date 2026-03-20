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

<h2>MUST Screening Tool</h2>

<h3>Patient Considerations</h3>

<label>Oedema / fluid overload?</label>
<select id="oedema">
  <option value="no">No</option>
  <option value="yes">Yes</option>
</select>

<label>Amputee?</label>
<select id="amputee">
  <option value="no">No</option>
  <option value="yes">Yes</option>
</select>

<label>Unable to weigh/measure?</label>
<select id="muac">
  <option value="no">No</option>
  <option value="yes">Yes</option>
</select>

<h3>1. BMI</h3>
<label>Current Weight (kg)</label>
<input type="number" id="weight">

<label>Height (cm)</label>
<input type="number" id="height">

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
  <option value="2">Yes (no intake >5 days)</option>
</select>

<button onclick="calc()">Calculate MUST</button>

<div id="result"></div>

<script>
function calc(){

  let w = parseFloat(document.getElementById("weight").value);
  let h = parseFloat(document.getElementById("height").value);
  let prevW = parseFloat(document.getElementById("prevWeight").value);
  let wlSelect = document.getElementById("weightLossSelect").value;
  let acute = parseInt(document.getElementById("acute").value);
  let oedema = document.getElementById("oedema").value;
  let amputee = document.getElementById("amputee").value;
  let muac = document.getElementById("muac").value;

  let warnings = "";

  if(oedema === "yes"){
    warnings += "⚠ Oedema may falsely increase weight.<br>";
  }

  if(amputee === "yes"){
    warnings += "⚠ Adjust weight for amputation before interpreting BMI.<br>";
  }

  if(muac === "yes"){
    document.getElementById("result").innerHTML =
      `<div class="box medium">
        <b>USE MUAC PATHWAY</b><br><br>
        >23.5 cm = Low risk<br>
        <23.5 cm = Increased risk<br><br>
        <b>Action:</b> Consider dietitian referral if concerns
      </div>`;
    return;
  }

  if(!w || !h){
    alert("Enter weight and height");
    return;
  }

  let bmi = w / ((h/100)*(h/100));
  let bmiScore = bmi < 18.5 ? 2 : (bmi < 20 ? 1 : 0);

  let wlScore = 0;

  if(wlSelect === "auto" && prevW){
    let percentLoss = ((prevW - w) / prevW) * 100;

    if(percentLoss < 5) wlScore = 0;
    else if(percentLoss < 10) wlScore = 1;
    else wlScore = 2;

    warnings += `Weight loss: ${percentLoss.toFixed(1)}%<br>`;
  }
  else if(wlSelect === "unknown"){
    wlScore = 1;
    warnings += "⚠ Weight loss unknown — assume moderate risk<br>";
  }
  else{
    wlScore = parseInt(wlSelect);
  }

  let total = bmiScore + wlScore + acute;

  let text = "";
  let style = "";

  if(total === 0){
    style="low";
    text = "Routine care<br>Rescreen weekly";
  }
  else if(total === 1){
    style="medium";
    text = "Start food chart<br>Encourage intake<br>Repeat screening";
  }
  else{
    style="high";
    text = "<b>Refer to dietitian TODAY</b><br>Food + fluid chart<br>Consider supplements<br>Escalate if intake poor";
  }

  document.getElementById("result").innerHTML =
    `<div class="box ${style}">
      <b>BMI:</b> ${bmi.toFixed(1)}<br>
      <b>Total Score:</b> ${total}<br><br>
      ${text}
      <div class="warning">${warnings}</div>
    </div>`;
}
</script>

</body>
</html>
