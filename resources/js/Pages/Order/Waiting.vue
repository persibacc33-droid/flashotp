<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Head, router, usePage } from '@inertiajs/vue3';

const props = defineProps({
    order: Object,
    errors: Object
});

const page = usePage();

const localOrder = ref(props.order);
const otpError = ref(props.errors?.otp || '');
const otpSuccess = ref(page.props.flash?.success || '');
const hasEverReceivedOtp = ref(false);

// --- STATE MODAL KONFIRMASI ---
const showCancelModal = ref(false);
const showCompleteModal = ref(false);

// --- FITUR COPY ---
const copiedItem = ref('');
const copyToClipboard = (text, item) => {
    navigator.clipboard.writeText(text);
    copiedItem.value = item;
    setTimeout(() => {
        copiedItem.value = '';
    }, 2000);
};

// --- LOGIKA TIMER HITUNG MUNDUR ---
const timeLeftTotal = ref(1200); // 20 Menit
const timeLeftLock = ref(120);   // 2 Menit
let timerInterval = null;

const formatTime = (seconds) => {
    const m = Math.floor(seconds / 60).toString().padStart(2, '0');
    const s = (seconds % 60).toString().padStart(2, '0');
    return `${m}:${s}`;
};

const updateTimers = () => {
    if (localOrder.value.status !== 'waiting_otp') return;
    
    const orderId = localOrder.value.id;
    let startTime = localStorage.getItem(`order_start_${orderId}`);
    
    // Simpan waktu pertama kali halaman dibuka ke Local Storage biar anti-refresh
    if (!startTime) {
        startTime = Date.now();
        localStorage.setItem(`order_start_${orderId}`, startTime);
    }
    
    const elapsed = Math.floor((Date.now() - parseInt(startTime)) / 1000);
    timeLeftTotal.value = Math.max(0, 1200 - elapsed);
    timeLeftLock.value = Math.max(0, 120 - elapsed);
};

// Jaga-jaga kalau data aslinya berubah
watch(() => props.order, (newVal) => {
    localOrder.value = newVal;
    if (newVal.status !== 'waiting_otp' && timerInterval) {
        clearInterval(timerInterval);
    }
}, { deep: true });

let pollInterval = null;

const startSilentPolling = () => {
    // Pastikan interval bersih sebelum dinyalakan
    if (pollInterval) clearInterval(pollInterval);
    
    pollInterval = setInterval(async () => {
        let targetUrl = window.location.href;
        
        if (localOrder.value.status === 'waiting_otp' && !localOrder.value.otp_code) {
            targetUrl = route('order.check-otp', localOrder.value.id);
        } else if (localOrder.value.status !== 'pending_payment') {
            // MATIKAN RADAR JIKA OTP SUDAH MASUK
            clearInterval(pollInterval);
            pollInterval = null; 
            return;
        }

        try {
            const response = await fetch(targetUrl, {
                method: 'GET',
                headers: {
                    'X-Inertia': 'true',
                    'X-Inertia-Partial-Data': 'order,errors,flash',
                    'X-Inertia-Partial-Component': page.component,
                    'X-Inertia-Version': page.version || '', 
                    'X-Requested-With': 'XMLHttpRequest',
                    'Accept': 'application/json'
                }
            });

            // KALAU SERVER NGERESTART/UPDATE VERSI, PAKSA REFRESH OTOMATIS
            if (response.status === 409) {
                window.location.reload();
                return;
            }

            if (response.ok) {
                const data = await response.json();
                if (data.props) {
                    if (data.props.order) {
                        localOrder.value = data.props.order;
                        if (data.props.order.otp_code) {
                            hasEverReceivedOtp.value = true;
                            localStorage.setItem(`otp_received_${data.props.order.id}`, 'true');
                        }
                    }
                    if (data.props.errors && data.props.errors.otp) otpError.value = data.props.errors.otp;
                    if (data.props.flash && data.props.flash.success) otpSuccess.value = data.props.flash.success;
                }
            }
        } catch (error) {
            console.error("Polling error:", error);
        }
    }, 5000); 
};

onMounted(() => {
    if (['pending_payment', 'waiting_otp'].includes(localOrder.value.status) && !localOrder.value.otp_code) {
        startSilentPolling();
    }

    if (localOrder.value.status === 'waiting_otp') {
        updateTimers();
        timerInterval = setInterval(updateTimers, 1000);
    }
});

