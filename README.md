# money-saver
demo website 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Money Saver Tracker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
            max-width: 500px;
            width: 100%;
        }

        h1 { color: #333; text-align: center; margin-bottom: 10px; }
        p { text-align: center; color: #666; margin-bottom: 30px; }

        .input-section { background: #f8f9fa; padding: 20px; border-radius: 10px; margin-bottom: 20px; }

        .input-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; color: #555; }
        input { width: 100%; padding: 12px; border: 1px solid #ddd; border-radius: 5px; box-sizing: border-box; font-size: 16px; }
        button { background: #28a745; color: white; border: none; padding: 12px 20px; border-radius: 5px; cursor: pointer; font-size: 16px; width: 100%; margin-top: 10px; }
        button:hover { background: #218838; }

        .withdraw-section button { background: #ffc107; color: #212529; }
        .withdraw-section button:hover { background: #e0a800; }

        .summary { background: #d4edda; padding: 20px; border-radius: 10px; text-align: center; }
        .summary p { font-size: 18px; margin: 10px 0; }
        .summary span { font-weight: bold; font-size: 20px; }
        .saved { color: #155724; }
        .spent { color: #856404; }

        .history-section h3 { margin-top: 0; }
        .history-section ul { list-style: none; padding: 0; max-height: 150px; overflow-y: auto; margin-bottom: 10px; }
        .history-section li { background: #e9ecef; padding: 10px; margin-bottom: 10px; border-radius: 5px; display: flex; justify-content: space-between; align-items: center; }
        .delete-btn { background: #dc3545; padding: 5px 10px; font-size: 14px; }
        .delete-btn:hover { background: #c82333; }
        .savings-li { border-left: 4px solid #28a745; }
        .withdrawals-li { border-left: 4px solid #ffc107; }

        .clear-btn { background: #6c757d; margin-top: 20px; }
        .clear-btn:hover { background: #545b62; }
    </style>
</head>
<body>
    <div class="container">
        <h1>💰 Money Saver</h1>
        <p>Save, withdraw, and track your pocket money/salary</p>

        <div class="input-section">
            <h2>Set Total Allowance/Salary</h2>
            <div class="input-group">
                <label for="total-income">Pocket Money / Salary (₹):</label>
                <input type="number" id="total-income" placeholder="e.g. 5000" min="0">
                <button onclick="setIncome()">Set Income</button>
            </div>
        </div>

        <div class="input-section">
            <h2>💾 Save Money</h2>
            <div class="input-group">
                <label for="save-amount">Amount (₹):</label>
                <input type="number" id="save-amount" placeholder="e.g. 1000" min="0">
                <label for="save-note">Note:</label>
                <input type="text" id="save-note" placeholder="e.g. For books">
                <button onclick="addSaving()">Save Now</button>
            </div>
        </div>

        <div class="input-section withdraw-section">
            <h2>💸 Withdraw Money</h2>
            <div class="input-group">
                <label for="withdraw-amount">Amount (₹):</label>
                <input type="number" id="withdraw-amount" placeholder="e.g. 200" min="0">
                <label for="withdraw-note">Note:</label>
                <input type="text" id="withdraw-note" placeholder="e.g. Snacks">
                <button onclick="addWithdrawal()">Withdraw Now</button>
            </div>
        </div>

        <div class="summary">
            <h2>Summary</h2>
            <p>Total Income: ₹<span id="total-income-display">0</span></p>
            <p>Total Saved: ₹<span id="total-saved" class="saved">0</span></p>
            <p>Total Spent: ₹<span id="total-spent" class="spent">0</span></p>
            <p>Remaining: ₹<span id="remaining">0</span></p>
        </div>

        <div class="history-section">
            <h3>💾 Savings History</h3>
            <ul id="savings-list"></ul>
            <h3>💸 Withdrawals History</h3>
            <ul id="withdrawals-list"></ul>
            <button onclick="clearAll()" class="clear-btn">Clear All Data</button>
        </div>
    </div>

    <script>
        let totalIncome = localStorage.getItem('totalIncome') ? parseFloat(localStorage.getItem('totalIncome')) : 0;
        let savings = JSON.parse(localStorage.getItem('savings')) || [];
        let withdrawals = JSON.parse(localStorage.getItem('withdrawals')) || [];

        const totalIncomeDisplay = document.getElementById('total-income-display');
        const totalSavedEl = document.getElementById('total-saved');
        const totalSpentEl = document.getElementById('total-spent');
        const remainingEl = document.getElementById('remaining');
        const savingsList = document.getElementById('savings-list');
        const withdrawalsList = document.getElementById('withdrawals-list');

        function setIncome() {
            const income = parseFloat(document.getElementById('total-income').value);
            if (isNaN(income) || income <= 0) return alert('Enter valid income');
            totalIncome = income;
            localStorage.setItem('totalIncome', totalIncome);
            updateDisplay();
            document.getElementById('total-income').value = '';
        }

        function addSaving() {
            const amount = parseFloat(document.getElementById('save-amount').value);
            const note = document.getElementById('save-note').value.trim();
            if (isNaN(amount) || amount <= 0) return alert('Enter valid save amount');
            const saving = { amount, note, date: new Date().toLocaleDateString() };
            savings.push(saving);
            localStorage.setItem('savings', JSON.stringify(savings));
            document.getElementById('save-amount').value = '';
            document.getElementById('save-note').value = '';
            renderSavings();
            updateDisplay();
        }

        function addWithdrawal() {
            const amount = parseFloat(document.getElementById('withdraw-amount').value);
            const note = document.getElementById('withdraw-note').value.trim();
            if (isNaN(amount) || amount <= 0) return alert('Enter valid withdraw amount');
            const withdrawal = { amount, note, date: new Date().toLocaleDateString() };
            withdrawals.push(withdrawal);
            localStorage.setItem('withdrawals', JSON.stringify(withdrawals));
            document.getElementById('withdraw-amount').value = '';
            document.getElementById('withdraw-note').value = '';
            renderWithdrawals();
            updateDisplay();
        }

        function renderSavings() {
            savingsList.innerHTML = '';
            savings.forEach((saving, index) => {
                const li = document.createElement('li');
                li.className = 'savings-li';
                li.innerHTML = `
                    <span>${saving.note ? saving.note + ' - ' : ''}₹${saving.amount} on ${saving.date}</span>
                    <button class="delete-btn" onclick="deleteSaving(${index})">Delete</button>
                `;
                savingsList.appendChild(li);
            });
        }

        function renderWithdrawals() {
            withdrawalsList.innerHTML = '';
            withdrawals.forEach((withdrawal, index) => {
                const li = document.createElement('li');
                li.className = 'withdrawals-li';
                li.innerHTML = `
                    <span>${withdrawal.note ? withdrawal.note + ' - ' : ''}₹${withdrawal.amount} on ${withdrawal.date}</span>
                    <button class="delete-btn" onclick="deleteWithdrawal(${index})">Delete</button>
                `;
                withdrawalsList.appendChild(li);
            });
        }

        function deleteSaving(index) {
            savings.splice(index, 1);
            localStorage.setItem('savings', JSON.stringify(savings));
            renderSavings();
            updateDisplay();
        }

        function deleteWithdrawal(index) {
            withdrawals.splice(index, 1);
            localStorage.setItem('withdrawals', JSON.stringify(withdrawals));
            renderWithdrawals();
            updateDisplay();
        }

        function updateDisplay() {
            const totalSaved = savings.reduce((sum, s) => sum + s.amount, 0);
            const totalSpent = withdrawals.reduce((sum, w) => sum + w.amount, 0);
            const netRemaining = Math.max(0, totalIncome - totalSpent);
            totalIncomeDisplay.textContent = totalIncome.toLocaleString();
            totalSavedEl.textContent = totalSaved.toLocaleString();
            totalSpentEl.textContent = totalSpent.toLocaleString();
            remainingEl.textContent = netRemaining.toLocaleString();
        }

        function clearAll() {
            if (confirm('Clear all savings and withdrawals?')) {
                savings = [];
                withdrawals = [];
                localStorage.setItem('savings', JSON.stringify(savings));
                localStorage.setItem('withdrawals', JSON.stringify(withdrawals));
                renderSavings();
                renderWithdrawals();
                updateDisplay();
            }
        }

        // Load on start
        renderSavings();
        renderWithdrawals();
        updateDisplay();
    </script>
</body>
</html>
