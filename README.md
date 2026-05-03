<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lithuania Car Import Calculator (2026)</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; background-color: #f0f2f5; color: #333; margin: 0; padding: 20px; display: flex; justify-content: center; }
       .container { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.08); width: 100%; max-width: 500px; }
        h2 { margin-top: 0; color: #1a73e8; border-bottom: 2px solid #f0f2f5; padding-bottom: 10px; font-size: 1.5rem; }
       .input-group { margin-bottom: 18px; }
        label { display: block; font-weight: 600; margin-bottom: 6px; font-size: 0.9rem; }
        input, select { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; font-size: 1rem; }
        input:focus, select:focus { border-color: #1a73e8; outline: none; box-shadow: 0 0 0 2px rgba(26,115,232,0.2); }
        button { width: 100%; background-color: #1a73e8; color: white; border: none; padding: 12px; border-radius: 6px; font-size: 1.1rem; font-weight: 600; cursor: pointer; transition: background 0.2s; margin-top: 10px; }
        button:hover { background-color: #1557b0; }
       .results { margin-top: 25px; background: #f8f9fa; padding: 20px; border-radius: 8px; }
       .res-row { display: flex; justify-content: space-between; margin-bottom: 10px; font-size: 0.95rem; }
       .res-total { display: flex; justify-content: space-between; margin-top: 15px; padding-top: 15px; border-top: 2px solid #ddd; font-weight: bold; font-size: 1.2rem; color: #d93025; }
       .note { font-size: 0.75rem; color: #666; margin-top: 15px; line-height: 1.4; }
    </style>
</head>
<body>

<div class="container">
    <h2>LT Import Calculator</h2>
    
    <div class="input-group">
        <label>Car Price (EUR)</label>
        <input type="number" id="price" value="20000">
    </div>

    <div class="input-group">
        <label>Shipping / Ocean Freight (EUR)</label>
        <input type="number" id="shipping" value="2500">
    </div>

    <div class="input-group">
        <label>Marine Insurance (EUR)</label>
        <input type="number" id="insurance" value="186">
    </div>

    <div class="input-group">
        <label>Vehicle Origin (Import Duty)</label>
        <select id="origin">
            <option value="0">South Korea (KDM) - 0% Duty</option>
            <option value="0.10">Japan / Germany / Other - 10% Duty</option>
        </select>
    </div>

    <div class="input-group">
        <label>Fuel Type</label>
        <select id="fuel">
            <option value="petrol">Petrol / LPG / CNG</option>
            <option value="diesel">Diesel</option>
            <option value="electric">Electric (BEV)</option>
        </select>
    </div>

    <div class="input-group">
        <label>CO2 Emissions (g/km)</label>
        <input type="number" id="co2" value="150">
    </div>

    <button onclick="calculate()">Calculate Final Cost</button>

    <div class="results" id="results-box" style="display:none;">
        <div class="res-row"><span>Customs Value (CIF):</span> <span id="res-cif">€0.00</span></div>
        <div class="res-row"><span>Import Duty:</span> <span id="res-duty">€0.00</span></div>
        <div class="res-row"><span>Import VAT (21%):</span> <span id="res-vat">€0.00</span></div>
        <div class="res-row"><span>Registration Tax (CO2):</span> <span id="res-pollution">€0.00</span></div>
        <div class="res-row"><span>Legalization Fees*:</span> <span>€615.00</span></div>
        <div class="res-total"><span>TOTAL COST:</span> <span id="res-total">€0.00</span></div>
    </div>

    <p class="note">
        *Legalization includes Regitra registration (~€35), Technical Inspection (~€30), and mandatory IVVA individual approval for non-EU spec cars (~€550). Math follows EU Customs stacking: VAT is applied to (CIF Value + Duty).
    </p>
</div>

<script>
    function getRegistrationTax(co2, fuel) {
        if (fuel === 'electric' |

| co2 <= 130) return 0;
        
        // 2025-2026 Factual Registration Tax Brackets
        const brackets = [
            { limit: 140, petrol: 20.91, diesel: 41.82 },
            { limit: 150, petrol: 41.82, diesel: 83.64 },
            { limit: 160, petrol: 62.73, diesel: 125.46 },
            { limit: 170, petrol: 83.64, diesel: 167.28 },
            { limit: 180, petrol: 104.55, diesel: 209.10 },
            { limit: 190, petrol: 125.46, diesel: 250.92 },
            { limit: 200, petrol: 146.37, diesel: 292.74 },
            { limit: 210, petrol: 167.28, diesel: 334.56 },
            { limit: 220, petrol: 188.19, diesel: 376.38 },
            { limit: 230, petrol: 209.10, diesel: 418.20 },
            { limit: 240, petrol: 230.01, diesel: 460.02 },
            { limit: 250, petrol: 250.92, diesel: 501.84 },
            { limit: 260, petrol: 271.83, diesel: 543.66 },
            { limit: 270, petrol: 292.74, diesel: 585.48 },
            { limit: 280, petrol: 313.65, diesel: 627.30 },
            { limit: 290, petrol: 334.56, diesel: 669.12 },
            { limit: 300, petrol: 355.47, diesel: 710.94 },
            { limit: 999, petrol: 376.38, diesel: 752.76 }
        ];

        for (let b of brackets) {
            if (co2 <= b.limit) return fuel === 'diesel'? b.diesel : b.petrol;
        }
        return 0;
    }

    function calculate() {
        const price = parseFloat(document.getElementById('price').value) |

| 0;
        const shipping = parseFloat(document.getElementById('shipping').value) |

| 0;
        const insurance = parseFloat(document.getElementById('insurance').value) |

| 0;
        const dutyRate = parseFloat(document.getElementById('origin').value);
        const fuel = document.getElementById('fuel').value;
        const co2 = parseInt(document.getElementById('co2').value) |

| 0;

        // 1. Customs Basis (CIF)
        const cif = price + shipping + insurance;
        
        // 2. EU Duties (10% or 0% for FTA)
        const duty = cif * dutyRate;
        
        // 3. Import VAT (Tax on CIF + Duty)
        const vat = (cif + duty) * 0.21;
        
        // 4. Local Registration/Pollution Tax
        const pollutionTax = getRegistrationTax(co2, fuel);
        
        // 5. Fixed Statutory Costs (Regitra + TA + IVVA)
        const fixedCosts = 615;

        const total = cif + duty + vat + pollutionTax + fixedCosts;

        // UI Updates
        document.getElementById('res-cif').innerText = '€' + cif.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
        document.getElementById('res-duty').innerText = '€' + duty.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
        document.getElementById('res-vat').innerText = '€' + vat.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
        document.getElementById('res-pollution').innerText = '€' + pollutionTax.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
        document.getElementById('res-total').innerText = '€' + total.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
        
        document.getElementById('results-box').style.display = 'block';
    }
</script>

</body>
</html>