onUnmounted(() => {
    if (pollInterval) clearInterval(pollInterval);
    if (timerInterval) clearInterval(timerInterval);
});

// === EKSEKUSI TOMBOL DARI MODAL ===
const executeComplete = () => {
    showCompleteModal.value = false;
    router.post(route('order.complete', localOrder.value.id));
};

const executeCancel = () => {
    showCancelModal.value = false;
    router.post(route('order.cancel', localOrder.value.id));
};
</script>

<template>
    <Head title="Status Pesanan - Flash OTP" />

    <AuthenticatedLayout>
        <div class="relative space-y-6 animate-fade-in pb-10 min-h-screen text-slate-200">
            
            <div class="absolute top-0 left-1/2 -translate-x-1/2 w-[300px] md:w-[400px] h-[300px] md:h-[400px] bg-blue-600/10 rounded-full blur-[100px] pointer-events-none"></div>

            <div class="max-w-xl mx-auto space-y-6 relative z-10 mt-4 md:mt-8">
                
                <div v-if="localOrder.status === 'pending_payment'" class="bg-slate-800/60 backdrop-blur-md p-6 md:p-10 rounded-3xl md:rounded-[2rem] border border-slate-700/80 shadow-2xl text-center relative overflow-hidden animate-slide-up">
                    
                    <div class="bg-blue-500/10 text-blue-400 border border-blue-500/20 px-5 py-2.5 rounded-full text-[10px] md:text-xs font-black uppercase tracking-widest mb-6 inline-block animate-pulse">
                        Menunggu Pembayaran
                    </div>
                    
                    <h3 class="text-xs md:text-sm text-slate-400 mb-1 font-black uppercase tracking-widest">Total Pembayaran</h3>
                    <div class="font-black text-transparent bg-clip-text bg-gradient-to-r from-blue-400 to-indigo-400 mb-6 text-4xl md:text-5xl tracking-tighter drop-shadow-lg">
                        Rp {{ Number(localOrder.price).toLocaleString('id-ID') }}
                    </div>
                    
                    <p class="text-xs md:text-sm text-slate-300 font-medium mb-6 px-4">
                        Selesaikan pembayaran dengan memindai kode QRIS di bawah ini.
                    </p>  

                    <div class="p-4 md:p-5 bg-white rounded-3xl shadow-[0_0_40px_rgba(255,255,255,0.1)] border-4 border-slate-700 mb-6 inline-block transform transition hover:scale-105 duration-300">
                        <img :src="'https://api.qrserver.com/v1/create-qr-code/?size=300x300&margin=10&data=' + encodeURIComponent(localOrder.payment_qr)" 
                             alt="Scan QRIS" class="w-56 h-56 md:w-64 md:h-64 object-contain rounded-xl">
                    </div>
                    
                    <div class="bg-slate-900/60 backdrop-blur-sm p-4 rounded-2xl w-full border border-slate-700/50 mb-8 shadow-inner flex justify-between items-center">
                        <div class="text-left">
                            <p class="text-[9px] md:text-[10px] font-black text-slate-500 uppercase tracking-widest mb-0.5">Order ID</p>
                            <div class="flex items-center gap-2 mt-1">
                                <p class="text-xs md:text-sm font-bold text-white">{{ localOrder.payment_ref }}</p>
                                <button @click="copyToClipboard(localOrder.payment_ref, 'invoice')" class="p-1 rounded-md hover:bg-slate-700 text-slate-400 hover:text-white transition" title="Salin Invoice">
                                    <svg v-if="copiedItem !== 'invoice'" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path></svg>
                                    <svg v-else class="w-4 h-4 text-emerald-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path></svg>
                                </button>
                            </div>
                        </div>
                    </div>

                    <div class="space-y-4">
                        <button @click="router.reload({ only: ['order'] })" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-black py-4 rounded-2xl transition-all shadow-[0_0_20px_rgba(37,99,235,0.3)] hover:shadow-[0_0_30px_rgba(37,99,235,0.5)] tracking-widest text-xs md:text-sm uppercase active:scale-95">
                            Cek Status Pembayaran
                        </button>
                        <button @click="showCancelModal = true" class="w-full bg-slate-800 hover:bg-slate-700 text-rose-400 font-bold py-3.5 rounded-2xl transition border border-slate-700 hover:border-rose-500/30 tracking-widest text-xs uppercase">
                            Batalkan Pembayaran
                        </button>
                    </div>
                </div>

                <div v-else class="bg-slate-800/60 backdrop-blur-md p-6 md:p-10 rounded-3xl md:rounded-[2rem] border border-slate-700/80 shadow-2xl relative overflow-hidden animate-slide-up">

                    <!-- 🌟 INFO TIMER 20 MENIT UNTUK WAITING OTP -->
                    <div v-if="localOrder.status === 'waiting_otp' && !localOrder.otp_code" class="bg-blue-500/10 border border-blue-500/20 p-4 rounded-2xl text-center mb-6 shadow-inner animate-fade-in">
                        <p class="text-[10px] md:text-xs text-blue-300 font-bold uppercase tracking-widest mb-1">Masa Berlaku Nomor</p>
                        <p class="text-3xl md:text-4xl font-black text-blue-400 tracking-[0.1em] font-mono drop-shadow-md">{{ formatTime(timeLeftTotal) }}</p>
                        <p class="text-[9px] text-blue-400/70 mt-2 uppercase tracking-wider">Sistem akan membatalkan otomatis jika waktu habis</p>
                    </div>

                    <div class="flex justify-between items-center mb-6" :class="{'mt-10 md:mt-8': localOrder.status !== 'waiting_otp' || localOrder.otp_code}">
                        <h2 class="text-xl md:text-2xl font-black text-white uppercase tracking-tighter drop-shadow-md m-0">DETAIL ORDER</h2>
                        <div>
                            <span v-if="localOrder.status === 'waiting_otp'" class="bg-amber-500/10 text-amber-400 text-[9px] md:text-[10px] font-black px-3 py-1.5 rounded-lg uppercase tracking-widest border border-amber-500/30 shadow-[0_0_10px_rgba(245,158,11,0.2)] animate-pulse">Menunggu OTP</span>
                            <span v-else-if="localOrder.status === 'completed'" class="bg-emerald-500/10 text-emerald-400 text-[9px] md:text-[10px] font-black px-3 py-1.5 rounded-lg uppercase tracking-widest border border-emerald-500/30 shadow-[0_0_10px_rgba(16,185,129,0.2)]">SELESAI</span>
                            <span v-else class="bg-rose-500/10 text-rose-400 text-[9px] md:text-[10px] font-black px-3 py-1.5 rounded-lg uppercase tracking-widest border border-rose-500/30">DIBATALKAN</span>
                        </div>
                    </div>

                    <div class="space-y-4 mb-8 bg-slate-900/40 p-5 md:p-6 rounded-2xl border border-slate-700/50 shadow-inner">
                        <!-- 🌟 TOMBOL COPY INVOICE -->
                        <div class="flex justify-between border-b border-slate-700/50 pb-4 items-center">
                            <span class="text-slate-400 text-[10px] md:text-xs font-bold uppercase tracking-widest">No. Invoice</span>
                            <div class="flex items-center gap-2">
                                <span class="text-white font-black text-xs md:text-sm">{{ localOrder.payment_ref }}</span>
                                <button @click="copyToClipboard(localOrder.payment_ref, 'invoice')" class="p-1 rounded-md hover:bg-slate-700 text-slate-400 hover:text-white transition" title="Salin Invoice">
                                    <svg v-if="copiedItem !== 'invoice'" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path></svg>
                                    <svg v-else class="w-4 h-4 text-emerald-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path></svg>
                                </button>
                            </div>
                        </div>

                        <div class="flex justify-between border-b border-slate-700/50 pb-4 items-center">
                            <span class="text-slate-400 text-[10px] md:text-xs font-bold uppercase tracking-widest">Layanan</span>
                            <span class="text-white font-black text-xs md:text-sm uppercase bg-slate-800 px-3 py-1 rounded-md border border-slate-700">{{ localOrder.service_code }} ({{ localOrder.api_version }})</span>
                        </div>

                        <!-- 🌟 TOMBOL COPY NOMOR VIRTUAL -->
                        <div class="flex justify-between border-b border-slate-700/50 pb-4 items-center">
                            <span class="text-slate-400 text-[10px] md:text-xs font-bold uppercase tracking-widest">Nomor Virtual</span>
                            <div class="flex items-center gap-2">
                                <button @click="copyToClipboard('+' + localOrder.phone_number, 'phone')" class="flex items-center gap-2 text-blue-400 font-mono font-black text-sm md:text-base tracking-wider bg-blue-500/10 px-3 py-1.5 rounded-lg border border-blue-500/30 cursor-pointer hover:bg-blue-500/20 hover:scale-105 transition-all shadow-[0_0_10px_rgba(59,130,246,0.1)]">
                                    <span>+{{ localOrder.phone_number }}</span>
                                    <svg v-if="copiedItem !== 'phone'" class="w-4 h-4 opacity-80" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path></svg>
                                    <svg v-else class="w-4 h-4 text-emerald-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path></svg>
                                </button>
                            </div>
                        </div>

                        <div class="flex justify-between pt-2 items-center">
                            <span class="text-slate-400 text-[10px] md:text-xs font-bold uppercase tracking-widest">Harga Total</span>
                            <span class="text-emerald-400 font-black text-base md:text-lg">Rp {{ Number(localOrder.price).toLocaleString('id-ID') }}</span>
                        </div>
                    </div>

                    <div v-if="localOrder.otp_code" @click="copyToClipboard(localOrder.otp_code, 'otp')" class="bg-emerald-500/10 border border-emerald-500/30 rounded-3xl p-6 md:p-8 text-center mb-8 shadow-[0_0_40px_rgba(16,185,129,0.15)] cursor-pointer hover:scale-[1.02] transition transform relative overflow-hidden group">
                        <p class="text-emerald-300/80 text-[10px] md:text-xs font-black tracking-widest uppercase relative z-10">Kode OTP / SMS Diterima:</p>
                        <p class="text-emerald-300/80 text-[10px] md:text-xs font-black tracking-widest mb-3 relative">(Tap untuk menyalin)</p>
                        <p class="text-4xl md:text-5xl font-black text-emerald-400 tracking-[0.2em] font-mono drop-shadow-md relative z-10">{{ localOrder.otp_code }}</p>
                        <p v-if="copiedItem === 'otp'" class="text-white bg-emerald-500 px-3 py-1 rounded-full text-[10px] font-black mt-4 inline-block relative z-10 uppercase tracking-widest shadow-lg animate-fade-in">Berhasil Disalin!</p>
                    </div>
                    
                    <div v-else-if="localOrder.status === 'waiting_otp'" class="bg-slate-900/60 border border-slate-700/80 rounded-3xl p-6 md:p-8 text-center mb-8 shadow-inner flex flex-col items-center gap-4">
                        <div class="relative">
                            <div class="w-12 h-12 border-4 border-slate-700 rounded-full"></div>
                            <div class="w-12 h-12 border-4 border-blue-500 border-t-transparent rounded-full animate-spin absolute top-0 left-0"></div>
                        </div>
                        <div>
                            <p class="text-blue-400 text-xs md:text-sm font-black uppercase tracking-widest animate-pulse">Memantau SMS Masuk...</p>
                            <p class="text-[10px] md:text-xs text-slate-500 mt-2 font-bold">Jangan tutup halaman ini. OTP akan muncul otomatis.</p>
                            <p v-if="otpError && !localOrder.otp_code" class="text-[10px] font-black text-rose-400 mt-3 bg-rose-500/10 px-3 py-1 rounded-md border border-rose-500/20 inline-block">{{ otpError }}</p>
                        </div>
                    </div>

                    <div v-if="localOrder.status === 'waiting_otp'" class="flex gap-3 md:gap-4 mt-2">
                        <!-- 🌟 TOMBOL CANCEL DENGAN LOGIKA TIMER -->
                        <button v-if="!localOrder.otp_code" 
                                @click="showCancelModal = true" 
                                :disabled="timeLeftLock > 0"
                                :class="[
                                    'w-1/2 border font-black tracking-widest text-[10px] md:text-xs py-4 rounded-2xl transition uppercase',
                                    timeLeftLock > 0 
                                        ? 'bg-slate-800/50 text-slate-500 border-slate-700/50 cursor-not-allowed' 
                                        : 'bg-slate-800 hover:bg-slate-700 text-rose-400 border-slate-700 hover:border-rose-500/30'
                                ]">
                            {{ timeLeftLock > 0 ? 'Tunggu ' + formatTime(timeLeftLock) : 'Batal & Refund' }}
                        </button>
                        
                        <button @click="showCompleteModal = true" :class="!localOrder.otp_code ? 'w-1/2' : 'w-full'" class="bg-blue-600 hover:bg-blue-500 text-white font-black tracking-widest text-[10px] md:text-xs py-4 rounded-2xl shadow-[0_0_20px_rgba(37,99,235,0.3)] hover:shadow-[0_0_30px_rgba(37,99,235,0.5)] transition uppercase active:scale-95">
                            Pesanan Selesai
                        </button>
                    </div>
                    
                    <div v-if="localOrder.status !== 'waiting_otp'" class="mt-4">
                        <button @click="router.visit(route('dashboard'))" class="w-full bg-slate-800 hover:bg-slate-700 text-white font-black py-4 md:py-5 rounded-2xl transition border border-slate-700 shadow-lg uppercase tracking-widest text-[10px] md:text-xs active:scale-95">
                            Kembali ke Dashboard
                        </button>
                    </div>
                </div>

            </div>
        </div>

        <div v-if="showCancelModal" class="fixed inset-0 z-[100] flex items-center justify-center p-4 bg-slate-950/80 backdrop-blur-sm animate-fade-in">
            <div class="bg-slate-800 border border-slate-700 w-full max-w-sm rounded-[2rem] shadow-[0_0_50px_rgba(0,0,0,0.5)] overflow-hidden transform transition-all p-6 md:p-8 text-center relative">
                <div class="mx-auto w-16 h-16 bg-rose-500/10 border-2 border-rose-500/30 rounded-full flex items-center justify-center mb-5 shadow-[0_0_30px_rgba(225,29,72,0.2)]">
                    <svg class="w-8 h-8 text-rose-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"></path></svg>
                </div>
                <h3 class="text-xl font-black text-white uppercase tracking-tighter mb-2">Batalkan Pesanan?</h3>
                <p class="text-slate-400 text-xs md:text-sm font-medium leading-relaxed mb-8">
                    Saldo akan dikembalikan ke akun Anda secara otomatis 100%. Yakin ingin membatalkan?
                </p>
                <div class="flex flex-col gap-3">
                    <button @click="executeCancel" class="w-full bg-rose-600 hover:bg-rose-500 text-white font-black py-3.5 md:py-4 rounded-xl transition-all shadow-lg shadow-rose-900/50 uppercase tracking-widest text-[10px] md:text-xs active:scale-95">
                        Ya, Batalkan & Refund
                    </button>
                    <button @click="showCancelModal = false" class="w-full bg-slate-900 hover:bg-slate-700 text-slate-300 border border-slate-700 font-bold py-3.5 md:py-4 rounded-xl transition uppercase tracking-widest text-[10px] md:text-xs">
                        Kembali
                    </button>
                </div>
            </div>
        </div>

        <div v-if="showCompleteModal" class="fixed inset-0 z-[100] flex items-center justify-center p-4 bg-slate-950/80 backdrop-blur-sm animate-fade-in">
            <div class="bg-slate-800 border border-slate-700 w-full max-w-sm rounded-[2rem] shadow-[0_0_50px_rgba(0,0,0,0.5)] overflow-hidden transform transition-all p-6 md:p-8 text-center relative ">
                <div class="mx-auto w-16 h-16 bg-blue-500/10 border-2 border-blue-500/30 rounded-full flex items-center justify-center mb-5 shadow-[0_0_30px_rgba(59,130,246,0.2)]">
                    <svg class="w-8 h-8 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7"></path></svg>
                </div>
                <h3 class="text-xl font-black text-white uppercase tracking-tighter mb-2">Selesaikan Pesanan?</h3>
                <p class="text-slate-400 text-xs md:text-sm font-medium leading-relaxed mb-8">
                    Pastikan kode OTP sudah berhasil Anda gunakan di aplikasi tujuan sebelum menyelesaikan pesanan ini.
                </p>
                <div class="flex flex-col gap-3">
                    <button @click="executeComplete" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-black py-3.5 md:py-4 rounded-xl transition-all shadow-[0_0_20px_rgba(37,99,235,0.3)] hover:shadow-[0_0_30px_rgba(37,99,235,0.5)] uppercase tracking-widest text-[10px] md:text-xs active:scale-95">
                        Ya, Selesaikan Order
                    </button>
                    <button @click="showCompleteModal = false" class="w-full bg-slate-900 hover:bg-slate-700 text-slate-300 border border-slate-700 font-bold py-3.5 md:py-4 rounded-xl transition uppercase tracking-widest text-[10px] md:text-xs">
                        Kembali
                    </button>
                </div>
            </div>
        </div>

    </AuthenticatedLayout>
</template>

<style scoped>
@keyframes slide-up { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
.animate-slide-up { animation: slide-up 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards; }

@keyframes fade-in { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }
.animate-fade-in { animation: fade-in 0.2s ease-out forwards; }
</style>