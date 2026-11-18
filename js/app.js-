const INITIAL_STATE = {
    lang: 'bn',
    defaultWallet: 'Cash',
    carryForwardEnabled: false,
    carryForwardAmount: 500,
    wallets: [
        { name: 'Credit', balance: 800, icon: '💳', color: 'bg-indigo-500' },
        { name: 'Cash', balance: 1500, icon: '💵', color: 'bg-emerald-500' },
        { name: 'Bank', balance: 500, icon: '🏛️', color: 'bg-blue-500' }
    ],
    categories: {
        expense: [
            { name: 'Food & Dining', icon: '🍴', color: 'bg-amber-500' },
            { name: 'Groceries & Bazar', icon: '🛒', color: 'bg-emerald-500' },
            { name: 'Transport & Fuel', icon: '🚗', color: 'bg-blue-500' },
            { name: 'Bills & Utilities', icon: '⚡', color: 'bg-purple-500' }
        ],
        deposit: [
            { name: 'Monthly Salary', icon: '💼', color: 'bg-emerald-500' },
            { name: 'Freelance & Projects', icon: '💻', color: 'bg-blue-500' },
            { name: 'Gift / Family Help', icon: '🎁', color: 'bg-rose-500' }
        ]
    },
    transactions: [
        { id: 1, type: 'expense', amount: 200, wallet: 'Credit', category: 'Food & Dining', date: '2026-11-17', time: '8:34 PM', icon: '🍴', color: 'bg-amber-500' },
        { id: 2, type: 'expense', amount: 500, wallet: 'Cash', category: 'Food & Dining', date: '2026-11-17', time: '8:34 PM', icon: '🍴', color: 'bg-amber-500' },
        { id: 3, type: 'deposit', amount: 3500, wallet: 'Bank', category: 'Monthly Salary', date: '2026-11-17', time: '8:33 PM', icon: '💼', color: 'bg-emerald-500' }
    ],
    debts: [
        { id: 1, person: 'রহিম ভাই', amount: 1200, type: 'lend', wallet: 'Cash', date: '2026-11-15', settled: false },
        { id: 2, person: 'করিম সাহেব', amount: 2000, type: 'borrow', wallet: 'Bank', date: '2026-11-10', settled: false }
    ]
};

let state = JSON.parse(localStorage.getItem('takatrack_v5')) || INITIAL_STATE;
let currentMode = 'expense';
let selectedWallet = 'Cash';
let selectedCategory = state.categories.expense[0].name;
let transferFrom = 'Cash';
let transferTo = 'Credit';
let pickerContext = 'single';

function saveAndRender() {
    localStorage.setItem('takatrack_v5', JSON.stringify(state));
    renderDashboard();
}

function fmt(n) { return '৳' + Number(n || 0).toLocaleString(); }
function closeModal(id) { document.getElementById(id).classList.add('hidden'); }

function openTxModal(mode = 'expense', walletName = null) {
    const now = new Date();
    document.getElementById('txDate').value = now.toISOString().split('T')[0];
    document.getElementById('txAmount').value = '';
    document.getElementById('txFee').value = '0';
    document.getElementById('txNote').value = '';
    selectedWallet = walletName || state.defaultWallet;
    switchTxMode(mode);
    document.getElementById('txModal').classList.remove('hidden');
}

