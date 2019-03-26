<template>
    <div id="app" class="flex justify-center h-screen w-screen bg-grey-lighter p-10">
        <InputDatepicker
            class="z-10 mr-2"
            :always_show="true"
            format="MM/DD/YYYY"
            :start="new Date().setMonth(new Date().getMonth() - 9)"
            :end="new Date().setFullYear(new Date().getFullYear() + 2)"
            v-model="selected_date"
        />

        <InputTimepicker
            class="z-10 ml-2"
            :hour_scroll_to="8"
            format="HH:mm:ss"
            v-model="selected_time"
        />
    </div>
</template>

<script>
import InputDatepicker from './components/InputDatepicker'
import InputTimepicker from './components/InputTimepicker'

export default {
    components: {
        InputDatepicker,
        InputTimepicker,
    },
    data() {
        return {
            selected_date: null,
            selected_time: null,
        }
    },
}
</script>
