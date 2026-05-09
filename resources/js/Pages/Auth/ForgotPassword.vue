<script setup>
import { Head, Link, useForm } from '@inertiajs/vue3';

defineProps({
    status: String,
});

const form = useForm({
    email: '',
});

const submit = () => {
    form.post(route('password.email'));
};
</script>

<template>
    <Head title="Forgot Password" />

    <div class="min-h-screen bg-slate-900 flex flex-col justify-center items-center p-6 font-sans">
        <div class="w-full max-w-md bg-slate-800 p-8 sm:p-10 rounded-[1.5rem] shadow-2xl border border-slate-700 animate-fade-in">
            
            <h2 class="text-2xl font-black text-white uppercase tracking-tighter mb-4 text-center">Lupa Password?</h2>
            
            <div class="mb-6 text-sm font-medium text-slate-400 text-center">
                Tenang bos, masukin email akun lu di bawah. Kita bakal kirim link buat bikin password baru.
            </div>

            <div v-if="status" class="mb-6 p-4 bg-emerald-500/10 border border-emerald-500/20 text-emerald-400 rounded-xl text-sm font-bold text-center">
                {{ status }}
            </div>

            <form @submit.prevent="submit" class="space-y-5">
                <div>
                    <label class="block text-xs font-black text-slate-400 uppercase tracking-widest mb-2">Email Anda</label>
                    <input type="email" v-model="form.email" class="w-full px-5 py-4 bg-slate-900 border border-slate-700 rounded-2xl focus:border-blue-500 text-white outline-none transition" required autofocus>
                    <p v-if="form.errors.email" class="text-rose-400 text-xs mt-2 font-bold">{{ form.errors.email }}</p>
                </div>

                <button type="submit" :disabled="form.processing" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-black py-4 rounded-2xl transition shadow-lg shadow-blue-900/50 uppercase tracking-widest text-sm disabled:opacity-50 mt-4">
                    {{ form.processing ? 'MENGIRIM LINK...' : 'KIRIM LINK RESET' }}
                </button>
                
                <div class="text-center mt-6">
                    <Link :href="route('login')" class="text-sm font-bold text-slate-400 hover:text-white transition">Kembali ke Login</Link>
                </div>
            </form>
        </div>
    </div>
</template>

<style scoped>
@keyframes fade-in { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
.animate-fade-in { animation: fade-in 0.4s ease-out forwards; }
</style>