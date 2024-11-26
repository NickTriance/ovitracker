<template>
    <div id="tracker">
        <h1>Goals:</h1>
        <div v-if="loading">Loading...</div>
        <div v-else-if="error">Fucky-wucky UwU: {{ error }}</div>
        <div v-else>
            <p>goals: {{ goals }}</p>
            <p>goals to tie: {{ goalsToTie }}</p>
            <p>goalsToBreak: {{ goalsToBreak }}</p>
            <p>current scoring rate: {{ currentScoringRate }} goals per game</p>
            <p>At his current pace, he should tie the record in {{ gamesToTie }} games, and break it in {{ gamesToBreak }} games.</p>
        </div>
    </div>
</template>

<script>
import { ref, onMounted } from 'vue';

export default {
    name: 'App',

    setup() {
        const goals = ref(null);
        const loading = ref(true);
        const error = ref(null);
        const goalsToTie = ref(null)
        const goalsToBreak = ref(null)
        const currentScoringRate = ref(null);
        const gamesToTie = ref(null);
        const gamesToBreak = ref(null);

        const fetchGoals = async () => {
            try {
                // make the request to nhl api. We'll just hardcode this to go to ovi.
                //const response = await fetch('https://api-web.nhle.com/v1/player/8471214/landing');
                const response = await fetch('/api/v1/player/8471214/landing')
                if (!response.ok) {
                    throw new Error('Failed to fetch data from NHL api');
                }

                const data = await response.json();
                console.log(data);
                //TODO: use last 5 games to calculate average scoring pace, and use that to estimate when the record will be broken. default to career average if he's on a drougt
                const careerRegSeason = data.careerTotals?.regularSeason;
                const last5Games = data.last5Games;
                let goalsLast5Games = 0
                for (let i = 0; i < last5Games.length; i++) {
                    goalsLast5Games += last5Games[i].goals;
                }

                if (goalsLast5Games != 0) {
                    currentScoringRate.value = goalsLast5Games / 5;
                } else {
                    currentScoringRate.value = careerRegSeason.goals / careerRegSeason.gamesPlayed;
                }

                goals.value = careerRegSeason.goals ?? 0;
                goalsToTie.value = 894 - goals.value;
                goalsToBreak.value = goalsToTie.value + 1;
                gamesToTie.value = Math.ceil(goalsToTie.value / currentScoringRate.value);
                gamesToBreak.value = Math.ceil(goalsToBreak.value / currentScoringRate.value);
                     
            } catch (e) {
                error.value = e.message;
            } finally {
                loading.value = false;
            }
        };

        onMounted(fetchGoals);

        return {goals, loading, error, goalsToTie, goalsToBreak, currentScoringRate, gamesToTie, gamesToBreak };
    },
};
</script>