<template>
    <div class="w-36">
        <input class="border p-3"
            ref="input"
            :value="selected_value_display"
            @change="updateSelectedValue($event.target.value)"
        />

        <div class="flex bg-white border border-t-0 shadow"
            :style="{ height: container_height + 'px' }"
        >
            <div class="flex-1 border-r border-grey-lighter overflow-scroll"
                ref="hour"
                :style="{ height: container_height + 'px' }"
            >
                <div v-for="hour in hours"
                    class="text-base cursor-pointer px-3"
                    :class="hour == selected_hour ? 'bg-blue-light text-white' : 'hover:bg-grey-lighter'"
                    :key="hour.value"
                    :style="{ height: row_height + 'px', lineHeight: row_height + 'px' }"
                    @click="selectHour(hour)"
                >
                    {{ hour.display }}
                </div>
            </div>

            <div class="flex-1 border-r border-grey-lighter overflow-scroll"
                :style="{ height: container_height + 'px' }"
            >
                <div v-for="minute in minutes"
                    class="text-base cursor-pointer px-3"
                    :class="minute == selected_minute ? 'bg-blue-light text-white' : 'hover:bg-grey-lighter'"
                    :key="minute.value"
                    :style="{ height: row_height + 'px', lineHeight: row_height + 'px' }"
                    @click="selectMinute(minute)"
                >
                    {{ minute.display }}
                </div>
            </div>

            <div class="flex-1 overflow-scroll">
                <div class="text-base cursor-pointer px-3"
                    :class="selected_am_pm == 'am' ? 'bg-blue-light text-white' : 'hover:bg-grey-lighter'"
                    :style="{ height: row_height + 'px', lineHeight: row_height + 'px' }"
                    @click="selectAmPm('am')"
                >
                    am
                </div>

                <div class="text-base cursor-pointer px-3"
                    :class="selected_am_pm == 'pm' ? 'bg-blue-light text-white' : 'hover:bg-grey-lighter'"
                    :style="{ height: row_height + 'px', lineHeight: row_height + 'px' }"
                    @click="selectAmPm('pm')"
                >
                    pm
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import moment from 'moment';

