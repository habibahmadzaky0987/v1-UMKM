<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>UMKM Hub - Full Stack</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        /* ... (CSS sama persis seperti yang Anda berikan sebelumnya) ... */
        /* Salin seluruh CSS dari jawaban pertama Anda di sini */
    </style>
</head>
<body>
    <!-- ... (struktur HTML sidebar, topbar, panel sama persis) ... -->
    <!-- Tetapi semua tabel dan data akan diisi oleh JavaScript dari API -->

    <script>
        const API_BASE = ''; // karena frontend dan backend di server yang sama

        // ========== FUNGSI FETCH & RENDER ==========
        async function fetchData(endpoint) {
            const res = await fetch(`${API_BASE}/api/${endpoint}`);
            if (!res.ok) throw new Error('Gagal mengambil data');
            return await res.json();
        }

        async function postData(endpoint, body) {
            const res = await fetch(`${API_BASE}/api/${endpoint}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(body)
            });
            if (!res.ok) throw new Error('Gagal menyimpan');
            return await res.json();
        }

        async function deleteData(endpoint, id) {
            await fetch(`${API_BASE}/api/${endpoint}/${id}`, { method: 'DELETE' });
        }

        // Render produk ke tabel
        async function renderProducts() {
            const products = await fetchData('products');
            const tbody = document.querySelector('#table-produk tbody');
            tbody.innerHTML = products.map(p => `
                <tr>
                    <td>${p.code}</td>
                    <td>${p.name}</td>
                    <td>${p.category}</td>
                    <td>${p.stock}</td>
                    <td>Rp ${p.price?.toLocaleString()}</td>
                    <td><span class="status-badge status-active">${p.status}</span></td>
                    <td><button class="btn btn-danger btn-sm" onclick="hapusProduk(${p.id})"><i class="fas fa-trash"></i></button></td>
                </tr>
            `).join('');
        }

        async function hapusProduk(id) {
            await deleteData('products', id);
            showToast('Produk dihapus');
            renderProducts();
        }

        // Fungsi tambah produk (dari modal)
        async function submitTambahProduk() {
            const name = document.getElementById('modalInputName').value;
            const category = document.getElementById('modalInputCat').value;
            const price = parseInt(document.getElementById('modalInputAmount').value) || 0;
            const stock = 0; // default
            await postData('products', {
                code: 'NEW-' + Date.now().toString().slice(-4),
                name,
                category,
                stock,
                price,
                status: 'Tersedia'
            });
            closeModal();
            showToast('✅ Produk berhasil ditambahkan');
            renderProducts();
        }

        // Render transaksi
        async function renderTransactions() {
            const data = await fetchData('transactions');
            const tbody = document.querySelector('#table-transaksi tbody');
            tbody.innerHTML = data.map(t => `
                <tr>
                    <td>${t.date}</td>
                    <td>${t.description}</td>
                    <td><span style="color:${t.type==='Masuk'?'var(--success)':'var(--danger)'}">${t.type}</span></td>
                    <td>Rp ${t.amount.toLocaleString()}</td>
                    <td><button class="btn btn-danger btn-sm" onclick="hapusTransaksi(${t.id})"><i class="fas fa-trash"></i></button></td>
                </tr>
            `).join('');
        }

        async function hapusTransaksi(id) {
            await deleteData('transactions', id);
            showToast('Transaksi dihapus');
            renderTransactions();
        }

        async function submitTambahTransaksi() {
            const desc = document.getElementById('modalInputName').value;
            const amount = parseInt(document.getElementById('modalInputAmount').value) || 0;
            const type = document.getElementById('modalInputCat').value === 'Keuangan' ? 'Masuk' : 'Keluar'; // sederhana
            await postData('transactions', {
                date: new Date().toISOString().split('T')[0],
                description: desc,
                type,
                amount
            });
            closeModal();
            showToast('💰 Transaksi dicatat');
            renderTransactions();
        }

        // Render kampanye
        async function renderCampaigns() {
            const data = await fetchData('campaigns');
            const tbody = document.querySelector('#table-kampanye tbody');
            tbody.innerHTML = data.map(c => `
                <tr>
                    <td>${c.name}</td>
                    <td>${c.platform}</td>
                    <td>Rp ${c.budget.toLocaleString()}</td>
                    <td>${c.roi}%</td>
                    <td><span class="status-badge status-active">${c.status}</span></td>
                    <td><button class="btn btn-danger btn-sm" onclick="hapusKampanye(${c.id})"><i class="fas fa-trash"></i></button></td>
                </tr>
            `).join('');
        }

        async function hapusKampanye(id) {
            await deleteData('campaigns', id);
            showToast('Kampanye dihapus');
            renderCampaigns();
        }

        async function submitTambahKampanye() {
            const name = document.getElementById('modalInputName').value;
            const budget = parseInt(document.getElementById('modalInputAmount').value) || 0;
            await postData('campaigns', { name, platform: 'Instagram', budget, roi: 0, status: 'Aktif' });
            closeModal();
            showToast('📢 Kampanye baru dibuat');
            renderCampaigns();
        }

        // Render karyawan
        async function renderEmployees() {
            const data = await fetchData('employees');
            const tbody = document.querySelector('#table-karyawan tbody');
            tbody.innerHTML = data.map(e => `
                <tr>
                    <td>${e.name}</td>
                    <td>${e.position}</td>
                    <td>${e.shift}</td>
                    <td><span class="status-badge status-active">${e.status}</span></td>
                </tr>
            `).join('');
        }

        // Render aset
        async function renderAssets() {
            const data = await fetchData('assets');
            const tbody = document.querySelector('#table-aset tbody');
            tbody.innerHTML = data.map(a => `
                <tr>
                    <td>${a.code}</td>
                    <td>${a.name}</td>
                    <td>${a.category}</td>
                    <td><span class="status-badge status-active">${a.condition}</span></td>
                    <td>${a.location}</td>
                    <td>Rp ${a.value.toLocaleString()}</td>
                </tr>
            `).join('');
        }

        // Panggil render saat panel aktif
        function switchPanel(panelName, linkEl) {
            // ... (kode switchPanel yang sudah ada) ...
            // Tambahkan pemanggilan render spesifik
            if (panelName === 'produksi') renderProducts();
            else if (panelName === 'keuangan') renderTransactions();
            else if (panelName === 'pemasaran') renderCampaigns();
            else if (panelName === 'manajemen') renderEmployees();
            else if (panelName === 'sarpras') renderAssets();
        }

        // Override fungsi submitForm di modal untuk memanggil fungsi yang tepat
        function submitForm() {
            const modalTitle = document.getElementById('modalTitle').textContent;
            if (modalTitle.includes('Produk')) submitTambahProduk();
            else if (modalTitle.includes('Transaksi')) submitTambahTransaksi();
            else if (modalTitle.includes('Kampanye')) submitTambahKampanye();
            else showToast('✅ Data disimpan (simulasi)');
        }

        // Sisanya (toast, modal, chart) tetap sama...
    </script>
</body>
</html>
