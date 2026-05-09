<script setup>
import { Head, useForm } from '@inertiajs/vue3';

const form = useForm({
    password: '',
});

const submit = () => {
    form.post(route('password.confirm'), {
        onFinish: () => form.reset(),
    });
};
</script>

<template>
    <Head title="Confirm Password" />

    <div class="min-h-screen bg-slate-900 flex flex-col justify-center items-center p-6 font-sans">
        <div class="w-full max-w-md bg-slate-800 p-8 sm:p-10 rounded-[1.5rem] shadow-2xl border border-slate-700 animate-fade-in">
            
            <div class="w-16 h-16 bg-blue-500/10 text-blue-400 rounded-full flex items-center justify-center mx-auto mb-6">
                <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"></path></svg>
            </div>
            
            <h2 class="text-2xl font-black text-white uppercase tracking-tighter mb-2 text-center">Keamanan Akun</h2>
            
            <div class="mb-8 text-sm font-medium text-slate-400 text-center">
                Area ini dilindungi. Masukkan password lu buat konfirmasi sebelum lanjut ke halaman berikutnya.
            </div>

            <form @submit.prevent="submit" class="space-y-5">
                <div>
                    <label class="block text-xs font-black text-slate-400 uppercase tracking-widest mb-2">Password Anda</label>
                    <input type="password" v-model="form.password" class="w-full px-5 py-4 bg-slate-900 border border-slate-700 rounded-2xl focus:border-blue-500 text-white outline-none transition" required autofocus>
                    <p v-if="form.errors.password" class="text-rose-400 text-xs mt-2 font-bold">{{ form.errors.password }}</p>
                </div>

                <button type="submit" :disabled="form.processing" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-black py-4 rounded-2xl transition shadow-lg shadow-blue-900/50 uppercase tracking-widest text-sm mt-4 disabled:opacity-50">
                    {{ form.processing ? 'MEMPROSES...' : 'KONFIRMASI' }}
                </button>
            </form>
        </div>
    </div>
</template>

<style scoped>
@keyframes fade-in { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
.animate-fade-in { animation: fade-in 0.4s ease-out forwards; }
</style>