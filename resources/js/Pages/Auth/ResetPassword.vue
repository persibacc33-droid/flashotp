<script setup>
import { Head, useForm } from '@inertiajs/vue3';

const props = defineProps({
    email: String,
    token: String,
});

const form = useForm({
    token: props.token,
    email: props.email,
    password: '',
    password_confirmation: '',
});

const submit = () => {
    form.post(route('password.store'), {
        onFinish: () => form.reset('password', 'password_confirmation'),
    });
};
</script>

<template>
    <Head title="Reset Password" />

    <div class="min-h-screen bg-slate-900 flex flex-col justify-center items-center p-6 font-sans">
        <div class="w-full max-w-md bg-slate-800 p-8 sm:p-10 rounded-[1.5rem] shadow-2xl border border-slate-700 animate-fade-in">
            
            <h2 class="text-2xl font-black text-white uppercase tracking-tighter mb-8 text-center">Bikin Password Baru</h2>

            <form @submit.prevent="submit" class="space-y-5">
                <div>
                    <label class="block text-xs font-black text-slate-400 uppercase tracking-widest mb-2">Email</label>
                    <input type="email" v-model="form.email" class="w-full px-5 py-4 bg-slate-900 border border-slate-700 rounded-2xl focus:border-blue-500 text-white outline-none transition" required autofocus>
                    <p v-if="form.errors.email" class="text-rose-400 text-xs mt-2 font-bold">{{ form.errors.email }}</p>
                </div>

                <div>
                    <label class="block text-xs font-black text-slate-400 uppercase tracking-widest mb-2">Password Baru</label>
                    <input type="password" v-model="form.password" class="w-full px-5 py-4 bg-slate-900 border border-slate-700 rounded-2xl focus:border-blue-500 text-white outline-none transition" required>
                    <p v-if="form.errors.password" class="text-rose-400 text-xs mt-2 font-bold">{{ form.errors.password }}</p>
                </div>

                <div>
                    <label class="block text-xs font-black text-slate-400 uppercase tracking-widest mb-2">Konfirmasi Password Baru</label>
                    <input type="password" v-model="form.password_confirmation" class="w-full px-5 py-4 bg-slate-900 border border-slate-700 rounded-2xl focus:border-blue-500 text-white outline-none transition" required>
                    <p v-if="form.errors.password_confirmation" class="text-rose-400 text-xs mt-2 font-bold">{{ form.errors.password_confirmation }}</p>
                </div>

                <button type="submit" :disabled="form.processing" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-black py-4 rounded-2xl transition shadow-lg shadow-blue-900/50 uppercase tracking-widest text-sm mt-6 disabled:opacity-50">
                    {{ form.processing ? 'MEMPROSES...' : 'SIMPAN PASSWORD' }}
                </button>
            </form>
        </div>
    </div>
</template>

<style scoped>
@keyframes fade-in { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
.animate-fade-in { animation: fade-in 0.4s ease-out forwards; }
</style>