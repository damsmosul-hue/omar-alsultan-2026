<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>حاسبة غرامات التأخير</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      font-family: Tahoma, sans-serif;
      margin: 0;
      padding: 0;
      background: linear-gradient(to right, #4e54c8, #8f94fb);
      color: #fff;
    }
    header {
      background: rgba(0,0,0,0.3);
      padding: 20px;
      text-align: center;
    }
    header h1 {
      margin: 0;
      font-size: 28px;
    }
    main {
      max-width: 600px;
      margin: 40px auto;
      background: rgba(255,255,255,0.1);
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 0 15px rgba(0,0,0,0.2);
    }
    label {
      display: block;
      margin-top: 15px;
      font-weight: bold;
    }
    select, input, button {
      width: 100%;
      padding: 10px;
      margin-top: 8px;
      font-size: 16px;
      border-radius: 6px;
      border: none;
    }
    button {
      background: #ffcc00;
      font-weight: bold;
      cursor: pointer;
      margin-top: 20px;
    }
    button:hover {
      background: #ffe066;
    }
    .result {
      margin-top: 20px;
      font-size: 18px;
      font-weight: bold;
      background: rgba(0,0,0,0.2);
      padding: 15px;
      border-radius: 8px;
    }
    .note {
      font-size: 14px;
      color: #ffeaa7;
    }
    footer {
      text-align: center;
      padding: 15px;
      background: rgba(0,0,0,0.3);
      margin-top: 40px;
      font-size: 14px;
    }
    .date-selects {
      display: flex;
      gap: 10px;
    }
    .date-selects select {
      flex: 1;
    }
  </style>
</head>
<body>

<header>
  <h1>حاسبة غرامات التأخير</h1>
</header>

<main>
  <label>تاريخ توقيع العقد / شراء الوحدة:</label>
  <div class="date-selects">
    <!-- السنة على اليسار -->
    <select id="year">
      <option value="">السنة</option>
      <script>
        for(let y=2000; y<=2030; y++){
          document.write(`<option value="${y}">${y}</option>`);
        }
      </script>
    </select>

    <!-- الشهر في الوسط -->
    <select id="month">
      <option value="">الشهر</option>
      <script>
        for(let m=1; m<=12; m++){
          document.write(`<option value="${m}">${m}</option>`);
        }
      </script>
    </select>

    <!-- اليوم على اليمين -->
    <select id="day">
      <option value="">اليوم</option>
      <script>
        for(let d=1; d<=31; d++){
          document.write(`<option value="${d}">${d}</option>`);
        }
      </script>
    </select>
  </div>

  <label>نوع الشقة:</label>
  <select id="apartmentType">
    <option value="">-- اختر --</option>
    <option value="2+1">2 + 1</option>
    <option value="3+1">3 + 1</option>
  </select>

  <label>سعر صرف الدولار (دينار):</label>
  <input type="number" id="exchangeRate" placeholder="مثال: 1320">

  <label>الغرامة الشهرية:</label>
  <input type="number" id="monthlyFine" placeholder="أدخل مبلغ الغرامة">

  <select id="currency">
    <option value="USD">دولار</option>
    <option value="IQD">دينار</option>
  </select>

  <button onclick="calculate()">احسب الغرامات</button>

  <div class="result" id="result"></div>

  <div id="stopSection" style="display:none;">
    <label>عدد أيام التوقف:</label>
    <input type="number" id="stopDays" value="0">
    <button onclick="applyStopDays()">احسب المبلغ النهائي</button>
    <div class="result" id="finalResult"></div>
  </div>
</main>

<footer>
  &copy; 2026 جميع الحقوق محفوظة — <strong>Omar Alsultan</strong>
</footer>

<script>
let dailyFineIQD = 0;
let baseAmount = 0;

function calculate() {
  const day = document.getElementById("day").value;
  const month = document.getElementById("month").value;
  const year = document.getElementById("year").value;

  if(!day || !month || !year){
    document.getElementById("result").innerText = "⚠️ يرجى اختيار يوم وشهر وسنة";
    return;
  }

  const purchaseDate = new Date(year, month-1, day);
  const targetDate = new Date("2026-01-01");
  const rate = parseFloat(document.getElementById("exchangeRate").value);
  const monthlyFine = parseFloat(document.getElementById("monthlyFine").value);
  const currency = document.getElementById("currency").value;
  const resultDiv = document.getElementById("result");

  if (purchaseDate > targetDate) {
    resultDiv.innerText = "⚠️ يرجى إدخال تاريخ صحيح";
    return;
  }
  if (isNaN(monthlyFine)) {
    resultDiv.innerText = "⚠️ يرجى إدخال مبلغ الغرامة";
    return;
  }
  if (currency === "USD" && isNaN(rate)) {
    resultDiv.innerText = "⚠️ يرجى إدخال سعر صرف الدولار";
    return;
  }

  // حساب الفرق بالأشهر والأيام
  let months = 0;
  let temp = new Date(purchaseDate);
  while (temp < targetDate) {
    temp.setMonth(temp.getMonth() + 1);
    if (temp <= targetDate) months++;
    else {
      temp.setMonth(temp.getMonth() - 1);
      break;
    }
  }
  let days = Math.floor((targetDate - temp) / (1000 * 60 * 60 * 24));

  // خصم 18 شهر
  let fineMonths = months - 18;
  if (fineMonths < 0) fineMonths = 0;

  // الغرامة اليومية
  let dailyFine = monthlyFine / 30;
  if (currency === "USD") {
    dailyFineIQD = dailyFine * rate;
  } else {
    dailyFineIQD = dailyFine;
  }

  // حساب المبلغ
  baseAmount = (fineMonths * 30 + days) * dailyFineIQD;

  resultDiv.innerHTML = `
    ⏳ <strong>الفرق الكلي</strong><br>
    <span class="note">(من تاريخ توقيع العقد إلى تاريخ 1/1/2026)</span><br>
    عدد الأشهر: ${months} شهر &nbsp;|&nbsp; عدد الأيام: ${days} يوم<br><br>

    ⚠️ <strong>مدة الغرامات بعد خصم 18 شهر</strong>: ${fineMonths} شهر و ${days} يوم<br>
    💰 <strong>المبلغ الابتدائي</strong>: ${baseAmount.toLocaleString()} دينار<br><br>
    <strong>Omar Alsultan</strong>
  `;

  document.getElementById("stopSection").style.display = "block";
}

function applyStopDays() {
  const stopDays = parseInt(document.getElementById("stopDays").value);
  const finalDiv = document.getElementById("finalResult");

  let stopAmount = stopDays * dailyFineIQD;
  let finalAmount = baseAmount - stopAmount;
  if (finalAmount < 0) finalAmount = 0;

  finalDiv.innerHTML = `
    💸 خصم أيام التوقف: ${stopAmount.toLocaleString()} دينار<br>
    ✅ <strong>المبلغ النهائي المستحق</strong>: ${finalAmount.toLocaleString()} دينار<br><br>
    <strong>Omar Alsultan</strong>
  `;
}
</script>

</body>
</html>
