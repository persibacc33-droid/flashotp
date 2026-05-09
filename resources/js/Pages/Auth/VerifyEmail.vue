<script setup>
import { computed } from 'vue';
import { Head, Link, useForm } from '@inertiajs/vue3';

const props = defineProps({
    status: String,
});

const form = useForm({});

const submit = () => {
    form.post(route('verification.send'));
};

const verificationLinkSent = computed(
    () => props.status === 'verification-link-sent',
);
</script>

<template>
    <Head title="Email Verification" />

    <div class="min-h-screen bg-slate-900 flex flex-col justify-center items-center p-6 font-sans">
        <div class="w-full max-w-md bg-slate-800 p-8 sm:p-10 rounded-[2.5rem] shadow-2xl border border-slate-700 animate-fade-in text-center">
            
            <div class="w-16 h-16 bg-blue-500/10 text-blue-400 rounded-full flex items-center justify-center mx-auto mb-6">
                <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"></path></svg>
            </div>

            <h2 class="text-2xl font-black text-white uppercase tracking-tighter mb-4">Cek Email Anda</h2>

            <div class="mb-8 text-sm font-medium text-slate-400 leading-relaxed">
                Terima kasih sudah mendaftar! Sebelum mulai transaksi, tolong verifikasi alamat email Anda dengan mengeklik link yang baru saja kami kirim. Kalau emailnya belum masuk, kita bisa kirim ulang kok.
            </div>

            <div v-if="verificationLinkSent" class="mb-6 p-4 bg-emerald-500/10 border border-emerald-500/20 text-emerald-400 rounded-xl text-sm font-bold">
                Link verifikasi baru telah dikirim ke email yang Anda gunakan saat mendaftar.
            </div>

            <form @submit.prevent="submit" class="space-y-6">
                <button type="submit" :disabled="form.processing" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-black py-4 rounded-2xl transition shadow-lg shadow-blue-900/50 uppercase tracking-widest text-sm disabled:opacity-50">
                    {{ form.processing ? 'MENGIRIM...' : 'KIRIM ULANG LINK' }}
                </button>

                <div class="pt-2">
                    <Link :href="route('logout')" method="post" as="button" class="text-sm font-bold text-slate-500 hover:text-rose-400 transition underline tracking-wider uppercase">
                        Keluar / Log Out
                    </Link>
                </div>
            </form>
            
        </div>
    </div>
</template>

<style scoped>
@keyframes fade-in { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
.animate-fade-in { animation: fade-in 0.4s ease-out forwards; }
</style>