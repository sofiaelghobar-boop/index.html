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

<div style="text-align:center;">
  <img src="IMG_4549.jpeg" style="max-width:200px;">
</div>

<h2>SVUH MUST Screening Tool</h2>

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
  <select id="amputeeType">
    <option value="16">Entire leg</option>
    <option value="6">Lower leg</option>
    <option value="1.5">Foot</option>
    <option value="5">Entire arm</option>
    <option value="2.3">Forearm</option>
    <option value="0.7">Hand</option>
  </select>
</div>

<h3>Measurements</h3>
<input id="weight" type="number" placeholder="Weight (kg)">
<input id="height" type="number" placeholder="Height (cm)">
<input id="ulna" type="number" placeholder="Ulna (optional)">
<input id="calf" type="number" placeholder="Calf (optional)">

<h3>Weight Loss</h3>
<input id="prevWeight" type="number" placeholder="Previous weight">
<select id="weightLossSelect">
  <option value="auto">Auto</option>
  <option value="0"><5%</option>
  <option value="1">5–10%</option>
  <option value="2">>10%</option>
  <option value="unknown">Unknown</option>
</select>

<h3>Acute</h3>
<select id="acute">
  <option value="0">No</option>
  <option value="2">Yes</option>
</select>

<h3>Intake</h3>
<select id="intake">
  <option value="good">&gt;50% meals</option>
  <option value="poor">&lt;50% meals</option>
</select>

<button onclick="calc()">Calculate</button>

<div id="result"></div>

<script>

function showAmputeeType(){
  document.getElementById("amputeeDiv").style.display =
    document.getElementById("amputee").value==="yes" ? "block" : "none";
}

function calc(){

  let w = parseFloat(document.getElementById("weight").value);
  let h = parseFloat(document.getElementById("height").value);
  let ulna = parseFloat(document.getElementById("ulna").value);
  let prevW = parseFloat(document.getElementById("prevWeight").value);
  let wlSelect = document.getElementById("weightLossSelect").value;
  let acute = parseInt(document.getElementById("acute").value);
  let intake = document.getElementById("intake").value;
  let oedema = document.getElementById("oedema").value;
  let amputee = document.getElementById("amputee").value;
  let calf = parseFloat(document.getElementById("calf").value);

  let warnings = "";

  if(!w){
    alert("Enter weight");
    return;
  }

  if(amputee==="yes"){
    let adj = parseFloat(document.getElementById("amputeeType").value);
    w = w/(1-adj/100);
  }

  if(!h && ulna){
    h = ulna*4.67+70.9;
    warnings += "Height estimated<br>";
  }

  if(!h){
    alert("Enter height or ulna");
    return;
  }

  let bmi = w/((h/100)*(h/100));
  let bmiScore = bmi<18.5?2:(bmi<20?1:0);

  let wlScore = 0;
  let percentLoss = 0;

  if(wlSelect==="auto" && prevW){
    percentLoss=((prevW-w)/prevW)*100;
    wlScore = percentLoss<5?0:(percentLoss<10?1:2);
  } else if(wlSelect==="unknown"){
    wlScore=1;
  } else {
    wlScore=parseInt(wlSelect);
  }

  let total=bmiScore+wlScore+acute;

  let phenotypic = (percentLoss>5)||(bmi<20)||(calf && calf<31);
  let etiologic = (intake==="poor")||(acute===2);

  let glim = (phenotypic && etiologic)
    ? "GLIM criteria met"
    : "GLIM not met";

  document.getElementById("result").innerHTML =
    `<div class="box">
    MUST Score: ${total}<br>
    BMI: ${bmi.toFixed(1)}<br>
    Intake: ${intake==="good"?">50%":"<50%"}<br><br>
    ${glim}
    </div>`;
}

</script>

</body>
</html>
