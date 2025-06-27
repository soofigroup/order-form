
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Event Receipt Form</title>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&family=Hidayatullah&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Roboto', sans-serif;
            background-color: #f7f9fc;
            color: #333;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 800px;
            margin: auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
        }
        h1, h3, .title {
            color: #2c3e50;
            text-align: center;
            font-weight: bold;
        }
        h1 {
            font-family: 'Hidayatullah', sans-serif;
            color: #0f7893;
            font-size: 20px;
        }
        h3 {
            font-size: 10px;
        }
        .title {
            font-size: 16px;
            margin-bottom: 20px;
        }
        input[type="text"], input[type="number"], input[type="date"], textarea {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ccc;
            border-radius: 5px;
            box-sizing: border-box;
            font-size: 16px;
            color: #0f7893;
        }
        input[readonly] {
            background-color: #e9ecef;
            color: #0f7893;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 10px;
            text-align: left;
            color: #0f7893;
        }
        th {
            background-color: #3498db;
            color: white;
        }
        button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 10px 15px;
            margin: 5px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #2980b9;
        }
        .footer {
            margin-top: 20px;
            text-align: center;
            color: #888;
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>
    <script>
        let itemCount = 1;

        function setCurrentDate() {
            const today = new Date().toISOString().split('T')[0];
            document.getElementById("receiptDate").value = today;
        }

        function calculateTotal(row) {
            const rate = parseFloat(row.querySelector(".rate").value) || 0;
            const quantity = parseFloat(row.querySelector(".quantity").value) || 0;
            const totalCell = row.querySelector(".total");
            totalCell.value = (rate * quantity).toFixed(2);
            calculateSubtotalAndTotal();
        }

        function calculateSubtotalAndTotal() {
            const totals = document.querySelectorAll(".total");
            let subtotal = 0;

            totals.forEach(cell => {
                subtotal += parseFloat(cell.value) || 0;
            });

            document.getElementById("subtotal").value = subtotal.toFixed(2);

            const otherCharges = parseFloat(document.getElementById("otherCharges").value) || 0;
            const discount = parseFloat(document.getElementById("discount").value) || 0;
            const totalAmount = subtotal + otherCharges - discount;
            document.getElementById("totalAmount").value = totalAmount.toFixed(2);

            const advance = parseFloat(document.getElementById("advance").value) || 0;
            const balance = totalAmount - advance;
            document.getElementById("balance").value = balance.toFixed(2);
        }

        function addItemRow() {
            itemCount++;
            const tableBody = document.querySelector("table tbody");
            const newRow = document.createElement("tr");

            newRow.innerHTML = ` 
                <td>${itemCount}</td>
                <td><input type="text" /></td>
                <td><input type="number" class="quantity" oninput="calculateTotal(this.closest('tr'))" /></td>
                <td><input type="number" class="rate" oninput="calculateTotal(this.closest('tr'))" /></td>
                <td><input type="number" class="total" readonly /></td>
            `;

            tableBody.appendChild(newRow);
        }

        async function printForm() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            // Add header
            doc.setFont("helvetica", "bold");
            doc.setTextColor("#0f7893");
            doc.setFontSize(45);
            doc.text("SOOFI", 105, 20, { align: "center" });

            // Contact and Instagram details
            doc.setTextColor(0, 0, 0);
            doc.setFontSize(15);
            doc.text("EVENTS & TRAVELS", 105, 30, { align: "center" });
            doc.setFontSize(12);
            doc.text("Phone: 7994 189 193 | 70254 50050", 105, 40, { align: "center" });
            doc.text("Instagram: @soofi_events_", 105, 50, { align: "center" });
            doc.line(10, 55, 200, 55); // Horizontal line

            // Add "Receipt" title
            doc.setFontSize(14);
            doc.text("Receipt", 105, 60, { align: "center" });

            // Receipt details
            const receiptDate = document.getElementById("receiptDate").value;
            const eventDate = document.getElementById("eventDate").value;
            const name = document.getElementById("name").value;
            const venue = document.getElementById("venue").value;
            const guests = document.getElementById("guests").value;

            doc.setFontSize(12);
            doc.text(`Date: ${receiptDate}`, 15, 70);
            doc.text(`Name: ${name}`, 15, 80);
            doc.text(`Venue: ${venue}`, 15, 90);
            doc.text(`Event Date: ${eventDate}`, 15, 100);
            doc.text(`No. of Guests: ${guests}`, 15, 110);

            // Table Data (Exclude Rate and Total columns in PDF)
            const tableData = Array.from(document.querySelectorAll("table tbody tr")).map((row, index) => {
                const description = row.querySelector("input[type='text']").value || "";
                const quantity = row.querySelector(".quantity").value || 0;
                return [index + 1, description, quantity];
            });

            doc.autoTable({
                startY: 130,
                head: [['S.No', 'Item Description', 'Quantity']],
                body: tableData,
                theme: 'grid',
                headStyles: {
                    fillColor: '#0f7893',
                    textColor: '#ffffff',
                },
                bodyStyles: {
                    textColor: '#0f7893',
                },
                alternateRowStyles: {
                    fillColor: '#f2f7f9'
                }
            });

            // Amount details below table
            const subtotal = document.getElementById("subtotal").value;
            const otherCharges = document.getElementById("otherCharges").value;
            const discount = document.getElementById("discount").value;
            const totalAmount = document.getElementById("totalAmount").value;
            const advance = document.getElementById("advance").value;
            const balance = document.getElementById("balance").value;

            let finalY = doc.lastAutoTable.finalY + 10;
            doc.text(`Subtotal: ${subtotal}`, 15, finalY);
            doc.text(`Other Charges: ${otherCharges}`, 15, finalY + 10);
            doc.text(`Discount: ${discount}`, 15, finalY + 20);
            doc.text(`Total Amount: ${totalAmount}`, 15, finalY + 30);
            doc.text(`Advance: ${advance}`, 15, finalY + 40);
            doc.text(`Balance: ${balance}`, 15, finalY + 50);

            // Save as PDF
            doc.save(`Event_Receipt.pdf`);
        }

        document.addEventListener("DOMContentLoaded", function() {
            setCurrentDate();
        });
    </script>
