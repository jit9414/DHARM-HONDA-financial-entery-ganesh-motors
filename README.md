<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Financial Entry System</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
    }
    table {
      border-collapse: collapse;
      width: 100%;
      margin-bottom: 20px;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
    }
    th {
      background-color: #f2f2f2;
    }
    input[type="text"], input[type="number"], input[type="date"] {
      width: 100%;
      padding: 5px;
    }
    .btn {
      padding: 5px 10px;
      margin-top: 10px;
      cursor: pointer;
    }
    h2 {
      margin-top: 40px;
    }
    .footer-summary {
      font-size: 18px;
      font-weight: bold;
      background: #e6f7ff;
      padding: 10px;
      border: 2px solid #ccc;
    }

    @media print {
      .btn {
        display: none !important;
      }
    }
  </style>
</head>
<body>
  <h1>Financial Data Entry System</h1>
  <label>Financial Year: <input type="text" id="financialYear" placeholder="2024-2025" /></label>
  <br/><br/>

  <div id="sections"></div>

  <button class="btn" onclick="addSection('Jodhpur Tyres Payment', ['Chq No.', 'Date', 'Amount', 'Remark'])">Add Jodhpur Tyres Payment</button>
  <button class="btn" onclick="addSection('Bike Purchase', ['Sr.', 'Invoice No.', 'Date', 'Amount', 'Stock', 'Remark'])">Add Bike Purchase</button>
  <button class="btn" onclick="addSection('Parts Purchase', ['Sr.', 'Invoice No.', 'Date', 'Amount', 'Remark'])">Add Parts Purchase</button>
  <br/><br/>
  <button class="btn" onclick="saveData()">💾 Save Data</button>
  <button class="btn" onclick="window.print()">🖨️ Print</button>
  <button class="btn" onclick="downloadPDF()">⬇️ Download PDF</button>

  <div class="footer-summary" id="finalSummary">🔄 Final Calculation Loading...</div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script>
    let sectionId = 0;
    const columnMap = {};
    const sectionTitleMap = {};

    function addSection(title, columns) {
      sectionId++;
      const secId = 'section' + sectionId;
      columnMap[secId] = columns;
      sectionTitleMap[secId] = title;

      let html = `<h2>${title}</h2>`;
      html += `<table id="${secId}">
        <thead><tr>`;
      columns.forEach(col => html += `<th>${col}</th>`);
      html += `<th>Action</th></tr></thead><tbody></tbody>`;
      html += `<tfoot><tr>`;
      columns.forEach(col => {
        if (col === 'Amount' || col === 'Stock') {
          html += `<td>Total: <span class="total-${col.toLowerCase()}">0</span></td>`;
        } else {
          html += `<td></td>`;
        }
      });
      html += `<td></td></tr></tfoot>`;
      html += `</table>
      <button class="btn" onclick="addRow('${secId}')">Add Row</button><br/><br/>`;

      document.getElementById('sections').insertAdjacentHTML('beforeend', html);
      addRow(secId);
    }

    function addRow(tableId) {
      const columns = columnMap[tableId];
      const table = document.getElementById(tableId).getElementsByTagName('tbody')[0];
      const row = document.createElement('tr');
      columns.forEach(col => {
        let inputType = col.toLowerCase().includes('date') ? 'date' :
                        col.toLowerCase().includes('amount') || col.toLowerCase().includes('stock') ? 'number' : 'text';
        row.innerHTML += `<td><input type="${inputType}" oninput="updateTotal('${tableId}')" /></td>`;
      });
      row.innerHTML += `<td><button class="btn" onclick="this.parentElement.parentElement.remove(); updateTotal('${tableId}')">Remove</button></td>`;
      table.appendChild(row);
      updateTotal(tableId);
    }

    function updateTotal(tableId) {
      const table = document.getElementById(tableId);
      const headers = Array.from(table.querySelectorAll('thead th'));
      const amounts = Array.from(table.querySelectorAll('tbody tr')).map(row => {
        return {
          amount: parseFloat(row.cells[headers.findIndex(h => h.innerText === 'Amount')]?.querySelector('input')?.value || 0),
          stock: parseFloat(row.cells[headers.findIndex(h => h.innerText === 'Stock')]?.querySelector('input')?.value || 0)
        };
      });

      const totalAmount = amounts.reduce((sum, r) => sum + r.amount, 0);
      const totalStock = amounts.reduce((sum, r) => sum + (r.stock || 0), 0);

      if (table.querySelector('tfoot .total-amount'))
        table.querySelector('tfoot .total-amount').innerText = totalAmount.toFixed(2);
      if (table.querySelector('tfoot .total-stock'))
        table.querySelector('tfoot .total-stock').innerText = totalStock;

      updateFinalSummary();
    }

    function updateFinalSummary() {
      let tyresTotal = 0, bikeTotal = 0, partsTotal = 0;

      for (let id in sectionTitleMap) {
        const title = sectionTitleMap[id];
        const table = document.getElementById(id);
        const amountCell = table?.querySelector('tfoot .total-amount');
        const total = parseFloat(amountCell?.innerText || 0);
        if (title.includes("Tyres")) tyresTotal += total;
        else if (title.includes("Bike")) bikeTotal += total;
        else if (title.includes("Parts")) partsTotal += total;
      }

      const totalPayment = bikeTotal + partsTotal;
      const diff = tyresTotal - totalPayment;
      const message = diff >= 0
        ? `✅ ₹${diff.toFixed(2)} Advance Available`
        : `❌ ₹${Math.abs(diff).toFixed(2)} Payment Due`;

      document.getElementById('finalSummary').innerText =
        `Tyres Payment: ₹${tyresTotal.toFixed(2)} | Bike: ₹${bikeTotal.toFixed(2)} | Parts: ₹${partsTotal.toFixed(2)} → ${message}`;
    }

    function saveData() {
      const data = {
        financialYear: document.getElementById('financialYear').value,
        sections: {}
      };

      for (let id in columnMap) {
        const rows = [];
        const table = document.getElementById(id);
        const tbody = table.querySelector('tbody');
        tbody.querySelectorAll('tr').forEach(tr => {
          const rowData = [];
          tr.querySelectorAll('td input').forEach(input => {
            rowData.push(input.value);
          });
          rows.push(rowData);
        });
        data.sections[id] = {
          title: sectionTitleMap[id],
          columns: columnMap[id],
          rows: rows
        };
      }

      localStorage.setItem('financialData', JSON.stringify(data));
      alert("✅ Data saved successfully!");
    }

    function downloadPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      doc.text("Financial Entry System\nYear: " + document.getElementById('financialYear').value, 10, 10);
      doc.html(document.body, {
        callback: function (doc) {
          doc.save("financial-entry.pdf");
        },
        x: 10,
        y: 20,
        width: 180
      });
    }

    window.onload = function() {
      const data = JSON.parse(localStorage.getItem('financialData'));
      if (!data) return;
      document.getElementById('financialYear').value = data.financialYear;

      for (let id in data.sections) {
        const section = data.sections[id];
        addSection(section.title, section.columns);
        const currentId = 'section' + sectionId;
        const table = document.getElementById(currentId).querySelector('tbody');
        table.innerHTML = '';
        section.rows.forEach(rowData => {
          const row = document.createElement('tr');
          section.columns.forEach((col, idx) => {
            let inputType = col.toLowerCase().includes('date') ? 'date' :
                            col.toLowerCase().includes('amount') || col.toLowerCase().includes('stock') ? 'number' : 'text';
            row.innerHTML += `<td><input type="${inputType}" value="${rowData[idx]}" oninput="updateTotal('${currentId}')" /></td>`;
          });
          row.innerHTML += `<td><button class="btn" onclick="this.parentElement.parentElement.remove(); updateTotal('${currentId}')">Remove</button></td>`;
          table.appendChild(row);
        });
        updateTotal(currentId);
      }
    };
  </script>
</body>
</html>
