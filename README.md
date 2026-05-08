<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hệ Thống POS Thông Minh - QR & NFC</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; }
        .scanner-container { position: relative; overflow: hidden; border-radius: 1rem; border: 2px solid #e5e7eb; }
        #reader { width: 100% !important; border: none !important; }
        #reader__dashboard { padding: 10px !important; }
        #reader__status_span { display: none; }
        .nfc-pulse { animation: pulse 2s infinite; }
        @keyframes pulse {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.05); opacity: 0.8; }
            100% { transform: scale(1); opacity: 1; }
        }
    </style>
</head>
<body class="p-4 md:p-8">
    <div class="max-w-4xl mx-auto">
        <header class="mb-8 text-center">
            <h1 class="text-3xl font-bold text-gray-800">Cửa Hàng Tiện Lợi 24/7</h1>
            <p class="text-gray-500 text-sm mt-2">Quét mã QR để thêm món - Chạm NFC để thanh toán</p>
        </header>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
            <div class="space-y-6">
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
                    <h2 class="text-lg font-semibold mb-4 flex items-center">
                        <svg class="w-5 h-5 mr-2 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"></path></svg>
                        Quét Mã QR Sản Phẩm
                    </h2>
                    <div class="scanner-container bg-black aspect-square flex items-center justify-center">
                        <div id="reader"></div>
                    </div>
                    <div class="mt-4 flex space-x-2">
                        <button onclick="startScanner()" id="start-btn" class="flex-1 bg-blue-600 text-white py-2 rounded-lg font-medium hover:bg-blue-700 transition">Mở Camera</button>
                        <button onclick="stopScanner()" id="stop-btn" class="bg-gray-200 text-gray-400 px-4 py-2 rounded-lg font-medium cursor-not-allowed transition" disabled>Tắt</button>
                    </div>
                </div>

                <div id="nfc-area" class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100 hidden">
                    <h2 class="text-lg font-semibold mb-4 flex items-center">
                        <svg class="w-5 h-5 mr-2 text-emerald-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 3v2m6-2v2M9 19v2m6-2v2M5 9H3m2 6H3m18-6h-2m2 6h-2M7 19h10a2 2 0 002-2V7a2 2 0 00-2-2H7a2 2 0 00-2 2v10a2 2 0 002 2zM9 9h6v6H9V9z"></path></svg>
                        Thanh Toán NFC
                    </h2>
                    <div id="nfc-status" class="bg-emerald-50 border border-emerald-100 p-8 rounded-xl text-center">
                        <div class="nfc-pulse inline-block p-4 bg-emerald-500 rounded-full text-white mb-4">
                            <svg class="w-8 h-8" fill="currentColor" viewBox="0 0 20 20"><path d="M10 2a8 8 0 100 16 8 8 0 000-16zM5 10a5 5 0 0110 0v1a1 1 0 11-2 0v-1a3 3 0 00-6 0v1a1 1 0 11-2 0v-1z"></path></svg>
                        </div>
                        <p class="text-emerald-800 font-medium">Đang chờ chạm thẻ...</p>
                        <p class="text-xs text-emerald-600 mt-2">Đưa thẻ NFC hoặc điện thoại khác vào mặt lưng thiết bị</p>
                    </div>
                </div>
            </div>

            <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100 h-fit sticky top-4">
                <h2 class="text-lg font-semibold mb-4">Giỏ Hàng</h2>
                <div id="cart-items" class="space-y-3 mb-6 max-h-80 overflow-y-auto">
                    <p id="empty-cart" class="text-gray-400 italic text-center py-4">Chưa có sản phẩm nào</p>
                </div>
                
                <div class="border-t pt-4">
                    <div class="flex justify-between items-center mb-2">
                        <span class="text-gray-600">Số lượng:</span>
                        <span id="total-qty" class="font-bold">0</span>
                    </div>
                    <div class="flex justify-between items-center text-xl font-bold">
                        <span>Tổng tiền:</span>
                        <span class="text-blue-600"><span id="total-price">0</span> VNĐ</span>
                    </div>
                </div>

                <button id="checkout-btn" disabled onclick="prepareNFC()" class="w-full mt-6 bg-gray-300 text-gray-500 py-3 rounded-xl font-bold text-lg cursor-not-allowed transition-all">
                    THANH TOÁN
                </button>
                <button onclick="clearCart()" class="w-full mt-4 text-gray-400 text-sm hover:text-red-500 transition">Xóa tất cả đơn hàng</button>
            </div>
        </div>
    </div>

    <div id="modal" class="fixed inset-0 bg-black/50 hidden flex items-center justify-center p-4 z-50">
        <div class="bg-white rounded-2xl p-8 max-w-sm w-full text-center shadow-2xl">
            <div id="modal-icon" class="mb-4"></div>
            <h3 id="modal-title" class="text-xl font-bold mb-2"></h3>
            <p id="modal-msg" class="text-gray-600 mb-6"></p>
            <button onclick="closeModal()" class="w-full bg-blue-600 text-white py-3 rounded-lg font-semibold hover:bg-blue-700 transition">Đóng</button>
        </div>
    </div>

    <script>
        let cart = [];
        let html5QrCode;
        let isScanning = false;

        // --- HÀM XỬ LÝ GIỎ HÀNG ---
        function updateCartUI() {
            const container = document.getElementById('cart-items');
            const totalQty = document.getElementById('total-qty');
            const totalPrice = document.getElementById('total-price');
            const checkoutBtn = document.getElementById('checkout-btn');

            if (cart.length === 0) {
                container.innerHTML = '<p id="empty-cart" class="text-gray-400 italic text-center py-4">Chưa có sản phẩm nào</p>';
                checkoutBtn.disabled = true;
                checkoutBtn.className = "w-full mt-6 bg-gray-300 text-gray-500 py-3 rounded-xl font-bold text-lg cursor-not-allowed transition-all";
            } else {
                container.innerHTML = cart.map((item, index) => `
                    <div class="flex justify-between items-center bg-gray-50 p-3 rounded-lg border border-gray-100">
                        <div>
                            <p class="font-bold text-gray-800">Sản phẩm #${item.id}</p>
                            <p class="text-xs text-gray-400">${new Date(item.time).toLocaleTimeString()}</p>
                        </div>
                        <span class="font-bold text-blue-600">${item.price.toLocaleString()}đ</span>
                    </div>
                `).reverse().join('');
                checkoutBtn.disabled = false;
                checkoutBtn.className = "w-full mt-6 bg-blue-600 text-white py-3 rounded-xl font-bold text-lg hover:bg-blue-700 shadow-lg shadow-blue-200 transition-all active:scale-95";
            }

            const total = cart.reduce((sum, item) => sum + item.price, 0);
            totalQty.innerText = cart.length;
            totalPrice.innerText = total.toLocaleString();
        }

        function clearCart() {
            if(cart.length > 0 && confirm("Bạn muốn xóa toàn bộ giỏ hàng?")) {
                cart = [];
                updateCartUI();
            }
        }

        // --- HÀM QUÉT QR ---
        async function startScanner() {
            if (isScanning) return;
            
            const startBtn = document.getElementById('start-btn');
            const stopBtn = document.getElementById('stop-btn');
            
            startBtn.innerText = "Đang khởi động...";
            startBtn.disabled = true;

            html5QrCode = new Html5Qrcode("reader");
            const config = { fps: 10, qrbox: { width: 250, height: 250 } };

            try {
                await html5QrCode.start(
                    { facingMode: "environment" }, 
                    config, 
                    (decodedText) => {
                        // Giả định nội dung QR là giá tiền (ví dụ: "50000")
                        const price = parseInt(decodedText.replace(/\D/g,'')); // Lấy số từ chuỗi
                        if (!isNaN(price) && price > 0) {
                            const newItem = {
                                id: Math.floor(1000 + Math.random() * 9000),
                                price: price,
                                time: Date.now()
                            };
                            cart.push(newItem);
                            updateCartUI();
                            
                            // Phản hồi rung (nếu có)
                            if (navigator.vibrate) navigator.vibrate(100);
                            
                            // Hiệu ứng Flash xanh
                            const readerEl = document.getElementById('reader');
                            readerEl.style.outline = "8px solid #10b981";
                            setTimeout(() => readerEl.style.outline = "none", 300);
                        }
                    }
                );
                isScanning = true;
                startBtn.innerText = "Đang Quét...";
                startBtn.classList.replace('bg-blue-600', 'bg-gray-400');
                stopBtn.disabled = false;
                stopBtn.classList.replace('text-gray-400', 'text-gray-700');
                stopBtn.classList.add('hover:bg-gray-300');
            } catch (err) {
                showModal("Lỗi Camera", "Không thể truy cập camera. Vui lòng cấp quyền thiết bị.", "❌");
                startBtn.innerText = "Mở Camera";
                startBtn.disabled = false;
            }
        }

        async function stopScanner() {
            if (html5QrCode && isScanning) {
                await html5QrCode.stop();
                isScanning = false;
                const startBtn = document.getElementById('start-btn');
                const stopBtn = document.getElementById('stop-btn');
                
                startBtn.innerText = "Mở Camera";
                startBtn.disabled = false;
                startBtn.classList.replace('bg-gray-400', 'bg-blue-600');
                
                stopBtn.disabled = true;
                stopBtn.classList.replace('text-gray-700', 'text-gray-400');
                stopBtn.classList.remove('hover:bg-gray-300');
            }
        }

        // --- HÀM XỬ LÝ NFC ---
        async function prepareNFC() {
            if (!('NDEFReader' in window)) {
                showModal("Thông báo", "Trình duyệt này chưa hỗ trợ Web NFC. Đang chuyển sang chế độ giả lập thanh toán...", "📱");
                setTimeout(() => processPayment("GIA-LAP-ID"), 2000);
                return;
            }

            try {
                document.getElementById('nfc-area').classList.remove('hidden');
                const ndef = new NDEFReader();
                await ndef.scan();
                
                ndef.onreading = event => {
                    processPayment(event.serialNumber);
                };

                ndef.onreadingerror = () => {
                    showModal("Lỗi NFC", "Không thể đọc thẻ. Hãy thử lại.", "⚠️");
                };

            } catch (error) {
                console.error(error);
                showModal("Lỗi", "Vui lòng bật tính năng NFC trên điện thoại.", "❌");
            }
        }

        function processPayment(cardId) {
            const total = cart.reduce((sum, item) => sum + item.price, 0);
            showModal(
                "Thanh Toán Xong!", 
                `Tổng tiền: ${total.toLocaleString()} VNĐ\nPhương thức: Thẻ NFC (${cardId})`, 
                "✅"
            );
            cart = [];
            updateCartUI();
            document.getElementById('nfc-area').classList.add('hidden');
        }

        // --- UI HELPERS ---
        function showModal(title, msg, icon) {
            document.getElementById('modal-title').innerText = title;
            document.getElementById('modal-msg').innerText = msg;
            document.getElementById('modal-icon').innerHTML = `<span class="text-6xl block mb-2">${icon}</span>`;
            document.getElementById('modal').classList.remove('hidden');
        }

        function closeModal() {
            document.getElementById('modal').classList.add('hidden');
        }
    </script>
</body>
</html>
