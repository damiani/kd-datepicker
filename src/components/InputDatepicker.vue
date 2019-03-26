<template>
    <div v-on-clickaway="hideCalendar" class="flex flex-col w-64 relative">
        <input
            class="text-input border w-full p-3"
            :class="{ 'rounded-b-none': showing_calendar }"
            :value="selected_value_display"
            @change="updateSelectedValue($event.target.value)"
            @click="showCalendar"
            @focus="showCalendar"
            @keydown.tab="hideCalendar"
            @keyup.enter="hideCalendar"
        />

        <div class="focus:outline-none shadow" tabindex="0" ref="datepicker">
            <date-picker
                v-show="showing_calendar"
                v-model="selected_value"
                :attributes="attributes"
                class="w-64"
                :class="{ 'calendar-absolute': ! always_show }"
                :is-inline="true"
                :max-date="max_date"
                :min-date="min_date"
                mode="single"
                :show-day-popover="false"
                :theme-styles="theme_styles"
                :is-required="true"
                tint-color="#006da8"
                @input="hideCalendar"
            ></date-picker>
        </div>
    </div>
</template>

<script>
import moment from 'moment';
import {mixin as clickaway} from 'vue-clickaway';
import {setupCalendar, DatePicker}  from 'v-calendar';
import 'v-calendar/lib/v-calendar.min.css';

export default {
    components: {
        DatePicker,
    },
    mixins: [
        clickaway
    ],
    props: {
        always_show: { default: false },
        format: { default: 'YYYY-MM-DD hh:mm:ss' },
        start: {},
        end: {},
        value: {},
    },
    data() {
        return {
            attributes: [
                {
                    key: 'today',
                    contentStyle: {
                        color: '#46a7db',
                        fontWeight: 500,
                    },
                    dates: new Date(),
                }
            ],
            selected_value: this.getJsDate(this.value),
            showing_calendar: this.always_show,
            theme_styles: {
                dayCell: {
                    fontFamily: "'Aspira', sans-serif",
                },
                headerArrows: {
                    color: "#46a7db",
                },
                headerTitle: {
                    fontSize: '16px',
                    fontFamily: "'Aspira', sans-serif",
                },
                weekdays: {
                    fontFamily: "'Aspira', sans-serif",
                },
                wrapper: {
                    backgroundColor: '#ffffff',
                    border: '1px solid #e5e1dc',
                    borderTop: '0',
                },
            },
        }
    },
    computed: {
        entered_date_is_valid() {
            return this.selected_value !== 'Invalid Date' ? this.entered_date_is_after_min && this.entered_date_is_before_max : false;
        },
        entered_date_is_after_min() {
            return this.min_date ? this.selected_value >= this.min_date : true;
        },
        entered_date_is_before_max() {
            return this.max_date ? this.selected_value <= this.max_date : true;
        },
        max_date() {
            let end;

            if (this.end) {
                end = this.getJsDate(this.end);
                end.setHours(23, 59, 59, 999);

            }

            return end;
        },
        min_date() {
            let start;

            if (this.start) {
                start = this.getJsDate(this.start);
                start.setHours(0, 0, 0, 0);
            }

            return start;
        },
        selected_value_display() {
            return this.entered_date_is_valid ? moment(this.selected_value).format('l') : null;
        },
        selected_value_formatted() {
            return this.format ? moment(this.selected_value).format(this.format) : this.selected_value_display;
        },
    },
    methods: {
        getJsDate(date) {
            return date ? moment(date).toDate() : null;
        },
        hideCalendar() {
            if (this.always_show) {
                return;
            }

            this.showing_calendar = false;
        },
        showCalendar() {
            this.showing_calendar = true;
        },
        updateSelectedValue(value) {
            this.selected_value = new Date(value);
        },
    },
    watch: {
        selected_value_display() {
            this.$emit('input', this.selected_value_formatted);
        },
    },
    created() {
        setupCalendar({
            firstDayOfWeek: 1,
        });
    },
}
</script>

<style scoped>
.calendar-absolute {
    position: absolute;
    top: 44px;
}
</style>