export default {
    props: {
        hour_start: { default: 0 },
        hour_end: { default: 23 },
        hour_scroll_to: { default: 0 },
        minute_interval: { default: 15 },
        rows: { default: 6 },
        row_height: { default: 34 },
        value: {},
        format: {},
    },
    data() {
        return {
            entered_hour: null,
            entered_minute: null,
            entered_am_pm: null,
            selected_hour: null,
            selected_minute: null,
            selected_am_pm: null,
        };
    },
    computed: {
        container_height() {
            return this.rows * this.row_height;
        },
        hours() {
            return Array.from(
                {
                    length:
                        Math.min(this.hour_end, 23) -
                        Math.max(this.hour_start, 0) +
                        1,
                },
                (v, k) => k + this.hour_start
            ).map(hour => {
                return {
                    display:
                        hour == 0 ? '12' : String(hour < 13 ? hour : hour - 12),
                    value: String(hour).padStart(2, '0'),
                };
            });
        },
        minutes() {
            return Array.from(
                { length: Math.ceil(60 / this.minute_interval) },
                (v, k) => k * this.minute_interval
            ).map(minutes => {
                return {
                    display: ':' + String(minutes).padStart(2, '0'),
                    value: String(minutes).padStart(2, '0'),
                };
            });
        },
        hour_display() {
            return this.selected_hour
                ? this.selected_hour.display
                : this.entered_hour;
        },
        minute_display() {
            return this.selected_minute
                ? this.selected_minute.display
                : ':' + String(Number(this.entered_minute)).padStart(2, '0');
        },
        am_pm_display() {
            return this.selected_am_pm
                ? this.selected_am_pm
                : this.entered_am_pm;
        },
        selected_value_display() {
            return (this.hour_display && this.minute_display && this.am_pm_display)
                ? (this.hour_display + this.minute_display + ' ' + this.am_pm_display)
                : '';
        },
    },
    methods: {
        clearTime() {
            this.entered_hour = null;
            this.entered_minute = null;
            this.entered_am_pm = null;
            this.selected_hour = null;
            this.selected_minute = null;
            this.selected_am_pm = null;
        },
        formatFromInput(value) {
            return this.format
                ? moment(value, this.format).format('h:mm a')
                : value;
        },
        formatForOutput(value) {
            return this.format
                ? moment(value, 'h:mm a').format(this.format)
                : value;
        },
        getCorrespondingAmHour(pm_hour) {
            if (pm_hour) {
                return this.hours.find(hour => {
                    return hour.value == Number(pm_hour.value) - 12;
                });
            }
        },
        getCorrespondingPmHour(am_hour) {
            if (am_hour) {
                return this.hours.find(hour => {
                    return hour.value == Number(am_hour.value) + 12;
                });
            }
        },
        getMinute(minute) {
            return this.minutes.find(minute_object => {
                return Number(minute_object.value) == Number(minute);
            });
        },
        getHour(hour_24) {
            return this.hours.find(hour_object => {
                return Number(hour_object.value) == Number(hour_24);
            });
        },
        scrollToHour(hour) {
            var selected = hour ? hour : -1;

            if (! hour && this.selected_hour) {
                selected = this.hours.findIndex(hour => {
                    return hour == this.selected_hour;
                });
            }

            if (selected > -1) {
                this.$refs.hour.scrollTo(0, selected * this.row_height);
            }
        },
        selectAmPm(am_pm) {
            if (this.selected_am_pm != am_pm) {
                this.selected_am_pm = am_pm;
                this.switchAmPm(am_pm);
            }
        },
        selectHour(hour) {
            this.selected_hour = hour;

            if (! this.selected_minute && ! this.entered_minute) {
                this.selectMinute(this.getMinute(0));
            }

            this.setAmPm(hour.value);
        },
        selectMinute(minute) {
            this.selected_minute = minute;
        },
        setAmPm(hour) {
            this.selected_am_pm = hour < 12 ? 'am' : 'pm';
        },
        switchAmPm(am_pm) {
            switch (am_pm) {
                case 'am':
                    this.selected_hour = this.getCorrespondingAmHour(
                        this.selected_hour
                    );
                    break;
                case 'pm':
                    this.selected_hour = this.getCorrespondingPmHour(
                        this.selected_hour
                    );
                    break;
            }

            this.scrollToHour();
        },
        updateSelectedValue(time) {
            let parsed_time = this.parseTime(time);

            if (parsed_time) {
                this.entered_hour = String(parsed_time.hours_12);
                this.entered_minute = String(parsed_time.minutes);
                this.entered_am_pm = parsed_time.am_pm;
                this.selected_hour = this.getHour(parsed_time.hours_24);
                this.selected_minute = this.getMinute(parsed_time.minutes);
                this.selected_am_pm = parsed_time.am_pm;
                this.scrollToHour();
            } else {
                this.clearTime();
            }

            this.$refs.input.$refs.input.blur();
        },
        parseTime(time_string) {
            var hours_24;
            var match = time_string.match(/(\d+)(?::(\d\d))?\s*(p?)/i);

            if (! match) {
                return;
            }

            var [_, hours, minutes, am_pm] = match;

            am_pm = am_pm.toLowerCase() == 'p' ? 'pm' : 'am';
            hours_24 = hours;

            if (parseInt(hours) == 0) {
                am_pm = 'am';
                hours = 12;
            } else if (parseInt(hours) > 12) {
                am_pm = 'pm';
                hours = parseInt(hours) - 12;
            } else if (am_pm == 'pm' && parseInt(hours) < 12) {
                hours_24 = parseInt(hours) + 12;
            }

            return {
                hours_12: parseInt(hours),
                hours_24: parseInt(hours_24),
                minutes: parseInt(minutes) || 0,
                am_pm: am_pm,
            };
        },
    },
    mounted() {
        if (this.hour_scroll_to) {
            this.scrollToHour(this.hour_scroll_to);
        }

        if (this.value) {
            this.updateSelectedValue(this.formatFromInput(this.value));
        }
    },
    watch: {
        selected_value_display(val) {
            this.$emit('input', this.formatForOutput(val));
        },
    },
};
</script>
