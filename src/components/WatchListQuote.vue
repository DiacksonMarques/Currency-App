<template>
    <ListQuotes :quotes="quotes"  :listenQuotes="listenQuotes" @unlisten="onUnList"/>
    <div class="mt-2 text-right">
        <cite class="text-small">
            Atualizara novamente em <b>{{nextUpdateTime}} segundos</b>
        </cite>
    </div>
</template>

<script>
    import { reactive, ref, toRefs } from '@vue/reactivity';
    import api from '../services/api';
    import ListQuotes from './ListQuotes';
    import { onMounted, onUnmounted, watch } from '@vue/runtime-core';

    const CURRENT_UPDATE_TIME = 30;

    export default {
        props: {
            listenQuotes: {type: Array, required: true},
        },
        components:{
            ListQuotes
        },
        emits: ['unlisten'],
        setup(props, {emit}){
            const interval = ref(null);
            const quotes = ref({});
            const nextUpdateTime = ref(CURRENT_UPDATE_TIME);

            const methods = reactive({
                onUnList(code){
                    emit('unlisten', code)
                },

                restatInterval(){
                    clearInterval(interval.value);
                    nextUpdateTime.value = CURRENT_UPDATE_TIME;
                    interval.value = setInterval(() => {
                        nextUpdateTime.value -= 1;
                        if(nextUpdateTime.value === 0){
                            nextUpdateTime.value = CURRENT_UPDATE_TIME;
                            this.refresh();
                        }
                    }, 1000)
                },

                async refresh(){
                    const {data} = await api.listen(props.listenQuotes);
                    quotes.value = data
                }
            });

            onMounted(() => {
                methods.refresh();
                methods.restatInterval();
            });

            onUnmounted(()=>{
                clearInterval(interval.value)
            });

            watch(props, () =>{
                methods.refresh();
            });

            return { 
                ...toRefs(methods),
                quotes, 
                nextUpdateTime }
        }
    }
</script>