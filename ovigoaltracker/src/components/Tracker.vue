<template>
    <div id="tracker">
        <div v-if="loading">Loading...</div>
        <div v-else-if="error">Fucky-wucky UwU: {{ error }}</div>
        <div v-else>
            <div v-if="recordBroken">
                <h1>Yes, Alex Ovechkin is currently the NHL's all time goals leader.</h1>
                <h2>To date, he has scored {{ goals }} goals, which is {{ goalDelta }} more than Wayne Gretzky's 894.</h2>
            </div>
            <div v-else>
                <h1>Nope, he hasn't broken it yet.</h1>
                <h2>He's currently sitting at {{ goals }} career goals.</h2>
                <p>He needs {{ goalDelta }} more goals to tie the record, and {{ goalsToBreak }} goals to break it.</p>
                <p>Ovi is currently scoring at a pace of {{ currentScoringRate }} goals per game.</p>
                <p>At his current pace, he should tie the record in {{ gamesToTie }} games, and break it in {{ gamesToBreak }} games.</p>
            </div>
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
        const goalDelta = ref(null)
        const goalsToBreak = ref(null)
        const currentScoringRate = ref(null);
        const gamesToTie = ref(null);
        const gamesToBreak = ref(null);
        const recordBroken = ref(null);

        const fetchGoals = async () => {
            
            try {

                // make the request to nhl api. We'll just hardcode this to go to ovi.
                const response = await fetch('/api/v1/player/8471214/landing')
                if (!response.ok) {
                    throw new Error('Failed to fetch data from NHL api');
                }

                const data = await response.json();
                console.log(data);
                const careerRegSeason = data.careerTotals?.regularSeason;

                // Figure out his current goal scoring rate using data from his last 5 games
                // This isn't super accurate, and is likely to vary wildly from game to game as
                // his scoring rate changes, but it's fine for a rough ballpark.
                // A better method would be to look at shooting rates and shooting percentages, 
                // ideally broken down by opponent, and utilize that to estimate how many goals 
                // he will score in the upcoming games. This would be a good use case for
                // machine learning. 

                const last5Games = data.last5Games;
                let goalsLast5Games = 0
                for (let i = 0; i < last5Games.length; i++) {
                    goalsLast5Games += last5Games[i].goals;
                }
                if (goalsLast5Games != 0) {
                    currentScoringRate.value = goalsLast5Games / 5;
                } else {
                    // If he hasn't scored a goal in his last 5 games, we'll default to his career scoring rate.
                    currentScoringRate.value = careerRegSeason.goals / careerRegSeason.gamesPlayed;
                }

                goals.value = careerRegSeason.goals ?? 0;
                goalDelta.value = Math.abs(894 - goals.value); 
                recordBroken.value = goals.value > 894;
                goalsToBreak.value = goalDelta.value + 1;

                // Round games up to the nearest whole game.
                gamesToTie.value = Math.ceil(goalDelta.value / currentScoringRate.value);
                gamesToBreak.value = Math.ceil(goalsToBreak.value / currentScoringRate.value);
                     
            } catch (e) {
                error.value = e.message;
            } finally {
                loading.value = false;
            }
        };

        onMounted(fetchGoals);

        return {goals, loading, error, goalDelta, goalsToBreak, currentScoringRate, gamesToTie, gamesToBreak, recordBroken };
    },
};
</script>