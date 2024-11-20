<template>
    <div id="tracker">
        <h1>Goals:</h1>
        <div v-if="loading">Loading...</div>
        <div v-else-if="error">Fucky-wucky UwU: {{ error }}</div>
        <div v-else>
            <p>goals: {{ goals }}</p>
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
                const careerRegSeason = data.careerTotals?.regularSeason;
                goals.value = careerRegSeason.goals ?? 0;
                
                     
            } catch (e) {
                error.value = e.message;
            } finally {
                loading.value = false;
            }
        };

        onMounted(fetchGoals);

        return {goals, loading, error };
    },
};
</script>