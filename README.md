<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LT Car Import Calculator</title>
    <style>
        body { font-family: sans-serif; max-width: 600px; margin: 20px auto; padding: 20px; line-height: 1.4; background: #f4f4f9; }
       .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
       .field { margin-bottom: 15px; }
        label { display: block; font-weight: bold; margin-bottom: 5px; font-size: 0.9em; }
        input, select { width: 100%; padding: 8px; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box; }
       .result-row { display: flex; justify-content: space-between; border-bottom: 1px solid #eee; padding: 8px 0; }
       .total { font-size: 1.2em; font-weight: bold; color: #2c3e50; border-top: 2px solid #2c3e50; margin-top: 10px; }
       .highlight { color: #e67e22; }
    </style>
</head>
<body>

<div class="card">
    <h2>LT Import Calculator (2026)</h2>
    
    <div class="field">
        <label>Purchase Price (Value at Port)</label>
        <input type="number" id="price" value="20000" oninput="calculate()">
        <select id="currency" onchange="calculate()">
            <option value="EUR">EUR</option>
            <option value="USD">USD</option>
        </select>
    </div>

    <div class="field">
        <label>Shipping / Ocean Freight</label>
        <input type="number" id="shipping" value="2500" oninput="calculate()">
    </div>

    <div class="field">
        <label>Vehicle Origin (Duty)</label>
        <select id="origin" onchange="calculate()">
            <option value="0">South Korea (KDM) - 0% Duty</option>
            <option value="0.10">Re-import/Other (Germany/USA) - 10% Duty</option>
        </select>
    </div>

    <div class="field">
        <label>Fuel Type</label>
        <select id="fuel" onchange="calculate()">
            <option value="petrol">Petrol</option>
            <option value="diesel">Diesel</option>
            <option value="electric">Electric (BEV)</option>
        </select>
    </div>

    <div class="field">
        <label>CO2 Emissions (g/km)</label>
        <input type="number" id="co2" value="150" oninput="calculate()">
    </div>

    <div class="field">
        <label>Your Middleman Fee</label>
        <input type="number" id="fee" value="1200" oninput="calculate()">
    </div>

    <hr>

    <div id="results">
        <div class="result-row"><span>CIF Value (Basis for Tax):</span> <span id="res-cif">0</span></div>
        <div class="result-row"><span>Import Duty:</span> <span id="res-duty">0</span></div>
        <div class="result-row"><span>Import VAT (21%):</span> <span id="res-vat">0</span></div>
        <div class="result-row"><span>Pollution Tax (Regitra):</span> <span id="res-pollution">0</span></div>
        <div class="result-row"><span>Local Service Costs*:</span> <span id="res-fixed">€1,295</span></div>
        <div class="result-row total"><span>TOTAL LANDED COST:</span> <span id="res-total">0</span></div>
    </div>
    <p style="font-size: 0.7em; color: #666;">*Includes: Port Handling (€450), Broker (€200), IVVA Legalization (€550), TA Inspection (€30), Regitra Fees (€30+).</p>
</div>

<script>
    const USD_TO_EUR = 0.93;

    function getPollutionTax(co2, fuel) {
        if (fuel === 'electric' |

| co2 <= 130) return 0;
        
        // 2025-2026 Official Rates
        const rates = [
            { limit: 140, petrol: 20.91, diesel: 41.82 },
            { limit: 150, petrol: 41.82, diesel: 83.64 },
            { limit: 160, petrol: 62.73, diesel: 125.46 },
            { limit: 170, petrol: 83.64, diesel: 167.28 },
            { limit: 180, petrol: 104.55, diesel: 209.10 },
            { limit: 190, petrol: 125.46, diesel: 250.92 },
            { limit: 200, petrol: 146.37, diesel: 292.74 },
            { limit: 225, petrol: 354.00, diesel: 418.20 },
            { limit: 250, petrol: 522.00, diesel: 501.84 },
            { limit: 300, petrol: 756.00, diesel: 669.12 },
            { limit: 999, petrol: 870.00, diesel: 752.76 }
        ];

        for (let r of rates) {
            if (co2 <= r.limit) return r[fuel];
        }
        return 0;
    }

    function calculate() {
        let price = parseFloat(document.getElementById('price').value) |

| 0;
        let shipping = parseFloat(document.getElementById('shipping').value) |

| 0;
        let currency = document.getElementById('currency').value;
        let dutyRate = parseFloat(document.getElementById('origin').value);
        let fuel = document.getElementById('fuel').value;
        let co2 = parseInt(document.getElementById('co2').value) |

| 0;
        let serviceFee = parseFloat(document.getElementById('fee').value) |

| 0;

        // Convert USD to EUR if necessary
        if (currency === 'USD') {
            price *= USD_TO_EUR;
            shipping *= USD_TO_EUR;
        }

        const insurance = 186; // Factual estimate for marine insurance (~$200)
        const cif = price + shipping + insurance;
        const duty = cif * dutyRate;
        const vat = (cif + duty) * 0.21;
        const pollutionTax = getPollutionTax(co2, fuel);
        const fixedCosts = 1295; // Factual breakdown from Port/Broker/IVVA/TA/Regitra

        const total = cif + duty + vat + pollutionTax + fixedCosts + serviceFee;

        document.getElementById('res-cif').innerText = '€' + cif.toLocaleString(undefined, {minimumFractionDigits: 2});
        document.getElementById('res-duty').innerText = '€' + duty.toLocaleString(undefined, {minimumFractionDigits: 2});
        document.getElementById('res-vat').innerText = '€' + vat.toLocaleString(undefined, {minimumFractionDigits: 2});
        document.getElementById('res-pollution').innerText = '€' + pollutionTax.toLocaleString(undefined, {minimumFractionDigits: 2});
        document.getElementById('res-total').innerText = '€' + total.toLocaleString(undefined, {minimumFractionDigits: 2});
    }

    window.onload = calculate;
</script>

</body>
</html>