function switchTxMode(mode) {
    currentMode = mode;
    ['expense', 'deposit', 'transfer'].forEach(m => {
        const btn = document.getElementById('tabMode-' + m);
        if(btn) btn.className = "py-2 rounded-xl text-xs font-bold text-slate-400 cursor-pointer";
    });
    const std = document.getElementById('standardFields');
    const trf = document.getElementById('transferFields');
    const btn = document.getElementById('txSubmitBtn');

    if(mode === 'expense') {
        document.getElementById('tabMode-expense').className = "py-2 rounded-xl text-xs font-bold bg-[#1c202d] text-rose-400 cursor-pointer";
        document.getElementById('txModalTitle').innerText = "Record Expense";
        selectedCategory = state.categories.expense[0].name;
        std.classList.remove('hidden');
        trf.classList.add('hidden');
        btn.className = "bg-rose-500 text-white font-bold py-2.5 rounded-xl text-xs shadow-lg cursor-pointer";
        btn.innerText = "✓ Save Expense";
    } else if(mode === 'deposit') {
        document.getElementById('tabMode-deposit').className = "py-2 rounded-xl text-xs font-bold bg-[#1c202d] text-emerald-400 cursor-pointer";
        document.getElementById('txModalTitle').innerText = "Record Deposit";
        selectedCategory = state.categories.deposit[0].name;
        std.classList.remove('hidden');
        trf.classList.add('hidden');
        btn.className = "bg-emerald-500 text-slate-950 font-bold py-2.5 rounded-xl text-xs shadow-lg cursor-pointer";
        btn.innerText = "✓ Save Deposit";
    } else {
        document.getElementById('tabMode-transfer').className = "py-2 rounded-xl text-xs font-bold bg-[#1c202d] text-indigo-400 cursor-pointer";
        document.getElementById('txModalTitle').innerText = "Wallet Transfer";
        std.classList.add('hidden');
        trf.classList.remove('hidden');
        btn.className = "bg-indigo-600 text-white font-bold py-2.5 rounded-xl text-xs shadow-lg cursor-pointer";
        btn.innerText = "✓ Execute Transfer";
    }
    syncModalLabels();
    updateTransferPreview();
}

function syncModalLabels() {
    const wObj = state.wallets.find(w => w.name === selectedWallet) || state.wallets[0];
    const sEl = document.getElementById('selectedWalletText');
    if(sEl) sEl.innerText = wObj.name;
    const bEl = document.getElementById('lblSingleWalletBal');
    if(bEl) bEl.innerText = fmt(wObj.balance);
    const cEl = document.getElementById('selectedCatText');
    if(cEl) cEl.innerText = selectedCategory;

    const fObj = state.wallets.find(w => w.name === transferFrom) || state.wallets[0];
    const tObj = state.wallets.find(w => w.name === transferTo) || state.wallets[1] || state.wallets[0];
    const sfEl = document.getElementById('selectedFromText');
    if(sfEl) sfEl.innerText = fObj.name;
    const fbEl = document.getElementById('lblFromBal');
    if(fbEl) fbEl.innerText = fmt(fObj.balance);
    const stEl = document.getElementById('selectedToText');
    if(stEl) stEl.innerText = tObj.name;
    const tbEl = document.getElementById('lblToBal');
    if(tbEl) tbEl.innerText = fmt(tObj.balance);
}

function updateTransferPreview() {
    const amtField = document.getElementById('txAmount');
    if(!amtField) return;
    const amt = parseFloat(amtField.value) || 0;
    const tObj = state.wallets.find(w => w.name === transferTo) || state.wallets[0];
    const targetBox = document.getElementById('targetGetsBox');
    if(targetBox) targetBox.innerText = `${fmt(tObj.balance)} → ${fmt(tObj.balance + amt)}`;
}

function openDefaultWalletPicker() { openWalletPicker('default'); }
function openWalletPicker(ctx) {
    pickerContext = ctx;
    const list = state.wallets;
    const listEl = document.getElementById('walletPickerList');
    if(!listEl) return;
    listEl.innerHTML = list.map(w => `
        <div onclick="selectWalletFromPicker('${w.name}')" class="px-5 py-4 flex justify-between items-center cursor-pointer hover:bg-slate-800/50">
            <div class="flex items-center space-x-3.5">
                <div class="w-10 h-10 rounded-xl ${w.color} text-white flex items-center justify-center text-base">${w.icon}</div>
                <div><p class="font-bold text-sm text-white">${w.name}</p><p class="text-xs text-slate-400">${fmt(w.balance)}</p></div>
            </div>
        </div>
    `).join('');
    document.getElementById('walletPickerModal').classList.remove('hidden');
}

function selectWalletFromPicker(name) {
    if(pickerContext === 'single') selectedWallet = name;
    else if(pickerContext === 'default') state.defaultWallet = name;
    else if(pickerContext === 'from') transferFrom = name;
    else if(pickerContext === 'to') transferTo = name;
    closeModal('walletPickerModal');
    syncModalLabels();
    updateTransferPreview();
    saveAndRender();
}

