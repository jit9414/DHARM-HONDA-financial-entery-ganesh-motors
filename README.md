<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Invoice - SHRI GANESH MOTORS</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      margin: 40px;
      background-color: #f9f9f9;
      color: #333;
    }
    .header, .sub-header {
      text-align: center;
    }
    .header {
      font-size: 28px;
      font-weight: bold;
      color: #444;
    }
    .sub-header {
      font-size: 14px;
      margin-bottom: 40px;
    }
    .section {
      display: flex;
      justify-content: space-between;
      margin-bottom: 20px;
      flex-wrap: wrap;
    }
    .section > div {
      width: 48%;
    }
    .label {
      font-weight: bold;
      display: inline-block;
      width: 120px;
    }
    input[type="text"], input[type="number"], input[type="date"] {
      width: 60%;
      padding: 5px;
      margin: 5px 0;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      background: #fff;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 10px;
      text-align: center;
    }
    th {
      background-color: #eee;
    }
    .no-print {
      margin-top: 20px;
    }
    .no-print button {
      padding: 8px 15px;
      margin-right: 10px;
      background: #007BFF;
      color: #fff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    .no-print button:hover {
      background: #0056b3;
    }
    @media print {
      .no-print, .no-print-column {
        display: none !important;
      }
      input {
        border: none;
      }
    }
    .section-title {
      text-align: center;
      font-size: 20px;
      font-weight: bold;
      margin: 30px 0 10px;
    }
    .total-amount {
      margin-top: 20px;
      font-size: 18px;
      text-align: right;
      font-weight: bold;
    }
    .conditions {
      font-size: 12px;
      margin-top: 40px;
    }
    .signature {
      text-align: right;
      margin-top: 60px;
    }
  </style>
</head>
<body>

  <div class="header">SHRI GANESH MOTORS</div>
  <div class="sub-header">Near Bus Stand Bilara Road, Borunda, Jodhpur, Rajasthan 342604 | M. 6376978548</div>

  <div class="section">
    <div>
      <div><span class="label">Customer Name:</span><input type="text" id="custName"></div>
      <div><span class="label">Address:</span><input type="text" id="address"></div>
      <div><span class="label">Mobile No:</span><input type="text" id="mobile"></div>
      <div><span class="label">Frame No:</span><input type="text" id="frame"></div>
      <div><span class="label">Register No:</span><input type="text" id="reg"></div>
      <div><span class="label">Model:</span><input type="text" id="model"></div>
      <div><span class="label">Service Type:</span><input type="text" id="service"></div>
    </div>
    <div>
      <div><span class="label">Invoice No:</span><input type="text" id="invoice"></div>
      <div><span class="label">Date:</span><input type="date" id="date"></div>
    </div>
  </div>

  <div class="section-title">Parts Details</div>
  <table id="partsTable">
    <thead>
      <tr>
        <th>Part Name</th>
        <th>Quantity</th>
        <th>MRP</th>
        <th>Total</th>
        <th class="no-print-column">Action</th>
      </tr>
    </thead>
    <tbody id="partsBody"></tbody>
  </table>

  <div class="no-print">
    <button onclick="addPart()">Add Part</button>
  </div>

  <div id="labourSection">
    <div class="section-title">Labour Details</div>
    <table id="labourTable">
      <thead>
        <tr>
          <th>Labour Name</th>
          <th>Charges</th>
          <th class="no-print-column">Action</th>
        </tr>
      </thead>
      <tbody id="labourBody"></tbody>
    </table>
  </div>

  <div class="no-print">
    <button onclick="addLabour()">Add Labour</button>
    <button onclick="window.print()">Print</button>
  </div>

  <div class="total-amount">
    Total Amount: ₹<span id="totalAmount">0.00</span>
  </div>

  <div class="conditions">
    <p><strong>Terms & Conditions:</strong> Goods once sold will not be taken back. Warranty as per manufacturer.</p>
  </div>

  <div class="signature">
    <p>Authorized Signature</p>
  </div>

  <script>
    function addPart() {
      const tbody = document.getElementById("partsBody");
      const row = document.createElement("tr");
      row.innerHTML = `
        <td><input type="text" name="partName[]"></td>
        <td><input type="number" name="qty[]" oninput="calculateTotal()"></td>
        <td><input type="number" name="mrp[]" step="0.01" oninput="calculateTotal()"></td>
        <td class="partTotal">0.00</td>
        <td class="no-print-column"><button onclick="removeRow(this)">Remove</button></td>
      `;
      tbody.appendChild(row);
    }

    function addLabour() {
      const tbody = document.getElementById("labourBody");
      const row = document.createElement("tr");
      row.innerHTML = `
        <td><input type="text" name="labourName[]"></td>
        <td><input type="number" name="charges[]" step="0.01" oninput="calculateTotal()"></td>
        <td class="no-print-column"><button onclick="removeRow(this)">Remove</button></td>
      `;
      tbody.appendChild(row);
      updateLabourSectionVisibility();
    }

    function removeRow(button) {
      button.closest("tr").remove();
      updateLabourSectionVisibility();
      calculateTotal();
    }

    function updateLabourSectionVisibility() {
      const tbody = document.getElementById("labourBody");
      const section = document.getElementById("labourSection");
      section.style.display = tbody.children.length > 0 ? 'block' : 'none';
    }

    function calculateTotal() {
      let total = 0;

      // Calculate Parts Total
      document.querySelectorAll("#partsTable tbody tr").forEach(row => {
        const qty = parseFloat(row.querySelector('[name="qty[]"]').value) || 0;
        const mrp = parseFloat(row.querySelector('[name="mrp[]"]').value) || 0;
        const partTotal = qty * mrp;
        row.querySelector(".partTotal").innerText = partTotal.toFixed(2);
        total += partTotal;
      });

      // Add Labour Charges
      document.querySelectorAll('[name="charges[]"]').forEach(input => {
        total += parseFloat(input.value) || 0;
      });

      document.getElementById("totalAmount").innerText = total.toFixed(2);
    }

    window.onload = updateLabourSectionVisibility;
  </script>

</body>
</html>
