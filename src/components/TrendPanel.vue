<template>
    <v-card class="mt-4 mx-auto" min-width="360" max-width="450">
        <v-sparkline :labels="$props.date.map(pickDate)" :value="$props.rate" line-width="2" padding="16"></v-sparkline>

        <v-card-text>
            <h3> {{ $props.ccy }} </h3>
            <v-divider class="my-2"></v-divider>
            <v-icon class="mr-2" small>
                mdi-clock
            </v-icon>
            <span class="text-caption grey--text font-weight-light">updated on {{ $props.date[$props.date.length - 1] }}</span>
        </v-card-text>
    </v-card>
</template>

<script>
export default {
    data: () => ({
    }),
    props: {
        minWidth: Number,
        ccy: String,
        rate: Array,
        date: Array
    },
    methods: {
        pickDate(date) {
            return date.substring(date.lastIndexOf("-") + 1, date.length);
        }
    }
};
</script>

<style>
.v-sheet--offset {
    top: -24px;
    position: relative;
}
</style>
