<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Church Pledge System</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<style>
body {
    font-family: Arial, sans-serif;
    background: #f4f8fb;
    margin: 0;
    padding: 10px;
}

header {
    display: flex;
    align-items: center;
    background: #1e88e5;
    color: white;
    padding: 10px;
    border-radius: 8px;
}

header img {
    height: 45px;
    margin-right: 10px;
}

h1 {
    font-size: 22px;
    margin: 0;
}

.container {
    margin-top: 15px;
}

input, button {
    font-size: 20px;
    padding: 10px;
    margin: 5px 0;
    width: 100%;
    box-sizing: border-box;
}

button {
    background: #1e88e5;
    color: white;
    border: none;
    border-radius: 6px;
}

button:hover {
    background: #1565c0;
}

.table-container {
    max-height: 300px;
    overflow-y: auto;
    margin-top: 10px;
}

table {
    width: 100%;
    border-collapse: collapse;
    font-size: 20px;
}

th {
    background: #1e88e5;
    color: white;
    padding: 8px;
    position: sticky;
    top: 0;
}

td {
    padding: 8px;
    border-bottom: 1px solid #ccc;
}

tr:nth-child(even) {
    background: #eef3f8;
}

.action-btn {
    font-size: 16px;
    padding: 4px 6px;
    margin: 2px;
}
</style>
</head>

<body>

<header>
    <img src="logo.png">
    <h1>Jesus Champions City International Mamba</h1>
</header>

<div class="container" id="projectScreen">
    <input id="projectName" placeholder="Enter Project Name">
    <button onclick="openProject()">Enter Project</button>
</div>

<div class="container" id="pledgeScreen" style="display:none;">
    <h2 id="projectTitle"></h2>

    <input id="name" placeholder="Member Name">
    <input id="phone" placeholder="Phone Number">
    <input id="amount" placeholder="Amount" type="number">

    <button onclick="savePledge()">Save Pledge</button>
    <button onclick="exportPDF()">Export PDF</button>

    <div class="table-container">
        <table>
            <thead>
                <tr>
                    <th>Name</th>
                    <th>Phone</th>
                    <th>Amount</th>
                    <th>Date</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody id="pledgeTable"></tbody>
        </table>
    </div>
</div>

<script>
let currentProject = "";
let editIndex = -1;

function openProject() {
    const p = document.getElementById("projectName").value.trim();
    if (!p) return alert("Enter project name");

    currentProject = p;
    document.getElementById("projectTitle").innerText = "Project: " + p;
    document.getElementById("projectScreen").style.display = "none";
    document.getElementById("pledgeScreen").style.display = "block";
    loadPledges();
}

function savePledge() {
    const name = document.getElementById("name").value;
    const phone = document.getElementById("phone").value;
    const amount = Number(document.getElementById("amount").value);

    if (!name || !phone || !amount) {
        alert("Fill all fields");
        return;
    }

    let data = JSON.parse(localStorage.getItem(currentProject)) || [];

    if (editIndex >= 0) {
        data[editIndex] = { name, phone, amount, date: data[editIndex].date };
        editIndex = -1;
    } else {
        data.push({
            name,
            phone,
            amount,
            date: new Date().toLocaleDateString()
        });
    }

    localStorage.setItem(currentProject, JSON.stringify(data));
    clearInputs();
    loadPledges();
}

function loadPledges() {
    let data = JSON.parse(localStorage.getItem(currentProject)) || [];
    let table = document.getElementById("pledgeTable");
    table.innerHTML = "";

    data.forEach((p, i) => {
        table.innerHTML += `
        <tr>
            <td>${p.name}</td>
            <td>${p.phone}</td>
            <td>${p.amount}</td>
            <td>${p.date}</td>
            <td>
                <button class="action-btn" onclick="editPledge(${i})">✏️</button>
                <button class="action-btn" onclick="deletePledge(${i})">🗑</button>
            </td>
        </tr>`;
    });
}

function editPledge(i) {
    let data = JSON.parse(localStorage.getItem(currentProject));
    document.getElementById("name").value = data[i].name;
    document.getElementById("phone").value = data[i].phone;
    document.getElementById("amount").value = data[i].amount;
    editIndex = i;
}

function deletePledge(i) {
    let data = JSON.parse(localStorage.getItem(currentProject));
    data.splice(i, 1);
    localStorage.setItem(currentProject, JSON.stringify(data));
    loadPledges();
}

function clearInputs() {
    name.value = "";
    phone.value = "";
    amount.value = "";
}

function exportPDF() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    const data = JSON.parse(localStorage.getItem(currentProject)) || [];
    const logo = new Image();
    logo.src = "logo.png";

    logo.onload = () => {
        doc.addImage(logo, "PNG", 10, 8, 20, 20);
        doc.setFontSize(16);
        doc.text("Jesus Champions City International Mamba", 35, 18);
        doc.setFontSize(13);
        doc.text("Project: " + currentProject, 14, 35);

        let y = 45;
        doc.setFontSize(12);
        doc.text("Name", 14, y);
        doc.text("Phone", 60, y);
        doc.text("Amount", 110, y);
        doc.text("Date", 150, y);
        y += 8;

        let total = 0;
        data.forEach(p => {
            doc.text(p.name, 14, y);
            doc.text(p.phone, 60, y);
            doc.text(String(p.amount), 110, y);
            doc.text(p.date, 150, y);
            total += p.amount;
            y += 8;
        });

        doc.text("Total: " + total, 14, y + 10);
        doc.save(currentProject + ".pdf");
    };
}
</script>

</body>
</html>