function openCategoryPicker() {
    const list = currentMode === 'deposit' ? state.categories.deposit : state.categories.expense;
    const listEl = document.getElementById('categoryPickerList');
    if(!listEl) return;
    listEl.innerHTML = list.map(c => `
        <div onclick="selectCategoryFromPicker('${c.name}')" class="px-5 py-3.5 flex justify-between items-center cursor-pointer hover:bg-slate-800/50">
            <div class="flex items-center space-x-3.5">
                <div class="w-10 h-10 rounded-xl ${c.color} text-white flex items-center justify-center text-base">${c.icon}</div>
                <span class="font-bold text-sm text-white">${c.name}</span>
            </div>
        </div>
    `).join('');
    document.getElementById('categoryPickerModal').classList.remove('hidden');
}

function selectCategoryFromPicker(cName) {
    selectedCategory = cName;
    closeModal('categoryPickerModal');
    syncModalLabels();
}

function promptAddNewCategory() {
    const name = prompt("নতুন ক্যাটাগরির নাম লিখুন:");
    if(!name || !name.trim()) return;
    const clean = name.trim();
    const list = currentMode === 'deposit' ? state.categories.deposit : state.categories.expense;
    list.push({ name: clean, icon: '🏷️', color: 'bg-teal-500' });
    selectedCategory = clean;
    closeModal('categoryPickerModal');
    syncModalLabels();
    saveAndRender();
}

function submitTransaction(e) {
    e.preventDefault();
    const amount = parseFloat(document.getElementById('txAmount').value);
    const date = document.getElementById('txDate').value;
    const note = document.getElementById('txNote').value;
    if(isNaN(amount) || amount <= 0) return;

    if(currentMode === 'transfer') {
        if(transferFrom === transferTo) { alert('একই ওয়ালেটে ট্রান্সফার করা যাবে না!'); return; }
        const fee = parseFloat(document.getElementById('txFee').value) || 0;
        const wFrom = state.wallets.find(w => w.name === transferFrom);
        const wTo = state.wallets.find(w => w.name === transferTo);
        if(wFrom && wTo) {
            wFrom.balance -= (amount + fee);
            wTo.balance += amount;
            state.transactions.unshift({ id: Date.now(), type: 'transfer', amount, wallet: `${transferFrom} → ${transferTo}`, category: note || 'Fund Transfer', date, time: '12:00 PM', icon: '⇄', color: 'bg-indigo-500' });
        }
    } else {
        const wObj = state.wallets.find(w => w.name === selectedWallet);
        const catList = currentMode === 'deposit' ? state.categories.deposit : state.categories.expense;
        const catObj = catList.find(c => c.name === selectedCategory) || catList[0];
        if(wObj) {
            if(currentMode === 'deposit') wObj.balance += amount;
            else wObj.balance -= amount;
        }
        state.transactions.unshift({ id: Date.now(), type: currentMode, amount, wallet: selectedWallet, category: note || selectedCategory, date, time: '12:00 PM', icon: catObj.icon, color: catObj.color });
    }
    closeModal('txModal');
    saveAndRender();
}

function deleteTx(id) {
    if(!confirm('লেনদেনটি মুছে ফেলতে চান?')) return;
    const tx = state.transactions.find(t => t.id === id);
    if(tx && tx.type !== 'transfer') {
        const w = state.wallets.find(x => x.name === tx.wallet);
        if(w) {
            if(tx.type === 'deposit') w.balance -= tx.amount;
            else w.balance += tx.amount;
        }
    }
    state.transactions = state.transactions.filter(t => t.id !== id);
    saveAndRender();
}

function toggleSettingsModal() {
    const m = document.getElementById('settingsModal');
    if(m) m.classList.toggle('hidden');
    const cfToggle = document.getElementById('carryForwardToggle');
    if(cfToggle) cfToggle.checked = state.carryForwardEnabled;
    const cfAmt = document.getElementById('carryForwardAmountInput');
    if(cfAmt) cfAmt.value = state.carryForwardAmount;
}

function toggleCarryForwardSetting(el) {
    state.carryForwardEnabled = el.checked;
    saveAndRender();
}

function updateCarryForwardAmount(el) {
    state.carryForwardAmount = parseFloat(el.value) || 0;
    saveAndRender();
}

function exportJSONBackup() {
    const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(state, null, 2));
    const a = document.createElement('a');
    a.setAttribute("href", dataStr);
    a.setAttribute("download", "TakaTrack_Backup.json");
    document.body.appendChild(a);
    a.click();
    a.remove();
}

