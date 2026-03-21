<!DOCTYPE html>
<html>
<head>
  <title>MUST + GLIM Tool</title>
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

<!-- Logo -->
<div style="text-align:center;">
  <img src="IMG_4549.jpeg" style="max-width:200px;">
</div>

<h2>MUST + GLIM Screening Tool</h2>

<h3>Patient Considerations</h3>

<label>Oedema?</label>
<select id="oedema">
  <option value="no">No</option>
  <option value="yes">Yes</option>
</select>

<label>Amputee?</label>
<select id="amputee" onchange="showAmp()">
  <option value="no">No</option>
  <option value="yes">Yes</option>
</select>

<div id="ampDiv" style="display:none;">
  <select id="ampType">
    <option value="16">Entire leg</option>
    <option value="6">Lower leg</option>
    <option value="1.5">Foot</option>
    <option value="5">Arm</option>
  </select>
</div>

<h3>1. Measurements</h3>

<input type="number" id="weight" placeholder="Weight (kg)">
<input type="number" id="height" placeholder="Height (cm)">
<input type="number" id="ulna" placeholder="Ulna (cm optional)">
<input type="number" id="calf" placeholder="Calf circumference (cm optional)">

<h3>2. Weight Loss</h3>

<input type="number" id="prevWeight" placeholder="Previous weight">
<select id="wl">
  <option value="auto">Auto</option>
  <option value="0"><5%</option>
  <option value="1">5–10%</option>
  <option value="2">>10%</option>
  <option value="unknown">Unknown</option>
</select>

<h3>3. Acute Illness</h3>
<select id="acute">
  <option value="0">No</option>
  <option value="2">Yes</option>
</select>

<h3>4. Intake</h3>
<select id="intake">
  <option value="yes">>50%</option>
  <option value="no"><50%</option>
</select>

<button onclick="calc()">Calculate</button>

<div id="result"></div>

<script>
function showAmp(){
  document.getElementById("ampDiv").style.display =
    document.getElementById("amputee").value==="yes" ? "block":"none";
}

function calc(){

  let w = parseFloat(weight.value);
  let h = height.value ? height.value : (ulna.value ? ulna.value*4.67+70.9 : null);
  let prev = parseFloat(prevWeight.value);
  let wl = wl.value;
  let acuteScore = parseInt(acute.value);
  let intake = intake.value;
  let calfVal = parseFloat(calf.value);

  let warnings = "";

  if(oedema.value==="yes"){
    warnings += "⚠ Oedema may affect weight<br>";
  }

  if(!w || !h){
    alert("Enter weight + height/ulna");
    return;
  }

  if(amputee.value==="yes"){
    let adj = parseFloat(ampType.value);
    w = w/(1-adj/100);
    warnings += "⚠ Amputee adjusted<br>";
  }

  let bmi = w/((h/100)*(h/100));
  let bmiScore = bmi<18.5?2:(bmi<20?1:0);

  let wlScore = 0;
  let percentLoss = 0;

  if(wl==="auto" && prev){
    percentLoss = ((prev-w)/prev)*100;
    wlScore = percentLoss<5?0:(percentLoss<10?1:2);
  } else if(wl==="unknown"){
    wlScore=1;
  } else {
    wlScore=parseInt(wl);
  }

  let total = bmiScore + wlScore + acuteScore;

  // Intake warning
  if(intake==="no"){
    warnings += "⚠ Intake <50%<br>";
  }

  // --- GLIM LOGIC ---

  let phenotypic = false;
  let etiologic = false;

  // Phenotypic
  if(percentLoss > 5 || bmi < 20 || (calfVal && calfVal < 31)){
    phenotypic = true;
  }

  // Etiologic
  if(intake==="no" || acuteScore === 2){
    etiologic = true;
  }

  let glimResult = "";
  if(phenotypic && etiologic){
    glimResult = "<b style='color:red;'>GLIM criteria met → Malnutrition likely</b>";
  } else {
    glimResult = "GLIM criteria not fully met";
  }

  let style = total===0?"low":(total===1?"medium":"high");
  let action = total===2?"Refer to dietitian":
               total===1?"Monitor + food chart":"Routine care";

  result.innerHTML = `
    <div class="box ${style}">
      <b>MUST Score: ${total}</b><br><br>
      BMI: ${bmi.toFixed(1)}<br>
      BMI Score: ${bmiScore}<br>
      WL Score: ${wlScore}<br><br>

      <b>${action}</b><br><br>

      <b>GLIM Assessment:</b><br>
      ${glimResult}

      <div class="warning">${warnings}</div>
    </div>
  `;
}
</script>

</body>
</html>
