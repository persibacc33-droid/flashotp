<script setup>
import { ref } from 'vue';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Head } from '@inertiajs/vue3';

const props = defineProps({
    referralCode: String,
    referralLink: String,
    referrals: Array
});

const isCopied = ref(false);

const copyLink = () => {
    navigator.clipboard.writeText(props.referralLink);
    isCopied.value = true;
    setTimeout(() => isCopied.value = false, 3000);
};
</script>

<template>
    <Head title="Program Referral" />

    <AuthenticatedLayout>
        <div class="max-w-4xl mx-auto pb-20 mt-4 text-slate-200 animate-fade-in">
            
            <div class="bg-slate-800 p-8 rounded-[1rem] border border-slate-700 shadow-2xl mb-8 flex flex-col md:flex-row justify-between items-center gap-6 relative overflow-hidden">
                
                <div>
                    <h1 class="text-3xl font-black text-white tracking-tighter uppercase">Program Referral</h1>
                    <p class="text-slate-400 font-medium text-sm mt-1">Ajak teman bergabung dan nikmati bonus cuannya bersama!</p>
                </div>
                <div class="bg-blue-500/10 border border-blue-500/20 px-6 py-3 rounded-2xl text-center">
                    <span class="block text-[10px] font-black text-blue-400 uppercase tracking-widest mb-1">Total Teman</span>
                    <span class="text-3xl font-black text-white">{{ referrals.length }}</span>
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
                
                <div class="md:col-span-1 space-y-6">
                    <div class="bg-slate-800 p-8 rounded-[1rem] border border-slate-700 shadow-xl text-center">
                        <div class="w-16 h-16 bg-blue-500/10 text-blue-400 rounded-full flex items-center justify-center mx-auto mb-6">
                            <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1"></path></svg>
                        </div>
                        <h3 class="font-black text-white uppercase tracking-widest text-sm mb-4">Kode Referral Kamu</h3>
                        <div class="bg-slate-900 border border-slate-700 p-4 rounded-xl text-2xl font-black text-blue-400 tracking-[0.3em] mb-4 select-all">
                            {{ referralCode }}
                        </div>
                        
                        <button @click="copyLink" 
                            class="w-full py-4 rounded-xl font-black text-xs uppercase tracking-widest transition-all shadow-lg"
                            :class="isCopied ? 'bg-emerald-500/20 text-emerald-400 border border-emerald-500/30' : 'bg-blue-600 hover:bg-blue-500 text-white shadow-blue-900/50'">
                            {{ isCopied ? 'Link Tersalin!' : 'Salin Link Ajak Teman' }}
                        </button>
                    </div>
                </div>

                <div class="md:col-span-2 bg-slate-800 p-8 rounded-[1rem] border border-slate-700 shadow-xl flex flex-col h-full">
                    <div class="flex justify-between items-center mb-6">
                        <h3 class="font-black text-white uppercase tracking-widest text-sm">Teman Bergabung</h3>
                    </div>

                    <div class="flex-1 overflow-y-auto pr-2 max-h-[350px] custom-scrollbar-dark">
                        <template v-if="referrals.length > 0">
                            <div class="space-y-3">
                                <div v-for="friend in referrals" :key="friend.email" class="bg-slate-900 p-4 rounded-2xl border border-slate-700 flex justify-between items-center hover:border-blue-500/30 transition">
                                    <div class="flex items-center gap-4">
                                        <div class="w-10 h-10 bg-slate-800 rounded-full flex items-center justify-center font-black text-slate-400 uppercase">
                                            {{ friend.name.charAt(0) }}
                                        </div>
                                        <div>
                                            <p class="font-black text-white text-sm">{{ friend.name }}</p>
                                            <p class="text-[10px] text-slate-500 font-bold mt-0.5">Mendaftar: {{ new Date(friend.created_at).toLocaleDateString('id-ID') }}</p>
                                        </div>
                                    </div>
                                    <span class="bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 px-3 py-1 rounded-lg text-[10px] font-black uppercase tracking-widest">
                                        Aktif
                                    </span>
                                </div>
                            </div>
                        </template>
                        <template v-else>
                            <div class="text-center py-12 opacity-50">
                                <svg class="w-12 h-12 text-slate-500 mx-auto mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z"></path></svg>
                                <p class="text-sm font-bold text-slate-400">Belum ada teman yang bergabung.<br>Ayo sebarin link kamu!</p>
                            </div>
                        </template>
                    </div>
                </div>

            </div>
        </div>
    </AuthenticatedLayout>
</template>

<style scoped>
@keyframes fade-in { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
.animate-fade-in { animation: fade-in 0.4s ease-out forwards; }
.custom-scrollbar-dark::-webkit-scrollbar { width: 4px; }
.custom-scrollbar-dark::-webkit-scrollbar-track { background: transparent; }
.custom-scrollbar-dark::-webkit-scrollbar-thumb { background: #475569; border-radius: 10px; }
</style>