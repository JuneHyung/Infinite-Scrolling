<template>
    <div class="col-12 col-lg-8">
        <v-sheet height="64">
            <v-toolbar flat>
                <v-spacer></v-spacer>
                <v-btn fab text small color="grey darken-2" @click="prev">
                    <v-icon small> mdi-chevron-left </v-icon>
                </v-btn>
                <v-toolbar-title v-if="$refs.calendar">
                    {{ $refs.calendar.title }}
                </v-toolbar-title>
                <v-btn fab text small color="grey darken-2" @click="next">
                    <v-icon small> mdi-chevron-right </v-icon>
                </v-btn>
                <v-spacer></v-spacer>
                <v-btn outlined class="mr-4" color="grey darken-2" @click="focus = today">
                    Today
                </v-btn>
            </v-toolbar>
        </v-sheet>
        <v-sheet height="760">
            <v-calendar
                ref="calendar"
                v-model="focus"
                color="primary"
                :events="events"
                type="month"
                @click:event="showEvent"
                @click:more="viewDay"
                @click:date="viewDay"
                @change="updateRange"
            ></v-calendar>
            <v-menu
                v-model="selectedOpen"
                :close-on-content-click="false"
                :activator="selectedElement"
                offset-x
            >
                <v-card color="grey lighten-4" max-width="450px" flat>
                    <v-toolbar dark>
                        <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
                        <v-spacer></v-spacer>
                    </v-toolbar>
                    <v-card-text>
                        <span v-html="selectedEvent.description" style="color: black"></span>
                    </v-card-text>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn text color="secondary" @click="selectedOpen = false"> Cancel </v-btn>
                    </v-card-actions>
                </v-card>
            </v-menu>
        </v-sheet>
    </div>
</template>

<script>
import { mapState } from 'vuex';
export default {
    data() {
        return {
            focus: '',
            selectedEvent: {
                name: '',
                description: '',
            },
            selectedElement: null,
            selectedOpen: false,
            events: [],
            colors: ['#b2b1f0', '#b2d5', '#c945bc'],
        };
    },
    computed: {
        ...mapState(['todo', 'today']),
    },
    props: {
        items: Array,
    },
    created() {
        this.getTodoList();
        this.setCurToday();
    },
    methods: {
        async getTodoList() {
            await this.$store.dispatch('GET_ALL_TODOLIST');
            await this.updateRange();
        },
        prev() {
            this.$refs.calendar.prev();
        },
        next() {
            this.$refs.calendar.next();
        },
        setCurToday() {
            this.focus = this.today;
        },
        viewDay({ date }) {
            this.focus = date;
        },
        showEvent({ nativeEvent, event }) {
            const open = () => {
                this.selectedEvent = event;
                this.selectedElement = nativeEvent.target;
                setTimeout(() => {
                    this.selectedOpen = true;
                }, 10);
            };

            if (this.selectedOpen) {
                this.selectedOpen = false;
                requestAnimationFrame(function () {
                    requestAnimationFrame(() => open());
                });
            } else {
                open();
            }

            nativeEvent.stopPropagation();
        },
        updateRange() {
            const todoEvents = [];

            for (let i = 0; i < this.todo.length; i++) {
                let colorIdx = this.items.indexOf(this.todo[i].todoStatus.toUpperCase());
                todoEvents.push({
                    name: this.todo[i].todoTitle,
                    start: this.todo[i].startDate,
                    end: this.todo[i].endDate,
                    color: this.colors[colorIdx],
                    description: this.todo[i].todoContent,
                });
            }

            this.events = todoEvents;
        },
    },
};
</script>

<style scoped></style>
