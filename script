document.addEventListener('DOMContentLoaded', function() {
    // Elemen DOM
    const queueNumberInput = document.getElementById('queue-number');
    const operatorSelect = document.getElementById('operator-select');
    const callButton = document.getElementById('call-btn');
    const volumeSlider = document.getElementById('volume-slider');
    const volumeValue = document.getElementById('volume-value');
    const increaseBtn = document.getElementById('increase-btn');
    const decreaseBtn = document.getElementById('decrease-btn');
    const testVoiceBtn = document.getElementById('test-voice-btn');
    const resetBtn = document.getElementById('reset-btn');
    const currentQueueDisplay = document.getElementById('current-queue-display');
    const currentOperatorDisplay = document.getElementById('current-operator-display');
    const recentCallsList = document.getElementById('recent-calls-list');
    const operatorsGrid = document.getElementById('operators-grid');
    const timeDisplay = document.getElementById('time-display');
    const voiceStatusText = document.getElementById('voice-status-text');
    
    // Data aplikasi
    let recentCalls = JSON.parse(localStorage.getItem('spmbRecentCalls')) || [];
    let queueCounter = JSON.parse(localStorage.getItem('spmbQueueCounter')) || 1;
    let volume = parseInt(localStorage.getItem('spmbVolume')) || 70;
    let speechSynthesisSupported = 'speechSynthesis' in window;
    
    // Inisialisasi
    queueNumberInput.value = queueCounter;
    volumeSlider.value = volume;
    volumeValue.textContent = `${volume}%`;
    
    // Perbarui waktu
    function updateTime() {
        const now = new Date();
        const timeString = now.toLocaleTimeString('id-ID', {hour12: false});
        timeDisplay.textContent = timeString;
    }
    
    // Perbarui status suara
    function updateVoiceStatus() {
        if (!speechSynthesisSupported) {
            voiceStatusText.textContent = "Sistem Suara: Tidak Didukung";
            voiceStatusText.style.color = "#ff416c";
            callButton.disabled = true;
            testVoiceBtn.disabled = true;
        } else {
            voiceStatusText.textContent = "Sistem Suara: Aktif";
            voiceStatusText.style.color = "#1a2980";
        }
    }
    
    // Buar kartu operator
    function createOperatorCards() {
        const operatorNames = [
            "Operator 1 - Pendaftaran",
            "Operator 2 - Verifikasi Dokumen",
            "Operator 3 - Wawancara",
            "Operator 4 - Tes Akademik",
            "Operator 5 - Tes Psikologi",
            "Operator 6 - Pengumuman Hasil",
            "Operator 7 - Konsultasi",
            "Operator 8 - Administrasi"
        ];
        
        operatorsGrid.innerHTML = '';
        
        operatorNames.forEach((name, index) => {
            const operatorNumber = index + 1;
            const lastCall = recentCalls.find(call => call.operator === operatorNumber.toString());
            
            const card = document.createElement('div');
            card.className = 'operator-card';
            card.id = `operator-${operatorNumber}`;
            card.innerHTML = `
                <div class="operator-number">${operatorNumber}</div>
                <div class="operator-name">${name.split(' - ')[1]}</div>
                <div class="operator-status status-inactive">TIDAK AKTIF</div>
                <div class="last-call">${lastCall ? `Terakhir: ${lastCall.queueNumber}` : 'Belum ada panggilan'}</div>
            `;
            
            operatorsGrid.appendChild(card);
        });
    }
    
    // Perbarui tampilan operator yang aktif
    function updateActiveOperator(operatorNumber) {
        // Reset semua operator ke tidak aktif
        document.querySelectorAll('.operator-card').forEach(card => {
            const statusElement = card.querySelector('.operator-status');
            statusElement.textContent = 'TIDAK AKTIF';
            statusElement.className = 'operator-status status-inactive';
            card.classList.remove('active');
        });
        
        // Set operator yang aktif
        const activeOperatorCard = document.getElementById(`operator-${operatorNumber}`);
        if (activeOperatorCard) {
            const statusElement = activeOperatorCard.querySelector('.operator-status');
            statusElement.textContent = 'SEDANG MEMANGGIL';
            statusElement.className = 'operator-status status-active';
            activeOperatorCard.classList.add('active');
        }
    }
    
    // Fungsi untuk mengucapkan teks
    function speakText(text) {
        if (!speechSynthesisSupported) return;
        
        // Hentikan ucapan sebelumnya
        speechSynthesis.cancel();
        
        // Buat objek ucapan
        const utterance = new SpeechSynthesisUtterance(text);
        utterance.lang = 'id-ID';
        utterance.rate = 0.9; // Kecepatan ucapan
        utterance.pitch = 1; // Nada suara
        utterance.volume = volume / 100; // Volume (0-1)
        
        // Coba gunakan suara wanita jika tersedia
        const voices = speechSynthesis.getVoices();
        const femaleVoice = voices.find(voice => 
            voice.lang.includes('id') && voice.name.toLowerCase().includes('female')
        ) || voices.find(voice => voice.lang.includes('id')) || voices[0];
        
        if (femaleVoice) {
            utterance.voice = femaleVoice;
        }
        
        // Ucapkan teks
        speechSynthesis.speak(utterance);
    }
    
    // Format nomor antrian (tambah nol di depan)
    function formatQueueNumber(number) {
        return number.toString().padStart(3, '0');
    }
    
    // Panggil antrian
    function callQueue() {
        const queueNumber = parseInt(queueNumberInput.value);
        const operatorNumber = operatorSelect.value;
        const operatorText = operatorSelect.options[operatorSelect.selectedIndex].text;
        
        // Simpan panggilan
        const callData = {
            queueNumber: formatQueueNumber(queueNumber),
            operator: operatorNumber,
            operatorText: operatorText,
            timestamp: new Date().toLocaleTimeString('id-ID', {hour12: false})
        };
        
        // Tambahkan ke daftar panggilan terakhir
        recentCalls.unshift(callData);
        
        // Simpan hanya 5 panggilan terakhir
        if (recentCalls.length > 5) {
            recentCalls = recentCalls.slice(0, 5);
        }
        
        // Simpan ke localStorage
        localStorage.setItem('spmbRecentCalls', JSON.stringify(recentCalls));
        
        // Perbarui counter antrian
        queueCounter = queueNumber + 1;
        localStorage.setItem('spmbQueueCounter', queueCounter.toString());
        queueNumberInput.value = queueCounter;
        
        // Perbarui tampilan
        updateDisplay(callData);
        updateRecentCallsList();
        updateActiveOperator(operatorNumber);
        
        // Ucapkan panggilan
        const announcement = `Nomor antrian ${formatQueueNumber(queueNumber)}, silakan menuju ${operatorText.split(' - ')[1]}`;
        speakText(announcement);
        
        // Tampilkan notifikasi
        showNotification(`Antrian ${formatQueueNumber(queueNumber)} dipanggil ke ${operatorText.split(' - ')[1]}`);
    }
    
    // Perbarui tampilan utama
    function updateDisplay(callData) {
        currentQueueDisplay.innerHTML = `<div class="queue-num-display">${callData.queueNumber}</div>`;
        currentOperatorDisplay.textContent = callData.operatorText;
    }
    
    // Perbarui daftar panggilan terakhir
    function updateRecentCallsList() {
        if (recentCalls.length === 0) {
            recentCallsList.innerHTML = '<p class="empty-message">Belum ada panggilan antrian</p>';
            return;
        }
        
        recentCallsList.innerHTML = '';
        recentCalls.forEach(call => {
            const callItem = document.createElement('div');
            callItem.className = 'call-item';
            callItem.innerHTML = `
                <div class="queue-num">Antrian ${call.queueNumber}</div>
                <div class="operator-name">${call.operatorText}</div>
                <div class="call-time">${call.timestamp}</div>
            `;
            recentCallsList.appendChild(callItem);
        });
    }
    
    // Tampilkan notifikasi
    function showNotification(message) {
        // Buat elemen notifikasi
        const notification = document.createElement('div');
        notification.className = 'notification';
        notification.textContent = message;
        notification.style.cssText = `
            position: fixed;
            top: 20px;
            right: 20px;
            background: #1a2980;
            color: white;
            padding: 15px 25px;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            z-index: 1000;
            animation: slideIn 0.3s ease;
        `;
        
        document.body.appendChild(notification);
        
        // Hapus notifikasi setelah 3 detik
        setTimeout(() => {
            notification.style.animation = 'slideOut 0.3s ease';
            setTimeout(() => {
                document.body.removeChild(notification);
            }, 300);
        }, 3000);
    }
    
    // Reset antrian
    function resetQueue() {
        if (confirm('Apakah Anda yakin ingin mereset antrian? Semua data akan dikembalikan ke pengaturan awal.')) {
            queueCounter = 1;
            recentCalls = [];
            
            localStorage.setItem('spmbQueueCounter', queueCounter.toString());
            localStorage.setItem('spmbRecentCalls', JSON.stringify(recentCalls));
            
            queueNumberInput.value = queueCounter;
            currentQueueDisplay.innerHTML = '<div class="placeholder">-</div>';
            currentOperatorDisplay.innerHTML = '<div class="placeholder">Belum ada panggilan</div>';
            
            updateRecentCallsList();
            createOperatorCards();
            
            showNotification('Antrian berhasil direset');
        }
    }
    
    // Event Listeners
    callButton.addEventListener('click', callQueue);
    
    increaseBtn.addEventListener('click', () => {
        queueNumberInput.value = parseInt(queueNumberInput.value) + 1;
    });
    
    decreaseBtn.addEventListener('click', () => {
        const currentValue = parseInt(queueNumberInput.value);
        if (currentValue > 1) {
            queueNumberInput.value = currentValue - 1;
        }
    });
    
    volumeSlider.addEventListener('input', () => {
        volume = parseInt(volumeSlider.value);
        volumeValue.textContent = `${volume}%`;
        localStorage.setItem('spmbVolume', volume.toString());
    });
    
    testVoiceBtn.addEventListener('click', () => {
        const testText = "Ini adalah uji suara sistem antrian SPMB SMA Negeri 1 Magetan.";
        speakText(testText);
        showNotification('Uji suara dilakukan');
    });
    
    resetBtn.addEventListener('click', resetQueue);
    
    // Event listener untuk tombol Enter pada input nomor antrian
    queueNumberInput.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') {
            callQueue();
        }
    });
    
    // Event listener untuk tombol Enter pada select operator
    operatorSelect.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') {
            callQueue();
        }
    });
    
    // Inisialisasi aplikasi
    function initApp() {
        updateTime();
        updateVoiceStatus();
        createOperatorCards();
        updateRecentCallsList();
        
        // Perbarui waktu setiap detik
        setInterval(updateTime, 1000);
        
        // Tambahkan style untuk animasi notifikasi
        const style = document.createElement('style');
        style.textContent = `
            @keyframes slideIn {
                from { transform: translateX(100%); opacity: 0; }
                to { transform: translateX(0); opacity: 1; }
            }
            @keyframes slideOut {
                from { transform: translateX(0); opacity: 1; }
                to { transform: translateX(100%); opacity: 0; }
            }
        `;
        document.head.appendChild(style);
        
        // Jika ada panggilan terakhir, tampilkan
        if (recentCalls.length > 0) {
            updateDisplay(recentCalls[0]);
            updateActiveOperator(recentCalls[0].operator);
        }
        
        // Inisialisasi SpeechSynthesis voices
        if (speechSynthesisSupported) {
            // Chrome memerlukan event ini untuk memuat voices
            speechSynthesis.onvoiceschanged = function() {
                // Voices sudah dimuat
            };
        }
    }
    
    // Jalankan inisialisasi
    initApp();
});