</head>
<body>
    <div class="container">
        <h1>Event Receipt Form</h1>
        <h3>For Event and Travel Booking</h3>
        <div class="title">Receipt Information</div>

        <!-- Receipt Information -->
        <form id="receiptForm">
            <div>
                <label for="receiptDate">Date:</label>
                <input type="date" id="receiptDate" readonly>
            </div>
            <div>
                <label for="eventDate">Event Date:</label>
                <input type="date" id="eventDate">
            </div>
            <div>
                <label for="name">Customer Name:</label>
                <input type="text" id="name">
            </div>
            <div>
                <label for="venue">Venue:</label>
                <input type="text" id="venue">
            </div>
            <div>
                <label for="guests">No. of Guests:</label>
                <input type="number" id="guests">
            </div>

            <!-- Table of Items -->
            <table>
                <thead>
                    <tr>
                        <th>S.No</th>
                        <th>Item Description</th>
                        <th>Quantity</th>
                        <th>Rate</th>
                        <th>Total</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>1</td>
                        <td><input type="text"></td>
                        <td><input type="number" class="quantity" oninput="calculateTotal(this.closest('tr'))"></td>
                        <td><input type="number" class="rate" oninput="calculateTotal(this.closest('tr'))"></td>
                        <td><input type="number" class="total" readonly></td>
                    </tr>
                </tbody>
            </table>

            <button type="button" onclick="addItemRow()">Add Item</button>

            <!-- Amount Details -->
            <div>
                <label for="subtotal">Subtotal:</label>
                <input type="number" id="subtotal" readonly>
            </div>
            <div>
                <label for="otherCharges">Other Charges:</label>
                <input type="number" id="otherCharges" oninput="calculateSubtotalAndTotal()">
            </div>
            <div>
                <label for="discount">Discount:</label>
                <input type="number" id="discount" oninput="calculateSubtotalAndTotal()">
            </div>
            <div>
                <label for="totalAmount">Total Amount:</label>
                <input type="number" id="totalAmount" readonly>
            </div>
            <div>
                <label for="advance">Advance:</label>
                <input type="number" id="advance" oninput="calculateSubtotalAndTotal()">
            </div>
            <div>
                <label for="balance">Balance:</label>
                <input type="number" id="balance" readonly>
            </div>

            <button type="button" onclick="printForm()">Generate PDF</button>
        </form>
    </div>
</body>
</html>
