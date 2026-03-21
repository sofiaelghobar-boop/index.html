<!DOCTYPE html>
<html>
<head>
  <title>MUST Screening Tool</title>
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
    .low { background:#d4edda;}
    .medium { background:#fff3cd;}
    .high { background:#f8d7da;}
    .warning { background:#e2e3e5; padding:10px; border-radius:6px; margin-top:10px;}
  </style>
</head>

<body>

<!-- LOGO -->
<div style="text-align:center;">
  <img src="IMG_4549.jpeg" style="max-width:200px;">
</div>

<h2>SVUH MUST Screening Tool</h2>

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

<h3>1. Measurements</h3>

<input type="number" id="weight" placeholder="Weight (kg)">
<input type="number" id="height" placeholder="Height (cm)">
<input type="number" id="ulna" placeholder="Ulna length (cm optional)">
<input type="number" id="calf" placeholder="Calf circumference (cm optional)">

<h3>2. Weight Loss</h3>

<input type="number" id="prevWeight" placeholder="Previous weight">
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

<h3>4. Intake</h3>
<select id="intake">
  <option value="good">>50% meals</option>
  <option value="poor"><50% meals</option>
</select>

<button onclick="calc()">Calculate MUST</button>

<div id="result"></div>

<script>
function showAmputeeType(){
  document.getElementById("amputeeDiv").style.display =
    document.getElementById("amputee").value==="yes" ? "block" : "none";
}

function calc(){

  let w = parseFloat(document.getElementById("weight").value);
  let hInput = parseFloat(document.getElementById("height").value);
  let ulnaInput = parseFloat(document.getElementById("ulna").value);
  let prevW = parseFloat(document.getElementById("prevWeight").value);
  let wlSelect = document.getElementById("weightLossSelect").value;
  let acute = parseInt(document.getElementById("acute").value);
  let intake = document.getElementById("intake").value;
  let oedema = document.getElementById("oedema").value;
  let amputee = document.getElementById("amputee").value;
  let calf = parseFloat(document.getElementById("calf").value);

  let warnings = "";

  if(oedema === "yes"){
    warnings += "⚠ Oedema may falsely elevate weight.<br>";
  }

  if(!w){
    alert("Enter weight");
    return;
  }

  if(amputee === "yes"){
    let adj = parseFloat(document.getElementById("amputeeType").value);
    w = w / (1 - adj/100);
    warnings += "⚠ Weight adjusted for amputation.<br>";
  }

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

  let bmi = w / ((h/100)*(h/100));

  // BMI category
  let bmiCategory = "";
  if(bmi < 18.5) bmiCategory = "Underweight";
  else if(bmi < 25) bmiCategory = "Healthy range";
  else bmiCategory = "Overweight";

  let bmiScore = bmi < 18.5 ? 2 : (bmi < 20 ? 1 : 0);

  // Weight loss
  let wlScore = 0;
  let percentLoss = 0;

  if(wlSelect === "auto" && prevW){
    percentLoss = ((prevW - w) / prevW) * 100;
    if(percentLoss < 5) wlScore = 0;
    else if(percentLoss < 10) wlScore = 1;
    else wlScore = 2;
  } else if(wlSelect === "unknown"){
    wlScore = 1;
    warnings += "⚠ Weight loss unknown.<br>";
  } else {
    wlScore = parseInt(wlSelect);
  }

  let total = bmiScore + wlScore + acute;

  let style = total===0?"low":(total===1?"medium":"high");

  let action = total===0 ? "Routine care"
             : total===1 ? "Observe, food chart, review"
             : "Refer to dietitian";

  // GLIM
  let phenotypic = (percentLoss > 5) || (bmi < 20) || (calf && calf < 31);
  let etiologic = (intake === "poor") || (acute === 2);

  let glimText = (phenotypic && etiologic)
    ? "<b style='color:red;'>GLIM criteria met (malnutrition likely)</b>"
    : "GLIM criteria not fully met";

  document.getElementById("result").innerHTML =
    `<div class="box ${style}">
      <b>MUST Score: ${total}</b><br><br>

      BMI: ${bmi.toFixed(1)} (${bmiCategory})<br>
      BMI Score: ${bmiScore}<br>
      Weight Loss Score: ${wlScore}<br>
      Acute Score: ${acute}<br><br>

      <b>Intake:</b> ${intake === "good" ? ">50% meals" : "<50% meals"}<br><br>

      <b>Action:</b> ${action}<br><br>

      <b>GLIM:</b><br>${glimText}<br><br>

      <i>Note: In older adults, higher BMI may still be acceptable; interpret clinically.</i>

      <div class="warning">${warnings}</div>
    </div>`;
}
</script>

</body>
</html>