function renderDashboard() {
    const defW = state.wallets.find(w => w.name === state.defaultWallet) || state.wallets[0];
    const defIcon = document.getElementById('dashDefIcon');
    if(defIcon) defIcon.innerText = defW.icon;
    const defNameUp = document.getElementById('dashDefNameUpper');
    if(defNameUp) defNameUp.innerText = defW.name.toUpperCase();
    const defSub = document.getElementById('dashDefNameSub');
    if(defSub) defSub.innerText = defW.name;
    const defBal = document.getElementById('dashDefBalance');
    if(defBal) defBal.innerText = fmt(defW.balance);

    const walletsSum = state.wallets.reduce((s, w) => s + w.balance, 0);
    const carryExtra = state.carryForwardEnabled ? state.carryForwardAmount : 0;
    const totalBal = walletsSum + carryExtra;

    const totalIn = state.transactions.filter(t => t.type === 'deposit').reduce((s, t) => s + t.amount, 0);
    const totalOut = state.transactions.filter(t => t.type === 'expense').reduce((s, t) => s + t.amount, 0);

    const banner = document.getElementById('dashCarryForwardBanner');
    if(banner) {
        if(state.carryForwardEnabled) {
            banner.classList.remove('hidden');
            const cfAmt = document.getElementById('dashCarryForwardAmount');
            if(cfAmt) cfAmt.innerText = `+${fmt(state.carryForwardAmount)}`;
        } else {
            banner.classList.add('hidden');
        }
    }

    const tBal = document.getElementById('dashTotalBalance');
    if(tBal) tBal.innerText = fmt(totalBal);
    const accBadge = document.getElementById('dashAccBadge');
    if(accBadge) accBadge.innerText = `${state.wallets.length} accounts`;
    const wList = document.getElementById('dashWalletNamesList');
    if(wList) wList.innerText = `Sum of ${state.wallets.map(w => w.name).join(', ')}`;
    const tIn = document.getElementById('dashTotalIn');
    if(tIn) tIn.innerText = `+${fmt(totalIn)}`;
    const tOut = document.getElementById('dashTotalOut');
    if(tOut) tOut.innerText = `-${fmt(totalOut)}`;

    const wGrid = document.getElementById('dashWalletsGrid');
    if(wGrid) {
        wGrid.innerHTML = state.wallets.map(w => `
            <div class="bg-[#242938] border ${w.name === defW.name ? 'border-amber-500/60' : 'border-slate-700/60'} p-3.5 rounded-2xl space-y-2">
                <div class="flex justify-between items-center">
                    <span class="w-8 h-8 rounded-xl ${w.color} flex items-center justify-center text-sm text-white">${w.icon}</span>
                    ${w.name === defW.name ? '<span class="text-[9px] bg-amber-500/20 text-amber-300 px-1.5 py-0.5 rounded font-bold">★ Default</span>' : ''}
                </div>
                <div><p class="text-xs font-bold text-slate-200">${w.name}</p><p class="text-base font-extrabold text-white mt-0.5">${fmt(w.balance)}</p></div>
            </div>
        `).join('');
    }

    const recBadge = document.getElementById('dashRecentBadge');
    if(recBadge) recBadge.innerText = `${state.transactions.length} total`;
    const recCont = document.getElementById('dashRecentContainer');
    if(recCont) {
        recCont.innerHTML = `
            <div class="bg-[#242938]/70 border border-slate-700/50 rounded-2xl divide-y divide-slate-700/50">
                ${state.transactions.slice(0, 5).map(t => `
                    <div class="p-3.5 flex justify-between items-center">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl ${t.color} text-white flex items-center justify-center text-sm">${t.icon}</div>
                            <div><h4 class="font-bold text-xs text-white">${t.category}</h4><p class="text-[10px] text-slate-400">${t.wallet} • ${t.date}</p></div>
                        </div>
                        <div class="flex items-center space-x-3">
                            <span class="font-extrabold text-xs ${t.type === 'deposit' ? 'text-emerald-400' : t.type === 'expense' ? 'text-rose-400' : 'text-indigo-400'}">${t.type === 'deposit' ? '+' : t.type === 'expense' ? '-' : ''}${fmt(t.amount)}</span>
                            <button type="button" onclick="deleteTx(${t.id})" class="text-slate-500 hover:text-rose-400 text-xs">🗑️</button>
                        </div>
                    </div>
                `).join('')}
            </div>
        `;
    }
}

renderDashboard();
